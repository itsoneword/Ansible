# Ansible

deploying web services on AWS  with ansible


### 1. Structure

Prod:
2 Ubuntu distribs withthe setup:
    Ansible user with SSH key authentication.
    zsh bash for ansible user.
    Installation and version checker for services:

        Python scheduled task checking repo and mooving from 1 folder to another
        web services showing state of the latest task
DR:
2 Ubuntu destribs mirroring prod. Active in case of prod load is higher then expected (80% cpu)
    Ansible user with SSH key auth
    zsh bash
    same roles as for 1st infra.

Monitoring:
1 ubuntu machine with grafana (prometey?) showing resource usage for prod and DR. web service.

Ansible:
1 ubuntu server with ansible services responsible for deploying infra.

### 2. Choosing infra setup

all VMs are deployed with masterKey (should be enabled later?)
Ansible roles:
    user
    zsh
    webservice
    python    (python packs check ?)
Ansible hosts:
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
#### 1st installing AWS.
[AWS CLI reffeernce](https://docs.aws.amazon.com/cli/latest/reference/ec2/index.html)
```bash
~: sudo apt install awscli
~: aws configure
AWS Access Key ID [None]: ******
AWS Secret Access Key [None]: ****
Default region name [None]:  eu-central-1
Default output format [None]: json
```
Using **Access Key ID** and **Secret Access Key** from AWS IAM - Users - Security credentials.
[Region names are available zones](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html)
[Output formats](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-output-format.html)

also [installing jq](https://stedolan.github.io/jq/) to work with json output - will be needed for variables. 
```bash 
~: sudo apt get install jq
```


### 4. Setting up ansible

### 5. Playbooks

### 6. Testing\Running
