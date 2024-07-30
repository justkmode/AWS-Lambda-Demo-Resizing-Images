# AWS Lambda Demo: Resizing Images

This project demonstrates how to use AWS Lambda to resize images uploaded to an S3 bucket. When an image is uploaded to a specific folder in the bucket, a Lambda function is triggered to resize the image and store it in a different folder.

## Lambda Function Code

```python
import boto3
import os
from PIL import Image

s3_client = boto3.client('s3')

def resize_image(image_path, resized_path):
    with Image.open(image_path) as image:
        image.thumbnail((128, 128))
        image.save(resized_path)

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    download_path = f'/tmp/{key}'
    upload_path = f'/tmp/resized-{key}'

    s3_client.download_file(bucket, key, download_path)
    resize_image(download_path, upload_path)
    s3_client.upload_file(upload_path, f'{bucket}', f'output/resized-{key}')
