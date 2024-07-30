# Fortigate autoscaling in AWS
This repository provides detailed information and insights on how to get started with **Terraform modules** that facilitate the implementation of **autoscaling FortiGate instances in an AWS Gateway Load Balancer (GWLB)** scenario.

At a very high level, the solution is divided into two main components:
- Autoscaling in AWS
- Implementing autoscaling on FortiGate

When you are familiar with the solution, you can get additional info [Getting Started](./documentation/Getting_started.md)<br>
There is also a workshop available [here](https://fortinetcloudcse.github.io/FortiGate-AWS-Autoscale-TEC-Workshop/)

## Solution Overview

The core of the solution involves two **AWS Auto Scaling Groups** (ASGs):
- One ASG for controlling the number of BYOL (Bring Your Own License) instances.
- One ASG for controlling the number of PAYG (Pay-As-You-Go) FortiGate instances.

An Auto Scaling Group (ASG) in Amazon Web Services (AWS) is a feature that ensures you have the correct number of Amazon EC2 instances available to handle the load for your application. ASGs automatically adjust the number of EC2 instances in response to changes in demand, based on predefined scaling policies. This helps maintain application availability and optimize costs.

Key components of an Auto Scaling Group include:
- **Launch Configuration or Launch Template:** Defines the instance configuration, such as the AMI, instance type, key pair, security groups, and block device mapping.
- **Scaling Policies:** Rules that determine when and how the ASG should scale in or out. These can be based on various metrics such as CPU utilization, network traffic, or custom CloudWatch metrics.
- **Desired Capacity:** The ideal number of instances the ASG should maintain. This value can be adjusted manually or automatically based on scaling policies.
- **Minimum and Maximum Size:** Defines the lower and upper limits on the number of instances in the group. The ASG will not scale below the minimum size or above the maximum size.
- **Health Checks:** Mechanisms to ensure that instances are running properly. ASGs can use EC2 status checks or Elastic Load Balancer (ELB) health checks to determine the health of instances.
- **Lifecycle Hooks:** Allow you to perform custom actions at different stages of the instance lifecycle, such as when instances are launching or terminating.

By using Auto Scaling Groups, you can ensure that your Fortigate deployment is resilient, scalable, and cost-effective, automatically responding to varying loads **without** manual intervention.<br>
Please note there are 2 ASG's, respectivily for BYOL and PAYG instance management.

**Lifecycle Hooks** in AWS Auto Scaling Groups enable you to perform custom actions at different stages of the instance lifecycle, such as during instance launch or termination. These hooks allow for tasks like configuring software, running scripts, or saving logs, ensuring a smoother and more controlled scaling process. In this deployment, a notifications of these events happening is picked up on the **AWS EventBridge**.

**AWS CloudWatch** continuously monitors various metrics such as CPU utilization, memory usage, disk I/O, and network traffic for your AWS resources. It provides real-time insights and triggers alarms when metrics breach predefined thresholds. These alarms can then initiate scale-in or scale-out operations within an Auto Scaling Group, automatically adjusting the number of EC2 instances to match the current demand. This ensures optimal performance and cost-efficiency by dynamically scaling your infrastructure in response to actual usage patterns.

### Autoscaling Process (scaling out)
1. Traffic conditions change 
2. CloudWatch alerts will trigger and instructs an ASG to scale out. 
3. While the FortiGate instance is started, **Cloud-init** configures it with a basic network setup, enabling access to the instance, as specified in the **Launch Configuration**.
4. **AWS EventBridge** rule monitors notifications indicating successful FortiGate EC2 instance launches (via **ASG Lifecycle hooks**) and triggers an **AWS Lambda function** to further configure the newly created FortiGate instance. A similar approach is followed when removing instances.

### Lambda Functions 
The Lambda functions (fgt_asg_launch_fgt_byol_asg and fgt_asg_launch_fgt_on_demand_asg) are responsible for configuring the FortiGate instance with the following:
- License file, FortiFlex or PAYG 
- Initial policy configuration
- Advanced network configuration (GENEVE tunnels)
- Configuration of system auto-scaling

### Auto-Scale Configuration
The `config system auto-scale` feature requires particular attention. To configure a newly created FortiGate instance, the Lambda function must identify the **PRIMARY** FortiGate and configure the new instance as a **SECONDARY**. The state is tracked in an AWS DynamoDB. If there is no PRIMARY instance, the new unit will assume this role.
The DynomaDB table is aware of all instances that are in use (license mgmt) as well as the current PRIMARY.

**AWS DynamoDB** is a fully managed NoSQL database service designed for fast and predictable performance with seamless scalability. It provides high availability and durability by automatically distributing data across multiple servers and regions. DynamoDB supports both document and key-value store models, making it flexible for a wide range of applications.