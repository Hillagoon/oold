---
title: "Linux LVM 설정"
date: 2022-07-29T22:54:00-00:00
categories:
  - Linux
tags:
  - Linux
  - LVM
  - Storage
---

# LVM
## LVM 구성 순서
파티션 -> 물리 볼륨 구성 -> 물리 그룹 구성 -> 논리 볼륨 구성 -> 파일 시스템 구성   


## 파티션 정보
### 디스크 리스트 표시
```
[root@master ~]# lsblk
NAME        MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda           8:0    0  20G  0 disk 
├─sda1        8:1    0   1G  0 part /boot
└─sda2        8:2    0  19G  0 part 
  ├─cl-root 253:0    0  17G  0 lvm  /
  └─cl-swap 253:1    0   2G  0 lvm  [SWAP]
sdb           8:16   0  20G  0 disk 
└─sdb1        8:17   0  20G  0 part /os
sdc           8:32   0  20G  0 disk 
└─sdc1        8:33   0  20G  0 part /os1
sdd           8:48   0  20G  0 disk 
└─sdd1        8:49   0  20G  0 part /NFS
sde           8:64   0   1G  0 disk 
```

### 파티션 리스트 표시
```
[root@master ~]# fdisk -l
Disk /dev/sdc: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0925d3d6

Device     Boot Start      End  Sectors Size Id Type
/dev/sdc1        2048 41943039 41940992  20G 83 Linux
(...중략)
```


## 파티션 구성
###LVM 으로 구성
/dev/sde 를 LVM 저장 공간으로 만들기   
```
[root@master ~]# fdisk /dev/sde
(...중략)

Command (m for help): n <- 새로 만들기
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p <-- primary
Partition number (1-4, default 1): <-- 파티션 번호 확인
First sector (2048-2097151, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-2097151, default 2097151): +500M <-- 500MByte

Created a new partition 1 of type 'Linux' and of size 500 MiB.

Command (m for help): t <-- 디스크 타입 설정
Selected partition 1
Hex code (type L to list all codes): 8e <-- Linux LVM 으로 지정
Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): wq <-- 저장후 나가기
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

### SWAP 으로 구성 
/dev/sde 를 SWAP 메모리 저장 공간으로 만들기   
```
[root@master ~]# fdisk /dev/sde

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (2-4, default 2): <-- 파티션 번호 확인
First sector (1026048-2097151, default 1026048): 
Last sector, +sectors or +size{K,M,G,T,P} (1026048-2097151, default 2097151): +500M

Created a new partition 2 of type 'Linux' and of size 500 MiB.

Command (m for help): t <-- 디스크 타입 설정
Partition number (1,2, default 2): 2 <-- 파티션 번호 /dev/
Hex code (type L to list all codes): 82 <-- 스왑 메모리는 '82'

Changed type of partition 'Linux' to 'Linux swap / Solaris'.

Command (m for help): w <-- 
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

### 결과확인
아래와 같이 sde1,2 로 파티션이 나누어 분할된 것을 확인 할 수 있다.   
```
[root@master ~]# fdisk -l /dev/sde
Disk /dev/sde: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdf4044fe

Device     Boot   Start     End Sectors  Size Id Type
/dev/sde1          2048 1026047 1024000  500M 8e Linux LVM
/dev/sde2       1026048 2050047 1024000  500M 82 Linux swap / Solaris
```


### 파티션 타입 코드
fdisk 명령어에서 타입 설정 모드에서  L - List 로 확인 할 수 있다.   
```
Command (m for help): t
Hex code (type L to list all codes): L

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden or  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi ea  Rufus alignment
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         eb  BeOS fs        
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ee  GPT            
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        ef  EFI (FAT-12/16/
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f0  Linux/PA-RISC b
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f1  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f4  SpeedStor      
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      f2  DOS secondary  
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fb  VMware VMFS    
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fc  VMware VMKCORE 
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fd  Linux raid auto
1c  Hidden W95 FAT3 75  PC/IX           bc  Acronis FAT32 L fe  LANstep        
1e  Hidden W95 FAT1 80  Old Minix       be  Solaris boot    ff  BBT            
```

* 자주쓰는 타입   
82  Linux swap  
8e  Linux LVM   

## 물리 볼륨 구성
pvcreate 로 물리 볼륨 구성   
vgcreate 로 물리 볼륨 그룹 구성   

/dev/sde1을 물리 볼륨화(pv)   
example 이라는 물리 볼륩 그룹(vg)에 물리 볼륨(pv) 할당   
```
[root@master ~]# pvcreate /dev/sde1 
  Physical volume "/dev/sde1" successfully created.
[root@master ~]# 
[root@master ~]# pvs
  PV         VG      Fmt  Attr PSize   PFree  
  /dev/sda2  cl      lvm2 a--  <19.00g      0 
  /dev/sde1  example lvm2 a--  496.00m 496.00m
[root@master ~]# vgcreate example /dev/sde1
[root@master ~]# 
  Volume group "example" successfully created
[root@master ~]# 
[root@master ~]# vgs
  VG      #PV #LV #SN Attr   VSize   VFree  
  cl        1   2   0 wz--n- <19.00g      0 
  example   1   0   0 wz--n- 496.00m 496.00m
```

## 논리 볼륨 구성
lvcreate 로 논리 볼륨 구성   
-l (length) 에 100%FREE 를 지정하면 남은 공간 100%를 지정 할 수 있다.   
-n (name) 논리 볼륨 이름 지정   

지금까지의 구조   
vgroup (lv)   
ㄴexample (vg)   
  ㄴ/dev/sde1 (pv)   
  
```
[root@master ~]# lvcreate -l 100%FREE -n vgroup example
  Logical volume "vgroup" created.
[root@master ~]# 
[root@master ~]# lvs
  LV     VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   cl      -wi-ao---- <17.00g                                                    
  swap   cl      -wi-ao----   2.00g                                                    
  vgroup example -wi-a----- 496.00m                                                    
```

## 파일 시스템 구성
mkfs.* 명령어로 각 파일 시스템 종류에 따라 사용 가능 하다.   
아래 참조   
```
[root@master ~]# mkfs.
mkfs.cramfs  mkfs.ext3    mkfs.fat     mkfs.msdos   mkfs.xfs     
mkfs.ext2    mkfs.ext4    mkfs.minix   mkfs.vfat   
[root@master ~]# 
[root@master ~]# mkfs.xfs /dev/example/vgroup 
meta-data=/dev/example/vgroup    isize=512    agcount=4, agsize=31744 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=126976, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
mkfs.xfs /dev/example/vgroup   
디렉토리의 구조가 /dev/example*(vg)/vgroup*(lv)   

## 마운트 테스트
루트(/) 아래에 test디렉토리 생성후 마운트를 해본다.
아래와 같이 마운트 후 인식이 잘 되는 것을 확인 할 수 있다.
```
[root@master ~]# mkdir /test
[root@master ~]# mount -t xfs /dev/example/vgroup /test
[root@master ~]# 
[root@master ~]# 
[root@master ~]# df -h
Filesystem                  Size  Used Avail Use% Mounted on
(...중략)
/dev/mapper/example-vgroup  491M   29M  463M   6% /test
[root@master ~]# 
[root@master ~]# lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
(...중략)
├─sde1               8:65   0  500M  0 part 
│ └─example-vgroup 253:2    0  496M  0 lvm  /test
└─sde2               8:66   0  500M  0 part 
```

## 자동 마운트 설정   
장치ID 확인   
```
[root@master ~]# blkid
/dev/sda1: UUID="f7e8b2cc-f587-49d9-a280-02483f4f082e" TYPE="ext4" PARTUUID="a7402d34-01"
/dev/sda2: UUID="3QPjO2-q2RV-RgFi-7Nhm-DOGy-ZnOY-fNBULr" TYPE="LVM2_member" PARTUUID="a7402d34-02"
/dev/sdb1: UUID="54bc1f42-ab3d-4048-9c0b-1c5655dc583e" TYPE="ext4" PARTUUID="415fb890-01"
/dev/sdc1: UUID="f4ca6e6a-68ea-4016-bb63-dbd0c308336e" TYPE="ext4" PARTUUID="0925d3d6-01"
/dev/sdd1: UUID="ae4a229c-9ff3-4e22-9633-9faa993bb64f" TYPE="ext4" PARTUUID="2f58189f-01"
/dev/sde1: UUID="wiP3O2-wnOV-TEYU-uqHj-MOfz-HOb0-Wo1L96" TYPE="LVM2_member" PARTUUID="df4044fe-01"
/dev/mapper/cl-root: UUID="03a7cd11-90c9-4bda-bc52-f77b5c2667d2" TYPE="xfs"
/dev/mapper/cl-swap: UUID="194dd349-5a88-4a70-8f70-12d13b0fd415" TYPE="swap"
/dev/mapper/example-vgroup: UUID="b7b0931f-df6a-41ab-a309-6dc34a467982" TYPE="xfs"
/dev/sde2: PARTUUID="df4044fe-02"
```

/etc/fstab 에 장치 정보 추가   
형식은 아래 더블쿼테이션("") 부분 정도만 기억해도 충분하다.   
UUID="id" "mount_point" "partition_type" defaults 0 0   

```
[root@master ~]# vi /etc/fstab
# /etc/fstab
# Created by anaconda on Fri Feb 19 18:29:02 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=f7e8b2cc-f587-49d9-a280-02483f4f082e /boot                   ext4    defaults        1 2
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
UUID=54bc1f42-ab3d-4048-9c0b-1c5655dc583e /os ext4      defaults        0 0
UUID=f4ca6e6a-68ea-4016-bb63-dbd0c308336e /os1 ext4     defaults        0 0
UUID=ae4a229c-9ff3-4e22-9633-9faa993bb64f /NFS  ext4    defaults        0 0
UUID=b7b0931f-df6a-41ab-a309-6dc34a467982 /test  xfs    defaults        0 0
```

fstab 설정값 읽어 들이기   
```
[root@master ~]# mount -a
[root@master ~]# 
```
* 확인 작업 스킵 후 재부팅했을때 무한 부팅현상이 생길격우. 그럴경우 침착하게 리커버리모드로 진입해서 해당 설정을 제거하면 부팅가능함.   

** 설정내용에 문제있을 경우 아래와 같이 출력됨   
```
[root@master ~]# mount -a
mount: /test: can't find UUID=ab7b0931f-df6a-41ab-a309-6dc34a467982.
```

## 마운트 확인   
```
[root@master ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             3.1G     0  3.1G   0% /dev
tmpfs                3.1G     0  3.1G   0% /dev/shm
tmpfs                3.1G   10M  3.1G   1% /run
tmpfs                3.1G     0  3.1G   0% /sys/fs/cgroup
/dev/mapper/cl-root   17G   16G  2.0G  89% /
/dev/sda1            976M  194M  715M  22% /boot
/dev/sdd1             20G   45M   19G   1% /NFS
/dev/sdc1             20G   45M   19G   1% /os1
tmpfs                628M  1.2M  627M   1% /run/user/42
tmpfs                628M  4.6M  623M   1% /run/user/0
/dev/mapper/example-vgroup  491M   29M  463M   6% /test
```

## 스왑 설정
사전에 파티션을 만든 /dev/sde2 를 스왑 메모리로 사용하도록 아래와 같이 진행
swapon 전/후의 Swap 메모리 total이 증가된 것을 확인 할 수 있었다.
```
[root@master ~]# mkswap /dev/sde2
Setting up swapspace version 1, size = 500 MiB (524283904 bytes)
no label, UUID=8bdc6289-6b5e-4941-8d3b-007523ed1464
[root@master ~]# 
[root@master ~]# 
[root@master ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           6272        4410         487          12        1374        1596
Swap:          2047          17        2030
[root@master ~]# 
[root@master ~]# 
[root@master ~]# swapon /dev/sde2
[root@master ~]# 
[root@master ~]# 
[root@master ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           6272        4410         486          12        1374        1595
Swap:          2547          17        2530
```

## 자동 마운트 설정
이전 LVM 스텝에서 진행한 것과 동일하다.   
```
[root@master ~]# vi /etc/fstab
# /etc/fstab
# Created by anaconda on Fri Feb 19 18:29:02 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=f7e8b2cc-f587-49d9-a280-02483f4f082e /boot                   ext4    defaults        1 2
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
UUID=54bc1f42-ab3d-4048-9c0b-1c5655dc583e /os ext4      defaults        0 0
UUID=f4ca6e6a-68ea-4016-bb63-dbd0c308336e /os1 ext4     defaults        0 0
UUID=ae4a229c-9ff3-4e22-9633-9faa993bb64f /NFS  ext4    defaults        0 0
UUID=b7b0931f-df6a-41ab-a309-6dc34a467982 /test  xfs    defaults        0 0
UUID=8bdc6289-6b5e-4941-8d3b-007523ed1464       swap    swap    defaults        0 0
```

## 마운트 확인   
```
[root@master ~]# mount -a
[root@master ~]# 
[root@master ~]# lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                  8:0    0   20G  0 disk 
├─sda1               8:1    0    1G  0 part /boot
└─sda2               8:2    0   19G  0 part 
  ├─cl-root        253:0    0   17G  0 lvm  /
  └─cl-swap        253:1    0    2G  0 lvm  [SWAP]
sdb                  8:16   0   20G  0 disk 
└─sdb1               8:17   0   20G  0 part /os
sdc                  8:32   0   20G  0 disk 
└─sdc1               8:33   0   20G  0 part /os1
sdd                  8:48   0   20G  0 disk 
└─sdd1               8:49   0   20G  0 part /NFS
sde                  8:64   0    1G  0 disk 
├─sde1               8:65   0  500M  0 part 
│ └─example-vgroup 253:2    0  496M  0 lvm  /test
└─sde2               8:66   0  500M  0 part [SWAP]
```


## 용량 확장
### 시나리오
시스템운용을 하던중 디스크 용량이 부족해졌다는 시나리오로   
새로운 하드디스크를 추가하여 확장하려고 한다.   

기존 그룹에 새로운 하드디스크 /dev/sdf1 을 추가해 보자.

### 볼륨 구성
```
[root@master ~]# pvcreate /dev/sdf1 
  Physical volume "/dev/sdf1" successfully created.
[root@master ~]# vgextend example /dev/sdf1
  Volume group "example" successfully extended
[root@master ~]# lvextend /dev/
/dev/cl/root         /dev/cl/swap         /dev/example/vgroup
[root@master ~]# lvextend /dev/example/vgroup -l +100%FREE
  Size of logical volume example/vgroup changed from 496.00 MiB (124 extents) to 1.48 GiB (379 extents).
  Logical volume example/vgroup successfully resized.
[root@master ~]# 
[root@master ~]# 
[root@master ~]# lvs
  LV     VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   cl      -wi-ao---- <17.00g                                                    
  swap   cl      -wi-ao----   2.00g                                                    
  vgroup example -wi-ao----   1.48g                                                    
[root@master ~]# 
```

df -h 명령어에 /dev/sdf1 이 보이지 않지만   
lsblk에는 인식하고 있다.   
df 명령어는 마운트된 장치에 대해서만 정보를 출력하기 때문이다.   

```
[root@master ~]# df -h
Filesystem                  Size  Used Avail Use% Mounted on
devtmpfs                    3.1G     0  3.1G   0% /dev
tmpfs                       3.1G     0  3.1G   0% /dev/shm
tmpfs                       3.1G   11M  3.1G   1% /run
tmpfs                       3.1G     0  3.1G   0% /sys/fs/cgroup
/dev/mapper/cl-root          17G   16G  2.0G  89% /
/dev/mapper/example-vgroup  491M   29M  463M   6% /test
/dev/sdc1                    20G  5.5G   14G  30% /os1
/dev/sdb1                    20G  5.6G   13G  30% /os
/dev/sdd1                    20G  5.8G   13G  31% /NFS
/dev/sda1                   976M  194M  715M  22% /boot
tmpfs                       628M  1.2M  627M   1% /run/user/42
tmpfs                       628M  4.6M  623M   1% /run/user/0
[root@master ~]# 
[root@master ~]# lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
(...중략)
sde                  8:64   0    1G  0 disk 
├─sde1               8:65   0  500M  0 part 
│ └─example-vgroup 253:2    0  1.5G  0 lvm  /test
└─sde2               8:66   0  500M  0 part [SWAP]
sdf                  8:80   0    1G  0 disk 
└─sdf1               8:81   0 1023M  0 part 
  └─example-vgroup 253:2    0  1.5G  0 lvm  /test
```

### 파일시스템 확장
```
[root@master ~]# xfs_growfs /dev/example/vgroup 
meta-data=/dev/mapper/example-vgroup isize=512    agcount=4, agsize=31744 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=126976, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 126976 to 388096
[root@master ~]# 
[root@master ~]# df -h
Filesystem                  Size  Used Avail Use% Mounted on
devtmpfs                    3.1G     0  3.1G   0% /dev
tmpfs                       3.1G     0  3.1G   0% /dev/shm
tmpfs                       3.1G   11M  3.1G   1% /run
tmpfs                       3.1G     0  3.1G   0% /sys/fs/cgroup
/dev/mapper/cl-root          17G   16G  2.0G  89% /
/dev/mapper/example-vgroup  1.5G   37M  1.5G   3% /test
/dev/sdc1                    20G  5.5G   14G  30% /os1
/dev/sdb1                    20G  5.6G   13G  30% /os
/dev/sdd1                    20G  5.8G   13G  31% /NFS
/dev/sda1                   976M  194M  715M  22% /boot
tmpfs                       628M  1.2M  627M   1% /run/user/42
tmpfs                       628M  4.6M  623M   1% /run/user/0
```
파일 시스템을 새로 만들때는 mkfs.* 명령어를 사용했는데
확장할때는 약간 다르다.
xfs 파일시스템의 경우는 xfs_growfs 명령어로 확장을 시도하고,
ext 시리즈의 경우는 resize2fs 명령어로 확장을 시도하면 된다.


### 볼륨 구조 확인
작업 전
vgroup (lv)   
ㄴexample (vg)   
  ㄴ/dev/sde1 (pv)   
  
작업 후
vgroup (lv)   
ㄴexample (vg)   
  ㄴ/dev/sde1 (pv)   
