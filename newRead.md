
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
   ```

2. Create Lambda function with:
   - Python 3.8+ runtime
   - S3 trigger from source bucket
   - Permission to read/write to both buckets
   - Pillow layer added (see below)

3. Configure S3 event notification to trigger Lambda on PUT events

## Lambda Function Code (Python)

Save this as `lambda_function.py`:

```python
import boto3
import os
from PIL import Image
from io import BytesIO

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get the uploaded file details
    source_bucket = event['Records'][0]['s3']['bucket']['name']
    source_key = event['Records'][0]['s3']['object']['key']
    
    try:
        # Download the image
        file_obj = s3.get_object(Bucket=source_bucket, Key=source_key)
        file_content = file_obj['Body'].read()
        
        # Process with Pillow
        image = Image.open(BytesIO(file_content))
        
        # Resize and convert to webp
        image.thumbnail((800, 800))
        output_buffer = BytesIO()
        image.save(output_buffer, format='WEBP', quality=80)
        output_buffer.seek(0)
        
        # Upload processed image
        dest_key = f"processed/{os.path.splitext(os.path.basename(source_key))[0]}.webp"
        s3.put_object(
            Bucket="destination-for-processed-images",
            Key=dest_key,
            Body=output_buffer,
            ContentType='image/webp'
        )
        
        return {
            'statusCode': 200,
            'body': f"Successfully processed {source_key}"
        }
        
    except Exception as e:
        print(e)
        raise e
```

## Required Dependencies

Create a `requirements.txt` file:
```
Pillow==9.5.0
boto3==1.26.0
```

## Deployment Package

1. Create a layer for Pillow:
   ```bash
   mkdir -p python/lib/python3.8/site-packages
   pip install Pillow -t python/lib/python3.8/site-packages/
   zip -r pillow_layer.zip python
   aws lambda publish-layer-version --layer-name pillow --zip-file fileb://pillow_layer.zip
   ```

2. Upload your Lambda function:
   ```bash
   zip lambda_function.zip lambda_function.py
   aws lambda create-function \
       --function-name image-processor \
       --runtime python3.8 \
       --handler lambda_function.lambda_handler \
       --role your-lambda-execution-role-arn \
       --zip-file fileb://lambda_function.zip \
       --layers your-pillow-layer-arn
   ```

## Security Considerations
- Set S3 bucket policies to restrict access
- Use IAM roles with least privilege principle
- Enable S3 server-side encryption
- Rotate IAM credentials regularly

## Cost Optimization Tips
- Set appropriate Lambda memory (128MB-1GB typically sufficient)
- Implement S3 Lifecycle Policies for older images
- Monitor with AWS Cost Explorer

## Troubleshooting
- Check CloudWatch Logs for Lambda execution
- Verify S3 bucket permissions
- Test with small images first
- Ensure Lambda has enough memory/timeout for large images
```

This version includes:
1. Complete Python Lambda code using Pillow
2. Instructions for creating a Lambda layer for Pillow
3. Deployment commands for the Python version
4. All original architecture documentation updated for Python runtime

The code handles:
- Image download from S3
- Resizing to 800px max dimension
- Conversion to WEBP format
- Upload to destination bucket
- Proper error handling