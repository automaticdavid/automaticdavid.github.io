Title: Configure an SSH proxy for Ansible Tower
Category: Tower
Date: 2019-04-19



In my lab, some of the VMs I provision are only accessible through a jump host. This means Tower needs to connect to the jump host first then to the target VMs in order to run playbooks. 

One way to enable Tower to reach isolated nodes is to use a dedicated ansible host, that is both accessible by Tower and able to reach the target network. This is called an isolated node and the way to set it up is described at: [6.3. Isolated Instance Groups](http://docs.ansible.com/ansible-tower/latest/html/administration/clustering.html#isolated-instance-groups)

Isolated nodes will have Ansible installed on them, and sometimes that's not possible on the jump host. But you can also configure Tower to use the jump host as an ssh proxy: Tower will ssh to the jump host and from it access the targets. This is documented at: [22.6. Setting up a jump host to use with Tower](http://docs.ansible.com/ansible-tower/latest/html/administration/tipsandtricks.html#setting-up-a-jump-host-to-use-with-tower)

I had some problems getting it to work so decided to document what I did. 


### 1. Disable PRoot
As described in the "Tower Tips and Tricks" official documentation page.

### 2. Add SSH connection arguments at the inventory level
I added the following variables at the inventory level:

```
ansible_connection: ssh
ansible_ssh_common_args: '-o ProxyCommand="ssh -W %h:%p -q jump-host-user@jump-host.example.com"'
```

Defining those at the inventory level means that only the hosts of that inventory will be using the jump host. And it also means you don't have to edit SSH configuration files on the Tower host.

### 3. Add the SSH private key of the jump host to Tower
Tower will use it to ssh to the jump host: copy the `id_rsa` file of the jump host on the tower host into `/var/lib/awx/.ssh/`
Make sure it has the following user, group and permissions:

```
-rw-------. 1 awx awx 1679 Apr 30 16:37 id_rsa
```

You can also put the key in another directory and specify its path using the -i switch of the ProxyCommand.

### 4. Add the credentials for the target machine
Once connected to the jump host, Tower needs to connect to the target machine. 
I defined a Tower Machine Credential, in my case for the root user (of the target node) using a password to connect.
You can also use an ssh key if your target nodes support it, as with any machine credentials.

### 5. Define the Job Template
The tower template needs to have: 
 
* The inventory used in 2
* The machine credential defined in 4.


And it works ! Thanks to [@victorock](https://github.com/victorock) ;)


