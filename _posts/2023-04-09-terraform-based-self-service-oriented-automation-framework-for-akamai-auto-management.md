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
- Development workflows:
- Amakai configuration management workflows

## Infrastructure Administrative Workflows

Akamai service owners and engineers access to various building blocks to set up, configure, and maintain the framework infrastructure. This includes the initial set up such as account creation, access permissions as well as ongoing configuration fine-tunning and support. These workflows are enabled by the following types of CI/CD pipelines:<br>
  - AWS IaC pipeline
  - Terraform configuration and state onboarding pipelines for existing Akamai configurations
  - Akamai Terraform configuration and state OOB sync pipelines
  - Vault IaC pipeline

## Development Workflows

Automation developers develop and update the Akamai automation and infrastructure code base that triggers development CI/CD pipeline to package and publish the code base to software artifactory and Docker repositories. They also develop the CI/CD pipelines that are used by Akamai service owners, engineers and end users. These workflows are enabled by the following types of CI/CD pipelines:<br>
- - AWS IaC pipeline to perform CloudFormation template checks and security and compliance scanning
- Automation code pipeline to perform unit testing, scan code for security issues and package Terraform and Python code and publish it to software artifactory repositories
- Docker image pipeline to package Docker images and push them to a Docker repositories. These images are used by all the other pipelines like AWS IaC pipelines, Automation code pipeline, and self service pipelines.

## Amakai Configuration Management Workflows

Akamai service consumers/end users raise and submit configuration management requests against automation configuration data repositories, i.e. self service portals. Configuration management approvers and/or peer reviewers are then notified of these requests and conduct the approval process. The approval leads to the configuration data merge into the repository. This trigger a configuration deployment/update CI/CD pipeline that pulls the automation code from the software artifactory repository, runs the code within a Docker container hosted by a CI agent and use the merged configuration data to deploy requested configuration changes onto the Akamai Staging and Production networks.

# Automation Solution Implementations

## Akamai IP/Geo Firewall Self Service Portal

## Akamai CDN ACL Self Service Portal