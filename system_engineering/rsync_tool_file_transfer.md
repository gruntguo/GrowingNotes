# Rsync
	rsync 是一个常用的file 同步工具,
	用来在本地与远程主机之间同步文件, 也可以当作 cp 来用.
	
	

## 安装
  常用工具, 发行版通常都有打包, 通过包管理工具安装.
  
	sudo apt install rsyrc

## 基本使用

  文件同步: 

  ```
  -r 递归包含子目录:
    rsync -r src des
	rsync -r src1 src2 des
	
  -a 同步元信息
    rsync -a src des
	
  src/ 源目录增加斜杠, 同步src中的内容
    rsync -a src/ des
 
  -n 或 --dry-run, 模拟执行,  
  -v 输出到 stdout
	rsync -anv src des
	
  --delete 在 des 中删除 src 不存在的文件
    rsync -av --delete src des
	
  --exclude 或 --exclude-from='filename' 排除文件, 可使用通配符号
    rsync -av --exclude='*.log' src des
	rsync -av --exclude-from='exclude-file.txt' source/ destination
	
  --include='*.log' 与exclude 结合, 列出必须同步的文件

  -e 
  rsync -avz /etc/hosts -e 'ssh -p 22' root@x.x.x.x:/mnt

  -z --compress 传输时进行压缩
  
  -P --partial 保留那些因故未传送完的文件, 以加快后续传输
  
  -H  保持硬链接

  -u  --update 跳过des 中, 时间新于客户端的文件
  ```
  
  远程同步:
  
  ```
   rsync -av source/ username@remote_host:destination
   rsync -av username@remote_host:source/ destination
  ```
  

## 高级功能

### 增量

  通过 `--link-dest` 指定基准目录, 为没有更改的文件创建硬链接.
  
  例如: 每天备份时, 指定--link-dest 为昨天备份的目录, 
        当今天执行备份时, 如果文件没有发生变化, 只会创建硬链接.

  ```
   rsync -a --delete --link-dest /compare/path /source/path /target/path
  ```
  
  脚本示例:
  ```
    #!/bin/bash

    # A script to perform incremental backups using rsync

    set -o errexit
    set -o nounset
    set -o pipefail

    readonly SOURCE_DIR="${HOME}"
    readonly BACKUP_DIR="/mnt/data/backups"
    readonly DATETIME="$(date '+%Y-%m-%d_%H:%M:%S')"
    readonly BACKUP_PATH="${BACKUP_DIR}/${DATETIME}"
    readonly LATEST_LINK="${BACKUP_DIR}/latest"

    mkdir -p "${BACKUP_DIR}"

    rsync -av --delete \
      "${SOURCE_DIR}/" \
      --link-dest "${LATEST_LINK}" \
      --exclude=".cache" \
      "${BACKUP_PATH}"

    rm -rf "${LATEST_LINK}"
    ln -s "${BACKUP_PATH}" "${LATEST_LINK}"

  ```

### 限速

  --bwlimit=RATE
  参数值的默认单位是KBPS，也就是每秒多少KB, 
  ```
	rsync -auvz --progress --delete --bwlimit=1000 远程文件  本地文件
	限制为 1000k Bytes/s
  ```

### 以守护进程运行

>	rsync --daemon
 
  设置配置文件, [参考](https://download.samba.org/pub/rsync/rsyncd.conf.5)
	


## 问题记录

- 报错 "rsync: not found"
  需要两边主机都安装rsync


## rsync 数据处理过程

### 术语
- client, role, 源主机
- server, role, 对象主机, 连接建立以后, 即clinet 是发送方, server 是接收方
- daemon, role & process, 等待client连接的进程
- remote shell, role & set of process, 对象主机端的一个和多个进程,来提供连接
- sender, role & process, 一个进程来读取 src 文件 并进行同步
- receiver, role & process, 一个进程在ds, 接收数据并落盘
- generator, process, 一个来识别文件变更和文件层面的一些管理工作

### 主要流程
client 建立与 server process 的连接, pipe 或者 socket

mode 1 (non-daemon),   
  如果是non-daemon server端, 首先 fork 一个 remote shell,  
  client 端和 server 端的通讯通过 pipe 使用 remote shell,  
  直到确认网络间没有数据传送,  
  在这个模式下, server 的参数通过 command line 进行解析, 然后启动一个remote shell  
mode 2 (daemon),  
  直接通过socket 进行通讯, 
  
因为sender 和 receiver 在连接初期, 两边使用支持的最高协议版本进行协商, 最终使用最低支持的level.

### File list

包括 pathname, ownership, mode, permissons, size, modtime; 
指定了 --checksum 的话, 还包括 checkums.

首先sender 会建立filelist, 然后传送给 receiver,
然后根据相对的路径进行排序.
收到file list后 ,fork出来一个generator

rsync 用pipline 组织数据流:  
generator -> sender -> receiver

每个操作都使用单独进程, 数据是单向的.

generator 比较file list, 如果 --delete 指定, 会先找出需要删除的文件.  
然后, generator 将遍历list,   
通常场景: 比较更改时间和大小  
校验码场景:  会进行校验码的比较,后面会解释该算法, 默认是时间和大小.

### sender
读取 generator 产生的 file index 和 checksum, 包括为文件 block 建立hash table,
接着就是循环 [校验算法](# 一些算法)

简单来讲, 就是 src 文件按块进行匹配, 把不匹配的块,该块的偏移和接收端的块的大小,
按这种方式, 即使同一个文件块的发送顺序不一样,对最后的结构不会有影响,

比较: 
- 使用校验算法比较占用cpu时间, 但节省带宽, 适用那种文件变更少的场合.
- 使用默认,即时间和大小, 对单个文件来讲,就不是增量方式了.

### receiver

接收端会根据 index number 读取数据, 然后打开文件,并创建临时文件.  
接收端读取非匹配块和匹配的块, 最终重新组合成一个文件,这里有两点:  
1. 收到非匹配块, 被写入临时文件
2. 收到匹配块儿, 按偏移量进行寻找, 将该块写入临时文件
按这种方式, 把各个块组合成一个临时文件,  
文件重建后, 会再跟发送端的检验码进行校验; 校验不符, 删除临时文件,重建.
默认只会重试一次, 之后再有问题, 就会出错误报告.  
最终加上文件所有者,权限和时间等信息, 然后替代基础文件.


接收端主要是从基础文件复制到临时文件, 主要消耗磁盘 IO, 
要注意大文件的处理, 数据是随机进行读, 
如果读写超过磁盘缓存的限制, 会有所谓"寻找风暴",进一步损伤性能

### 协议

数据作为字节流被传输

例如, 发送一个文件列表, 就是发送列表中的每一条, 发送结束就是一个NULL字节,
每一条都是变长字符串, 被NULL终结.

每个版本的协议表现不完全一致.

所以很难进行文档化.

### 文档参考

参考[how rsync works](https://rsync.samba.org/how-rsync-works.html)

## 一些算法

其核心是rolling checksum算法, 在备份领域经常使用.

```
a，分块checksum算法：用于将目标文件平均切分成若干块，然后对每块计算两个checksum<一弱：用于区别不同。一强：用于确认相同>。

b，传输算法：同步目标端，会把目标文件的checksum列表<强，弱，文件块编号>传给同步源。

c，checksum查找算法：同步源拿到目标文件的checksum数组后，把数据存在hash table里，用弱做hash。

d，对比算法：

        1，取源文件的第一个文件块，做弱计算，计算好的值去hash table中查找。

        2，如果查到了，说明发现在目标文件又潜在相同的文件块。再比较强计算。如果弱强都相同，标记这一块在目标文件的文件编号。

        3，如果没查到不用计算强，算法往后setp 1个字节，取源文件的第二个块做弱计算。

        4，根据以上，找出源文件相邻两次匹配中的文本字符，就是需要往目标同步的文件内容了。

```

### 文档参考
[参考算法](https://rsync.samba.org/tech_report/tech_report.html)
[陈皓-rsync算法解析](https://coolshell.cn/articles/7425.html)

###  adler-32
	Mark Adler发明的校验和算法
