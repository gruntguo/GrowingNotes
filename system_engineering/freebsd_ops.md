# FreeBSD 常用工具
  记录freebsd 常用的管理工具, 使用时可以速查
  

## file
  
  - [net](#Net)

  cpdup

## dev
  pciconf: 查看pci bus  例: pciconf -lv
  devinfo: 查看设备树?

## sys
  fstat: 查看文件打开状态
  sysctl: 查看/设置 kernel状态

## boot

	disable-module
	

## system utils

### date
    
    # tzsetup
    
    # cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

    # cat /var/db/zoneinfo

## terminals
  vt
  
## vnc
  pkg install wayvnc

## Net
  
  sockstat:     列出监听端口
  
  ath:  wireless ethernet driver
  
  netstat: 
  
  cu: establishes a full-duplex connection to another machine
  
  fetch: retrieve a file by Uniform Resource Locator

### firewall

    /etc/services, port number
    
#### PF

    pfctl(8)
    
    pflog(4)
    
### bridge

    # ifconfig bridge create
    
    # ifconfig br0
    
    # ifconfig bridge0 inet 192.168.0.1/24
    
    ```
    # 配置 rc.conf
    cloned_interfaces="bridge0"
    ifconfig_bridge0="addm fxp0 addm fxp1 up"
    ifconfig_fxp0="up"
    ifconfig_fxp1="up"
    
    ```
#### spanning tree    

    ```
    # ifconfig bridge0 stp fxp0 stp fxp1
    
    ```
    parameter:
    
    - private
    - span
    - sticky

    monitor mode: packets are discarded after bpf(4), 
                  not processed or forward further
                  
#### snmp monitoring

    bsnmpd(1), 可以监控 STP parameters
    
    /etc/snmpd.config  ,  uncomment this line  
    
    ```
    begemotSnmpdModulePath."bridge" = "/usr/lib/snmp_bridge.so"
    ```
#### link aggregation and failover

    lagg(4), 
    

## kernel

  kldload:  load a file to kernel
  

## disk

    gpart create -s GPT ada1  
    gpart add -t freebsd-ufs -a 1M ada1  
    gpart show ada1  
    newfs -U /dev/ada1p1  
    gpart show ada0  
    gpart recover ada0  
    gpart show ada0  
    swapoff /dev/ada0p3  
    gpart delete -i 3 ada0  
    gpart show ada0  
    gpart resize -i 2 -s 47G -a 4k ada0  
    gpart add -t freebsd-swap -a 4k ada0  
    gpart show ada0  
    swapon /dev/ada0p3  
    growfs /dev/ada0p2  

    zpool online -e zroot /dev/ada0p2  

    camcontrol devlist
    
### usb

    usbconfig

### cd

    camcontrol devlist  

    sysutils/cdrtools installs mkisofs, ISO 9660  
    mkisofs -o imagefile.iso /path/to/tree
    
    ```
    
    mount iso as memory disk:
    # mdconfig -a -t vnode -f /tmp/bootable.iso -u 0
    # mount -t cd9660 /dev/md0 /mnt
    
    ```
### dvd

    sysutils/dvd+rw-tools   中提供了 growisofs(1). 
     
     ATAPI/CAM  需要被loanded
     
     ```
         DVD+R or a DVD-R
         # growisofs -dvd-compat -Z /dev/cd0 -J -R /path/to/data
         
         To burn a pre-mastered image
         # growisofs -dvd-compat -Z /dev/cd0=imagefile.iso
     ```

### NTFS
    
    - load kernel  
      kldload fusefs  
      sysrc kld_list+=fusefs  
    - install NTFS  
      pkg install fusefs-ntfs  
    - mkdir  
      mkdir /mnt/usb
    - gpart show da0
    - mount  
      ntfs -3g /dev/da0s1 /mnt/usb/
    - fstab  
      /dev/da0s1  /mnt/usb	ntfs mountprog=/usr/local/bin/ntfs-3g,noauto,rw  0 0  
    - mount /mnt/usb
    
### backups

#### dump

    rdump(8) 
    rrestore(8)
    
    ```
    # /sbin/dump -0uan -f - /usr | gzip -2 | ssh -c blowfish \
          targetuser@targetmachine.example.com dd of=/mybigfiles/dump-usr-l0.gz
          
    # env RSH=/usr/bin/ssh /sbin/dump -0uan -f targetuser@targetmachine.example.com:/dev/sa0 /usr
    
    # tar czvf /tmp/mybackup.tgz .
    
    # tar xzvf /tmp/mybackup.tgz
    
    # ls -R | cpio -ovF /tmp/mybackup.cpio
    
    # pax -wf /tmp/mybackup.pax .
    
    ```

#### tape
    sa(4),  SCSI devices
    mt(1), 操作磁带的命令
    
#### third-party

    - Amanda
    - Bacula
    - rsync
    - duplicity
    
    
### memory disk
    md(4)
    
    GENERIC 确保 `device md`
    
    
    ```
    # mdconfig -f diskimage.iso -u 0
    # mount -t cd9660 /dev/md0 /mnt
    ```

### snapshot
    
    mksnap_ffs(8) 
    
    ```
    # mount -u -o snapshot /var/snapshot/snap /var
    # mksnap_ffs /var /var/snapshot/snap
    # find /var -flags snapshot
    ```
    
    
    [参考](http://www.mckusick.com/)
    
### disk quotas (UFS)


    查看是否支持:
    ```
    sysctl kern.features.ufs_quota
    ```
    
    不支持的话: 
    ```
    options QUOTA
    quota_enable="YES"
    
    ```
    
    quota 检查:  
    quotacheck(8), 默认检查
    ```
    关闭:
    check_quotas="NO" 
    ```
    使用fstab, 开启quota

    ```
    user:
    /dev/da1s2g   /home    ufs rw,userquota 1 2
    group:
    /dev/da1s2g    /home    ufs rw,userquota,groupquota 1 2
    ```
    
    quotacheck(8), quotaon(8), quotaoff(8)
    
    quota -v
    
    edquota

### HAST highly available storage
    
    CARP（Common Access Redundancy Protocol）, 多台主机共享同一个IP;
    [参考](https://docs.freebsd.org/en/books/handbook/advanced-networking/index.html#carp)
    
    HAST provides regular GEOM providers in `/dev/hast/` 
    
    HAST 管理一个 on-disk 的 dirty bitmap, 只同步脏块
    
    同步策略: 
      - memsync ,  默认模式, 本地写成功, 远程接收到data即返回
      - fullsync,  类似双写, 本地远程都写成功后返回
      - async, 本地写成功即返回
      
    
    hastd(8) 提供同步,  当该守护进程start, 自动load geom_gate.ko  
    hastctl(8) 用户空间管理工具  
    hast.conf(5) 配置文件  
    
### GEOM
    
    GEOM, Modular Disk Transformation.
    
    
#### raid0, striping
    
    gstripe(8)
    
    ```
    # kldload geom_stripe
    # gstripe label -v st0 /dev/ad2 /dev/ad3
    # bsdlabel -wB /dev/stripe/st0
    # newfs -U /dev/stripe/st0a
    # mount /dev/stripe/st0a /mnt
    # mkdir /stripe
    # echo "/dev/stripe/st0a /stripe ufs rw 2 2" \
    >> /etc/fstab
    # echo 'geom_stripe_load="YES"' >> /boot/loader.conf
    
    ```
    
#### raid1, mirroring

    gmirror(8)
    
#### raid3

    graid(8)
    
#### GEOM Gate Network

    - ggated
    - ggatec
    
    /etc/gg.exports
    
    ```
    # ggatec create -o rw 192.168.1.1 /dev/da0s4d
    ggate0
    # mount /dev/ggate0 /mnt

    ```

#### labeling disk devices
    
    在最后一个sector 存储label, 开机时可以识别.  
    
    permanent label  
    
    
    ```
        # tunefs -L home /dev/da3
        
        /dev/ufs/home		/home            ufs     rw              2      2    
    ```
    
    temporary label  
    使用 `glabel create`
    

#### UFS Journaling Through GEOM

    `gjournal` 可以用来配置, 基于 `block`实现 , 并非基于文件系统实现.
    这是GEOM的一个extension
    
    ```
    /boot/loader.conf
    geom_journal_load="YES"
    
    GENERIC
    options	GEOM_JOURNAL
    ```
    
    ```
    # gjournal load
    # gjournal label /dev/da4

    # newfs -O 2 -J /dev/da4.journal
    # mount /dev/da4.journal /mnt

    ```
    
###  ZFS
      

## media

    kldload snd_hda


### idioms

```
    # -l 监听中的端口
	$ sockstat -4l 
	# -c 已连接的端口
	$ sockstat -4c

	
```
  
  
## user
    pw lock xxx
    chsh -s /usr/sbin/nologin toor
    
### password
    
    #begin_src shell
        pam_passwdqc.so
        /etc/pam.d/passwd

        password        requisite       pam_passwdqc.so         min=disabled,disabled,disabled,12,10 similar=deny retry=3 enforce=users
    #end_src

### rootkit
    security/rkhunter
    rkhunter -c

### IDS
    intrusion detection system
    
    mtree :
    https://docs.freebsd.org/en/books/handbook/security/
    
    security/aide :
    
### tune
    
    kern_securelevel_enable
    kern_securelevel
    
    net.inet.tcp.blackhole
    net.inet.udp.blackhole
    
    net.inet.icmp.drop_redirect 
    net.inet.ip.redirect
    
    net.inet.icmp.bmcastecho
    
### OPIE

    One-time Passwords In Everything OPIE
    
### TCP wrapper

### Kerberos

    security/krb5
    
    security/heimdal 
    
    KDC,  key distribution center
    
    
### vulnerability
    pkg audit -F
    
### process accounting
    
    sysrc accounting_enable=yes  
    service accounting start
    
    /var/account
    
    sa  
    lastcomm  
    acct  
    
### Resource Limits

    rctl
    
    cap.mkdb(1)
    getrlimit(2)
    login.conf(5)
    
    kern.racct.enable
    
    rctl -a jail:httpd:memoryuse:deny=2G/jail
    
    删除rule  
    rctl -r user:trhodes:maxproc:deny=10/user
    
### sudo


    sudoreplay 
    
#### doas

## etc

    mergemaster
    
## mac

    
## desktop

### i3


https://unixsheikh.com/tutorials/how-to-setup-freebsd-with-a-riced-desktop-part-3-i3.html

https://www.qluoman.top/2021/06/15/freebsd-13%E7%AC%94%E8%AE%B0%E6%9C%ACi3wm%E6%A1%8C%E9%9D%A2%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AEv2/


## tools

sysutils/monit 
net-mgmt/observium 
sysutils/webmin
net-mgmt/nagios
net-mgmt/zabbix32-server
sysutils/munin-master
net-mgmt/cacti
net-mgmt/netdata

https://www.logicmonitor.com/infrastructure-monitoring
