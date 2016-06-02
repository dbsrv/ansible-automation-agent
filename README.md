# Install Automation Agent using Ansible

This document describes how to use Ansible to install MongoDB Ops Manager **Automation Agent**. Unlike Chef or Puppet, Ansible does not require an agent on a node, instead it uses SSH to access a node and execute tasks remotely. Simply assign a host to install and run Ansible, then provide a list of nodes (**hosts** file) for Automation Agent installation.

For more on Ansible, read [How Ansible Works](https://www.ansible.com/how-ansible-works).

### Install Ansible

[Installation on RedHat/Fedora/CentOS:](http://docs.ansible.com/ansible/intro_installation.html#latest-release-via-yum)

1) Configure EPEL
```bash
## RHEL/CentOS 6 64-Bit ##
wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm
```
2) Install Ansible
```bash
sudo yum install ansible
```

### Usage

1) Edit list of `hosts`, e.g.
```
[servers]
server1
server2
```

2) Edit default variables in `roles/install/default/main.yml`
```
---

## Automation Agent file path
automation_agent_pkg: http://opsmanager:8080/download/agent/automation/mongodb-mms-automation-agent-manager-latest.x86_64.rpm

## Automation Agent configuration (required)
mms_group_id:		      "574735e6ec2ea11b4a000000"
mms_api_key:		      "206e2ec9bf362ebcb5f4ec46fe300000"
mms_base_url:		      "http://opsmanager:8080"

## Automation Agent configuration (optional)
mms_config_backup:	  "/var/lib/mongodb-mms-automation/mms-cluster-config-backup.json"
log_file:             "/var/log/mongodb-mms-automation/automation-agent.log"
log_level:		        "INFO"
max_log_files:		    10
max_log_file_size:	  268435456
```

3) Run playbook
```bash
ansible-playbook -i hosts install.yml --user johnny
```

### Ansible Config
This playbook uses customized `ansible.cfg`. Feel free to modify.
```
[defaults]
host_key_checking =   False
retry_files_enabled = False
log_path =            ./log/playbook.log
ask_pass =            True
become =              True
become_ask_pass =     True
```
[Full list of ansible config](http://docs.ansible.com/ansible/intro_configuration.html)
