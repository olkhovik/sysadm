# **Домашняя работа к занятию «3.5. Файловые системы»**
## _Задача №1_
**Узнайте о sparse (разряженных) файлах.**

_Выполнено_

## _Задача №2_
**Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?**

Нет, не могут, так как они все ссылаются на один Inode, а метаданные файла, в том числе и права доступа, владельцы хранятся в Inode.


## _Задача №3_
**Сделайте vagrant destroy на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:**

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.provider :virtualbox do |vb|
    lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
    lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
    vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
    vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
  end
end
```
**Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.**

Новая виртуалка поднялась:
```
root@vagrant:/home/vagrant# fdisk -l
Disk /dev/sda: 64 GiB, 68719476736 bytes, 134217728 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3f94c461

Device     Boot   Start       End   Sectors  Size Id Type
/dev/sda1  *       2048   1050623   1048576  512M  b W95 FAT32
/dev/sda2       1052670 134215679 133163010 63.5G  5 Extended
/dev/sda5       1052672 134215679 133163008 63.5G 8e Linux LVM


Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/vgvagrant-root: 62.55 GiB, 67150807040 bytes, 131153920 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/vgvagrant-swap_1: 980 MiB, 1027604480 bytes, 2007040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

## _Задача №4_
**Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.**

```
root@vagrant:/home/vagrant# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x648bb8a8.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-5242879, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (4196352-5242879, default 4196352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879):

Created a new partition 2 of type 'Linux' and of size 511 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```



## _Задача №5_
**Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.**

```
root@vagrant:/home/vagrant# sfdisk -d /dev/sdb | sfdisk /dev/sdc
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0x648bb8a8.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x648bb8a8

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

## _Задача №6_
**Соберите `mdadm` RAID1 на паре разделов 2 Гб.**

```
root@vagrant:/home/vagrant# mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

## _Задача №7_
**Соберите mdadm RAID0 на второй паре маленьких разделов.**

```
root@vagrant:/home/vagrant# mdadm --create --verbose /dev/md1 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```

## _Задача №8_
**Создайте 2 независимых PV на получившихся md-устройствах.**

```
root@vagrant:/home/vagrant# pvcreate /dev/md0
  Physical volume "/dev/md0" successfully created.
root@vagrant:/home/vagrant# pvcreate /dev/md1
  Physical volume "/dev/md1" successfully created.
```

## _Задача №9_
**Создайте общую volume-group на этих двух PV.**

```
root@vagrant:/home/vagrant# vgcreate VolumeGroup_01 /dev/md0 /dev/md1
  Volume group "VolumeGroup_01" successfully created
```
проверяем:
```
root@vagrant:/home/vagrant# vgdisplay
  --- Volume group ---
  VG Name               vgvagrant
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <63.50 GiB
  PE Size               4.00 MiB
  Total PE              16255
  Alloc PE / Size       16255 / <63.50 GiB
  Free  PE / Size       0 / 0
  VG UUID               PaBfZ0-3I0c-iIdl-uXKt-JL4K-f4tT-kzfcyE

  --- Volume group ---
  VG Name               VolumeGroup_01
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <2.99 GiB
  PE Size               4.00 MiB
  Total PE              765
  Alloc PE / Size       0 / 0
  Free  PE / Size       765 / <2.99 GiB
  VG UUID               ZFq7tN-OIIn-82X9-i8Jz-I1xj-Eo0C-9mhLNS
```

## _Задача №10_
**Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.**

```
root@vagrant:/home/vagrant# lvcreate -L100 -n md0_1 VolumeGroup_01 /dev/md1
  Logical volume "md0_1" created.
```
проверяем:
```
root@vagrant:/home/vagrant# lvdisplay
  --- Logical volume ---
  LV Path                /dev/vgvagrant/root
  LV Name                root
  VG Name                vgvagrant
  LV UUID                ybvP3g-N4gJ-FMMr-WfRk-Ermg-cftw-In20VW
  LV Write Access        read/write
  LV Creation host, time vagrant, 2021-07-28 17:45:53 +0000
  LV Status              available
  # open                 1
  LV Size                <62.54 GiB
  Current LE             16010
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

  --- Logical volume ---
  LV Path                /dev/vgvagrant/swap_1
  LV Name                swap_1
  VG Name                vgvagrant
  LV UUID                GoQVTk-fU79-pbTZ-vX0W-9DbL-p7OI-XCSHPj
  LV Write Access        read/write
  LV Creation host, time vagrant, 2021-07-28 17:45:53 +0000
  LV Status              available
  # open                 2
  LV Size                980.00 MiB
  Current LE             245
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1

  --- Logical volume ---
  LV Path                /dev/VolumeGroup_01/md0_1
  LV Name                md0_1
  VG Name                VolumeGroup_01
  LV UUID                F3uycN-Thzr-ejTr-TL1l-ceXT-gqcj-lCbAKJ
  LV Write Access        read/write
  LV Creation host, time vagrant, 2021-11-22 22:39:45 +0000
  LV Status              available
  # open                 0
  LV Size                100.00 MiB
  Current LE             25
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     4096
  Block device           253:2
```
## _Задача №11_
**Создайте `mkfs.ext4` ФС на получившемся LV.**

```
root@vagrant:/home/vagrant# mkfs.ext4 /dev/VolumeGroup_01/md0_1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```
проверяем:
```
root@vagrant:/home/vagrant# blkid
/dev/sda1: UUID="7D3B-6BE4" TYPE="vfat" PARTUUID="3f94c461-01"
/dev/sda5: UUID="Mx3LcA-uMnN-h9yB-gC2w-qm7w-skx0-OsTz9z" TYPE="LVM2_member" PARTUUID="3f94c461-05"
/dev/mapper/vgvagrant-root: UUID="b527b79c-7f45-4e2b-a90f-1a4e9cb477c2" TYPE="ext4"
/dev/mapper/vgvagrant-swap_1: UUID="fad91b1f-6eed-4e4b-8dbf-913ba5bcacc7" TYPE="swap"
/dev/sdb1: UUID="df347ca8-bef5-f41b-1c0d-333b9238b1f4" UUID_SUB="9b84eaa3-1d66-e23f-9dab-2fb82a5f819a" LABEL="vagrant:0" TYPE="linux_raid_member" PARTUUID="648bb8a8-01"
/dev/sdb2: UUID="075c569d-4e46-cc12-c9b6-aa99f77c8ff9" UUID_SUB="8957e140-78fb-df5e-d936-ba3abed5fa40" LABEL="vagrant:1" TYPE="linux_raid_member" PARTUUID="648bb8a8-02"
/dev/sdc1: UUID="df347ca8-bef5-f41b-1c0d-333b9238b1f4" UUID_SUB="a56eb368-3949-8f54-3a2b-f1ad9db1861b" LABEL="vagrant:0" TYPE="linux_raid_member" PARTUUID="648bb8a8-01"
/dev/sdc2: UUID="075c569d-4e46-cc12-c9b6-aa99f77c8ff9" UUID_SUB="1537d5c8-4099-1984-f7b6-39bbde057d84" LABEL="vagrant:1" TYPE="linux_raid_member" PARTUUID="648bb8a8-02"
/dev/md0: UUID="xKFUOJ-8hqH-YL70-5lrB-UeLu-mgZR-S8NQTq" TYPE="LVM2_member"
/dev/md1: UUID="e9MANv-Ixvl-KHsn-4KK6-q360-fX4E-Dpn9nt" TYPE="LVM2_member"
/dev/mapper/VolumeGroup_01-md0_1: UUID="e4fa881b-b5ac-466f-9188-2bf3fe5502a2" TYPE="ext4"
```

## _Задача №12_
**Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.**

```
root@vagrant:/home/vagrant# mkdir /tmp/new
root@vagrant:/home/vagrant# mount /dev/mapper/VolumeGroup_01-md0_1 /tmp/new
```

## _Задача №13_
**Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.**

```
root@vagrant:/home/vagrant# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2021-11-22 22:45:56--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22460939 (21M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz                                    100%[==================================================================================================================================================================>]  21.42M  11.2MB/s    in 1.9s

2021-11-22 22:45:58 (11.2 MB/s) - ‘/tmp/new/test.gz’ saved [22460939/22460939]
```

## _Задача №14_
**Прикрепите вывод `lsblk`.**

```
root@vagrant:/home/vagrant# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                          8:0    0   64G  0 disk
├─sda1                       8:1    0  512M  0 part  /boot/efi
├─sda2                       8:2    0    1K  0 part
└─sda5                       8:5    0 63.5G  0 part
  ├─vgvagrant-root         253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1       253:1    0  980M  0 lvm   [SWAP]
sdb                          8:16   0  2.5G  0 disk
├─sdb1                       8:17   0    2G  0 part
│ └─md0                      9:0    0    2G  0 raid1
└─sdb2                       8:18   0  511M  0 part
  └─md1                      9:1    0 1018M  0 raid0
    └─VolumeGroup_01-md0_1 253:2    0  100M  0 lvm   /tmp/new
sdc                          8:32   0  2.5G  0 disk
├─sdc1                       8:33   0    2G  0 part
│ └─md0                      9:0    0    2G  0 raid1
└─sdc2                       8:34   0  511M  0 part
  └─md1                      9:1    0 1018M  0 raid0
    └─VolumeGroup_01-md0_1 253:2    0  100M  0 lvm   /tmp/new
```
## _Задача №15_
**Протестируйте целостность файла:**

```
root@vagrant:/home/vagrant# gzip -t /tmp/new/test.gz
root@vagrant:/home/vagrant# echo $?
0
```

## _Задача №16_
**Используя `pvmove`, переместите содержимое PV с RAID0 на RAID1.**

```
root@vagrant:/home/vagrant# pvmove -n md0_1 /dev/md1 /dev/md0
  /dev/md1: Moved: 12.00%
  /dev/md1: Moved: 100.00%
```

## _Задача №17_
**Сделайте `--fail` на устройство в вашем RAID1 md.**

```
root@vagrant:/home/vagrant# mdadm --fail /dev/md0 /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md0
```

## _Задача №18_
**Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.**

```
root@vagrant:/home/vagrant# dmesg | grep md0
[ 1032.590261] md/raid1:md0: not clean -- starting background reconstruction
[ 1032.590265] md/raid1:md0: active with 2 out of 2 mirrors
[ 1032.590300] md0: detected capacity change from 0 to 2144337920
[ 1032.593286] md: resync of RAID array md0
[ 1043.183927] md: md0: resync done.
[ 2097.935522] md/raid1:md0: Disk failure on sdb1, disabling device.
               md/raid1:md0: Operation continuing on 1 devices.
```

## _Задача №19_
**Протестируйте целостность файла, несмотря на «сбойный» диск он должен продолжать быть доступен:**

```
root@vagrant:/home/vagrant# gzip -t /tmp/new/test.gz
root@vagrant:/home/vagrant# echo $?
0
```


## _Задача №20_
**Погасите тестовый хост, `vagrant destroy`.**