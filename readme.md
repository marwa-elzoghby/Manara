# Serverless Image Processing with S3 and Lambda

![Serverless Image Processing Architecture](./assets/serverless%20image%20processing.drawio%20(2).png)

## Overview
This project implements a fully serverless image processing pipeline where users upload images that are automatically processed (resized, optimized, watermarked) using AWS Lambda functions triggered by S3 bucket events.

## Architecture Flow

1. **Image Upload**: Users upload images to the source S3 bucket
2. **Lambda Trigger**: S3 bucket event automatically invokes the Lambda function
3. **Image Processing**: Lambda performs transformations (resizing, watermarking, etc.)
4. **Result Storage**: Processed images are saved to the destination S3 bucket
5. **Access**: Users can request/download processed images

## Key Components

### Core Services
- **Amazon S3**:
  - Source Bucket: Stores original uploaded images (`the-source-for-uploaded-images`)
  - Destination Bucket: Stores processed images (`destination-for-processed-images`)
- **AWS Lambda**: 
  - `lambda-function-to-process-the-image`: Processes images upon upload
  - Auto-scales based on upload frequency

### Event Flow
1. User action: `upload-images`
2. S3 event: `invoke-lambda-function`
3. Processing: `lambda-function-to-process-the-image`
4. Output: `upload-new-images` to destination bucket
5. User access: `request-processed-image`

## Setup Instructions

### Prerequisites
- AWS account with IAM permissions
- AWS CLI configured
- Python 3.8+ runtime for Lambda

### Deployment Steps

1. Create S3 buckets:
   ```bash
   aws s3 mb s3://source-for-uploaded-images
   aws s3 mb s3://destination-for-processed-images