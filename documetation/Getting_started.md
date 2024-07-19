# Getting started

## Pre-requisites
Prepare an AWS environment. 
- AWS access key / secret key
- SSH key

## Setup
```
git clone https://github.com/fortinet/terraform-aws-cloud-modules.git
```

## Architecture 
The repo contains:
- terraform modules
- examples

The examples directory contains 3 'packaged' deployments

### 1. spk_gwlb_asg_fgt_gwlb_igw
	
### 2. spk_tgw_gwlb_asg_fgt_gwlb_igw
	
### 3. spk_tgw_gwlb_asg_fgt_igw
This terraform plan will create 
- a transit gateway
- GWLB and endpoints
- ASG for byol/payg fortigates
- Internet gateway
- Support for E/W
- Support for EGRESS (NAT on FGT)

Optionlly, one or more spokes are connected via transit gateway attachments to the transit gateway

Traffic is routed from the spokes, via the TGW attachments and TGW to the GWLB.<br>
The GWLB will forward the traffic to the available Fortigates.<br>
E/W traffic is routed back towards teh GWLB.<br>
Internet (EGRESS) is NATted on the FGT's and reaches the internet via the IGW.<br>
