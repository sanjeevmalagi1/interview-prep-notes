Dropbox is a cloud based file storage service that allows users to store and share files.
It provides a secure and relaiable way to store and access file form anywhere on any device.

Functional Requirements:
- Users should be able to upload a file from any device
- Users should be able to download a file from any device
- Users should be able to share a file with other users and view the files shared with them
- Users can automatically sync files across devices

out of scope:
- Users should be able to edit files
- Users should be able to view files without downloading them

Non-Functional Requirements:
- The system should be highly available (prioritizing availability over consistency).
- The system should support files as large as 50GB.
- The system should be secure and reliable. We should be able to recover files if they are lost or corrupted.
- The system should make upload, download, and sync times as fast as possible (low latency).

Core Entities:
- File
- FileMetadata
- User

API Design / System Interface

- Upload files:
```
POST /files
{
  file,
  fileMetadata
}

200 OK
```

- Download files:
```
GET /files/{file_id}

200
{
  file,
  fileMetadata
}
```

- Share File:
```
POST /files/{fileId}/share
{
  User[]
}

- Changes:
```
GET /files/changes?since={timestamp}

200
{
  ChangeEvent[
    {
      file_id,
      type: created | updated | deleted
    }
  ]
}
```

High Level Design
- Users should be able to upload a file from any device
  - we store the file metadata in a unstructured file storage like dynamodb.
  ```
  {
    "id": "123",
    "name": "file.txt",
    "size": 1000,
    "mimeType": "text/plain",
    "uploadedBy": "user1"
  }
  ```
  - Upload the File directly to the blob storage
    - Generate a Presigned URL
    - Save metadata with status: uploading
    - Once file upload is completed use S3 notification events to update the status.
    - 

- User should be able to download a file from any device
  - since we store files in blob storage we can enable a CDN.
  - we can generate a presigned url with permissions and endtime for security
  - once user gets the presigned url they can download the file directly from CDN.

- User should be able to share file with other users
  - Fully normalize the data
  - store user_id (PK) and file_id (SK) in a separate dynamodb table.
  - we can get all the files shared to the user by simple get query.

- Users can automatically sync files across devices
  - Local -> Remote
    - user updates a file on their local machine
    - Monitors the local Dropbox folder for changes using OS-specific file system events
    - When it detects a change, it queues the modified file for upload locally
    - uses upload API to send the changes to the server with metadata
    - if conflicts its resolved by last write wins strategy
  - Remote -> Local
    - Maintain last synced for each device/user.
    - Use a hybrid service where we use *websockets* to sync with periodic *polling* to keep up with changes.

- Tying it all together
  ![system design](/interview_prep/sprint_1/hld_dropbox.png)

- Potential Deep Dives
    - How can you support large files?

      - Limitations with uploading large file via single POST requests
        - Timeouts
        - Browser / Server Limitation
        - Network Limitations
        - User experience - user cannot see the progress

      To address limitations the user experience should include:
        - Progress Indicator
        - Resumable Uploads

      - To do that we need to do chunking. (ie. breaking down the files into small chunks) and upload them one at a time or some in parallel. Note that chunking should be done at client side (not server side). 

      With chunks, it's rather straightforward - progress indicator to the user keep track of each chunks like (uploading, uploaded, not-uploaded).
      `FileMetadata` can have schema to include a chunk field.
      ```
      {
        "id": "123",
        "name": "file.txt",
        "size": 1000,
        "mimeType": "text/plain",
        "uploadedBy": "user1",
        "status": "uploading",
        "chunks": [
          {
            "id": "chunk1",
            "status": "uploaded"
          },
          {
            "id": "chunk2",
            "status": "uploading"
          },
          {
            "id": "chunk3",
            "status": "not-uploaded"
          }
        ]
      }
      ```
      this also helps in resuming the uploads. 

      But how should we ensure this chunks field is kept in sync with the actual chunks that have been uploaded?
      - Server-Side Chunk verification.
      Let the client make a patch request to upload the chunks with a Etag (supported on S3's multipart file upload feature).
      ```
      PATCH /files/{fileId}/chunks
      Request:
      {
        "chunks": [
          {
            "id": "chunk1",
            "status": "uploaded"
            "ETag": 1
          },
        ]
      }
      ```

      Each chunk gets an ETag upon successful upload, which the client can include in the PATCH request to our backend. Our backend can then verify these ETags by calling S3's ListParts API, providing an efficient way to validate multiple chunks at once.

      how to uniquely identify a file and a chunk?
      - (1) Have I tried to upload this file before?
      - (2) If yes, which chunks have I already uploaded?

      we need to rely on a unique identifier that is derived from the file's content. This is called a fingerprint. using encryption sha-256 we can get a hash value. and store this value in as a field.

      For resumable uploads, the process involves not only fingerprinting the entire file but also generating fingerprints for each individual chunk. This chunk-level fingerprinting allows the system to precisely identify which parts of the file have already been transmitted.


      - The client will chunk the file into 5-10MB pieces and calculate a fingerprint for each chunk. 
      - The client will send a request to check if a file with the same fingerprint already exists for this user. If it does and has a status of "uploading", the client can resume the upload by fetching the existing chunk statuses.
      - If the file does not exist, the client will POST a request to initiate a multipart upload. The backend will call S3's CreateMultipartUpload API to get an uploadId, generate presigned URLs for each part, save the file metadata in the FileMetadata table with a status of "uploading", and return the uploadId along with presigned URLs for each chunk.
      - The client will then upload each chunk to S3 using its corresponding presigned URL (each part requires its own presigned URL with the uploadId and partNumber). After each chunk is uploaded, the client sends a PATCH request to our backend with the chunk status and ETag. Our backend can then verify the chunk uploads with S3's ListParts API before updating the chunks field in the FileMetadata table to mark the chunk as "uploaded".
      - Once all chunks in our chunks array are marked as "uploaded", the backend calls S3's CompleteMultipartUpload API with the list of part numbers and ETags. This tells S3 to assemble all the parts into a single object. Only after S3 confirms successful assembly does the backend update the FileMetadata table to mark the file as "uploaded".

  - How can we make uploads, downloads, and syncing as fast as possible?
    - For Download: we use CDNs. 
    - For Upload: Chunking gives option to parallel upload. 
    - Also use compression like Gzip. But make sure we match the threshold

  - How can we ensure the file security?
    - Encryption at transfer: user https.
    - Encryption at rest. Encode and save files on there.
    - Access Control. we already have a `shareList` which acts as a basic ACL.
      - Here presigned URLs also come handy.




  