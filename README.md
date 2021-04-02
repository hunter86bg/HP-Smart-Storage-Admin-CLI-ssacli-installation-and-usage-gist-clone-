# HP Smart Storage Admin CLI (`ssacli`) installation and usage on Proxmox PVE (6.x)

## Why use HP Smart Storage Admin CLI?
You can use `ssacli` (smart storage administrator command line interface) tool to manage any of supported HP Smart Array Controllers in your Proxmox host without need to reboot your server to access *Smart Storage Administrator* in BIOS.  That means no host downtime when managing your storage.

CLI is not as convenient as GUI interface provided by BIOS or desktop utilities, but still allows you to fully manage your controller, physical disks and logical drives on the fly with no Proxmox host downtime.

`ssacli` replaces older `hpssacli`, but shares the same syntax and adds support for newer servers and controllers.

## Installation
Installation process of the  `ssacli` package on Proxmox is the same as on any other Debian based system.

*NOTE: HP does not provide **Buster** repository for Debian 10/Proxmox 6.x, but we can still use **Stretch** repository even that it's targeted for Debian 9/Proxmox 5.x.*

### Add HP's MCP repository
To add HP's MCP repository to `apt` sources list use this command:

	echo "deb http://downloads.linux.hpe.com/SDR/repo/mcp stretch/current non-free" > /etc/apt/sources.list.d/hp-mcp.list

### Download and add public keys for the MCP repository
Public keys are needed by `apt` to verify repository signatures, so we need to download and add them. There are four of them and we need to download and them all:

	wget -q -O - http://downloads.linux.hpe.com/SDR/hpPublicKey1024.pub | apt-key add -
	wget -q -O - http://downloads.linux.hpe.com/SDR/hpPublicKey2048.pub | apt-key add -
	wget -q -O - http://downloads.linux.hpe.com/SDR/hpPublicKey2048_key1.pub | apt-key add -
	wget -q -O - http://downloads.linux.hpe.com/SDR/hpePublicKey2048_key1.pub | apt-key add -

### Installing the `ssacli` package

First, we need to update available packages:

	apt update

And then we can install the `ssacli` package:

	apt install ssacli

## Using the `ssacli` tool

**Command short and long names**
All commands have a short name to reduce the length of the total input provided to the `ssacli` tool. You can use short or long name. Here is a list of all commands and their long and short names:

* chassisname = ch
* controller = ctrl
* logicaldrive = ld
* physicaldrive = pd
* drivewritecache = dwc
* licensekey = lk

You can specify disks also as:

* A range of drives (bay 1 to 3): 1I:1:1-1I:1:3
* Drives that are unassigned: allunassigned

## Command examples
Here are few examples of  `ssacli` commands that you can use to diagnose and manage your HP Smart Storage Controller.

**Show available controllers**

	ssacli ctrl all show

**Show controllers status**

	ssacli ctrl all show status

**Show detailed controllers information**

	ssacli ctrl all show detail

**Show controllers configuration**

	ssacli ctrl all show config

**Rescan for new devices**
Useful after swapping disks in bays, etc...
	
	ssacli rescan

**Show all physical disks (or their status) (controller slot 0)**

	ssacli ctrl slot=0 pd all show
	ssacli ctrl slot=0 pd all show status

**Show all physical disks detailed information (controller slot 0)**
	
	ssacli ctrl slot=0 pd all show detail

**Show logical drives (or their status) (controller slot 0, all or specific logical drive(s))**

	ssacli ctrl slot=0 ld all show
	ssacli ctrl slot=0 ld all show status
	
	ssacli ctrl slot=0 ld 1 show
	ssacli ctrl slot=0 ld 1 show status

**Show detailed logical drives information (controller slot 0, all or specific logical drive(s))**

	ssacli ctrl slot=0 ld all show detail
	ssacli ctrl slot=0 ld 1 show detail

**Show array information (controller slot 0, array A)**	

	ssacli ctrl slot=0 array a show

**Show array status (controller slot 0, all arrays)**	

	ssacli ctrl slot=0 array all show status

**Create new RAID 0 logical drive (controller slot 0, disk in port 1I:box 1:bay 1)**

	ssacli ctrl slot=0 create type=ld drives=1I:1:1 raid=0

**Create new RAID 1 logical drive (controller slot 0, disks in port 1I:box 1:bay 1 and 2)**
	
	ssacli ctrl slot=0 create type=ld drives=1I:1:1,1I:1:2 raid=1

**Create new RAID 5 logical drive  (controller slot 0, diks in port 1I:box 1:bay 1 to 4)**

	ssacli ctrl slot=0 create type=ld drives=1I:1:1-1I:1:4 raid=5

**Delete logical drive (controller slot 0, logical drive 1)**

	ssacli ctrl slot=0 ld 1 delete

**Add new physical disks to logical drive (controller slot 0, logical drive 1, disks in port 1I:box 1:bay 6 and 7)**

	ssacli ctrl slot=0 ld 2 add drives=1I:1:6,1I:1:7

**Add spare disks (controller slot 0, logical drive 1, array A, disks in port 1I:box 1:bay 6 and 7)**

	ssacli ctrl slot=0 array a add spares=1I:1:6,1I:1:7

**Add global spare disks (controller slot 0, logical drive 1, all arrays, disks in port 1I:box 1:bay 6 and 7)**

	ssacli ctrl slot=0 array all add spares=1I:1:6,1I:1:7

**Turn on/off blink logical drive LED (controller slot 0, logical drive 1)**

	ssacli ctrl slot=0 ld 1 modify led=on
	ssacli ctrl slot=0 ld 1 modify led=off

**Turn on/off blink physical disk LED (controller slot 0, physical disk port 1I:box 1:bay 1)**  
	
	ssacli ctrl slot=0 pd 1I:1:1 modify led=on
	ssacli ctrl slot=0 pd 1I:1:1 modify led=off

**Modify smart array cache read and write ratio (controller slot 0, cacheratio 80% read/20% write)**  

	ssacli ctrl slot=0 modify cacheratio=80/20

**Show physical drive write cache status (controller slot 0)** 

	ssacli ctrl slot=0 modify dwc=?

**Enable/disable physical drive write cache (controller slot 0)** 
Important: Because physical drive write cache is not battery-backed, you could lose data if a power failure occurs during a write process. To minimize this possibility, use a backup power supply.

	ssacli ctrl slot=0 modify dwc=enable
	ssacli ctrl slot=0 modify dwc=disable

**Show status of smart array write cache when no battery is present (no-battery write cache option, controller slot 0)**

	ssacli ctrl slot=0 modify nbwc=?

**Enable/disable smart array write cache when no battery is present (no-battery write cache option, controller slot 0)**

	ssacli ctrl slot=0 modify nbwc=enable
	ssacli ctrl slot=0 modify nbwc=disable

**Enable/disable smart array cache for certain Logical Volume (controller slot 0, logical drive 1)**
	
	ssacli ctrl slot=0 ld 1 modify arrayaccelerator=enable
	ssacli ctrl slot=0 ld 1 modify arrayaccelerator=disable

**Enable/disable SSD Smart Path (controller slot 0, array A)**
	
	ssacli ctrl slot=0 array a modify ssdsmartpath=enable
	ssacli ctrl slot=0 array a modify ssdsmartpath=disable

**Show spare activation mode**

	ssacli ctrl slot=0 modify spareactivationmode=?

**Set spare activation mode**

	ssacli ctrl slot=0 modify spareactivationmode=predictive
	ssacli ctrl slot=0 modify spareactivationmode=failure

**Show rebuild priority**
	
	ssacli ctrl slot=0 modify rp=?
	
**Modify rebuild priority**

	ssacli ctrl slot=0 modify rp=low
	ssacli ctrl slot=0 modify rp=medium
	ssacli ctrl slot=0 modify rp=mediumhigh
	ssacli ctrl slot=0 modify rp=high

**Erase Physical Drive (controller slot 0, physical disk port 1I:box 1:bay 1)**  

	ssacli ctrl slot=0 pd 1I:1:1 modify erase
