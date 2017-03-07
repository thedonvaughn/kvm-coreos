# kvm-coreos
## CoreOS (Docker Container OS) using KVM on CentOS 7.x hypervisor

### This is a quick guide to setup KVM hypervisor on CentOS 7

### Network setup

* Disable selinux

`setenforce 1`

`sed -i --follow-symlinks 's/^SELINUX=.*/SELINUX=disabled/g' /etc/sysconfig/selinux`

* Stop firewall-cmd

`systemctl disable firewalld`

`systemctl stop firewalld`

* Now we'll bridge your primary network interface.  This bridge will also be used for our virtual machines.  In this example we'll create a bridge named "bridge0".  We'll then bridge interface "eth0" to "bridge0" followed by a server reboot.
* Copy ifcfg-eth0 to ifcfg-bridge0

`cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-bridge0`

* Edit /etc/sysconfig/network/network-scripts/ifcfg-bridge0. 
  * Change DEVICE=eth0 to DEVICE=bridge0
  * Change TYPE=Ethernet to TYPE="Bridge"

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

* Edit /etc/sysconfig/network-scripts/ifcfg-eth0 and bridge it to bridge0

```
   DEVICE=eth0
   ONBOOT=yes
   TYPE=Ethernet
   BOOTPROTO=none
   BRIDGE="bridge0"
   DNS1=4.2.2.2
   DNS2=8.8.8.8
```

* Reboot the server

`reboot`

### Install KVM packages

* Once the server comes up, install packages needed for kvm/libvirt/qemu:

```
  yum -y install qemu-kvm libvirt virt-install bridge-utils bind-utils \
  virt-manager wget net-tools virt-viewer genisoimage epel-release
```

* Enabled and start libvirtd

`systemctl enable libvirtd`

`systemctl start libvirtd`

* Verify 

```
[root@kvm-host ~]# virsh nodeinfo
CPU model:           x86_64
CPU(s):              8
CPU frequency:       1995 MHz
CPU socket(s):       2
Core(s) per socket:  4
Thread(s) per core:  1
NUMA cell(s):        1
Memory size:         16772028 KiB
```

### Install Kimchi
 
```
cd /tmp
wget http://kimchi-project.github.io/wok/downloads/latest/wok.el7.centos.noarch.rpm
wget http://kimchi-project.github.io/gingerbase/downloads/latest/ginger-base.el7.centos.noarch.rpm
wget http://kimchi-project.github.io/kimchi/downloads/latest/kimchi.el7.centos.noarch.rpm
yum -y install wok.el7.centos.noarch.rpm
yum -y install ginger-base.el7.centos.noarch.rpm
yum -y install kimchi.el7.centos.noarch.rpm
sed -i  's/^#session_timeout = .*/session_timeout = 1440/g' /etc/wok/wok.conf
systemctl enable wokd
systemctl start wokd
```

* Go to https://YOUR_KVM_SERVER:8001 in your web browswer and login using root creds

### Download CoreOS QEMU Image

* Make dir to hold coreos image and guest images

`mkdir -p /vmstorage/coreos`

* Download CoreOS QEMU Image

`wget https://stable.release.core-os.net/amd64-usr/current/coreos_production_qemu_image.img.bz2 -O - | bzcat > /vmstorage/coreos/os.img`

* Create cloud-config template.  Can download mine from here.

`wget https://gist.githubusercontent.com/dutchiechris/e5689a526d4651f6dec4880230fe3c89/raw/4576b956ab00430e644917f17080f4ccfd408615/user_data`
