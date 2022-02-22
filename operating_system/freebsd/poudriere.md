# Poudriere

  用来创建和测试 FreeBSD package 的一个工具.
  学习这个是因为它通过 jails 创建一个隔离的环境, 可以用来在不同版本的 FreeBSD 上build packages.
  jails 在另外进行介绍, 先入门 poudriere.
  
## 安装
  使用 `ports-mgmt/poudriere` 进行安装.
  
  会生成一个配置文件的sample, ** /usr/local/etc/poudriere.conf.sample **.
  复制出来作为配置文件, 列出几个需要更改的配置项.
  
  ```
  # ZFS , 如果文件系统不是zfs, 设置NO_ZFS 为 yes
  ZPOOL=zroot
  #NO_ZFS=yes
  
  # jails 的镜像
  FREEBSD_HOST=ftp://ftp.jp.freebsd.org

  # jail 和 ports 的存储目录
  BASEFS=/usr/local/poudriere
  
  # 其他暂时默认
  
  
  ```


  接下来根据官网手册进行初始化, 创建jails, 指定一个 FreeBSD 版本 
  ```
  poudriere jail -c -j 11amd64 -v 13.0-RELEASE

  -K myGENERIC
  设置内核, 使用GENERIC 默认不需要这个选项.
  
  查看
  poudriere jail -l

  删除jail
  poudriere jail -d -j 11amd64

  ```
  
  初始化port tree
  ```
  poudriere ports -c -p local -m git+https
  
  使用本机 ports 的话, 用 -M 参数
  poudriere ports -c -m null -M /usr/ports -p local
  
  查看
  poudriere ports -l 
  
  ```
  
  通常会有多个 jails 对应不同版本或架构, 同时可以对应不同的ports tree
  

  感觉空间不够了.
  ```
  zroot/poudriere                  6.6G     96K    6.6G     0%    /zroot/poudriere
  zroot/poudriere/jails            6.6G     96K    6.6G     0%    /zroot/poudriere/jails
  zroot/poudriere/jails/11amd64    8.0G    1.4G    6.6G    17%    /usr/local/poudriere/jails/11amd64
  zroot/poudriere/ports            6.6G     96K    6.6G     0%    /zroot/poudriere/ports

  ```


## 测试


  ```
  poudriere testport -j 11amd64 -p local -o net/netcat
  ```

## build

  ```
  poudriere bulk -f ./mysamplelist -j 11amd64
  ```
  
## 其他
  (待补充)
  
