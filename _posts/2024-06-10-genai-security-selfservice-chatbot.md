---
title: GenAI Powered Security Self-Service Chatbot
layout: post
post-image: "/assets/images/blog/20240610/generative-AI-customer-self-service.jpg"
description: A PoC AWS Bedrock GenAI-powered security self-service chatbot
tags:
- project
- selfservice-security
- ai-powered-application
- blog
---

# High Level Architecture and Design

![GenAI Powered Security Self-service Chatbot HLSA](/assets/images/blog/20240610/security_selfservice_chatbot_hlsa.png "GenAI Powered Security Self-service Chatbot HLSA")

The PoC generative AI powered security self-service chatbot is built upon the following building blocks:

- AI powered application<br>
  - [Front end chatbot code](https://github.com/trantdai/genai/blob/main/selfservicebot.py)
  - [AWS Lambda Knowledge Base autosync code](https://github.com/trantdai/genai/blob/main/autosync_kb.py)
- Infrastructure<br>
  - [GitHub code repository](https://github.com/trantdai/genai)
  - [GitHub Actions - CI/CD engine](https://github.com/trantdai/genai/tree/main/.github/workflows)
  - AWS services<br>
    - AWS IAM OIDC provider for GitHub
    - AWS IAM roles
    - AWS S3
    - AWS Lambda function
    - AWS Knowledge Base
    - AWS Text Embeddings model
    - AWS OpenSearch Serverless vector database
    - AWS Anthropic Claude 3 foundation model

# Infrastructure Setup

## AWS IAM OIDC Provider for GitHub

![AWS IAM OIDC provider for GitHub](/assets/images/blog/20240610/aws_github_oidc_idp.png "AWS IAM OIDC provider for GitHub")

This OIDC provider authenticates the GitHub Actions Knowledge Base update workflow and authorize it to assume the IAM role `GitHubActionsAssumeRoleForSelfServiceChatBot` that is assigned to the OIDC provider.

![GitHub Actions Role](/assets/images/blog/20240610/github_role.png "GitHub Actions Role")

## AWS Knowledge Base

AWS Bedrock Knowledge Base is the AWS Bedrock orchestration service that co-ordinates the following AWS services to convert data from data source to indexed vectors in the vector database:

- AWS S3
- AWS Lambda function
- AWS Knowledge Base
- AWS Text Embeddings model
- AWS OpenSearch Serverless vector database
- AWS Anthropic Claude 3 foundation model
<br>

### AWS Bedrock Model Access
![AWS Bedrock Model Access](/assets/images/blog/20240610/aws_bedrock_model_access.png "AWS Bedrock Model Access")

### AWS Bedrock Knowledge Base Configurations
![AWS Bedrock Knowledge Base Configurations](/assets/images/blog/20240610/aws_bedrock_knowledge_base.png "AWS Bedrock Knowledge Base Configurations")

### AWS IAM Service Role for Bedrock Knowledge Base
![AWS IAM Service Role for Bedrock Knowledge Base](/assets/images/blog/20240610/aws_iam_kb_role.png "AWS IAM Service Role for Bedrock Knowledge Base")

### AWS IAM Service Role for Knowledge Base Autosync Lambda
![Knowledge Base Autosync Lambda Permission Configurations](/assets/images/blog/20240610/kbautosync_lambda_permissions.png "Knowledge Base Autosync Lambda Permission Configurations")
![AWS IAM Service Role for Knowledge Base Autosync Lambda](/assets/images/blog/20240610/kbautosync_lambda_iam_role.png "AWS IAM Service Role for Knowledge Base Autosync Lambda")

# Application Workflows

## Knowledge Base Update Workflow
1. Process owners (security engineers) create a documentation branch from the `main` branch of the code repository
2. Process owners (security engineers) document and/or update self-service workflows and security knowledge in Markdown files, push the changes to the documentation branch, and create a PR to the `main` branch
3. The GitHub Actions workflow triggered by the PR merge uploads the new and updated documents to an AWS S3 bucket
4. The S3 bucket document upload creates an event that triggers the Lambda function that performs the Knowledge Base Sync:
   1. Knowledge Base service retrieves updated source data from the S3 bucket
   2. It splits data into chunks (aka performs data chunkcing)
   3. It sends data chunks to the text embeddings model to create vectors
   4. It indexes and saves the created vectors in AWS OpenSearch Serverless vector database
5. Test the updated knowledge base by running a query to generate responses. If the reponses are not accurate, repeate the whole workflow to fine tune the answers.

## Security Self-Service Chatbot Workflow
1. Security consumers ask the chatbot security domain questions like `What does Cloudflare web application firewall do?` and `What does Cloudflare rate limit do?` to gain understanding of the security technlogies in question<br>![WAF Question Answer](/assets/images/blog/20240610/waf_question_answer.png "WAF Question Answer")<br>![Rate Limit Question Answer](/assets/images/blog/20240610/rate_limit_question_answer.png "Rate Limit Question Answer")
2. Security consumers ask the chatbot about how to create and manage the security configuration using an IaC tool like `Terraform` to get the suggested sample code. Some examples of the questions are `Tell me how to create Cloudflare WAF managed ruleset in Terraform` and `Show me Terraform code to create a Cloudflare HTTP rate limit resource`<br>![WAF Ruleset Prompt Answer](/assets/images/blog/20240610/chatbot_prompt_waf_ruleset.png "WAF Ruleset Prompt Answer")<br>![HTTP Rate Limit Prompt Answer](/assets/images/blog/20240610/chatbot_prompt_rate_limit.png "[HTTP Rate Limit Prompt Answer")
3. Security consumers compose their security self-service requests based on the suggested sample code and ask the chatbot to create a GitHub pull request on their behalf like this prompt ```createpr->resource "cloudflare_rate_limit" "example" { zone_id = "your_zone_id" name = "example-rate-limit" description = "Example rate limit" disabled = false match { request { methods = ["GET", "POST"] schemes = ["HTTP", "HTTPS"] path { values = ["/example/*"] } } } threshold = 10 period = 1 action { mode = "simulate" } }```
4. The chatbot creates a PR<br>![Rate Limit PR Creation](/assets/images/blog/20240610/create_pr_rate_limit.png "Rate Limit PR Creation")<br>![Rate Limit PR Creation on GitHub Repo](/assets/images/blog/20240610/pull_request_created_by_chatbot.png "Rate Limit PR Creation on GitHub Repo")
5. Process owners (security engineers) review and merge the PR that triggers the configuration management pipeline


# Chatbot Front-End Hosting

## Local Hosting/Testing

Set up the following environment variables:
```Windows
SET GITHUB_PAT=ghp_*******************

SET AWS_ACCESS_KEY_ID=*******************
SET AWS_SECRET_ACCESS_KEY=*******************
SET AWS_SESSION_TOKEN=*******************
```

```Linux
export GITHUB_PAT=ghp_*******************

export AWS_ACCESS_KEY_ID=*******************
export AWS_SECRET_ACCESS_KEY=*******************
export AWS_SESSION_TOKEN=*******************
```

Install Python packages:
```
pip install -r requirements.txt
```

Run the chatbot:
```Python
streamlit run selfservicebot.py
```

## Hosting via AWS
TBD

# References
1. [Security self service GitHub code repository](https://github.com/trantdai/genai)
2. [Use IAM roles to connect GitHub Actions to actions in AWS](https://aws.amazon.com/blogs/security/use-iam-roles-to-connect-github-actions-to-actions-in-aws/)
3. [Implementing RAG App Using Knowledge Base from Amazon Bedrock and Streamlit](https://medium.com/@saikatm.courses/implementing-rag-app-using-knowledge-base-from-amazon-bedrock-and-streamlit-e52f8300f01d)
4. [Amazon Bedrock & AWS Generative AI - Beginner to Advanced](https://cba.udemy.com/course/amazon-bedrock-aws-generative-ai-beginner-to-advanced/)
5. [Build a basic LLM chat app](https://docs.streamlit.io/develop/tutorials/llms/build-conversational-apps)