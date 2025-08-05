## Lab Overview
Let's start with the High Level Design.
<img width="908" height="269" alt="lab_overview_diagram" src="../Images/lab_overview_diagram.PNG" />  
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
## Create Lambda Function  
### To create the function  
1. Click "Create function" in AWS Lambda Console  
2. Select "Author from scratch". Use name LambdaFunctionOverHttps , select Python 3.13 as Runtime. Under Permissions, click the arrow beside "Change default execution role", then "use an existing role" and select lambda-apigateway-role that we created, from the drop down  
![lambda-basic-info](https://github.com/user-attachments/assets/1174add5-389f-4f28-8e3f-2b8729e4b853)  
3. Click "Create function"  
4. Replace the boilerplate coding with the following code snippet and click "Deploy"  
### Example python Code  
```python
from __future__ import print_function
import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
```  

![lambda-code-paste](https://github.com/user-attachments/assets/0d8b283e-e339-4805-a792-0d6d4bded404)  

### Test Lambda Function  
Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.  

1. Click the "Test" tab right beside "Code" tab  
2. Give "Event name" as echotest  
3. Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Save" to save  
```json
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```  
4. Click "Test", and it will execute the test event. You should see the output in the console  

![execute-test](https://github.com/user-attachments/assets/f9f07db1-5316-4cb4-be35-f89429eecf51)  

We're all set to create DynamoDB table and an API using our lambda as backend!

## Create DynamoDB Table  
Create the DynamoDB table that the Lambda function uses.  
#### To create a DynamoDB table  
1. Open the DynamoDB console.
2. Choose "tables" from left pane, then click "Create table" on top right.
3. Create a table with the following settings.  
    - Table name – lambda-apigateway
    - Partition key – id (string)  
4. Choose "Create table".  
![create-dynamo-table](https://github.com/user-attachments/assets/2d1c2087-b83c-4d9d-9a81-2d36615cec08)  

## Create API  
### To create the API  
1. Go to API Gateway console
2. Click Create API
3. Scroll down and select "Build" for REST API
4. Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"
![create-new-api](https://github.com/user-attachments/assets/1b3913ad-4dd3-4e3b-b707-e9f50b040ac2)  
5. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time you only have the root resource, but let's add a resource next. Click "Create Resource"  
6. Input "DynamoDBManager" in the Resource Name. Click "Create Resource"  
![create-resource-name](https://github.com/user-attachments/assets/48ac068a-c5c5-408f-b1bf-8b781cce413d)  
7. Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, click "Create Method".  
![create-method-1](https://github.com/user-attachments/assets/bec67302-73e5-4147-9ef5-e7c7f9a2503f)  
8. Select "POST" from drop down.  
9. Integration type should be pre-selected as "Lambda function". Select "LambdaFunctionOverHttps" function that we created earlier. As you start typing the name, your function name will show up.Select the function, scroll down and click "Create method".  
![create-lambda-integration](https://github.com/user-attachments/assets/8180a6ac-c1b6-485d-9a86-ac8ed16bd787)  

Our API-Lambda integration is done!
### Deploythe API  
In this step, you deploy the API that you created to a stage called prod.  
    1. Click "Deploy API" on top right
    2. Now it is going to ask you about a stage. Select "[New Stage]" for "Stage". Give "Prod" as "Stage name". Click "Deploy"  
![deploy-api-2](https://github.com/user-attachments/assets/ec1ada8b-32dc-4a7b-b991-992df87b1c1b)  
    3. We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", keep expanding till you see "POST", select "POST" method, and copy the "Invoke URL" from screen  
![copy-invoke-url](https://github.com/user-attachments/assets/c540f16c-50d7-40f2-af02-b0723841cc91)  

### Running our solution  
1. The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:  
```json
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
```  
2. To execute our API from local machine, we are going to use Postman and Curl command. You can choose either method based on your convenience and familiarity.  
    - To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.
![create-from-postman](https://github.com/user-attachments/assets/2356cfa9-ba21-44c4-8cc5-e92626d3a187)  
    - To run this from terminal using Curl, run the below  
```curl
$ curl -X POST -d "{\"operation\":\"create\",\"tableName\":\"lambda-apigateway\",\"payload\":{\"Item\":{\"id\":\"1\",\"name\":\"Bob\"}}}" https://$API.execute-api.$REGION.amazonaws.com/prod/DynamoDBManager
```  
3. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Explore table items" button from top right, and the newly inserted item should be displayed.
![dynamo-show-item](https://github.com/user-attachments/assets/f6e62b02-aafd-4e6c-a91a-55f09aebf560)  
4. To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table  
```json
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}
```
![dynamo-item-list](https://github.com/user-attachments/assets/fd88a6cb-3c0b-4050-8d13-f1a3b09ffe79)
We have successfully created a serverless API using API Gateway, Lambda, and DynamoDB!






