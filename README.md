# Deploying EC2 instances on AWS via ansible


### 1. Structure

#### Prod:
2 Ubuntu distribs withthe setup:
   * Ansible user with SSH key authentication.
   * zsh for ansible user.
   * Installation and version checker for services:

        Python scheduled task checking repo and mooving from 1 folder to another
        web services showing state of the latest task
#### DR:
2 Ubuntu destribs mirroring prod. Active in case of prod load is higher then expected (80% cpu)
   * Ansible user with SSH key auth
   * zsh 
   * same roles as for 1st infra.

#### Monitoring:
* 1 ubuntu machine with grafana (prometey?) showing resource usage for prod and DR. web service.

#### Ansible:
* 1 ubuntu server with ansible services responsible for deploying infra.

### 2. Choosing infra setup

all VMs are deployed with masterKey (should be enabled later?)

#### Ansible roles:
   * user
   * zsh
   * webservice
   * python    (python packs check ?)

#### Ansible hosts:
[prod]
    host1
    host2
[DR]
    host3
    host4
[Monitoring]
    host5
    
### 3. Deploying machines

To deploy AWS EC2 instances it was choosen to use awscli.
These steps might be automized later and be used for deploying in scripts.
#### 3.1 1st installing AWS cli : [AWS CLI refference](https://docs.aws.amazon.com/cli/latest/reference/ec2/index.html)
```bash
	$ sudo apt install awscli
	$ aws configure
		AWS Access Key ID [None]: ******
		AWS Secret Access Key [None]: ****
		Default region name [None]:  eu-central-1
		Default output format [None]: json
	```
Using **Access Key ID** and **Secret Access Key** from `AWS IAM - Users - Security credentials`.
[Region names and available zones](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html)

[Output formats](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-output-format.html)

also [installing jq](https://stedolan.github.io/jq/) to work with json output - will be needed for variables. 
```bash 
	$ sudo apt get install jq
```

All right, the process of creating infrastructure is the following:
 1. Create VPC (Virtual Provate Cloud)
 2. Create Subnets inside this VPC
 3. Create GW and Routing tables
 4. Allocate routing tables to subnets
 5. Deploy security groups 
 6. Creating VM instance.

During the procedure it would make sence to store some data as variables - it would help us in future infrastructure understenting.
#### 3.2Configuring network
So, starting with creating VPC:

```bash
$ aws ec2 create-vpc --cidr-block 192.168.20.0/24 
#where --cidr-block is the IP range of available by private cloud addresses
#Output
	{
	"Vpc": {
		"CidrBlock": "192.168.10.0/24",
		"DhcpOptionsId": "dopt-0dd6d24ea7585150a",
		"State": "pending",
		"VpcId": "vpc-00756859b9e6628fb",
		"OwnerId": "746002417955",
		"InstanceTenancy": "default",
		"Ipv6CidrBlockAssociationSet": [],
		"CidrBlockAssociationSet": [
				{
				"AssociationId": "vpc-cidr-assoc-05e963551ba31ffe8",
				"CidrBlock": "192.168.20.0/24",
				"CidrBlockState": {
				"State": "associated"
				}
		}
		],
		"IsDefault": false
		}
	}

#and I would recommend to store vpcid in the variable and config file right after that:
$ aws ec2 describe-vpcs --filters Name=cidr,Values=192.168.20.0/24 | jq -r '.Vpcs | .[] | .VpcId'

vpc-00756859b9e6628fb
#we can also use variables for the filter, saving there cidr brfore: cidr=192.168.20.0/24
#Please, be accurate with quotes.

$ awsvpcid=$(aws ec2 describe-vpcs --filters "Name=cidr,Values=192.168.20.0/24" | jq -r '.Vpcs | .[] | .VpcId')


#Then storing data to the config file, just in case:
$ cd ~
$ mkdir awsConfig
$ touch awsConfig//awsdeploy.log
$ echo $(date), user=$USER >> awsConfig/awsdeploy.log
$ echo vpcid : $awsvpcid >> awsConfig/awsdeploy.log

```

all right, we have VPC created, and data stored in vars and config file. Let's now create 2 subnets:

```bash
$ aws ec2 create-subnet --vpc-id $awsvpcid --cidr-block 192.168.20.0/26

#Output:
	{
	"Subnet": {
		"AvailabilityZone": "eu-central-1c",
		"AvailabilityZoneId": "euc1-az1",
		"AvailableIpAddressCount": 59,
		"CidrBlock": "192.168.10.0/26",
		"DefaultForAz": false,
		"MapPublicIpOnLaunch": false,
		"State": "available",
		"SubnetId": "subnet-01bad4acbbef2bd1f",
		"VpcId": "vpc-00756859b9e6628fb",
		"OwnerId": "746002417955",
		"AssignIpv6AddressOnCreation": false,
		"Ipv6CidrBlockAssociationSet": [],
		"SubnetArn": "arn:aws:ec2:eu-central-1:746002417955:subnet/subnet-01bad4acbbef2bd1f",
		"EnableDns64": false,
		"Ipv6Native": false,
		"PrivateDnsNameOptionsOnLaunch": {
			"HostnameType": "ip-name",
			"EnableResourceNameDnsARecord": false,
			"EnableResourceNameDnsAAAARecord": false
			}
	}
	}

#Store IDs into vars and config file:
$ awssubnetid1=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$awsvpcid | jq -r '.Subnets |.[] |.SubnetId')
$ echo SubnetId1 : $awssubnetid1 >> awsConfig/awsdeploy.log
```
#### Creating Internet Gateway and routing tables

Next, we need to deploy Gateway for our VPC. But the problem is - by default it is not assigned to any other objects, so we need to somehow figure out its ID. We can use tags for that, but its not really conviniet as far as the same tag can be assigned to 2 different objects. So, the best way here is to get the ID right after the execution:

```bash
$ awsigwid=$(aws ec2 create-internet-gateway | jq -r '.InternetGateway | .InternetGatewayId')

#There will be no output, as far as it goes to filter and then straight to the variable, but just for information output looks like that:
	{
	"InternetGateway": {
		"Attachments": [],
		"InternetGatewayId": "igw-06d93681ae82dfcfb",
		"OwnerId": "746002417955",
		"Tags": []
	}
	}

#and store it into config file as well:
$ echo internetGateWayID : $awsigwid >> awsConfig/awsdeploy.log

```
Once new GW is created - we need to assign it to our VPC:

```bash
$ aws ec2 attach-internet-gateway --vpc-id $awsvpcid --internet-gateway-id $awsigwid

```
and create route -ables:

```bash
$ aws ec2 create-route-table --vpc-id $awsvpcid
#Output
{
	"RouteTable": {
		"Associations": [],
		"PropagatingVgws": [],
		"RouteTableId": "rtb-03b29e8e92d817b8a",
		"Routes": [
			{
			"DestinationCidrBlock": "192.168.10.0/24",
			"GatewayId": "local",
			"Origin": "CreateRouteTable",
			"State": "active"
			}
			],
		"Tags": [],
		"VpcId": "vpc-00756859b9e6628fb",
		"OwnerId": "746002417955"
		}
	}
#assign table id to variable and store into config file:
awsrtid=$(aws ec2 create-route-table --vpc-id $awsvpcid | jq -r '.RouteTable | .RouteTableId ' )
echo routeTableId : $awsrtid >> awsConfig/awsdeploy.log

#associate RouteTable with Subnets:

$ aws ec2 associate-route-table --subnet-id $awssubnetid1 --route-table-id $awsrtid
#output:
	{
	"AssociationId": "rtbassoc-09f6ff1516b7d9eb7",
		"AssociationState": {
		"State": "associated"
		}
	}
#assign public IDs to subnet:

$ aws ec2 modify-subnet-attribute --subnet-id <SubnetId> --map-public-ip-on-launch
```

Almost done. we only need to create key pair for login and allow ssh (and http if needed):
#### Creating Key Pair and Security Group

```bash
#key pair:
$ aws ec2 create-key-pair --key-name AWS-Keypair --query "KeyMaterial" --output text > "awsConfig/AWS_Keypair.pem"
#security groups
$ awssgid=$(aws ec2 create-security-group --group-name allowssh --description "allows ssh (22 port) connection to the machine" --vpc-id $awsvpcid | jq -r '.GroupId')
#Output will be empty again. Example for refference:
	{
		"GroupId": "sg-0199af8cc0d8447bb"
	}
#Saving data to config file	
$ echo securityGroupId : $awssgid >> awsConfig/awsdeploy.log
#Enabling the settings (all IPs via 22 port) for created group:
$ aws ec2 authorize-security-group-ingress --group-id $awssgid  --protocol tcp --port 22 --cidr 0.0.0.0/0
#oiutput example:
	{
		"Return": true,
		"SecurityGroupRules": [
		{
			"SecurityGroupRuleId": "sgr-07f6d01b908482133",
			"GroupId": "sg-034bceb76fae44ec5",
			"GroupOwnerId": "746002417955",
			"IsEgress": false,
			"IpProtocol": "tcp",
			"FromPort": 22,
			"ToPort": 22,
			"CidrIpv4": "0.0.0.0/0"
		}
		]
	}
```


So thats pretty it. Running the command for instance creation finally:
```bash

$: aws ec2 run-instances --image-id <ami-id> --count 1 --instance-type t2.micro  --key-name <Keypair-name> --security-group-ids <SecurityGroupId> --subnet-id <SubnetId>

# AMI ID:
# ami-06ce824c157700cd2 -  Ubuntu
# ami-0b8ea3624881b47a1 - Red Hat
# ami-0a261c0e5f51090b1 - amazon linux
# [for refference](https://cloud-images.ubuntu.com/locator/ec2/)



### 4. Setting up ansible

### 5. Playbooks

### 6. Testing\Running
