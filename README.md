# Fortigate autoscaling in AWS
This repo contains additional information and insights in how to get started with the **terraform modules** that can help implements autoscaling fortigates in a AWS GWLB scenario.

There are two parts when analysing the solution.
- The part that handles autoscaling in AWS
- Logic to implement autoscaling on Fortigate

  
The core of the solution, are **two AWS autoscaling groups (ASG)**, one to control respectivily the number of BYOL instances and one ASG for PAYG Fortigate instances.
**Cloudwatch** will continously monitor the resource utilsation and scale in/out the autoscaling groups based on predefined tresholds, in other words, add or remove Fortigate instances when required.

The creation of a FGT instance is initiated by Cloudwatch instructing the ASG.
Once the FGT instances is started, **cloud-init** is used to configure the newly instantiated FGT.


