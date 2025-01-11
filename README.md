# Assignment-2.12 Shakeela

# Lambda Function and S3 Bucket Integration

## Questions and Answers

### 1. What is the purpose of the execution role on the Lambda function?
The execution role is an IAM role associated with the Lambda function that grants it the necessary permissions to interact with other AWS services and resources. For example, if the Lambda function needs to read data from an S3 bucket, write to another bucket, or log messages to Amazon CloudWatch Logs, the execution role determines whether these actions are permitted. Without this role, the function cannot perform operations that involve AWS resources because it lacks the required access.

The execution role typically includes permissions such as:
- Writing logs to CloudWatch.
- Accessing specific S3 buckets to read or write objects.
- Reading or writing to other AWS services like DynamoDB, SNS, or SQS, depending on the function's requirements.

By attaching policies to this role, you define the scope of what the Lambda function can and cannot do.

### 2. What is the purpose of the resource-based policy on the Lambda function?
A resource-based policy on the Lambda function specifies which external services, AWS accounts, or users are allowed to invoke the function. For instance, if an S3 bucket is configured to trigger the Lambda function upon a specific event, such as file creation, a resource-based policy on the Lambda function must grant the S3 service permission to invoke it.

This policy ensures that:
- Only authorized entities (e.g., specific S3 buckets, AWS accounts, or IAM users) can trigger the Lambda function.
- Security is enforced at the resource level, complementing IAM roles and permissions.

Without this policy, external triggers like S3 events cannot invoke the function, even if the execution role allows interactions with S3.

### 3. If the function is needed to upload a file into an S3 bucket, describe:

#### Needed Update on the Execution Role
To enable the Lambda function to upload files into an S3 bucket, the execution role must be updated with the following:

1. **Actions to allow:**
   - `s3:PutObject`: This action permits the Lambda function to upload objects to the bucket.
   - (Optional) `s3:GetBucketLocation`: Useful if the function needs to validate the bucket's location or confirm its existence before uploading.

2. **Resource specification:**
   - The policy should specify the ARN of the target bucket where the Lambda function will upload files. For example:
     ```json
     "Resource": "arn:aws:s3:::example-bucket/*"
     ```
     This ensures the function can only perform actions on the specified bucket and its objects, enhancing security.

3. **Minimal permissions principle:**
   - To follow security best practices, avoid granting broad permissions like `s3:*`. Restrict the Lambda function to only the actions and buckets it requires access to.

Here is an example policy statement for the execution role:
```json
{
  "Effect": "Allow",
  "Action": [
    "s3:PutObject",
    "s3:GetBucketLocation"
  ],
  "Resource": "arn:aws:s3:::example-bucket/*"
}
```

#### New Resource-Based Policy
In this scenario, no additional resource-based policy is required on the Lambda function itself. The Lambda function is initiating the upload operation, and resource-based policies are only needed when external entities (such as S3 events or another AWS account) need to invoke the Lambda function.

However, if the S3 bucket has a bucket policy restricting who can upload files, the bucket policy must allow the Lambda function's execution role to perform the `s3:PutObject` action. Here is an example bucket policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/lambda-execution-role"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
