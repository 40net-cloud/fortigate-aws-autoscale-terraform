# Fortigate autoscaling in AWS
This repo contains additional information and insights in how to get started with the **terraform modules** that can help implements autoscaling fortigates in a AWS GWLB scenario.

There are two parts when analysing the solution.
- The part that handles autoscaling in AWS
- Logic to implement autoscaling on Fortigate

  
The core of the solution, are **two AWS autoscaling groups (ASG)**, one to control respectivily the number of BYOL instances and one ASG for PAYG Fortigate instances.
**Cloudwatch** will continously monitor the resource utilsation and scale in/out the autoscaling groups based on predefined tresholds, in other words, add or remove Fortigate instances when required.

The creation of a FGT instance is initiated by Cloudwatch instructing the ASG.
Once the FGT instances is started, **cloud-init** is used to configure the newly instantiated FGT with a basic network configuration configuration to allow access to the instance.

**AWS EventBridge** looks out for notifications from **CloudWatch** indicating successfull FGT EC2 instance launches (and removal) and triggers an **AWS LAMBDA function** to configure the newly created FGT.
A similar approach is folowed when removing an instances.

The purpose of the Lambda function (fgt_asg_launch_fgt_byol_asg and fgt_asg_launch_fgt_on_demand_asg) is to configure the FGT with
- license file or Fortiflex
- initial policy config
- the advanced network configuration (GENEVE tunnels)
- configuring cluster-autoscale





