Title: Using conditional import of external vars with Tower's Smart Inventories
Category: Tower
Date: 2019-10-20


The problem we are trying to solve is that smart inventories in Tower do not include groups, and therefore we can't use group_vars defined at the inventories level. Since host_vars are imported we will use a host_var to trigger import of the wanted vars.

## 1- VM Properties

We're going to use the "Custom Attributes" feature of vSphere to inherit a host_vars. Each VM has a custom attribute `dcl-custom-attr` but with a diferent value for each of our test VMs:

![VM Custom prop]({attach}/images/custom-prop.png)

![VM Custom prop]({attach}/images/custom-prop-2.png)

## 2- Dynamic Inventory & Smart Inventory in Tower

We're using the VMware dynamic inventory with source variables: 

![VM Custom prop]({attach}/images/inv.png)

We want to test importing vars for hosts sourced from a smart inventory so we define a smart inventory:

![VM Custom prop]({attach}/images/smart.png)

![VM Custom prop]({attach}/images/smart-hosts.png)

The VMs in the smart inventory show the custom attributes as host_vars:

![VM Custom prop]({attach}/images/vars.png)

## 3- Conditional include of vars

Now we can use the following playbook:

	---
	- name: Conditional vars inclusion based on host_vars 
	  hosts: all
	  gather_facts: no
	  
	  tasks:
	  	        
	    - name: Extract Custom Prop Value
	      set_fact:
	        vm_props: "{{ value | map(attribute='value')| list }}" 
	        
	    - name: Include for first group
	      include_vars:
	        file: first.yml
	      when: '"dcl-1" in vm_props'
	      
	    - name: Include for second group
	      include_vars:
	        file: second.yml
	      when: '"dcl-2" in vm_props'
	
	    - name: Display imported value
	      debug: 
	        msg: "{{ my_imported_value }}"
	        
Depending on the value of the host_var we include a file or another, that defines the variable `my_imported_value` but with different values. <BR>
See: https://github.com/automaticdavid/set_vars
	
We define the associated Tower template:

![VM Custom prop]({attach}/images/template.png)

Execution yields the inclusion of the var with the correct value for each VM: 

![VM Custom prop]({attach}/images/result.png)

We can use the same mechanism to include different vars for each VM.
 
This could be used in a role that would get included at the begining of each playbook, or with a Tower Workflow. 




