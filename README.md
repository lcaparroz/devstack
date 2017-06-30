# Setting up Openstack with DevStack + VirtualBox + CentOS 7

This a follow-through guide for setting up a developer environment for
OpenStack, Ocata (15th) release, using DevStack for deployment in a VirtualBox
virtual machine running CentOS 7.

The main goal is to provide a **minimal environment for using the OpenStack
API in a development context**, on a personal computer. Running OpenStack with
full functionality (as in a production or staging environment) needs robust
hardware and involves a lot of security requirements that is out of this guide
scope.

## 0. Requirements

#### Devstack

This guide is practically based on much of the available information on the
DevStack documentation. DevStack is, as defined in its own documentation,

> DevStack is a series of extensible scripts used to quickly bring up a complete
> OpenStack environment based on the latest versions of everything from git
> master. It is used interactively as a development environment and as the basis
> for much of the OpenStack project’s functional testing.

Further detail of some of the steps in this guide can be found at
https://docs.openstack.org/developer/devstack/.

**Note**: As this guide has been written, the latest OpenStack branch is
**Ocata**, its 15th release (https://www.openstack.org/software/ocata/), so
every information here is based on this release.

#### VirtualBox

You must have VirtualBox installed on your system. Instructions on how to
install depending on the OS you use can be found in the following link
https://www.virtualbox.org/wiki/Downloads.

#### CentOS 7

This guide uses CentOS 7 as the operational system for the virtual machine that
will run OpenStack. You can download the ISO minimal image (without GUI)
at https://wiki.centos.org/Download.

#### 1. VirtualBox Setup

Open Oracle VM VirtualBox Manager and access the menu **File > Preferences**.

In the _Network_ section, select the _Host-only Networks_ tab.

![Image of VirtualBox Preferences, Network
section](/images/1-preferences-network.png)

You need to add 2 new host-only networks:

1. vboxnet0:

   On _Adapter_ tab:

   * **IPv4 Address**: 172.241.0.100
   * **IPv4 Network Mask**: 255.255.255.128

   ![Image of Host-only Network vboxnet0](/images/1-vboxnet0.png)

   On DHCP Server tab, **disable DHCP server**.

   ![Image of Host-only Network DHCP Server
configuration](/images/1-dhcp-server-disabled.png)

2. vboxnet1:

   On _Adapter_ tab:

   * **IPv4 Address**: 172.24.4.100
   * **IPv4 Network Mask**: 255.255.255.128

   ![Image of Host-only Network vboxnet1](/images/1-vboxnet1.png)

   On DHCP Server tab, **disable DHCP server**, as for _vboxnet0_.

## 2. Creating a Virtual Machine on VirtualBox

The following virtual machine configuration is a suggestion for running CentOS
with OpenStack in a personal computer, tested in a laptop with the following
configuration:

* Processor: Intel® Core™ i5-3230M CPU @ 2.60GHz × 4
* Memory: 8 GB RAM
* HDD: 500 GB
* OS: Ubuntu 16.04 LTS

Regarding processor and memory, a better virtual machine configuration can be
achieved, but it should be kept in mind that it might consume valuable hardware
resource from the PC.

On Oracle VM VirtualBox Manager, select **Machine > New** on the menu bar.

Set up a new machine with the following characteristics:

* **Name**: devstack
* **Type**: Linux
* **Version**: Red Hat
* **Memory size**: 2048 MB
* **Hard disk**: option _Create a virtual hard disk now_

![Image of creating a virtual machine](/images/2-creating-vm.png)

After clicking on _Create_, configure the Virtual Hard Disk as follows:

* **File size**: 10 GB
* **Hard disk file type**: option _VDI (VirtualBox Disk Image)_
* **Storage on physical hard disk**: option _Fixed syze_

![Image of creating a virtual hard
disk](/images/2-creating-virtual-hard-disk.png)

The basic setup is done and the virtual machine should be listed on the
VirtualBox Manager. Select the brand-new machine and select **Machine >
Settings**. There will be a few sections and tabs to configure your VM in
further detail:

* **System > Motherboard**:

  * Chipset: ICH9
  * Boot Order: Select both _Optical_ and _Hard Disk_ and make _Optical_ the
    first item in the list.

  ![Image of Machine Settings, System,
Motherboard](/images/2-system-motherboard.png)

* **System > Processor**:

  * Processor(s): 1 CPU

  ![Image of Machine Settings, System,
Processor](/images/2-system-processor.png)

* **System > Acceleration**:

  * Paravirtualization Interface: default
  * Hardware Virtualization: select _Enable VT-x/AMD-V_

  ![Image of Machine Settings, System,
Acceleration](/images/2-system-acceleration.png)

* **Network > Adapter 1**:

  * Select _Enable Network Adapter_
  * Attached to: _NAT_
  * Advanced > Select _Cable connected_

  ![Image of Machine Settings, Network, Adapter
1](/images/2-network-adapter-1.png)

* **Network > Adapter 2**:

  * Select _Enable Network Adapter_
  * Attached to: _Host-only Adapter_
  * Name: _vboxnet0_
  * Advanced > Promiscuous Mode: select _Allow All_
  * Advanced > Select _Cable connected_

  ![Image of Machine Settings, Network, Adapter
2](/images/2-network-adapter-2.png)

* **Network > Adapter 3**:

  * Select _Enable Network Adapter_
  * Attached to: _Host-only Adapter_
  * Name: _vboxnet1_
  * Advanced > Promiscuous Mode: select _Allow All_
  * Advanced > Select _Cable connected_

  ![Image of Machine Settings, Network, Adapter
3](/images/2-network-adapter-3.png)

* **Storage**

  Select an ISO image to be booted when running the machine for the first time:

  * Storage Tree: select _Empty_ on _Controller: IDE_. On _Attributes_ click the
    disk icon and then _Select Virtual Optical Disk File_. Select the Centos 7
    ISO image on your hard disk.

  ![Image of Machine Settings, Storage](/images/2-storage-centos-iso.png)

  **Make sure you correctly configured the boot order in the previous steps.**

* **Audio**

  * Disable Audio (no need for audio)

The VM is configured and you should be able to run it and boot the CentOS image.

When selecting the VM on VirtualBox Manager, the final configuration should
resemble something like this:

![Image of VM final configuration](/images/2-vm-final-configuration.png)

**Note**: Setting up a virtual machine with 4096 MB (4 GB) of memory and 2 CPU
might provide a better fluid experience for using OpenStack DashBoard and API
simultaneously. However, as mentioned before, the PC overall performance might
be compromised.

#### Gathering information about network adapters

Before proceeding to the next step for _Installing CentOS 7 on the Virtual
Machine_, it will be very useful to have information about each network adapter
for detecting which adapter an interface on Linux corresponds to.

Access **Machine > Settings** and go to _Network_ section. There, on each
adapter tab, expand the _Advanced_ section and take note of the adapter MAC
Address, associating it with the its number and to what it is attached:

| VM Network Adapter | Attached To | MAC Address |
| :---: | :---: | :---: |
| 1 | NAT      | 080027F90EC0 |
| 2 | vboxnet0 | 080027CBFC33 |
| 3 | vboxnet1 | 080027A73390 |


#### Taking a Virtual Machine Snapshot

From now on, you can take _VM snapshots_ for each step, giving them meaningful
names, so you can step back if something goes wrong. For example, at finishing
this step, the VM is configured and the CentOS 7 image is loaded into the IDE
Optical Driver (like a CD). You can take a snapshot at this point so you always
have a powered-off machine at hand if you want to try another configuration or
another ISO image.

To take a snapshot, on VirtualBox manager, click on _Snapshots_ (right top
corner), select the _Current State_ line and either click on the camera icon or
click with your mouse right button over the _Current State_ line and select
_Take Snapshot_.

![Image of taking a virtual machine snapshot](/images/2-taking-a-snapshot.png)

Then, give the snapshot a name and add some useful information so it describe
the virtual machine state.

![Image of naming a virtual machine snapshot](/images/2-naming-a-snapshot.png)

After this, the snapshot will be added to the VM snapshot list. The red square
with the camera icon in the snapshot line indicates that when the snapshot was
taken, the virtual machine was powered off. In later steps, it will be very
useful to take snapshots when the machine status is _running_, so you can
quickly set an environment up without bothering making additional steps.

![Image of a virtual machine snapshot taken](/images/2-snapshot-taken.png)

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
      keyboard;
   3. Click on **System > Installation Destination** and then on _Done_. This
      will create an automatic partition scheme. Unless you need a custom setup
      for the partitions, the automatic scheme suffices for running DevStack;
   4. On **System > Kdump** disable the _kdump_ feature (it increases memory
      usage and it is not needed for running DevStack);
   5. Click on **Network & Host Name**. There should be three Ethernet
      interfaces listed, each with a name (enp0s**X**) and a MAC Address, that
      can be visualized when selecting the interface, Each one corresponds to a
      VM a network adapters. Use the information gathered on **2. Creating a
      Virtual Machine on VirtualBox** to identify which network adapter
      corresponds to a interface or check the MAC Addresses of the adapters on
      the VirtualBox Manager by selecting the VM and then **Machine > Settings >
      Network > Adapter > Advanced > MAC Address**. Keep this information in
      hand for the next steps:

      | VM Network Adapter | Attached To | MAC Address | Interface |
      | :---: | :---: | :---: | --- |
      | 1 | NAT      | 080027F90EC0 | Ethernet (enp0s3) |
      | 2 | vboxnet0 | 080027CBFC33 | Ethernet (enp0s8) |
      | 3 | vboxnet1 | 080027A73390 | Ethernet (enp0s9) |

      Identify which interface corresponds to _Adapter 1_ (attached to NAT).
      This interface will be used to connect the VM to the internet for
      installing packages and cloning repositories. Select it from the list and
      then turn it on. After a few seconds, its status should be _Connected_.

      ![Image of CentOS network interfaces at
installation](/images/3-centos-network.png)

   6. The setup for installation is done, click on _Begin installation_.

      ![Image of CentOS installation setup
done](/images/3-centos-installation-setup.png)

5. After installation begins, a screen titled _Configuration_ will be shown. On
   **User Settings > Root Password**, enter a password for the root user. It
   will be used to access the machine and the installation only finishes after
   setting it up. There is no need to create another user at this step.
6. Once the installation succeeds, after a short time you will be prompted to
   reboot the machine and then click on _Reboot_.
7. After successfully rebooting, the machine should boot on the hard disk, load
   CentOS and then a prompt for login will be displayed. The CentOS installation
   is done.

**Tip**: It is recommended to take a snapshot of the VM right after installing
the OS. It can be useful in the case of DevStack installation messes up.

## 4. Configuring CentOS

After installing CentOS 7 on the machine, you should be able to log in with root
credentials, using the password you entered during the installation.

1. Log in at localhost with username _root_ and its password.

   ![Image of CentOS login prompt](/images/4-centos-login-prompt.png)

2. To both test the internet and update the repositories and packages:

   ```
   $ yum update -y
   ```

   If your host machine have a working connection to the internet, the VM should
   be able to update the packages.

3. Before using SSH to connect to the VM, it is necessary to configure the
   Ethernet interfaces. On CentOS 7 the configuration files for network
   interfaces can be found in `/etc/sysconfig/network-scripts/`.

   To list the files corresponding to those interfaces, execute:

   ```
   $ ls -1 /etc/sysconfig/network-scripts/ifcfg*

   /etc/sysconfig/network-scripts/ifcfg-enp0s3
   /etc/sysconfig/network-scripts/ifcfg-enp0s8
   /etc/sysconfig/network-scripts/ifcfg-enp0s9
   /etc/sysconfig/network-scripts/ifcfg-lo
   ```

   Supposing the interfaces listed earlier, the interface **enp0s8** attached to
   _vboxnet0_ will be configured with an static IP address so you be able to
   access the VM through SSH and to use the OpenStack dashboard. The interface
   **enp0s9** will allow external access to the instances through a
   bridge created during OpenStack deployment with DevStack.

   Edit the files with the following content:

   Interface **enp0s8** (/etc/sysconfig/network-scripts/ifcfg-enp0s8):

   ```
   TYPE=Ethernet
   DEVICE=enp0s8
   BOOTPROTO=static
   ONBOOT=yes
   NETWORK=172.241.0.0
   NETMASK=255.255.255.128
   BROADCAST=172.241.0.127
   IPADDR=172.241.0.101
   USERCTL=no
   ```

   **Note**: An example file can be found in this repository:
   [ifcfg-enp0s8](/config-files/ifcfg-enp0s8)

   Interface **enp0s9** (/etc/sysconfig/network-scripts/ifcfg-enp0s9):

   ```
   TYPE=Ethernet
   DEVICE=enp0s9
   BOOTPROTO=none
   ONBOOT=yes
   NETWORK=172.24.4.0
   NETMASK=255.255.255.128
   BROADCAST=172.24.4.127
   IPADDR=172.24.4.101
   USERCTL=no
   ```

   **Note**: An example file can be found in this repository:
   [ifcfg-enp0s9](/config-files/ifcfg-enp0s9)

   Check the files content  with the _cat_ command:

   ```
   $ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
   $ cat /etc/sysconfig/network-scripts/ifcfg-enp0s9
   ```

   After changing the content in each file, run the following command to restart
   the network service on CentOS:

   ```
   $ service network restart
   ```

   This command will apply the changes you made to the interfaces, so they
   should have the IP addresses described in the configuration files. The IP
   addresses can be checked with the commands below:

   ```
   $ ip addr show enp0s8 | grep "inet "
   $ ip addr show enp0s9 | grep "inet "
   ```

   ![Image fir ip addr show command](/images/4-ip-addr-show.png)

   From now on, you should be able to connect to your guest VM from your host
   terminal. To test it, open a terminal on your PC an issues the command as
   follow:

   ```
   $ ssh root@172.241.0.101
   ```

   If everything is OK, you be able to connect to your VM.

#### Editing Configuration Files

After a CentOS fresh installation, the only text editor available on the system
is _vi_ (if you prefer another one, you can install with `$ yum install -y
TEXT_EDITOR`). As an example, open a file for edition as follows:

```
$ vi /etc/sysconfig/network-scripts/ifcfg-enp0s8
```

If you do not feel comfortable with the _vi_ editor, you can use _curl_ combined
with Linux/Unix piping to get the content directly from this repository and
overwrite the files. The commands below use the links for the raw content of the
GitHub repository files:

For interface **enp0s8**:

```
$ curl -L \
  https://github.com/lcaparroz/devstack/raw/master/config-files/ifcfg-enp0s8 > \
  /etc/sysconfig/network-scripts/ifcfg-enp0s8
```

For interface **enp0s9**:

```
$ curl -L \
  https://github.com/lcaparroz/devstack/raw/master/config-files/ifcfg-enp0s9 > \
  /etc/sysconfig/network-scripts/ifcfg-enp0s9
```

![Image for curl command and piping to configuration
file](/images/4-curl-command.png)

The `-L` option for _curl_ makes the command follow redirections should the
response return an HTTP code for redirection. You can just use _curl_ with the
URL after redirection. This final URL can be obtained by browsing this
repository, accessing a file and clicking on **Raw**. The URL should look like
this:

https://raw.githubusercontent.com/lcaparroz/devstack/master/config-files/ifcfg-enp0s8

#### Installing additional packages

To install OpenStack, DevStack scripts use the _git_ command. The minimal CentOS
7 image does not include _git_ by default. Install it with:

```
$ yum install -y git
```

Although not required, you can also install _nmap_ to scan the VM ports. After
installing OpenStack, port 80 is not open by default.

```
$ yum install -y nmap
```

## 5. Installing OpenStack

DevStack needs a _stack_ user to run commands and scripts. If you try to run the
scripts with root privilege (with root user or using sudo), execution will be
stopped and you will be warned.

To create the _stack_ user, run these commands:

```
$ sudo useradd -s /bin/bash -d /opt/stack -m stack
$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
```

With the commands above, the _stack_ user does not have a password. To use the
system as _stack_ user, run:

```
$ su - stack
```

Within the _stack_ user session, clone the DevStack repository to the VM and
change the current working directory:

```
$ git clone https://github.com/openstack-dev/devstack.git /opt/stack/devstack
$ cd /opt/stack/devstack
```

**Note**: The cloned repository will have the _master_ branch files. To use
**Ocata** branch files, the release tested in this guide, execute the command
below from within directory `/opt/stack/devstack`:

```
$ git checkout stable/ocata
```

Always check the available DevStack branches at
https://github.com/openstack-dev/devstack/branches together with OpenStack ones
at https://git.openstack.org/cgit. Note that even though the DevStack branches
may refer to older OpenStack releases, it does not mean the scripts will be able
to clone the right OpenStack repositories for each module.  For example, at the
writing of this guide, DevStack have two "_stale branches_" referring to
**Kilo** and **Liberty** releases, but the OpenStack repositories for the _Nova_
module (https://git.openstack.org/cgit/openstack/nova/) does not include these
releases anymore. You can set the branch for each individual module on
`local.conf` file, explained in the next section.

#### local.conf

DevStack script **stack.sh** uses the file `/opt/stack/devstack/local.conf` to
configure the installation process, setting passwords, IP address ranges and
other parameters. Although the DevStack documentation mentions a minimum
`local.conf` file to install OpenStack, defining some parameters at this step
are useful as DevStack takes care of them, saving developer's effort in
post-installation. Based on the VM configuration, the `local.conf` file should
look like this:

```
[[local|localrc]]
HOST_IP=172.241.0.101
SERVICE_HOST=$HOST_IP
DEST=/opt/stack

Q_USE_SECGROUP=True

FIXED_RANGE=10.0.0.0/25
IPV4_ADDRS_SAFE_TO_USE=10.0.0.0/25
NETWORK_GATEWAY=10.0.0.1

FLOATING_RANGE=172.24.4.0/25
Q_FLOATING_ALLOCATION_POOL=start=172.24.4.111,end=172.24.4.120
PUBLIC_NETWORK_GATEWAY=172.24.4.1
PUBLIC_INTERFACE=enp0s9

# Open vSwitch provider networking configuration
Q_USE_PROVIDERNET_FOR_PUBLIC=True
OVS_VLAN_RANGE=physnet1
PHYSICAL_NETWORK=physnet1
OVS_PHYSICAL_BRIDGE=br-ex
PUBLIC_BRIDGE=br-ex
OVS_BRIDGE_MAPPINGS=public:br-ex

ADMIN_PASSWORD=secret
DATABASE_PASSWORD=$ADMIN_PASSWORD
MYSQL_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

SERVICE_TOKEN=token
```

If you have restriction on the IP addresses you can use, change the file
accordingly.

To set repository branches for each module, add the respective lines to
`local.conf`. Taking the branch `stable/ocata` as an example, to set both _Nova_
and _Glance_ modules branches, add the following lines:

```
NOVA_BRANCH=stable/ocata
GLANCE_BRANCH=stable/ocata
```

The DevStack repository includes a `local.conf` sample file which includes some
of the modules whose branches can be set
(`/opt/stack/devstack/sample/local.conf`). Check the OpenStack documentation to
get further detail.

**Note**: You can use your preferred text editor, the command `curl` or transfer
a file from your host machine (PC) to your guest VM through SSH to get
`local.conf`.  To use the later option and assuming that there is a file named
`local.conf` on the current working directory of the host machine, run the
command `scp` on host:

```
$ scp ./local.conf root@172.241.0.101:/opt/stack/devstack
```

**Tip**: Right now is a good moment to take a snapshot from the VM. The DevStack
scripts make a lot of changes to the system and take long to complete.

#### Running stack.sh

After setting up `local.conf`, run the `stack.sh` script to trigger the
OpenStack installation:

```
$ ./stack.sh
```

The installation process can take from 20 minute to about one hour. Always keep
an eye on the screen output, it's easy to the installation to stop due to an
error, even though this guide was thought to run smoothly. Make sure the
internet connection is working with a good speed, as the scripts will download
several applications and clone git repositories and timeouts can stop them.

Once the installation is done, the script output shows some information about IP
addresses, how to access the dashboard and default users.

![Image of OpenStack successful
installation](/images/5-successful-installation.png)

Although the OpenStack dashboard is available and running after installation,
the system firewall is blocking access to the HTTP service port 80 and trying to
access it through a web browser will result in an error. Port 80 is open but it
must be added to the firewall rules thought the _iptables_ service. Given that
_nmap_ is installed on the VM OS, run the command below to check which ports are
open:

```
$ sudo nmap -sT -O localhost
```

To add a rule for access through port 80 on the VM and make changes permanent,
run:

```
$ sudo iptables -I INPUT -p tcp --destination-port 80 -j ACCEPT
$ sudo service iptables save
```

Now the access to the HTTP service is granted and you can use one of the
credentials to log in to the dashboard, either as _demo_ user or as _admin_
user. Their passwords were set in the `local.conf` file and they are also shown
after the `stack.sh` succeeds. Access the dashboard at
http://172.241.0.101/dashboard.

#### API Access

After accessing the dashboard, you can check which API endpoints are available
in the project by selecting **Project > API Access** in the dashboard left bar.

**Tip**: Once the Dashboard and the API endpoints are working, a snapshot of the
VM provides a way to always have OpenStack up quickly and working.

## 6. Accessing OpenStack instances

Before going on more details on how to create key pairs and security rules, it
is important to configure access to the OpenStack command line interface.

#### Using the OpenStack command line interface (CLI)

OpenStack have a command line interface with which you can manage your projects,
with access to the same functions in the dashboard and some more. After DevStack
scripts finish, although the command line interface is available, the system
environment is missing authentication parameters:

```
$ openstack image list
Missing value auth-url required for auth plugin password
```

In `/opt/stack/devstack` there is a script called `openrc` that export some
variables to the environment, granting access to the OpenStack CLI with
credentials from the _demo_ user. Run the command below to set the required
environment variables:

```
$ . /opt/stack/devstack/openrc
```

To get _admin_ access, you can log into the dashboard with _admin_ credentials,
click on the user name (_admin_) at the top navigation bar and then on
"OpenStack RC File" (either _v3_ or _v2_, depending on which Identity API
version you are going to use) or go to **Project > API Access**, click on the
button "Download OpenStack RC file" and choose the version.

![Image of OpenStack Dashboard, API Access page](/images/5-api-access-admin.png)

Transfer the downloaded file from the host machine to the guest VM with the
command _scp_ and execute it, for example:

**On host machine**:

```
$ scp ./demo-openrc root@172.241.0.101:/opt/stack/devstack
```

**On guest VM**:

```
$ . /opt/stack/destack/demo-openrc
```

#### Configuring access and security for instances

The following information is based on the OpenStack User Guide documentation,
available at
https://docs.openstack.org/user-guide/cli-nova-configure-access-security-for-instances.html.

Before launching an instance, it is useful to configure default security rules
and key pairs so you can always access it through SSH.

1. **Key Pair**

   There a few ways to create a default key pair in the OpenStack project. This
   guide uses a public SSH key generated at the host machine and import into the
   OpenStack project, allowing only the host machine to access instances.

   1. On host machine, run the following commands:

      ```
      $ cd ~/.ssh
      $ ssh-keygen -t rsa -f devstack.key
      $ scp ./devstack.key.pub root@172.241.0.101:/opt/stack/
      ```

   2. On guest VM, execute the commands below to import the ssh key to the
      openstack project:

      ```
      $ mkdir ~/.ssh
      $ mv /opt/stack/devstack.key.pub ~/.ssh
      $ openstack keypair create --public-key ~/.ssh/devstack.key.pub host_pc
      ```

   3. If the key pair was imported, its fingerprint, name and user_id will be
      printed out on screen. You can also check if the key pair was added with

      ```
      $ openstack keypair list
      ```

2. **Security Group Rules**

   OpenStack has a default security group, from which all instances inherit
   security rules if you do not specify them manually. The _default_ security
   group does not include neither rules to allow SSH access to the instances nor
   rules to allow pinging them, so these rules must be added manually.

   1. On guest VM, to allow SSH access and pinging, run:

      ```
      $ openstack security group rule create default \
        --protocol tcp --dst-port 22:22 --remote-ip 0.0.0.0/0
      $ openstack security group rule create --protocol icmp default
      ```

   2. After the rules were added, them can be checked with the command

      ```
      $ openstack security group rule list default
      ```

Now, you can launch an instance through the dashboard or through the Compute API
(_Nova_ module), associate a floating IP address to it and test the access.

**Tip**: The instance flavors you can use are limited to the VM hardware
configuration, so choose wisely. Creating new volumes for instances resulted in
OpenStack crashing, so it is another point to pay attention when launching new
instances.

---

**End Note**: The purpose of this guide is not to replace the official
OpenStack documentation. Always check it to get updated information and for
troubleshooting.
