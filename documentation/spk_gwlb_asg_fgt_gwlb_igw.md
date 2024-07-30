### 1. spk_gwlb_asg_fgt_gwlb_igw (aka DISTRIBUTED GWLB deployment)
This terraform plan will simply create
- Security VPC
- GWLB
- ASG for byol/payg fortigates and related resources

Optionlly, one or more spokes can be connected.<br>
Please note the setup assumes that your spokes are in the ranges defined here `spoke_cidr_list    = ["10.0.0.0/16"]` in `terraform.tfvars`

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