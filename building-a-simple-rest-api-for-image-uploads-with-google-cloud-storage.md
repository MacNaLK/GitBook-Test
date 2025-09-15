# 🚀 Building a Simple REST API for Image Uploads with Google Cloud Storage

When you’re experimenting with Google Cloud, a common proof-of-concept (POC) is to integrate with Google Cloud Storage (GCS). In this post, we’ll walk through creating a simple CRUD-style REST API that lets you upload, list, download, and delete images from GCS.

We’ll use FastAPI (Python) for the backend, and keep the setup lean so you can run it locally or deploy to Cloud Run later.

***

### 🔧 What We’ll Build

Our API will have four endpoints:

* POST /upload/ → Upload an image to GCS
* GET /list/ → List images stored in GCS
* GET /download/{filename} → Get a public URL to download an image
* DELETE /delete/{filename} → Remove an image from GCS

***

### 🏗 Step 1: Create a GCS Bucket

First, create a bucket in your project:

```
gcloud config set project <PROJECT_ID>
gsutil mb -c standard -l us-central1 gs://my-poc-images-bucket/
```

***

### 🔑 Step 2: Configure Service Account

We’ll need a service account with permissions to manage objects in the bucket.

```hcl
# Create service account
gcloud iam service-accounts create gcs-poc-api \
  --description="POC API for GCS" \
  --display-name="GCS POC API"

# Grant object admin rights
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:gcs-poc-api@<PROJECT_ID>.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"

# Generate a key
gcloud iam service-accounts keys create key.json \
  --iam-account=gcs-poc-api@<PROJECT_ID>.iam.gserviceaccount.com
```

Place the key.json in your project folder. (For production, avoid keys and use Workload Identity instead.)

***

### ⚡ Step 3: Write the FastAPI Code

Install dependencies:

```bash
pip install fastapi uvicorn google-cloud-storage python-multipart
```

Create a file main.py:

```python
from fastapi import FastAPI, File, UploadFile, HTTPException
from google.cloud import storage
import os

app = FastAPI()

# Load credentials
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "key.json"

BUCKET_NAME = "my-poc-images-bucket"
client = storage.Client()
bucket = client.bucket(BUCKET_NAME)

@app.post("/upload/")
async def upload_image(file: UploadFile = File(...)):
    blob = bucket.blob(file.filename)
    blob.upload_from_file(file.file, content_type=file.content_type)
    return {"message": "Uploaded", "filename": file.filename}

@app.get("/list/")
async def list_images():
    blobs = bucket.list_blobs()
    return {"files": [b.name for b in blobs]}

@app.get("/download/{filename}")
async def download_image(filename: str):
    blob = bucket.blob(filename)
    if not blob.exists():
        raise HTTPException(status_code=404, detail="File not found")
    return {"download_url": blob.public_url}

@app.delete("/delete/{filename}")
async def delete_image(filename: str):
    blob = bucket.blob(filename)
    if not blob.exists():
        raise HTTPException(status_code=404, detail="File not found")
    blob.delete()
    return {"message": "Deleted", "filename": filename}
```

***

### ▶️ Step 4: Run Locally

```bash
uvicorn main:app --reload
```

Now test with Postman or curl:

```bash
# Upload
curl -X POST "http://127.0.0.1:8000/upload/" -F "file=@test.jpg"

# List files
curl "http://127.0.0.1:8000/list/"

# Download
curl "http://127.0.0.1:8000/download/test.jpg"

# Delete
curl -X DELETE "http://127.0.0.1:8000/delete/test.jpg"
```

***

### 🚀 Step 5: Deploy to Cloud Run (Optional)

To make this accessible publicly without managing servers:

```hcl
gcloud run deploy gcs-poc-api \
  --source . \
  --region us-central1 \
  --allow-unauthenticated
```

Cloud Run will give you a URL like:

```hcl
https://gcs-poc-api-xyz.a.run.app
```

You can now call the same endpoints live in the cloud.

***

### 🔮 Next Steps

* Use Signed URLs instead of public\_url for secure downloads.
* Add authentication to the API (Firebase Auth, Identity-Aware Proxy, or OAuth).
* Connect a simple React or Vue frontend to make an image manager.
* Replace key.json with Workload Identity Federation for better security.

***

### ✅ Conclusion

In just a few steps, we created a functional CRUD API that interacts with Google Cloud Storage. This setup is great for POCs, hackathons, or learning projects — and with a few tweaks (auth, signed URLs, Cloud Run deployment), it can scale into a production-ready service.
