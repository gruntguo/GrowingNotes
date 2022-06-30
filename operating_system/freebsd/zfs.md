# The Z File System (ZFS)

SUN 开发, 开源在 [OpenZFS](https://openzfs.org/wiki/Main_Page) Project;

ZFS 三个主要目标:
  - Data integrity, 数据完整性  
      数据有checksum, ZFS 会自动修复error ,when  ditto-, mirror-, or parity-blocks are available

  - Pooled storage, 池化  
      物理硬盘加入到 pool 中, 新的空间可以被所有的卷和文件系统使用
      
  - Performence, 性能  
      caching 用来提高性能;  
      ARC(Adaptive Replacement Cache) is advanced memory-based read cache.  
      L2ARC second level disk-based read cache, 可使用ssd.  
      ZIL is a disk-based synchronous write cache, 同步时使用, 普通cp不用.
      
所有的[feature](https://docs.freebsd.org/en/books/handbook/zfs/#zfs-term)


## ZFS 的特点

###  pool
###  vdev
	可以是磁盘(/dev/ada0),   
	文件(通常用来test),   
	mirror, 由多块盘组成, 同时写  
	RAID-Z,  3个level RAID-Z1,Z2,Z3  
	Spare, pseudo-vdev  
	Log, ZFS Intent Log (ZIL)   
	Cache, 使用L2ARC  
	Transaction Group (TXG),  block 操作的原子,  
	    三个状态: Open, Quiescing, Syncing  
	Adaptive Replacement Cache (ARC),  对比LRU, 包含4个队列  
	    - Most Recently Used (MRU)  
        - Most Frequently Used (MFU)  
		以上两种分别包含 ghost 队列, 用来跟踪被驱逐的object, 阻止他们再次加入队列,  
		可以提高命中率  
	L2ARC, 相较于 ARC, 是second level cache, 经常使用cache vdev, 并使用ssd  
	    可以加速deduplication,   
	ZIL, 使用 ssd 加速同步写, 异步写不使用ZIL  
	Dataset, 基本单位,   
	scrub, 类似fsck  
	Quota, Dataset Quota, Reference Quota, User Quota  
	Dataset Reservation, Reference Reservation  
	Resilver, 换盘时的流程
	

## Quick start

    ```
    zfs_enable="YES"
    
    service zfs start
    ```
    
###  single disk pool

    ```
    # zpool create example /dev/da0
    
    create dataset on this pool with compression enabled

    # zfs create example/compressed
    # zfs set compression=gzip example/compressed

    # zfs set compression=off example/compressed
    
    # zfs mount example/compressed
    # zfs umount example/compressed
    
    # zfs create example/data
    # zfs set copies=2 example/data

    删除
    # zfs destroy example/compressed
    # zfs destroy example/data
    # zpool destroy example
    
    ```
    
### raid-z

    raid-z 比 mirrored pools 提供楼更多的space
    
    ```
    # zpool create storage raidz da0 da1 da2
    
    ```
    
    Note:
      推荐使用 3-9 个盘, 如果有10个以上的盘, 考虑分成更多的raid-z
      
    ```
    # zfs create storage/home
    
    # zfs set copies=2 storage/home
    # zfs set compression=gzip storage/home

    # cp -rp /home/* /storage/home
    # rm -rf /home /usr/home
    # ln -s /storage/home /home
    # ln -s /storage/home /usr/home

    # zfs snapshot storage/home@08-30-08
    # zfs rollback storage/home@08-30-08
    
    # ls /storage/home/.zfs/snapshot
    # zfs destroy storage/home@08-30-08
    
    # zfs set mountpoint=/home storage/home
    ```
### data verification
    
    checksum

    
    ```
    verify data checksums(called scrubbing)
    # zpool scrub storage
    ```
    
    其他请参照zfs(8) , zpool(8)
    
    
    
## pool管理

    ```
    create
    
    # zpool create mypool mirror /dev/ada1 /dev/ada2
    # zpool status
    
    add 
    第一种方式, 直接添加一个盘到vdev: 
    zpool attach mypool ada0p3 ada1p3
    第二种方式, 添加一个mirror;
    # zpool add mypool mirror ada2p3 ada3p3
    # gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada2
    bootcode written to ada2
    # gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada3
    bootcode written to ada3    
    ```
    
### clear error

    zpool clear mypool
    
### replacing

    ```
    # zpool replace mypool ada1p3 ada2p3
    ```
    
### scrubbing a Pool
    
    至少一个月一次
    
    
    
    ```
    # zpool scrub mypool
    cancel
    # zpool scrub -s mypool
    ```

### self-healing
    
    ```
    # zpool create healer mirror /dev/ada0 /dev/ada1
    ```
### importing and exporting

    ```
    list
    # zpool import
    fy
    # zpool import -o altroot=/mnt mypool
    ```
    
### upgrade

    ```
        # zpool upgrade
        
        查看新版本有哪些特性
        # zpool upgrade -v
    ```

### history

    ```
    zpool history 
    
    zpool history -i
    zpool history -l
    ```
### performance monitoring

    

    ```
        # zpool iostat
                    capacity     operations     bandwidth 
        pool        alloc   free   read  write   read  write
        ----------  -----  -----  -----  -----  -----  -----
        zroot       10.0G  7.47G      1      2  20.7K  29.4K
        
        operations : per sec
        
        持续观测, 2秒刷一次
        # zpool iostat 2
        
        Per device info
        # zpool iostat -v
        
    ```

## zfs管理

### datasets

    ZFS 不像通常的磁盘管理, 不需要提前分配空间. ZFS 可以随时创建新的 fs;  
    每个dataset 可以有自己的特性, 例如: 压缩, 去重, 缓存, 配额,等等.  
    
    ```
    # zfs list
    # zfs create -o  compress=lz4 mypool/usr/mydataset
    # zfs destroy mypool/usr/mydataset
    ```
    
    destroy 是异步的, 
     `zpool get freeing poolname` 查看 freeing property, 
     如果有子dataset 或者 snapshots, 父级不能被删掉, 使用 `-r`递归删除, 
     使用 `-n -v`列出删除的datasets
     
### volume
    
    volume 是特殊的 dataset, 作为一个block 存储设备 `/dev/zvol/poolname/dataset`,
    可以使用volume 作为其他fs, vm的disk, network fs (iscsi, hast)等等；  
    
    ```
    # zfs create -V 250m -o compression=on tank/fat32
    # zfs list tank
    NAME USED AVAIL REFER MOUNTPOINT
    tank 258M  670M   31K /tank
    # newfs_msdos -F32 /dev/zvol/tank/fat32
    # mount -t msdosfs /dev/zvol/tank/fat32 /mnt
    # df -h /mnt | grep fat32
    Filesystem           Size Used Avail Capacity Mounted on
    /dev/zvol/tank/fat32 249M  24k  249M     0%   /mnt
    # mount | grep fat32
    /dev/zvol/tank/fat32 on /mnt (msdosfs, local)


    ```

### set dataset properties

    ```
    zfs set property=value dataset
    ```
    
    `zfs get` 获取可能的属性  
    `zfs inherit` 恢复为继承的value  
    `(:)` 用来设置自定义value  
    `zfs inherit -r ` 用来remove 自定义属性  
    
    ```
    # zfs get sharenfs mypool/usr/home
    NAME             PROPERTY  VALUE    SOURCE
    mypool/usr/home  sharenfs  on       local
    # zfs get sharesmb mypool/usr/home
    NAME             PROPERTY  VALUE    SOURCE
    mypool/usr/home  sharesmb  off      local
    
    #  zfs set sharenfs=on mypool/usr/home
    #  zfs set sharenfs="-alldirs,-maproot=root,-network=192.168.1.0/24" mypool/usr/home
    ```

### snapshots

    ZFS 使用COW(copy-on-write)技术, 保留老版本的data;
    snapshots 作用于整个dataset, 而不是文件或文件夹;
    snapshots 会将所有东西作为复本, 包括fs 属性, fs, 文件, 目录, 权限, 等等.
    
    ```
    # zfs snapshot -r mypool@my_recursive_snapshot
    # zfs list -t snapshot
    ```
    
    
    ```
    # zfs list -rt all mypool/usr/home
    NAME                                    USED  AVAIL  REFER  MOUNTPOINT
    mypool/usr/home                         184K  93.2G   184K  /usr/home
    mypool/usr/home@my_recursive_snapshot      0      -   184K  -
    
    # cp /etc/passwd /var/tmp
	# zfs snapshot mypool/var/tmp@after_cp
	# zfs list -rt all mypool/var/tmp
	NAME                                   USED  AVAIL  REFER  MOUNTPOINT
	mypool/var/tmp                         206K  93.2G   118K  /var/tmp
	mypool/var/tmp@my_recursive_snapshot    88K      -   152K  -
	mypool/var/tmp@after_cp                   0      -   118K  -
    
    ```
    
### comparing

    ```
    zfs diff 
    # zfs diff mypool/var/tmp@my_recursive_snapshot
    M       /var/tmp/
    +       /var/tmp/passwd
    
    # zfs diff mypool/var/tmp@my_recursive_snapshot mypool/var/tmp@diff_snapshot
    M       /var/tmp/
    +       /var/tmp/passwd
    +       /var/tmp/passwd.copy
    
    # zfs diff mypool/var/tmp@my_recursive_snapshot mypool/var/tmp@after_cp
    M       /var/tmp/
    +       /var/tmp/passwd
    
    ```
    
### rollback

    `zfs rollback snapshotname` 可以用来rollback;  
    如果有过多的改变, 会花费长的时间, 在此过程中, dataset 仍保持一致性,
    此过程有点像数据库遵守 ACID 的原则.
    
    ```
    # zfs rollback mypool/var/tmp@diff_snapshot
    ```
    
    
    快照在 `.zfs/snapshots/snapshotname` 中,
    可以从中恢复一个单独的文件.
    
    ```
    # zfs get snapdir mypool/var/tmp
    NAME            PROPERTY  VALUE    SOURCE
    mypool/var/tmp  snapdir   hidden   default
    
    # zfs set snapdir=visible mypool/var/tmp
    
    # ls /var/tmp/.zfs/snapshot
    after_cp                my_recursive_snapshot
    # ls /var/tmp/.zfs/snapshot/after_cp
    passwd          vi.recover
    # cp /var/tmp/.zfs/snapshot/after_cp/passwd /var/tmp
    
    ```
    
### clone

    使用 `zfs clone` , 创建一个可写和可挂载的dataset, 一旦有子clone,父不可以删除
    
    使用 `zfs promote`, 反转一个clone的 父/子 关系;
    
    因为快照不可写,  一般可以先使用clone,然后做一些操作, 在达到想要的结果后,
    提升`promote` 这个clone 的dataset, 然后将父给删除掉.
    
    ```
    # zfs clone camino/home/joe@backup camino/home/joenew
	# ls /usr/home/joe*
	/usr/home/joe:
	backup.txz     plans.txt
	
	/usr/home/joenew:
	backup.txz     plans.txt
	# df -h /usr/home
	Filesystem          Size    Used   Avail Capacity  Mounted on
	usr/home/joe        1.3G     31k    1.3G     0%    /usr/home/joe
	usr/home/joenew     1.3G     31k    1.3G     0%    /usr/home/joenew
    ```
    
    
    ```
    # zfs get origin camino/home/joenew
    NAME                  PROPERTY  VALUE                     SOURCE
    camino/home/joenew    origin    camino/home/joe@backup    -
    # zfs promote camino/home/joenew
    # zfs get origin camino/home/joenew
    NAME                  PROPERTY  VALUE   SOURCE
    camino/home/joenew    origin    -       -
    ```
    
### replication

    ```
    send 
    # zfs send mypool@backup1 > /backup/backup1
     
    incremental send, -i
    # zfs send -v -i mypool@replica1 mypool@replica2 | zfs receive /backup/mypool
    ssh
    % zfs send -R mypool/home@monday | ssh someuser@backuphost zfs recv -dvu recvpool/backup
    ```
    
    
### quota

    quota:
    Reference Quotas: 计算
    
    ```
    # zfs set quota=10G storage/home/bob
    
    # zfs set refquota=10G storage/home/bob
    
    remove
    # zfs set quota=none storage/home/bob
    
    # zfs set userquota@joe=50G
    # zfs set userquota@joe=none
    
    User quota properties are not displayed by zfs get all.
    一旦有权限的话就可以查看和设置所有人的quota
    
    
    # zfs set groupquota@firstgroup=50G
    
    ```

### reservation

    

    ```
    # zfs set reservation=10G storage/home/bob
    # zfs set reservation=none storage/home/bob
    
    # zfs get reservation storage/home/bob
    # zfs get refreservation storage/home/bob
    ```
    
### compression
    
    Tranparent compression
    
    ```
    # zfs get used,compressratio,compression,logicalused mypool/compressed_dataset
    ```
    algorithm:
    
    - LZ4
    - Zstd(Zstandard), 19 levels , default is zstd-3
    
    Adaptive Replacement Cache (ARC) caches data in RAM
    
### Deduplication
    
     使用 checksum 寻找重复的block, 
     
     ```
     # zfs set dedup=on pool
     
     # zpool list
     NAME  SIZE ALLOC  FREE   CKPOINT  EXPANDSZ   FRAG   CAP   DEDUP   HEALTH   ALTROOT
     pool 2.84G 2.19M 2.83G         -         -     0%    0%   1.00x   ONLINE   -
     ```

    deduplication 只影响新写入的data;
    1.0x 代表还没开始
    
    不是所有情况下, dedup都会有效减少空间
    
    ```
    # zdb -S pool
		Simulated DDT histogram:
		
		bucket              allocated                       referenced
		______   ______________________________   ______________________________
		refcnt   blocks   LSIZE   PSIZE   DSIZE   blocks   LSIZE   PSIZE   DSIZE
		------   ------   -----   -----   -----   ------   -----   -----   -----
		     1    2.58M    289G    264G    264G    2.58M    289G    264G    264G
		     2     206K   12.6G   10.4G   10.4G     430K   26.4G   21.6G   21.6G
		     4    37.6K    692M    276M    276M     170K   3.04G   1.26G   1.26G
		     8    2.18K   45.2M   19.4M   19.4M    20.0K    425M    176M    176M
		    16      174   2.83M   1.20M   1.20M    3.33K   48.4M   20.4M   20.4M
		    32       40   2.17M    222K    222K    1.70K   97.2M   9.91M   9.91M
		    64        9     56K   10.5K   10.5K      865   4.96M    948K    948K
		   128        2   9.50K      2K      2K      419   2.11M    438K    438K
		   256        5   61.5K     12K     12K    1.90K   23.0M   4.47M   4.47M
		    1K        2      1K      1K      1K    2.98K   1.49M   1.49M   1.49M
		 Total    2.82M    303G    275G    275G    3.20M    319G    287G    287G
		
		dedup = 1.05, compress = 1.11, copies = 1.00, dedup * compress / copies = 1.16
    ```
    
### zfs and jail

    `zfs jail jailid ` 会将 dataset 分配给指定的 jail;
    出于安全的原因, ZFS 禁止host挂载一个jaild dataset;
    
### Delegated Administration
    
    给特定用户权限在父级下创建dataset, 创建包括挂载;
    需要`vfs.usermount` 设置为1 允许非root用户挂载fs.
    ```
    zfs allow someuser create mydataset 
    ```
    
    赋予该用户给其他用户授权的权限
    ```
    zfs allow someuser allow mydataset
    ```
    
## 高级

### Tuning
    
    - vfs.zfs.arc_max: 
    - vfs.zfs.arc_meta_limit: 
    - vfs.zfs.arc_min:
    - vfs.zfs.vdev.cache.size:
    - vfs.zfs.min_auto_ashift: 2的n次方, 
    - vfs.zfs.prefetch_disable: 随机读写多时, 可以关闭
    - vfs.zfs.vdev.trim_on_init: 
    - vfs.zfs.vdev.max_pending: 
    - vfs.zfs.top_maxinflight: 
    - vfs.zfs.l2arc_write_max:
    - vfs.zfs.l2arc_write_boost:
    - vfs.zfs.scrub_delay:
    - vfs.zfs.resilver_delay:
    - vfs.zfs.scan_idle:
    - vfs.zfs.txg.timeout:

## i386
### Memory
    
    最少: 1GB  
    通常: 1GB per 1TB storage
    dedup场合: 5GB per 1TB
    
### kernel

    ```
    options        KVA_PAGES=512
    
    vm.kmem_size="330M"
    vm.kmem_size_max="330M"
    vfs.zfs.arc_max="40M"
    vfs.zfs.vdev.cache.size="5M"
    ```
    
### 其他参考    
    [ZFSTuningGuide](https://wiki.freebsd.org/ZFSTuningGuide)
    openzfs手册(https://docs.oracle.com/cd/E56344_01/html/E54077/zfs-1m.html)
