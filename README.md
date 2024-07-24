# Fortigate autoscaling in AWS
This repo contains additional information and insights in how to get started with the **terraform modules** that can help implements autoscaling fortigates in a AWS GWLB scenario.

There are two parts when analysing the solution.
- The part that handles autoscaling in AWS
- Logic to implement autoscaling on Fortigate

  
The core of the solution, are **two AWS autoscaling groups (ASG)**, one to control respectivily the number of BYOL instances and one ASG for PAYG Fortigate instances.
**Cloudwatch** will continously monitor the resource utilsation and trigger scale in/out operations based on predefined tresholds.

As explained, the creation of a FGT instance is initiated by Cloudwatch instructing the ASG to scale out.
When the FGT instance is started, **cloud-init** is used to configure the newly instantiated FGT with a very basic network configuration configuration allowing access to the instance.

**AWS EventBridge** looks out for notifications indicating successfull FGT EC2 instance launches (via **ASG Lifecycle hooks**) and triggers an **AWS LAMBDA function** to configure the newly created FGT.
A similar approach is folowed when removing an instances.

The purpose of the Lambda function (fgt_asg_launch_fgt_byol_asg and fgt_asg_launch_fgt_on_demand_asg) is to configure the FGT with
- license file or Fortiflex (or PAYG)
- initial policy config
- the advanced network configuration (GENEVE tunnels)
- config system auto-scale

`config system auto-scale` requires particular attention. In order to configure a newly created FGT, the LAMBDA function needs to determine the **PRIMARY FGT** and configure it as a **SECONDARY** accordingly. The state is tracked in an **AWS Dynamo Database**. If there is no PRIMAY, the unit will assume this ROLE.






