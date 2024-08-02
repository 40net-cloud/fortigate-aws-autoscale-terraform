### spk_gwlb_asg_fgt_gwlb_igw (Distributed GWLB deployment)
This terraform plan will simply create
- Security VPC
- GWLB
- ASG for byol/payg fortigates and related resources

Optionlly, one or more spokes can be connected.<br>

Please note the setup assumes that your spokes are in the ranges defined here `spoke_cidr_list    = ["10.1.0.0/16"]` in `terraform.tfvars`

#### Simplified flow
1. Traffic is routed from the spoke EC2 instances towards GWLB endpoints (GWLBe) **in the same spoke vpc**. <br>
   Please note that these endpoints are **NOT** created by default.<br>
2. The GWLB will pickup the incoming traffic via the GWLBe and forward it over one of the GENEVE tunnels terminiated on port1 of the selected (load balancing) Fortigates.<br>
3. **All traffic** after inspection is routed back towards the GENEVE tunnel via policy based routes on the Fortigate to the GWLB back to the GWLBe.
   Traffic exiting the GWLBe is subject to the routing table of that subnet.<br>
<br>
The concept is explained here:

![Alt text](https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2023/01/27/GWLB-MSR-Blog-Figure-2-1024x538.png)
Source: [AWS Blogs](https://aws.amazon.com/blogs/networking-and-content-delivery/vpc-routing-enhancements-and-gwlb-deployment-patterns/)

### Setup
Navigate to:
```
cd ./examples/spk_gwlb_asg_fgt_gwlb_igw/
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
For reference find an example.
```
## Note: Please go through all arguments in this file and replace the content with your configuration! This file is just an example.
## "<YOUR-OWN-VALUE>" are parameters that you need to specify your own value.

## Root config
access_key = "<YOUR-OWN-VALUE>"
secret_key = "<YOUR-OWN-VALUE>"
region     = "<YOUR-OWN-VALUE>" # e.g. "us-east-2"

## VPC
security_groups = {
  secgrp1 = {
    description = "Security group by Terraform"
    ingress = {
      all_traffic = {
        from_port   = "0"
        to_port     = "0"
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }
    }
    egress = {
      all_traffic = {
        from_port   = "0"
        to_port     = "0"
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }
    }
  }
}

vpc_cidr_block     = "<YOUR-OWN-VALUE>" # e.g. "10.0.0.0/16"
spoke_cidr_list    = "<YOUR-OWN-VALUE>" # e.g. ["10.1.0.0/16"]
availability_zones = "<YOUR-OWN-VALUE>" # e.g. ["us-east-2a", "us-east-2b"]

...
```
## Adding a VPC 
You can connect an existing VPC by filling out the required information.<br>
Please note that no current associations or default gateways should be configured.
```
## Adding GWLBe and updating routing tables in an existing Spoke VPC 
    spk_vpc = {
      ## Adding GWLBe in the designated subnets
      "spk_vpc1" = {
        vpc_id = "<VPC_ID_TO_ADD>",
        gwlbe_subnet_ids = [
          "<SUBNET_ID_GWLBe_AZ1>",
          "<SUBNET_ID_GWLBe_AZ2>"
        ]

        route_tables = {
          ## igw_inbound - defines the routes for the IGW association to reach the EC2 instances
          igw_inbound = {
            routes = {
              az1 = {
                destination_cidr_block = "<CIDR_APP_SERVERS_AZ1>"
                gwlbe_subnet_id        = "<SUBNET_ID_GWLBe_AZ1>"
              },
              az2 = {
                destination_cidr_block = "<CIDR_APP_SERVERS_AZ2>"
                gwlbe_subnet_id        = "<SUBNET_ID_GWLBe_AZ2>"
              },
            },
            rt_association_gateways = ["<IGW_ID>"]
          },

          ## gwlbe_outbound - defines the routes for the GWLB subnets (outbound)
          gwlbe_outbound = {
            routes = {
              az1 = {
                destination_cidr_block = "0.0.0.0/0"
                gateway_id        = "<IGW_ID>"
              }
            },
            rt_association_subnets = ["<SUBNET_ID_GWLBe_AZ1>", "<SUBNET_ID_GWLBe_AZ2>"]
          },

          ## pc_outbound_azX - defines the route updates for the EC2 instances subnets (outbound)
          pc_outbound_az1 = {
            routes = {
              az1 = {
                destination_cidr_block = "0.0.0.0/0"
                gwlbe_subnet_id        = "<SUBNET_ID_GWLBe_AZ1>"
              }
            },
            existing_rt = {
              id = "<RTB_APP_SERVER_AZ1_ID>"
            }
          },

          pc_outbound_az2 = {
            routes = {
              az1 = {
                destination_cidr_block = "0.0.0.0/0"
                gwlbe_subnet_id        = "<SUBNET_ID_GWLBe_AZ2>"
              }
            },
            existing_rt = {
              id = "<RTB_APP_SERVER_AZ2_ID>"
            }
          },
        }
      }
    }
```