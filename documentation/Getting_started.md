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
1. spk_gwlb_asg_fgt_gwlb_igw
2. spk_tgw_gwlb_asg_fgt_gwlb_igw
3. spk_tgw_gwlb_asg_fgt_igw

The naming more or less describes the flow for E/W and Egress traffic flows. <br>
Please note the modules **DO NOT** create any spoke VPC.
The Terraform modules **can** update exsting spoke VPC's (route table, endpoint creation, etc) to integrate with the deployed infrastructure.

### 1. spk_gwlb_asg_fgt_gwlb_igw (aka DISTRIBUTED GWLB deployment)
This terraform plan will simply create
- Security VPC
- GWLB
- ASG for byol/payg fortigates and related resources

Optionlly, one or more spokes can be connected.<br>
Please note the setup assumes that your spokes are in the ranges defined <br>
in `spoke_cidr_list    = ["10.0.0.0/16"]` in `terraform.tfvars`

#### Simplified flow
1. Traffic is routed from the spoke EC2 instances towards GWLB endpoints (GWLBe) **in the same spoke vpc**. <br>
   Please note that these endpoints are **NOT** created by default.<br>
2. The GWLB will pickup the incoming traffic and forward it over one of the GENEVE tunnels terminiated on port1 of the selected (load balancing) Fortigates<br>
3. **All traffic** after inspection is routed back towards the GENEVE tunnel via policy based routes on the Fortigate to the GWLB back to the GWLBe.<br>
   Traffic existing the GWLBe is subject to the routing table of that subnet.
	
### 2. spk_tgw_gwlb_asg_fgt_gwlb_igw
This terraform plan will create 
- Security VPC
- a transit gateway
- GWLB and endpoints
- ASG for byol/payg fortigates
- NAT gateway in the security VPC
- Support for E/W
- Support for EGRESS (via NAT Gateway)

Optionlly, one or more spokes are connected via transit gateway attachments to the transit gateway.<br>
Please note and adapt the `spoke_cidr_list    = ["10.1.0.0/16"]` in `terraform.tfvars`

1. Traffic is routed from the spokes, via the TGW attachments and TGW to the GWLB endpoints in the security VPC.<br>
2. The GWLB will pickup the incoming traffic and forward it over one of the GENEVE tunnels terminiated on port1 of the available Fortigates<br>
3. E/W and EGRESS is routed back towards the GWLB over the GENEVE tunnels via ??policy based?? routes.<br>
4. EGRESS (Internet) traffic is routed towards the NAT gateway, E/W traffic is is routed towards the TGW.<br>
	
### 3. spk_tgw_gwlb_asg_fgt_igw
This terraform plan will create 
- Security VPC
- a transit gateway
- GWLB and endpoints
- ASG for byol/payg fortigates
- Internet gateway in the security VPC
- Support for E/W
- Support for EGRESS (NAT on FGT)

Optionlly, one or more spokes are connected via transit gateway attachments to the transit gateway.<br>
Please note and adapt the `spoke_cidr_list    = ["10.1.0.0/16"]` in `terraform.tfvars`

1. Traffic is routed from the spokes, via the TGW attachments and TGW to the GWLB endpoints in the security VPC.<br>
2. The GWLB will pickup the incoming traffic and forward it over one of the GENEVE tunnels terminiated on port1 of the available Fortigates<br>
3. E/W traffic is routed back towards the GWLB over the GENEVE tunnels via policy based routes.<br>
4. EGRESS (Internet) traffic is source NATted on the FGT's (port2) and reaches the internet via the IGW and natted on the corresponding EIP's.<br>
