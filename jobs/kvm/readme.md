# Building a kvm army

This example parallels the [docker example](https://github.com/jfelten/knowhow_example_repo/tree/master/jobs/docker), but uses kvm and libvirtd instead of docker for the virtualization.  There are many cases where a docker container is not enough, and a full blown host or different OS is needed.  Fortunately automating kvm guest creatation is just as easy.

In this example we use centos7, but it can be easily modified for any other OS.

## Install KVM on centos 7

First install the EPEL yum repository on your host system, and then use yum to install all the libvirtd packages.  I found it useful to have the kvm X GUI applications, since the CentOS installer uses a GUI for it's installation.  It makes troubleshooting much easier and they are included in this job.  This job requires root access so run as a user with the appropriate sudo rights.

### A note about networking

As with other virtualization platforms networking can be a pain with kvm.  Out of the box kvm installs its own virtual bridge that uses NAT(Netowrk Address Translation) and uses this to assign ips to guests.  Outbound traffic from guests works as part of the install, but inbound network traffic to each guest is blocked.  KVM deals with this by adding a hooks directory in /etc/libvirt/hooks (on centos) where custom ip chain rules can be used to route traffic to your guests.  This gets ugly fast as more guests and ports are added, and doesn't play nice with firewalld on RHEL7 variants.  The solution I used was to set up my own network bridge with its own DHCP server and dynamic dns, so that netowrking works automagically when new guests are installed.  Installing a netowrk bridge, dchpd, and dynamic-dns is major write up unto itself and I won't get into that here.  The last 4 commands commented out for an example on how to use /etc/libvirt/hooks.

      {
      "id": "install libvirtd and kvm pages on centos 7",
      "working_dir": "./",
      "options": {
        "timeoutms": 360000
      },
      "shell": {
        "command": "sudo su -",
        "onConnect": {
          "responses": {
            "[Pp]assword": "${PASSWORD}"
          },
          "errorConditions": [
            "Sorry",
            "[Dd]enied"
          ],
          "waitForPrompt": "[#]"
        },
        "onExit": {
          "command": "exit"
        }
      },
      "files": [],
      "script": {
        "env": {
          "PASSWORD": "my_password",
          "EPEL_RPM": "epel-release-7-5.noarch.rpm",
          "EPEL_RPM_LOC": "http://dl.fedoraproject.org/pub/epel/7/x86_64/e/${EPEL_RPM}",
          "VIR_BR_NETWORK": "10.10.1"
        },
        "commands": [
          {
            "command": "curl -O ${EPEL_RPM_LOC}"
          },
          {
            "command": "rpm -ivh --replacepkgs ${EPEL_RPM}"
          },
          {
            "command": "yum -y install qemu-kvm libvirt virt-install bridge-utils libguestfs-tools"
          },
          {
            "command": "yum -y install xorg-x11-xauth xorg-x11-fonts-* xorg-x11-utils"
          },
          {
            "command": "yum -y install virt-manager virt-manager"
          },
          {
            "command": "sed -i 's/^\\(net.ipv4.ip_forward =\\).*/\\1 1/' /etc/sysctl.conf; sysctl -p"
          },
          {
            "command": "systemctl start libvirtd"
          },
          {
            "command": "systemctl enable libvirtd"
          },
          {
            "description": "Only do this if you are using the KVM virtual NAT bridge and if you want to allow ssh and know how agents into your guests.  Remove the echo command to execute:",
            "command": "echo 'mkdir -p  /etc/libvirt/hooks'"
          },
          {
            "description": "Only do this if you are using the KVM virtual NAT bridge and if you want to allow ssh and know how agents into your guests.  Remove the echo command to execute:",
            "command": "echo 'echo \"!#/bin/bash\" > /etc/libvirt/hooks/qemu'"
          },
          {
            "description": "Only do this if you are using the KVM virtual NAT bridge and if you want to allow ssh and know how agents into your guests.  Remove the echo command to execute, replace with the firewall-cmd if you are using firewalld",
            "command": "echo 'echo \"iptables -I FORWARD -d ${VIR_BR_NETWORK}.0/24 -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT\" >> /etc/libvirt/hooks/qemu'"
          },
          {
            "description": "Only do this if you are using the KVM virtual NAT bridge and if you want to allow ssh and know how agents into your guests.  Remove the echo command to execute, replace with the firewall-cmd if you are using firewalld",
            "command": "echo 'echo \"iptables -I FORWARD -d ${VIR_BR_NETWORK}.0/24 -p tcp -m state --state NEW -m tcp --dport 3000 -j ACCEPT\" >> /etc/libvirt/hooks/qemu'"
          }
        ]
      }
    }


## Create a guest template

For my purposes I created a centos 7 instance manually and then wrote a job to install standard software that I need on it.  Whenever I need a new guest I just clone this instance.  It would be possible to automate the template install using a centos kickstart file, but I didn't have the need to do so.

## Job to create a new guest by cloning the template

In this job we clone a template guest OS by:

* use virt-clone to clone the template
* resize the disk to the size desired
* set autostart so the vm starts on reboot
* use virt-sysprep to set the correct host name
* set memory and cpus
* 

Per above we are using a network bridge and dhcp networking is already configured for this in the template.

      {
      "id": "clones a kvm guest",
      "working_dir": "./",
      "options": {
        "timeoutms": 360000
      },
      "files": [],
      "script": {
        "env": {
          "PATH": "/opt/local/bin:/opt/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin",
          "TEMPLATE_NAME": "template",
          "NEW_GUEST_NAME": "worker102",
          "DOMAIN": "zenzic.com",
          "RAM": "2",
          "TEMPLATE_DISK_SPACE": "5",
          "DISK_SPACE": "5",
          "DISK": "path=/var/kvm/images/${NEW_GUEST_NAME}.img,size=${DISK_SPACE}",
          "VCPUS": "2",
          "OS_TYPE": "linux",
          "OS_VARIANT": "rhel7",
          "NETWORK_BRIDGE": "br0"
        },
        "commands": [
          {
            "command": "virsh destroy ${NEW_GUEST_NAME} || :"
          },
          {
            "command": "virsh undefine ${NEW_GUEST_NAME}; sleep 15 || :"
          },
          {
            "command": "rm -f /home/kvm/disks/${NEW_GUEST_NAME}.dsk"
          },
          {
            "command": "virt-clone --connect=qemu:///system -o ${TEMPLATE_NAME} -n ${NEW_GUEST_NAME} -f /home/kvm/disks/${NEW_GUEST_NAME}.dsk --force; sleep 10"
          },
          {
            "command": "qemu-img resize /home/kvm/disks/${NEW_GUEST_NAME}.dsk ${DISK_SPACE}G"
          },
          {
            "command": "virt-resize --expand /dev/vda2 --LV-expand /dev/centos/root /home/kvm/disks/${TEMPLATE_NAME}.dsk /home/kvm/disks/${NEW_GUEST_NAME}.dsk"
          },
          {
            "command": "virsh autostart ${NEW_GUEST_NAME}"
          },
          {
            "command": "virt-sysprep -d ${NEW_GUEST_NAME} --hostname ${NEW_GUEST_NAME}.${DOMAIN}"
          },
          {
            "command": "virsh setmaxmem ${NEW_GUEST_NAME} ${RAM}G --config"
          },
          {
            "command": "virsh start ${NEW_GUEST_NAME}"
          },
          {
            "command": "virsh setvcpus ${NEW_GUEST_NAME} ${VCPUS}"
          }
        ]
      }
    }
