# EchoLearn Deployment Guide

This guide walks you through deploying EchoLearn to AWS for your hackathon demo.

## Prerequisites

- AWS Account with Bedrock access enabled
- AWS CLI installed and configured
- Node.js 18.x or higher
- SAM CLI (Serverless Application Model) or Terraform

## Step 1: Enable Amazon Bedrock

1. Log into AWS Console
2. Navigate to Amazon Bedrock
3. Request access to models:
   - Amazon Titan Embeddings G1 - Text
   - Anthropic Claude v2 or v3
4. Wait for approval (usually instant for Titan, may take time for Claude)

## Step 2: Set Up AWS Infrastructure

### Option A: Using AWS SAM (Recommended)

1. **Install SAM CLI**
```bash
pip install aws-sam-cli
```

2. **Create SAM template** (`template.yaml`)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]

Resources:
  # DynamoDB Tables
  NotesTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub echolearn-notes-${Environment}
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: PK
          AttributeType: S
        - AttributeName: SK
          AttributeType: S
        - AttributeName: GSI1PK
          AttributeType: S
        - AttributeName: GSI1SK
          AttributeType: S
      KeySchema:
        - AttributeName: PK
          KeyType: HASH
        - AttributeName: SK
          KeyType: RANGE
      GlobalSecondaryIndexes:
        - IndexName: GSI1
          KeySchema:
            - AttributeName: GSI1PK
              KeyType: HASH
            - AttributeName: GSI1SK
              KeyType: RANGE
          Projection:
            ProjectionType: ALL
      SSESpecification:
        SSEEnabled: true
      PointInTimeRecoverySpecification:
        PointInTimeRecoveryEnabled: true

  # S3 Bucket for Media
  MediaBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub echolearn-media-${Environment}
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      CorsConfiguration:
        CorsRules:
          - AllowedOrigins: ['*']
            AllowedMethods: [GET, PUT, POST]
            AllowedHeaders: ['*']
            MaxAge: 3000

  # Lambda Function
  ApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub echolearn-api-${Environment}
      Runtime: nodejs18.x
      Handler: index.handler
      CodeUri: ./backend
      MemorySize: 1024
      Timeout: 30
      Environment:
        Variables:
          DYNAMODB_TABLE_NAME: !Ref NotesTable
          S3_BUCKET_NAME: !Ref MediaBucket
          BEDROCK_REGION: !Ref AWS::Region
          EMBEDDING_MODEL_ID: amazon.titan-embed-text-v1
          LLM_MODEL_ID: anthropic.claude-v2
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref NotesTable
        - S3CrudPolicy:
            BucketName: !Ref MediaBucket
        - Statement:
            - Effect: Allow
              Action:
                - bedrock:InvokeModel
              Resource: '*'
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /{proxy+}
            Method: ANY

  # API Gateway
  ApiGateway:
    Type: AWS::Serverless::Api
    Properties:
      StageName: !Ref Environment
      Cors:
        AllowOrigin: "'*'"
        AllowHeaders: "'*'"
        AllowMethods: "'GET,POST,PUT,DELETE,OPTIONS'"

Outputs:
  ApiUrl:
    Description: API Gateway endpoint URL
    Value: !Sub https://${ApiGateway}.execute-api.${AWS::Region}.amazonaws.com/${Environment}
  NotesTableName:
    Description: DynamoDB table name
    Value: !Ref NotesTable
  MediaBucketName:
    Description: S3 bucket name
    Value: !Ref MediaBucket
```

3. **Deploy with SAM**
```bash
cd backend
sam build
sam deploy --guided
```

Follow the prompts:
- Stack Name: `echolearn-dev`
- AWS Region: `us-east-1` (or your preferred region)
- Confirm changes: Y
- Allow SAM CLI IAM role creation: Y
- Save arguments to config file: Y

### Option B: Manual Setup (Quick & Dirty)

1. **Create DynamoDB Table**
```bash
aws dynamodb create-table \
  --table-name echolearn-notes-dev \
  --attribute-definitions \
    AttributeName=PK,AttributeType=S \
    AttributeName=SK,AttributeType=S \
  --key-schema \
    AttributeName=PK,KeyType=HASH \
    AttributeName=SK,KeyType=RANGE \
  --billing-mode PAY_PER_REQUEST
```

2. **Create S3 Bucket**
```bash
aws s3 mb s3://echolearn-media-dev
aws s3api put-bucket-cors --bucket echolearn-media-dev --cors-configuration file://cors.json
```

`cors.json`:
```json
{
  "CORSRules": [{
    "AllowedOrigins": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST"],
    "AllowedHeaders": ["*"],
    "MaxAgeSeconds": 3000
  }]
}
```

3. **Create Lambda Function**
```bash
cd backend
npm install
zip -r function.zip .

aws lambda create-function \
  --function-name echolearn-api-dev \
  --runtime nodejs18.x \
  --role arn:aws:iam::YOUR_ACCOUNT_ID:role/lambda-execution-role \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --memory-size 1024 \
  --environment Variables="{DYNAMODB_TABLE_NAME=echolearn-notes-dev,S3_BUCKET_NAME=echolearn-media-dev,BEDROCK_REGION=us-east-1}"
```

4. **Create API Gateway**
- Go to AWS Console → API Gateway
- Create REST API
- Create resources and methods
- Integrate with Lambda function
- Deploy to stage

## Step 3: Configure Backend Code

1. **Install dependencies**
```bash
cd backend
npm install aws-sdk @aws-sdk/client-bedrock-runtime @aws-sdk/client-dynamodb @aws-sdk/client-s3
```

2. **Create Lambda handler** (`backend/index.js`)
```javascript
const { BedrockRuntimeClient, InvokeModelCommand } = require('@aws-sdk/client-bedrock-runtime');
const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, PutCommand, GetCommand, QueryCommand } = require('@aws-sdk/lib-dynamodb');
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');

const bedrockClient = new BedrockRuntimeClient({ region: process.env.BEDROCK_REGION });
const dynamoClient = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const s3Client = new S3Client({});

exports.handler = async (event) => {
  // Route handling logic
  const path = event.path;
  const method = event.httpMethod;
  
  // Add your routing logic here
  // Example: POST /notes, GET /notes, POST /match, etc.
  
  return {
    statusCode: 200,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ message: 'Success' })
  };
};
```

## Step 4: Build Browser Extension

1. **Install dependencies**
```bash
cd extension
npm install react react-dom
npm install -D webpack webpack-cli babel-loader @babel/preset-react
```

2. **Create manifest.json**
```json
{
  "manifest_version": 3,
  "name": "EchoLearn",
  "version": "1.0.0",
  "description": "AI-powered learning companion",
  "permissions": ["storage", "activeTab", "scripting"],
  "host_permissions": ["<all_urls>"],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [{
    "matches": ["<all_urls>"],
    "js": ["content.js"],
    "css": ["notepad.css"]
  }],
  "action": {
    "default_popup": "popup.html",
    "default_icon": "icon.png"
  }
}
```

3. **Configure API endpoint**

Create `extension/config.js`:
```javascript
export const API_BASE_URL = 'https://YOUR_API_GATEWAY_URL/dev';
```

4. **Build extension**
```bash
npm run build
```

## Step 5: Load Extension in Browser

1. Open Chrome and navigate to `chrome://extensions`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select the `extension/dist` folder
5. Extension should now appear in your browser

## Step 6: Test the System

1. **Test note creation**
   - Click extension icon
   - Type a note
   - Verify it appears in DynamoDB

2. **Test context matching**
   - Navigate to a webpage
   - Verify relevant notes appear

3. **Test chat**
   - Open chat interface
   - Ask a question
   - Verify AI response

## Troubleshooting

### Bedrock Access Denied
- Ensure you've requested model access in Bedrock console
- Check IAM role has `bedrock:InvokeModel` permission

### CORS Errors
- Verify API Gateway has CORS enabled
- Check S3 bucket CORS configuration
- Ensure Lambda returns proper CORS headers

### Lambda Timeout
- Increase timeout in Lambda configuration
- Check Bedrock API latency
- Optimize embedding generation

### DynamoDB Throttling
- Switch to on-demand billing mode
- Increase provisioned capacity if using provisioned mode

## Monitoring

1. **CloudWatch Logs**
```bash
aws logs tail /aws/lambda/echolearn-api-dev --follow
```

2. **CloudWatch Metrics**
- Lambda invocations, errors, duration
- API Gateway requests, latency
- DynamoDB read/write units

3. **Cost Monitoring**
- Set up billing alerts in AWS Console
- Monitor Bedrock usage (most expensive component)

## Cleanup (After Hackathon)

```bash
# Delete SAM stack
sam delete --stack-name echolearn-dev

# Or manually delete resources
aws dynamodb delete-table --table-name echolearn-notes-dev
aws s3 rb s3://echolearn-media-dev --force
aws lambda delete-function --function-name echolearn-api-dev
```

## Production Considerations

For production deployment:
- Add Cognito authentication
- Implement rate limiting
- Add CloudFront CDN
- Set up CI/CD pipeline
- Enable AWS WAF
- Implement backup strategy
- Add monitoring and alerting
- Use custom domain name

## Support

For issues during deployment:
- Check AWS CloudWatch Logs
- Review IAM permissions
- Verify Bedrock model access
- Test API endpoints with Postman

Good luck with your hackathon! 🚀
