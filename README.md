# Ansible for AWS
<<<<<<< HEAD
=======
- This playbook deploys the whole AWS (VPC, Bastion, Web, and natClient(mySQL-Client), and RDS Instances). natClient Configuration is currently manual
- Must have credentials exported for AWS IAM
- RDS may take up to 30 minutes to deploy the instance
>>>>>>> 88dc297cc812ca780d9da103de66db9358d7b5f0

## Task:
Write an Ansible Role that includes a number of tasks:
- Power up an RDS instance and set the security group so that only the EC2 instance (below) can connect to it on port 3306 (via internal IP addresses)
- Power up an EC2 instance, running Ubuntu 16.04 and the latest version of the software package mysql-client

<<<<<<< HEAD
### Instructions should be provided on how to run the role, including how we can pass in the following parameters:
=======
**Instructions should be provided on how to run the role, including how we can pass in the following parameters:**
>>>>>>> 88dc297cc812ca780d9da103de66db9358d7b5f0
- instance sizes
- AWS region
- AWS creds
- DB creds

We will run on one of our test AWS environments. The goal here is to produce a reusable role.

<<<<<<< HEAD
Prerequisites:

1. AWS Authentication with Ansible
2. Building the AWS Network
3. Building the Bastion
5. Building RDS
6. Building the MySQL-Client

- This playbook deploys the whole AWS Network, Bastion, Web, and RDS Instances.  Web Configuration is currently manual
- Must have credentials exported for AWS IAM
- RDS may take up to 30 minutes to deploy the instance

Ansible Workstation Setup:
# Setup Environment
=======
# Prerequisites:

1. AWS account Authentication with Ansible
2. Building the AWS VPC and NAT
3. Building the Bastion
4. ~~ Building Web~~
5. Building natClient and install role mysql-client
6. Lunch RDS

# Setup Environment

**Installation**

On macOS, if you already have Homebrew installed, you can install with:
```
$ brew install ansible
```

If you prefer to install via Python and pip, please read the [documentation](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-pip).

On **Ubuntu**:

$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
>>>>>>> 88dc297cc812ca780d9da103de66db9358d7b5f0

# AWS Authentication with Ansible

After installation, create a file named boto and provide the necessary credentials.

AWS uses public-key cryptography to secure the login information for your instance. A Linux instance has no password; you use a key pair to log in to your instance securely.

```
<<<<<<< HEAD
$ ansible-playbook -i hosts keypair.yml
```

Create file ~/.boto
=======
[Credentials]
aws_access_key_id = (access key)
aws_secret_access_key = (secret access key)
```

AWS uses public-key cryptography to secure the login information for your instance. A Linux instance has no password; you use a key pair to log in to your instance securely.

>>>>>>> 88dc297cc812ca780d9da103de66db9358d7b5f0
```
[Credentials]
aws_access_key_id = (access key)
aws_secret_access_key = (secret access key)
```

<<<<<<< HEAD
You need to set environment variables by specifying your Secret Key and Access Key.
=======
You need also to set environment variables by specifying your Secret Key and Access Key.
>>>>>>> 88dc297cc812ca780d9da103de66db9358d7b5f0

```
$ export AWS_ACCESS_KEY_ID= 'YOUR_AWS_API_KEY'
$ export AWS_SECRET_ACCESS_KEY= 'YOUR_AWS_API_SECRET_KEY'
```

# Role Variables
**List of Variables**
```

```

# Ansible Imagination Dev Site
To help make the roles reusable and easily updated, the variables were placed in the main site.yml file for configuring all of the aspects from the network, bastion, and RDS.

*Site.yml*
```
---
- name: Deploy RDS Infratructure
  hosts: localhost
  connection: local
  # become: yes
  gather_facts: false
  vars:

# Global AWS Variables
    region: eu-central-1
    vpc_name: dev

 # VPC Variables
    vpc_cidr: 10.5.0.0/16
    public_subnet_1_cidr: 10.5.0.0/24
    public_subnet_1_az: eu-central-1a
    private_subnet_1_cidr: 10.5.2.0/24
    private_subnet_1_az: eu-central-1b
    private_subnet_2_cidr: 10.5.3.0/24
    private_subnet_2_az: eu-central-1c

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
<<<<<<< HEAD
  - aws-network
  - aws-bastion
  - aws-rds
=======
   - vpc
   - bastion
   - natclient
   - rds

- hosts: natclient
  become: yes

  roles:
  - common
  - mysql-client
>>>>>>> 88dc297cc812ca780d9da103de66db9358d7b5f0
```

# Building AWS-VPC

– Build VPC
– Build Subnets
  – Public subnet to access the environment
  – Two Private subnets for internal traffic. Two because the RDS Subnet group requires two for redundancy.
– Internet Gateway for the VPC
– NAT Gateway
– Security groups to allow specific traffic into specific instances

## Building the MySQL-Client
```

```

<<<<<<< HEAD
## Connecting to a DB Instance Running the MySQL Database Engine
```
$ mysql -h myinstance.123456789012.eu-west-1.rds.amazonaws.com -P 3306 -u mymasteruser -p
=======

# Connect to RDS through EC2 > Bastion
For example, on Linux or OSX

```
$ ssh -L 3306:myinstance.123456789012.us-east-1.rds.amazonaws.com:3306  52.250.250.25
>>>>>>> 88dc297cc812ca780d9da103de66db9358d7b5f0
```
You can easily set up this tunnel every time you log into your remote EC2 instance and log into it with whatever name you prefer:

<<<<<<< HEAD
## Conclusion:
=======
Modify this to ssh.cfg:

```
Host 10.5.*
	User ubuntu
	ProxyCommand ssh ubuntu@54.93.88.15 nc %h %p

Host 54.93.88.15
	Hostname 54.93.88.15
	User ubuntu
	ControlMaster   auto
	ControlPath     ~/.ssh/mux-%r@%h:%p
	ControlPersist  15m
	IdentityFile ~/.ssh/imagination-key.pem
```
Then, just:

```
$ ssh -L 3306:dev-rds.cj3xloa8ykzj.eu-central-1.rds.amazonaws.com:3306 10.5.2.57 -F sshd.cfg
```

And you can then access your remote MySQL server as if it was running locally:
```
$ mysql -h dev-rds.cj3xloa8ykzj.eu-central-1.rds.amazonaws.com -P 3306 -u root -p
```
>>>>>>> 88dc297cc812ca780d9da103de66db9358d7b5f0

**Connecting to a DB Instance Running the MySQL Database Engine**
```
$ mysql -h myinstance.123456789012.eu-west-1.rds.amazonaws.com -P 3306 -u mymasteruser -p
```

## Issues:
- If you're using Ansible >2.2.0, you can set the ansible_python_interpreter configuration option to /usr/bin/python3:
```
ansible my_ubuntu_host -m ping -e 'ansible_python_interpreter=/usr/bin/python3'
```
https://docs.ansible.com/ansible/latest/python_3_support.html

- decided to upgrade to the latest version of ansible (2.3.0.0). Then I created a...

```
group_vars/all

..file and added...

ansible_python_interpreter: /usr/bin/python3
```

# Sources:
- https://blog.scottlowe.org/2015/12/11/using-ssh-multiplexing/
- https://www.cyberciti.biz/faq/linux-unix-osx-bsd-ssh-multiplexing-to-speed-up-ssh-connections/
- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html
