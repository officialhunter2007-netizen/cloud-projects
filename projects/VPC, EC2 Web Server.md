# Project 2: Building a Custom VPC and Deploying an EC2 Web Server

## ☁️ Project Overview
In this project, I designed and built a custom cloud network from scratch. Instead of relying on default AWS settings, I manually provisioned a Virtual Private Cloud (VPC), configured public subnets, and established internet connectivity using an Internet Gateway and custom Route Tables. Finally, I deployed an EC2 instance into this custom network and used a bash script (User Data) to automatically bootstrap an Apache web server.

## 🏗️ Architecture Diagram
*(This diagram illustrates the flow of internet traffic through the Internet Gateway, routed via the Route Table, into the Public Subnet where the EC2 instance resides).*

<img width="1068" height="539" alt="Screenshot 2026-07-20 151334" src="https://github.com/user-attachments/assets/e3cb68c9-1faa-4cde-a854-9f538933726d" />


## 🛠️ Technologies Used
* **Amazon VPC:** Created an isolated network environment (`10.0.0.0/16`).
* **Subnets & Route Tables:** Segmented the network (`10.0.1.0/24`) and directed traffic flow.
* **Internet Gateway (IGW):** Allowed communication between the VPC and the internet.
* **Amazon EC2:** Provisioned a Linux virtual server (`t2.micro`).
* **Security Groups:** Configured a virtual firewall to allow inbound HTTP (Port 80) traffic.
* **Bash Scripting:** Automated the installation and configuration of the Apache web server.

## 📝 Step-by-Step Implementation

1. **Network Foundation:** Created a custom VPC and a Public Subnet.
2. **Internet Connectivity:** Provisioned an Internet Gateway, attached it to the VPC, and updated the Route Table to direct `0.0.0.0/0` traffic to the IGW.
3. **Security Configuration:** Created a Security Group (virtual firewall) to explicitly allow inbound HTTP traffic from anywhere, ensuring the web server would be publicly accessible.
4. **Server Deployment:** Launched an Amazon Linux 2023 EC2 instance into the custom Public Subnet.
5. **Automation:** Injected a bash script into the EC2 User Data to automatically update the OS, install Apache (`httpd` ), start the service, and generate a custom `index.html` file upon boot.


<img width="1366" height="722" alt="Screenshot 2026-07-20 200522" src="https://github.com/user-attachments/assets/176c9ee8-9308-4bde-9148-300be684c84f" />

## 🧠 Challenges Faced and Lessons Learned
**Understanding HTTP vs HTTPS:** 
When initially testing the web server, the browser automatically attempted to connect via HTTPS (Port 443), which resulted in a timeout. I realized that my Security Group was only configured to allow HTTP (Port 80) traffic. By explicitly typing `http://` in the browser, the connection succeeded. This reinforced my understanding of how Security Groups act as strict gatekeepers at the port level, and how browsers handle default protocols.

## 📸 Final Result
Here is the live Apache web server successfully running from inside the custom VPC:

<img width="1364" height="725" alt="Screenshot 2026-07-20 200357" src="https://github.com/user-attachments/assets/ba529e8d-42c9-48d0-b8ef-b29754b2ef6b" />

