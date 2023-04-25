---
title: Terraform Based Self Service Oriented Automation Framework for Akamai Auto-management
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

The pipeline is integrated with the Terraform Docker GitHub repository that has the following structure:

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

This pipeline is integrated with the automation code GitHub repository that has the following structure:
- Directory `appsec` is used to store Terraform code for Akamai WAF configuration management
- Directory `network-list` is used to store Terraform code for Akamai Network List (IP/Geo Firewall) configuration management
- Directory `property` is used to store Terraform code for Akamai property (CDN) configuration management
- Directory `scripts` is used to store some Python scripts that used automate some pipeline tasks
- Security scanning as code configuration file for the static application security testing

The pipeline consists of the following build steps to eventually package the Terraform code into a tar.gz compressed file and push it to an Artifactory repository:
1. Retrieve Artifactory credential (username and API key) for authentication with the Artifactory. This step is executed in both CI and CD phases.
2. Perform quality assurance and security scanning for the updated modules. This step is triggered when changes are submitted to a change branch in the CI phase only<br>
   - Run a Python script to identify all directories of the updated modules
   - Cd into each directory to run `terraform init`, `terraform validate`, Terraform linting and code security scanning
   - Fail the build if any of the above commands produces non-zero exit
3. Perform the static application security testing for the Python code
4. Collect the build branch name and the triggering timestamp to be used as for the package versioning. This is executed in the CD phase only.
5. Package the updated modules using the commands `find`, `tar`, `gzip`, and push it to the Artifactory using the `curl` command. This is executed in the CD phase only.

## AWS IaC Pipeline

This pipeline is used to manage AWS services as code through a list of CloudFormation YAML templates stored in the AWS IaC GitHub repository. These templates define the following resources in a CloudFormation nested stack across `development` and `production` AWS accounts:
- AWS IAM ManagedPolicy for AdminRole, PipelineRole, CloudFormationRole, and S3BackendRole roles
  - The `PipelineRole` has only enough permissions to:<br>
    - retrieve the CloudFormation templates from a central S3 bucket
    - create, update and delete CloudFormation stacks
    - Pass role to `CloudFormationRole` via action `iam:PassRole`
  - The `CloudFormationRole` has all the required permissions to manage S3 buckets, DynamoDB tables, VPC interface endpoints and security groups
- S3 Terraform state buckets to keep Terraform states managed by the Terraform workload executed on the on-prem CI/CD agents and bucket policies that denies HTTP traffic and allows all access from the relevant VPCs
- DynamoDB table to enable the Terraform state locking feature and audit log bucket to keep S3 Terraform state bucket access logs
- VPC interface endpoints that make S3 Terraform states accessible from the on-prem Terraform workload via `AWS Direct Connect`
- Security group protecting the VPC interface endpoint by allowing only the on-prem Terraform workload to target S3 Terraform states

This pipeline make use of other AWS infrastructure services like AWS VPC, routing, KMS and SSM that are managed centrally by different platform pipelines.

## Onboarding and Out-of-Band Sync Pipelines

Each Akamai configuration like `network list` (IP/Geo firewalling) and `property` has its own onboarding and OOB sync pipeline to onboard the configuration onto the self service platform and allow for the OOB changes made via Akamai Control Center in case of emergencies.

### Network List Pipeline

This pipeline consist of the following build steps:
1. Retrieve the configured Akamai API client from Vault
2. Retrieve all access control groups that are accessible by the provided API client
3. Retrieve all network list names belonging to a given access control group
4. Use the above retrieved network list names as the input in the `terraform.tfvars` of the `id-fetch` module to retrieve network list configurations/contents
   1. Run `terraform apply` in the `id-fetch` module to create a local state
   2. Run `terraform output -json > netlist_name_id.json`
   3. Use in-line Python script to load `netlist_name_id.json` into a JSON object through which the network list names and IDs are iterated. They are then put together in a Terraform import command with the format `f"terraform import 'akamai_network_list.network_lists[\"{netlist_name}\"]' {netlist_id}\n"` and written into the file `import.sh`
   4. Run `import.sh` command to create the local state
   5. Run commands `terraform show -json > terraform.tfstate.json` and `jq . terraform.tfstate.json` to show the network list contents
5. Retrieve AWS access key id and secret access key from Vault and load them into the env variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
6. Use the above AWS credentials to assume `PipelineRole` and update env variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`:
   ```
   aws sts assume-role \
      --role-arn arn:aws:iam::<aws account id>:role/PipelineRole \
      --role-session-name spmkeeper@gmail.com > authz.json
   
   export AWS_ACCESS_KEY_ID=$(cat authz.json | jq -r ".Credentials.AccessKeyId")
   export AWS_SECRET_ACCESS_KEY=$(cat authz.json | jq -r ".Credentials.SecretAccessKey")
   export AWS_SESSION_TOKEN=$(cat authz.json | jq -r ".Credentials.SessionToken")
   ```
7. Perform state onboarding or OOB sync
   1. Perform the workflow as described in the step 4 above to create `import.sh` and `staterm.sh`. The `staterm.sh` consists of the `terraform state rm` commands, one per line with this format `f"terraform state rm 'akamai_network_list.network_lists[\"{netlist_name}\"]'\n"`
   2. Run `terraform init` with the AWS S3 backend configurations. **Note**: To make this command work, the `S3BackendRole` needs to be configured to allow `PipelineRole` to assume it.
      ```
      terraform init -backend-config="bucket=<S3 bucket>" \
                     -backend-config="key=<terraform state file as S3 object>" \
                     -backend-config="encryption=true" \
                     -backend-config="region=<AWS region>" \
                     -backend-config="dynamodb_table=<AWS DynamoDB table>" \
                     -backend-config="kms_key_id=<AWS KMS key id>" \
                     -backend-config="role_arn=<arn:aws:iam::<aws account id>:role/S3BackendRole>" \
      ```
   3. If the task is onboarding, run `import.sh`. If the task is OOB syncing, run `staterm.sh` then `import.sh`. These commands will create/update the state on S3.
8. Replace the network list configurations/contents in the `terraform.tfvars` with what retrieved in step 4 via a feature branch. Merge the feature branch into the main branch that triggers the CD pipeline to complete the final config sync.  

### Property Pipeline

This pipeline consist of the following build steps:
1. Retrieve the configured Akamai API client from Vault
2. Retrieve and show property configurations
   1. Run Akamai CLI Terraform command to pull property configurations including the JSON rules in `property-snippets` and auto-generate Terraform code (`property.tf` and `variables.tf`) and `import.sh`: `akamai-terraform --edgerc $HOME/.edgerc --section default --version <production network active version> <akamai property name>`
   2. Run `import.sh` to create the local state
   3. Convert the state into JSON: `terraform show -json > ./terraform.tfstate.json`
   4. Use `jq -r` command to extract `property name`, `edge hostnames`, `hostnames`, `product id`, `rule format`, staging network active version
3. Retrieve AWS access key id and secret access key from Vault and load them into the env variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
4. Use the above AWS credentials to assume `PipelineRole` and update env variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`:
   ```
   aws sts assume-role \
      --role-arn arn:aws:iam::<aws account id>:role/PipelineRole \
      --role-session-name spmkeeper@gmail.com > authz.json
   
   export AWS_ACCESS_KEY_ID=$(cat authz.json | jq -r ".Credentials.AccessKeyId")
   export AWS_SECRET_ACCESS_KEY=$(cat authz.json | jq -r ".Credentials.SecretAccessKey")
   export AWS_SESSION_TOKEN=$(cat authz.json | jq -r ".Credentials.SessionToken")
   ```
5. Perform state onboarding or OOB sync
   1. Download property automation code from Artifactory
   2. Extract the code and remove `property-snippets`
   3. Run Akamai CLI Terraform command to pull property configurations including the JSON rules in `property-snippets` and auto-generate Terraform code (`property.tf` and `variables.tf`) and `import.sh`: `akamai-terraform --edgerc $HOME/.edgerc --section default --version <production network active version> <akamai property name>`
   4. Remove auto-generated `variables.tf`, rename auto-generated `import.sh`, `property.tf` to `import.sh.origin`, `property.tf.origin`
   5. Git clone the property self service repo and copy the Terraform automation code (generated in step 2) into this directory
   6. Extract the property id, contract id, group id, version no. from `import.sh.origin` to build new `import.sh` that has the command `terraform import akamai_property.property <property id,contract id,group id,version no.>`
   7. Run `terraform init` with the AWS S3 backend configurations. **Note**: To make this command work, the `S3BackendRole` needs to be configured to allow `PipelineRole` to assume it.
      ```
      terraform init -backend-config="bucket=<S3 bucket>" \
                     -backend-config="key=<terraform state file as S3 object>" \
                     -backend-config="encryption=true" \
                     -backend-config="region=<AWS region>" \
                     -backend-config="dynamodb_table=<AWS DynamoDB table>" \
                     -backend-config="kms_key_id=<AWS KMS key id>" \
                     -backend-config="role_arn=<arn:aws:iam::<aws account id>:role/S3BackendRole>" \
      ```
   8. Create `staterm.sh` that will be used for the OOB sync case
      1. Run `terraform state list > property_state_list.txt`
      2. Create in-line Python file to extract lines that starts with `akamai_edge_hostname.edge_hostnames` or `akamai_property.property` using regex
      3. create `staterm.sh` that has lines in the format `terraform state rm 'akamai_edge_hostname.edge_hostnames<>'` or `terraform state rm 'akamai_property.property<>'` to remove those resources
   9.  If the task is to perform the OOB sync, run `staterm.sh` and `import.sh`. Otherwise, only run `import.sh`
   10. Git clone the property self service repo into another directory, create an onboarding or OOB sync branch using git commands, copy the newly created `property-snippets` directory into this branch, extract the active version from state and replace version in `terraform.tfvars` with it, and finally perform `git add`, `git commit`, and `git push` to push updates to the new branch
6. Create a pull request from that branch to the main branch. The merge of this PR will trigger the pipeline to apply changes to Akamai networks.

## Self Service Portal Use Cases

### Akamai IP/Geo Firewall Self Service Portal

After the onboarding, the network list configurations are managed via `terraform.tfvars`. To make changes, a change maker needs to create a feature branch and modifies the content of `terraform.tfvars`. That triggers a CI pipeline that run `terraform plan` to show the planned changes. The merge of the pull request from the feature branch to the main branch triggers the CD/implementation pipeline that applies changes to Akamai networks.

### Akamai CDN ACL Self Service Portal

After the onboarding, the CDN ACL configurations are managed via JSON files in `property-snippets`. To make changes, a change maker needs to create a feature branch and modifies the content of the relevant JSON files. That triggers a CI pipeline that run `terraform plan` to show the planned changes. The merge of the pull request from the feature branch to the main branch triggers the CD/implementation pipeline that applies changes to Akamai networks.