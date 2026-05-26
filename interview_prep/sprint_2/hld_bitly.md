Bit.ly is a URL shortening service that converts long URLs into shorter, manageable links. It also provides analytics for the shortened URLs.

## Requirements

### Functional Requirements
- Users should be able to submit a long URL and receive a shortened version.
  - Optionally, users should be able to specify a custom alias for their shortened URL (ie. "www.short.ly/my-custom-alias")
  - Optionally, users should be able to specify an expiration date for their shortened URL.
- Users should be able to access the original URL by using the shortened URL.

### Non Functional Requirements
- The system should ensure uniqueness for the short codes 
- The redirection should occur with minimal delay (< 100ms)
- Availability >> Consistency
- The system should scale to support 1B shortened URLs and 100M DAU


## Core Entities

- Original URL
- Short URL
- User

## API Design

- Create a Short URL
```
POST /urls
{
  "long_url": "https://www.example.com/some/very/long/url",
  "custom_alias": "optional_custom_alias",
  "expiration_date": "optional_expiration_date"
}
->
{
  "short_url": "http://short.ly/abc123"
}
```

- Access the URL
```
// Redirect to Original URL
GET /{short_url}
-> HTTP 302 Redirect to the original long URL
```

## High Level Design

### Users should be able to submit a long URL and receive a shortened version

- Validate the URL
- Generate a unique code (assume magic function).
- if user need a custom url check for collisions
- Store the data in db: shortURL, short code (or custom alias), Long URL, exp date

### Users should be able to access the original URL by using the shortened URL.

- Look up original URL from DB
- Redirect with 302 redirect

Also, evict stale URLs and expired URLs by running a BG job.

## Potential Deep Dives

### How can we ensure short urls are unique?
- Unique counter with base64 encoding

###  How can we ensure that redirects are fast?
- In Memory Cache: Redis
- Leveraging CDN/ Edge Computing

