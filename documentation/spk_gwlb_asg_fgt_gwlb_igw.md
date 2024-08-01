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

## Auto scale group
# This example is a hybird license ASG
fgt_intf_mode = "2-arm"
asgs = {
  fgt_byol_asg = {
      template_name = "fgt_asg_template"
      fgt_version = "7.2"
      license_type = "byol"
      fgt_password = "<YOUR-OWN-VALUE>" # e.g. "fortinet"
      keypair_name = "<YOUR-OWN-VALUE>" # Keypair should be created manually
      lic_folder_path = "./license"
      # fortiflex_refresh_token = "<YOUR-OWN-VALUE>" # e.g. "NasmPa0CXpd56n6TzJjGqpqZm9Thyw"
      # fortiflex_sn_list = "<YOUR-OWN-VALUE>" # e.g. ["FGVMMLTM00000001", "FGVMMLTM00000002"]
      # fortiflex_configid_list = "<YOUR-OWN-VALUE>" # e.g. [2343]
      enable_fgt_system_autoscale = true
      intf_security_group = {
        login_port = "secgrp1"
        internal_port = "secgrp1"
      }
      user_conf_file_path = "<YOUR-OWN-VALUE>" # e.g. "./fgt_config.conf"
        # There are 3 options for providing user_conf data: 
        # user_conf_content : FortiGate Configuration
        # user_conf_file_path : The file path of configuration file
        # user_conf_s3 : Map of AWS S3 
      asg_max_size = 1
      asg_min_size = 1
      # asg_desired_capacity = 1
      create_dynamodb_table = true
      dynamodb_table_name = "fgt_asg_track_table"
  },
  fgt_on_demand_asg = {
      template_name = "fgt_asg_template_on_demand"
      fgt_version = "7.2"
      license_type = "on_demand"
      fgt_password = "<YOUR-OWN-VALUE>" # e.g."fortinet"
      keypair_name = "<YOUR-OWN-VALUE>" # e.g. Keypair should be created manually
      enable_fgt_system_autoscale = true
      intf_security_group = {
        login_port = "secgrp1"
        internal_port = "secgrp1"
      }
      user_conf_file_path = "<YOUR-OWN-VALUE>" # e.g. "./fgt_config.conf"
        # There are 3 options for providing user_conf data: 
        # user_conf_content : FortiGate Configuration
        # user_conf_file_path : The file path of configuration file
        # user_conf_s3 : Map of AWS S3 
      asg_max_size = 2
      asg_min_size = 0
      # asg_desired_capacity = 0
      dynamodb_table_name = "fgt_asg_track_table"
      scale_policies = {
        byol_cpu_above_80 = {
            policy_type               = "SimpleScaling"
            adjustment_type           = "ChangeInCapacity"
            cooldown                  = 60
            scaling_adjustment        = 1
        },
        byol_cpu_below_30 = {
            policy_type               = "SimpleScaling"
            adjustment_type           = "ChangeInCapacity"
            cooldown                  = 60
            scaling_adjustment        = -1
        },
        ondemand_cpu_above_80 = {
            policy_type               = "SimpleScaling"
            adjustment_type           = "ChangeInCapacity"
            cooldown                  = 60
            scaling_adjustment        = 1
        },
        ondemand_cpu_below_30 = {
            policy_type               = "SimpleScaling"
            adjustment_type           = "ChangeInCapacity"
            cooldown                  = 60
            scaling_adjustment        = -1
        }
      }
  }
}

## Cloudwatch Alarm
cloudwatch_alarms = {
  byol_cpu_above_80 = {
    comparison_operator = "GreaterThanOrEqualToThreshold"
    evaluation_periods  = 2
    metric_name         = "CPUUtilization"
    namespace           = "AWS/EC2"
    period              = 120
    statistic           = "Average"
    threshold           = 80
    dimensions = {
      AutoScalingGroupName = "fgt_byol_asg"
    }
    alarm_description = "This metric monitors average ec2 cpu utilization of Auto Scale group fgt_asg_byol."
    datapoints_to_alarm = 1
    alarm_asg_policies     = {
      policy_name_map = {
        "fgt_on_demand_asg" = ["byol_cpu_above_80"]
      }
    }
  },
  byol_cpu_below_30 = {
    comparison_operator = "LessThanThreshold"
    evaluation_periods  = 2
    metric_name         = "CPUUtilization"
    namespace           = "AWS/EC2"
    period              = 120
    statistic           = "Average"
    threshold           = 30
    dimensions = {
      AutoScalingGroupName = "fgt_byol_asg"
    }
    alarm_description = "This metric monitors average ec2 cpu utilization of Auto Scale group fgt_asg_byol."
    datapoints_to_alarm = 1
    alarm_asg_policies     = {
      policy_name_map = {
        "fgt_on_demand_asg" = ["byol_cpu_below_30"]
      }
    }
  },
  ondemand_cpu_above_80 = {
    comparison_operator = "GreaterThanOrEqualToThreshold"
    evaluation_periods  = 2
    metric_name         = "CPUUtilization"
    namespace           = "AWS/EC2"
    period              = 120
    statistic           = "Average"
    threshold           = 80
    dimensions = {
      AutoScalingGroupName = "fgt_on_demand_asg"
    }
    alarm_description = "This metric monitors average ec2 cpu utilization of Auto Scale group fgt_asg_ondemand."
    alarm_asg_policies     = {
      policy_name_map = {
        "fgt_on_demand_asg" = ["ondemand_cpu_above_80"]
      }
    }
  },
  ondemand_cpu_below_30 = {
    comparison_operator = "LessThanThreshold"
    evaluation_periods  = 2
    metric_name         = "CPUUtilization"
    namespace           = "AWS/EC2"
    period              = 120
    statistic           = "Average"
    threshold           = 30
    dimensions = {
      AutoScalingGroupName = "fgt_on_demand_asg"
    }
    alarm_description = "This metric monitors average ec2 cpu utilization of Auto Scale group fgt_asg_ondemand."
    alarm_asg_policies     = {
      policy_name_map = {
        "fgt_on_demand_asg" = ["ondemand_cpu_below_30"]
      }
    }
  }
}

## Gateway Load Balancer
enable_cross_zone_load_balancing = true

## Tag
general_tags = {
  "purpuse" = "ASG_TEST"
}
```
You can connect an existing VPC by filling out the required information.
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