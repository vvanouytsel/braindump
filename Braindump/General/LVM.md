#linux 

## Setting up a LV

- Create a new partition with type 'Linux LVM (83)'

```bash
$ fdisk -c /dev/vdb
```

- Create a PV from the volume

```bash
$ pvcreate /dev/vdb1
```

- Create a VG on the PV

```bash
$ vgcreate vg_tm-poc-k3s_data /dev/vdb1
```

- Create a LV on the VG

```bash
$ lvcreate -l 100%FREE -n lv_data vg_tm-poc-k3s_data
```

- Create a ext4 filesystem on the LV

```bash
$ mkfs.xfs /dev/mapper/vg_tm--poc--k3s_data-lv_data
```

- Mount the LV

```bash
$ mount /dev/mapper/vg_tm--poc--k3s_data-lv_data /mnt/data/
# You could also add the following line to /etc/fstab
/dev/mapper/vg_tm--poc--k3s_data-lv_data /mnt/data                    xfs     defaults        0 0
```

## Listing

- List logical volumes and the physical device they use

```bash
$ lvs -o +devices
  LV                         VG                Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices
  lv_local_images            vg_tm-host32_root -wi-ao---- 500.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(288)
  lv_local_images            vg_tm-host32_root -wi-ao---- 500.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(25504)
  lv_local_images            vg_tm-host32_root -wi-ao---- 500.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(21664)
  lv_local_images            vg_tm-host32_root -wi-ao---- 500.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(23424)
  lv_local_images            vg_tm-host32_root -wi-ao---- 500.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(9504)
  lv_local_images            vg_tm-host32_root -wi-ao---- 500.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(28416)
  lv_local_images            vg_tm-host32_root -wi-ao---- 500.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(10624)
  lv_local_images            vg_tm-host32_root -wi-ao---- 500.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(18784)
  lv_mdatp_var               vg_tm-host32_root -wi-ao----   1.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(28384)
  lv_root                    vg_tm-host32_root -wi-ao----   4.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(160)
  lv_swap                    vg_tm-host32_root -wi-ao----   1.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(0)
  lv_tm-elk31_data           vg_tm-host32_root -wi-ao---- 128.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(5408)
  lv_tm-elk31_root           vg_tm-host32_root -wi-ao----  10.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(5088)
  lv_tm-test-mercator31_data vg_tm-host32_root -wi-ao---- 100.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(15904)
  lv_tm-test-mercator31_data vg_tm-host32_root -wi-ao---- 100.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(25184)
  lv_tm-test-mercator31_root vg_tm-host32_root -wi-ao----  10.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(24864)
  lv_var                     vg_tm-host32_root -wi-ao----   4.00g                                                     /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080(32)
```

- List physical volumes and the logical volumes that it uses

```bash
$ pvdisplay -m
  --- Physical volume ---
  PV Name               /dev/mapper/luks-a8d23313-d6ec-436a-8a6b-08342d59b080
  VG Name               vg_tm-host32_root
  PV Size               893.12 GiB / not usable 31.00 MiB
  Allocatable           yes
  PE Size               32.00 MiB
  Total PE              28579
  Free PE               4323
  Allocated PE          24256
  PV UUID               m7Wk62-4O5D-GBvu-kb9b-I093-tars-zkwR2f

  --- Physical Segments ---
  Physical extent 0 to 31:
    Logical volume /dev/vg_tm-host32_root/lv_swap
    Logical extents 0 to 31
  Physical extent 32 to 159:
    Logical volume /dev/vg_tm-host32_root/lv_var
    Logical extents 0 to 127
```