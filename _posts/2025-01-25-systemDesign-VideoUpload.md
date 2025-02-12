---
layout: post
title: "System design basic - media file uploading"
date: 2025-01-25
description: "System design"
tag: System Design
---

# Large Video/Image Upload **(chunked/multipart uploads)**

Uploading large files in single request can lead to timeouts or failure due to network instability. Instead, breaking large files into smaller chunks and uploading them sequestially or in parallel can improve upload efficiency and reliability.

## Signed URLs

Signed URLs are specially-crafted URLs that grant access to objects in cloud, like S3 or Google Cloud.

Eg.

```
https://example-bucket.s3.amazonaws.com/my-file.txt?\
  X-Amz-Algorithm=AWS4-HMAC-SHA256&
  X-Amz-Credential=AKIAIOSFODNN7EXAMPLE%2F20220407%2Fus-east-1%2Fs3%2Faws4_request&\
  X-Amz-Date=20220407T123456Z&\
  X-Amz-Expires=3600&\
  X-Amz-SignedHeaders=host&\
  X-Amz-Signature=a9c8a7d1644c7b351ef3034f4a1b4c9047e891c7203eb3a9f29d8c7a74676d88
```

## upload large video and image files in chunks using signed URLs.

The process will be like below:

1. Divide the file into smaller chunks: (Generally on client side) need to balance the number of requests and the size of each chunk to optimize upload performance.

2. Request singed URLs for each chunk

3. Upload chunks using signed URLs: Using signed URLS to upload the chunk to cloud. sequential or parallel can be based on concurrency level and network conditions.

4. Confirm successful uploads and reassemble

- 1. Notify server the upload is complete
- 2. server reassemble the chunks into original file

5. Handle failed uploads:

- 1. retry with a new signed URL
- 2. implement an error-handling strategy
