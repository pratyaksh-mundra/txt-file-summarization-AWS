# Text File Summarization with AWS

This repository contains an AWS Lambda function designed to summarize text files uploaded via API Gateway. The Lambda function interacts with the Mistral AI model to generate summaries of the provided text.

## Architecture

![Architecture Diagram](architecture.png)


## Overview

This solution leverages AWS Lambda to run a text summarization service triggered by an API Gateway endpoint. It uses the Mistral AI model via AWS Bedrock for processing and returning the summaries.

### Key Components:
- **AWS Lambda**: The compute service that processes the text and interacts with the Mistral AI model.
- **API Gateway**: Exposes the API that allows users to upload text files.
- **Amazon Bedrock**: A fully managed service from AWS that makes it easy to build and scale generative AI applications. It provides access to foundational models like Mistral for tasks such as text summarization.
- **Mistral AI Model (via Amazon Bedrock)**: A language model accessed through Amazon Bedrock, used for summarizing large blocks of text in this solution.

## Deployment

### 1. Create the Lambda Function:
- Go to the AWS Lambda console.
- Create a new function and paste the code into the Lambda editor.

### 2. Set Up API Gateway:
- Navigate to API Gateway in the AWS console.
- Create a new API in API Gateway.
- Create a POST method and integrate it with your Lambda function.

### 3. Deploy API Gateway:
- Deploy the API to a stage and use the provided URL to test your Lambda function.
- Use the provided URL to test the Lambda function through the API Gateway.

## Lambda Function

The Lambda function reads text data from an HTTP request, sends it to the Mistral AI model for summarization, and returns the summarized text.

### Lambda Function Code

```python
import json
import boto3

client_bedrock = boto3.client('bedrock-runtime')

def lambda_handler(event, context):
    # Check if the request contains body
    if 'body' not in event:
        return {
            'statusCode': 400,
            'body': json.dumps('No content found in the request.')
        }

    # Extract the file content from the request body
    try:
        body = json.loads(event['body'])
        file_content = body.get('file_content', '')
    except (KeyError, json.JSONDecodeError) as e:
        return {
            'statusCode': 400,
            'body': json.dumps(f"Error in request body: {str(e)}")
        }

    # Prepare the prompt for the Mistral model
    formatted_prompt = f"<s>[INST] Summarize the following text: {file_content} [/INST]"

    # Format the request payload using the Mistral model's structure
    mistral_request = {
        "prompt": formatted_prompt,
        "max_tokens": 200,
        "temperature": 0.5,
        "top_p": 0.9,
        "top_k": 50
    }

    # Convert the request payload to JSON
    request_payload = json.dumps(mistral_request)

    # Invoke the Mistral model
    try:
        client_bedrock_request = client_bedrock.invoke_model(
            modelId="mistral.mistral-large-2402-v1:0",
            body=request_payload,
            contentType="application/json",
            accept="application/json"
        )
        
        client_bedrock_byte = client_bedrock_request['body'].read()
        model_response = json.loads(client_bedrock_byte)

        print("Model Response:", model_response)  # Debugging line
        
        # Extract the summarized text from the response
        summarized_text = model_response.get("outputs", [{}])[0].get("text", "No text generated.")
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps(f"Error invoking Mistral model: {str(e)}")
        }

    return {
        'statusCode': 200,
        'body': json.dumps({'summarized_text': summarized_text})
    }

```

# Usage

## 1. Upload a Text File and Summarize

### Command Line Query for Windows for calling the api:
```powershell
# Read content from a text file
$fileContent = Get-Content -Raw -Path "path/to/sample.txt"

# Create JSON body with file content
$jsonBody = @{ file_content = $fileContent } | ConvertTo-Json

# Define the API endpoint
$apiUrl = "https://xxxxxxx.execute-api.us-east-1.amazonaws.com/prod/text"

# Invoke the API and save the response
$response = Invoke-RestMethod -Uri $apiUrl -Method Post -ContentType "application/json" -Body $jsonBody

# Save summarized text to a file
$response.summarized_text | Out-File -FilePath "summary.txt"
```
### 2. Verify the Response
The summarized text will be saved to `summary.txt`. Check this file to see the result of the summarization.

# Results

- the original file is from a Courcera certification course transcript 1.14.txt

- this is the summary provided:

    The text discusses the use of existing Language Learning Models (LLMs) for application development and the situations where pretraining a model from scratch might be necessary. This is typically when the target domain uses unique vocabulary and language structures not commonly found in everyday language, such as in legal or medical fields.
    
    In the legal domain, terms like 'mens rea' and 'res judicata', and the use of common words in different contexts, like 'consideration', can pose challenges for existing LLMs. Similarly, medical language contains many uncommon words and uses language in idiosyncratic ways, which may not appear frequently in training datasets.
    
    Pretraining a model from scratch can result in better models for highly specialized domains like law, medicine, finance, or science. The text then discusses BloombergGPT, a large language model pretrained for the specific domain of finance. The researchers used a combination of finance data and general-purpose tax data for pretraining.
    
    The Bloomberg researchers followed the Chinchilla scaling laws for guidance but had to make tradeoffs due to real-world constraints. The model size of BloombergGPT roughly follows the Chinchilla approach, but the actual number of tokens used to pretrain the model is below the recommended Chinchilla value due to the limited availability of financial domain data.
    
    The text concludes with a recap of the week's topics, including common use cases for LLMs, the transformer architecture, model training during the pretraining phase, computational challenges, quantization, and scaling laws for LLMs.
