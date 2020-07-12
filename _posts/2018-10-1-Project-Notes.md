---
layout: post
title: Independent Project - PredictedIt - Project Notes
tags: predictedit notes
excerpt_separator: <!--more-->
---

## Project Overview
I have a personal interest in politics as well as economics. I find I am
especially intrigued by the idea of markets to predict outcomes, and in the
past I've played around with placing political bets on the website PredictIt.org.

I would love to be able to practice my data analysis skills on the PredictIt
market data, and even learn a bit about quantitative trading without wading
into the rats nest that is the stock market. Unfortunately, the PredictIt API is pretty
minimal, so for this project I am setting out to archive data from PredictIt using the cheapest, abstracted-AWS services I can find; structure the data and make it accessible to others in a cheap,
but scalable, microservices architecture; Analyze the data myself and potentially practice
applying some quant-trading algorithms along the way.

<!--more-->

## Project Goals
* Learn how to build and release an AWS-based architecture
* Practice data acquisition, cleaning, and structuring
* Practice data analysis
* Learn a bit about common quantitative training techniques
* Archive PredictIt pricing data and make it easily accessible for others

## Requirements
* Minimize costs
* Limit cloud resources to AWS because:
  + We are an AWS shop at work, so this aids professional development
  + I am planning to take the AWS Certified Developer - Associate Certification Exam
* Make the data I archive easily accessible and ensure access is scalable
* Follow data fault-tolerance and security-best practices

## POC Architecture:

TODO(nadir.sidi): Add a lucid-chart diagram here

## ToDo:

### Data Acquisition
* Review the existing PredictIt API endpoints using Postman
* Write a Python Lambda function to hit the API and stash the data
* Set-Up a deployment pipeline
  + Set-up AWS CodeBuild with hooks to Github repo with Lambda
  + Learn about CloudWatch SAM files to handle deployment from S3 artifact

### Data Storage
* Determine initial partition-scheme for stashing PredictIt data
* Investigate transforming the data to store into a DB
* Write a Athena query to read data directly from S3
  + Is is bad practice to write a API that hits the Athena schema-on-read directly?

### Data Access
* Investigate querying data from Athenta

{% include archive.html %}
