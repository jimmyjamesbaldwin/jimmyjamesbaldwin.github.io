---
layout: post
title: "Project: InvoiceMe"
categories: [general, project]
tags: [general, project, IAC, terraform, lambda, serverless]
fullview: true
---

_A cloud-native app to automatically generate invoices from the [Harvest](https://www.getharvest.com/) timesheet application and notify via Slack._
 
#### Why
Creating invoices can be tricky. It requires:
 * Remembering to create and send the invoice in the first place
 * Manually looking at the number of days worked on a timesheet application (Harvest)
 * Ensure the information is accurate (prone to human error)
 * etc.

#### Enter InvoiceMe
InvoiceMe is a python lambda function that combines static invoice data (a mix of config files and state in a dynamodb table) with the dynamic timesheet content from Harvest to produce an invoice. The lambda builds a json payload for the invoice and makes use of [invoiced](https://invoice-generator.com/#/1) to generate the PDF. This is then uploaded to S3 and a slack notification with a signed URL to the file is sent to the user. The app uses the following AWS services:
* Lambda - for running serverless functions
* DynamoDB - for keeping a tiny bit of state used to generate invoices
* Cloudwatch - for scheduling the lambda to run automatically
* IAM - for permissions
* S3 - for long term invoice storage
* Secrets Manager - for storing api keys

#### Deploying
I've supplied a bunch of terraform templates to simplify the deployment of the app. All you have to do is:
1. Edit `secretsmanager.tf` and enter your Harvest API token & Slack webhook
2. Edit `lambda.tf` and enter your Harvest account ID in the environment variables
2. Run the following commands:
```
./terraform init
./terraform apply
```
*(note: the `init` is important as I use the `archive` provider to zip the lambda src on the fly, so it'll fail if you don't have the provider setup).*

Once deployed, when the lambda runs you'll recieve a slack notification like so:
![](https://i.imgur.com/WJ1AXwP.jpg)
...which links you to your shiny new professional-looking PDF:

![screenshot of the produced pdf report](https://i.imgur.com/5OSKJkG.jpg)
##### Considerations
* The code could probably do with being split up and just made a bit nicer in general. The scope of the app changed as I was going along so the code could be cleaned up. Error handling is lacking...
