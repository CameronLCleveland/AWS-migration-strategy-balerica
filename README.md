# Balerica Inc. Cloud Migration Strategy

**Author:** Cameron Cleveland
**Date:** August 29, 2025
**Project:** Individual Armageddon - Cloud Migration Strategy

## Project Overview

Balerica Inc., an educational services provider headquartered in Sao Paulo, sought to modernize its IT infrastructure, expand globally, and enhance security. This project outlines a comprehensive migration strategy from their existing on-premises setup to a secure, automated, and scalable environment on **Amazon Web Services (AWS)**.

### Goals
- Automate deployment and management processes.
- Reduce or eliminate "click-ops" administration.
- Enable secure remote control capabilities for administrators.
- Drastically reduce desktop reimaging times.
- Develop a scalable, secure browser for certification tests.
- Establish a secure, redundant global network connecting testing centers in 5 countries.

## Current State Analysis

The following diagram illustrates Balerica's existing on-premises network topology, which is manual, fragile, and not built for scale.

![Balerica Current Network Topology](AWS-Before.jpeg)

### Diagram Explanation & Key Issues:
*   **Flat Network:** All devices reside on a single network segment connected via **unmanaged TP-Link switches**, creating a major security risk where a breach on one device could compromise all assets.
*   **Manual Processes:** The workflow is characterized by **"click-ops"**—manual administration via CMD scripts, best-effort backups, and manual deployment of the secure exam browser.
*   **Lack of Central Management:** There is no central codebase or configuration management. **International testing centers** have no standardized, secure connection to the Sao Paulo HQ, relying on insecure, ad-hoc links.
*   **Desktop Management Overhead:** Reimaging the 30+ Lenovo M90 desktops is a slow, manual process using the factory Lenovo image.

## Future State Recommendation: AWS Architecture

The proposed future state leverages Amazon Web Services to create a resilient, automated, and globally scalable infrastructure that addresses all of Balerica's expressed goals.

![Balerica Future State on AWS](AWS-After.jpeg)

### Diagram Explanation & Networking Choices:

This architecture is built on a **global hub-and-spoke model** using AWS's enterprise-grade services.

**1. Global Network & Security:**
*   **AWS Transit Gateway:** This acts as the central cloud router, simplifying connectivity between all regional VPCs and on-premises locations. It provides a scalable and managed hub for a secure global network.
*   **Hybrid Connectivity:** The São Paulo HQ uses **AWS Direct Connect** for a high-bandwidth, low-latency dedicated network connection, with a backup **AWS Site-to-Site VPN** for failover. Global testing centers connect via Site-to-Site VPN to their nearest AWS region. This meets the requirement for **secure and redundant communications**.
*   **Zero Trust Security:** **AWS Systems Manager Session Manager** is implemented for all remote administration. This ensures that **no EC2 instances require open inbound ports (SSH/RDP)**, as access is brokered through AWS's secure APIs. This fulfills the goal of "secure remote control capabilities for administrators only."

**2. Automation & Modernization:**
*   **Amazon ECS (Elastic Container Service):** The homegrown secure browser is **containerized** and runs on this fully managed container orchestration service. It automatically handles deployment, scaling, and management of the browser backend, making it highly scalable and reliable.
*   **Infrastructure as Code (IaC):** The entire environment can be provisioned using AWS CloudFormation or Terraform, eliminating manual "click-ops" and ensuring consistent, repeatable deployments.
*   **Amazon WorkSpaces:** The existing Lenovo desktops are used as thin clients to access cloud-hosted virtual desktops. A single "golden image" of the secure browser environment is maintained, allowing for instant, centralized deployment and "reimaging" in seconds by simply rebooting the virtual desktop.

**3. Resilience and Performance:**
*   **Application Load Balancer:** Users worldwide connect to a secure endpoint provided by the ALB, which automatically distributes exam traffic across the healthy container instances in multiple Availability Zones, ensuring high availability and performance.
*   **Managed Services:** Using **Amazon RDS** (Relational Database Service) and **Amazon EFS** (Elastic File System) ensures databases and shared storage are highly available, scalable, and automatically backed up, moving away from the on-premises "best effort" policy.

## Task 2: Pain Point Analysis & Solutions

### **1. Pain Point: Single Points of Failure and Lack of Redundancy**

The most critical pain point is the environment's fragility due to multiple single points of failure, particularly the standalone database server and the lack of redundant network paths. The current configuration means a hardware failure in the São Paulo data center would cause complete, prolonged downtime for all global testing centers. This directly threatens business continuity, leading to canceled exams, lost revenue, and significant damage to Balerica's reputation as a reliable testing provider. The best-effort backup strategy further exacerbates this risk, potentially leading to data loss.

Our solution is to architect for high availability on AWS. We will migrate the database to **Amazon RDS configured for Multi-AZ deployment**, which automatically provisions a synchronous standby replica in a separate Availability Zone, enabling automatic failover in the event of an outage. For connectivity, each global testing center will establish **AWS Site-to-Site VPN connections** to their nearest AWS region, while the HQ will use **AWS Direct Connect** with a backup VPN for resilient, high-speed access. This transformation eliminates critical single points of failure, reducing potential downtime from hours to seconds. The primary drawback is a predictable increase in operational costs due to the doubled database instance and data transfer charges. However, this is a net positive; the business cost of downtime far outweighs these cloud expenditures, directly achieving the goal of secure and redundant communications.

### **2. Pain Point: Manual, Inefficient Desktop Management ("Click-Ops")**

The second critical pain point is the operational inefficiency and inconsistency of desktop management. The current process of manually reimaging each Lenovo desktop with a factory image via CMD scripts is incredibly time-consuming for IT staff. This "click-ops" approach leads to long resolution times for testing centers, creates non-standardized environments that are difficult to secure and troubleshoot, and directly contributes to examiner and candidate frustration during technical issues.

Our solution is to modernize the endpoint strategy by leveraging Amazon WorkSpaces. The existing Lenovo hardware will be repurposed as thin clients to access cloud-hosted **Amazon WorkSpaces** virtual desktops. IT administrators will maintain a single, centralized "golden image" containing the secure browser configuration. "Reimaging" a problematic desktop is reduced from a manual, multi-hour process to a simple reboot of the virtual desktop, which takes seconds and restores a pristine, compliant state. The potential drawbacks include a dependency on stable internet connectivity at each testing center and the ongoing subscription cost of WorkSpaces. Despite this, the solution is a definitive net positive; it directly and completely accomplishes the goal of drastically reducing desktop reimaging times and occurrences.

### **3. Pain Point: Insecure Administrative Access and Flat Network**

The third pressing pain point is the severe security vulnerability created by the flat network architecture and insecure administrative practices. The unmanaged switches allow unfettered lateral movement, meaning a compromise of one device could lead to a full network breach. Furthermore, current remote administration methods likely rely on insecure protocols like RDP with public-facing ports, creating a direct attack vector for threat actors targeting Balerica's critical infrastructure.

Our solution is to implement a defense-in-depth security model within AWS. We will replace the flat network with a **segmented VPC architecture** using security groups and network ACLs to enforce strict traffic controls between subnets. Most critically, we will eliminate all public management ports by leveraging **AWS Systems Manager Session Manager** for all remote administrative access to EC2 instances. Session Manager provides secure, auditable, and port-free access, gating connections behind IAM permissions. The primary drawback is the need for administrators to adopt a new workflow using the AWS Console or CLI instead of traditional RDP/SSH clients. This is a clear net positive; it is the most secure method to achieve the goal of "remote control capabilities on desktops from administrators only" and fundamentally transforms the security posture from vulnerable to robust.

## Conclusion

This migration strategy transforms Balerica Inc. from a manually-intensive, geographically-limited operation into a fully automated, secure, and global organization capable of scaling effortlessly to meet its business goals. The choice of **Amazon Web Services** provides a robust, secure, and comprehensive foundation for this transformation. By directly addressing the three most critical pain points with targeted AWS solutions, we ensure the migration not only meets but exceeds Balerica's stated goals for security, efficiency, and reliability.
