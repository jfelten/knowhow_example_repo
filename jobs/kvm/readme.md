# Building a kvm army

This example parallels the [docker example](https://github.com/jfelten/knowhow_example_repo/tree/master/jobs/docker), but uses kvm and libvirtd instead of docker for the virtualization.  There are many cases where a docker container is not enough, and a full blown host or different OS is needed.  Fortunately automating kvm guest creatation is just as easy.

In this example we use centos7, but it can be easily modified for any other OS.

## Install KVM on centos 7

First install the EPEL yum repository on you host system, and then use yum to install all the lib virtd packages.  I found it useful to have the kvm X GUI applications, since the CentOS installer uses a GUI for it's installation.  It makes troubleshooting much easier and they are included in this job.  This job requires root access so run as a user with the appropriate sudo rights.
