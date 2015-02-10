# Building a kvm army

This example parallels the [docker example](https://github.com/jfelten/knowhow_example_repo/tree/master/jobs/docker), but uses kvm and libvirtd instead of docker for the virtualization.  There are many cases where a docker container is not enough, and a full blown host or different OS is needed.  Fortunately automating kvm guest creatation is just as easy.

In this example we use centos7, but it can be easily modified for any other OS.

## Install KVM on centos 7

First install the EPEL yum repository on your host system, and then use yum to install all the lib virtd packages.  I found it useful to have the kvm X GUI applications, since the CentOS installer uses a GUI for it's installation.  It makes troubleshooting much easier and they are included in this job.  This job requires root access so run as a user with the appropriate sudo rights.

### A note about networking

As with other virtualization platforms networking can be a pain with kvm.  Out of the box kvm installs its own virtual bridge that uses NAT(Netowrk Address Translation) and uses this to assign ips to guests.  Outbound traffic from guests works as part of the install, but inbound network traffic to each guest is blocked.  KVM deals with this by adding a hooks directory in /etc/libvirt/hooks (on centos) where custom ip chain rules can be used to route traffic to your guests.  This gets ugly fast as more guests and ports are added, and doesn't play nice with firewalld on RHEL7 variants.  The solution I used was to set up my own network bridge with its own DHCP server and dynamic dns, so that netowrking works automagically when new guests are installed.  Installing a netowrk bridge, dchpd, and dynamic-dns is major write up unto itself and I won't get into that here.  The last 4 commands commented out for an example on how to use /etc/libvirt/hooks.

## Create a guest template

For my purposes I created a centos 7 instance manually and then wrote a job to install standard software that I need on it.  Whenever I need a new guest I just clone this instance.  It would be possible to automate the template install using a centos kickstart file, but I didn't have the need to do so.

## Job to create a new guest by cloning the template
