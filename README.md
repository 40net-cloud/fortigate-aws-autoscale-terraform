# Fortigate autoscaling in AWS
This repo contains additional information and insights in how to get started with the **terraform modules** that can help implements autoscaling fortigates in a AWS GWLB scenario.
The core of the solution, are **two AWS autoscaling groups**, one to contrik respectivily the BYOL and one for PAYG Fortigate instances.
**Cloudwatch** will monitor the resource utilsation and scale in/out the autoscaling groups, in other words, add or remove Fortigate instances.
