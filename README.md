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
1 ubuntu machine with zabbix (prometey?) showing resource usage for prod and DR. web service.

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

### 4. Setting up ansible

### 5. Playbooks

### 6. Testing\Running
