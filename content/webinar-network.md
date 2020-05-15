Title: Using Ansible to manage ios devices
Category: Ansible
Date: 2020-05-15
Lang: en
Slug: webinar-05-2020

Here's a small write up on the demo I did for our "Beyond Linux, Automate all the things" webinar.

The code I used is at:  
[https://github.com/automaticdavid/webinar_052020](https://github.com/automaticdavid/webinar_052020)

The security workshop can be accessed at:  
[https://ansible.github.io/workshops/](https://ansible.github.io/workshops/)

The webinar replay (in french) is at:  
[L’automatisation au delà des systèmes Linux](https://www.redhat.com/en/events/webinar/red-hat-ansible-automation-platform-cloud-reseau-iot-lautomatisation-au-dela-des-systemes-linux)


## 1- Using facts & ios_command

![webinar1_demo_1]({static}/images/webinar1_demo_1.gif)

In this first part I showed the inventory used and how we can use inventory groups. In the `gather_ios_data.yml` playbook, note that I'm using `gather_facts: yes` for network devices which is new with Ansible 2.9. This allows me to use the `ansible_net_*` directly in my play!

I then show how to use `ios_command` to run commands and register the output. I use this to diplay snmp settings on my 4 routers.

The `config_routers.yml` playbook allows me to change those snmp settings. Note that when I run it 2 times, the second time does nothing: `changed=0` is displayed and all the nodes are green OK. That's **indempotency**: Ansible doesn't do anything if we're already in the state we want. 

I then log on to the device rtr1 and make a manual change. When I run the playbook again, Ansible corrects that one thing only.


## 2- Ressource modules & Source of truth. 

![webinar1_demo_2]({static}/images/webinar1_demo_2.gif)

To take this a bit further, I then use the `ios_facts` modules, and with Ansible 2.9 this comes with a powerfull feature: Ansible is able to extract information from my already configured devices and produce structured data. This is shown when I execute `gather_ios_resource.yml` and display the L3 interfaces configuration as a YAML list of interfaces:

![webinar1_demo_2]({static}/images/webinar1_demo2_list.png)

Each list element (starting with a dash) is a dictionnary that contains keys: in the example "ipv4" and "name". The "ipv4" key contains a list, here with only one element. This element is also a dictionary, with only one key: "address".  
This makes it very easy to read and modify configuration values. 

As a matter of fact the playbook also writes these values in a `host_vars` directory. In the demo I then make a change in these values and run the `config_l3.yml` playbook in check mode first: this tells us what Ansible would change but doesn't actually change anything. Then I apply the change by running in normal mode. When I run the `gather_ios_resource.yml` again, I can see that the change was made on rtr1. The local source of truth is not updated: it's already up to date.  

What we see here is that Ansible is able to produce structured configuration data (with the `ios_facts` module) and then consume this data to apply configuration changes (with the `ios_l3_interfaces` module). Since the configuration is in YAML, it can be managed as code in git, with the inventory itself. This means that you have full tracability of changes, you can re-apply any version of your infrastrcture or manage it with a gitflow. 

As always with Ansible, it starts with some very simple concepts but delivers powerfull ways of doing things. 

Find out more about ressource modules in this Ansible Automates talk by [@IPvSean](https://twitter.com/ipvsean?lang=en):  
[https://www.ansible.com/2019-ansible-network-automation-resource-modules](https://www.ansible.com/2019-ansible-network-automation-resource-modules) 

   




