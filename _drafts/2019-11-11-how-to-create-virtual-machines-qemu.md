---
title: How to Create Virtual Machines With Qemu
layout: post
date: 2019-11-11T13:52:22Z
draft: true
categories:
  sysadmin
---

This post outlines the steps needed to create a qemu environment within a machine and install virtual machines in that environment.

Software required:
- qemu-kvm
- libvirt
- virt-install
- virt-manager

virt-manager is a graphical interface to the resources, and is optional according to whether you run a graphical interface.

## Networking
One of the important tasks before installing a virtual machine is to set up the virtual networking. This means that as the VM installs, it can work with the networking.

Usually, libvirt comes with a default network bridge which starts when you start the libvirtd service. This appears as virbr0.

To find out what is the name of the bridge, you can use a tool:

    brctl show

### Bridge Control

    brctl show
    brctl addbr br0
    brctl stp br0 on

stp is to stop looping.

    brctl delbr br0

will delete the bridge that we created.

virsh net-define is the command needed to create a new virtual network. virsh net-autostart will autostart the virtual network interface at service start. virsh net-start will start the network interface. virsh net-edit will open up an editor that will let you edit an existing network interface.

* KVM (Kernel Based Virtual Machine)

Hypervisor - two types

