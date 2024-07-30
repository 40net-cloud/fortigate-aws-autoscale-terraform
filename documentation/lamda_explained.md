# Overview of the Lambda Script (fgt-asg-lambda_fgt_byol_asg)

The provided lambda script is designed to control auto-scaling with FortiGates in AWS. It is written in Python and utilizes various AWS services such as EC2, S3, and DynamoDB.
## Initialization
The script starts by initializing the necessary AWS clients and variables:

- ec2_client: an EC2 client for interacting with EC2 instances
- s3_client: an S3 client for interacting with S3 buckets
- dynamodb_client: a DynamoDB client for interacting with DynamoDB tables
- s3_bucket_name: the name of the S3 bucket used for storing interface track files
- dynamodb_table_name: the name of the DynamoDB table used for storing item
- intf_track.json: the name of the interface track file stored in S3
- asg-fgt-lic-track.json: the name of the license track file stored in S3 (BYOL)

## Main Functionality
The script's main functionality is defined in the main method, which takes an event object as input. This method is responsible for controlling the auto-scaling process based on the event type.
### Event Types
The script handles two types of events:

- EC2 Instance-launch Lifecycle Action: triggered when an EC2 instance is launched
- EC2 Instance-terminate Lifecycle Action: triggered when an EC2 instance is terminated

### Launch Event
When a launch event is received, the script performs the following actions:

- Retrieves the instance ID and availability zone from the event object
- Gets the interface settings from the environment variable network_interfaces
- Iterates through the interface settings and creates a new network interface for each one
- Attaches the new network interface to the EC2 instance
- Sets the DeleteOnTermination attribute for the network interface to True
- Associates a public IP address with the network interface if required

### Terminate Event
When a terminate event is received, the script performs the following actions:

- Retrieves the instance ID from the event object
- Gets the interface track dictionary from S3
- Iterates through the interface track dictionary and cleans up each interface
- Removes the interface track file from S3

### Interface Track File
The script uses an interface track file stored in S3 to keep track of the network interfaces created for each EC2 instance. The file contains a dictionary where the keys are instance IDs and the values are dictionaries of network interfaces.

### Locking Mechanism
The script uses a locking mechanism to prevent concurrent access to the interface track file. The locking mechanism uses a tag on the S3 object to indicate whether the file is locked or not.

### DynamoDB Operations
The script provides several methods for interacting with DynamoDB, including:

- get_item_from_dydb: retrieves an item from DynamoDB
- put_item_to_dydb: updates an item in DynamoDB
- add_item_to_dydb: adds a new item to DynamoDB
- remove_item_from_dydb: removes an item from DynamoDB

### Logging
The script uses the logging module to log important events and errors. The log level is set to INFO by default.

## Conclusion
In conclusion, the provided lambda script is designed to control auto-scaling with FortiGates in AWS. It handles launch and terminate events, creates and cleans up network interfaces, and uses a locking mechanism to prevent concurrent access to the interface track file. The script also provides several methods for interacting with DynamoDB.