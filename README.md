# symptom-checker-agent


## Create ECR on AWS

## Login to AWS CLI

aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 905418198287.dkr.ecr.eu-north-1.amazonaws.com

## Create DynamoDB Table:
Go to AWS Console -> DynamoDB.
Create a new table named chat_sessions.
Primary Key: Set session_id as the Partition Key (String type).
Add Sort Key (Optional but Recommended): Add timestamp as the Sort Key (Number type).
Use default settings for capacity mode (On-demand is good for low usage).

## Add the backend code.
Create requirements.txt, code, and dockerfile. containerize backend, tag it, and push it to ECR

docker build -t symptom-checker-agent-backend .
docker tag symptom-checker-agent-backend:latest 905418198287.dkr.ecr.eu-north-1.amazonaws.com/symptom-checker-agent-backend:latest
docker push 905418198287.dkr.ecr.eu-north-1.amazonaws.com/symptom-checker-agent-backend:latest

## Generate Gemini API key on GCP

## Create Lambda function on AWS. Store the Gemini API key on it

Go to AWS Console -> Lambda.
Create a new function, choose "Container image" as the package type.
Select the image you just pushed from ECR.
Runtime: Python 3.10 (though the container image handles this).
IAM Role: Create a new role with basic Lambda permissions. Crucially, add an inline policy to this role that grants dynamodb:PutItem and dynamodb:GetItem permissions to your chat_sessions table.
Environment Variables: Add GEMINI_API_KEY (with your key from GCP) and DYNAMODB_TABLE_NAME (set to chat_sessions).
Memory: Set to 128MB (lowest for free tier).
Timeout: Increase to 30-60 seconds (LLM calls can take time).

## Create API Gateway (HTTP API):
Go to AWS Console -> API Gateway.
Create a new HTTP API.
Add an integration: Select "Lambda", choose your SymptomCheckerAgent Lambda function.
Create a POST route (e.g., /chat).
Deploy the API. Note the Invoke URL. This is the endpoint your frontend will call.
