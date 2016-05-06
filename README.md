# ansible-opsmanager

Use Ansible to install MongoDB Ops Manager **Automation Agent** onto nodes. Unlike Chef or Puppet, Ansible does not require an agent on a node. Simply assign a host to run Ansible, then provide Ansible a list of nodes to install Automation Agent.

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

