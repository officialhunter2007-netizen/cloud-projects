# Project 1: Hosting a Static Website on AWS S3

## ☁️ Project Overview
In this project, I successfully transformed an Amazon S3 bucket into a static web server. I configured the bucket to host HTML files and applied custom security policies to make the website accessible to the public over the internet.

## 🏗️ Architecture Diagram
*(This diagram shows how the user connects to the AWS S3 bucket over the internet to request the website files).*

![Architecture Diagram](Project1-Diagram.png)

## 🛠️ Technologies Used
* **Amazon S3 (Simple Storage Service):** Used to store the website files and act as the web server.
* **AWS IAM (Identity and Access Management):** Used to write the Bucket Policy (JSON) to securely grant public read access.
* **HTML/CSS:** Used to write the actual website code.

## 📝 Step-by-Step Implementation

1. **Created an S3 Bucket:** Created a globally unique S3 bucket and disabled the "Block All Public Access" security feature to prepare it for web hosting.
2. **Uploaded Website Files:** Uploaded a custom `index.html` file containing the website's code.
3. **Enabled Static Website Hosting:** Configured the S3 bucket properties to act as a web server and set `index.html` as the default routing page.
4. **Configured Security Permissions:** Wrote and applied a JSON Bucket Policy to explicitly allow `s3:GetObject` actions, granting anyone on the internet permission to view the website while protecting the files from being modified or deleted.

## 🧠 Challenges Faced and Lessons Learned
**The "Block Public Access" Master Switch:** 
When I first tried to apply the JSON Bucket Policy to make the website public, AWS denied the action. I learned that AWS has a master security switch called "Block Public Access" that overrides all bucket policies. I had to explicitly turn off this master switch before AWS would accept my custom JSON policy. This taught me how AWS prioritizes security at multiple layers.

## 📸 Final Result
Here is a screenshot of the live website successfully running on the AWS S3 endpoint:

![Live Website](Project1-LiveWebsite.png)
