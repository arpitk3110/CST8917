# Lab 2: Smart Image Analyzer with Durable Functions
CST8917 – Serverless Applications

## Overview
This lab implements a Smart Image Analyzer using Azure Durable Functions with the Fan-Out/Fan-In pattern.

When an image is uploaded to Azure Blob Storage, the function:
1. Runs 4 analysis activities in parallel
2. Generates a combined report
3. Stores results in Azure Table Storage
4. Provides an HTTP endpoint to retrieve results

---

## Prerequisites

- Python 3.11 or 3.12
- Azure Functions Core Tools
- VS Code with:
  - Azure Functions extension
  - Azure Storage extension
  - Azurite extension
  - REST Client extension (optional)
- Azure Storage Explorer 

---

## Running Locally

### Step 1 – Create Virtual Environment

```
python -m venv .venv

Activate the virtual environment:

### Windows

.venv\Scripts\activate


### Install dependencies: 

pip install -r requirements.txt

```
### Step 2 – Start Azurite (Local Storage Emulator)

In VS Code:

 - Press F1

 - Run: Azurite: Start

Ensure Blob and Table services are running.

### Step 3 – Create Blob Container

In VS Code Azure panel:

Workspace
  → Attached Storage Accounts
    → Local Emulator
      → Blob Containers

#### Create a new container named

  - images

### Step 4 – Start Function App 

Run: 

 ``` 
 func start
 ```

OR press F5 in VS Code.

You should see all functions loaded, including:

```
blob_trigger
image_analyzer_orchestrator
get_results
```

### Step 5 – Upload Test Image (Local) 

Upload an image file (JPG/PNG) to the images container.

This automatically triggers the Durable Function orchestration.

### Step 6 – Retrieve Results (Local) 

Open browser:

```
http://localhost:7071/api/results
```

To retrieve a specific result:

```
http://localhost:7071/api/results/{id}
```

## Deploy to Azure 

### Step 1 – Create Azure Resources 

 - Create a Function App 

 - Create a Storage Account

### Step 2 – Configure Application Settings

In Function App → Environment Variables → App Settings:

Add:

```
 ImageStorageConnection
```

Set its value to your Storage Account connection string.

### Step 3 – Create Cloud Blob Container

In Azure Portal:

```
Storage Account → Containers → + Container
```

Create container named:

images


### Step 4 – Deploy

From VS Code:

 - Press F1

 - Run: Azure Functions: Deploy to Function App

 - Select your function app

### Step 5 – Test in Azure

Upload an image to the Azure images container.

Access:

```
https://<your-function-name>.azurewebsites.net/api/results
```

### Security Note 

Do NOT commit: 

 - local.settings.json


#### Use: 

local.settings.example.json -- with placeholder values instead. 

