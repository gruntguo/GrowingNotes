# jails
    
    freebsd的容器, 大名鼎鼎的jails. freebsd 4.X 时加入.  
    
    jail建立在在chroot的基础上, processes创建chrooted 环境后, 无法访问之外的环境  
    然而, chroot 也有局限, 比较适合简单的任务, 而且也有一些方法可以跳出chroot环境
    
    Jail 比chroot 有所加强, chroot只限制了文件系统, jail 扩大了限制的模型,
    提供了更多的控制, 可以被看作 操作系统层面的虚拟化.
    
    
    
## 4个特征

    - 目录树  
      一旦进入楼jail, precess是不允许访问之外的子目录的  
    - hostname  
      jail 有一个hostname  
    - IP address
      通常使用一个alias address, 在一个已经存在的网络interface上
    - 启动命令
      jail 中的相对路径和可执行程序
      
      
    同时, jail 有自己的user 和 root 用户, root 禁止在 jail 以外的环境执行命令.
    
    jail虽然很强大, 但也不是万能药；jail外层的用户可以与jail中的用户交互,
    使其达到提权访问外界的目的.
    
## 术语

    chroot(8)  
      更改root directory  
    chroot(2)  
      系统调用  
    jail(8)  
      jail 工具集
    host(system, process, user,etc.)  
      
    hosted(system, process, user,ect.)  
    
## 创建 jail

    一般来说, 分为 complete jail ,和 service jail;  
    顾名思义, complete jail 相当于虚拟了整个FreeBSD 系统,
    而service jail, 相当于只运行受限的程序.


### installation
    
    bsdinstall 可以帮助安装jail所需的binaries;  
    有base, lib32, ports 等等
    
    ```
      使用bsdinstall
      # bsdinstall jail /here/is/the/jail


      使用 iso
      # sh
      # export DESTDIR=/here/is/the/jail
      # mount -t cd9660 /dev/`mdconfig -f cdimage.iso` /mnt
      # cd /mnt/usr/freebsd-dist/
      解压
      # tar -xf base.txz -C $DESTDIR
      # for set in base ports; do tar -xf $set.txz -C $DESTDIR ; done
      
      从源码安装
      # setenv D /here/is/the/jail
      # mkdir -p $D      
      # cd /usr/src
      # make buildworld  
      # make installworld DESTDIR=$D  
      # make distribution DESTDIR=$D  
      # mount -t devfs devfs $D/dev 

    ```

### 配置
    
    jail(8)
    
    编辑 /etc/jail.conf
    sysrc jail_enable="YES"
    
    ```
        # service jail start www
        # jls
        JID  IP Address      Hostname                      Path
        17  192.168.0.10    www.example.org               /jails
        
        jexec 17 sh
    ```
    
    ip 地址不确定怎么配置, 后面再补充.
    参考: https://www.chinafreebsd.cn/article/5e9d356c7cb5c

### 网络

    开启pf
    ```
    
    # cp /usr/share/examples/pf/pf.conf /etc/pf.conf
    
    # service pf start                                                              # service pflog start
    
    pfctl -e ; pfctl -f /etc/pf.conf
    
    
    # sysctl net.inet.ip.forwarding=1
    # sysctl net.inet6.ip6.forwarding=1
    # sysrc gateway_enable=yes
    # sysrc ipv6_gateway_enable=yes

    ```
    
    if_bridge(4) and if_epair(4).
    ```
    # ifconfig bridge create
    bridge0
    # ifconfig epair create
    epair0a
    # ifconfig bridge0 addm rm0
    # ifconfig epair0a mtu 1450
    # ifconfig bridge0 addm epair0a
    
    ```

### 其他配置

    允许ping: 
      sysctl security.jail.allow_raw_sockets=1
      
      
    其他:
    security.jail.set_hostname_allowed: 1
    security.jail.socket_unixiproute_only: 1
    security.jail.sysvipc_allowed: 0
    security.jail.enforce_statfs: 2
    security.jail.allow_raw_sockets: 0
    security.jail.chflags_allowed: 0
    security.jail.jailed: 0



### 第三方配置工具
    
    sysutils/ezjail
    
### update
    ```
    # freebsd-update -b /here/is/the/jail fetch
    # freebsd-update -b /here/is/the/jail install
    # freebsd-update -b /here/is/the/jail --currently-running 12.0-RELEASE -r 12.1-RELEASE upgrade
    # freebsd-update -b /here/is/the/jail install
    # service jail restart myjail
    # freebsd-update -b /here/is/the/jail install

    # pkg -j myjail upgrade -f
    # service jail restart myjail
    ```

    
    
## 优化和管理

    主要通过sysctl(8) 进行调整
    

    security.jail.set_hostname_allowed: 1

    security.jail.socket_unixiproute_only: 1

    security.jail.sysvipc_allowed: 0

    security.jail.enforce_statfs: 2

    security.jail.allow_raw_sockets: 0

    security.jail.chflags_allowed: 0

    security.jail.jailed: 0


## ezjail
    
    
    
    
    
    
