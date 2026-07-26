# Project 5: Enterprise-Grade Multi-AZ Architecture (Provisioned via Terraform)

## 🚀 Project Overview
In this project, I architected and deployed a highly available, fault-tolerant enterprise environment on AWS using **Infrastructure as Code (IaC)**. Moving away from manual provisioning, I utilized **Terraform** to declare a 3-tier-ready network across multiple Availability Zones, ensuring that the application can survive the total failure of a data center while maintaining zero-trust security boundaries.

## 📋 Technical Architecture

```mermaid
flowchart TB
    %% Styling for Professional Look
    classDef internet fill:#f3f4f6,stroke:#333,stroke-width:2px;
    classDef vpc fill:#e0f2fe,stroke:#2563eb,stroke-width:3px,stroke-dasharray: 5 5;
    classDef public fill:#dcfce7,stroke:#16a34a,stroke-width:2px;
    classDef private fill:#fee2e2,stroke:#dc2626,stroke-width:2px;
    classDef data fill:#fef3c7,stroke:#d97706,stroke-width:2px;

    %% External Entities
    User(("👤 End User (Customer)")):::internet
    GlobalInternet(("🌍 Public Internet (Updates/APIs)")):::internet

    %% The VPC Boundary
    subgraph VPC ["☁️ Custom Enterprise VPC (CIDR: 10.0.0.0/16)"]
        IGW["🚪 Internet Gateway (IGW)  
(1-to-1 NAT / The Door in the Wall)"]

        %% Availability Zone A
        subgraph AZ_A ["🏢 Availability Zone A (us-east-1a)"]
            
            subgraph Pub_A ["🟢 Tier 1: Public Subnet A (10.0.1.0/24)"]
                RT_Pub_A[/"🗺️ Public Route Table  
(0.0.0.0/0 ➔ IGW)"/]
                ALB_A{"⚖️ ALB Node A  
(Public IP)"}
                NAT_A["🔄 NAT Gateway A  
(Public IP / PAT)"]
            end

            subgraph Priv_A ["🔴 Tier 2: Private Compute Subnet A (10.0.2.0/24)"]
                RT_Priv_A[/"🗺️ Private Route Table A  
(0.0.0.0/0 ➔ NAT A)"/]
                ASG_A["⚙️ Auto Scaling Group"]
                EC2_A["🖥️ Web Server A  
(Private IP Only)"]
            end

            subgraph Data_A ["🟤 Tier 3: Private Data Subnet A (10.0.3.0/24)"]
                DB_Primary[("🗄️ Primary Database  
(Active)")]
            end
        end

        %% Availability Zone B (Disaster Recovery)
        subgraph AZ_B ["🏢 Availability Zone B (us-east-1b)"]
            
            subgraph Pub_B ["🟢 Tier 1: Public Subnet B (10.0.4.0/24)"]
                RT_Pub_B[/"🗺️ Public Route Table  
(0.0.0.0/0 ➔ IGW)"/]
                ALB_B{"⚖️ ALB Node B  
(Public IP)"}
                NAT_B["🔄 NAT Gateway B  
(Public IP / PAT)"]
            end

            subgraph Priv_B ["🔴 Tier 2: Private Compute Subnet B (10.0.5.0/24)"]
                RT_Priv_B[/"🗺️ Private Route Table B  
(0.0.0.0/0 ➔ NAT B)"/]
                ASG_B["⚙️ Auto Scaling Group"]
                EC2_B["🖥️ Web Server B  
(Private IP Only)"]
            end

            subgraph Data_B ["🟤 Tier 3: Private Data Subnet B (10.0.6.0/24)"]
                DB_Standby[("🗄️ Standby Database  
(Passive Replica)")]
            end
        end
    end

    %% ==========================================
    %% 1. INBOUND TRAFFIC FLOW (Solid Lines)
    %% ==========================================
    User == "1. HTTP Request" ==> IGW
    IGW == "2. Routes to Load Balancer" ==> ALB_A & ALB_B
    ALB_A == "3. Distributes Traffic" ==> EC2_A & EC2_B
    ALB_B == "3. Distributes Traffic" ==> EC2_A & EC2_B
    
    %% Internal Database Flow
    EC2_A == "4. Reads/Writes Data" ==> DB_Primary
    EC2_B == "4. Reads/Writes Data" ==> DB_Primary
    DB_Primary -. "5. Synchronous Replication" .-> DB_Standby

    %% ==========================================
    %% 2. OUTBOUND TRAFFIC FLOW (Dotted Lines)
    %% ==========================================
    EC2_A -. "A. Needs Update" .-> RT_Priv_A
    RT_Priv_A -. "B. GPS points to NAT" .-> NAT_A
    NAT_A -. "C. Translates IP" .-> RT_Pub_A
    RT_Pub_A -. "D. GPS points to IGW" .-> IGW
    
    EC2_B -. "A. Needs Update" .-> RT_Priv_B
    RT_Priv_B -. "B. GPS points to NAT" .-> NAT_B
    NAT_B -. "C. Translates IP" .-> RT_Pub_B
    RT_Pub_B -. "D. GPS points to IGW" .-> IGW

    IGW -. "E. Fetches Update" .-> GlobalInternet

```
The architecture follows the **Well-Architected Framework** by separating concerns into distinct layers:
*   **Public Tier:** Hosts the Application Load Balancer (ALB) and NAT Gateways.
*   **Private Tier:** Hosts the Web Server fleet, completely isolated from direct internet access.
*   **Data Tier (Designated):** Pre-configured subnets for future database isolation.

## 🛠️ Technologies & Core Concepts
*   **Terraform (HCL):** Used for declarative infrastructure management, state tracking, and resource lifecycle automation.
*   **High Availability (HA):** Deployed across `us-east-1a` and `us-east-1b` to eliminate single points of failure.
*   **Security Group Chaining:** Implemented a "Zero-Trust" model where Web Servers only accept traffic originating from the ALB's specific Security Group.
*   **Elastic Load Balancing (ALB):** Manages traffic distribution and health checks across the server fleet.
*   **Auto Scaling Group (ASG):** Ensures the fleet automatically heals and scales based on demand.
*   **NAT Gateways:** Provides secure, one-way outbound internet access for private instances (for updates/patches) without exposing them to inbound threats.

## 🚀 Implementation Deep-Dive

### 1. Networking Foundation (`network.tf`)
I bypassed the AWS Default VPC to build a custom CIDR-blocked environment. I provisioned:
*   **Internet Gateway (IGW)** for public ingress.
*   **Dual NAT Gateways** (one per AZ) to ensure that even if one zone fails, the other maintains outbound connectivity.
*   **Route Table Logic:** Configured specific routing rules to ensure private traffic is masked behind the NAT Gateways while public traffic is routed to the IGW.

### 2. Security Architecture (`security.tf`)
Instead of using open ports, I implemented **Identity-Based Security Groups**:
*   **ALB Bouncer:** Accepts only HTTP (Port 80) from the public internet.
*   **Web Bouncer:** Rejects all traffic unless it originates from the ALB Bouncer's ID, preventing "Side-Channel" attacks.

### 3. Compute & Scaling (`compute.tf`)
I utilized **Launch Templates** to define the server's "DNA" (Amazon Linux 2023, Apache, and automated bootstrap scripts). The **Auto Scaling Group** was then configured to maintain a minimum of 2 instances across different zones, providing 99.99% availability.

## 🧠 Strategic Challenges & Solutions

### Challenge: The "Hidden Dependency" Violation
**The Problem:** During the iterative testing phase, I encountered a `DependencyViolation` when attempting to destroy the infrastructure. The Security Group refused to delete because the Elastic Network Interface (ENI) from the Fargate/EC2 instances was still detaching.
**The Solution:** I analyzed the resource lifecycle and implemented a strategic wait-state in the automation logic, ensuring that network interfaces are fully released before the security layer is dismantled.

### Challenge: State Consistency in Local Environments
**The Problem:** Managing infrastructure from a local VPS introduced risks of state drift.
**The Solution:** I mastered the use of `terraform plan` to perform dry-run inspections before every deployment, ensuring that the "Desired State" in my code perfectly matched the "Actual State" in AWS.

## 📈 Business Impact
By migrating to this Terraform-based architecture, a business reduces its **Recovery Time Objective (RTO)** to near zero. The automated nature of the deployment eliminates human error, reduces configuration drift, and allows for the entire global infrastructure to be replicated in minutes rather than days.

<img src="https://github.com/officialhunter2007-netizen/cloud-projects/blob/main/projects/Screenshot%202026-07-24%20153426.png" width="800" alt="Architecture Diagram">
---
<img src="https://github.com/officialhunter2007-netizen/cloud-projects/blob/main/projects/Screenshot%202026-07-24%20162820.png" width="800" alt="Architecture Diagram">
