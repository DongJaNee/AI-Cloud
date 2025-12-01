## Storage Scale 
### 1. Enviroment 
- Hyper-V (RedHat Linux)
- node1 (단일 Cluster)

### 2. Cluster 
- Packages : rpm-qa | grep -i gpfs
- Hostname : hostnamectl set-hostname [Name]
- cat /etc/hosts Define
- SSH Key : ssh-keygen -t rsa
- Key Copy : ssh-copy-id [Name] 

```
cat node
node01.gpfs.mgt:quorm            //node01.gpfs.mgt는 hostname

mmcrcluster -N ./node -C clusterA            //clusterA라는 이름으로 Cluster 생성 

mmlscluster            // 생성된 cluster 확인
```

<img width="993" height="112" alt="image" src="https://github.com/user-attachments/assets/3f5e29f0-0f23-458d-84ba-9326b928a0fa" />

#### 2-1) license 
```
mmchlicense server --accept -N node01     //license 허용
```

#### 2-2) Daemon Start
```
mmstartup     // Daemon 시작
fdisk -l       // Disk 용량 확인
```
```
<b>Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors</b>     // OS Disk 사용 불가능 
Disk model: Virtual Disk    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0xb43659c3

<b>Device    </b> <b>Boot</b> <b>    Start</b> <b>      End</b> <b>  Sectors</b> <b> Size</b> <b>Id</b> <b>Type</b>
/dev/sda1  *         2048 102762495 102760448   49G 83 Linux
/dev/sda2       102762496 104857599   2095104 1023M 82 Linux swap / Solaris


<b>Disk /dev/sdb: 4 GiB, 4294967296 bytes, 8388608 sectors</b>     // 사용 가능 Disk 
Disk model: Virtual Disk    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes


<b>Disk /dev/sdc: 4 GiB, 4294967296 bytes, 8388608 sectors</b>      // 사용 가능 Disk 
Disk model: Virtual Disk    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

### 3. NSD (Network Shared Disk)
#### 3-1) Create NSD 
```
nano nsd      
    %nsd:
     device=/dev/sdb
     nsd=nsd1
     servers=node01
     usage=dataAndMetadata
     failuserGroup=1
     pool=system

    %nsd:
     device=/dev/sdc
     nsd=nsd2
     servers=node01
     usage=dataOnly
     failuserGroup=1
     pool=data1
```

```
mmcrnsd -F ./nsd       //nsd 생성
mmmount -a       // 전체 mount
mmlsnsd        // nsd 리스트

 File system   Disk name       NSD servers                                    
------------------------------------------------------------------------------
 (free disk)   nsd1            node01.gpfs.mgt          
 (free disk)   nsd2            node01.gpfs.mgt

```

### 4. File System
#### 4-1) Create File System
```
mmcrfs gpfs -F ./nsd -A yes -B 4M -T /gpfs

 File system   Disk name       NSD servers                                    
------------------------------------------------------------------------------
 gpfs          nsd1            node01.gpfs.mgt          
 gpfs          nsd2            node01.gpfs.mgt


mmmount gpfs -a

df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           450M     0  450M   0% /dev/shm
tmpfs           180M  5.8M  175M   4% /run
/dev/sda1        49G   19G   31G  38% /
tmpfs            90M  100K   90M   1% /run/user/1000
tmpfs            90M   36K   90M   1% /run/user/0
gpfs            8.0G  1.7G  6.4G  21% /gpfs

cd /gpfs
touch test
echo &quot;111&quot; &gt; test

```

```
mmdf gpfs --block-size auto

disk                disk size  failure holds    holds                 free                free
name                             group metadata data        in full blocks        in fragments
--------------- ------------- -------- -------- ----- -------------------- -------------------
Disks in storage pool: system (Maximum disk size allowed is 42.74 GB)
nsd1                       4G        1 예      예          2.391G ( 60%)        11.62M ( 0%) 
                -------------                         -------------------- -------------------
(pool total)               4G                                2.391G ( 60%)        11.62M ( 0%)

Disks in storage pool: data1 (Maximum disk size allowed is 42.74 GB)
nsd2                       4G        1 아니요 예           3.93G ( 98%)        7.867M ( 0%) 
                -------------                         -------------------- -------------------
(pool total)               4G                                 3.93G ( 98%)        7.867M ( 0%)

                =============                         ==================== ===================
(data)                     8G                                 6.32G ( 79%)        19.48M ( 0%)
(metadata)                 4G                                2.391G ( 60%)        11.62M ( 0%)
                =============                         ==================== ===================
(total)                    8G                                 6.32G ( 79%)        19.48M ( 0%)

Inode Information
-----------------
Number of used inodes:            4039
Number of free inodes:          130105
Number of allocated inodes:     134144
Maximum number of inodes:       134144
```

### 5. Node
#### 5-1) Assign Role to Node 
```
mmchnode --manager -N node01         //node01의 역할을 manager로 변경 

GPFS cluster information
========================
  GPFS cluster name:         node01.gpfs.mgt
  GPFS cluster id:           11272164317478775499
  GPFS UID domain:           node01.gpfs.mgt
  Remote shell command:      /usr/bin/ssh
  Remote file copy command:  /usr/bin/scp
  Repository type:           CCR

 Node  Daemon node name  IP address     Admin node name  Designation
---------------------------------------------------------------------
   1   node01.gpfs.mgt   192.168.45.20  node01.gpfs.mgt  quorum-manager
```

#### 5-2) Nodeclass
```
mmcrnodeclass nsd_server -N node01

[root@qwe gpfs]# mmlsnodeclass
Node Class Name       Members
--------------------- -----------------------------------------------------------
nsdserver             node01.gpfs.mgt
```

### 6. Configuration
```
mmchconfig maxFilesToCache=128k -N nsdserver

mmshutdown
mmstartup

maxFilesToCache 64k  
maxFilesToCache 128k [nsdserver]
```

### 7. Fileset
#### 7-1) Create Fileset 
```
mmcrfileset gpfs fset1 --inode-space new        //독립된 fest1 fileset 생성 

mmlsfileset gpfs        //gpfs Fileset확인

Filesets in file system &apos;gpfs&apos;:
Name                     Status    Path                                    
root                     Linked    /gpfs                                   
fset1                    Unlinked  --
```

```
mmlinkfileset gpfs fset1 -J /gpfs/fest1        //Fileset을 경로에 연결(Link) 

mmlsfileset gpfs

Filesets in file system &apos;gpfs&apos;:
Name                     Status    Path                                    
root                     Linked    /gpfs                                   
fset1                    Linked    /gpfs/fest1
```

### 8. Snapshot
```
mmlssnapshot gpfs -j fest1

mmcrsnapshot gpfs fest1            // Snapshot 생성

mmlssnapshot gpfs        //gpfs에 있는 snapshot list 확인

Snapshots in file system gpfs:
Directory                SnapId    Status  Created                   ExpirationTime            Fileset
fest1                    1         Valid   Thu Nov 27 16:39:15 2025  Thu Nov 27 16:39:15 2025
```

```
mmhealth cluster show

Node name:      node01.gpfs.mgt
Node status:    <font color="#2AA1B3"><b>TIPS</b></font>
Status Change:  17 min. ago

Component      Status        Status Change     Reasons &amp; Notices
----------------------------------------------------------------------------------------------
GPFS           <font color="#2AA1B3"><b>TIPS</b></font>          17 min. ago       callhome_not_enabled, swapped_warn
NETWORK        <font color="#26A269"><b>HEALTHY</b></font>       1 hour ago        -
FILESYSTEM     <font color="#26A269"><b>HEALTHY</b></font>       17 min. ago       -
DISK           <font color="#26A269"><b>HEALTHY</b></font>       17 min. ago       -
FILESYSMGR     <font color="#26A269"><b>HEALTHY</b></font>       17 min. ago       -
[root@qwe fest1]# mmhealth node show gpfs

Node name:      node01.gpfs.mgt

Component     Status        Status Change     Reasons &amp; Notices
---------------------------------------------------------------------------------------------
GPFS          <font color="#2AA1B3"><b>TIPS</b></font>          18 min. ago       callhome_not_enabled, swapped_warn


Event                 Parameter  Severity    Active Since         Event Message
------------------------------------------------------------------------------------------------------------------------
callhome_not_enabled  GPFS       TIP         1 hour ago           Call home is not installed, configured, or enabled.
swapped_warn          GPFS       TIP         1 hour ago           Swap memory usage exceeds the expected threshold value
```

```
mmhealth node show gpfs


Node name:      node01.gpfs.mgt

Component     Status        Status Change     Reasons &amp; Notices
--------------------------------------------------------------------------------------------------------------------
GPFS          <font color="#2AA1B3"><b>TIPS</b></font>          Now               callhome_not_enabled, gpfs_maxstatcache_low, swapped_warn


Event                  Parameter  Severity    Active Since         Event Message
----------------------------------------------------------------------------------------------------------------------------
callhome_not_enabled   GPFS       TIP         Now                  Call home is not installed, configured, or enabled.
gpfs_maxstatcache_low  GPFS       TIP         Now                  The GPFS maxStatCache is smaller than the maxFilesToCache
                                                                   setting.
swapped_warn           GPFS       TIP         Now                  Swap memory usage exceeds the expected threshold value
                                                                   50000.
```


