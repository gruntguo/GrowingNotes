# FreeBSD 常用工具
  记录freebsd 常用的管理工具, 使用时可以速查
  
  莫名奇妙入了bsd的坑, 学习一波.
  
## file
  
  - [net](#Net)

  cpdup

## dev
  pciconf: 查看pci bus
  devinfo: 查看设备树?

## sys
  fstat: 查看文件打开状态
  sysctl: 查看/设置 kernel状态

## terminals
  vt
  
## vnc
  pkg install wayvnc

## Net
  
  sockstat:     列出监听端口
  
  ath:  wireless ethernet driver
  
  netstat: 
  

## kernel

  kldload:  load a file to kernel
  
    
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

    
