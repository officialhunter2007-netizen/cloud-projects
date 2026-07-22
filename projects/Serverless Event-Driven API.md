# Project 3: Serverless Event-Driven API (100% CLI Provisioned)

## ☁️ Project Overview
In this project, I architected and deployed a fully serverless, event-driven web API. To demonstrate advanced cloud proficiency and bypass local network firewall restrictions, **this entire infrastructure was provisioned, secured, tested, and destroyed exclusively using the AWS Command Line Interface (CLI) via AWS CloudShell**, bypassing the AWS Management Console entirely.

## 🏗️ Architecture Diagram
*(This diagram illustrates the Zero-Trust data flow from the public internet to the database).*


<img src="https://github.com/officialhunter2007-netizen/cloud-projects/blob/main/projects/Screenshot%202026-07-21%20081521.png" width="800" alt="Architecture Diagram">


## 🛠️ Technologies & Concepts Mastered
* **AWS CloudShell & Bash Scripting:** Used native Linux commands (`cat`, `zip`, variables) to automate the deployment of all resources.
* **Amazon API Gateway (v2):** Configured a public-facing HTTP API endpoint.
* **AWS Lambda (Python 3.12):** Wrote serverless compute logic using the `boto3` SDK.
* **Amazon DynamoDB:** Designed a NoSQL database table utilizing Partition Keys (`MessageId`).
* **AWS IAM (Zero-Trust Security):** Forged strict Execution Roles and Resource-Based Policies to prevent the "Confused Deputy" vulnerability.

## 📝 Step-by-Step Implementation

1. **Database Provisioning:** Created a DynamoDB table (`ContactMessages`) via CLI, establishing a NoSQL foundation for flexible data ingestion.
2. **Security Configuration:** Forged an IAM Execution Role with the Principle of Least Privilege, explicitly granting Lambda permission to write to DynamoDB and CloudWatch.
3. **Serverless Compute:** Wrote a Python script utilizing `boto3` to parse incoming JSON, generate a unique timestamp ID, and execute a `PutItem` API call. Used Bash `cat << 'EOF'` to generate the file, zipped it, and deployed it via `aws lambda update-function-code`.
4. **API Gateway Routing:** Provisioned an HTTP API Gateway and dynamically routed it to the Lambda function's ARN using the `--target` flag for rapid integration.
5. **Resource-Based Policy:** Locked the Lambda function's trigger permissions to explicitly allow *only* the specific API Gateway ID within my AWS Account ID, securing the endpoint.
6. **CLI Testing:** Executed an HTTP POST request across the public internet using `curl`, and verified the data ingestion using a DynamoDB `scan` command.
7. **Automated Teardown:** Wrote and executed a "Doomsday Script" to systematically detach IAM policies, delete the role, destroy the API Gateway, terminate the Lambda function, and drop the DynamoDB table to ensure a $0.00 billing footprint.

## 🧠 Deep Architectural Challenges & Solutions

### 1. Bypassing Local Firewalls with AWS CloudShell
**The Problem:** While attempting to create the Lambda function via the AWS Console, my local network/antivirus blocked the AWS CDN from downloading the default code template, resulting in a "Failed to fetch" error.
**The Solution:** Instead of relying on the GUI, I pivoted to **AWS CloudShell**. By utilizing the AWS CLI directly within the AWS network matrix, I completely bypassed my local firewall and forced the creation of the infrastructure using pure Bash scripting.

### 2. Defensive Programming & The "Missing Body" Error
**The Problem:** During initial testing, a web browser accidentally sent an HTTP `GET` request instead of a `POST` request. Because `GET` requests have no payload, the Python script threw a `KeyError: 'body'` and crashed. 
**The Solution:** I implemented Defensive Programming in the Lambda function. I wrote logic to check `if 'body' not in event:`, allowing the API to gracefully return a `400 Bad Request` and print the raw event ticket for debugging, rather than suffering a fatal `500 Internal Server Error`.

### 3. DynamoDB Reserved Word Collisions & NoSQL Injection
**The Problem:** When attempting to query the database via the CLI to find a specific user's name, the command failed because `Name` is an AWS Reserved Word.
**The Solution:** I learned to strictly separate logic from data. I utilized Expression Attribute Names (`#nameKey`) and Expression Attribute Values (`:nameValue`) to safely alias the reserved word. This not only fixed the error but acts as a best practice to prevent NoSQL injection attacks.

### 4. The Confused Deputy Problem
**The Problem:** Simply creating an API Gateway does not give it permission to trigger Lambda due to AWS's Zero-Trust architecture. 
**The Solution:** I wrote a strict Resource-Based Policy using `aws lambda add-permission`. I explicitly defined the `--source-arn` using my exact AWS Account ID and Region to ensure no external or malicious API Gateways could trigger my compute resources.

## 🔬 Advanced Database Mechanics Demonstrated
* **Scan vs. Query:** I utilized the `scan` command for this small dataset, but documented the architectural need for **Global Secondary Indexes (GSIs)** and the `query` command for cost-effective data retrieval at an enterprise scale.
* **Targeted Deletion (The Sniper Rifle):** Unlike SQL databases that allow mass deletions via filters, I demonstrated DynamoDB's strict deletion mechanics by using `aws dynamodb delete-item`, requiring the exact `MessageId` (Partition Key) to surgically remove a single record, preventing catastrophic accidental data loss.

## 📸 Final Result
*(Here is the successful terminal output proving the API Gateway and Lambda function successfully processed the HTTP POST request and saved the data).*

<img src="https://github.com/officialhunter2007-netizen/cloud-projects/blob/main/projects/Screenshot%202026-07-22%20092244.png" width="800" alt="Architecture Diagram">
