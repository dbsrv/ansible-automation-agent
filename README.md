# Install Automation Agent using Ansible

This document describes how to use Ansible to install **MongoDB Ops Manager Automation Agent**. Unlike Chef or Puppet, Ansible does not require an agent on a node, instead it uses SSH to access a node and execute tasks remotely. Simply assign a host to install and run Ansible, then provide a list of remote servers (`hosts`) where Automation Agent to be installed.

For more on Ansible, read [How Ansible Works](https://www.ansible.com/how-ansible-works).

*Disclaimer*: This playbook is designed for RPM distribution of Linux only (RedHat/Fedora/CentOS)

### Install Ansible

You'll need a dedicated host to run Ansible. You can use the host where Ops Manager is installed. Follow [installation guide](http://docs.ansible.com/ansible/intro_installation.html#latest-release-via-yum), or these two easy steps:

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

**1)** Edit list of remote `hosts`, e.g.
```
[servers]
server1
server2
```
(replace `server1`, `server2`, ...`serverN`, with FQDN or IP address of remote servers)

**2)** Edit default variables in `roles/install/default/main.yml`. Variables can be obtained in Ops Manager > Settings > Agents.
```
---

## Automation Agent file path
automation_agent_pkg: http://opsmanager:8080/download/agent/automation/mongodb-mms-automation-agent-manager-latest.x86_64.rpm

## Automation Agent configuration (required)
mms_group_id:         "574735e6ec2ea11b4a000000"
mms_api_key:          "206e2ec9bf362ebcb5f4ec46fe300000"
mms_base_url:         "http://opsmanager:8080"

## Automation Agent configuration (optional)
mms_config_backup:	  "/var/lib/mongodb-mms-automation/mms-cluster-config-backup.json"
log_file:             "/var/log/mongodb-mms-automation/automation-agent.log"
log_level:            "INFO"
max_log_files:        10
max_log_file_size:    268435456
```

**3)** (Optional but recommended) Check status on remote `hosts`  
Automation Agent should be not installed or not running. Run this [ad-hoc](http://docs.ansible.com/ansible/intro_adhoc.html) Ansible command:
```bash
ansible all -i hosts -a "service mongodb-mms-automation-agent status" --user johnny
```
Results should be either:
```
server1 | FAILED | rc=3 >>
mongodb-mms-automation-agent: unrecognized service
```
Or
```
server1 | FAILED | rc=3 >>
mongodb-mms-automation-agent is NOT running
```
If the result shows Automation Agent is already running on a remote host. You should take it off the list of `hosts` and inspect it manually. It may be already automated by Ops Manager.

**4)** Run playbook
```bash
ansible-playbook -i hosts install.yml --user johnny
```
Logs are kept in `log/playbook.log`  

*Tip*: for troubleshooting purpose, increase verbosity with `-v`, up to `-vvvv`. E.g
```
ansible-playbook -i hosts install.yml --user johnny -vvvv
```

### Ansible Config
This playbook uses customized `ansible.cfg`. Feel free to modify.
```
[defaults]
host_key_checking =   False
retry_files_enabled = False
log_path =            ./log/playbook.log
ask_pass =            True
command_warnings =    False

[privilege_escalation]
become =              True
become_ask_pass =     True
```
[Description and full list of ansible config](http://docs.ansible.com/ansible/intro_configuration.html)

### Example
```bash
johnny@ansible:~/ansible$ ansible-playbook -i hosts install.yml --user johnny
SSH password: 
SUDO password[defaults to SSH password]: 

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [server1]

TASK [install : Download automation agent] *************************************
changed: [server1]

TASK [install : Install automation agent] **************************************
changed: [server1]

TASK [install : Set directory attribute where config file is kept] *************
changed: [server1]

TASK [install : Create Automation Agent config file] ***************************
changed: [server1]

TASK [install : Configure Automation Agent config file] ************************
changed: [server1]

TASK [install : Start automation agent, and enable start on boot] **************
changed: [server1]

PLAY RECAP *********************************************************************
server1            : ok=7    changed=6    unreachable=0    failed=0   
```

### Reference
[Install the Automation Agent with rpm Packages](https://docs.cloud.mongodb.com/tutorial/install-automation-agent-with-rpm-package/)
