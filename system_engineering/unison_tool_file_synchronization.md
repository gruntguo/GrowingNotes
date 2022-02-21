# Unison

``` 
Unison is a file-synchronization tool for OSX, Unix, and Windows. It allows two replicas of a collection of files and directories to be stored on different hosts (or different disks on the same host), modified separately, and then brought up to date by propagating the changes in each replica to the other
```

## 简介
  Unison是一个多个平台(linux,windows,unix 等)下的文件同步工具.

  优势总结:

  - 跨平台
  - 双向同步; 根据冲突可选择相应的策略
  - 使用socket 或 ssh 连接两台主机, 可使用 rsync 进行传输

  劣势(notorious)总结:

  - 两方 unison 版本必须一致; 
	涉及到一些算法,不同版本之间有差异,会导致同步报错;
	特别是由于源码是ocaml写的,所以编译时的ocaml版本也要一致.
  - 配置项较多;
	由于有些需要在实践中琢磨.
  - 场景有限;
	双向同步虽然是优势, 
	但一般场景使用 `rsync` 做备份就够了, 再复杂就是配置管理了, 感觉不太需要双向.

## 安装


### unison 安装
  常见Linux发行版使用包管理工具安装;
  
	```
	sudo apt search unison
	unison-2.51+4.11.1/stable,now 2.51.3-1 amd64 [已安装，可自动卸载]
	file-synchronization tool for Unix and Windows
	```

  特别注意, 同步双方的机器都需要安装, 并且版本要一致.
  
  推荐下载编译后的文件, 这样版本容易对齐.
  
  参考 [release](https://github.com/bcpierce00/unison/releases)
  
### ssh key 配置
  两台机器间可使用ssh 进行通讯和数据拷贝, 过程不详述.
  
  使用 `ssh-keygen` 生成key
  使用 `ssh-copy-id` 把key复制到对象主机
  
  
## 使用

  可以在单机上同步两个文件夹, 

### 单机使用入门
  单机上, 可执行命令来同步两个文件夹.
  
  ```
  unison a.tmp b.tmp
  
  ```
  
  第一次执行时, 会弹出一堆说明, 
  如果两个文件夹一致时, 直接结束, 不一致时, 会有交互提示出现
  
  ```
  a.tmp          b.tmp              
  new file ---->            z01  [f] 
  
  ```

  这时, 中括号内是默认操作, 比如 a 中增加了新文件, 但 b 中没有;
  默认给了一个策略, 即从 a 复制到 b, 敲下 'f' 即是默认策略， “/” 跳过，等等
  此时, 敲个'?' 可以查看下支持的命令.
  
  ```
  a.tmp          b.tmp              
  new file <-?-> new file   z02  [] 
  ```
  当两个文件都存在, 但是内容不一致时, 会出现  <-?->;
  这时, 无法给出推荐的策略, 需要手工去选择.
  下文讲的非交互式执行时, 有不少配置可以对自动选择策略进行影响, 默认是 skip.
  
	
### 双机同步
  
  双机同步跟单机一样, 两个root 文件夹,  其中远程主机设置成ssh就行;
  同时, 不使用命令行参数, 而通过配置文件设置各项参数.
  
  ```
  默认配置文件: ~/.unison/default.prf
  
  执行时 : unison default.prf
  
    
  ```
  注: 多个配置时, 建议更改default名字, 
      我这边进行尝试时, 虽然指向其他配置文件, 但是default 还是会读取
	  
  以下是一些我用的配置项:
  
  ```
  # Unison preferences file
  # 两个需要同步的目录, 以ssh 方式
  root = /home/xxx/working/Bak-from-xxx
  root = ssh://xxx@xx//home/xxx/backup

  # 强迫以某一个root为基准 
  # 即设置该配置时, 同步由双向变成了单向
  # force
  
  # 远端的执行命令, 需要保证版本一致
  servercmd=unison
  
  # 忽略哪些目录或文件
  # ignore

  # 可以使用通配符
  # Some regexps specifying names and paths to ignore
  # ignore = Name temp.*
  # ignore = Name *~
  # ignore = Name .*~
  # ignore = Path */backup/Archive_*
  # ignore = Name *.o
  # ignore = Name *.tmp
  
  # 无交互模式
  # 无冲突（可执行默认策略）的文件直接进行， 有冲突的文件忽略
  batch = true
  auto = true

  # 可以根据文件变更进行触发
  # 间隔 1秒再次进行
  # repeat = 1
  # 根据文件监控进行触发, 使用自带的另一个工具 unison-fsmonitor
  # repeat = watch 
  
  # 失败后重试的次数
  # retry = 3

  # 同步文件权限
  # 同步用户组信息
  # windows 尚不清楚如何处理
  owner = true
  group = true
  
  # 文件权限设置
  # 设置成 -1 表示同步所有的权限位
  perms = -1

  # 表示同步时仅通过文件的创建时间来比较，如果选项为false，
  # Unison则将比较两地文件的内容。
  # default, 使用fast checks on Unix, slow checks on windows
  # fastcheck = false
  
  # 使用rsync 传输
  # 大文件传输时， 官方推荐使用rsync
  # 同时可以通过设置其他参数，大于某个size的文件，使用rsync
  # 換句话说， false 是全量传输
  # 当日志中反复出现 “ rsync failure ” 时， 考虑该参数，同时需要配合其他的参数
  # rsync = false

  # 使用ssh 的压缩传输方式
  sshargs = -C

  # default 是ture
  # 避免已存在的file的传输？
  # xferbycopying = true
  
  
  # 当 batch 模式时，遇见整个目录为空的对象，忽略;
  # 非 batch 模式时， 额外的交互确认
  confirmbigdel=true
  
  # 输出log
  log = true
  logfile = /home/xxx/.unison/unison_default.log

  ```

### 问题
  列一下实践时遇到的几个问题:
  
  - 如何确定第一次执行还是已同步过?
    
	在 ~/.usison/中, 会为每次同步的文件生成个cache 文件, 
	该文件记录上次的状态, 具体方式待后续研究下代码再补充.
	如果需要重新同步, 可以删除这些cache文件
	
  - 出现冲突时, 如何进行处理?
    命令行方式可以有交互模式进行选择, batch 模式时, 如果没有其他的配置, 默认是skip


  
  
  
  
  
  
  


