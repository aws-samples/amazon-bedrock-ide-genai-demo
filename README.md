# Amazon Bedrock IDE Demo

This repository provides supplemental artifacts for our blog post "Build generative AI applications in a few clicks with Amazon Bedrock IDE in Amazon SageMaker Unified Studio" It includes structured sales data, unstructured product data and customer reviews, OpenAPI schemas, CloudFormation templates, and LLM prompts.

## Description

In this blog post, we'll show how, with just a few clicks, anyone in your company can create a generative AI chat agent application that analyzes sales performance data using Amazon Bedrock IDE, a key component of SageMaker Unified Studio. We guide you through:
- Setting up your development environment
- Leveraging pre-built models and templates
- Integrating APIs and knowledge bases
- Deploying your application

The use case demonstrates:
- Creating a text-to-SQL engine for natural language queries
- Processing structured data for insights
- Integrating unstructured data sources
- Combining results for comprehensive analysis

## Dataset

- Main dataset: [Sample CSV Files / Data Sets for Testing](https://excelbianalytics.com/wp/downloads-18-sample-csv-files-data-sets-for-testing-sales/) (Sales Datasets up to 5 Million Records)
- Additional data: Synthetic product data and customer reviews
- Automatic dataset deployment via CloudFormation template

## Architecture

![Architecture Diagram](assets/architecture_diagram.png "Architecture of the generative AI Sales analysis app")

### Provisioned Resources

The CloudFormation template creates:
- AWS Lambda functions for Athena query execution
- IAM roles for Lambda-Athena access
- API Gateway REST API with POST method
- S3 bucket for data storage
- AWS Glue Database and Tables for Sales dataset

## User Interaction Flow

Amazon Bedrock IDE chat agent application enable builders to execute multistep tasks in generative AI applications. In this architecture, a chat agent initiate the process by sequentially reasoning through the user's query. It breaks down the query into smaller components and understand the context behind each component. Through contextual understanding, the chat agent maintain a coherent Chain-of-Thought (CoT), ensuring that each part of the query contributes logically to the overall intent. The following steps show how Bedrock IDE chat agent uses chaining of thoughts to execute the task of converting natural language to either SQL queries or to apply a semantic search using retrieval augmented generation (RAG):
### 1.	Prompt 
- The user initiates the process by providing a natural language prompt, such as "What were the key insights from the positive and negative product reviews for our high-revenue cosmetics line in Asia?" 
- The prompt is sent to Amazon Bedrock IDE, which serves as the interface for interacting with the generative AI application.
### 2.	Agents for Amazon Bedrock: 
Bedrock IDE forwards the input to a chat agent. The chat agent uses prompts, conversation history, functions, knowledge bases, and specific instructions to interpret and process the input.
### 3.	Chain-of-Thought (CoT) Reasoning
- The chat agent use Chain-of-Thought reasoning to break down a complex user query inter simpler, logical parts.
- The chat agents then determine a course of action for how they will find the necessary information to respond to the user query (ex. Search the knowledge base or query the database).
### 4.	Task Execution
- The chat agent executes the necessary tasks based on the broken-down query components:
- Structured Data Query:
    - The chat agent uses the "Query Database Function"
    - It then queries the database by interacting with Amazon API Gateway.
    - Amazon API Gateway forwards the requests to a the Lambda function to processe the SQL queries using Amazon Athena.
- Unstructured Data Query:
    - The chat agent searches the knowledge base for relevant information.
    - The knowledge base, containing sales related insights, uses Amazon Titan embedding model for data indexing and retrieval.
    - Amazon OpenSearch Service facilitates searching through the knowledge base. 
### 5.	Response
- The chat agent compiles the results from both structured and unstructured data sources.
- The processed and formatted response is sent back to the user through Bedrock IDE.
### 6.	Session Store: 
Throughout the process, Bedrock IDE maintains the conversation history and session state to handle follow-up questions and maintain context.


## Requirements

- AWS CLI (v2.x)
- AWS SDK (v3.x)
- Python 3.8+
- AWS account with appropriate permissions

## Deployment

### Console Deployment

Download the GitHub repository (includes OpenAPI schemas, text files, and example pages)

#### Main Stack Deployment
1. Download all files from the GitHub repository
2. Navigate to the AWS CloudFormation console
3. Click "Create stack" at the top right
4. Select "Upload a template file"
5. Upload the `cloudformation.yaml` template file
6. Click "Next"
7. Enter a stack name (e.g., "amazon-bedrock-ide-genai-demo")
8. Review the stack details (no custom parameters required)
9. Click "Next"
10. Review the configuration and acknowledge IAM resource creation
11. Click "Create stack"
12. Wait for stack creation to complete (this may take several minutes)

#### Retrieve Important Information
After stack creation, go to the Outputs tab to find:
- `TextToSqlEngineAPIGatewayURL`: API Gateway endpoint URL
- Navigate from the AWS console to AWS Secrets Manager and find the secret `<StackName>-api-keys`. Click on Retrieve secret and copy the apiKey value from the plaintext string `{"clientId":"default","allowedOperations":["query"],"apiKey":"xxxxxxxx"}`


Note: The Sales data will be automatically downloaded and stored in the S3 bucket during stack creation.

### CLI Deployment

Note: Commands are for Linux-based systems. Windows users may need to adjust syntax.

1. Clone the repository:
```bash
git clone https://github.com/aws-samples/amazon-bedrock-ide-genai-demo.git
cd amazon-bedrock-ide-genai-demo
```

2. Deploy main stack:
```bash
aws cloudformation create-stack \
    --stack-name amazon-bedrock-ide-genai-demo \
    --template-body file://cloudformation/cloudformation.yaml \
    --capabilities CAPABILITY_NAMED_IAM
```

3. Monitor deployment:
```bash
aws cloudformation describe-stacks \
    --stack-name amazon-bedrock-ide-genai-demo \
    --query 'Stacks[0].StackStatus'
```

4. Get deployment information:

API Gateway URL
```bash
aws cloudformation describe-stacks \
    --stack-name amazon-bedrock-ide-genai-demo \
    --query 'Stacks[0].Outputs[?OutputKey==`TextToSqlEngineAPIGatewayURL`].OutputValue' \
    --output text
```
Get AWS Secrets Manager 
```bash
aws secretsmanager get-secret-value \
    --secret-id $(aws cloudformation describe-stacks \
    --stack-name amazon-bedrock-ide-genai-demo \
    --query 'Stacks[0].Outputs[?OutputKey==`ApiKeySecretArn`].OutputValue' \
    --output text) \
    --query 'SecretString' \
    --output text | grep -o '"apiKey":"[^"]*"' | cut -d'"' -f4
```
## Usage

Query Athena
```bash
curl -X POST <TextToSqlEngineAPIGatewayURL>/query \
    -H "x-api-key: <AWS Secrets Manager apiKey>" \
    -H "Content-Type: application/json" \
    -d '{
        "query": "SELECT * FROM sales_records_parquet Limit 10",
        "database": "sales"
    }'
```

## Clean Up
1. Delete CloudFormation stack
```bash
aws cloudformation delete-stack --stack-name amazon-bedrock-ide-genai-demo
```
2. Empty and delete S3 buckets
3. Delete any manually created resources