# Building a Serverless Architecture with Amazon SNS, SQS, and Lambda

## Overview
This project demonstrates the implementation of a serverless architecture using Amazon Simple Notification Service (SNS), Amazon Simple Queue Service (SQS), and AWS Lambda. The architecture is designed to efficiently handle events triggered by file uploads to an Amazon S3 bucket, allowing for automated processing and distribution of notifications.

## Task List

### Task 1: Create a standard Amazon SNS topic
#### Introduction:
Creating an Amazon SNS topic establishes a centralized point for distributing event notifications. These notifications serve as triggers for subsequent actions in the serverless architecture, ensuring timely and coordinated responses to file uploads.

#### Actions:
1. **Accessing the Amazon SNS Console:** I navigated to the Amazon SNS Console in the AWS Management Console to begin creating a new SNS topic.
   
2. **Creating the SNS Topic:** Within the console, I initiated the creation of a new SNS topic, specifying its type as "Standard" and providing a unique name, such as "resize-image-topic-xxxx," to facilitate identification.
   
3. **Saving Topic Details:** After successful creation, I noted down essential details such as the topic's Amazon Resource Name (ARN) and the AWS account ID of the topic owner for future reference.

- <img width="802" alt="Screenshot 2024-01-26 193120" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/01f3b342-3816-49d4-ae19-c9eb90b8c0a4">
- <img width="428" alt="Screenshot 2024-01-26 193230" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/6634e903-9114-4167-909b-6e373cbe7768">


### Task 2: Create three Amazon SQS queues

Amazon SQS queues serve as intermediate channels for distributing notifications received from the SNS topic. By creating multiple queues, each subscribed to the SNS topic, we enable parallel processing and distribution of tasks to different Lambda functions.

#### Actions:
1. **Accessing the Amazon SQS Console:** I accessed the Amazon SQS Console in the AWS Management Console to create the required queues.
   
2. **Creating the Thumbnail Queue (Task 2.1):** I created the "thumbnail-queue" to handle tasks related to generating thumbnail images. This queue was subsequently subscribed to the SNS topic created in Task 
   
3. **Creating Additional Queues (Task 2.3):** Following a similar process, I created two more SQS queues named "web-queue" and "mobile-queue" to handle tasks specific to generating web and mobile-sized images, respectively. These queues were also subscribed to the SNS topic.

   - <img width="770" alt="Screenshot 2024-01-26 193324" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/91c3654b-89f6-4026-b4c7-59e9624aaa6a">
   - <img width="772" alt="Screenshot 2024-01-26 193415" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/a21d59a6-8a81-4462-9c7f-a02444cdd347">
   - <img width="778" alt="Screenshot 2024-01-26 193504" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/b0d9756e-4709-490d-b3a2-311055b7506b">
   - <img width="827" alt="Screenshot 2024-01-26 193525" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/51290cc5-09d9-410f-a1bf-4d79902a99e0">
   - <img width="817" alt="Screenshot 2024-01-26 193604" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/24d3ccce-8829-481a-8e8f-8d75dda1c750">






### Task 3: Create an Amazon S3 event notification

Configuring an Amazon S3 event notification enables automatic triggering of the SNS topic whenever new objects are uploaded to the designated S3 bucket. This integration ensures seamless communication between S3 and the serverless architecture.

#### Actions:
1. **Accessing the Amazon S3 Console:** I navigated to the Amazon S3 Console in the AWS Management Console to configure event notifications for the designated bucket.

   <img width="804" alt="Screenshot 2024-01-26 194048" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/03d1e7ec-7c15-42e3-938b-a8ecfa3c3215">
Using this code: 
```
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__default_statement_ID",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": [
        "SNS:GetTopicAttributes",
        "SNS:SetTopicAttributes",
        "SNS:AddPermission",
        "SNS:RemovePermission",
        "SNS:DeleteTopic",
        "SNS:Subscribe",
        "SNS:ListSubscriptionsByTopic",
        "SNS:Publish",
        "SNS:Receive"
      ],
      "Resource": "arn:aws:sns:us-west-2:467100228310:resize-image-topic3428",
      "Condition": {
        "StringEquals": {
          "AWS:SourceAccount": "467100228310"
        }
      }
    },
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:us-west-2:467100228310:resize-image-topic3428",
      "Condition": {
        "StringEquals": {
          "AWS:SourceAccount": "467100228310"
        }
      }
    }
  ]
}
```
   
3. **Configuring Event Notifications (Task 3.1):** Within the bucket properties, I configured an event notification to trigger the designated SNS topic whenever new objects were uploaded to the "ingest" folder with a ".jpg" extension.

   - <img width="636" alt="Screenshot 2024-01-26 194556" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/3907dd8c-e43b-4a2d-9535-7f857b4d5830">
   - <img width="599" alt="Screenshot 2024-01-26 194711" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/94ff58dd-ceb6-418f-a32b-a2f73be0b400">



### Task 4: Create and configure three AWS Lambda functions

AWS Lambda functions are the core components responsible for processing incoming messages from the SQS queues. By creating and configuring these functions, we define the logic for image processing and ensure seamless integration with other AWS services.

#### Actions:
1. **Accessing the AWS Lambda Console:** I accessed the AWS Lambda Console in the AWS Management Console to create and configure the required Lambda functions.
   
2. **Creating Lambda Functions (Task 4.1):** I initiated the creation of the "CreateThumbnail" Lambda function, specifying its runtime environment, execution role, and basic configuration details.
   
3. **Configuring Triggers and Deployment (Task 4.2):** I configured the Lambda function to trigger on messages received from the "thumbnail-queue" SQS queue. Additionally, I uploaded the Python deployment package containing the function code and dependencies.
   
4. **Repeating the Process (Task 4.3):** Following a similar process, I created two additional Lambda functions named "CreateWebImage" and "CreateMobileImage" to handle image processing tasks for web and mobile-sized images, respectively.

   - <img width="815" alt="Screenshot 2024-01-26 194929" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/e802596d-3ce5-4242-b75b-dbfaa1de0831">
   - <img width="612" alt="Screenshot 2024-01-26 195027" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/e91bbb91-7403-465d-8aa7-7ee6dc98d9bd">
   - <img width="627" alt="Screenshot 2024-01-26 195154" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/c17a94ef-6385-42ce-b8c8-5c2688bf3e94">
   

   



### Task 5: Upload an object to the Amazon S3 bucket

Uploading an object to the designated S3 bucket initiates the entire serverless architecture, triggering event notifications and subsequent processing by the Lambda functions. This step validates the functionality and responsiveness of the implemented architecture.

#### Actions:
1. **Accessing the Amazon S3 Console:** I accessed the Amazon S3 Console in the AWS Management Console to upload an image file for testing purposes.
   
2. **Uploading the Image File:** I uploaded an image file, such as "InputFile.jpg," to the designated "ingest" folder within the S3 bucket.

   - <img width="638" alt="Screenshot 2024-01-26 200210" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/11cbadb4-eed1-4021-b133-fc08d26ac5a7">
   - <img width="1022" alt="Screenshot 2024-01-26 200742" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/faa72f52-c05b-4012-a835-14c8e102c576">



### Task 6: Validate the processed file

Validating the processed file involves verifying that the uploaded image triggered all relevant Lambda functions and that the processed images are correctly stored in the designated folders within the S3 bucket. This step ensures the successful execution and completion of the serverless architecture workflow.

#### Actions:
1. **Monitoring Lambda Activity (Task 6.1):** I monitored Lambda function invocations and inspected CloudWatch logs to identify any errors or debugging information related to the processing of uploaded images.

   - <img width="1006" alt="Screenshot 2024-01-26 200916" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/dc499cc2-ec3b-4e18-9070-3a0d5e393101">
   - <img width="859" alt="Screenshot 2024-01-26 201049" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/30a7fd88-f6b9-49ec-b25a-40dc861a4f65">
   

   
3. **Validating S3 Bucket Contents (Task 6.2):** I navigated to the S3 bucket to verify the presence of resized images in the designated folders, confirming successful image processing by the Lambda functions.
   - <img width="298" alt="Screenshot 2024-01-26 201412" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/833f1387-b73e-421e-b2be-d7fa91692449">
   - <img width="347" alt="Screenshot 2024-01-26 201418" src="https://github.com/aasif287/AWS-Serverless-Architecture/assets/155476415/0bbe376d-6ad3-4b4f-a57d-c087922299e1">



