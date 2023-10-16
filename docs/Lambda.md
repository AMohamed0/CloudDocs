# AWS Lambda Tutorial: Working with Amazon S3

This step-by-step tutorial will guide you through the process of creating and configure a Lambda function that resizes images added to an Amazon Simple Storage Service (Amazon S3) bucket. When you add an image file to your bucket, Amazon S3 invokes your Lambda function. The function then creates a thumbnail version of the image and outputs it to a different Amazon S3 bucket.

## Prerequisites

Before you get started, ensure that you have the following prerequisites in place:

- [AWS account](https://aws.amazon.com/)
- [AWS CLI](https://aws.amazon.com/cli/)
- Basic knowledge of AWS Lambda and Amazon S3

## Steps

### 1. Create two Amazon S3 buckets

To create the Amazon S3 buckets (console)

1. Open the [Buckets](https://console.aws.amazon.com/s3/buckets) page of the Amazon S3 console.

2. Choose __Create bucket__.

3. Under General configuration, do the following:
    - For __Bucket name__, enter a globally unique name that meets the Amazon S3 Bucket naming rules. Bucket names can contain only lower case letters, numbers, dots (.), and hyphens (-).

    - For __AWS Region__, choose the AWS Region closest to your geographical location. Later in the tutorial, you must create your Lambda function in the same AWS Region, so make a note of the region you chose.

4. Leave all other options set to their default values and choose __Create bucket__.

5. Repeat steps 1 to 4 to create your destination bucket. For __Bucket name__, enter `SOURCEBUCKET-resized`, where `SOURCEBUCKET` is the name of the source bucket you just created.

![Screenshot 2023-10-16 065423.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20065423.png)

### 2. Upload a test image to your source bucket

To upload a test image to your source bucket (console)

1. Open the [Buckets](https://console.aws.amazon.com/s3/buckets) page of the Amazon S3 console.

2. Choose __Upload__.

3. Choose __Add file__ and use the file selector to choose the object you want to upload.

4. Choose __Open__, then choose __Upload__.

![Screenshot 2023-10-16 065649.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20065649.png)

### 3. Create a permissions policy

To create the policy (console)

1. Open the [Policies](https://console.aws.amazon.com/iamv2/home#policies) page of the AWS Identity and Access Management (IAM) console.

2. Choose __Create policy__.

3. Choose the __JSON tab__, and then paste the following custom policy into the JSON editor.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:PutLogEvents",
                "logs:CreateLogGroup",
                "logs:CreateLogStream"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::*/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::*/*"
        }
    ]
}
```

1. Choose __Next: Tags__.

2. Choose __Next: Review__.

3. Under __Review policy__, for __Name__, enter `LambdaS3Policy`.

4. Choose __Create policy__.

![Screenshot 2023-10-16 070045.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20070045.png)

### 3. Create an execution role

To create an execution role and attach your permissions policy (console)

1. Open the [Roles](https://console.aws.amazon.com/iamv2/home#roles) page of the (IAM) console.

2. Choose __Create role__.

3. For __Trusted entity type__, select __AWS service__, and for __Use case__, select __Lambda__.

4. Choose __Next__.

5. Add the permissions policy you created in the previous step by doing the following:

    - In the policy search box, enter `LambdaS3Policy`.

    - In the search results, select the check box for `LambdaS3Policy`.

    - Choose __Next__.

6. Under __Role details__, for the __Role name__ enter `LambdaS3Role`.

7. Choose __Create role__.

![Screenshot 2023-10-16 070259.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20070259.png)

### 4. Create the function deployment package

1. Save the example code as a file named `lambda_function.py`.

```python
import boto3
import os
import sys
import uuid
from urllib.parse import unquote_plus
from PIL import Image
import PIL.Image
            
s3_client = boto3.client('s3')
            
def resize_image(image_path, resized_path):
  with Image.open(image_path) as image:
    image.thumbnail(tuple(x / 2 for x in image.size))
    image.save(resized_path)
            
def lambda_handler(event, context):
  for record in event['Records']:
    bucket = record['s3']['bucket']['name']
    key = unquote_plus(record['s3']['object']['key'])
    tmpkey = key.replace('/', '')
    download_path = '/tmp/{}{}'.format(uuid.uuid4(), tmpkey)
    upload_path = '/tmp/resized-{}'.format(tmpkey)
    s3_client.download_file(bucket, key, download_path)
    resize_image(download_path, upload_path)
    s3_client.upload_file(upload_path, '{}-resized'.format(bucket), 'resized-{}'.format(key)) 
```

1. In the same directory in which you created your `lambda_function.py file`, create a new directory named `package` and install the [Pillow (PIL)](https://pypi.org/project/Pillow/) library and the AWS SDK for Python (Boto3). Although the Lambda Python runtime includes a version of the Boto3 SDK, we recommend that you add all of your function's dependencies to your deployment package, even if they are included in the runtime. For more information, see [Runtime dependencies in Python](https://docs.aws.amazon.com/lambda/latest/dg/python-package.html#python-package-dependencies).

```bash
mkdir package
pip install \
--platform manylinux2014_x86_64 \
--target=package \
--implementation cp \
--python-version 3.9 \
--only-binary=:all: --upgrade \
pillow boto3 
```

![Screenshot 2023-10-16 070915.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20070915.png)

1. Create a .zip file containing your application code and the Pillow and Boto3 libraries. In Linux or MacOS, run the following commands from your command line interface.

```bash
cd package
zip -r ../lambda_function.zip .
cd ..
zip lambda_function.zip lambda_function.py
```

In Windows, use your preferred zip tool to create the lambda_function.zip file. Make sure that your lambda_function.py file and the folders containing your dependencies are all at the root of the .zip file.

![zipf.gif](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/zipf.gif)

### 5. Create the Lambda function

To create the function (console)

To create your Lambda function using the console, you first create a basic function containing some ‘Hello world’ code. You then replace this code with your own function code by uploading the.zip or JAR file you created in the previous step.

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) of the Lambda console.

2. Make sure you're working in the same AWS Region you created your Amazon S3 bucket in. You can change your region using the drop-down list at the top of the screen.

3. Choose __Create function__.

4. Choose __Author from scratch__.

5. Under __Basic information__, do the following:

    - For __Function name__, enter `CreateThumbnail`.

    - For __Runtime__ choose  __Python 3.9__.

    - For __Architecture,__ choose __x86_64__.

6. In the __Change default execution role__ tab, do the following:

    - Expand the tab, then choose __Use an existing role__.

    - Select the LambdaS3Role you created earlier.

7. Choose Create function.

![Screenshot 2023-10-16 071227.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20071227.png)

To upload the function code (console)

1. In the Code source pane, choose Upload from.

2. For the Python and Node.js runtimes, choose .zip file. For the Java runtime, choose .zip or .jar file.

3. Choose Upload.

4. In the file selector, select your .zip or JAR file and choose Open.

5. Choose Save.

6. next add a layer from this link (pick the correct csv for your Region): [https://github.com/keithrozario/Klayers/tree/master/deployments/python3.9](https://github.com/keithrozario/Klayers/tree/master/deployments/python3.9)
   - select the corresponding arn for the pillow module:
  ![Screenshot 2023-10-16 094218.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20094218.png)
  ![Screenshot 2023-10-16 073211.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20073211.png)

### 6. Configure Amazon S3 to invoke the function

To configure the Amazon S3 trigger (console)

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) of the Lambda console and choose your function (`CreateThumbnail`).

2. Choose __Add trigger__.

3. Select __S3__.

4. Under __Bucket__, select your source bucket.

5. Under __Event types__, select __All object create events__.

6. Under __Recursive invocation__, select the check box to acknowledge that using the same Amazon S3 bucket for input and output is not recommended. You can learn more about recursive invocation patterns in Lambda by reading Recursive patterns that cause run-away Lambda functions in Serverless Land.

7. Choose __Add__.

![Screenshot 2023-10-16 071614.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20071614.png)

### 7. Test your Lambda function with a dummy event

To test your Lambda function with a dummy event (console)

1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) of the Lambda console and choose your function (`CreateThumbnail`).

2. Choose the Test tab.

3. To create your test event, in the __Test event__ pane, do the following:

    - Under __Test event action__, select __Create new event__.

    - For __Event name__, enter `myTestEvent`.

    - For __Template__, select __S3 Put__.

    - Replace the values for the following parameters with your own values.

        - For `awsRegion`, replace `us-east-1` with the AWS Region you created your Amazon S3 buckets in.

        - For `name`, replace `example-bucket` with the name of your own Amazon S3 source bucket.

        - For `key`, replace `test%2Fkey` with the filename of the test object you uploaded to your source bucket in the step Upload a test image to your source bucket.

```json
{
  "Records": [
    {
      "eventVersion": "2.0",
      "eventSource": "aws:s3",
      "awsRegion": "us-east-1",
      "eventTime": "1970-01-01T00:00:00.000Z",
      "eventName": "ObjectCreated:Put",
      "userIdentity": {
        "principalId": "EXAMPLE"
      },
      "requestParameters": {
        "sourceIPAddress": "127.0.0.1"
      },
      "responseElements": {
        "x-amz-request-id": "EXAMPLE123456789",
        "x-amz-id-2": "EXAMPLE123/5678abcdefghijklambdaisawesome/mnopqrstuvwxyzABCDEFGH"
      },
      "s3": {
        "s3SchemaVersion": "1.0",
        "configurationId": "testConfigRule",
        "bucket": {
          "name": "example-bucket",
          "ownerIdentity": {
            "principalId": "EXAMPLE"
          },
          "arn": "arn:aws:s3:::example-bucket"
        },
        "object": {
          "key": "test%2Fkey",
          "size": 1024,
          "eTag": "0123456789abcdef0123456789abcdef",
          "sequencer": "0A1B2C3D4E5F678901"
        }
      }
    }
  ]
}
```

  Choose __Save__.

1. In the __Test event pane__, choose __Test__.

 ![Screenshot 2023-10-16 073246.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20073246.png)

- too ensure the function works. increase the timeout length to 15 seconds.
  
  ![Screenshot 2023-10-16 072000.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20072000.png)

1. To check the your function has created a resized verison of your image and stored it in your target Amazon S3 bucket, do the following:

    -Open the [Buckets page](https://console.aws.amazon.com/s3/buckets) of the Amazon S3 console.

    -Choose your target bucket and confirm that your resized file is listed in the __Objects__ pane.

      ![Screenshot 2023-10-16 073417.png](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/Screenshot%202023-10-16%20073417.png)

??? note "Clean up your resources"

    You can now delete the resources that you created for this tutorial, unless you want to retain them. By deleting AWS resources that you're no longer using, you prevent      unnecessary charges to your AWS account.

    To delete the Lambda function

    1. Open the [Functions page](https://console.aws.amazon.com/lambda/home#/functions) of the Lambda console.

    2. Select the function that you created.

    3. Choose __Actions__, __Delete__.

    4. Type `delete` in the text input field and choose __Delete__.

    To delete the policy that you created

    1. Open the [Policies page](https://console.aws.amazon.com/iam/home#/policies) of the IAM console.

    2. Select the policy that you created (__AWSLambdaS3Policy__).

    3. Choose __Policy actions__, __Delete__.

    4. Choose __Delete__.

    To delete the execution role

    1. Open the [Roles page](https://console.aws.amazon.com/iam/home#/roles) of the IAM console.

    2. Select the execution role that you created.

    3. Choose __Delete__.

    4. Enter the name of the role in the text input field and choose __Delete__.

    To delete the S3 bucket

    1. Open the [Amazon S3 console](https://console.aws.amazon.com/s3/home#).

    2. Select the bucket you created.

    3. Choose __Delete__.

    4. Enter the name of the bucket in the text input field.

    5. Choose __Delete bucket__.

## Conclusion

Congratulations on creating a resized image with Amazon S3 and Lambda!
