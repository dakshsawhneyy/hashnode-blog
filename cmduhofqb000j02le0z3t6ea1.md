---
title: "Creation of Secure Web Server Using Ansible"
datePublished: Sat Aug 02 2025 16:50:59 GMT+0000 (Coordinated Universal Time)
cuid: cmduhofqb000j02le0z3t6ea1
slug: creation-of-secure-web-server-using-ansible
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1754153595350/4443e51e-26d4-45f7-a953-f214e3cc42cb.png
tags: web-servers, ansible, automation, devops, web-server, ansible-playbook, ad-hoc

---

**Deploying a secure and scalable web server shouldn't be a mystery or a manual process.**

Whether you're a DevOps enthusiast, a student building cloud skills, or an engineer tired of repetitive tasks, automation is your best friend.

In this walkthrough, we'll use Ansible to automate the creation of an AWS EC2 instance and set up a secure web server running Nginx - all without clicking through the AWS console or manually configuring a thing. If you’ve ever thought *“****there must be a better way****”*, this guide is for you.

### 🟢 Why This Is Needed:

> Manually launching EC2 instances and configuring them can quickly become error-prone and inefficient, especially when you're managing multiple environments or need to redeploy often. With Ansible, you can turn that entire workflow → provisioning infrastructure, securing users, installing services, and even setting up firewall rules into repeatable code. This not only saves time but ensures your environments are consistent, secure, and ready for scaling.

# Let’s start with our Automated Web Server

## Creation of EC2 Instance

### Installing Dependencies

```bash
sudo apt install python3-boto3
ansible-galaxy collection install amazon.aws
```

### Set up Vault

```yaml
openssl rand -base64 2048 > vault.pass
ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass
```

### EC2 Creation Playbook

```yaml
-
  name: Create EC2 instance with termination protection turned on
  hosts: localhost
  connection: local # Tells ansible it runs on same device
  tasks:
    - name: Launch EC2 instance
      amazon.aws.ec2_instance:
        name: "my-ec2-instance"
        key_name: "general-key-pair" # key present in aws
        # vpc_subnet_id: "subnet-5ca1ab1e"
        instance_type: "t2.micro"
        security_group: default
        region: "ap-south-1"
        aws_access_key: "{{aws_access_key}}"
        aws_secret_key: "{{aws_secret_key}}" # From vault as defined
        image_id: "ami-0e35ddab05955cf57"
        network:
          assign_public_ip: true
```

### Run the playbook

```bash
ansible-playbook ec2_insatnce.yml --vault-password-file vault.pass
```

### Create Folder Structure

```bash
ansible-galaxy init secure-web-server-project
```

### Create Inventory and Main Playbook

```yaml
[all]
ubuntu ansible_host=52.66.244.170 ansible_user=ubuntu 

[all:vars]
ansible_ssh_key=/home/dakshsawhneyy/dakshsawhneyy/general-key-pair.pem
```

```yaml
-
  name: Secure Web Server Setup
  hosts: all
  become: true
  roles:
    - secure-web-server
```

Run tasks inside tasks/main.yml

### Create User

```yaml
- name: Create deploy user
	ansible.builtin.user:
		name: daksh
		shell: /bin/bash  # specify to use which shell
		groups: sudo  # add it to root user permission group
		create_home: true  #create home directory
		state: present
```

`We have created user because ec2 has default ubuntu so to login using that user, we need key which we going to generate inside files/`

### Now creating SSH key to SSH into deploy user

Inside /files

```bash
ssh-keygen -t rsa -b 2048 -f deploy_key
```

### Create .ssh directory

```yaml
- name: Create .ssh directory
  ansible.builtin.file:
    path: /home/daksh/.ssh
    state: directory
    owner: daksh
    group: daksh
    mode: '0700'  #Make it super secure
```

### Adding SSH\_Key to user

```yml
- name: Copy public key to the server
  ansible.builtin.authorized_key:
    user: daksh
    state: present
    key: "{{ lookup('file', 'files/daksh_key.pub') }}"
```

`This is very useful as it reduces manual effort to copy ssh into home directory of user to ssh into that by automating it and copying it by himself, just need public key file and user name`

Now we can easily ssh into the instance

```bash
ssh -i secure-web-server/files/daksh_key daksh@52.66.244.170
```

## Now Updating system and Installing NginX

```yaml
- name: Update system
  ansible.builtin.apt:
    update_cache: yes
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present
- name: Start Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes
```

### Copy index.html to /var/www/html

```yaml
- name: copy index.html to server
  ansible.builtin.copy:
    src: files/index.html
    dest: /var/www/html/index.html
```

### Allowing ports using UFW

```yaml
- name: Allow HTTP port 80
  ansible.builtin.ufw:
    rule: allow
    port: 80
    proto: tcp
- name: Allow HTTP port 443
  ansible.builtin.ufw:
    rule: allow
    port: 443
    proto: tcp
- name: enable ufw firewall
  ansible.builtin.ufw:
    state: enabled
```

### Add a CronJob for scheduling updates

```yaml
- name: Schedule daily update with cron
  ansible.builtin.cron:
    name: "Daily update"
    minute: 0
    hour: 2
    job: "apt-get update -y && apt-get -y upgrade"
    user: root
```

## Delete or Terminate EC2 - Playbook

```yaml
-
  name: Create EC2 instance with termination protection turned on
  hosts: localhost
  connection: local # Tells ansible it runs on same device
  tasks:
    - name: Launch EC2 instance
      amazon.aws.ec2_instance:
        name: "my-ec2-instance"
        key_name: "general-key-pair" # key present in aws
        # vpc_subnet_id: "subnet-5ca1ab1e"
        instance_type: "t2.micro"
        security_group: default
        region: "ap-south-1"
        aws_access_key: "{{aws_access_key}}"
        aws_secret_key: "{{aws_secret_key}}" # From vault as defined
        image_id: "ami-0e35ddab05955cf57"
        network:
          assign_public_ip: true
```