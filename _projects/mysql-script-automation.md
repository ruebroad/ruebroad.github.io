---
layout: page
title: "Automating RDS Aurora/MySQL scripts"
categories: test
---

## Overview

The current workflow for running SQL scripts against prod and pre-prod RDS databases involves connecting to a bastion host and using MySQL Workbench to run scripts manually. The ServiceNow change control process ensures there's a valid and approved CR before running the script. Approval implies that someone has validated the script to ensure it's correct and won't cause issues. In theory.

Scripts are also linked to a Jira ticket describing why the script needs to be run.

The goal of this project was to provide a way to automate the workflow, reduce time taken from receiving the request to running the request and automatically updating both Jira and ServiceNow with the result. We also wanted to leverage Pull Requests to ensure that the appropriate engineers had viewed, validated and approved the script before it was run.

To make things a little more interesting, we need to ensure that scripts are run on the appropriate customer RDS instance - enforcing separation of code and credentials to mitigate the risk of running a script on the wrong instance.

## The New Workflow

### Stage 1

* Create a Jira ticket
* Create a feature branch in the script automation repository
* Push the script to the new FB and create a pull request.
* On PR creation run a Bitbucket pipeline to:
  * Create a new ServiceNow Change Request using SN api with all relevant info and the script file
  * Update the Jira ticket with the new SN CR number
  * Create an SSM parameter to store the SN id (we'll need it later)
  * Notify Slack at each stage
* On approval an merge:
  * Upload the script to S3

### Stage 2

On upload to S3:

* CloudWatch event triggers Lambda function (python)
* Python script builds the connection to the correct RDS database
* Runs SQL script
* Updates Jira with the result
* Update ServiceNow CR with result and sets ticket status to PIR (require post implementation review)
  * To be completed

## Links

* [Stage 1 - Bitbucket pipeline](https://github.com/ruebroad/mysql-scripts)
* [Stage 2 - Lambda function & Terraform](https://github.com/ruebroad/tfm-mysql-scripts-auto)
