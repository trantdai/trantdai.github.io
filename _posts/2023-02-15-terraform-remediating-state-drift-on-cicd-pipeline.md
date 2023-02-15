---
title: Terraform - Remediating State Drift on TeamCity CI/CD Pipeline
layout: post
post-image: "/assets/images/blog/terraform_detect_manage_state_drift.png"
description: How to remediate Terraform state drift on TeamCity pipeline
tags:
- howto
- blog
- tips
- Terraform
- State synchronization
- devops
- akamai
---

Today, when I tried to build a TeamCity pipeline in an effort to automate the Akamai Terraform state drift remediation. The pipeline is designed to work in as follows:
1. It contructs all the `terraform import` commands to import existing Akamai network lists into a Terraform state. This is part of the onboarding process for the network list to be managed via Terraform.
2. It runs step 1 again to sync the Terraform state with the out of band changes introduced via Akamai Control Center (Web GUI)

However, step 2 failed and threw the following error:
```
Error: Resource already managed by Terraform

Terraform is already managing a remote object for akamai_networklist_network_list.network_lists["<Network List Name>"]. To import this address you must first remove the existing object from the state.
```

I found the article [Detecting and Remediating State Drift in Terraform](https://aqibrahman.com/detecting-and-remediating-state-drift-in-terraform) that explains clearly how the `terraform state rm` helps resolve this error.
