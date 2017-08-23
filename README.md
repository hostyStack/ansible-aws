# Ansible for AWS
This playbook deploys the whole AWS (VPC, Bastion, Web, and natClient(mySQL-Client), and RDS Instances). ~~natClient Configuration is currently manual~~. Bastion is used to control cluster deployed on AWS. Baction machine inventory:

- Ubuntu LTS server (currently "trusty" 16.04)

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

1. Setup Environment
2. AWS account Authentication with Ansible
3. Building the AWS VPC and NAT
4. Building the Bastion
5. ~~ Building Web~~
6. Building natClient and install role mysql-client
7. Lunch RDS

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
[credentials]
AWS_ACCESS_KEY_ID =""
AWS_SECRET_ACCESS_KEY =""
```

AWS uses public-key cryptography to secure the login information for your instance. A Linux instance has no password; you use a key pair to log in to your instance securely.
```
$ ansible-playbook playbooks/keypair.yml -vvvv
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
---
aws_secret_key: required
aws_access_key: required
region:
keypair_name:
ansible_user:
bastion_user:
```

# Ansible Hosty Dev Site
To help make the roles reusable and easily updated, the variables were placed in the main `site.yml` file for configuring all of the aspects from the network, bastion, and RDS.

Example **Site.yml**
```
---
- name: Deploy RDS Infratructure
  hosts: localhost
  connection: local
  gather_facts: false
  vars:

# Global AWS Variables
    region: eu-west-2
    vpc_name: dev

 # VPC Variables
    vpc_cidr: 10.5.0.0/16
    public_subnet_1_cidr: 10.5.0.0/24
    public_subnet_1_az: eu-west-2a
    public_subnet_2_cidr: 10.5.1.0/24
    public_subnet_2_az: eu-west-2b
    private_subnet_1_cidr: 10.5.2.0/24
    private_subnet_1_az: eu-west-2a
    private_subnet_2_cidr: 10.5.3.0/24
    private_subnet_2_az: eu-west-2b

# Bastion Variables (Ubuntu 16.04)
    ami_id: "ami-996372fd"  #eu-west-2
    keypair_name: "hostykey"

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
> **Note:** connection can be tested from bastion/web/natclient to RDS

# Building AWS-VPC

– Build VPC
– Build Subnets
  – Two Public subnet to access the environment
  – Two Private subnets for internal traffic. Two because the RDS Subnet group requires two for redundancy.
– Internet Gateway for the VPC
– NAT Gateway for Client Instance
– Security groups to allow specific traffic into specific instances

## [Connecting to a DB Instance Running the MySQL Database Engine](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html)

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
Then, **Connect to RDS through EC2 < Bastion for example, on Linux or OSX**:

```
$ ssh -L 3306:dev-rds.cj3xloa8ykzj.eu-central-1.rds.amazonaws.com:3306 10.5.2.57 -F sshd.cfg -v
```

And you can then access your remote MySQL server as if it was running locally:
```
$ mysql -h dev-rds.cj3xloa8ykzj.eu-central-1.rds.amazonaws.com -P 3306 -u root -p
```

# Issues:
## Python:
- If you're using Ansible >2.2.0, you can set the ansible_python_interpreter configuration option to `/usr/bin/python3`: [Python 3 Support](https://docs.ansible.com/ansible/latest/python_3_support.html)

```
ansible my_ubuntu_host -m ping -e 'ansible_python_interpreter=/usr/bin/python3'
```

- Then decided to upgrade to the latest version of ansible (2.3+). Then I created a **group_vars/all** file and added...

```
ansible_python_interpreter: /usr/bin/python3
```

~~- Testing from another ENV Others two solutions help from macOS~~

```
---
- name: install python
  raw: bash -c "test -e /usr/bin/python || (apt -qqy update && apt install -qqy python python-pip python3 python3-pip)"
  register: output
  changed_when: output.stdout != ""
```
