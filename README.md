# Install Automation Agent using Ansible

This document describes how to use Ansible to install **MongoDB Ops Manager Automation Agent**. Unlike Chef or Puppet, Ansible does not require an agent on a node, instead it uses SSH to access a node and execute tasks remotely. Simply assign a host to install and run Ansible, then provide a list of remote servers (`hosts`) where Automation Agent to be installed.

For more on Ansible, read [How Ansible Works](https://www.ansible.com/how-ansible-works).

*Disclaimer*: This playbook is designed for RPM distribution of Linux only (RedHat/Fedora/CentOS)

## Usage

### Install Ansible
You'll need a dedicated host to run Ansible (you can use the host where Ops Manager is installed). Follow [installation guide](http://docs.ansible.com/ansible/intro_installation.html#latest-release-via-yum), or these two easy steps:

**1)** Configure EPEL
```bash
## RHEL/CentOS ##
sudo rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```
**2)** Install Ansible
```bash
sudo yum install -y ansible
```

Alternative:  
If the host is in prod and does not have internet access, you can install Ansible on a (Linux) jumpbox. Or alternatively, download zipped Ansible package to the jumpbox, then transfer to the host in prod:  
<url>

After transfer to the host in prod, do a yum localinstall:
```bash
sudo mkdir ansiblepkg
sudo tar -zxvf ansiblepkg.tar.gz ansiblepkg/
sudo yum localinstall ansiblepkg/*.rpm
```

### Checkout repo
**1)** Prepare [Git](http://rogerdudler.github.io/git-guide/) and checkout this repository  
Install Git if it is not yet installed
```
sudo yum install git
```
```
git init
git clone https://github.com/dbsrv/ansible-automation-agent.git
cd ansible-automation-agent
```
If the host that runs Ansible has no http access, you can checkout the repository at another server, zip it, and transfer back to the host.  

### Run playbook
**1)** Edit list of remote `hosts`, e.g.
```
vim hosts
```
```
[servers]
server1
server2
```
(replace `server1`, `server2`, ...`serverN`, with FQDN or IP address of remote servers)

You can also use [patterns](http://docs.ansible.com/ansible/intro_patterns.html) to add multiple servers, e.g.
```
[servers]
server[1-10]
```
This will add `server1` thru `server10`.

**2)** Edit default variables in `roles/install/defaults/main.yml`. Variables can be obtained in Ops Manager > Settings > Agents.
```
vim roles/install/defaults/main.yml
```
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

**Check status on Automation Agent**:  
Automation Agent should be not installed or not running. Run this [ad-hoc](http://docs.ansible.com/ansible/intro_adhoc.html) Ansible command:
```bash
ansible all -i hosts -a "service mongodb-mms-automation-agent status" --user johnny
(replace `johnny` with your username) 
```
Results should be either:
```
server1 | FAILED | rc=3 >>
mongodb-mms-automation-agent: unrecognized service
                   Or
mongodb-mms-automation-agent is NOT running
```
If the result shows Automation Agent is already running on a remote host. You should take it off the list of `hosts` and inspect it manually. It may be already automated by Ops Manager.

**Check status on SELinux**:  
```bash
ansible all -i hosts -a "sestatus" --user johnny
```
If SELinux is enabled, *preflight* role would result in error. You can skip *preflight* step when prompted:
```bash
Install/Update OpenSSL? [Y]: N
```
You would have to manually install/update OpenSSL on each host.

**4)** Run playbook
```bash
ansible-playbook -i hosts install.yml --user johnny
```
Enter SSH password, and SUDO password (press enter again to use same password as ssh). Choose whether to install or update OpenSSL (default Y = Yes)
```bash
SSH password: 
SUDO password[defaults to SSH password]: 
Install/Update OpenSSL? [Y]: 
```
### Example Output
```bash
johnny@server1:~/ansible$ ansible-playbook -i hosts install.yml --user johnny
SSH password: 
SUDO password[defaults to SSH password]: 
Install/Update OpenSSL? [Y]:

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [server1]

TASK [preflight : Collect status on OpenSSL] ***********************************
changed: [server1]

TASK [preflight : Perform Yum Update] ******************************************
ok: [server1]

TASK [preflight : Download OpenSSL (openssl-1.0.1e-42.el6.x86_64.rpm)] *********
ok: [server1]

TASK [preflight : Install/Upgrade OpenSSL] *************************************
ok: [server1]

TASK [preflight : Confirm OpenSSL is in version "1.0.1e"] **********************
ok: [server1]

TASK [install : Download automation agent] *************************************
changed: [server1]

TASK [install : Check for existing Automation Agent config file] ***************
ok: [server1]

TASK [install : Back up Automation Agent config file if exists] ****************
skipping: [server1]

TASK [install : Install automation agent] **************************************
changed: [server1]

TASK [install : Configure Automation Agent config file] ************************
changed: [server1]

TASK [install : Collect status on /data directory] *****************************
ok: [server1]

TASK [install : If /data exists, set ownership to 'mongod'] ********************
skipping: [server1]

TASK [install : If /data not exists, create directory and set ownership to 'mongod'] ***
changed: [server1]

TASK [install : Start Automation Agent, and enable start on boot] **************
changed: [server1]

TASK [install : Confirm Automation Agent is running] ***************************
ok: [server1]

PLAY RECAP *********************************************************************
server1            : ok=7    changed=4    unreachable=0    failed=0   
```

### Verify Installation
Verify installation with:
```bash
ansible all -i hosts -a "service mongodb-mms-automation-agent status" --user johnny
```
You should see successful messages:
```
server1 | SUCCESS | rc=0 >>
mongodb-mms-automation-agent is running
```

### Tips
Logs are kept in `log/playbook.log`  

If automation agent fails to start, refer to log file at Ops Manager console, or at  
`sudo tail -n 200 /var/log/mongodb-mms-automation/automation-agent-fatal.log`

For troubleshooting purpose, run commands `-v`, up to `-vvvv`, to increase verbosity. E.g
```
ansible-playbook -i hosts install.yml --user johnny -vvvv
```

### Ansible Config
This playbook uses customized `ansible.cfg`. Leave it as it is, or modify to fit your need.
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

### Reference
[How Ansible Works](https://www.ansible.com/how-ansible-works)  
[Install the Automation Agent with rpm Packages](https://docs.cloud.mongodb.com/tutorial/install-automation-agent-with-rpm-package/)
