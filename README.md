# Ansible for AWS
This playbook deploys the whole AWS (VPC, Bastion, Web, and natClient(mySQL-Client), and RDS Instances). ~~natClient Configuration is currently manual~~. Bastion is used to control cluster deployed on AWS. Baction machine inventory:

- Ubuntu LTS server (currently "trusty" 16.04)

## Imagination Task:
Write an Ansible Role that includes a number of tasks:
- Power up an RDS instance and set the security group so that only the EC2 instance (below) can connect to it on port 3306 (via internal IP addresses)
- Power up an EC2 instance, running Ubuntu 16.04 and the latest version of the software package mysql-client

**Instructions should be provided on how to run the role, including how we can pass in the following parameters:**
- instance sizes
- AWS region
- AWS creds
- DB creds

We will run on one of our test AWS environments. The goal here is to produce a reusable role.

# Requirements:
```
$ ansible --version
ansible 2.3.2.0
  config file = ~/ansible-aws/ansible.cfg
  configured module search path = Default w/o overrides
  python version = 2.7.13
```
> **Note:** RDS may take up to 30 minutes to deploy the instance


# Prerequisites:

2. Setup Environment
3. AWS account Authentication with Ansible
2. Building the AWS VPC and NAT
3. Building the Bastion
4. ~~ Building Web~~
5. Building natClient and install role mysql-client
6. Lunch RDS

# Setup Environment

In order to run this playbook you need to have the following installed on your machine:

- Python 2.7.x
- pip - Python package manager
- pip modules:
 - ansible - Ansible tool
 - awscli - Amazon CLI for Python
 - boto - AWS libraries

Run following command to install required modules
```
$ pip install ansible awscli boto
```

# AWS Authentication with Ansible

After installation, create a file named boto and provide the necessary credentials.

**Create file `~/.boto`**
```
[Credentials]
AWS_ACCESS_KEY_ID=""
AWS_SECRET_ACCESS_KEY=""
```

AWS uses public-key cryptography to secure the login information for your instance. A Linux instance has no password; you use a key pair to log in to your instance securely.
```
$ ansible-playbook hosts keypair.yml
```

You need also to set environment variables by specifying your Secret Key and Access Key

```
$ export AWS_ACCESS_KEY_ID=""
$ export AWS_SECRET_ACCESS_KEY=""
```
> **Note:** protect your AWS access key and secret access key by using  `ansible-valut`

# Role Variables
**List of Variables**
```

```

# Ansible Imagination Dev Site
To help make the roles reusable and easily updated, the variables were placed in the main site.yml file for configuring all of the aspects from the network, bastion, and RDS.

Example **Site.yml**
```
---
- name: Deploy RDS Infratructure
  hosts: localhost
  connection: local
  gather_facts: false
  vars:

# Global AWS Variables
    region: eu-central-1
    vpc_name: dev

 # VPC Variables
    vpc_cidr: 10.5.0.0/16
    public_subnet_1_cidr: 10.5.0.0/24
    public_subnet_1_az: eu-central-1a
    public_subnet_2_cidr: 10.5.1.0/24
    public_subnet_2_az: eu-central-1b
    private_subnet_1_cidr: 10.5.2.0/24
    private_subnet_1_az: eu-central-1a
    private_subnet_2_cidr: 10.5.3.0/24
    private_subnet_2_az: eu-central-1b

# Bastion Variables
    ami_id: "ami-1e339e71"
    keypair_name: "hosty-key"

# RDS Variables
    rds_user: root
    rds_pass: Default_Pass123!
    rds_instance_type: db.t2.small
    rds_size_gb: 15
    rds_parameter_engine: mysql5.6
    rds_instance_engine: 5.6.35
    rds_parameters:
      - { param: 'binlog_format', value: 'ROW' }
      - { param: 'general_log', value: '1' }

  roles:
  - vpc
  - bastion
  - natclient
  - rds

- hosts: natclient
  become: yes

  roles:
  - common
  - mysql-client
```

# Building AWS-VPC

– Build VPC
– Build Subnets
  – Two Public subnet to access the environment
  – Two Private subnets for internal traffic. Two because the RDS Subnet group requires two for redundancy.
– Internet Gateway for the VPC
– NAT Gateway for Client Instance
– Security groups to allow specific traffic into specific instances

## [Connecting to a DB Instance Running the MySQL Database Engine](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html)

**Connect to RDS through EC2 < Bastion
For example, on Linux or OSX**

Set up this tunnel every time you log into your remote EC2 instance and log into it with whatever name you prefer:


Modify **ssh.cfg**:

```
Host 10.5.*
  User {{ bastion_user }}
  ProxyCommand ssh -o "StrictHostKeyChecking=no" {{ bastion_user }}@{{ bastion_public_ip }} nc %h %p

Host {{ bastion_public_ip }}
	Hostname {{ bastion_public_ip }}
	User {{ bastion_user }}
	ControlMaster   auto
	ControlPath     ~/.ssh/mux-%r@%h:%p
	ControlPersist  15m
	IdentityFile    ~/.ssh/hosty-key.pem
```
Then, just:

```
$ ssh -L 3306:dev-rds.cj3xloa8ykzj.eu-central-1.rds.amazonaws.com:3306 10.5.2.57 -F sshd.cfg
```

And you can then access your remote MySQL server as if it was running locally:
```
$ mysql -h dev-rds.cj3xloa8ykzj.eu-central-1.rds.amazonaws.com -P 3306 -u root -p
```

## Issues:
- If you're using Ansible >2.2.0, you can set the ansible_python_interpreter configuration option to `/usr/bin/python3`: [Python 3 Support](https://docs.ansible.com/ansible/latest/python_3_support.html)
```
ansible my_ubuntu_host -m ping -e 'ansible_python_interpreter=/usr/bin/python3'
```

- Then decided to upgrade to the latest version of ansible (2.3+). Then I created a...

```
group_vars/all

..file and added...

ansible_python_interpreter: /usr/bin/python3
```
