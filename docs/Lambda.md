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

![Screenshot%202023-10-16%20065423.png](file:///C:/Users/abuba/Pictures/Screenshots/Screenshot%202023-10-16%20065423.png)



### 2. Upload a test image to your source bucket

To upload a test image to your source bucket (console)

1. Open the [Buckets](https://console.aws.amazon.com/s3/buckets) page of the Amazon S3 console.

2. Choose __Upload__.

3. Choose __Add file__ and use the file selector to choose the object you want to upload.

4. Choose __Open__, then choose __Upload__.

### 2. Create an IAM Role

2.1. In the AWS Management Console, go to the IAM service.

2.2. Create a new IAM Role for your Lambda function with the necessary permissions to access the S3 bucket.

### 3. Create a Lambda Function

3.1. In the AWS Management Console, navigate to the Lambda service.

3.2. Click "Create Function."

3.3. Configure your Lambda function using the following settings:
   - **Name:** Your function name
   - **Runtime:** Choose the runtime
   - **Execution role:** Select the IAM role created in step 2

3.4. Click "Create Function."

### 4. Add Trigger

4.1. In the Lambda function configuration, go to the "Triggers" section.

4.2. Click "Add Trigger."

4.3. Select "S3" as the trigger source.

4.4. Configure the S3 trigger with the following settings:
   - **Bucket:** Select the S3 bucket you created in step 1.
   - **Event Type:** Choose the event type (e.g., ObjectCreated).

4.5. Save the trigger.

### 5. Write Lambda Code

5.1. In the Lambda function configuration, scroll down to the "Function code" section.

5.2. Write your Lambda function code that will execute when a change occurs in the S3 bucket. You can use the AWS SDK to interact with S3.

### 6. Test the Function

6.1. Configure a test event and run a test to ensure your Lambda function is working correctly.

### 7. Deploy the Lambda Function

7.1. Once your Lambda function is working as expected, click "Deploy."

### 8. Monitor and Troubleshoot

8.1. Use CloudWatch Logs and other monitoring tools to track the performance and troubleshoot any issues with your Lambda function.

## Conclusion

Congratulations! You've successfully created a Lambda function that responds to changes in an S3 bucket. You can now use this knowledge to build powerful serverless applications with AWS Lambda and S3.

For more details and advanced configurations, refer to the official [AWS Lambda documentation](https://docs.aws.amazon.com/lambda/latest/dg/with-s3-tutorial.html).

