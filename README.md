# Fortigate autoscaling in AWS
This repo contains additional information and insights in how to get started with the **terraform modules** that can help implements autoscaling fortigates in a AWS GWLB scenario.
The core of the solution, are **two AWS autoscaling groups (ASG)**, one to control respectivily the BYOL instances and one ASG for PAYG Fortigate instances.
**Cloudwatch** will monitor the resource utilsation and scale in/out the autoscaling groups based on alarms, in other words, add or remove Fortigate instances.

There are two parts when analysing the solution.
- The part that handles autoscaling in AWS
- Logic to implement autoscaling on Fortigate 

The moment a new Fortigate instance is added ...
