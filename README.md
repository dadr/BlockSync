# BlockSync
Version 1.0 
## Description
 BlockSync will sync a clone of a btrfs drive used for Urbackup.  This script is intended to be used to sync a primary backup disk drive with one or more additional drives that are not always attached to the system.  The idea is to allow these additional drives to be stored off-line and off-site - except when they are synced with the original backup drive.
 


Urbackup is a decent backup system that can support significant de-duplication across backup clients when it's used with a btrfs backup volume. (Urbackup can do many other things too, but this is the model that this script is concerned with.)

However, Urbackup does not have good tools to create offline and offsite duplicates of the backup store.  The best solution Urbackup offers for this type of need with btrfs is to run multiple instances of the backup service and to point those different instances to different backup volumes.

So to backup the backup, another tool is needed, like dd for example.  However, dd would copy all the data from one disk to another every time.  Such an arrangement might take days each time a sync is performed.

Enter blocksync-fast: a C program that essentially does what dd does, except it also tracks blocks that change, so that future copies become sync operations that only copy blocks that have changed in the original.

## Dependencies
 BlockSync has a major dependency on blocksync-fast, which is available here on github:  https://github.com/nethappen/blocksync-fast   Also, this script was developed and tested on Ubuntu 24.04.
 
When you clone a btrfs volume, there is a risk that the filesystem will get confused if two disks with the same UUID are mounted.   There is some good news that btrfs has become more robust about this and supports "temporary fsid" specifically to deal with cloned drives as of linux kernel 6.6.  ( https://btrfs.readthedocs.io/en/latest/Feature-by-version.html )    To work with older kernels and to limit the dependency on this feature it is recommended that /etc/fstab have entries added to prevent the system from trying to mount or present the opportunity to mount a clone volume from the desktop.   An example of fstab showing both the backup drive as well as clone-to drives follows:

	# Mount backup filesysem.
	/dev/disk/by-id/ata-OOS14000G_000BJQZD-part1 /mnt/backup btrfs nosuid,nodev,noexec 0 0
	
	# Do not mount the drives that are clones of the backup drive
	/dev/disk/by-id/ata-OOS14000G_0007QVYW-part1 /media/clone1/	btrfs defaults,noauto	00
	/dev/disk/by-id/ata-OOS14000G_000HPZ44-part1 /media/clone2/	btrfs defaults,noauto	00
	

Regardless of the new feature or your fstab configuration, this script never mounts the cloned-to drive.

## Usage

To make use of this script you should edit the file and makes changes in the configuration block as described below.

### Command line

	blocksync [-h | -?]
Prints out usage syntax

	blocksync [-d clone-device | -t]
Performs sync operation with options:

-d provides a clone device to use in addition to the ones that are pre-configured  

-t performs a "dry run" of the sync operation, printing the blocksync-fast command instead of running it.

The program must be run as root, and will ask for authorization if not run that way.

### Configuration
To configure blocksync, edit the script file.  There is a section labeled *Configuration Block* near the start of the script.
That section includes the following items:

**sourceDisk** -  the path to the unique disk used as the source to sync from

**sourceMntPt** - the path to the mount point for sourceDisk

**backupDisks** - a list of path names for unique devices of candidate clone-to disks 

**digestFileDir** - the directory where digest files are to be stored.  These track hash values for the blocks on the backup disks

**hashAlgo** - the hashing algorithm to use to track changes in disk blocks

The script has some default values for these options, but it will be necessary at a minimum to set sourceDisk and at least one value for backupDisks.   Note that the devices are entire disks, and not disk partitions.

For the disks identified by sourceDisk and backupDisks it is recommended to identify the drive using its wwn number.   If that is not available, then use the link-id.  These can be determined with the following command:

	lsblk -ndo name,wwn,id-link

Take the output of this command to determine the proper path and filename in the form of 

	/dev/disk/by-id/wwn-<result from lsblk>

While the wwn identifiers require prependiing "wwn-" the link-id identifiers do not.  Examples of both are in the defaults of the script.

Do not use UUID because this script will clone the drive (like dd would) and UUID will not be unique.
 
 You **can** use /dev/sda etc., but beware that it can change if hardware changes between boots.  
 
 Also beware using **/dev/disk/by-id/usb-*** as that can name the adapter instead of the drive attached to it.
 
 If there is not a **wwn-** entry for the desired drive, then use the device that is listed by the lsblk id-link output column.  This is safe because it is composed of the interface type, model and  serial number of the device.  
 

For **hashAlgo** , the default, "CRC32_RFC1510", is probably a good choice.  To determine whether a better choice exists, use the command
 
	 blocksync-fast --benchmark-algos
 
 The results will show both the speed and bytes for the algorithm which will affect run time and digest file size, respectively.
 
## License 
This script is provided under Apache license 2.0 - except the hms() function which
             is/was published at https://www.shellscript.sh/examples/hms/ without mentioning a license.

 
 



