# Boot BeagleBone Black board from tftp server

### 1-First we need to know our BBB ip address:In my case My BBB IP address is (inet addr:inet addr:192.168.1.4)
  ```
  ubuntu@arm:~$ ifconfig
eth0      Link encap:Ethernet  HWaddr 80:30:dc:5e:f8:54  
          inet addr:192.168.1.4  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::8230:dcff:fe5e:f854/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST DYNAMIC  MTU:1500  Metric:1
          RX packets:43 errors:0 dropped:0 overruns:0 frame:0
          TX packets:63 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:4051 (4.0 KB)  TX bytes:7819 (7.8 KB)
          Interrupt:45 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:472 (472.0 B)  TX bytes:472 (472.0 B)
ubuntu@arm:~$ 
```

### 2-Second we need to know our Host IP Address(x86 Intel),In my case My Host IP address is (inet addr:192.168.1.2)

```
sandeeptux@sandeeplinux:~$ ifconfig
eth0      Link encap:Ethernet  HWaddr ec:b1:d7:bb:3d:8d  
          inet addr:192.168.1.2  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::eeb1:d7ff:febb:3d8d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:831041 errors:0 dropped:0 overruns:0 frame:0
          TX packets:519174 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:1107999284 (1.1 GB)  TX bytes:46049224 (46.0 MB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:22140 errors:0 dropped:0 overruns:0 frame:0
          TX packets:22140 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:2248954 (2.2 MB)  TX bytes:2248954 (2.2 MB)
```

## Boot BeagleBone Black board from tftp server :

### 3-Installing and Testing TFTP Server in Ubuntu

a. Install following packages.

`sandeeptux@sandeeplinux:~$ sudo apt-get install xinetd tftpd tftp`

b. Create /etc/xinetd.d/tftp and put the following lines & save it.

`sandeeptux@sandeeplinux:~$ sudo vi /etc/xinetd.d/tftp`

```
service tftp
{
protocol        = udp
port            = 69
socket_type     = dgram
wait            = yes
user            = nobody
server          = /usr/sbin/in.tftpd
server_args     = /tftpboot
disable         = no
}
```
c. Create a folder /tftpboot this should match whatever you gave in server_args. In my case  it is tftpboot. 
```
sandeeptux@sandeeplinux:~$ sudo mkdir /tftpboot
sandeeptux@sandeeplinux:~$ sudo chmod -R 777 /tftpboot/
sadeeptux@sandeeplinux:~$ sudo chown -R nobody /tftpboot/
```

d. Restart the xinetd service. 
```
sandeeptux@sandeeplinux:~$ sudo /etc/init.d/xinetd stop
xinetd stop/waiting
sandeeptux@sandeeplinux:~$ sudo /etc/init.d/xinetd start
xinetd start/running, process 16431
```
e. Now your tftp server should be up and running. To testing tftp server, create a file named test with some content in /tftpboot path of the tftp server. 

```
sandeeptux@sandeeplinux:~$ ls / > /tftpboot/test
sandeeptux@sandeeplinux:~$ sudo chmod -R 777 /tftpboot/
sandeeptux@sandeeplinux:~$ ls /tftpboot/test -1h
/tftpboot/test
```
f. Obtain the ip address of the tftp server using ifconfig command, in this example we will consider the ip address as 192.168.1.2. Now in some other system follow the following steps
```
sandeeptux@sandeeplinux:~$ tftp 192.168.1.2
tftp> get test
Received 154 bytes in 0.1 seconds
tftp> quit
```
-----------------------------------
### Boot BeagleBone Black from your TFTP server

1. Connect the Ethernet cable to the board's PHY.

2. Copy the kernel Image(uImage), dtb files to the /tftpboot directory of your Host Machine (x86 intel Linux)
```
sandeeptux@sandeeplinux:/tftpboot$ ls
am335x-boneblack.dtb  uImage
```
3. Power on your BeagleBone black board. Hit any key to stop autoboot and enter U-boot commandline.

4. Use the following settings for U-Boot environment variables bootargs and bootcmd to boot into the kernel. 

 ```
=> setenv serverip 192.168.1.2
=> setenv ipaddr 192.168.1.4
=> setenv bootargs console=ttyO0,115200n8 root=/dev/mmcblk0p2 ro rootfstype=ext4 rootwait
=> tftpboot 0x80F80000 am335x-boneblack.dtb
=> tftpboot 0x80007FC0 uImage 
=> bootm 0x80007FC0 - 0x80F80000
```
### The example shown is to use the filesystem already in place (i.e. ext4 on mmcblk0p2). The preferred option is to export a rootFS filesystem. 

### Note:Why uImage ? 
### Ans- An uImage includes a U-Boot header which contains the load address that the kernel should be loaded & A uImage file is a kernel with a modified header for u-boot. A tool called mkimage is used to convert a zImage (regular kernel compressed image) to a uImage(u-boot image)file. And No, zImage files, as they are, are not compatible with U-Boot.An uImage can generate by using this Command called `make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- uImage LOADADDR=0X80008000 -j4` while Kernel complation process.

### Example In my BBB board :
```
U-Boot SPL 2018.01-00531-g748277c (Jan 27 2018 - 22:04:49)
Trying to boot from MMC1
*** Warning - bad CRC, using default environment



U-Boot 2018.01-00531-g748277c (Jan 27 2018 - 22:04:49 +0530)

CPU  : AM335X-GP rev 2.1
Model: TI AM335x BeagleBone Black
DRAM:  512 MiB
NAND:  0 MiB
MMC:   OMAP SD/MMC: 0, OMAP SD/MMC: 1
*** Warning - bad CRC, using default environment

No USB device found
<ethaddr> not set. Validating first E-fuse MAC
Net:   eth0: ethernet@4a100000
Hit any key to stop autoboot:  0 
=> setenv serverip 192.168.1.2
=> setenv ipaddr 192.168.1.4  
=> tftpboot 0x80F80000 am335x-boneblack.dtb
link up on port 0, speed 100, full duplex
Using ethernet@4a100000 device
TFTP from server 192.168.1.2; our IP address is 192.168.1.4
Filename 'am335x-boneblack.dtb'.
Load address: 0x80f80000
Loading: #############
	 961.9 KiB/s
done
Bytes transferred = 63081 (f669 hex)
=> setenv bootargs console=ttyO0,115200n8 root=/dev/mmcblk0p2 ro rootfstype=ext4 rootwait
=> tftpboot 0x80F80000 am335x-boneblack.dtb
link up on port 0, speed 100, full duplex
Using ethernet@4a100000 device
TFTP from server 192.168.1.2; our IP address is 192.168.1.4
Filename 'am335x-boneblack.dtb'.
Load address: 0x80f80000
Loading: #######
	 889.6 KiB/s
done
Bytes transferred = 34639 (874f hex)

=> tftpboot 0x80007FC0 uImage
link up on port 0, speed 100, full duplex
Using ethernet@4a100000 device
TFTP from server 192.168.1.2; our IP address is 192.168.1.4
Filename 'uImage'.
Load address: 0x80007fc0
Loading: #################################################################
	 #################################################################
	 #################################################################
	 #################################################################
	 #################################################################
	 #################################################################
	 #################################################################
	 #################################################################
	 #################################################################
	 #################################################################
	 #################################################################
	 #################################################################
	 ########################
	 1.7 MiB/s
done
Bytes transferred = 4113328 (3ec3b0 hex)
=> bootm 0x80007FC0 - 0x80F80000
## Booting kernel from Legacy Image at 80007fc0 ...
   Image Name:   Linux-4.4.110+
   Created:      2018-02-19   4:32:37 UTC
   Image Type:   ARM Linux Kernel Image (uncompressed)
   Data Size:    8042952 Bytes = 7.7 MiB
   Load Address: 80008000
   Entry Point:  80008000
   Verifying Checksum ... OK
## Flattened Device Tree blob at 80f80000
   Booting using the fdt blob at 0x80f80000
   XIP Kernel Image ... OK
   Loading Device Tree to 8ffed000, end 8ffff668 ... OK

Starting kernel ...
```
