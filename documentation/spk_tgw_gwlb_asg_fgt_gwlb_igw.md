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