# Text File Summarization with AWS

This repository contains an AWS Lambda function designed to summarize text files uploaded via API Gateway. The Lambda function interacts with the Mistral AI model to generate summaries of the provided text.

## Architecture

![Architecture Diagram](path/to/architecture-image.png)

*Replace `path/to/architecture-image.png` with the actual path to your architecture image.*

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
$fileContent = Get-Content -Raw -Path "E:\Downloads\1.14.txt"

# Create JSON body with file content
$jsonBody = @{ file_content = $fileContent } | ConvertTo-Json

# Define the API endpoint
$apiUrl = "https://0qqxe20xs3.execute-api.us-east-1.amazonaws.com/prod/text"

# Invoke the API and save the response
$response = Invoke-RestMethod -Uri $apiUrl -Method Post -ContentType "application/json" -Body $jsonBody

# Save summarized text to a file
$response.summarized_text | Out-File -FilePath "summary.txt"
```
### 2. Verify the Response
The summarized text will be saved to `summary.txt`. Check this file to see the result of the summarization.

## Deployment

### 1. Create the Lambda Function:
- Go to the AWS Lambda console.
- Create a new function and paste the code into the Lambda editor.

### 2. Set Up API Gateway:
- Create a new API in API Gateway.
- Create a POST method and integrate it with your Lambda function.

### 3. Deploy API Gateway:
- Deploy the API to a stage and use the provided URL to test your Lambda function.

