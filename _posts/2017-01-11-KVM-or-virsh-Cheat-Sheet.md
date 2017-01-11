---
title:  "KVM or Virsh Cheat Sheet"
header:
  image: /assets/images/zmjpgbyybuyvrb5cfb5a.jpg
date:  2017-01-11
tags:
  - Semarang
  - food
  - Culinary
description: ''
categories:
  - Others
---

Just my note of Virsh Command

## About Instance / Virtual Machine

Command | About 
------- | ----- 
```virsh list``` | List running Virtual Machine | 
```virsh list --all``` | List all Virtual Machines | 
```virsh start <instance>``` | Start Virtual Machine |
```virsh shutdown <instance>``` | Shutdown Virtual Machine |
```virsh destroy <instance>``` | Destroy Virtual Machine |
```virsh suspend <instance>``` | Suspend Virtual Machine |
```virsh resume <instance>``` | Resume Virtual Machine |
```virsh console <instance>``` | Access Virtual Machine Console <br />to exit use this command : <br /> ```Cntrl-]``` 
```virsh autostart <instance>``` | Virtual Machine starts at boot |
```virsh autostart --disable <instance>``` | Disable Virtual Machine starts at boot |
```virsh edit <instance>``` | Virtual Machine Configuration | 
```virsh save <instance> <filename>``` | Save Virtual Machine to file | 
```virsh restore $FILENAME``` | Load Virtual machine from file | 
```virt-clone \``` <br /> ```--original [VM to Clone] \``` <br /> ```--auto-clone \``` <br /> ```--name [new name]``` | Simple VM Cloning | 
```virsh net-list``` | List Running Network Configs | 
```virsh net-list --all``` | List All Network Configs <br /> You can find network configs stored in ```/home/<user's>/network-configs/```| 
```virsh net-list [network name here]``` | Edit Network Config | 
```virsh net-create --file [full file path here]``` | Create Temporary Network Config | 
```virsh net-define --file [full file path here]``` | Create Permanent Network Config | 
```virsh net-start [network identifier]``` | Start Network Config | 
```net-autostart --network [network identifier]``` | Enable Network Autostart | 
```net-autostart --network [network identifier] --disable``` | Disable Network Autostart | 
```virsh snapshot-create <instance>``` | Create Snapshot | 
```virsh snapshot-create-as <instance> [name]``` | Create Snapshot With Name | 
```virsh snapshot-create-as [<instance> [name] [description]``` | Create Snapshot With Name and Description | 
```virsh snapshot-list <instance>``` | List Snapshots <br /> Snapshot-list defaults to being in alphabetical rather than chronological order. If you want to find out what your latest snapshots are, you may wish to add the optional ```--tree``` or  ```--leaves``` parameters.  | 
```virsh snapshot-revert <instance> [Snapshot Name]``` | Restore Snapshot | 
```virsh snapshot-delete <instance> [Snapshot Name]``` | Delete Snapshot | 
```virsh blockresize <instance> --path vda --size 100G``` | Online resizing instance | 
```virsh dominfo``` | show domain information |	
```virsh vcpuinfo``` | show domain vcpu information | 
```virsh nodeinfo``` | show node information | 
```virsh quit``` | Leave CLI | 


## Network Example

## Example Bridge Network Config File
```python
<network>
  <name>examplebridge</name>
  <forward mode='route'/>
  <bridge name='kvmbr0' stp='on' delay='0'/>
  <ip address='192.168.1.1' netmask='255.255.255.0' />
</network>
```

## Example Manual Network Config With Bridge

This example of network config ubuntu users in ```/etc/network/interfaces``` :

```python
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto p17p1
iface p17p1 inet manual

auto kvmbr0
iface kvmbr0 inet static
    address 192.168.1.19
    netmask 255.255.255.0
    network 192.168.1.0
    broadcast 192.168.1.255
    gateway 192.168.1.254
    bridge_ports p17p1
    bridge_stp off
    bridge_fd 0
    bridge_maxwait 0
```

## Configure VM To Use Manual Bridge

Edit the instance
```python
virsh edit [guest identifier]
```

Find the following section
```python
 <interface type='network'>
      <mac address='52:54:00:4d:3a:bd'/>
      <source network=''/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </interface>
```
Change into 
```python
<interface type='bridge'>
        <mac address='52:54:00:4d:3a:bd'/>
        <source bridge='[bridge name here]'/>
        <model type='virtio'/>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </interface>
```

Power cycle the instance using **shutdown** and **start** command 

```python
virsh shutdown [guest identifier]
virsh start [guest identifier]
```

## Resize Memory

Enter into instance configuration
```python
virsh edit $VM_ID 
```

Change the **memory** and **currentMemory** fields to be the size you want in KiB.

```python
<domain type='kvm'>
...
  <memory unit='KiB'>52488</memory>
  <currentMemory unit='KiB'>52488</currentMemory>
...
```
[Don't Use "virsh memtune" Click here for reason](http://libvirt.org/formatdomain.html#elementsMemoryTuning){: .btn .btn--danger}



## CPU Management

**Discover CPU Scheduling Parameters**
```python
virsh schedinfo [guest ID or name]
```

**Permanently Set CPU Shares For Live Running Instance**
```python
sudo virsh schedinfo [guest ID or name] \
--set cpu_shares=[0-262144] \
--live \
--current \
--config
```




