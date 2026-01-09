# Ansible configuration management
Ansible configuration management is the practice of using Ansible (an open-source automation tool) to define, enforce, and maintain the desired state of systems—servers, network devices, cloud resources—consistently and repeatably.

## Ansible is:
- Agentless (managed nodes don't require software)
- SSH and WinRM are used
- YAML (human-readable) writing
- Designed for orchestration, automation, and configuration management

## Configuration management with Ansible means:
- Setting up packages
- Managing services
- Modifying configuration files
- Implementing security configurations
- Maintaining systems in a predetermined, ideal state

### Setup Jenkins Server
Ensure the Jenkins server infrastructure is properly deployed and configured from the [previous project decumentation](https://github.com/Eiphie/Introduction-to-Jenkins)

### Install and configure ansible
```
sudo apt update
sudo apt install -y ansible
```

## Connect VS Code to Jenkins Server
### Local VS Code + GitHub
- Edit playbooks locally
- Push to GitHub
- Jenkins pulls code
```
git --version
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```
Clone ansible-config-mgt Repository
```
https://github.com/<your-username>/ansible-config-mgt.git
```


## Set up a Jenkins job to archive the repository for each change.
### Create Jenkins Freestyle Job
- Jenkins → New Item
- Name: ansible
- Type: Freestyle project
- Click OK

### Direct Job to the GitHub Repo
```
https://github.com/<your-username>/ansible-config-mgt.git
*/main
```
### Enable GitHub Webhook Trigger and Archive All Repository Files

## Configure GitHub Webhook
GitHub repo → Settings → Webhooks → Add webhook
```
http://<jenkins-ip>:8080/github-webhook/
```

Verify Artifacts Saved
```
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```

## Begin Ansible Development on your local machine
```
cd ansible-config-mgt
git checkout -b feature/prj-145-ansible-structure
```
Create folders;
- playbooks
- inventory

Create files;
- playbooks/common.yml
- inventory/dev.ini
- inventory/staging.ini
- inventory/uat.ini
- inventory/prod.ini

## Set Up Ansible Inventory
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/your-key.pem
```
Update inventory/dev.ini
```
[lb]
<load-balancer-ip> ansible_user=ubuntu

[webservers]
<web1-ip> ansible_user=ubuntu
<web2-ip> ansible_user=ubuntu
```

Create common.yml
```
---
# Playbook: common.yml
# Purpose: Configure tasks common to all Ubuntu servers

- name: Update webservers and Load Balancer
  hosts: webservers,lb
  become: yes
  tasks:
    - name: Update apt repository cache
      apt:
        update_cache: yes

    - name: Ensure Wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```
Test Playbook connectivity
```
ansible-playbook -i inventory/dev playbooks/common.yml --check
```











