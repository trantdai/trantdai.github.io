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
The framework architecture is built upon the following building blocks and tools:
- Terraform
- Akamai provider for Terraform
- Akamai CLI Terraform
- Akamai services
- GitHub repositories
- CI/CD servers
- Artifactory repositories
- Docker repositories
- Vault
- AWS services including CloudFormation, IAM service accounts, roles, AWS S3 and DynamoDB tables

These building blocks are integrated to create the following types of workflows:
- Infrastructure administrative workflows
- Development workflows
- Akamai configuration management workflows

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

The pipeline builds a Docker image that is used as a Docker container in the other pipelines. This Docker image is versioned and pushed to a internal Docker registry.

The pipeline is integrated with a GitHub repository that has the following structure:

- Directory `requirements` consists of text files that store the versions of the tools Akamai CLI, Akamai CLI Terraform, AWS CLI, internal root CA bundle, required OS packages, PyPI packages, Terraform, Terraform providers including Akamai provider, Terraform linting and code security scanners.
- Directory `scripts` contains a shell script to install the list of the Terraform providers whose versions are fetched from the above Terraform provider text file
- Dockerfile
- CODEOWNERS
- Markdown files: CHANGELOG.md, CONTRIBUTING.md, README.md

The CI phase of the pipeline triggered when change makers introduce updates to the code base via a change branch (`feature/*`, `bugfix/*`, `hotfix/*`) performs the following build steps:

1. Extract the build branch name and the triggering timestamp. These will be used to form the version number or tag that is attached to the Docker image name following this convention `<docker_registry_hostname/terraform:<branch_name>-<yyyymmdd>.<hhmmss>`.
2. Build the Docker image based on the Dockerfile. The following are installed as part of this build:<br>
   - Terraform
   - Akamai Terraform provider. Note: The  provider is installed in `.terraform.d/plugins/registry.terraform.io` located in the login user's home directory
   - Akamai CLI Terraform
   - Internal root CA certificates
   - AWS CLI
   - Python and PyPi packages
   - Security scanning tools for Terraform code
3. Test the built Docker image by running it as a Docker container and check if the required tools are installed properly and the login user is actually the assigned non-root user. Unsuccessful tests fail the build.

In the CD phase, a pull request is created for one of the three following scenarios:
- Merging from the change branch to the master/main branch to create a development image
- Merging from the master/main branch to a release branch create a release image
- Merging from the release branch to the released branch to create a released image (stable/production image)

The merge of this pull request performs the following build steps:
1. Create the image version/tag based on the branch name and the triggering timestamp
2. Build the Docker image based on the Dockerfile as the step 2 of the CI phase
3. Push the Docker image to the Docker registry

## Automation Code Pipeline

This pipeline is integrated with a GitHub repository that has the following structure:
- Directory `appsec` is used to store Terraform code for Akamai WAF configuration management
- Directory `network-list` is used to store Terraform code for Akamai Network List (IP/Geo Firewall) configuration management
- Directory `property` is used to store Terraform code for Akamai property (CDN) configuration management
- Directory `scripts` is used to store some Python scripts that used automate some pipeline tasks
- Security scanning as code configuration file for the static application security testing

The pipeline consists of the following build steps to eventually package the Terraform code into a tar.gz compressed file and push it to an Artifactory repository:
1. Retrieve Artifactory credential (username and API key) for authentication with the Artifactory. This step is executed in both CI and CD phases.
2. Perform quality assurance and security scanning for the updated modules. This step is triggered when changes are submitted to a change branch in the CI phase only
   - Run a Python script to identify all directories of the updated modules
   - Cd into each directory to run `terraform init`, `terraform validate`, Terraform linting and code security scanning
   - Fail the build if any of the above commands produces non-zero exit
3. Perform the static application security testing for the Python code
4. Collect the build branch name and the triggering timestamp to be used as for the package versioning. This is executed in the CD phase only.
5. Package the updated modules using the commands `find`, `tar`, `gzip`, and push it to the Artifactory using the `curl` command. This is executed in the CD phase only.

## AWS IaC Pipeline

This pipeline is used to manage AWS services as code based on a list of CloudFormation YAML templates.

## Self Service Portal Pipeline

## Self Service Portal Use Cases

### Akamai IP/Geo Firewall Self Service Portal

### Akamai CDN ACL Self Service Portal
