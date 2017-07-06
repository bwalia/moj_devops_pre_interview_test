# Write a script in python or bash that can configure ephemeral storage on an AWS instance to create a logical volume that can be mounted on /opt.
# The script would be run as part of the bootstrap process and make use of all ephemeral storage devices.

# Requirements

Bash or Python script

# Steps

----------------

#Initialize devices to use with the LVM
#In order to do this, we need to use the pvcreate command. pvcreate is used to initialize disk or partitions that will be used by LVM. It can either initialize a complete physical disk or a partition on physical disks.

Syntax: # sudo pvcreate <device-1> <device-2> <device-3>

sudo pvcreate /dev/sdf /dev/sdg /dev/sdh

#Display device attributes - For this, we need to use the pvdisplay command to display information about physical disks

sudo pvdisplay

# Create the LVM volume group

# Once the devices are initialized, you can create the LVM volume group. A group of physical volumes or disks are combined together into a single storage file which is referred to as the LVM volume group. For this, we used the vgcreate command to create a volume group by providing the name of the volume group and the path of actual physical volumes.

Syntax: # sudo vgcreate <volume-name> <device-1> <device-2> <device-3> 

sudo vgcreate my-vol /dev/sdf /dev/sdg /dev/sdh

# Once the LVM volume group is created, you can use the vgdisplay command to show its attributes.

sudo vgdisplay

# There are also commands like "vgscan", which scan all disks for LVM volume groups

# Create Logical Volumes
#Once the LVM volume group is created, it's time to create logical volumes.

Syntax: sudo lvcreate --name <logical-volume-name> --size <size-of-volume> <lvm-volume-name>

#In this case, we are creating two logical volumes (data and backup).

#For data logical volume:

sudo lvcreate --name data --size 5GB my-logical-volume

#For backup logical volume:
 
sudo lvcreate --name backup --size 15GB my-logical-volume

#Once the logical volumes are created, we can view their status using the "lvdisplay" command.

#Format your logical volumes
#Once the logical volumes are created, we can format them using any filesystem like ext4, XFS, and more.

#If you are using the ext4 filesystem:

#mkfs.ext4 <logical-volume-path>

#In our case, it's:

# mkfs.ext4 /dev/my-logical-volume/data

sudo mkfs.ext4 /dev/my-logical-volume/data

# mkfs.ext4 /dev/my-logical-volume/backup

sudo mkfs.ext4 /dev/my-logical-volume/backup

#Mount the logical volumes, Use the mount command to mount:

# mount <logical-volume> <mount-point>

sudo mkdir /data
sudo mount /dev/my-logical-volume/data /data

sudo mkdir /backup
sudo mount /dev/my-logical-volume/backup /backup

# If you want you mount points to be available after reboot, you can add mount point entries in '/etc/fstab' as:

/dev/my-logical-volume/data /data ext4 defaults 0 0
/dev/my-logical-volume/backup /backup ext4 defaults 0 0

To view the status of the mount, use the mount command.

Once the logical volumes are created and mounted, you can use them as your normal volumes. However, LVM offers:
Better performance – If your data is spread across multiple EBS volumes using LVM, you can leverage dedicated network throughput between EC2 instances wand EBS volumes. This provides you better network throughput over a single network channel between EC2 and EBS volumes.
The ability to grow – You can expand your volume at any time according to your requirement. More EBS volumes can be added to your LVM volume rather than creating a snapshot of an EBS volume and expanding it.
EBS volume snapshots – You need to ensure that there are no operations happening in your EBS volume during snapshots. You can suspend operations on your LVM volume by using the dmsetup command.
Syntax for suspend : # dmsetup suspend <lvm-volume-name> 
Syntax for resume : # dmsetup resume <lvm-volume-name>

As mentioned above, LVM volume backup can be done using the EBS snapshot procedure, ensuring that LVM volume operations are suspended for that duration of time. To understand more about LVM backup concepts, check out "Disk Array Backup on EC2 Part I: Concepts". If you are interested in understanding consistency challenges with LVM backups, take a look at "Disk Array Backup on EC2 Part II: Consistency Issues". 
In order to perform LVM EBS snapshot backups, see "Disk Array Backup on EC2 Part III: LVM backup with EBS Volumes".


