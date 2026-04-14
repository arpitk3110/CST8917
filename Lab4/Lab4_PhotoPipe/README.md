# PhotoPipe — Event-Driven Image Processing (Lab 4)

## Overview

This Lab demonstrates an event-driven image processing pipeline built using Azure services. The system automatically processes images when they are uploaded to Azure Blob Storage using Event Grid and Azure Functions.

When a user uploads an image, an event is triggered which invokes serverless functions to process the image and log the activity.

---

### Demo Video Link: https://youtu.be/nL6Iqwmwduk

## Architecture Summary

The system consists of the following workflow:

1. User uploads an image to Blob Storage (`image-uploads`)
2. Event Grid detects a `BlobCreated` event
3. Two subscriptions handle the event:
   - **process-image function** → processes image and stores metadata
   - **audit-log function** → logs event details in Table Storage
4. Processed results are stored in `image-results`
5. Audit logs are stored in Table Storage 

---

## Services Used

- Azure Blob Storage
- Azure Event Grid (System Topic + Subscriptions)
- Azure Functions (Python)
- Azure Table Storage
- Simple Web Client (HTML)

---

## Setup Instructions

### 1. Create Resource Group
- Go to Azure Portal
- Create a resource group (e.g., `rg-serverless-lab4`)
- Select a region (e.g., Canada Central)

---

### 2. Create Storage Account

- Create a new storage account
- Enable **Blob anonymous access**
- Create the following containers:
  - `image-uploads` (public access: Blob)
  - `image-results` (private)

---

### 3. Configure CORS

- Go to **Resource sharing (CORS)** under Blob service
- Add the following settings:
  - Allowed Origins: `*`
  - Allowed Methods: `GET, PUT, OPTIONS, HEAD`
  - Allowed Headers: `*`
  - Exposed Headers: `*`

---

### 4. Create and Deploy Azure Functions

- Create a new Azure Functions project (Python)
- Add the provided `function_app.py`
- Install dependencies from `requirements.txt`
- Deploy the function app to Azure

Functions included:
- `process-image`
- `audit-log`
- `get-results`
- `get-audit-log`
- `health`

---

### 5. Configure Application Settings

In the Azure Function App:

1. Go to **Environment variables** under Settings  
2. Click **+ Add** under App settings  
3. Add the following key-value pair:

Name: STORAGE_CONNECTION_STRING
Value: <your-storage-account-connection-string>


4. Click **Apply** and then **Confirm**

> Note: Make sure to use the connection string of the main storage account (used for blob containers), not the function app’s default storage account.


---

### 6. Create Event Grid System Topic

- Go to Event Grid System Topics
- Create a new topic for your storage account

---

### 7. Create Event Subscriptions

#### Subscription 1 (Image Processing)
- Event Type: Blob Created
- Endpoint: `process-image`
- Filters:
  - Subject Begins With: `/blobServices/default/containers/image-uploads`
  - Advanced Filter:
    - subject ends with `.jpg`
    - subject ends with `.png`

#### Subscription 2 (Audit Log)
- Event Type: Blob Created
- Endpoint: `audit-log`
- Filter:
  - Subject Begins With: `/blobServices/default/containers/image-uploads`

---

### 8. Configure Web Client

- Open `client.html`
- Enter:
  - Storage Account Name
  - SAS Token
  - Function App URL

---

## Testing the System

1. Upload a `.jpg` file  
   → Should trigger both processing and audit log

2. Upload a `.png` file  
   → Same behavior as above

3. Upload a `.txt` or other file  
   → Only audit-log function should trigger

---

## Expected Output

- Processed image metadata stored in `image-results`
- Audit entries stored in Table Storage (`processinglog`)
- Results and logs visible in the web client

---

## Demo Video

A short demo video is included showing:
- Storage account configuration
- Event Grid subscriptions
- Image upload and processing
- Audit log entries

---

## Notes

- Event Grid filtering ensures only image files are processed
- Audit logging captures all upload events
- CORS is required for browser-based uploads
- Sensitive data (keys, tokens) is not included in this repository

---

