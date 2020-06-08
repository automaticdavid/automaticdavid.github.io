Title: Interop Ansible & Cisco NSO
Category: Ansible
Date: 2020-06-05
Lang: en
Slug: nso-06-2020

Ansible is a powerful solution to manage network infrastructure. But sometimes there's already a [Cisco NSO](https://www.cisco.com/c/en/us/solutions/service-provider/solutions-cloud-providers/network-services-orchestrator-solutions.html) setup. The Ansible modules for NSO allow using the two solutions side by side, automating NSO with Ansible. 

The code is available at:  
[https://github.com/automaticdavid/ansible-nso](https://github.com/automaticdavid/ansible-nso)

This can be tested using the Cisco labs:  
[https://devnetsandbox.cisco.com/RM/Topology](https://devnetsandbox.cisco.com/RM/Topology)


## 1- Sync NSO

![NSO]({static}/images/nso1.gif)

In this first part we start by executing the `check_sync.yml` playbook which verifies if the network devices are in sync with the NSO database. It's an example of using the `nso_action` Ansible module. We're using it to check the sync status of a given switch, then we check the status for all the devices. 

Then I run the `show_address.yml` playbook: it's an example of using the  `nso_query` module to run an XPATH query against NSO. I use it to display the names and IP address of my devices.

I then telnet into one switch (the whole cisco lab is telnet only) and make a manual confiugration change. When I run  `check_sync.yml` again, I see that the device is "out-of-sync". With Ansible it becomes easy to deal with such an exception: we could automatically open a ticket in an ITSM system, or trigger a remediation action. 

This remediation could be the next playbook: `sync_to.yml`, which executes a sync-to NSO action. This updates the switch configuration with what's defined in the NSO configuration, discarding the manual change. When I run `check_sync.yml` again, everything is in sync. 

## 2- Using NSO as a Source Of Truth 

![NSO]({static}/images/nso2.gif)

We start with a playbook that uses the `nso_verify` Ansible module to perform a backup of the NSO config. We get a local directory with YAML files describing the configuration of each device. Since it's YAML we could store this in git and start managing all our configs as code.  

To show how this would look like, I change the config of a device by modifying its YAML file. I then run the `apply_sot.yml` playbook. The first task reads the data from our local "source of truth" and applies it to the NSO config using the `nso_config` module. We then use the `nso_verify` module and it shows there is indeed a configuration violation between the NSO config and what is actually defined on the switch: 

![NSO]({static}/images/nso3.png)

The playbook follows with a task called "Apply SOT" that executes a sync-to, in order to push the NSO config to the device. We then run one last "Check sync" to confirm that everything is now in sync. 

It is also possible to verify a specific configuration element, as show in the `verify_config.yml` playbook. We just check a subset of the NSO config. It can be usefull, for example to check for something across many devices. 

We then run the `apply_sot.yml` playbook a second time: nothing happens and "changed" is at 0. That's because the playbook is indempotent and Ansible doesn't do anything when there's nothing to do. Combined with the use of `--check-mode` this can be used to check for the compliance of the NSO config. 

Finaly I telnet into the device to show that the change we made in the source of truth was indeed pushed to the physical device. 


## 3- Ansible Resource Modules & NSO

![NSO]({static}/images/nso4.gif)
   
Here we use one of the many Ansible network modules to change the config of an NXOS device directly, without using NSO. To use Ansible, I enabled ssh on one of the lab's equipement and created an Ansible inventory file. We then use the `nxos_vlan` module: 

```
- name: Direct device change
  nxos_vlans:
    config:
      - vlan_id: 501
        name: ansible
    state:  merged
  register: r_vlan
```

Using Ansible's native network modules to automate the network can be easier than through NSO. Here we see that the module shows the "before" and "after" states for the change. 

We then use `nso_verify` which shows a violations since NSO is now out of sync. Finally we run a sync_from to push the changed we made with Ansible into the NSO config. 


