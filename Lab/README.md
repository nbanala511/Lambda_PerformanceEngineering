## Lab Overview
Let's start with the High Level Design.
<img width="908" height="269" alt="lab_overview_diagram" src="https://github.com/user-attachments/assets/8956b8d5-cd8f-4015-968b-09125473300a" />  
An Amazon API Gateway is a collection of resources and methods. For this tutorial, you create one resource (DynamoDBManager) and define one method (POST) on it. The method is backed by a Lambda function (LambdaFunctionOverHttps). That is, when you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

Create, update, and delete an item.
Read an item.
Scan an item.
Other operations (echo, ping), not related to DynamoDB, that you can use for testing.
The request payload you send in the POST request identifies the DynamoDB operation and provides necessary data. For example:

The following is a sample request payload for a DynamoDB create item operation:  
```json
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Bob"
        }
    }
}
```    

The following is a sample request payload for a DynamoDB read item operation:  
```json
{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "1"
        }
    }
}
```
## Setup  

### Create Custom Policy  

We need to create a custom policy for least privilege

1. Open policies page in the IAM console
2. Click "Create policy" on top right corner
3. In the policy editor, click JSON, and paste the following  
```json
{
    "Version": "2012-10-17",
    "Statement": [
    {
      "Sid": "",
      "Action": [
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:UpdateItem"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "",
      "Resource": "*",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Effect": "Allow"
    }
    ]
    }
```

![create-policy](https://github.com/user-attachments/assets/2f4ea890-a2be-4fc8-85f9-24574ad76ef4)  
4. Give name "lambda-custom-policy", and click "Create policy" on botom right  
### Create Lambda IAM Role  
Create the execution role that gives your function permission to access AWS resources.

To create an execution role

1. Open the roles page in the IAM console.
2. Choose Create role.
3. Create a role with the following properties.

    - Trusted entity type – AWS service, then select Lambda from Use case  
    - Permissions – In the Permissions policies page, in the search bar, type lambda-custom-policy. The newly created policy should show up. Select it, and click Next.
    - Role name – lambda-apigateway-role.  
    - Click "Create role"  

![create-lambda-role](https://github.com/user-attachments/assets/23fe8405-553c-456b-b1c8-d306ed3750d5)
