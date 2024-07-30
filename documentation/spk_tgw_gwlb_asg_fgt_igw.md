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
