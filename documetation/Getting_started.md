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
This terraform plan will simply create
- Security VPC
- GWLB
- ASG for byol/payg fortigates

Optionlly, one or more spokes can be connected.<br>
Please note and adapt the `spoke_cidr_list    = ["10.1.0.0/16"]` in `terraform.tfvars`

	
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
2. The GWLB will forward the incoming traffic over Geneve tunnels terminiated on port1 on one of the available Fortigates<br>
3. E/W and EGRESS is routed back towards the GWLB over the Geneve tunnels via ??policy based?? routes.<br>
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
2. The GWLB will forward the incoming traffic over Geneve tunnels terminiated on port1 on one of the available Fortigates<br>
3. E/W traffic is routed back towards the GWLB over the Geneve tunnels via policy based routes.<br>
4. EGRESS (Internet) traffic is source NATted on the FGT's (port2) and reaches the internet via the IGW and natted on the corresponding EIP's.<br>
