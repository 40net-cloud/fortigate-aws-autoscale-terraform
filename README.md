# Fortigate autoscaling in AWS
This repository provides detailed information and insights on how to get started with Terraform modules that facilitate the implementation of autoscaling FortiGate instances in an AWS Gateway Load Balancer (GWLB) scenario.

The solution is divided into two main components:
- Autoscaling in AWS
- Implementing autoscaling on FortiGate

## Solution Overview

The core of the solution involves two **AWS Auto Scaling Groups** (ASGs):
- One ASG for controlling the number of BYOL (Bring Your Own License) instances.
- One ASG for controlling the number of PAYG (Pay-As-You-Go) FortiGate instances.

**AWS CloudWatch** continuously monitors resource utilization and triggers scale-in or scale-out operations based on predefined thresholds.

## Autoscaling Process
When a FortiGate instance creation is needed, CloudWatch instructs the ASG to scale out. 
Once the FortiGate instance is started, **Cloud-init** configures it with a basic network setup, enabling access to the instance.

**AWS EventBridge** monitors notifications indicating successful FortiGate EC2 instance launches (via **ASG Lifecycle hooks**) and triggers an **AWS Lambda function** to further configure the newly created FortiGate instance. 
A similar approach is followed when removing instances.

## Lambda Function 
The Lambda functions (fgt_asg_launch_fgt_byol_asg and fgt_asg_launch_fgt_on_demand_asg) are responsible for configuring the FortiGate instance with the following:
- License file or FortiFlex (or PAYG)
- Initial policy configuration
- Advanced network configuration (GENEVE tunnels)
- Configuration of system auto-scaling

## Auto-Scale Configuration
The `config system auto-scale` feature requires particular attention. To configure a newly created FortiGate instance, the Lambda function must identify the **PRIMARY** FortiGate and configure the new instance as a **SECONDARY**. The state is tracked in an AWS DynamoDB. If there is no PRIMARY instance, the new unit will assume this role.
