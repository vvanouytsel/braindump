---
tags:
  - storage
  - linux
  - disks
---

## Working with Devices

In Linux, your storage disks are available as block devices in `/dev/`.

* Check the layout of your storage devices via `lsblk`

```bash
$ lsblk

NAME               MAJ:MIN RM    SIZE RO TYPE  MOUNTPOINTS
loop0                7:0    0    3.2G  1 loop  
nvme0n1            259:0    0    3.5T  0 disk  
└─nvme0n1p1        259:2    0    3.5T  0 part  
  └─md3              9:3    0    3.5T  0 raid1 
    └─libvirt-data 253:0    0    3.5T  0 lvm   
nvme3n1            259:1    0    3.5T  0 disk  
└─nvme3n1p1        259:3    0    3.5T  0 part  
  └─md3              9:3    0    3.5T  0 raid1 
    └─libvirt-data 253:0    0    3.5T  0 lvm   
nvme1n1            259:4    0    1.7T  0 disk  
├─nvme1n1p1        259:6    0      1G  0 part  
│ └─md0              9:0    0   1022M  0 raid1 
├─nvme1n1p2        259:7    0      1G  0 part  
│ └─md1              9:1    0 1023.9M  0 raid1 
└─nvme1n1p3        259:8    0    1.7T  0 part  
  └─md2              9:2    0    1.7T  0 raid1 
    ├─os-swap      253:1    0      1G  0 lvm   
    ├─os-tmp       253:2    0    100G  0 lvm   
    ├─os-var       253:3    0    200G  0 lvm   
    └─os-root      253:4    0    100G  0 lvm   
nvme2n1            259:5    0    1.7T  0 disk  
├─nvme2n1p1        259:9    0      1G  0 part  
│ └─md0              9:0    0   1022M  0 raid1 
├─nvme2n1p2        259:10   0      1G  0 part  
│ └─md1              9:1    0 1023.9M  0 raid1 
└─nvme2n1p3        259:11   0    1.7T  0 part  
  └─md2              9:2    0    1.7T  0 raid1 
    ├─os-swap      253:1    0      1G  0 lvm   
    ├─os-tmp       253:2    0    100G  0 lvm   
    ├─os-var       253:3    0    200G  0 lvm   
    └─os-root      253:4    0    100G  0 lvm 
```

* Check your RAID devices

```bash
$ cat /proc/mdstat

Personalities : [raid1] 
md2 : active raid1 nvme1n1p3[0] nvme2n1p3[1]
      1873143808 blocks super 1.2 [2/2] [UU]
      bitmap: 0/14 pages [0KB], 65536KB chunk

md1 : active raid1 nvme1n1p2[0] nvme2n1p2[1]
      1048512 blocks super 1.0 [2/2] [UU]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md0 : active raid1 nvme1n1p1[0] nvme2n1p1[1]
      1046528 blocks super 1.2 [2/2] [UU]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md3 : active raid1 nvme0n1p1[0] nvme3n1p1[1]
      3750604800 blocks super 1.2 [2/2] [UU]
      bitmap: 0/28 pages [0KB], 65536KB chunk

unused devices: <none>
root@rescue ~ # 
```

You can use the `mdadm` command to manage RAID devices.

* List active RAID arrays

```bash
$ mdadm --detail --scan

ARRAY /dev/md/3 metadata=1.2 name=3 UUID=f2b17e3d:3e11e377:d495bced:98f4c630
ARRAY /dev/md/2 metadata=1.2 name=2 UUID=8840caf7:e18d445c:cf9de997:59e8fd68
```

* Get more details about a specific RAID array

```bash
$ mdadm --detail /dev/md2

/dev/md2:
           Version : 1.2
     Creation Time : Mon Sep 23 16:26:37 2024
        Raid Level : raid1
        Array Size : 1873143808 (1786.37 GiB 1918.10 GB)
     Used Dev Size : 1873143808 (1786.37 GiB 1918.10 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Tue Sep 24 09:08:21 2024
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : bitmap

              Name : 2
              UUID : 8840caf7:e18d445c:cf9de997:59e8fd68
            Events : 331

    Number   Major   Minor   RaidDevice State
       0     259        8        0      active sync   /dev/nvme1n1p3
       1     259       11        1      active sync   /dev/nvme2n1p3
```

## Disk IDs

During boot names are assigned to your disks. For example `/dev/nvme0n1`.  However these names are not persistent. If you need to make sure that you are always using the exact same disk, you will have to use IDs instead.

* List the disks by ID

```bash
$ ls -l /dev/disks/by-id

lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-eui.01000000000000005cd2e49c9a615651 -> ../../nvme3n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-eui.01000000000000005cd2e4be3a605651 -> ../../nvme0n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-eui.3634473057c218010025385300000001 -> ../../nvme2n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-eui.3634473057c218050025385300000002 -> ../../nvme1n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-INTEL_SSDPF2KX038T1_PHAX345307ZP3P8CGN -> ../../nvme0n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-INTEL_SSDPF2KX038T1_PHAX345307ZP3P8CGN_1 -> ../../nvme0n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-INTEL_SSDPF2KX038T1_PHAX346304RA3P8CGN -> ../../nvme3n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-INTEL_SSDPF2KX038T1_PHAX346304RA3P8CGN_1 -> ../../nvme3n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-SAMSUNG_MZQL21T9HCJR-00A07_S64GNS0WC21801 -> ../../nvme2n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-SAMSUNG_MZQL21T9HCJR-00A07_S64GNS0WC21801_1 -> ../../nvme2n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-SAMSUNG_MZQL21T9HCJR-00A07_S64GNS0WC21805 -> ../../nvme1n1
lrwxrwxrwx 1 root root 13 Sep 24 09:41 nvme-SAMSUNG_MZQL21T9HCJR-00A07_S64GNS0WC21805_1 -> ../../nvme1n1
```