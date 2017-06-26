# Setting up Openstack with DevStack + VirtualBox + CentOS 7

## 0. Requirements

You must have VirtualBox installed on your system. Instructions on how to
install depending on the OS you use can be found in the following link
https://www.virtualbox.org/wiki/Downloads.

This guide uses CentOS 7 as the operational system for the virtual machine that
will run OpenStack. You can download the ISO minimal image (without GUI)
at https://wiki.centos.org/Download.

## 1. VirtualBox Setup

Open Oracle VM VirtualBox Manager and access the menu **File > Preferences**.

In the _Network_ section, select the _Host-only Networks_ tab.

You need to add 2 new host-only networks:

1. vboxnet0:

   * **IPv4 Address**: 172.16.0.0
   * **IPv4 Network Mask**: 255.255.0.0

2. vboxnet1:

   * **IPv4 Address**: 10.0.0.0
   * **IPv4 Network Mask**: 255.255.255.0

## 2. Creating a Virtual Machine on VirtualBox

On Oracle VM VirtualBox Manager, select **Machine > New** on the menu bar.

Set up a new machine with the following characteristics:

* **Name**: devstack
* **Type**: Linux
* **Version**: Red Hat
* **Memory size**: 2048 MB
* **Hard disk**: option _Create a virtual hard disk now_

After clicking on _Create_, configure the Virtual Hard Disk as follow:

* **File size**: 10 GB
* **Hard disk file type**: option _VDI (VirtualBox Disk Image)_
* **Storage on physical hard disk**: option _Fixed syze_

The basic setup is done and the virtual machine should be listed on the
VirtualBox Manager. Select the brand-new machine and select **Machine >
Settings**. There will be a few sections and tabs to configure your VM in
further detail:

* **System > Motherboard**:

  * Chipset: ICH9
  * Boot Order: Select both _Optical_ and _Hard Disk_ and make _Optical_ the
    first item in the list.

* **System > Processor**:

  * Processor(s): 1 CPU

* **System > Acceleration**:

  * Paravirtualization Interface: default
  * Hardware Virtualization: select _Enable VT-x/AMD-V_

* **Network > Adapter 1**:

  * Select _Enable Network Adapter_
  * Attached to: _NAT_
  * Advanced > Select _Cable connected_

* **Network > Adapter 2**:

  * Select _Enable Network Adapter_
  * Attached to: _Host-only Adapter_
  * Name: _vboxnet0_
  * Advanced > Promiscuous Mode: select _Allow All_
  * Advanced > Select _Cable connected_

* **Network > Adapter 3**:

  * Select _Enable Network Adapter_
  * Attached to: _Host-only Adapter_
  * Name: _vboxnet1_
  * Advanced > Promiscuous Mode: select _Allow All_
  * Advanced > Select _Cable connected_

* **Storage**

  Select an ISO image to be booted when running the machine for the first time:

  * Storage Tree: select _Empty_ on _Controller: IDE_. On _Attributes_ click the
    disk icon and then _Select Virtual Optical Disk File_. Select the Centos 7
    ISO image on your hard disk.

    **Make sure you correctly configured the boot order in the previous steps.**

The VM is configured and you should be able to run it and boot the CentOS image.

> **Tip**: from now on, you can take snapshots for each step, naming them with
> meaningful names, so you can step back if something goes wrong.

## 3. Installing CentOS 7 on the Virtual Machine

Although the CentOS 7 minimal image does not include a windows/desktop manager,
the installation process provides a graphical user interface. Going through the
steps is intuitive and straightforward. If you have any doubts on how to proceed
or if you want to install CentOS from the command line you can find valuable
information at https://wiki.centos.org/.

Follow the next steps to install CentOS 7 throught the GUI:

1. On VirtualBox Manager, select the _devstack_ VM and then **Machine > Start**;
2. Wait the VM to boot and select _Install CentOS Linux 7_;
3. After loading the necessary files, an user interface will be displayed. On
   it, select a language and click on _Continue_;
4. Once the _Installation Summary_ is shown, perform a basic setup for the
   installation:

   1. On **Localization > Date & Time**, set the machine timezone or adjust the
      time properly;
   2. On **Localization > Keyboard**, select a keyboard layout that matches your
      keyaboard;
   3. Click on **System > Installation Destination** and the on _Done_. This
      will create an automatic partition scheme. Unless you need a custom setup
      for the partitions, the automatic scheme suffices for running DevStack;
   4. On **System > Kdump** disable the _kdump_ feature (it increases memory
      usage and it is not needed for running DevStack);
   5. Click on **Network & Host Name**. There should be three Ethernet
      interfaces listed, each with a name (enp0s**X**) and a MAC Address,
      corresponding to the VM network adapters. To identify which network
      adapter corresponds to a interface, you can check the MAC Addresses of the
      adapters on the VirtualBox Manager by selecting the VM and then
      **Machine > Settings > Network > Adapter > Advanced > MAC Address**. Keep
      this information in hand for future steps.

      Identify interface corresponds to _Adapter 1_ (attached to NAT). This
      interface will be used to connect the VM to the internet for installing
      packages and cloning repositories. Selec it from the list and then turn it
      on. After a few seconds, its status should be _Connected_.

      The interfaces should look somehow like the example below:

      | Interface | Attached To | VM Network Adapter |
      | --- | :---: | :---: |
      | Ethernet (enp0s3) | NAT | 1 |
      | Ethernet (enp0s8) | vboxnet0 | 2 |
      | Ethernet (enp0s9) | vboxnet1 | 3 |

   6. Click on _Begin installation_.

5. After installation begins, a screen titled _Configuration_ will be shown. On
   **User Settings > Root Password**, enter a password for the root user. It
   will be use to access the machine and the installation only finished afer
   setting it up. There is no need to create another use at this step.
6. After the installation succeed, click on _Finish configuration_ and wait all
   the process to finish. After a short time you will be prompted to reboot the
   machine and the click on _Reboot_.
7. After successfully rebooting, the machine should boot on the hard disk, load
   CentOS and then a prompt for login will be displayed. The CentOS installation
   is done.

> **Tip**: a snapshot of the VM right after installing the OS is a good resource
> in case DevStack installation messes up.

## 4. Configuring CentOS

After installing CentOS 7 on the machine, you should be able with root
credentials, using the password you entered during the installation.

1. Log in at localhost with username _root_ and its password.
2. To both test the internet and update the repositories and packages:

   `yum update -y`

   If your host machine have a working connection to the internet, the VM should
   be able to update the packages.

3. Before using SSH to connect to the VM, it is necessary to configure the
   Ethernet interfaces. On CentOS 7 the configuration files for network
   interfaces can be found in `/etc/sysconfig/network-scripts/`.

   To list the files corresponding to those interfaces, execute:

   `ls -la /etc/sysconfig/network-scripts/ifcfg*`

   Supposing the interfaces listed on the previous step, the interface
   **enp0s8** attached to _vboxnet0_ will be configured with an static IP
   address so you be able to access the VM through SSH and to use the OpenStack
   dashboard. After a CentOS fresh install, the only text editor available on
   the system is _vi_ (if you prefer another one, you can install with `yum
   install -y TEXT_EDITOR`). Open the file for edition as follow:

   `vi /etc/sysconfig/network-scripts/ifcfg-enp0s8`

   Edit the file with the following parameter list:


   ```
TYPE=Ethernet
DEVICE=enp0s8
BOOTPROTO=none
ONBOOT=yes
NETWORK=172.16.0.0
NETMASK=255.255.0.0
BROADCAST=172.16.255.255
IPADDR=172.16.0.1
USERCTL=no
   ```
---
### TO-DO: Step 4 => How to configure enp0s9 to public interface with br-ex
---

4. 
