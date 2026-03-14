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

      - To do that we need to do chunking. (ie. breaking down the files into small chunks) and upload them one at a time or 



  