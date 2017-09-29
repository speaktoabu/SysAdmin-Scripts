# Quick Introduction To Ansible
Ansible is configuration Managment Tool. Besides resource provisioning and configuration management, Ansible can also orchestrate complex sequences of events like rolling upgrades and zero-downtime provisioning in a simple or multi-tier application environment. The power of Ansible is not limited to managing servers: it also can manage network switches, firewalls, and load balancers. Ansible has been designed to work seamlessly within cloud environments like AWS, VMWare, and Microsoft Azure.


### Ansible’s unique feature set:

** Based on an agent-less architecture (unlike Chef or Puppet).
** Accessed mostly through SSH (it also has local and paramiko modes).
**  No custom security infrastructure is required.\n
** Configurations (playbooks, modules etc.) written in the easy-to-use YML format.
** Shipped with more than 250 built-in modules.
** Full configuration management, orchestration, and deployment capability.
** Ansible interacts with its clients either through playbooks or a command-line tool.

### Ansible Architecture

### Ansible and AWS

Ansible runs as a server on just about anything: even a humble PC or laptop. It has an inventory of hosts, modules, and playbooks that define various automation tasks. Individual modules are pushed to manage entities via SSH and, once a result is returned, the modules are deleted – which is why server-based agent installations are unnecessary.

### Installation

There are multiple ways to install Ansible. You can use yum or apt-get from a Linux repository, git checkout, or PIP. Here is a sample installation command, run through yum:

``` 
#yum install Ansible

===========================================================
Package                    Arch            Version                Repository                  Size
===========================================================
Installing:
ansible                    noarch          1.9.2-1.el6            epel                       1.7 M
Installing for dependencies:
PyYAML                     x86_64          3.10-3.1.el6           public_ol6_latest          157 k
libyaml                    x86_64          0.1.3-4.el6_6          public_ol6_latest           51 k
python-babel               noarch          0.9.4-5.1.el6          public_ol6_latest          1.4 M
python-crypto2.6           x86_64          2.6.1-2.el6            epel                       513 k
python-httplib2            noarch          0.7.7-1.el6            epel                        70 k
python-jinja2              x86_64          2.2.1-2.el6_5          public_ol6_latest          465 k
python-keyczar             noarch          0.71c-1.el6            epel                       219 k
python-pyasn1              noarch          0.0.12a-1.el6          public_ol6_latest           70 k
python-simplejson          x86_64          2.0.9-3.1.el6          public_ol6_latest          126 k

Transaction Summary
===========================================================
Install      10 Package(s)

```

As you will notice, this has installed Ansible-1.9.2 on the server. The default host inventory file should be located at /etc/ansible/hosts. You can configure your managed servers in this file. For example, a group called test-hosts might be configured as follows:

```
[test-hosts]
3.3.86.253
3.3.86.254
```

Ansible assumes you have SSH access from the Ansible server to the machines behind the above two IP addresses. To run a sample Ansible command, you can run the following:

```
[root@ansible opt]# ansible -m ping test-hosts --ask-pass -u "ansibleadmin"
SSH password:
3.3.86.254 | success >> {
"changed": false,
"ping": "pong"
}
3.3.86.253 | success >> {
"changed": false,
"ping": "pong"
}
[root@ansible opt]#
```
As you can see, Ansible is able to communicate with the listed servers without installing any agents as such. Assuming you have generated and placed ssh-keys in their required locations, everything works over SSH.

# Ansible components

### Inventory
The “inventory” is a configuration file where you define the host information. In the above /etc/ansible/hosts example, we declared two servers under test-hosts.

### Playbooks
In most cases – especially in enterprise environments – you should use Ansible playbooks. A playbook is where you define how to apply policies, declare configurations, orchestrate steps and launch tasks either synchronously or asynchronously on your servers. Each playbook is composed of one or more “plays”. Playbooks are normally maintained and managed in a version control system like Git. They are expressed in YAML (Yet Another Markup Language).

### Plays
Playbooks contain plays. Plays are essentially groups of tasks that are performed on defined hosts to enforce your defined functions. Each play must specify a host or group of hosts. For example, using:

```
 – hosts: all

```
…we specify all hosts. Note that YML files are very sensitive to white spaces, so be careful!

### Tasks
Tasks are actions carried out by playbooks. One example of a task in an Apache playbook is:

```
- name: Install Apache httpd
A task definition can contain modules such as yum, git, service, and copy.

```
### Roles
A role is the Ansible way of bundling automation content and making it reusable. Roles are organizational components that can be assigned to a set of hosts to organize tasks. Therefore, instead of creating a monolithic playbook, we can create multiple roles, with each role assigned to complete a unit of work. For example: a webserver role can be defined to install Apache and Varnish on a specified group of servers.

### Handlers
Handlers are similar to tasks except that a handler will be executed only when it is called by an event. For example, a handler that will start the httpd service after a task installed httpd. The handler is called by the [notify] directive. Important: the name of the notify directive and the handler must be the same.

### Templates
Templates files are based on Python’s Jinja2 template engine and have a .j2 extension. You can, if you need, place contents of your index.html file into a template file. But the real power of these files comes when you use variables. You can use Ansible’s [facts] and even call custom variables in these template files.

### Variables
As the name suggests, you can include custom-made variables in your playbooks. Variables can be defined in five different ways:

1. Variables defined in the play under vars_files attribute:

```
vars_files:

- "/path/to/var/file"
```
2. Variables defined in <role>/vars/apache-install.yml

3. Variables passed through the command line:

```
# ansible-playbook apache-install.yml -e "http-port=80"
```

4. Variables defined in the play under vars
```
vars: http_port: 80
```
5. Variables defined in group_vars/ directory
```
Sample Playbook:

## PLAYBOOK TO INSTALL AND CONFIGURE APACHE HTTP ON Servers
- hosts: all
  tasks:
   - name: Install Apache httpd
     yum: pkg=httpd state=installed
     notify:
       - Start Httpd
  handlers:
    - name: Start httpd
      service: name=httpd state=started
```
You run the playbook from your Ansible Server. The following command has been run from within the path where our apache-playbook.yml playbook is stored:

```
# ansible-playbook apache-install.yml  --ask-pass -u "ansibleadmin"
```
### Ansible and AWS: provisioning and Installation

Let’s try to provision an AWS EC2 instance using both the Ansible EC2 module and a playbook. An full example is beyond the scope of this document, however there’s plenty of great documentation available. The ec2_module documentation can be seen here.

### Step-1: Install python-boto on your Ansible host:

```
#yum install python-boto
```
### Step-2:  Install argparse (in case you need it):

```		
#yum install python-argparse.noarch
```
Create AWS_ACCESS_KEY and AWS_SECRET_ACCESS_KEY environment variables (either export in a shell or place in your ~/.bashrc file)

### Step-3: Add a local inventory to the /etc/ansible/hosts file:

```
[local]
localhost
```
### Step-4: Run the following command (if you are running in adhoc mode):
```

# ansible localhost -m ec2 -a "image=ami-d44b4286 ec2_region=ap-southeast-1 instance_type=m3.medium count=1  keypair=ansible-key group=ansible-ws  wait=yes" -c  local

```
As I mentioned, you can also do the same thing through a playbook:

```
- hosts: localhost
  connection: local
  gather_facts: False
  tasks:
     - name: Provision a set of instances
      ec2:
         key_name: ansible-key
         group: ansible-ws
         instance_type: m3.medium
         ec2_region: ap-southeast-1
         image: "ami-d44b4286"
         wait: true
         count: 1
         instance_tags:
            Name: Demo
      register: ec2

```
### Conclusion

If you stayed with me so far, you should now have a fair idea of how Ansible and AWS can work together and especially how well it integrates with Amazon’s EC2. Ansible provides a great IT automation and orchestration tool for the cloud environment, and with so much portability in its command syntax, it’s easy to create either playbooks or out-of-the-box modules.
