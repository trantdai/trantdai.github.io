---
title: Terraform Based Self Service Oriented Automation Framekwork for Akamai Auto-management
layout: post
post-image: "/assets/images/projects/akamai_logo.svg"
description: Automation framework using DevOps methodology and Akamai provider for Terraform to build self service auto-management solutions
tags:
- project
- application-security
- content-delivery-network
- network-security
- self-service
- blog
---

# Framework Architecture and Design
<br>
<br>
![Akamai Automation Framework](/assets/images/blog/terraform_akamai_automation.drawio.svg "Akamai Automation Framework")
<br>
<br>
The framework architecture is built upon the following building blocks:
- Terraform
- Akamai provider for Terraform
- Akamai CLI Terraform
- Akamai services
- GitHub repositories
- CI/CD servers
- Artifactory repositories
- Docker repositories
- Vault
- AWS S3 and DynamoDB tables

These building blocks are integrated to create the following types of workflows:
- Infrastructure administrative workflows
- Development workflows
- Amakai configuration management workflows

## Infrastructure Administrative Workflows

Akamai service owners and engineers access to various building blocks to set up, configure, and maintain the framework infrastructure. This includes the initial set up such as account creation, access permissions as well as ongoing configuration fine-tunning and support. These workflows are enabled by the following types of CI/CD pipelines:<br>
  - AWS IaC pipeline
  - Terraform configuration and state onboarding pipelines for existing Akamai configurations
  - Akamai Terraform configuration and state OOB sync pipelines
  - Vault IaC pipeline

## Development Workflows

Automation developers develop and update the Akamai automation and infrastructure code base that triggers development CI/CD pipeline to package and publish the code base to software artifactory and Docker repositories. They also develop the CI/CD pipelines that are used by Akamai service owners, engineers and end users. These workflows are enabled by the following types of CI/CD pipelines:<br>
- AWS IaC pipeline to perform CloudFormation template checks and security and compliance scanning
- Automation code pipeline to perform unit testing, scan code for security issues and package Terraform and Python code and publish it to software artifactory repositories
- Terraform Docker image pipeline to package Docker images and push them to a Docker repositories. These images are used by all the other pipelines like AWS IaC pipelines, Automation code pipeline, and self service pipelines.

## Akamai Configuration Management Workflows

Akamai service consumers/end users raise and submit configuration management requests against automation configuration data repositories, i.e. self service portals via the GitHub feature branches and pull requests. This submission triggers a CI/CD pipeline that retrieves the runtime credentials such as Akamai API clients and AWS service account credentials from the vault, pulls the automation code from the software artifactory repository, runs the Terraform code (`terraform plan`) within a Docker container hosted by a CI agent to generate a Terraform plan for review.
Configuration management approvers and/or peer reviewers are then notified of these requests and conduct the approval process. The approval leads to the configuration data merge into the repository. This trigger a configuration deployment/update CI/CD pipeline that retrieves the runtime credentials such as Akamai API clients and AWS service account credentials from the vault, pulls the automation code from the software artifactory repository, runs the code (`terraform apply`) within a Docker container hosted by a CI agent and use the merged configuration data to deploy requested configuration changes onto the Akamai Staging and Production networks.
The state locking is acquired and released before and after the Terraform operations `terraform plan` and `terraform apply`.

# Akamai Automation Framework Implementation

## Terraform Docker Image Pipeline

This pipeline build a Docker image that is run as a Docker container in the other pipelines. This Docker image is versioned and pushed to a internal Docker registry.

The CI part of the pipeline triggered when change makers introduce updates to the code base via a change branch (`feature/*`, `bugfix/*`, `hotfix/*`) performs the following build steps:

1. Extract the build branch name and the triggering timestamp. These will be used to form the version number or tag that is attached to the Docker image name following this convention `<docker_registry_hostname/terraform:<branch_name>-<yyyymmdd>.<hhmmss>`.
2. Build the Docker image based on the Dockerfile. The following are installed as part of this build:<br>
   - Terraform
   - Akamai Terraform provider and Akamai CLI Terraform
   - Internal root CA certificates
   - AWS CLI
   - Python and PyPi packages
   - Security scanning tools for Terraform code
3. Test the built Docker image by running it as a Docker container and check if the required tools are installed properly and the login user is actually the assigned non-root user. Unsuccessful tests fail the build.

## Automation Code Pipeline

## AWS IaC Pipeline

## Self Service Portal Pipeline

## Self Service Portal Use Cases

### Akamai IP/Geo Firewall Self Service Portal

### Akamai CDN ACL Self Service Portal
