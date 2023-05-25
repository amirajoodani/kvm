\# yum install qemu-kvm

\# rpm -qa | grep qemu

#qemu-

\# qemu-img -h | grep Supported

\# qemu-img create -f raw debian.img 10G     (create raw image with 10GB volume)

\# file -s debian.img   (check the data )

#qemu-img info debian.img  (info about image )

#modprobe nbd

\# qemu-nbd --format=raw --connect=/dev/nbd0   (Using the qemu-nbd tool, associate the blank image file to the /dev/nbd0 block device)

\# sfdisk /dev/nbd0 << EOF   (Create two partitions on the block device. One will be used for swap, and the other as the root partition for the guest OS)

#ls -la /dev/nbd0\* (List the available block devices after the partitioning:)

\# mkswap /dev/nbd0p1  (Create the swap partition)

\# mkfs.ext4 /dev/nbd0p2  (Make the EXT4 filesystem on the root partition:)

\# file -s /dev/nbd0

\# file -s debian.img

\# apt install -y debootstrap

\# mount /dev/nbd0p2 /mnt/ (Mount the root partition from the Network Block Device (NBD)device and ensure that it was mounted successfully:)

\# mount | grep mn

root@kvm:~# debootstrap --arch=amd64 --include="openssh-server

ls -lah /mnt/  (Ensure the root filesystem was created, by listing all the files at the mounted location)

root@kvm:~# mount --bind /dev/ /mnt/dev (Bind and mount the devices directory from the host to the image filesystem)

root@kvm:~# ls -la /mnt/dev/ | grep nbd0   (Ensure that the nbd devices are now present inside the mount location)

\# chroot /mnt/   (Change the directory namespace to be the root filesystem of the image and ensure the operation succeeded)

\# pwd

\# cat /etc/debian\_version (Check the distribution version inside the chroot environment)

\# root@kvm:/# mount -t proc none /proc  (Mount the proc and sysfs virtual filesystems inside the chrooted environment:)

\# root@kvm:/# mount -t sysfs none /sys

root@kvm:/# apt-get install -y --force-yes linux-image-amd64 grub2  (While still inside the chrooted location, install the Debian kernel metapackage and the grub2 utilities)

NOTE : If asked to select target device for GRUB to install on, do not select any and just continue

root@kvm:/# grub-install /dev/nbd0 --force (Install GRUB on the root device)

root@kvm:/# update-grub2  (Update the GRUB configs and the initrd image)

root@kvm:/# passwd  (Change the root password of the guest:)

root@kvm:/# echo "pts/0" >> /etc/securetty  (Allow access to the pseudo Terminal inside the new guest OS)

root@kvm:/# systemctl set-default multi-user.target (Change the systemd run level to the multi-user level:)

root@kvm:/# echo "/dev/sda2 / ext4 defaults,discard 0 0" > /etc  (Add the root mountpoint to the fstab file, so it can persist reboots)

root@kvm:/# umount /proc/ /sys/ /dev/ (Unmount the following filesystems as we are done using them for now)

root@kvm:/# exit (Exit the chrooted environment)

root@kvm:~# grub-install /dev/nbd0 --root-directory=/mnt --modules="  (Install GRUB on the root partition of the block device associated with the raw image)

root@kvm:~# sed -i 's/nbd0p2/sda2/g' /mnt/boot/grub/grub.cfg (Update the GRUB configuration file to reflect the correct block device for the guest image)

root@kvm:~# umount /mnt (Unmount the nbd0 device:)

root@kvm:~# qemu-nbd --disconnect /dev/nbd0  (Disassociate the nbd0 device from the raw image)

(resize the image):

root@kvm:~# apt install kpartx

root@kvm:~# qemu-img info debian.img

#qemu-img resize -f raw debian.img +10GB (add 10GB) (just raw image support resize . if you have other image format first change format to raw and then resize it )

#qemu-img info debian.img

#losetup -f (Print the name of the first unused loop device:)

#losetup /dev/loop1 debian.img (Associate the first unused loop device with the raw image file:)

root@kvm:~# kpartx -av /dev/loop1 (Read the partition information from the associated loop device and create the device mappings)

root@kvm:~# ls -la /dev/mapper (Examine the new device maps, representing the partitions on the raw image:)

#tune2fs -l /dev/mapper/loop1p2 (Obtain some information from the root partition mapping:)

root@kvm:~# e2fsck /dev/mapper/loop1p2 (Check the filesystem on the root partition of the mapped device:)

root@kvm:~# tune2fs -O ^has\_journal /dev/mapper/loop1p2 (Remove the journal from the root partition device:)

root@kvm:~# tune2fs -l /dev/mapper/loop1p2 | grep "features" (Ensure that the journaling has been removed:)

root@kvm:~# kpartx -dv /dev/loop1 (Remove the partition mappings:)

root@kvm:~# losetup -d /dev/loop1 (Detach the loop device from the image:)

root@kvm:~# qemu-nbd --format=raw --connect=/dev/nbd0 debian.i (Associate the raw image with the network block device:)

root@kvm:~# fdisk /dev/nbd0 (Using fdisk, list the available partitions, then delete the root partition,recreate it, and write the changes:)

root@kvm:~# fdisk /dev/nbd0 (Using fdisk, list the available partitions, then delete the root partition, recreate it, and write the changes)

root@kvm:~# losetup /dev/loop1 debian.img (Associate the first unused loop device with the raw image file)

root@kvm:~# kpartx -av /dev/loop1 (Read the partition information from the associated loop device and create the device mappings)

root@kvm:~# e2fsck -f /dev/mapper/loop1p2 (After the partitioning is complete, perform a filesystem check:)

root@kvm:~# resize2fs /dev/nbd0p2 (Resize the filesystem on the root partition of the mapped device:)

root@kvm:~# tune2fs -j /dev/mapper/loop1p2 (Create the filesystem journal because we removed it earlier:)

root@kvm:~# kpartx -dv /dev/loop1 (Remove the device mappings:)

root@kvm:~# losetup -d /dev/loop1

(Using pre-existing images)

root@kvm:~tmp# wget https://people.debian.org/~aurel32/qemu/amd64/debian\_--2017-03-09 (Download the image using wget:)

root@kvm:~# qemu-img info debian\_wheezy\_amd64\_standard.qcow2 (Inspect the type of the image:)

root@kvm:/tmp# wget https://cloud.centos.org/centos/7/images/CentOS-7-x86\_64-GenericCloud.qcow2

root@kvm:~# qemu-img info CentOS-7-x86\_64-GenericCloud.qcow2

(Running virtual machines with qemusystem-\*)

root@kvm:~# ls -la /usr/bin/qemu-system-\*

root@kvm:~# qemu-system-x86\_64 --cpu help (Let's have a look at what CPU architectures QEMU supports on the host system:)

root@kvm:~# qemu-system-x86\_64 -name debian -vnc 146.20.141.254 (Start a new QEMU virtual machine using the x86\_64 CPU architecture:)

root@kvm:~# pgrep -lfa qemu (Ensure that the instance is running:)

root@kvm:~# pkill qemu (Terminate the Debian QEMU instance:)

root@kvm:~# qemu-system-x86\_64 -vnc 146.20.141.254:0 -m 1024 -hda CentOS- (Start a new QEMU instance using the prebuilt CentOS image:)

root@kvm:~# pgrep -lfa qemu (Ensure that the instance is running:)

root@kvm:~# pkill qemu (Terminate the CentOS QEMU instance:)

(Starting the QEMU VM with KVM support)

root@kvm:~# cat /proc/cpuinfo | egrep "vmx|svm" | uniq

\# modeprobe kvm

#root@kvm:~# qemu-system-x86\_64 -name debian -vnc 146.20.141.254:0 -m ()

root@kvm:~# pgrep -lfa qemu

root@kvm:~# pkill qemu

#apt install qemu-kvm (As an alternative to directly running the qemu-system-\* commands, on Ubuntu systems there's the qemu-kvm package that provides the /usr/bin/kvm binary.)

root@kvm:~# kvm -name debian -vnc 146.20.141.254:0 -cpu Nehalem -m

root@kvm:~# pgrep -lfa qemu

(Connecting to a running instance with VNC)

root@kvm:~# qemu-system-x86\_64 -name debian -vnc 146.20.141.254: (Start a new KVM-accelerated qemu instance:)

root@kvm:~# pgrep -lfa qemu (Ensure that the instance is running:)

Start the VNC client and connect to the VNC server on the IP address and display port you specified in step 1:

\------------------------------------------------------------------------------------------------------------------

(To install libvirt from packages and source follow the following steps:)

On Ubuntu, install the package by running:

root@kvm:~# apt update && apt install libvirt-bin

root@kvm:~# pgrep -lfa libvirtd (Ensure that the libvirt daemon is running by executing:)

root@kvm:~# cat /etc/libvirt/libvirtd.conf | grep -vi "#" (Examine the default configuration:)

root@kvm:~# vim /etc/libvirt/qemu.conf (Disable the security driver in QEMU by editing the qemu configuration file as follows:)

root@kvm:~# /etc/init.d/libvirt-bin restart (Restart the libvirt daemon:)

(Depending on your Linux distribution, the name of the libvirt service may be different. On RHEL/CentOS, the name of the service is libvirtd; to restart it, run service libvirtd restart.)

\----------------------

qemu.conf is the main configuration file for the QEMU driver that

libvirt uses. We can configure options such as the VNC server

address, the security driver that we saw in step 4 and the user and

group for the QEMU process.

Once we create a QEMU/KVM virtual machine,

the /etc/libvirt/qemu/ directory will contain the XML configuration

definition for that instance, as we are going to see in the following

recipes.

Finally, the /etc/libvirt/qemu/networks/ directory contains

configuration files for the networking.

\---------------------------

Defining KVM instances:

In this recipe, we are going to define a virtual instance by creating a simple XML configuration file that libvirt can use to build the virtual machine.

root@kvm:~# virsh list --all (List all virtual machines on the host OS)

Create the following XML definition file:

root@kvm:~# cat kvm1.xml

<domain type='kvm' id='1'>

<name>kvm1</name>

<memory unit='KiB'>1048576</memory>

<vcpu placement='static'>1</vcpu>

<os>

<type arch='x86\_64' machine='pc-i440fx-trusty'>hvm</type>

<boot dev='hd'/>

</os>

<on\_poweroff>destroy</on\_poweroff>

<on\_reboot>restart</on\_reboot>

<on\_crash>restart</on\_crash>

<devices>

<emulator>/usr/bin/qemu-system-x86\_64</emulator>

<disk type='file' device='disk'>

<driver name='qemu' type='raw'/>

<source file='/tmp/debian.img'/>

<target dev='hda' bus='ide'/>

<alias name='ide0-0-0'/>

<address type='drive' controller='0' bus='0' target='0' unit='0'/>

</disk>

<interface type='network'>

<source network='default'/>

<target dev='vnet0'/>

<model type='rtl8139'/>

<alias name='net0'/>

<address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='</interface>

<graphics type='vnc' port='5900' autoport='yes' listen='146.20.141.158'>

<listen type='address' address='146.20.141.158'/>

</graphics>

</devices>

<seclabel type='none'/>

</domain>

root@kvm:~#

Define the virtual machine:

root@kvm:~# virsh define kvm1.xml

Domain kvm1 defined from kvm1.xml

List all instances in all states:

root@kvm:~# virsh list --all

Id Name State

\----------------------------------------------------

- kvm1 shut off

better toools for making template of vm :

root@kvm:~# apt install virtinst

Next, we define and start the new instance by invoking the virtinstall command (if an instance with the same name already exist, you'll need to destroy and undefine it first :

root@kvm:~# virt-install --name kvm1 --ram 1024 --disk path=/tmp/debian.Starting install...

Creating domain... | 0 B 00:00

Domain creation completed. You can restart your domain by running:

virsh --connect qemu:///system start kvm1

The new VM has now been defined and started. To confirm, execute:

root@kvm:~# virsh list --all

Id Name State

\----------------------------------------------------

10 kvm1 running

We can see the virtual machine definition file that was automatically generated by running the following code :

root@kvm:~# cat /etc/libvirt/qemu/kvm1.xml

\-------------------------------------------------------------------------------------------------------------------------------------------

Starting, stopping, and removing KVM instances:

If you define a new instance from an XML file, by default the instance will not start automatically. In this recipe, we will see how to start an instance that was previously configured :

List all instances in all states:

root@kvm:~# virsh list --all

Id Name State

\----------------------------------------------------

- kvm1 shut off

Start the newly defined instance and verify its status :

root@kvm:~# virsh start kvm1

Domain kvm1 started

root@kvm:~#

root@kvm:~# virsh list --all

Id Name State

\----------------------------------------------------

1. kvm1 running

Examine the running process for the virtual machine:

root@kvm:~# pgrep -lfa qemu

1686 /usr/bin/qemu-system-x86\_64 -name kvm1 -S -machine pc-i440fx

Terminate the VM and ensure its status changed from running to shut off:

root@kvm:~# virsh destroy kvm1

Domain kvm1 destroyed

root@kvm:~# virsh list --all

Id Name State

\----------------------------------------------------

- kvm1 shut off

Remove the instance definition:

root@kvm:~# virsh undefine kvm1

Domain kvm1 has been undefined

root@kvm:~# virsh list --all

Id Name State

\----------------------------------------------------------------------------------------------------------------------------------------------------

Inspecting and editing KVM configs :

root@kvm:~# virsh list

Id Name State

\----------------------------------------------------

11 kvm1 running

Dump the instance configuration file to standard output (stdout).

root@kvm:~# virsh dumpxml kvm1

Save the configuration to a new file, as follows:

root@kvm:~# virsh dumpxml kvm1 > kvm1.xml

root@kvm:~# head kvm1.xml

Edit the configuration in place and change the available memory for

the VM:

root@kvm:~# virsh edit kvm1

Domain kvm1 XML configuration edited.

Libvirt provides two main ways to manipulate the configuration

definitions of the virtual instances. We can either dump the config from an

existing instance, as we did in steps 2 and 3, or edit the XML definition in

place, as we did in step 4.

Also keep in mind that, if you would like to create a new instance from the

dump of an existing one, you will need to change the <name> and delete the

<uuid> attributes, as the latter will be autogenerated once the new instance

has been defined.

\---------------------------------------------------------------------------------------------------------------------------------------------------

Building new KVM instances with virtinstall

and using the console

Install a new KVM virtual machine using the official Debian

repository:

root@kvm:~# virt-install --name kvm1 --ram 1024 --extra-args='

Attach to the console to complete the installation by running the

following code:

root@kvm:~# virsh console kvm1

root@kvm:~# virsh start kvm1

enable the serial console access by running the following

command:

root@debian:~# systemctl enable serial-getty@ttyS0.service

root@debian:~# systemctl start serial-getty@ttyS0.service

Close the VNC session and connect to the virtual instance from the

host OS, using virsh:

root@kvm:~# virsh console kvm1

Examine the image file created after the installation

root@kvm:~# qemu-img info /tmp/kvm1.img

If you are not using systemd-based init system on the distribution for the KVM machine, in order to allow access to the serial console of the instance, you will need to edit

the /etc/securetty or the /etc/inittab files

\-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Managing CPU and memory resources in KVM :

Get memory statistics for the running instance:

root@kvm:~# virsh dommemstat kvm1

actual 1048576

swap\_in 0

rss 333644

Update the available memory for the VM to 2 GB:

root@kvm:~# virsh setmem kvm1 --size 1049000

Stop the running instance

root@kvm:~# virsh destroy kvm1

Domain kvm1 destroyed

Set the maximum usable memory to 2 GB:

root@kvm:~# virsh setmaxmem kvm1 --size 2097152

Start the instance:

root@kvm:~# virsh start kvm1

Domain kvm1 started

Check the current allocated memory

root@kvm:~# virsh dommemstat kvm1

actual 2097152

swap\_in 0

rss 214408

Connect to the KVM instance and check the memory in the guest OS:

Connected to domain kvm1

Escape character is ^]

Debian GNU/Linux 8 debian ttyS0

debian login: root

Password:

...

root@debian:~# free -m

total used free shared buffers cached

Mem: 2010 93 1917 5 8 40

-/+ buffers/cache: 43 1966

Swap: 382 0 3 82

root@debian:~#

root@kvm:~#

Check the memory settings in the instance XML definition:

root@kvm:~# virsh dumpxml kvm1 | grep memory

<memory unit='KiB'>2097152</memory>

root@kvm:~#

Get information about the guest CPUs:

root@kvm:~# virsh vcpuinfo kvm1

VCPU: 0

CPU: 29

State: running

CPU time: 9.7s

CPU Affinity: yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy

List the number of virtual CPUs used by the guest OS:

root@kvm:~# virsh vcpucount kvm1

maximum config 1

maximum live 1

current config 1

current live 1

Change the number of allocated CPUs to 4 for the VM:

root@kvm:~# virsh edit kvm1

...

<vcpu placement='static'>4</vcpu>

...

Domain kvm1 XML configuration edited.

root@kvm:~#

Ensure that the CPU count update took effect:

maximum config 4

maximum live 4

current config 4

current live 4

root@kvm:~# virsh dumpxml kvm1 | grep -i cpu

<vcpu placement='static'>4</vcpu>

root@kvm:~#

page 253---------------------------------------------------------------------------------------------------------------------------------------------------

Attaching block devices to virtual machines :

1. Create a new 1 GB image file:

#dd if=/dev/zero of=/tmp/new\_disk.img bs=1M count=1024

1. Attach the file as a new disk to the KVM instance:

kvm:~# virsh attach-disk kvm1 /tmp/new\_disk.img vda --live

1. Connect to the KVM instance via the console:

root@kvm:~# virsh console kvm1

1. Print the kernel ring buffer and check for the new block device:

root@debian:~# dmesg | grep vda

1. Examine the new block device:

root@debian:~# fdisk -l /dev/vda

1. Dump the instance configuration from the host OS:

root@kvm:~# virsh dumpxml kvm1

1. Get information about the new disk:

root@kvm:~# virsh domblkstat kvm1 vda

1. Detach the disk:

root@kvm:~# virsh detach-disk kvm1 vda --live

1. Copy or create a new raw image:

root@kvm:~# cp /tmp/new\_disk.img /tmp/other\_disk.img

1. Write the following config file:

root@kvm:~# cat other\_disk.xml

<disk type='file' device='disk'>

<driver name='qemu' type='raw' cache='none'/>

<source file='/tmp/other\_disk.img'/>

<target dev='vdb'/>

</disk>

1. Attach the new device:

root@kvm:~# virsh attach-device kvm1 --live other\_disk.xml

1. Detach the block device:

root@kvm:~# virsh detach-device kvm1 other\_disk.xml --live

page265---------------------------------------------------------------------------------------------------------------

Sharing directories between a running VM and the host OS :

We can only perform this action on a stopped instance however

prerequisites :

Stopped libvirt KVM instance with console access

A guest OS with the 9p and virtio kernel modules (available on most Linux distributions by default)

Create a new directory on the host OS and add a file to it:

root@kvm:~# mkdir /tmp/shared

root@kvm:~# touch /tmp/shared/file

Add the following definition to the stopped KVM instance:

root@kvm:~# virsh edit kvm1

...

<devices>

...

<filesystem type='mount' accessmode='passthrough'>

<source dir='/tmp/shared'/>

<target dir='tmp\_shared'/>

</filesystem>

...

</devices>

Start the VM:

root@kvm:~# virsh start kvm1

Domain kvm1 started

Connect to the console as follows:

root@kvm:~# virsh console kvm1

Ensure that the 9p and the virtio kernel modules are loaded:

root@debian:~# lsmod | grep 9p

If this is not the case for your VM, load the modules by running:

root@debian:~# modprobe 9p virtio

Mount the shared directory to /mnt:

root@debian:~# mount -t 9p -o trans=virtio tmp\_shared /mnt

List the new mount:

root@debian:~# mount | grep tmp\_shared

Ensure that the shared file is visible in the host OS:

root@debian:~# ls -la /mnt/

\------------------

There are three access modes:

passthrough: This is the default mode, which accesses the shared

directory using the permissions of the user inside the guest OS

mapped: In this mode, the shared directory and its files are accessed

using the permissions of the QEMU user, inherited from the host

squash: This mode is similar to the passthrough mode; however, the

failures of privileged operations such as chmod are ignored

\------------------------

Autostarting KVM instances:

Once a KVM instance has been defined and started, it will run until the

host OS is up. Once the host OS restarts, instances build with libvirt will

not automatically start once the host is up and the libvirt daemon is

running. In this recipe, we are going to change this behavior and ensure

virtual instance start when the libvirt daemon starts.

Enable the VM autostart:

root@kvm:~# virsh autostart kvm1

Obtain information for the instance:

root@kvm:~# virsh dominfo kvm1

Stop the running instance and ensure that it is in the shut off state:

root@kvm:~# virsh destroy kvm1

root@kvm:~# virsh list --all

Stop the libvirt daemon and ensure that it is not running:

root@kvm:~# /etc/init.d/libvirt-bin stop

root@kvm:~# pgrep -lfa libvirtd

Start back the libvirt daemon:

root@kvm:~# /etc/init.d/libvirt-bin start

List all running instances:

root@kvm:~# virsh list --all

Disable the autostart option:

root@kvm:~# virsh autostart kvm1 --disable

Verify the change:

root@kvm:~# virsh dominfo kvm1 | grep -i autostart

\------------------------------------------------------------------------------------------------

Working with storage pools:

kind of storage pool:

Directory backend

Local filesystem backend

Network filesystem backend

Logical backend

Disk backend

iSCSI backend

SCSI backend

Multipath backend

RADOS block device backend

Sheepdog backend

Gluster backend

ZFS backend

Virtuozzo storage backend

Copy the raw Debian image file we created in the Building new KVM

instances with virt-install and using the console recipe earlier in this

chapter:

root@kvm:~# cp /tmp/kvm1.img /var/lib/libvirt/images/

Create the following storage pool definition:

root@kvm:~# cat file\_storage\_pool.xml

<pool type="dir">

<name>file\_virtimages</name>

<target>

<path>/var/lib/libvirt/images</path>

</target>

</pool>

Define the new storage pool:

root@kvm:~# virsh pool-define file\_storage\_pool.xml

List all storage pools:

root@kvm:~# virsh pool-list --all

Start the new storage pool and ensure that it's active:

root@kvm:~# virsh pool-start file\_virtimages

root@kvm:~# virsh pool-list --all

Enable the autostart feature on the storage pool:(By default, the autostart option is not enabled on a new storage pool.)

root@kvm:~# virsh pool-autostart file\_virtimages

root@kvm:~# virsh pool-list --all

Obtain more information about the storage pool:

root@kvm:~# virsh pool-info file\_virtimages

List all volumes that are a part of the storage pool:

root@kvm:~# virsh vol-list file\_virtimages

Obtain information on the volume:

root@kvm:~# virsh vol-info /var/lib/libvirt/images/kvm1.img

Start new KVM instance using the storage pool and volume, then

ensure that it's running

root@kvm:~# virt-install --name kvm1 --ram 1024 --graphics vnc,

root@kvm:~# virt-install --name kvm1 --ram 1024 --graphics vnc,listen=Starting install...

root@kvm:~# virsh list --all

---------------------------------------Managing volumes-------------------------------------------------------

List the available storage pools:

root@kvm:~# virsh pool-list --all

List the available volumes, that are a part of the storage pool:

root@kvm:~# virsh vol-list file\_virtimages

Create a new volume with the specified size:

root@kvm:~# virsh vol-create-as file\_virtimages new\_volume.img 9G

List the volumes on the filesystem:

root@kvm:~# ls -lah /var/lib/libvirt/images/

Obtain information about the new volume:

root@kvm:~# qemu-img info /var/lib/libvirt/images/new\_volume.img

Use the virsh command to get even more information:

root@kvm:~# virsh vol-info new\_volume.img --pool file\_virtimages

Dump the volume configuration:

root@kvm:~# virsh vol-dumpxml new\_volume.img --pool file\_virtimages

\-------

<volume type='file'>

<name>new\_volume.img</name>

<key>/var/lib/libvirt/images/new\_volume.img</key>

<source>

</source>

<capacity unit='bytes'>9663676416</capacity>

<allocation unit='bytes'>9663680512</allocation>

<target>

<path>/var/lib/libvirt/images/new\_volume.img</path>

<format type='raw'/>

<permissions>

<mode>0600</mode>

<owner>0</owner>

<group>0</group>

</permissions>

<timestamps>

<atime>1490301514.446004048</atime>

<mtime>1490301483.698003615</mtime>

<ctime>1490301483.702003615</ctime>

</timestamps>

</target>

</volume>

\-------------

Resize the volume and display the new size:

root@kvm:~# virsh vol-resize new\_volume.img 10G --pool file\_virtimages

root@kvm:~# virsh vol-info new\_volume.img --pool file\_virtimages

Delete the volume and list all available volumes in the storage pool:

root@kvm:~# virsh vol-delete new\_volume.img --pool file\_virtimages

root@kvm:~# virsh vol-list file\_virtimages

Clone the existing volume:

root@kvm:~# virsh vol-clone kvm1.img kvm2.img --pool file\_virtimages

root@kvm:~# virsh vol-list file\_virtimages

Sparse images

don't allocate all of the disk space and grow as more data is being written

to it.

----------------------------------------Managing secrets-----------------------------------------------

REQUIREMENT:

A storage pool with an iSCSI-backed volume

The libvirt package

List all available secrets:

root@kvm:~# virsh secret-list

Create the following secrets definition:

root@kvm:~# cat volume\_secret.xml

\---------

<secret ephemeral='no'>

<description>Passphrase for the iSCSI iscsi-target.linux-admins.net</description>

<usage type='iscsi'>

<target>iscsi\_secret</target>

</usage>

</secret>

Create the secret and ensure that it has been successfully created:

root@kvm:~# virsh secret-define volume\_secret.xml

root@kvm:~# virsh secret-list

Set a value for the secret:

root@kvm:~# virsh secret-set-value 7ad1c208-c2c5-4723-8dc5-e2f4f576101a

Create a new iSCSI pool definition file:

root@kvm:~# cat iscsi.xml

\-----

<pool type='iscsi'>

<name>iscsi\_virtimages</name>

<source>

<host name='iscsi-target.linux-admins.net'/>

<device path='iqn.2004-04.ubuntu:ubuntu16:iscsi.libvirtkvm'/>

<auth type='chap' username='iscsi\_user'>

<secret usage='iscsi\_secret'/>

</auth>

</source>

<target>

<path>/dev/disk/by-path</path>

</target>

</pool>

\---------

The <secret> root element, with an optional ephemeral attribute, telling

libvirt that the password should only be stored in memory, if set to

yes.

---------------------------------------KVM Networking with libvirt------------------------------------------

The Linux bridge is a software layer 2 device that provides some of the functionality of a physical bridge device. It can forward frames between

KVM guests, the host OS, and virtual machines running on other servers, or networks. The Linux bridge consists of two components--a userspace

administration tool that we are going to use in this recipe and a kernel module that performs all the work of connecting multiple Ethernet

segments together. Each software bridge we create can have a number of ports attached to it, where network traffic is forwarded to and from. When

creating KVM instances, we can attach the virtual interfaces that are associated with them to the bridge, which is similar to plugging a network

cable from a physical server's NIC to a bridge/switch device. Being a layer 2 device, the Linux bridge works with MAC addresses and maintains a

kernel structure to keep track of ports and associated MAC addresses in the form of a Content Addressable Memory (CAM) table.

To check whether your kernel is compiled with those features or

exposed as kernel modules, run the following command:

root@kvm:~# cat /boot/config-`uname -r` | grep -i bridg

To verify that the module is loaded and to obtain more information

about its version and features, execute the following command:

root@kvm:~# lsmod | grep bridge

The bridge-utils package that provides the tool to create and

manipulate the Linux bridge

Install the Linux bridge package, if it is not already present:

root@kvm:~# apt install bridge-utils

Build a new KVM instance

root@kvm:~# virt-install --name kvm1 --ram 1024 --disk path=/tmp

List all the available bridge devices:

root@kvm:~# brctl show

Bring the virtual bridge down, delete it, and ensure that it's been

deleted:

root@kvm:~# ifconfig virbr0 down

root@kvm:~# brctl delbr virbr0

root@kvm:~# brctl show

Create a new bridge and bring it up:

root@kvm:~# brctl addbr virbr0

root@kvm:~# ifconfig virbr0 up

Assign an IP address to bridge:

root@kvm:~# ip addr add 192.168.122.1 dev virbr0

root@kvm:~# ip addr show virbr0

List the virtual interfaces on the host OS:

root@kvm:~# ip a s | grep vnet

Add the virtual interface vnet0 to the bridge:

root@kvm:~# brctl addif virbr0 vnet0

root@kvm:~# brctl show virbr0

Enable the Spanning Tree Protocol (STP) on bridge and obtain more information:

root@kvm:~# brctl stp virbr0 on

root@kvm:~# brctl showstp virbr0

From inside the KVM instance, bring the interface up, request an IP address, and test connectivity to the host OS:

root@debian:~# ifconfig eth0 up

root@debian:~# dhclient eth0

root@debian:~# ip a s eth0

root@debian:~# ping 192.168.122.1 -c 3

\---------------

When we first installed and started the libvirt daemon, a few things

happened automatically:

A new Linux bridge was created with the name and IP address defined

in the /etc/libvirt/qemu/networks/default.xml configuration file

The dnsmasq service was started with a configuration specified in

the /var/lib/libvirt/dnsmasq/default.conf file

\----------------------

Let's examine the default libvirt bridge configuration:

root@kvm:~# cat /etc/libvirt/qemu/networks/default.xml

<network>

<name>default</name>

<bridge name="virbr0"/>

<forward/>

<ip address="192.168.122.1" netmask="255.255.255.0">

<dhcp>

<range start="192.168.122.2" end="192.168.122.254"/>

</dhcp>

</ip>

</network>


We can see that a DHCP server is running on the host OS and its

configuration file by running the following command:

root@kvm:~# pgrep -lfa dnsmasq

root@kvm:~# cat /var/lib/libvirt/dnsmasq/default.conf

To examine the table of MAC addresses the

bridge knows about, run the following command:

root@kvm:~# brctl showmacs virbr0

root@kvm:~# virsh console kvm1

root@debian:~# ip a s eth0

root@kvm:~# ifconfig | grep "fe:54:00:55:9b:d6"

We can set the time limit in

seconds before the bridge will expire the MAC address entry by executing

the following command:

root@kvm:~# brctl setageing virbr0 600

to use the latest version, or if a package is not

available for your distribution, we can build the utility from source by

cloning the project with git, then configure and compile

root@kvm:~# cd /usr/src/

root@kvm:/usr/src# apt-get update && apt-get install build-essential automake root@kvm:/usr/src# git clone git://git.kernel.org/pub/scm/linux/kernel

root@kvm:/usr/src/bridge-utils# autoconf

root@kvm:/usr/src/bridge-utils# ./configure && make && make install

root@kvm:/usr/src/bridge-utils# brctl --version

On a RedHat/CentOS host, the process is similar:

[root@centos ~]# cd /usr/src/

[root@centos src]# yum groupinstall "Development tools"

[root@centos src]# git clone git://git.kernel.org/pub/scm/linux/kernel

[root@centos bridge-utils]# autoconf

[root@centos bridge-utils]# ./configure && make && make install

[root@centos bridge-utils]# brctl --version

344 openvswitch----------------------------------------------------------------------------------

To create a new OVS bridge and attach the virtual interface of a KVM guest, follow these steps:

Remove the existing Linux bridge, if any:

root@kvm:~# brctl show

root@kvm:~# ifconfig virbr0 down

root@kvm:~# brctl delbr virbr0

root@kvm:~# brctl show

On some Linux distributions, it helps to unload the kernel module for the Linux bridge before using OVS. To do this,

execute

root@kvm:/usr/src# modprobe -r bridge

Install the OVS package on Ubuntu:

root@kvm:~# apt-get install openvswitch-switch

Ensure that the OVS processes are running:

root@kvm:~# pgrep -lfa switch

Ensure that the OVS kernel module has been loaded:

root@kvm:~# lsmod | grep switch

List the available OVS switches:

root@kvm:~# ovs-vsctl show

Create a new OVS switch:

root@kvm:~# ovs-vsctl add-br virbr1

root@kvm:~# ovs-vsctl show

Add the interface of the running KVM instance to the OVS switch:

root@kvm:~# ovs-vsctl add-port virbr1 vnet0

root@kvm:~# ovs-vsctl show

Configure an IP address on the OVS switch:

root@kvm:~# ip addr add 192.168.122.1/24 dev virbr1

root@kvm:~# ip addr show virbr1

Configure an IP address inside the KVM guest and ensure connectivity to the host OS (if the image does not have console access configure,

connect to it using VNC):

root@kvm:~# virsh console kvm1

root@debian:~# ifconfig eth0 up && ip addr add 192.168.122.210/24 dev

root@debian:~# ip addr show eth0

The ovsdb-server process that was also started after installing the package,

as seen from the output in step 3, is a database engine that uses

JSON Remote Procedure Calls (RPC) to communicate with the main

OVS daemon. The ovsdb server process stores information, such as the

switch network flows, ports, and QoS to name just few. You can query the

database by running the following command:

root@kvm:~# ovsdb-client list-dbs

root@kvm:~# ovsdb-client list-tables

root@kvm:~# ovsdb-client dump Open\_vSwitch

To remove the KVM virtual interface from the OVS switch, execute the

following command:

root@kvm:~# ovs-vsctl del-port virbr1 vnet0

To completely delete the OVS switch, run the following command:

root@kvm:~# ovs-vsctl del-br virbr1 && ovs-vsctl show

------------------------Configuring NAT forwarding network----------------------------------------------------

To configure a new NAT network and connect a KVM instance to it, run the following :

List all available networks:

root@kvm:~# virsh net-list --all

Dump the configuration of the default network:

root@kvm:~# virsh net-dumpxml default

Compare that with the XML definition file for the default network:

root@kvm:~# cat /etc/libvirt/qemu/networks/default.xml

List all running instances on the host:

root@kvm:~# virsh list --all

Ensure that the KVM instances are connected to the default Linux bridge:

Create a new NAT network definition:

root@kvm:~# cat nat\_net.xml

\----------

<network>

<name>nat\_net</name>

<bridge name="virbr1"/>

<forward/>

<ip address="10.10.10.1" netmask="255.255.255.0">

<dhcp>

<range start="10.10.10.2" end="10.10.10.254"/>

</dhcp>

</ip>

</network>

\---------------

Define the new network:

root@kvm:~# virsh net-define nat\_net.xml

root@kvm:~# virsh net-list --all

Start the new network and enable autostarting:

root@kvm:~# virsh net-start nat\_net

root@kvm:~# virsh net-autostart nat\_net

root@kvm:~# virsh net-list

Obtain more information about the new network:

root@kvm:~# virsh net-info nat\_net

Edit the XML definition of the kvm1 instance and change the name of the source network:

root@kvm:~# virsh edit kvm1

\---------------

...

<interface type='network'>

...

<source network='nat\_net'/>

...

</interface>

...

\--------------

Restart the KVM guest:

root@kvm:~# virsh destroy kvm1

root@kvm:~# virsh start kvm1

List all software bridges on the host:

root@kvm:~# brctl show

Connect to the KVM instances and check the IP address of the eth0 interface and ensure connectivity to the host bridge (if the image is not

configured for console access, use a VNC client instead):

root@kvm:~# virsh console kvm1

root@debian:~# ip a s eth0 | grep inet

On the host OS, examine which DHCP services are running:

root@kvm:~# pgrep -lfa dnsmasq

Check the IP of the new bridge interface:

root@kvm:~# ip a s virbr1

List the iptables rules for the NAT table:

root@kvm:~# iptables -L -n -t nat

--------------------------------------------------------Configuring bridged network-----------------------------------------------------

A server with at least two physical interfaces

The ability to provision and start KVM instances with libvirt

A running KVM instance

\-----------

To define a new bridged network and attach a guest to it, follow the steps:

1. Take down the interface we are going to bridge:

root@kvm:~# ifdown eth1

Edit the network configuration file on the host and replace the eth1

block with the following, if your host OS is Debian/Ubuntu:

root@kvm:~# vim /etc/network/interfaces

...

auto virbr2

iface virbr2 inet static

`	`address 192.168.1.2

`	`netmask 255.255.255.0

`	`network 192.168.1.0

`	`broadcast 192.168.1.255

`	`gateway 192.168.1.1

`	`bridge\_ports eth1

`	`bridge\_stp on

`	`bridge\_maxwait 0

...

root@kvm:~#

If using RedHat/CentOS distributions, edit the following two files

instead:

root@kvm:~# cat /etc/sysconfig/ifcfg-eth1

DEVICE=eth1

NAME=eth1

NM\_CONTROLLED=yes

ONBOOT=yes

TYPE=Ethernet

BRIDGE=virbr2

root@kvm:~# cat /etc/sysconfig/ifcfg-bridge\_net

DEVICE=virbr2

NAME=virbr2

NM\_CONTROLLED=yes

ONBOOT=yes

TYPE=Bridge

STP=on

IPADDR=192.168.1.2

NETMASK=255.255.255.0

GATEWAY=192.168.1.1

Start the new interface up:

root@kvm:~# ifup virbr2

Disable sending packets to iptables that originate from the guest VMs:

root@kvm:~# sysctl -w net.bridge.bridge-nf-call-iptables=0

net.bridge.bridge-nf-call-iptables = 0

root@kvm:~# sysctl -w net.bridge.bridge-nf-call-iptables=0

net.bridge.bridge-nf-call-iptables = 0

root@kvm:~# sysctl -w net.bridge.bridge-nf-call-arptables=0

net.bridge.bridge-nf-call-arptables = 0

root@kvm:~#

List all bridges on the host:

root@kvm:~# # brctl show

Edit the XML definition for the KVM instance:

root@kvm:~# virsh edit kvm1

...

<interface type='bridge'>

<source bridge='virbr2'/>

</interface>

...

Domain kvm1 XML configuration edited.

Restart the KVM instance:

root@kvm:~# virsh destroy kvm1

root@kvm:~# virsh start kvm1

------------------------------------------------------------Configuring PCI passthrough network--------------------------

The KVM hypervisor supports directly attaching PCI devices from the

host OS to the virtual machines. We can use this feature to attach a

network interface directly to the guest OS, without the need for using NAT

or software bridges.

In order to complete this recipe, we are going to need the following:

A physical host with NIC that supports SR-IOV

A 802.1Qbh capable switch with connection to the physical server

CPU with either the Intel VT-d or AMD IOMMU extensions

Linux host with libvirt installed, ready-to-provision KVM instances

To set up a new PCI passthrough network follow the steps:

1. Enumerate all devices on the host OS:

root@kvm:~# virsh nodedev-list --tree

List all PCI Ethernet adapters:

root@kvm:~# lspci | grep Ethernet

Obtain more information about NIC that the eth1 device is using:

root@kvm:~# virsh nodedev-dumpxml pci\_0000\_03\_00\_1

Convert the domain, bus, slot, and function values to hexadecimal:

root@kvm:~# printf %x 0

0r

oot@kvm:~# printf %x 3

3r

oot@kvm:~# printf %x 0

0r

oot@kvm:~# printf %x 1

1r

root@kvm:~#

Create a new libvirt network definition file:

root@kvm:~# cat passthrough\_net.xml

<network>

<name>passthrough\_net</name>

<forward mode='hostdev' managed='yes'>

<pf dev='eth1'/>

</forward>

</network>

root@kvm:~#

Define, start, and enable autostarting on the new libvirt network:

root@kvm:~# virsh net-define passthrough\_net.xml

Network passthrough\_net defined from passthrough\_net.xml

root@kvm:~# virsh net-start passthrough\_net

Network passthrough\_nett started

root@kvm:~# virsh net-autostart passthrough\_net

Network passthrough\_net marked as autostarted


root@kvm:~# virsh net-list

Name State Autostart Persistent

\----------------------------------------------------------

default active yes yes

passthrough\_net active yes yes


Edit the XML definition for the KVM guest:

root@kvm:~# virsh edit kvm1

...

<devices>

...

<interface type='hostdev' managed='yes'>

<source>

<address type='pci' domain='0x0' bus='0x00' slot='0x07' function='</source>

<virtualport type='802.1Qbh' />

</interface>

<interface type='network'>

<source network='passthrough\_net'>

</interface>

...

</devices>

...

Domain kvm1 XML configuration edited.



Restart the KVM instance:

root@kvm:~# virsh destroy kvm1

root@kvm:~# virsh start kvm1

List the Virtual Functions (VFs) provided by SR-IOV NIC:

root@kvm:~# virsh net-dumpxml passthrough\_net

<network connections='1'>

<name>passthrough\_net</name>

<uuid>a4233231-d353-a112-3422-3451ac78623a</uuid>

<forward mode='hostdev' managed='yes'>

<pf dev='eth1'/>

<address type='pci' domain='0x0000' bus='0x02' slot='0x10' function='<address type='pci' domain='0x0000' bus='0x02' slot='0x10' function='<address type='pci' domain='0x0000' bus='0x02' slot='0x10' function='<address type='pci' domain='0x0000' bus='0x02' slot='0x10' function='<address type='pci' domain='0x0000' bus='0x02' slot='0x11' function='<address type='pci' domain='0x0000' bus='0x02' slot='0x11' function='<address type='pci' domain='0x0000' bus='0x02' slot='0x11' function='</forward>

</network>


Using the PCI ID from step 1, we proceed to collect more information

about NIC in step 3. Note that 0000\_03\_00\_1 ID is broken down

into domain ID, bus ID, slot ID, and function ID, as shown by the XML

attributes.



--------------------------------------------Manipulating network interfaces-------------------------------------------------------------------------------

To create a new bridge interface using libvirt, run the following commands:

1. Create a new bridge interface configuration file:

root@kvm:~# cat test\_bridge.xml

root@kvm:~# cat test\_bridge.xml

<interface type='bridge' name='test\_bridge'>

<start mode="onboot"/>

<protocol family='ipv4'>

<ip address='192.168.1.100' prefix='24'/>

</protocol>

<bridge>

<interface type='ethernet' name='vnet0'>

<mac address='fe:54:00:55:9b:d6'/>

</interface>

</bridge>

</interface>


Define the new interface:

root@kvm:~# virsh iface-define test\_bridge.xml

List all interfaces libvirt knows about:

root@kvm:~# virsh iface-list --all

Start the new bridge interface:

root@kvm:~# virsh iface-start test\_bridge

root@kvm:~# virsh iface-list --all | grep test\_bridge

List all bridge devices on the host:

root@kvm:~# brctl show

Check the active network configuration of the new bridge:

root@kvm:~# ip a s test\_bridge

Obtain the MAC address of bridge:

root@kvm:~# virsh iface-mac test\_bridge

Obtain the name of the bridge based by providing its MAC address:

root@kvm:~# virsh iface-name 4a:1e:48:e1:e7:de

Destroy the interface, as follows:

root@kvm:~# virsh iface-destroy test\_bridge

root@kvm:~# virsh iface-list --all | grep test\_bridge

root@kvm:~# virsh iface-undefine test\_bridge

root@kvm:~# virsh iface-list --all | grep test\_bridge


405-------------------------------------------Migrating KVM Instances--------------------------------------------------------

Offline migration involves downtime for the instance. It works by first suspending the guest VM, then copying an image of the

guest memory to the destination hypervisor. The KVM machine is then resumed on the target host. If the filesystem of the VM is not on a

shared storage, then it needs to be moved to the target server as well. Live migration works by moving the instance in its current state with

no perceived downtime, preserving the memory and CPU register states.

offline mode procdure:

- Stopping the instance
- Dumping its XML definition to a file
- Copying the guest filesystem image to the destination server (if not using shared storage)
- Defining the instance on the destination host and starting it

online migration requires shared storage. such as NFS or GlusterFS, removing the need to transfer the guest filesystem to the target

server. The speed of the migration depends on how often the memory of the source instance is being updated/written to, the size of the memory, and

the available network bandwidth between the source and target hosts.

Live migration follows this process:

- The original VM continues to run while the content of its memory is being transferred to the target host
- Libvirt monitors for any changes in the already transferred memory pages, and if they have been updated, it retransmits them
- Once the memory content has been transferred to the destination host, the original instance is suspended and the new instance on the target host is resumed


requirement:

Two servers with libvirt and qemu installed and configured, named kvm1 and kvm2. The two hosts must be able to connect to each other

using SSH keys and short hostname. A server with an available block device that will be exported as an

iSCSI target and reachable from both libvirt servers. If a block device is not available, please refer to the There's more... section

in this recipe for instructions on how to create one using a regular file. The name of the iSCSI target server in this recipe is iscsi\_target.

Connectivity to a Linux repository to install the guest OS.


1-On the iSCSI target host, install the iscsitarget package and kernel module package:

root@iscsi\_target:~# apt-get update && apt-get install iscsitarget

2-Enable the target functionality:

root@iscsi\_target:~# sed -i 's/ISCSITARGET\_ENABLE=false/ISCSITARGET\_

root@iscsi\_target:~# cat /etc/default/iscsitarget

ISCSITARGET\_ENABLE=true

ISCSITARGET\_MAX\_SLEEP=3

ISCSITARGET\_MAX\_SLEEP=3

\# ietd options

\# See ietd(8) for details

ISCSITARGET\_OPTIONS=""

3-Configure the block device to export with iSCSI:

root@iscsi\_target:~# cat /etc/iet/ietd.conf

Target iqn.2001-04.com.example:kvm

Lun 0 Path=/dev/loop1,Type=fileio

Alias kvm\_lun

Replace the /dev/loop1 device with the block device you are exporting with iSCSI.

Restart the iSCSI target service:

root@iscsi\_target:~# /etc/init.d/iscsitarget restart

5-On both libvirt hosts, install the iSCSI initiator:

root@kvm1/2:~# apt-get update && apt-get install open-iscsi

6-On both libvirt servers, enable the iSCSI initiator service and start it:

root@kvm1/2:~# sed -i 's/node.startup = manual/node.startup = automatic/

root@kvm1/2:~# /etc/init.d/open-iscsi restart






























































































































