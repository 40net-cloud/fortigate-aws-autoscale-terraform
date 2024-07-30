# Getting started
## Introduction
This is a quickstart to get up and running quickly. <br>
If you need a more detailed step-by-step workshop (see [here](https://fortinetcloudcse.github.io/))

## Pre-requisites
To prepare your AWS environment, ensure you have the following:
- AWS account
- AWS access key and secret key
- SSH key
- Terraform

These essentials will allow you to securely access and manage your AWS resources.

## Setup
Make sure to use the official repository for the latest and updated versions of the modules. The repository includes:
- Terraform modules
- Examples
<br>
This ensures you have access to the most current and reliable resources for your setup.<br>
<br>
In your console, run:

```
git clone https://github.com/fortinet/terraform-aws-cloud-modules.git
```

The examples directory contains 3 'packaged' deployments
1. [Distributed GWLB deployment](spk_gwlb_asg_fgt_gwlb_igw.md)
2. [Centralised EGRESS using NAT gateways](spk_tgw_gwlb_asg_fgt_gwlb_igw.md)
3. [Centralised EGRESS using Fortigate](spk_tgw_gwlb_asg_fgt_igw.md)

The naming more or less describes the flow for E/W and Egress traffic flows. <br>
Please note the modules **DO NOT** create any spoke VPC.
The Terraform modules **can** update exsting spoke VPC's (route table, endpoint creation, etc) to integrate with the deployed infrastructure.