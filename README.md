dell-networking.dellos_network_validation
=========================

This roles is used to verify network validation. It validates network features of a wiring connection, BGP neighbors, the MTU between neighbors, and VLT pairing with Dell EMC SmartFabric OS10 switches.

- **Wiring validation** — based on the LLDP neighbor establishment; the intended neighbor input model is defined by the _group_var/all_ user which is compared with the actual LLDP neighbor; report is generated if there is any mismatch with the intended neighbors.

- **BGP validation** — based on the BGP neighbor state establishment; report is generated if the BGP neighbor state is not in an established state.

- **MTU validation** — based on the interface MTU; report is generated if there is an MTU mismatch between LLDP neighbors.

- **VLT validation** — based on the VLT information; report is generated if the backup VLT link is down or not present.

Installation
------------

    ansible-galaxy install Dell-Networking.dellos_network_validation

Role variables
--------------

- Variables and values are case-sensitive

**wiring_validation keys**

| Key        | Type                      | Description                                             | Support               |
|------------|---------------------------|---------------------------------------------------------|-----------------------|
| ``intended_neighbors`` | list  | Define topology details planned  | dellos10 |
| ``source_switch`` | string  | Define the source switch inventory name planned  | dellos10 |
| ``source_port`` | string  | Define the source port planned  | dellos10 |
| ``dest_switch`` | string  | Define the destination switch inventory name planned  | dellos10 |
| ``dest_port`` | string  | Define the destination port planned  | dellos10 |

**bgp_validation keys**

| Key        | Type                      | Description                                             | Support               |
|------------|---------------------------|---------------------------------------------------------|-----------------------|
| ``intended_bgp_neighbors`` | list  | Define topology details planned  | dellos10 |
| ``source_switch`` | string  | Define the source switch inventory name planned  | dellos10 |

**vlt_validation keys**

| Key        | Type                      | Description                                             | Support               |
|------------|---------------------------|---------------------------------------------------------|-----------------------|
| ``intended_vlt_pairs`` | list  | Define topology details planned  | dellos10 |
| ``primary`` | string  | Define the primary role of switch inventory name planned  | dellos10 |
| ``secondary`` | string  | Define the secondary role of switch inventory name planned  | dellos10 |

Connection variables
--------------------

Ansible Dell EMC roles require connection information to establish communication with the nodes in your inventory. This information can exist in the Ansible _group_vars_ or _host_vars_ directories or inventory, or in the playbook itself.

| Key         | Required | Choices    | Description                                         |
|-------------|----------|------------|-----------------------------------------------------|
| ``ansible_host`` | yes      |            | Specifies the hostname or address for connecting to the remote device over the specified transport |
| ``ansible_port`` | no       |            | Specifies the port used to build the connection to the remote device; if the value is unspecified, the ANSIBLE_REMOTE_PORT option is used; defaults to 22 |
| ``ansible_ssh_user`` | no       |            | Specifies the username that authenticates the CLI login for the connection to the remote device; if value is unspecified, the ANSIBLE_REMOTE_USER environment variable value is used  |
| ``ansible_ssh_pass`` | no       |            | Specifies the password that authenticates the connection to the remote device  |
| ``ansible_become`` | no       | yes, no\*   | Instructs the module to enter privileged mode on the remote device before sending any commands; if value is unspecified, the ANSIBLE_BECOME environment variable value is used, and the device attempts to execute all commands in non-privileged mode |
| ``ansible_become_method`` | no       | enable, sudo\*   | Instructs the module to allow the _become_ method to be specified for handling privilege escalation; if value is unspecified, the ANSIBLE_BECOME_METHOD environment variable value is used |
| ``ansible_become_pass`` | no       |            | Specifies the password to use, if required, to enter privileged mode on the remote device; if `ansible_become` is set to no, this key is not applicable |
| ``ansible_network_os`` | yes      | dellos10, null\*  | Value is used to load the correct terminal and cliconf plugins to communicate with the remote device |

> **NOTE**: Asterisk (*) denotes the default value if none is specified.

Dependencies
------------

The _xmltodict_ library should be installed to convert show command output in dictionary format from XML. To install the package, use `pip install xmltodict`.

The `dellos_fabric_summary` role must be included to query system network summary information.

Example playbook
----------------

This example uses the `dell-networking.dellos_network_validation` role to verify network validations. It creates a *hosts* file with the switch details and corresponding variables.


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
        
    cli:
     host: "{{ ansible_host }}"
     username: "{{ dellos10_cli_user | default('admin') }}"
     password: "{{ dellos10_cli_pass | default('admin') }}"

**Sample ``group_var/all``**
----------------------------
**Sample input for wiring validation**

 
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

**Sample input for bgp validation**

      intended_bgp_neighbors:
        - source_switch: site1-spine1
          neighbor_ip: ["10.11.0.1","10.9.0.1","10.9.0.3","10.9.0.5","1.1.1.1"]
        - source_switch: site1-spine2
          neighbor_ip: ["10.11.0.0","10.9.0.9","10.9.0.11","10.9.0.15"]

**Sample input for vlt validation**

      intended_vlt_pairs:
        - primary: site1-spine1
          secondary: site2-spine2
        - primary: site2-spine1
          secondary: site2-spine2


**Simple playbook to setup network validation**
-----------------------------------------------

**Sample playbook of ``validation.yaml`` to run complete validation**

	---
        - name: setup network validation
	  hosts: LeafAndSpineSwitch
	  gather_facts: no
	  connection: network_cli
	  roles:		
             - ansible-role-dellos-network-validation

**Sample playbook to run wiring validation**

        ---
        - name: setup wiring validation
          hosts: LeafAndSpineSwitch
          gather_facts: False
          connection: network_cli
          tasks:
            - import_role:
                name: ansible-role-dellos-network-validation
                tasks_from: wiring_validation.yaml

**Sample playbook to run BGP validation**

        ---
        - name: setup bgp validation
          hosts: LeafAndSpineSwitch
          gather_facts: False
          connection: network_cli
          tasks:
            - import_role:
                name: ansible-role-dellos-network-validation
                tasks_from: bgp_validation.yaml

**Sample playbook to run VLT validation**

        ---
        - name: setup vlt validation
          hosts: LeafAndSpineSwitch
          gather_facts: False
          connection: network_cli
          tasks:
            - import_role:
                name: ansible-role-dellos-network-validation
                tasks_from: vlt_validation.yaml

**Sample playbook to run MTU validation**

        ---
        - name: setup mtu validation
          hosts: LeafAndSpineSwitch
          gather_facts: False
          connection: network_cli
          tasks:
            - import_role:
                name: ansible-role-dellos-network-validation
                tasks_from: mtu_validation.yaml


**Run**

Execute the playbook and examine the results.

``ansible-playbook -i inventory.yaml validation.yaml``

**Sample output**
-----------------
**sample output of wiring validation**

    "results": [
        {
            "dest_port": "ethernet1/1/1",
            "dest_switch": "site2-spine2",
            "error_type": "link-missing",
            "reason": "link is not found for source switch: site2-spine1,port: ethernet1/1/1",
            "source_port": "ethernet1/1/1",
            "source_switch": "site2-spine1"
        },
        {
            "dest_port": "ethernet1/1/2",
            "dest_switch": "site2-spine1",
            "error_type": "link-mismatch",
            "reason": "Destination switch is not an expected value, expected switch: site2-spine1,port: ethernet1/1/2; actual switch: site1-spine2(svc-tag:J33FXC2, node_mac:e4:f0:04:9b:eb:dc), port: ethernet1/1/1",
            "source_port": "ethernet1/1/1",
            "source_switch": "site1-spine1"
        }
    ]

**sample output of BGP validation**

    "results": [
         {
             "bgp_neighbor": "10.9.0.1",
             "bgp_state": "idle",
             "error_type": "remote_port_down",
             "possible_reason": "remote port site2-spine1 ethernet1/1/2 is down",
             "source_switch": "site1-spine1"
         },
         {
             "bgp_neighbor": "-",
             "bgp_state": "idle",
             "error_type": "not_an_intended_neighbor",
             "possible_reason": "neighbor 10.9.0.7 is not an intended, please add this neighbor in the intended_bgp_neighbors",
             "source_switch": "site1-spine1"
         },
         {
             "bgp_neighbor": "1.1.1.1",
             "error_type": "config_missing",
             "possible_reason": "neighbor config missing",
             "source_switch": "site1-spine1"
         },
         {
             "bgp_neighbor": "10.9.0.9",
             "bgp_state": "idle",
             "error_type": "remote_port_down",
             "possible_reason": "remote port site2-spine1 ethernet1/1/3 is down",
             "source_switch": "site1-spine2"
         }
      ]

**sample output of VLT validation**

    "results": [
        {
            "error_type": "secondary_mismatch",
            "intended_primary": "site1-spine1",
            "intended_secondary": "site2-spine2",
            "possible_reason": "config mismatch as site2-spine2 is expected, but the actual secondary is site1-spine2 ",
            "secondary": "site1-spine2"
        },
        {
            "error_type": "peer_missing",
            "intended_primary": "site2-spine1",
            "intended_secondary": "site2-spine2",
            "possible_reason": "peer info is not configured or peer interface is down"
        }
    ]

**sample output of MTU validation**

    "msg": {
        "results": "There is no MTU mistmatch between neighbors"
    }


(c) 2019 Dell Inc. or its subsidiaries. All Rights Reserved.

