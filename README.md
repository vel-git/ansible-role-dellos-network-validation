======================================
ansible-role-dellos-network-validation 
======================================

This roles is used to verify Network Validation. It validates networking features of wiring connection,bgp neighbors,MTU between neighbors and VLT pair with Dell EMC Networking OS10 switches. 

- **Wiring Validation** is performed based on LLDP neighbor establishment. The planned neighbor input model defined by the user in ``group_var/all`` which is compared with actual lldp neighbor and notification is shown if there is an mismatch with planned input. 

- **BGP Validation** is performed based on bgp neighbor state establishment. Notification is shown if the bgp neighbor state is not in established state.

- **MTU Validation** is performed based on interface MTU, Notification is shown if there is an MTU mismatch between lldp neighbors.

- **VLT Validation** is performed based on VLT info, Notification is shown if the backup VLT link is down or not present.

Installation
------------

    ansible-galaxy install Dell-Networking.dellos-network-validation

Role variables
--------------

- Variables and values are case-sensitive

**wiring_validation**

| Key        | Type                      | Description                                             | Support               |
|------------|---------------------------|---------------------------------------------------------|-----------------------|
| ``intended_neighbors`` | list  | define topology details which is planned  | dellos10 |
| ``intended_bgp_neighbors`` | list  | define topology details which is planned  | dellos10 |
| ``intended_vlt_pairs`` | list  | define topology details which is planned  | dellos10 |

Connection variables
--------------------

Ansible Dell EMC Networking roles require connection information to establish communication with the nodes in your inventory. This information can exist in the Ansible *group_vars* or *host_vars* directories or inventory, or in the playbook itself.

| Key         | Required | Choices    | Description                                         |
|-------------|----------|------------|-----------------------------------------------------|
| ``ansible_host`` | yes      |            | Specifies the hostname or address for connecting to the remote device over the specified transport |
| ``ansible_port`` | no       |            | Specifies the port used to build the connection to the remote device; if value is unspecified, the ANSIBLE_REMOTE_PORT option is used; it defaults to 22 |
| ``ansible_ssh_user`` | no       |            | Specifies the username that authenticates the CLI login for the connection to the remote device; if value is unspecified, the ANSIBLE_REMOTE_USER environment variable value is used  |
| ``ansible_ssh_pass`` | no       |            | Specifies the password that authenticates the connection to the remote device.  |
| ``ansible_become`` | no       | yes, no\*   | Instructs the module to enter privileged mode on the remote device before sending any commands; if value is unspecified, the ANSIBLE_BECOME environment variable value is used, and the device attempts to execute all commands in non-privileged mode |
| ``ansible_become_method`` | no       | enable, sudo\*   | Instructs the module to allow the become method to be specified for handling privilege escalation; if value is unspecified, the ANSIBLE_BECOME_METHOD environment variable value is used. |
| ``ansible_become_pass`` | no       |            | Specifies the password to use if required to enter privileged mode on the remote device; if ``ansible_become`` is set to no this key is not applicable. |
| ``ansible_network_os`` | yes      | dellos10, null\*  | This value is used to load the correct terminal and cliconf plugins to communicate with the remote device. |

> **NOTE**: Asterisk (*) denotes the default value if none is specified.

Dependencies
------------

*xmltodict*  library should be installed to convert show command output in dict format from xml.
*ansible-role-dellos-fabric-summary* role should be included to query system network summary information. 

Example playbook
----------------

This example uses the *ansible-role-dellos-network-validation* role to verify the network validations. It creates a *hosts* file with the switch details and corresponding variables.


**Sample hosts file**

        site1-spine1 ansible_host=10.11.180.21 ansible_ssh_user=admin ansible_ssh_pass=admin ansible_network_os=dellos10
        site1-spine2 ansible_host=10.11.180.22 ansible_ssh_user=admin ansible_ssh_pass=admin ansible_network_os=dellos10
        site2-spine1 ansible_host=10.11.180.23 ansible_ssh_user=admin ansible_ssh_pass=admin ansible_network_os=dellos10
        site2-spine2 ansible_host=10.11.180.24 ansible_ssh_user=admin ansible_ssh_pass=admin ansible_network_os=dellos10
        [spine]
        site1-spine1
        site1-spine2
        site2-spine1
        site2-spine2
        [LeafAndSpineSwitch:children]
        spine


**Sample host_vars/site1-spine1**
        
:: 
    
    cli:
     host: "{{ ansible_host }}"
     username: "{{ dellos10_cli_user | default('admin') }}"
     password: "{{ dellos10_cli_pass | default('admin') }}"

**Sample ``group_var/all``**

Sample input for wiring validation:
 
::
 
    intended_neighbors:
      - source_switch: site1-spine2
        source_port: ethernet1/1/5
        dest_port: ethernet1/1/29
        dest_switch: site1-spine1
      - source_switch: site1-spine2
        source_port: ethernet1/1/6
        dest_port: ethernet1/1/30
        dest_switch: site1-spine1
      - source_switch: site1-spine2
        source_port: ethernet1/1/7
        dest_port: ethernet1/1/31
        dest_switch: site1-spine1
      - source_switch: site1-spine2
        source_port: ethernet1/1/8
        dest_port: ethernet1/1/32
        dest_switch: site1-spine1
      - source_switch: site1-spine2
        source_port: ethernet1/1/9
        dest_port: ethernet1/1/21
        dest_switch: site1-spine1
      - source_switch: site1-spine2
        source_port: ethernet1/1/7
        dest_port: ethernet1/1/29
        dest_switch: site1-spine3

Sample input for bgp validation:
 
::

    intended_bgp_neighbors:
      - source_switch: site1-spine1
        neighbor_ip: ["10.11.0.1","10.9.0.1","10.9.0.3","10.9.0.5","1.1.1.1"]
      - source_switch: site1-spine2
        neighbor_ip: ["10.11.0.0","10.9.0.9","10.9.0.11","10.9.0.15"]

Sample input for vlt validation:
 
::

    intended_vlt_pairs:
      - primary: site1-spine1
        secondary: site2-spine2
      - primary: site2-spine1
        secondary: site2-spine2


**Simple playbook to setup Network Validation**
Sample playbook of ``validation.yaml`` to run complete validation:

:: 

	---
        - name: setup network validation
	  hosts: localhost
	  gather_facts: no
	  connection: local
	  roles:		
                - ansible-role-dellos-network-validation

Sample playbook of ``validation.yaml`` to run a feature sepcific validation:

:: 

        ---
        - name: setup network validation
          hosts: localhost
          gather_facts: False
          connection: local
          tasks:
            - import_role:
                name: ansible-role-dellos-network-validation
                tasks_from: wiring_validation.yaml
            - import_role:
                name: ansible-role-dellos-network-validation
                tasks_from: bgp_validation.yaml
            - import_role:
                name: ansible-role-dellos-network-validation
                tasks_from: vlt_validation.yaml
            - import_role:
                name: ansible-role-dellos-network-validation
                tasks_from: mtu_validation.yaml


**Run**

Execute the playbook and Examine the results:

``ansible-playbook -i inventory.yaml validation.yaml``

(c) Copyright 2019 Dell Inc. or its subsidiaries. All Rights Reserved.
