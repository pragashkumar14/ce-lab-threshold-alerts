# Lab: Create and Configure S3 Buckets

## Overview

In this lab, you'll create S3 buckets, configure storage classes, upload objects, and practice AWS CLI commands for S3. You'll learn fundamental S3 operations that you'll use throughout your cloud engineering career.

**Estimated Time:** 45-60 minutes

## Learning Objectives

- Create S3 buckets with proper naming conventions
- Upload and manage objects via Console and CLI
- Configure bucket settings (versioning, encryption, tags)
- Choose appropriate storage classes
- Organize objects with folders (prefixes)
- Use AWS CLI for S3 operations

## Prerequisites

- AWS account with S3 access
- AWS CLI configured
- Understanding of S3 concepts from lesson

---

## Exercise 1: Create Your First S3 Bucket

### Via AWS Console

**1. Navigate to S3**
- AWS Console → S3

**2. Create Bucket**
- Click "Create bucket"
- Bucket name: `your-name-bootcamp-demo-2024` (must be globally unique!)
- Region: `us-east-1` (N. Virginia)
- Keep all other defaults (Block Public Access enabled)
- Click "Create bucket"

**3. Verify Creation**
- You should see your bucket in the list
- Click on bucket name to view details

### Deliverable:
- Screenshot: `screenshots/bucket-created.png`

---

## Exercise 2: Upload Objects

### Part A: Upload via Console

**1. Upload Files**
- Click on your bucket
- Click "Upload"
- Drag and drop or click "Add files"
- Upload at least 3 different file types:
  - A text file (e.g., README.txt)
  - An image (e.g., logo.png)
  - A document (e.g., resume.pdf)
- Click "Upload"

**2. Create Folders**
- In your bucket, click "Create folder"
- Folder name: `images`
- Create folder
- Repeat to create folders: `documents`, `backups`

**3. Organize Files**
- Upload an image to `images/` folder
- Upload a document to `documents/` folder

### Part B: Upload via CLI

```bash
# Create a test file
echo "Hello from CLI!" > cli-test.txt

# Upload to bucket root
aws s3 cp cli-test.txt s3://your-name-bootcamp-demo-2024/

# Upload to a folder
aws s3 cp cli-test.txt s3://your-name-bootcamp-demo-2024/backups/

# List bucket contents
aws s3 ls s3://your-name-bootcamp-demo-2024/

# List folder contents
aws s3 ls s3://your-name-bootcamp-demo-2024/backups/
```

### Deliverables:
- Screenshot of uploaded files: `screenshots/files-uploaded.png`
- Screenshot of folder structure: `screenshots/folder-structure.png`
- Save CLI outputs to `cli-outputs.txt`

---

## Exercise 3: Configure Storage Classes

### Part A: Upload with Different Storage Class

**1. Upload with Standard-IA**
- Upload → Add files → Select a large file
- Before clicking "Upload", expand "Properties"
- Storage class: Choose "Standard-IA"
- Upload

**2. View Storage Class**
- Click on the object
- Check "Storage class" in properties

### Part B: Change Storage Class

**1. Select Existing Object**
- Check the box next to an object
- Actions → Edit storage class

**2. Change to Intelligent-Tiering**
- Select "Intelligent-Tiering"
- Save changes

### Deliverable:
- Screenshot showing different storage classes: `screenshots/storage-classes.png`

---

## Exercise 4: Enable Bucket Features

### Part A: Enable Versioning

**1. Go to Bucket Properties**
- Click on bucket → Properties tab

**2. Edit Versioning**
- Bucket Versioning → Edit
- Enable
- Save changes

**3. Test Versioning**
- Upload a file named `version-test.txt` with content "Version 1"
- Edit the file locally to say "Version 2"
- Upload again (same name)
- Click on object → Versions tab
- You should see 2 versions!

### Part B: Enable Encryption

**1. Default Encryption**
- Properties tab → Default encryption → Edit
- Encryption type: Server-side encryption with Amazon S3 managed keys (SSE-S3)
- Enable
- Save changes

### Part C: Add Tags

**1. Add Bucket Tags**
- Properties → Tags → Edit
- Add tags:
  - Key: `Environment`, Value: `Development`
  - Key: `Project`, Value: `Bootcamp`
- Save changes

### Deliverables:
- Screenshot of versioning enabled: `screenshots/versioning-enabled.png`
- Screenshot of versions list: `screenshots/versions.png`
- Screenshot of encryption enabled: `screenshots/encryption-enabled.png`
- Screenshot of tags: `screenshots/tags.png`

---

## Exercise 5: Download and Delete Operations

### Part A: Download Objects

**Via Console:**
- Select object → Click "Download"

**Via CLI:**
```bash
# Download single file
aws s3 cp s3://your-name-bootcamp-demo-2024/cli-test.txt ./downloads/

# Download entire folder
aws s3 cp s3://your-name-bootcamp-demo-2024/images/ ./downloads/images/ --recursive
```

### Part B: Delete Objects

**Via Console:**
- Select object → Delete
- Type "permanently delete" to confirm

**Via CLI:**
```bash
# Delete single object
aws s3 rm s3://your-name-bootcamp-demo-2024/cli-test.txt

# Delete all objects in folder
aws s3 rm s3://your-name-bootcamp-demo-2024/backups/ --recursive
```

### Deliverable:
- Save CLI outputs to `cli-outputs.txt`

---

## Exercise 6: Sync Operations

Practice syncing local directories with S3.

### Create Local Project Structure

```bash
mkdir -p local-project/{src,docs,assets}
echo "console.log('Hello');" > local-project/src/app.js
echo "# Documentation" > local-project/docs/README.md
echo "<!-- HTML -->" > local-project/assets/index.html
```

### Sync to S3

```bash
# Sync entire directory to S3
aws s3 sync local-project/ s3://your-name-bootcamp-demo-2024/project/

# List synced files
aws s3 ls s3://your-name-bootcamp-demo-2024/project/ --recursive
```

### Modify and Re-Sync

```bash
# Modify a file
echo "console.log('Updated');" > local-project/src/app.js

# Sync only changes
aws s3 sync local-project/ s3://your-name-bootcamp-demo-2024/project/
```

### Deliverable:
- Screenshot of synced files: `screenshots/synced-files.png`
- Save CLI outputs to `cli-outputs.txt`

---

## Exercise 7: View Bucket Metrics

### Check Bucket Size

**Via Console:**
- Bucket → Metrics tab
- View storage metrics

**Via CLI:**
```bash
# Get bucket size (requires CloudWatch)
aws cloudwatch get-metric-statistics \
  --namespace AWS/S3 \
  --metric-name BucketSizeBytes \
  --dimensions Name=BucketName,Value=your-name-bootcamp-demo-2024 \
  Name=StorageType,Value=StandardStorage \
  --start-time $(date -u -d '7 days ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 86400 \
  --statistics Average
```

### Deliverable:
- Screenshot of metrics: `screenshots/bucket-metrics.png`

---

## Bonus Challenges

### Challenge 1: Lifecycle Policy

Create a lifecycle policy to:
- Transition objects to Standard-IA after 30 days
- Transition to Glacier after 90 days
- Delete after 365 days

**Screenshot:** `screenshots/lifecycle-policy.png`

---

### Challenge 2: Cross-Region Replication

Set up replication to another region (requires versioning):
- Create bucket in different region
- Configure replication rule
- Test by uploading object

---

### Challenge 3: S3 Inventory

Configure S3 Inventory to generate reports of bucket contents.

---

## Submission

### Required Deliverables:

1. **Screenshots folder** with all required screenshots
2. **SOLUTION.md** completed with:
   - Bucket name and region
   - Number and types of objects uploaded
   - Storage classes used
   - CLI commands and outputs
3. **cli-outputs.txt** with all CLI outputs

### How to Submit:

```bash
git add .
git commit -m "Complete S3 bucket lab"
git push origin main
```

Create Pull Request and submit URL.

---

## Success Criteria

- ✅ S3 bucket created with proper naming
- ✅ Multiple objects uploaded via console and CLI
- ✅ Folders (prefixes) created and organized
- ✅ Different storage classes configured
- ✅ Versioning enabled and tested
- ✅ Encryption enabled
- ✅ Tags added
- ✅ Sync operations completed
- ✅ All screenshots captured
- ✅ CLI outputs documented

---

## Cleanup (Important!)

To avoid charges:

```bash
# Delete all objects
aws s3 rm s3://your-name-bootcamp-demo-2024/ --recursive

# Delete bucket
aws s3 rb s3://your-name-bootcamp-demo-2024/
```

Or via console: Select bucket → Empty → Delete

---

## Troubleshooting

**Bucket name already exists:**
- Bucket names are globally unique
- Add your name or numbers to make unique

**Access Denied:**
- Check IAM permissions
- Ensure you have S3 access

**CLI not working:**
- Run: `aws configure`
- Verify credentials

---

**Happy bucketing! 🪣**
