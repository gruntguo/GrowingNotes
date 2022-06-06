# kernel configuring

[manual](https://docs.freebsd.org/en/books/handbook/kernelconfig/)

## intro
  /boot/kernel :  modules 可以被 kldload 
  /boot/loader.conf : 
  
### hardware
  
  dmesg: 查看硬件信息  
    /var/run/dmesg.boot  
  pciconf: 查看pci  
    pciconf -lv  
  man -k : 可以使用 -k 获取更多的信息  
  
  /usr/src/sys :  不同架构的文件  
    每种架构下,都有conf 文件夹 , *GENERIC* 就在其中
    
  man 5 config: 
  
### build 

  到路径下: cd /usr/src
  指定配置: make buildkernel KERNCONF=MYKERNEL
  安装: make installkernel KERNCONF=MYKERNEL
    将会把新的kernel 安装在 /boot/kernel/kernel/
    把旧kernel 重命名  /boot/kernel.old/kernel
  
  kernel 和 build 的tool 例如ps ,vmstat 版本保持一致,不然这些工具容易出问题
  
### NOTES
    /usr/src/sys/conf/NOTES
    /usr/src/sys/arch/conf/NOTES 
  
## Q&A
  
  
  

  
  
  
    
  
