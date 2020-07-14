Title: Infra As Code with Ansible for VMware vSphere
Category: Tower
Date: 2020-07-14
Lang: en
Slug: iac-vmware

This is a short write up about a demo I did for a Red Hat France Webinar.
You can find the full replay (in french) at:   
[Automatiser à l’échelle de l’entreprise et gérer l'infrastructure comme du code avec Red Hat Ansible Automation Platform](https://www.redhat.com/fr/events/webinar/automate-enterprise-infrastructure)

The code used is at:   
[github](https://github.com/automaticdavid/demo_vmware/tree/webinar_072020)

The inventory I'm using is at:  
[github](https://github.com/automaticdavid/vmware_services/tree/master/paris)

## 1 Setting up the Inventories in Tower

We're using a Tower inventory with 2 sources: one is the project that points to the git inventory "vmware service definition", the other is a vCenter based dynamic inventory:

![Inventory]({static}/images/webinar2_inventory.png)

Tower ships with a default vCenter inventory, but for this demo I'm using the latest upstream version. To do this I copied the upstream collection into my own repo at: [https://github.com/automaticdavid/vmware_inventory.git](https://github.com/automaticdavid/vmware_inventory.git).  
Note that I respected the collection directory path: `collections/ansible_collections/community/vmware/`  
  
I also added an `ansible.cfg` file that enables the vmware community inventory plugin and a configuration file for the plugin: `conf.vmware.yml` (this file has to end in .vmware.yml). I'm using this to force the dynamic inventory to report hostnames as the name of the VM, instead of the default which is a concatenation of the VM name and the VM UUID. 

To use vCenter tags, we need pyvmomi (obviously) and also the python VMware Automation SDK, as documented in: (https://docs.ansible.com/ansible/latest/plugins/inventory/vmware_vm_inventory.html).   
To run this cleanly, I created a dedicated venv as explained in: (https://docs.ansible.com/ansible-tower/latest/html/upgrade-migration-guide/virtualenv.html)

Inside Tower I also defined a custom credential to use with this inventory: 

![Credential]({static}/images/webinar2_cred.png)

This provides the environment variables needed to connect to the vCenter. Depending on the version of the collection you're using, you may also use a Tower vCenter credential for this. 
The inventory source definition in Tower is a standard "sourced from a project" definition, with an added variable to enable using the custom plugin: 

![Credential]({static}/images/webinar2_inventory2.png)

This also allows to specify that we're using our custom venv.

## 2 Provisioning a Service

We start with an empty vCenter and a service definition in git. The definition takes place in services.ini where we add the service `app1` to the inventory:

```
[services:children]
app1

[app1]
host1
```

We also create in the `group_vars` directory a file called `app1.yml` that defines the `app1` service:

```
service_config:
  vm: 
    project: terminator
    role: webserver
    service: app1
    count: 1
    size: small
  app: 
    name: terminator_v1
    quote: give me your boots, your gun, and your motorcycle
    version: 1 
  global: 
    context: movie_machine
```

I like how this provides a very clean and readable definition for a service. Although this is a simple example it could very well be extended to manage more complex use cases. 

We then run a Tower Workflow that does 4 things: 

- Refresh the project based inventory source to read the new service definitions from git
- Provision and tag the VMs needed for the new services
- Refresh the vCenter dynamic inventory once the VM are provisioned, getting the tags as Tower inventory groups
- Run a playbook that install the application layer needed on each VM of the services, based on the VM's groups.

Here's a video of what it looks like (better in full screen): 

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/V0hhakK6Uwc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

On my lab setup, the workflow takes less than 4mn to complete.
    

## 3 Adding and Modifying a Service

The playbook we call in the Job Template is indempotent: when I add a whole new service, running the same workflow just adds what's needed without doing anything to our already provisioned service. 

In the video below, I add a new service `app2` but also modify the `version` value of the `app1` service. In the Tower log we can see that Ansible does only what's needed and leaves what is already configured untouched. 

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/XfyPkvEOqSE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    
We can also use git to track changes to the services definitions: 

![Git diff]({static}/images/webinar2_git.png)
    

## 4 Force delete of a VM

What's cool with Ansible is that the IaC behaviour is based on indempotency of playbooks and definition of state in a source of truth (the inventory in git), as opposed to other tools that need their own statefull reference. This means that I can delete a VM in vCenter and when the Job Template is executed again, it just corrects what's missing (in this lab the IP of the recreated VM changes, but in real life we'd use an IPAM). We are not locked to only one control plane.   

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/K0xLGhgTuhA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

We can also run the Job Template in `check_mode` and do compliance checking. 


## What's next

There's a number of other things we could do from here:  

- Enable [webhooks](https://docs.ansible.com/ansible-tower/latest/html/userguide/webhooks.html)
- Enable different environements to test and promote our infrastructure and services definitions
- Add support for changing the VM's hardware definition of a provisioned service
- Use pull requests to manage the services definitions 
- Add other roles for services, for instance eap 







 
  
  