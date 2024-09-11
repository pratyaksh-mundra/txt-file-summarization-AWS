# Text File Summarization with AWS

This repository contains an AWS Lambda function designed to summarize text files uploaded via API Gateway. The Lambda function interacts with the Mistral AI model to generate summaries of the provided text.

## Architecture

![Architecture Diagram](path/to/architecture-image.png)

*Replace `path/to/architecture-image.png` with the actual path to your architecture image.*

## Lambda Function

The Lambda function reads text data from an HTTP request, sends it to the Mistral AI model for summarization, and returns the summarized text.

# Usage

## 1. Upload a Text File and Summarize

### PowerShell Commands:

```powershell
# Read content from a text file
$fileContent = Get-Content -Raw -Path "E:\Downloads\1.14.txt"

# Create JSON body with file content
$jsonBody = @{ file_content = $fileContent } | ConvertTo-Json

# Define the API endpoint
$apiUrl = "https://0qqxe20xs3.execute-api.us-east-1.amazonaws.com/prod/text"

# Invoke the API and save the response
$response = Invoke-RestMethod -Uri $apiUrl -Method Post -ContentType "application/json" -Body $jsonBody
$response.summarized_text | Out-File -FilePath "summary.txt"
```

### Command Line Query for Windows:
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

