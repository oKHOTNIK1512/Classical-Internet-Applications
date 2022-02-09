---
tags: Classical Internet Applications
---
:::success
# CIA Lab 1 Assignment: Booting 1
**Name: Ivan Okhotnikov**
:::


## Task 1: PXE Installation
:::info
**PXE Server Setup**

Create the first virtual machine using VirtualBox and isolate a private network on your
workstation: do not pollute our shared network with your own DHCP service. There are
several network settings offered by VirtualBox, choose the right one accordingly and
Install your PXE server there.
Question: why not run your DHCP service on the SNE network directly?
Hint: you need to set up DHCP + TFTP and some boot loader e.g. PXELINUX.
Your PXE server should serve the operating system of your choosing.

---

**PXE Client Setup**

Create the second virtual machine using VirtualBox in order to test the PXE service -
boot and install a new system with it and show the proof in the report.
:::


## Implementation:
### PXE Server Setup
Before getting started, I installed virtualbox version 6.1 and created a virtual machine called "server" and "client".
Since virtualbox asked to disable secure boot when starting the virtual machine, I had to turn it off in the BIOS



<center>

![](https://i.imgur.com/si1enVS.png)

Figure 1: Virtualbox VMs

</center>

On the virtual machine "server", I installed apache2, tftp and DHCP server with the command

```
sudo apt-get install apache2 tftpd-hpa inetutils-inetd isc-dhcp-server
```
After installing the necessary components, you must first configure the DHCP server
To do this, we need to specify the interface name in the /etc/default/isc-dhcp-server file
```
sudo nano /etc/default/isc-dhcp-server
```
<center>

![](https://i.imgur.com/zzGdTwo.png)
Figure 2: Configuring the ```/etc/default/isc-dhcp-server``` file
</center>

To complete the configuration of the DHCP file, I edited the /etc/dhcp/dhcpd.conf file as shown in the pictures
```
sudo nano /etc/dhcp/dhcpd.conf
```
<center>

![](https://i.imgur.com/gTI5Zje.png)
Figure 3: We specify the domain name and enable authorization

![](https://i.imgur.com/D6zLw9J.png)
Figure 4: We specify the ip range of the issued addresses/mask/host

![](https://i.imgur.com/N6fFA2V.png)
Figure 5: We allow the boot via dhcp and specify the file
</center>


I also need to manually register the ip address on the computer

<center>

![](https://i.imgur.com/u6gQ638.png)
Figure 6: Сonfiguring the network 
</center>

To start configuring ```TFTP```, I will need to create a directory ```/var/lib/tftpboot```

```
sudo mkdir /var/lib/tftpboot
```

Next, I need to configure the ```/etc/default/tftpd-hpa``` file as shown in the screenshot

<center>

![](https://i.imgur.com/7X7m28V.png)
Figure 7: In it, we configure the startup of the daemon and its parameters
</center>

I have edited the configuration file for the ```inetd``` service.
This is a service that allows I to run daemons at system startup and allows I to configure in the configuration file to listen to requests from connected clients

<center>

![](https://i.imgur.com/VtHpTpe.png)
Figure 8: My ```inetd``` configuration file
</center>

In order for our PXE server to work as expected, it is necessary to provide it with information about the system being loaded
The first thing I need to do is download the necessary image (I have ubuntu-16.04.7-server) and mount it in the /mnt/ folder for quick access to internal files

```
wget https://releases.ubuntu.com/16.04.7/ubuntu-16.04.7-server-amd64.iso
sudo mount -o loop ubuntu-16.04.7-server-amd64.iso /mnt
```

Copy the installation files of the operating system image to ```/var/lib/tftpboot```

```
cd /mnt
sudo cp -fr install/netboot/* /var/lib/tftpboot/
```

Next, we will copy the image files to the internal folder ```apache2```
```
sudo mkdir /var/www/html/linux
sudo cp -fr /mnt/* /var/www/html/linux/
```

```
sudo nano /var/lib/tftpboot/pxelinux.cfg/default
```

Now we just need to fix the installation configuration files in ```/var/lib/tftpboot```
<center>

![](https://i.imgur.com/p9O619P.png)
Figure 9: My configuration
</center>

Now the work on configuring the ```PXE server``` is completed and I can restart the DHCP server and ```TFTP```

```
sudo service tftpd-hpa restart
sudo service isc-dhcp-server restart
```

<center>

![](https://i.imgur.com/IzO6KZU.png)
Figure 10: The status of the ```DHCP``` server

![](https://i.imgur.com/rpSNOyy.png)
Figure 11: The status of the ```TFTP```

</center>

In order to avoid clogging the university network with my ```DHCP``` server, it is necessary to reconfigure the network adapter in ```Virtualbox```
<center>

![](https://i.imgur.com/gSNs5Vg.png)
Figure 12: Configuring a network adapter for a ```PXE server```
</center>


### PXE Client Setup

In order for the ```PXE client``` to boot from the network, you need to allow it to run over the network and configure the network adapter

<center>

![](https://i.imgur.com/mShc5Va.png)
Figure 13: Enabling permission to boot the operating system using the network

![](https://i.imgur.com/aN4XZLn.png)
Figure 14: Configuring a network adapter for a ```PXE client```

</center>

After launching the client, the operating system installation window opens immediately after connecting to the DHCP server

<center>

![](https://i.imgur.com/toYe8XE.png)
Figure 15: The download page for the ```PXE client```
</center>


---

## Task 2: Questions to answer
:::spoiler 1a) What is UEFI PXE booting?
UEFI (Extensible Firmware Interface) - The firmware that is built into the motherboard. It allows you to initialize the connected devices to the computer and configure the computer startup.

PXE (Pre-Boot Execution Environment) is an environment that allows you to download and install the computer's operating system using a network connection.
For implementation, it uses IP, UDP, BOOT, DHCP and TFTP
:::

:::spoiler 1b) How does it work?

When the computer turns on (and it has a pre-boot from the network specified in the BIOS settings), it tries to get an IP address from the DHCP server.
If he succeeded, he checks whether there is a PXE server in this network.
If a PXE server is found on the network, it will request boot files from it and load the operating system
:::

:::spoiler 2a) What is a GPT?
GPT (GUID Partition Table) - Standard for the hard disk layout format. It is part of UEFI and replaced MBR
:::

:::spoiler 2b) What is its layout? Explain each element.
The GPT consists of the following logical blocks:

**LBA0** - This is an MBR block. It is used to protect against accidental corruption of GPT partitions by programs that do not know anything about GPT

**LBA1** - Table of Contents of the partition table. Contains information about all logical blocks that can be used by the user.

**LBA2-33** - Disk partition table. They have a location with an equal increment of addresses.  The first 16 bytes determine the type of partition, the next 16 bytes-information about the unique GUID for a specific partition. Next, information about the LBA is recorded (if available) and the rest of the place deals with information about attributes and section names

**LBA 34** - Disk partition data

**LBA-34** - Copy of the disk partition table

**LBA-2** - Copy of the GPT header 

**LBA-1** - Is a copy of LBA 0. It will be used in the case when LBA 0 is damaged

<center>

![](https://i.imgur.com/m0nVPqw.png)
Figure 16: Layout of GPT 
</center>
:::

:::spoiler 2c) What is the role of a partition table?
The section table contains information about where sections begin and end. This helps the system to understand where the boot disk starts from and in which sector on the disk to search for data.
:::

:::spoiler 3a) What is gdisk?
gdisk is a command-line program that allows you to view and manage the GPT partition table.
:::

:::spoiler 3b) How does it work? 
When we run the gdisk program, it starts determining the type of partition used on the disk.
If it does not find GPT, it will not find MBR and BSD, it will try to convert the MBR or disk label to the GPT form.
If it finds GPT, it will use it
:::

:::spoiler 3c) What can you do with it?
    Using the gdisk utility, you can:
- create a backup of a GPT table
- change the name of the sections
- delete partitions on the disk
- create partitions on the disk
- display detailed information about the partition on the disk
- create a new empty GPT partition table
- check the disk
:::



:::spoiler 4a) What is a Protective MBR and why is it in the GPT?

Protective MBR partition inside the GPT. It is necessary so that programs that do not know about GPT can not accidentally damage it. For these programs, the disk structure will be a single partition that takes up all the space on the disk
:::

## Task 3: Partitions
:::info
1 Copy and dump the Protective MBR and GPT in hex format
:::

Using the dd utility, we will dump the MBR and GPT
This is done as follows


:::info
MBR dump
:::

```
sudo dd if=/dev/sda of=mbr bs=512 count=1
```
<center>

![](https://i.imgur.com/xt2QPJL.png)
Figure :Contents of the MBR dump
</center>

Since I have only one record, let's consider it

```0000 0200 EEFF FFFF 0100 0000 2F60 383A```


| 00 | 000200 | EE | FFFFFF | 0100 0000 | 602F 3A38 |
|---|---|---|---|---|---|
| Status or physical drive | Address of the first absolute sector in partition(in CHS) | Partition Type | Last Sector(in CHS) | Relative Sectors(or Offset) | Partition Size |
| 80 - active/ 00 - inactive | Location of GPT | EE means that the type we have is EFI_GPT_DISK | 16777215 | 16 | 500107861504 |

---

:::info
GPT dump
:::
```
sudo dd if=/dev/sda of=gpt_table bs=1280 count=1
```
<center>

![](https://i.imgur.com/eXmbdPl.jpg)
Figure: MBR 

![](https://i.imgur.com/vhKOMoH.jpg)
Figure: GPT Header
</center>

The GPT header consists of
- EFI SIGNATURE ```4546 4920 5041 5254```
- Version 1.0 ```0000 0100```
- The header size is ```5c00 0000``` (92 bytes).
- CRC32 header for detecting errors ```d08f 010e``` (234983376)
- The location of the current LBA ```0100 0000 0000 0000``` (1)
- The location of the backup header ```2f60 383a 0000 0000``` (976773167)
- First used for the DIF block ```2200 0000 0000 0000``` (34)
- Last used for partitions LBA block ```0e60 383a 0000 0000``` (976773134)
- - Unique disk ID GUID ```9d01 3f42 0350 114d ad11 7112 3eb9 e0d4```
- The beginning of the LBA for the records of the partition array ```0200 0000 0000 0000``` (2)
- The number of records in the partition array ```8000 0000``` (128)
- The size for one partition record ```8000 0000``` (128)
- CRC32 in the array of partition records for error detection ```761a 91a3``` (2744195702)

<center>

![](https://i.imgur.com/zRPM89u.jpg)
Figure: GPT entries
</center>

### The first record of GPT partitions: 
- Partition type ID GUID ```2873 2ac1 1ff8 d211 ba4b 00a0 c93e c93b``` (EFI System partition)
- Unique section IDENTIFIER ```051a 95d4 e5d1 9f40 ab1c 9b71 68fc fc87```
- First LBA ```0008 0000 0000 0000``` (2048) 
- Last LBA ff07 ```1000 0000 0000``` (1050623)
- Attribute flag ```0000 0000 0000 0000```
- Name of the partition ```4500 4600 4900 2000 5300 7900 7300 7400 6500 6d00 2000 5000 6100 7200 7400 6900 7400 6900 6f00 6e00 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000``` (EFI system partition).

### The second entry of the GPT partitions:
- Partition type ID GUID ```af3d c60f 8384 7247 8e79 3 d69 d847 7de4``` (Linux file system data)
- The unique partition ID is ```1109 bfd2 82a2 094c 8c55 2815 a8b6 4494```
- First LBA ```0008 1000 0000 0000``` (1050624)
- Last LBA ```ff57 383a 0000 0000``` (976771071)
- No name

---

:::spoiler (a) At what byte index from the start of the disk do the real partition table entries start?
In a real partition table, entries start with the third sector, since it will be preceded by LBA0, LBA1. Since the weight of 1 sector is 512 bytes, the index of the beginning of the records will be 0x400
:::

:::spoiler (b) At what byte index would the partition Table start if your server had a so-called “4K native” (4Kn) disk?
My partition table would then start with the index 8192 and will have a size of 4 blocks.
:::


---

:::info
2 Add partition called ØS3 to the table by hand
What values would you have to use for the entry?
:::
For the Freebsd ZFS partition, the GUID will be as follows:
```ba7c 6e51 cf6e d611 8ff8 0002 2d09 712b```
Then there is a unique section identifier GUID
For example: ```0edd 1b98 ba25 9e4e a1aa 4900 811a bbf6```

If you add it to the computer, then:
The first LBA will be ```0000 ec72 0000 0000```,
Last LBA ```ffa7 5174 0000 0000```

Then there are the attribute flags ```0000 0000 0000 0000``` and named ```D800 5A01 3300``` for ØS3

---

:::spoiler 3 Differences between primary and logical partitions in an MBR partitioning scheme.
The main difference is that we can load the operating system from the primary partition, but not from the logical one.
There can be 4 primary partitions (of which 1 is extended and can contain up to 23 logical partitions)
:::



---
## References:

1: [Installing virtualbox](https://losst.ru/ustanovka-virtualbox-v-ubuntu-18-04)
2: [How to install pxe server ubuntu](https://ostechnix.com/how-to-install-pxe-server-on-ubuntu-16-04/)
3: [UNIX / Linux: Copy Master Boot Record (MBR)](https://www.cyberciti.biz/faq/howto-copy-mbr/)
4: [MBR and GPT analysis](https://habr.com/en/post/347002/)
5: [GUID TABLE](https://ru.wikipedia.org/wiki/%D0%A2%D0%B0%D0%B1%D0%BB%D0%B8%D1%86%D0%B0_%D1%80%D0%B0%D0%B7%D0%B4%D0%B5%D0%BB%D0%BE%D0%B2_GUID)
6: [MBR analysis](http://blog.hakzone.info/posts-and-articles/bios/analysing-the-master-boot-record-mbr-with-a-hex-editor-hex-workshop/)
7: [MBR Partition Table](https://thestarman.pcministry.com/asm/mbr/PartTables3.htm)
