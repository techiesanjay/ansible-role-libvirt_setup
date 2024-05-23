# Ansible role `libvirt_setup`

Ansible role for setting up **libvirt** related packages to work effectively with kvm based virtualization on Linux. This role can be used to **add**, **modify** or **delete** additional libvirt storage pools and libvirt networks if that is required for specific applications that you are deploying. This role is also available as part of the **techiesanjay.libvirt_infra** content collection on Ansible Galaxy.

## Requirements

No specific requirements. The pre-flight validation within the role will ensure that the host on which this role is executed has support for virtualization.

## Role Variables
This role by default will initialize the libvirt default storage pool and libvirt default storage network. However, If additional libvirt **storage pools** and libvirt **networks** are required then the dictionary **libvirt:** will be required by the role. This is explained as follows:
```Yaml
libvirt:
  storage:
    # Storage pool specific variables go here
  network:
    # Network specific variables go here
```
> **NOTE**: If the above list of dictionaries are not specified then the role will just install the necessary libvirt packages for managing virtualization and start the default storage pool and default network. You should be able to provision VMs after the default pool and network are started.

### libvirt.storage dictionary variable
This dictionary is used when a new storage pool is required to hold VMs for a specific deployment.

| Variable | Comments |
|--|--|
| pool_name    | Name of the storage pool to create |
| pool_path    | Points to a folder inside the default storage path i.e. /var/lib/libvirt/images/ocp. The default libvirt storage pool path is /var/lib/libvirt/images |

### libvirt.network dictionary variable
This dictionary is used when a new network is required for VMs for a specific deployment that need to use a specific IP address range that does not interfere with existing applications.

| Variable | Comments|
| -- | -- |
| network_name | Name of the libvirt network to create |
| network_bridge | Name of the network bridge to be created |
| network_gateway | An ipv4 address to be used as a gateway to access external networks. Example **10.0.0.1** |
| network_mask | The subnet mask to be used for the network |
| network_dhcp | Should the network assign a DHCP lease to VMs. Possible values **True** or **False**. If this is set to False then ***network_dhcp_start*** and ***network_dhcp_end*** are not required and will be ignored if set. |
| network_dhcp_start    | An ipv4 address that specifies start range for a DHCP lease for a VM. Example **10.0.0.50**. This specific example case also assumes that you may want to use the range **10.0.0.2** to **10.0.0.49** for setting static IPs for your VMs since these are excluded from the DHCP range. |
| network_dhcp_end | An ipv4 address that specifies end range for a DHCP lease for a VM. Example **10.0.0.254** |

#### Example. 
 Creates a storage pool and a network that does not use DHCP. All VMs in this network will use  static IP addresses.
```Yaml
libvirt:
  storage:
    pool_name: ocp-store
    pool_path: /var/lib/libvirt/images/ocp
  network:
    network_name: ocp-net
    network_bridge: br0-ocp
    network_gateway: 192.168.100.1
    network_mask: 255.255.255.0
    network_dhcp: false
```

#### Example. 
 Creates a storage pool and a network that uses DHCP for VMs. Some IP addresses not part of the lease can be used for static IP assignment for VMs.
```Yaml
libvirt:
  storage:
    pool_name: ocp-store
    pool_path: /var/lib/libvirt/images/ocp
  network:
    network_name: ocp-net
    network_bridge: br0-ocp
    network_gateway: 192.168.100.1
    network_dhcp: true
    network_mask: 255.255.255.0
    network_dhcp_start: 192.168.100.50
    network_dhcp_end: 192.168.100.254
```


## How to use this role?

#### Playbook to create additional libvirt storage and network
```Yaml
- name: Starting with libvirt configuration
  hosts: localhost
  become: yes
  vars_files:
    - 'configs/password.yml'
    - 'configs/libvirt-config.yml'
    
  vars:
    ansible_become_password: "{{ sudo_password }}"

  roles:
    - libvirt_setup
```
For additional examples lookup the **tests** directory within the role.

#### configs/libvirt-config.yml
```Yaml
# libvirt storage pool and networking configuration and settings
libvirt:
  storage:
    pool_name: ocp-store
    pool_path: /var/lib/libvirt/images/ocp
  network:
    network_name: ocp-net
    network_bridge: br0-ocp
    network_gateway: 192.168.100.1
    network_dhcp: true
    network_mask: 255.255.255.0
    network_dhcp_start: 192.168.100.50
    network_dhcp_end: 192.168.100.254
```

#### configs/password.yml
```Yaml
# CHANGE this to match the host on which this role will be executed
sudo_password: GobbleDeGook
```

## Dependencies
No dependencies.

## License
BSD

## Author
**Sanjay Singh**
sanjay.kr.singh@gmail.com

> Written with [StackEdit](https://stackedit.io/).


