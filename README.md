# Ansible configuration management

Ansible configuration management is the practice of using Ansible (an open-source automation tool) to define, enforce, and maintain the desired state of systems—servers, network devices, cloud resources—consistently and repeatably.

## Ansible is

- Agentless (managed nodes don't require software)
- SSH and WinRM are used
- YAML (human-readable) writing
- Designed for orchestration, automation, and configuration management

## Configuration management with Ansible means

- Setting up packages
- Managing services
- Modifying configuration files
- Implementing security configurations
- Maintaining systems in a predetermined, ideal state

```sh
git clone https://github.com/<your-username>/ansible-config-mgt.git
```

```sh
cd ansible-config-mgt
git checkout -b dev
```

## Begin Ansible Development on your local machine

- Install ansible

```sh
brew install ansible # This is for a macos. Use the equivalent command for other operating systems
```

- Create playbooks and inventory

```sh
├── inventory
│   ├── dev.ini
│   ├── prod.ini
│   ├── staging.ini
│   └── uat.ini
└── playbooks
    └── common.yml
```

- update `playbooks/common.yml`

```yml
---
- name: update webservers
  hosts: webservers
  become: true
  remote_user: ubuntu
  tasks:
    - name: Update apt repo
      apt:
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  become: true
  remote_user: ubuntu
  tasks:
    - name: Update apt repo
      apt:
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

- update `inventory/dev.ini`

```sh
[webservers]
web1 ansible_host=<EC2_PUBLIC_IP> ansible_ssh_private_key_file=<PATH_TO_EC2_PEM_FILE>
web2 ansible_host=<EC2_PUBLIC_IP> ansible_ssh_private_key_file=<PATH_TO_EC2_PEM_FILE>
[lb]
lb ansible_host=<EC2_PUBLIC_IP> ansible_ssh_private_key_file=<PATH_TO_EC2_PEM_FILE>
```

## Test Playbook connectivity

- Create SSH security group rule for your local IP under each `webservers` and `lb` security group.
- Test the connection

```sh
ansible-playbook -i inventory/dev playbooks/common.yml --check
```

## Run Ansible Playbook

```sh
ansible-playbook -i inventory/dev playbooks/common.yml
```

### Verify Ansible successfully installed wireshark in each server

![Screenshot 2026-01-10 at 22 42 54](https://github.com/user-attachments/assets/54f71d23-7197-466f-8042-0e7c061f86c6)

![Screenshot 2026-01-10 at 22 42 20](https://github.com/user-attachments/assets/fc935723-f94c-460b-b072-cddb4f845d69)



## Running Ansible Playbook from Bastion Server

### Setup Jenkins Server

Ensure the Jenkins server infrastructure is properly deployed and configured from the [previous project documentation](https://github.com/Eiphie/Introduction-to-Jenkins)

### Install and configure ansible on the existing Jenkins EC2 instance

```sh
sudo apt update
sudo apt install -y ansible
```

## Set up a Jenkins job to archive the repository for each change

### Create Jenkins Freestyle Job

- Jenkins → New Item
- Name: ansible
- Type: Freestyle project
- Direct Job to the GitHub Repo
- Click OK
- Configure Jenkins to archive build artifacts

### Enable GitHub Webhook Trigger and Archive All Repository Files

- GitHub repo → Settings → Webhooks → Add webhook

```sh
http://<jenkins-ip>:8080/github-webhook/
```

- Clone ansible-config-mgt repository to local
- Edit playbooks locally
- Push to GitHub
- Jenkins pulls code through the webhook configured above
- Verify Artifacts Saved
```
ssh -i <PATH_TO_EC2_PEM_FILE> ubuntu@<jenkins-ip>
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
<img width="1497" height="760" alt="Screenshot 2026-01-08 at 21 03 09" src="https://github.com/user-attachments/assets/aff507fb-1726-4967-b350-7ab4928865c7" />

<img width="1012" height="524" alt="Screenshot 2026-01-10 at 23 56 26" src="https://github.com/user-attachments/assets/3303c2b5-b293-4119-9b54-0f940c52c339" />

### Run playbook

```sh
cd /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
ansible-playbook -i inventory/dev playbooks/common.yml
```

