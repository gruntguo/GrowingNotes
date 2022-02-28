# Lib


## std
stdio.h, 标准io
stdlib.h, 标准库
stddef.h, 标准宏和变量
stdarg.h, 头文件定义了一个变量类型 va_list 和三个宏

```
  NULL ((void *)0)
  offsetof()
  size_t
  ptrdiff_t, 通常用来表示两指针相减的结果
  wchar_t, 为国际化字符集提供宽字符
  
```

time.h 是 ISO C99 标准日期时间头文件。
sys/time.h 是 Linux系统 的日期时间头文件。
  sys/time.h 通常会包含 #include <time.h> 。
  
fcntl.h  文件控制
sys/ioctl.h
sys/filio.h  跟ioctl有关
signal.h
sys/wait.h  函数说明wait()会暂停进程的执行(挂起父进程),直到有信号来到或子进程结束。
ctype.h  字符处理库函数, 
  int isalpha(int c)  测试c是否为字母
  int isdigit(int c)  测试c是否为十进制数字
  int tolower(int c)  该函数把大写字母转换为小写字母。
  ...等等
grp.h 组文件,access group databases
glob.h 路径名模式匹配


## net

netinet/in.h, POSIX指定的标准头文件, 相当于 linux/in.h
netdb.h, 定义了与网络有关的结构,变量类型,宏,函数


## file

sys/file.h
sys/ndir.h
sys/dir.h


## system_call

### sysconf
```
sysconf( _SC_PAGESIZE );  此宏查看缓存内存页面的大小；打印用%ld长整型。
 sysconf( _SC_PHYS_PAGES ) 此宏查看内存的总页数；打印用%ld长整型。
 sysconf( _SC_AVPHYS_PAGES ) 此宏查看可以利用的总页数；打印用%ld长整型。
 sysconf( _SC_NPROCESSORS_CONF ) 查看cpu的个数；打印用%ld长整。
 sysconf( _SC_NPROCESSORS_ONLN ) 查看在使用的cpu个数；打印用%ld长整。
 (long long)sysconf(_SC_PAGESIZE) * (long long)sysconf(_SC_PHYS_PAGES) 计算内存大小。
 sysconf( _SC_LOGIN_NAME_MAX ) 查看最大登录名长度；打印用%ld长整。
 sysconf( _SC_HOST_NAME_MAX ) 查看最大主机长度；打印用%ld长整。
 sysconf( _SC_OPEN_MAX )  每个进程运行时打开的文件数目；打印用%ld长整。
 sysconf(_SC_CLK_TCK) 查看每秒中跑过的运算速率；打印用%ld长整

```
