# Dropbox Lite — REST API Guide

Dropbox Lite is a cloud based file-storage backend that provides APIs for account management, authentication, file uploads, folder management, file operations, and sharing.

This README explains how to use the API collection with an HTTP client and how to send requests manually without importing the collection.

## Table of Contents

- [Getting Started](#Getting-Started)
- [Importing the API Collection](#importing-the-api-collection)
- [Authentication](#authentication)
- [Recommended Testing Order](#recommended-testing-order)
- [API Reference](#api-reference)
    - [Health](#1.-health)
    - [Account and Authentication](#2.-account-and-authentication)
    - [File Operations](#3-file-operations)
    - [Folder Operations](#4-folder-operations)
    - [File Sharing](#5-file-sharing)
- [Using the APIs Without an HTTP Client](#using-the-apis-without-an-http-client)
- [Important Notes](#important-notes)


---

## Getting-Started

Before calling the APIs, make sure the Dropbox Lite backend is running and accessible from your machine.

You can test the health check of the application using 

```text
http://localhost:port/dropbox-lite/health
```

Replace `port` with the host and port where your application is running.

For example, if the application is running on another machine:

```text
http://192.168.1.10:8080/dropbox-lite
```

The examples use HTTP connection (even on the cloud).

## Importing-the-API-Collection

The repository contains individual HTTP request files organized into collections such as Auth, Files, Folder, and Sharing.

To use them:

1. Clone or download this repository.
2. Open your preferred API client, such as Postman, or another client that supports importing HTTP requests.
3. Import the provided collection or open the request files directly.
4. Locate the collection variable named `ip` (Set its value to your server's host and port) and  `json_web_token_016y` (Set the JWT token which you get after login into the application. The token can be fetched from the response headers -- Note remove the Bearer schema while pasting).

For example:

```text
ip = localhost:8080

if -- Bearer YOUR_JWT_TOKEN
json_web_token_016y = YOUR_JWT_TOKEN
```

## Authentication

Dropbox Lite uses a JWT bearer token for protected endpoints.

### Which endpoints require authentication?

The following endpoints do **not** require authentication:

- `GET /health`
- `POST /auth/otp`
- `POST /account/create`
- `POST /account/login`

All other endpoints in this guide should be called with a valid bearer token.

### Setting the JWT token

After logging in successfully, copy the JWT returned by the login response headers.
Configure the collection's `json_web_token_016y` variable with that token.

### Refreshing the token

Use the refresh endpoint when your current token needs to be refreshed, according to the server's token policy. The token can only be refresh at certain intervals like after 25 min of every current token generated

```http
POST /dropbox-lite/auth/refresh
Authorization: Bearer YOUR_JWT_TOKEN
```

copy the JWT returned from response headers.

---

## Recommended-Testing-Order

Follow this order when testing the APIs for the first time:

1. Check whether the server is running.
2. Request an OTP for your email address.
3. Create an account using the OTP.
4. Log in and obtain a JWT.
5. Create a folder if needed.
6. Initiate a file upload and complete it.
7. List, view, rename, copy, move, or archive files.
8. Share a file with another account.
9. Test account deletion only when you intend to delete the test account.

---

# API-Reference

## 1.-Health

### Check server health

Checks the health endpoint without authentication.

```http
GET /dropbox-lite/health
```

## 2.-Account-and-Authentication

### 2.1 Request an OTP

Requests an OTP for the supplied email address.

**Authentication:** Not required.

```http
POST /dropbox-lite/auth/otp
Content-Type: application/json
```

Request body:

```json
{
  "email": "dummy.email@gmail.com"
}
```

Use the OTP provided by the application when creating an account or resetting a password.

### 2.2 Create an account

Creates an account using an email address, password, and OTP.

**Authentication:** Not required.

```http
POST /dropbox-lite/account/create
Content-Type: application/json
```

Request body:

```json
{
  "email": "dummy.email@gmail.com",
  "password": "YOUR_PASSWORD",
  "otp": "YOUR_OTP"
}
```

Replace `YOUR_OTP` with a valid OTP.

### 2.3 Log in

Authenticates the account and obtains a JWT token for protected endpoints.

**Authentication:** Not required.

```http
POST /dropbox-lite/account/login
Content-Type: application/json
```

Request body:

```json
{
  "email": "dummy.email@gmail.com",
  "password": "YOUR_PASSWORD"
}
```

Save the token returned in the response and use it in the `Authorization` header.

### 2.4 Reset password

Resets the account password using an OTP.

**Authentication:** Required by the provided collection.

```http
POST /dropbox-lite/account/reset
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

Request body:

```json
{
  "email": "dummy.email@gmail.com",
  "password": "YOUR_NEW_PASSWORD",
  "otp": "YOUR_OTP"
}
```

### 2.5 Refresh token

Refreshes the current authentication token.

**Authentication:** Required.

```http
POST /dropbox-lite/auth/refresh
Authorization: Bearer YOUR_JWT_TOKEN
```

No request body is required in the provided collection.

### 2.6 Delete an account

Deletes the account associated with the supplied details.

**Authentication:** Required by the provided collection.

```http
DELETE /dropbox-lite/account?email=dummy.email@gmail.com&password=YOUR_PASSWORD&otp=YOUR_OTP
Authorization: Bearer YOUR_JWT_TOKEN
```

No request body is required.

---

## 3.-File-Operations

All file operations below require authentication.

### 3.1 Initiate a file upload

Starts the upload workflow and submits the file's metadata.
`folderId` can be null if the you want a file to exist without a folder

```http
POST /dropbox-lite/files/upload
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

Request body:

```json
{
  "folderId": null,
  "name": "document.pdf", // file name
  "contentType": "application/pdf", //mime content type
  "size": 1024
}
```

### 3.2 Complete a file upload

Uploads the file content and completes the upload workflow.

```http
POST /dropbox-lite/files/upload/UPLOAD_ID/complete
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: multipart/form-data
```

Replace `UPLOAD_ID` with the `fileId` identifier returned by the `/files/upload` request.
### 3.3 List files

Retrieves a paginated list of files.

```http
GET /dropbox-lite/files?page=0&size=20
Authorization: Bearer YOUR_JWT_TOKEN
```

To list files inside a particular folder:

```http
GET /dropbox-lite/files?page=0&size=20&parentFolderId=YOUR_FOLDER_ID
Authorization: Bearer YOUR_JWT_TOKEN
```

### 3.4 View a file

Requests the view representation of a file.

```http
GET /dropbox-lite/files/view/FILE_ID
Authorization: Bearer YOUR_JWT_TOKEN
```

Replace `FILE_ID` with the actual file ID.

The response will give you have `url` which can be opened in a browser

### 3.5 Rename a file

Changes the name of an existing file.

```http
PATCH /dropbox-lite/files/FILE_ID/rename
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

Request body:

```json
{
  "name": "renamed-document.pdf"
}
```

### 3.6 Copy a file

Copies a file into the specified destination folder.

```http
POST /dropbox-lite/files/FILE_ID/copy
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

Request body:

```json
{
  "toFolder": "YOUR_DESTINATION_FOLDER_ID"
}
```

Use the destination folder ID returned by the folder API. Follow the backend's rules for copying to the root folder or handling duplicate names.

### 3.7 Move a file

Moves a file to another folder.

```http
POST /dropbox-lite/files/FILE_ID/move
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

Request body:

```json
{
  "toFolder": "YOUR_DESTINATION_FOLDER_ID",
  "replace": false
}
```

- `toFolder`: Destination folder ID.
- `replace`: if true this will silently replace the file else throw error

### 3.8 Archive files

- Note : we planned to add folder ids too in future version

Archives one or more files using their IDs.

```http
POST /dropbox-lite/files/archive
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

Request body:

```json
{
  "fileIds": [
    "YOUR_FILE_ID"
  ]
}
```

### 3.9 Delete a file

Deletes a file by its ID.

```http
DELETE /dropbox-lite/files/FILE_ID
Authorization: Bearer YOUR_JWT_TOKEN
```

Replace `FILE_ID` with the file you intend to delete.

---

## 4.-Folder-Operations

All folder operations below require authentication.

### 4.1 Create a folder

Creates a folder under a parent folder.

```http
POST /dropbox-lite/folders/create
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

Request body:

```json
{
  "parentFolderId": "YOUR_PARENT_FOLDER_ID",
  "name": "Documents"
}
```

Use the parent folder ID where the new folder should be created.

For a root-level folder, use `null` for `parentFolderId` if supported by the backend.

### 4.2 Delete a folder

Deletes a folder by its ID.

```http
DELETE /dropbox-lite/folders/FOLDER_ID?recursive=false
Authorization: Bearer YOUR_JWT_TOKEN
```

To request recursive deletion:

```http
DELETE /dropbox-lite/folders/FOLDER_ID?recursive=true
Authorization: Bearer YOUR_JWT_TOKEN
```

- `recursive`: if true delete all the folder contents else throw error if folder have any contents in them

---

## 5.-File-Sharing

All sharing endpoints below require authentication.

### 5.1 Share a file

Creates a share for another account.

```http
POST /dropbox-lite/sharing
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

Request body:

```json
{
  "recipientEmail": "recipient@example.com",
  "objectId": "YOUR_FILE_ID", // fileid 
  "shareDurationInMin": 30
}
```

Fields:

- `objectId`: for now we only support file sharing
- `shareDurationInMin`: Requested sharing duration in minutes (min 30).

### 5.2 List files shared with you

Retrieves the list of files shared with the recipient account. These can be viewed at `.share` folder

```http
GET /dropbox-lite/sharing/.shared?page=0&size=20
Authorization: Bearer YOUR_JWT_TOKEN
```

### 5.3 List files you shared

Retrieves the list of shares  shared by owner account. These can be viewed at `.shared` folder

```http
GET /dropbox-lite/sharing/.sharing?page=0&size=20
Authorization: Bearer YOUR_JWT_TOKEN
```

### 5.4 View a shared file

Requests a shared file using its object ID and the sharer's email address.

```http
GET /dropbox-lite/sharing/OBJECT_ID/view?sharedBy=SHARER_EMAIL
Authorization: Bearer YOUR_JWT_TOKEN
```

Replace `OBJECT_ID` with the shared object's ID and `SHARER_EMAIL` with the owner's email address.
Returns a url which can be opened in the browser

### 5.5 Remove a share

Removes a share associated with the specified object and recipient.

```http
DELETE /dropbox-lite/sharing/OBJECT_ID?sharedTo=RECIPIENT_EMAIL
Authorization: Bearer YOUR_JWT_TOKEN
```

Replace `OBJECT_ID` with the object ID and `RECIPIENT_EMAIL` with the email address of the recipient whose share should be removed.

---
## Using-the-APIs-Without-an-HTTP-Client

You can call the endpoints from any HTTP client or programming language that supports HTTP requests.

For example, using `curl`:

### Check health

```bash
curl -X GET \
  "http://localhost:8080/dropbox-lite/health"
```

### Request an OTP

```bash
curl -X POST \
  "http://localhost:8080/dropbox-lite/auth/otp" \
  -H "Content-Type: application/json" \
  -d '{"email":"dummy.email@gmail.com"}'
```

### Log in

```bash
curl -X POST \
  "http://localhost:8080/dropbox-lite/account/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"dummy.email@gmail.com","password":"YOUR_PASSWORD"}'
```

### List files

```bash
curl -X GET \
  "http://localhost:8080/dropbox-lite/files?page=0&size=20" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Upload a file using multipart form data

```bash
curl -X POST \
  "http://localhost:8080/dropbox-lite/files/upload/UPLOAD_ID/complete" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -F "file=@document.pdf"
```

Replace the example host, credentials, JWT, upload ID, and file path with your actual values.

For endpoints that require a JSON body, use `-H "Content-Type: application/json"` and `-d` with the appropriate JSON.

---

## Important-Notes

-  This is a v1 of Dropbox lite. This there is a errors in api call you can fork the project or fix them locally 