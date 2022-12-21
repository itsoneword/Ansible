# deploying web services on AWS via ansible


### 1. Structure

#### Prod:
2 Ubuntu distribs withthe setup:
    Ansible user with SSH key authentication.
    zsh bash for ansible user.
    Installation and version checker for services:

        Python scheduled task checking repo and mooving from 1 folder to another
        web services showing state of the latest task
#### DR:
2 Ubuntu destribs mirroring prod. Active in case of prod load is higher then expected (80% cpu)
    Ansible user with SSH key auth
    zsh bash
    same roles as for 1st infra.

#### Monitoring:
1 ubuntu machine with grafana (prometey?) showing resource usage for prod and DR. web service.

#### Ansible:
1 ubuntu server with ansible services responsible for deploying infra.

### 2. Choosing infra setup

all VMs are deployed with masterKey (should be enabled later?)

#### Ansible roles:
    user
    zsh
    webservice
    python    (python packs check ?)

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
~: sudo apt install awscli
~: aws configure
AWS Access Key ID [None]: ******
AWS Secret Access Key [None]: ****
Default region name [None]:  eu-central-1
Default output format [None]: json
```
Using **Access Key ID** and **Secret Access Key** from AWS IAM - Users - Security credentials.
[Region names and available zones](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html)

[Output formats](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-output-format.html)

also [installing jq](https://stedolan.github.io/jq/) to work with json output - will be needed for variables. 
```bash 
~: sudo apt get install jq
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
~: aws ec2 create-vpc --cidr-block 192.168.20.0/24 
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
~: aws ec2 describe-vpcs --filters Name=cidr,Values=192.168.20.0/24 | jq -r '.Vpcs | .[] | .VpcId'

vpc-00756859b9e6628fb
#we can also use variables for the filter, saving there cidr brfore: cidr=192.168.20.0/24
#Please, be accurate with quotes.

~:  awsvpcid=$(aws ec2 describe-vpcs --filters "Name=cidr,Values=192.168.20.0/24" | jq -r '.Vpcs | .[] | .VpcId')


#Then storing data to the config file, just in case:
~: cd ~
~: mkdir awsConfig
~: touch awsConfig//awsdeploy.log
~: echo $(date), user=$USER >> awsConfig/awsdeploy.log
~: echo vpcid : $awsvpcid >> awsConfig/awsdeploy.log

```

all right, we have VPC created, and data stored in vars and config file. Let's now create 2 subnets:

```bash
~: aws ec2 create-subnet --vpc-id $awsvpcid --cidr-block 192.168.20.0/26

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

#Store IDs into vars:
~: awssubnetid1=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$awsvpcid | jq -r '.Subnets |.[] |.SubnetId')
~: echo SubnetId1 : $awssubnetid1 >> awsConfig/awsdeploy.log



```




### 4. Setting up ansible

### 5. Playbooks

### 6. Testing\Running
