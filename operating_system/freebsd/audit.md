# security event auditing

实现了SUN 的BSM(basic security module) API 和 文件format, 可以和Solaris,Mac 的audit兼容.

## 术语

event : 可以使用audit 子系统记录的事件, 一般以日志的形式;  
        事件可以追踪到用户, 也可以与用户无关, 比如在登录前发生的事.
        
class : 事件类型.  
        fc(file creation), ex(exec), lo(login logout)
        
record : 日志条目

trail : 日志文件, 由一系列 record 组成

selection expression : 用来匹配 event 的字符串

preselection : 识别和过滤的进程, preselection configuration 使用 selection expression 来识特定class 和相关用户的事件

reduction : 归档策略

## 基本配置

GENERIC kernel 默认已编译, 在rc.conf中, 开启 `auditd(8)`

```
auditd_enable="YES"
```
    
### default class

- all: all
- aa: authenticatication and authorization
- ad: administrative
- ap: application 
- cl: file close
- ex: exec, audit_control(5) 
- fa: file attribute access
- fc: file create
- fd: file delete
- fm: file attribute modify
- fr: file read
- fw: file write
- lo: ioctl
- ip: ipc
- lo: login_out;Audit login(1) and logout(1) events.
- na: non attributable
- no: invalid class
- nt: network; Audit events related to network actions such as connect(2) and accept(2).
- ot: other
- pc: process; Audit process operations such as exec(3) and exit(3).


### prefixes

- + : successful events
- - : failed events 
- ^ : neither successful nor failed events
- ^+: Do not audit successful events in this class.
- ^-: Do not audit failed events in this class.

没有前缀时, successful 和 failed 事件都会被audit

例: `lo,+ex`  
成功和失败的登录事件, 成功的程序执行事件

### configuration file

`/etc/security` 的配置:
 - audit_class
 - audit_control
 - audit_event
 - audit_user
 - audit_warn
 
 `audit_control` 配置:
 
 ```
     dir:/var/audit
     dist:off
     flags:lo,aa
     minfree:5
     naflags:lo,aa
     policy:cnt,argv
     filesz:2M
     expire-after:10M
 ```

`audit_user` 配置:

```
root:lo,+ex:no
www:fc,+ex:no
```

用户名:需要被审计的事件:不需要的事件

## audit trails

文件使用BSM格式, 使用工具更改和查看.

`praudit` 可以把文件转化为文本

```

header,116,11,execve(2),0,Thu Mar 24 07:26:18 2022, + 312 msec
exec arg,ls
path,/bin/ls
attribute,555,root,wheel,1374973508,267,2174235592
subject,guo,root,wheel,root,wheel,4355,4224,53950,192.168.0.1
return,success,0
trailer,116

```

- exec arg, 命令和参数
- path, 执行文件的路径
- attribute, 文件的一些属性
- subjetc, user ID, effective user ID and group ID, real user ID and group ID, process ID, session ID, port ID, and login address


`auditreduce` 可以用来过滤

```
过滤用户 guo 的事件
auditreduce -u guo current | praudit
```

## audit pipe

`/dev/auditpipe` 是一个伪设备, 可以用来收集实时的事件流.
主要用来做侵入探测和监控.

```
praudit /dev/auditpipe
```

默认只能root访问, 增加`/etc/devfs.rules`规则, 把pipe加入到audit组
```
add path 'auditpipe*' mode 0440 group audit
```

## Rotating and Compressing

由 audit(8) 管理, 不能使用 newsyslog.conf(5) 或者其他工具直接rotate.

```

开启一个新的 log 文件, 发送signal, 使 kernel 使用新的文件

audit -n

以下加入到 /etc/crontab 中, 每隔12小时 rotate 一次,
/etc/crontab 保存时生效

0     */12       *       *       *       root    /usr/sbin/audit -n
```

audit_control 文件中的 `files` 可以按大小进行rotate.

audit_warn 脚本可以根据审计的事件进行定制操作, 
仅当trails file 由auditd正常关闭时有效,非正常关闭不执行该脚本.

```
#
# Compress audit trail files on close.
#
if [ "$1" = closefile ]; then
        gzip -9 $2
fi
```

其他操作举例, 
- 复制trails 到中央日志服务器
- 删除老旧的trail
- reduce 用以减少不必要的记录

## ref

[openBSM](http://www.trustedbsd.org/openbsm.html "bsm")

[practical](http://www.nycbsdcon.org/2010/presentations/nycbsdcon-freebsd-audit.pdf)


