# FreeBSD 常用工具
  记录freebsd 常用的管理工具, 使用时可以速查
  
  莫名奇妙入了bsd的坑, 学习一波.
  
## 目录  
  
  - [net](#Net)

## Net
  
  sockstat:     列出监听端口

### idioms

```
    # -l 监听中的端口
	$ sockstat -4l 
	# -c 已连接的端口
	$ sockstat -4c

	
```
  
  
