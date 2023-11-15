---
title: How to create check-ins via API
description: How to create check-ins via API
---
!!! note "Important"
    Check-ins are created using X-GPS Tracker. 
    All the description below is necessary only for exceptional cases, such as creating your Mobile Tracker app.

Step 1. Create a form from a template with [checkin/form/create](../resources/field_service/checkin.md#formcreate)
API call. 
In the X-GPS Tracker, the form is created when the template is selected by a user.

Step 2. Create files for photos of check-in with [checkin/image/create](../resources/field_service/checkin.md#imagecreate) and upload photo data (see below). 
In the X-GPS Tracker, checkin photos are created as each photo is added.

Step 3. Create form files with [checkin/form/file](../resources/field_service/checkin.md#formfilecreate) API call and upload their data (see below). 
In the X-GPS Tracker, form files are created when they are added when the form is filled out.

Step 4. Create a check-in itself with [checkin/create](../resources/field_service/checkin.md#create) API call, where all the data is attached.
If the form includes optional fields that should be left empty for your check-in, simply refrain from adding these fields to the form submission object.

***

## File upload process

This is how files can be uploaded to the platform. If you have multiple files to upload, be sure to add a brief delay between uploading each one to ensure a smooth process.

Using the API calls [checkin/image/create](../resources/field_service/checkin.md#imagecreate) and 
[checkin/form/file](../resources/field_service/checkin.md#formfilecreate) the app asks - may I provide you with the file?
This file has size of X MB, file name is example.png, metadata of the image should be in EXIF format. This is already 
saved as metadata for your image in this format, you need to copy this info and add into request as a JSON object.
The platform says, okay. I'm ready to receive this file from your app. It will provide you with the location and token if it is a local storage on your server:

```json
{
  "success": true,
  "value": {
    "file_id": 111,
    "url": "http://bla.org/bla",
    "expires": "2020-02-03 03:04:00",
    "file_field_name": "example.png",
    "fields": {
      "token": "a43f43ed4340b86c808ac"
    }
  }
}
```

And for Cloud version of the platform Amazon S3 is used:

```json
{
  "success": true,
  "value": {
    "file_id": 111,
    "url": "https://bla.s3.amazonaws.com/",
    "expires": "2020-02-03 03:04:00",
    "file_field_name": "file",
    "fields": {
      "policy": "<Base64-encoded policy string>",
      "key": "user/user1/${filename}",
      "success_action_status": "200",
      "x-amz-algorithm": "AWS4-HMAC-SHA256",
      "x-amz-credential": "AKIAIOSFODNN7EXAMPLE/20151229/us-east-1/s3/aws4_request",
      "x-amz-date": "20151229T000000Z",
      "x-amz-signature": "<signature-value>",
      "x-amz-server-side-encryption": "AES256",
      "content-type": "image/png"
    }
  }
}
```

After it, the app must send the file. It should contain information from the platform's reply to the app's request.

Here's an example of upload you must make after receiving such response (assuming you uploading image named `actual_file_name.png`):

Internal storage example:

```http
POST /bla HTTP/1.1 - part of URL after the host part.
Host: bla.org - only the host part.
Content-Length: 1325 - The content length in this case is the size of data being sent, measured in bytes.
Origin: http://bla.org
... other headers ...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryePkpFF7tjBAqx29L - boundary is a marker used to separate the different pieces of data sent in a multipart form. It can be generated by the app randomly.

------WebKitFormBoundaryePkpFF7tjBAqx29L
Content-Disposition: form-data; name="token"

a43f43ed4340b86c808ac - here add the token, received from the platform
------WebKitFormBoundaryePkpFF7tjBAqx29L
Content-Disposition: form-data; name="example.png"; filename="example.png" - the file name.
Content-Type: image/png

... contents of file goes here ... The contents of the file in a .png or other format will depend on the image itself but generally it will be composed of binary data that represents the colors and shapes of the image.
------WebKitFormBoundaryePkpFF7tjBAqx29L--
```

Amazon S3 example:

```http
POST / HTTP/1.1 - part of URL after the host part
Host:  https://bla.s3.amazonaws.com - only the host part.
Content-Length: 1972 - The content length in this case is the size of data being sent, measured in bytes.
Origin: https://bla.s3.amazonaws.com/
... other headers ...
Content-Type: multipart/form-data; boundary=WebAppBoundary - boundary is a marker used to separate the different pieces of data sent in a multipart form. It can be generated by the app randomly.

--WebAppBoundary
Content-Disposition: form-data; name="policy"
Content-Type: text/plain

eyJleHBpcmF0aW9uIjogIjIwMjMtMDMtMjdUMjE6MTU6MzYuMDczn1dfQ==
--WebAppBoundary
Content-Disposition: form-data; name="key"
Content-Type: text/plain

nj9relv6m52qp01t0wv47wyk1ozd309g/${filename}
--WebAppBoundary
Content-Disposition: form-data; name="success_action_status"
Content-Type: text/plain

200
--WebAppBoundary
Content-Disposition: form-data; name="x-amz-algorithm"
Content-Type: text/plain

AWS4-HMAC-SHA256
--WebAppBoundary
Content-Disposition: form-data; name="x-amz-credential"
Content-Type: text/plain

AKIAIBQ6SRB65EVSSRMA/20230327/eu-central-1/s3/aws4_request
--WebAppBoundary
Content-Disposition: form-data; name="x-amz-date"
Content-Type: text/plain

20230327T210036Z
--WebAppBoundary
Content-Disposition: form-data; name="x-amz-signature"
Content-Type: text/plain

2df7efa0c0e0c5b97d0d9483acd77c9ec37360df921b019a4c4a93180a6136ad
--WebAppBoundary
Content-Disposition: form-data; name="x-amz-server-side-encryption"
Content-Type: text/plain

AES256
--WebAppBoundary
Content-Disposition: form-data; name="file"; filename="actual_file_name.png"
Content-Type: image/png

... contents of file goes here ...
--WebAppBoundary--
```
