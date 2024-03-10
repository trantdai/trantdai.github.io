---
title: AI Security PoC Using LLM Guard
layout: post
post-image: "/assets/images/projects/ai_security2.png"
description: An AI Security proof of concept that leverage open source LLM Guard API
tags:
- project
- ai-security
- ai-powered-application
- blog
---

# High Level Architecture and Design

<br>
<br>
![Akamai Automation Framework](/assets/images/blog/llm_guard_api_poc.drawio.svg "AIDefender - LLM Guard API Proof of Concept")
<br>
<br>
The framework architecture is built upon the following building blocks:
- AI powered application
  - [Python code](https://github.com/trantdai/aidefender/blob/main/llm_guard_solution/test_llm_guard_integration.py)
  - [OpenAI Python SDK](https://pypi.org/project/openai/)
- Docker based deployment [LLM Guard API](https://llm-guard.com/api/overview/)
  - [Docker base deployment](https://llm-guard.com/api/deployment/)
  - AWS EC2 `c5a.8xlarge` instance as a hosting VM
- Azure OpenAI service
  - Large language model `gpt-35-turbo` deployment created on `Azure OpenAI Studio`

# Infrastructure Setup

The following is the summary of the infrastructure setup:

- The Azure OpenAI service is deployed and its access from Internet is controlled using the following:
  - Keys and endpoint
  - Networking firewall rules to allow access from selected IP ranges
- LLM Guard API is deployed using the [laiyer/llm-guard-api](https://hub.docker.com/r/laiyer/llm-guard-api) Docker image. Its Docker container is hosted on AWS EC2 `c5a.8xlarge` instance. The API access is made accessible from the Internet via an AWS Elastic IP address associated with the EC2 instance and restricted using the API []`AUTH_TOKEN`](https://llm-guard.com/api/deployment/#from-docker).

# Implementation

- [LLM Guard API Deployment on EC2 Ubuntu](https://github.com/trantdai/aidefender/tree/main/llm_guard_solution#llm-guard-api-deployment-on-ec2-ubuntu)
- [Direct Azure OpenAI Model Access Script](https://github.com/trantdai/aidefender/tree/main/llm_guard_solution#direct-azure-openai-model-access-script)
- [LLM Guard API Usage Script](https://github.com/trantdai/aidefender/tree/main/llm_guard_solution#llm-guard-api-usage-script)
- [AI Powered Application and LLM Guard Integration Script](https://github.com/trantdai/aidefender/tree/main/llm_guard_solution#ai-powered-application-and-llm-guard-integration-script)
