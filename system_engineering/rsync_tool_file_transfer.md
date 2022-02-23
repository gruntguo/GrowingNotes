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
[参考算法](https://rsync.samba.org/tech_report/tech_report.html)
[陈皓-rsync算法解析](https://coolshell.cn/articles/7425.html)

###  adler-32
	Mark Adler发明的校验和算法
