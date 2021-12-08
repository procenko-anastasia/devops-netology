## Домашнее задание к занятию "3.5. Файловые системы"
# Проценко Анастасии

1. Узнала о sparse (разряженных) файлах.
2. Так как `hardlink` - это ссылка на тот же самый файл и имеет тот же inode, то права будут одни и те же.
3. Сделала `vagrant destroy` на имеющийся инстанс Ubuntu. Заменила содержимое Vagrantfile, проверила на наличие 2 дисков по 2.5GB.
```
vagrant@vagrant:~$ lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
sdc                    8:32   0  2.5G  0 disk
```

4.Cоздала разделы и разбила диск на 2 раздела.
```
Device     Boot   Start     End Sectors  Size Id Type
Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 5240831 1044480  510M Linux filesystem
```
```
root@vagrant:/home/vagrant# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
└─sdb2                 8:18   0  510M  0 part
sdc                    8:32   0  2.5G  0 disk
```

5. Перенесли с помощью команды `sfdisk -d /dev/sdb|sfdisk --force /dev/sdc`
```
root@vagrant:/home/vagrant# sfdisk -d /dev/sdb|sfdisk --force /dev/sdc
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
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new GPT disklabel (GUID: 45FD3710-180D-744E-8EE0-CED57348BC0F).
/dev/sdc1: Created a new partition 1 of type 'Linux filesystem' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux filesystem' and of size 510 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: gpt
Disk identifier: 45FD3710-180D-744E-8EE0-CED57348BC0F

Device       Start     End Sectors  Size Type
/dev/sdc1     2048 4196351 4194304    2G Linux filesystem
/dev/sdc2  4196352 5240831 1044480  510M Linux filesystem

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Проверяем:
```
root@vagrant:/home/vagrant# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
└─sdb2                 8:18   0  510M  0 part
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
└─sdc2                 8:34   0  510M  0 part
```

6. Создала RAID1
```
root@vagrant:/home/vagrant# mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```

7. Собрала RAID0 на второй паре маленьких разделов.
```
root@vagrant:/home/vagrant# mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

Проверяем:
```
root@vagrant:/home/vagrant# cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid0 sdc2[1] sdb2[0]
      1040384 blocks super 1.2 512k chunks

md1 : active raid1 sdc1[1] sdb1[0]
      2094080 blocks super 1.2 [2/2] [UU]
```

8. Создала 2 независимых PV на получившихся md-устройствах:
```
root@vagrant:/home/vagrant# pvcreate /dev/md1 /dev/md0
  Physical volume "/dev/md1" successfully created.
  Physical volume "/dev/md0" successfully created.
```

9. Создайла общую volume-group на этих двух PV с помощью команды `vgcreate vg_netology /dev/md1 /dev/md0`
```
Volume group "vg_netology" successfully created
```

Проверяем `vgdisplay`:
```
 --- Volume group ---
  VG Name               vg_netology
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
  VG Size               2.98 GiB
  PE Size               4.00 MiB
  Total PE              764
  Alloc PE / Size       0 / 0
  Free  PE / Size       764 / 2.98 GiB
  VG UUID               ipuCwr-QQG7-8HNi-0QU0-r29I-MGfF-9oS0tZ
```

 10. Создала LV размером 100 Мб, указав его расположение на PV с RAID0. 
```
 root@vagrant:/home/vagrant# lvcreate -L 100M vg_netology /dev/md0
  Logical volume "lvol0" created.
  
root@vagrant:/home/vagrant# vgs
  VG          #PV #LV #SN Attr   VSize   VFree
  vg_netology   2   1   0 wz--n-   2.98g <2.89g
  vgvagrant     1   2   0 wz--n- <63.50g     0
  
root@vagrant:/home/vagrant# lvs
  LV     VG          Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lvol0  vg_netology -wi-a----- 100.00m
  root   vgvagrant   -wi-ao---- <62.54g
  swap_1 vgvagrant   -wi-ao---- 980.00m
```

 11. Создала mkfs.ext4 ФС на получившемся LV.
```
 root@vagrant:/home/vagrant# mkfs.ext4 /dev/vg_netology/lvol0
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

12. Смонтировала раздел командами:
```
root@vagrant:/home/vagrant# mkdir /tmp/new
root@vagrant:/home/vagrant# mount /dev/vg_netology/lvol0 /tmp/new
```

13. Поместила туда тестовый файл.
```
root@vagrant:/home/vagrant# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2021-12-08 20:00:42--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22764982 (22M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz.                  100%[===============================================================>]  21.71M  5.51MB/s    in 4.1s

2021-12-08 20:00:46 (5.29 MB/s) - ‘/tmp/new/test.gz’ saved [22764982/22764982]
```

14. 
```
root@vagrant:/home/vagrant# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                       8:0    0   64G  0 disk
├─sda1                    8:1    0  512M  0 part  /boot/efi
├─sda2                    8:2    0    1K  0 part
└─sda5                    8:5    0 63.5G  0 part
  ├─vgvagrant-root      253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1    253:1    0  980M  0 lvm   [SWAP]
sdb                       8:16   0  2.5G  0 disk
├─sdb1                    8:17   0    2G  0 part
│ └─md1                   9:1    0    2G  0 raid1
└─sdb2                    8:18   0  510M  0 part
  └─md0                   9:0    0 1016M  0 raid0
    └─vg_netology-lvol0 253:2    0  100M  0 lvm   /tmp/new
sdc                       8:32   0  2.5G  0 disk
├─sdc1                    8:33   0    2G  0 part
│ └─md1                   9:1    0    2G  0 raid1
└─sdc2                    8:34   0  510M  0 part
  └─md0                   9:0    0 1016M  0 raid0
    └─vg_netology-lvol0 253:2    0  100M  0 lvm   /tmp/new
```

15. Протестировали целостность файла:
```
root@vagrant:/tmp/new# gzip -t /tmp/new/test.gz
root@vagrant:/tmp/new# echo $?
0
```

16. Используя pvmove, переместила содержимое PV с RAID0 на RAID1.
```
root@vagrant:/tmp/new# pvmove /dev/md0
  /dev/md0: Moved: 12.00%
  /dev/md0: Moved: 100.00%
  
root@vagrant:/tmp/new# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                       8:0    0   64G  0 disk
├─sda1                    8:1    0  512M  0 part  /boot/efi
├─sda2                    8:2    0    1K  0 part
└─sda5                    8:5    0 63.5G  0 part
  ├─vgvagrant-root      253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1    253:1    0  980M  0 lvm   [SWAP]
sdb                       8:16   0  2.5G  0 disk
├─sdb1                    8:17   0    2G  0 part
│ └─md1                   9:1    0    2G  0 raid1
│   └─vg_netology-lvol0 253:2    0  100M  0 lvm   /tmp/new
└─sdb2                    8:18   0  510M  0 part
  └─md0                   9:0    0 1016M  0 raid0
sdc                       8:32   0  2.5G  0 disk
├─sdc1                    8:33   0    2G  0 part
│ └─md1                   9:1    0    2G  0 raid1
│   └─vg_netology-lvol0 253:2    0  100M  0 lvm   /tmp/new
└─sdc2                    8:34   0  510M  0 part
  └─md0                   9:0    0 1016M  0 raid0
```

17. --fail на устройство в RAID1 md.
```
root@vagrant:/tmp/new# mdadm /dev/md1 --fail /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md1

root@vagrant:/tmp/new# cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid0 sdc2[1] sdb2[0]
      1040384 blocks super 1.2 512k chunks

md1 : active raid1 sdc1[1] sdb1[0](F)
      2094080 blocks super 1.2 [2/1] [_U]
```

18. Подтверждение выводом команды dmseg, что RAID1 работает в деградированном состоянии:
```
root@vagrant:/tmp/new# dmesg |grep md1
[18569.750116] md/raid1:md1: not clean -- starting background reconstruction
[18569.750119] md/raid1:md1: active with 2 out of 2 mirrors
[18569.750151] md1: detected capacity change from 0 to 2144337920
[18569.754082] md: resync of RAID array md1
[18580.429107] md: md1: resync done.
[20701.108537] md/raid1:md1: Disk failure on sdb1, disabling device.
               md/raid1:md1: Operation continuing on 1 devices.
```			   

19. Протестировала целостность файла, несмотря на "сбойный" диск:
```
root@vagrant:/tmp/new# gzip -t /tmp/new/test.gz
root@vagrant:/tmp/new# echo $?
0
```

20. Погасила тестовый хост
```
PS C:\HashiCorp\vagrant conf> vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```
