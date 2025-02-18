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

#### Simplified flow
1. Traffic is routed from the spokes, via the TGW attachments and TGW to the GWLB endpoints in the security VPC.<br>
2. The GWLB will pickup the incoming traffic and forward it over one of the GENEVE tunnels terminiated on port1 of the available Fortigates<br>
3. E/W traffic is routed back towards the GWLB over the GENEVE tunnels via policy based routes.<br>
4. EGRESS (Internet) traffic is source NATted on the FGT's (port2) and reaches the internet via the IGW and natted on the corresponding EIP's.<br>

### Setup
Navigate to:
```
cd ./examples/spk_tgw_gwlb_asg_fgt_igw/
```
Initialise Terraform:
```
terraform init
```
Copy `terraform.tfvars.txt` to `terraform.tfvars`.
```
cp terraform.tfvars.txt terraform.tfvars
```
Update the `terraform.tfvars` and replace all placeholders "\<YOUR-OWN-VALUE\>" with your the required information.<br>
You can comment/uncomment adding/removing `#`<br>

## Adding a VPC
Example:
```
spk_vpc = {
# This is optional. The module will create Transit Gateway Attachment under each subnet in argument 'subnet_ids', and also create route table to let all traffic (0.0.0.0/0) forward to the TGW attachment with the subnets associated.
   "spk_vpc1" = {
     vpc_id = "vpc-01e491cdf48eb8fcf", #VPC to add
     subnet_ids = [
       "subnet-04a02980ac1bff7d0", #subnet to attach TGW AZ1
       "subnet-053ec8fbef047f505"  #subnet to attach TGW AZ2
     ]
   }
 }
```