# kvm-coreos
## CoreOS (Docker Container OS) using KVM on CentOS 7.x hypervisor

This is a quick guide to setup KVM hypervisor on CentOS 7

1. Disable selinux

`setenforce 1`

`sed -i --follow-symlinks 's/^SELINUX=.*/SELINUX=disabled/g' /etc/sysconfig/selinux`

2. Stop firewall-cmd

`systemctl disable firewalld`

`systemctl stop firewalld`

3. Now we'll bridge your primary network interface.  This bridge will also be used for our virtual machines.  In this example we'll create a bridge named "bridge0".  We'll then bridge interface "eth0" to "bridge0" followed by a server reboot.
4. Copy ifcfg-eth0 to ifcfg-bridge0

`cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-bridge0`

5. Edit /etc/sysconfig/network/network-scripts/ifcfg-bridge0. 
6. Change DEVICE=eth0 to DEVICE=bridge0
7. Change TYPE=Ethernet to TYPE="Bridge"

```
   DEVICE=bridge0
   ONBOOT=yes
   TYPE="Bridge"
   BOOTPROTO=none
   IPADDR=10.41.175.178
   NETMASK=255.255.255.0
   DNS1=4.2.2.2
   DNS2=8.8.8.8
```

8. Edit /etc/sysconfig/network-scripts/ifcfg-eth0 and bridge it to bridge0

```
   DEVICE=eth0
   ONBOOT=yes
   TYPE=Ethernet
   BOOTPROTO=none
   BRIDGE="bridge0"
   DNS1=4.2.2.2
   DNS2=8.8.8.8
```


