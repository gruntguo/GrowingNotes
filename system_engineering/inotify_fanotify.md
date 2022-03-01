# inotify
inotify 是kernel的一个功能.

被 merge 到 kernel 2.6.13 中, api 是 linux 特有的, 并非posix.

inotify 有些限制, 后来又有了 fanotify, 不光可以通知, 还允许listener 介入, 进行控制.



## inotify overview
kernel parameter
```
ls /proc/sys/fs/inotify/
max_queued_events   # event队列的事件数量
max_user_instances  # 每个用户可运行的 inotifywait 或 inotifywatch 的进程数
max_user_watches    # 每个进程可监视的文件数

```

文档查看

`man inotify`

系统调用:  

- int inotify_init1(int flags);
- int inotify_add_watch(int fd, const char *pathname, uint32_t mask);
- int inotify_rm_watch(int fd, int wd);

### 代码

```
struct inode{ ...
#ifdef CONFIG_FSNOTIFY
	__u32			i_fsnotify_mask; /* all events this inode cares about */
	struct fsnotify_mark_connector __rcu	*i_fsnotify_marks;
#endif
...
}

/*
 * A mark is simply an object attached to an in core inode which allows an
 * fsnotify listener to indicate they are either no longer interested in events
 * of a type matching mask or only interested in those events.
 *
 * These are flushed when an inode is evicted from core and may be flushed
 * when the inode is modified (as seen by fsnotify_access).  Some fsnotify
 * users (such as dnotify) will flush these when the open fd is closed and not
 * at inode eviction or modification.
 *
 * Text in brackets is showing the lock(s) protecting modifications of a
 * particular entry. obj_lock means either inode->i_lock or
 * mnt->mnt_root->d_lock depending on the mark type.
 */
struct fsnotify_mark {
...
}


int fsnotify_add_mark_locked(struct fsnotify_mark *mark,
			     fsnotify_connp_t *connp, unsigned int type,
			     int allow_dups, __kernel_fsid_t *fsid)
{
...
	list_add(&mark->g_list, &group->marks_list); // 将mark 加入到group 中
	...
	ret = fsnotify_add_mark_list(mark, connp, type, allow_dups, fsid);
	// 将mark 加入到 mark list 中, 用来确定event 发生时, 发送给哪些group和inode
	// 根据优先级排序
...
}

/*
 * A group is a "thing" that wants to receive notification about filesystem
 * events.  The mask holds the subset of event types this group cares about.
 * refcnt on a group is up to the implementor and at any moment if it goes 0
 * everything will be cleaned up.
 */
struct fsnotify_group {
	const struct fsnotify_ops *ops;	/* how this group handles things */
...

// 创建匿名文件句柄,关联文件
int anon_inode_getfd(const char *name, const struct file_operations *fops,
		     void *priv, int flags)
// 创建匿名文件
struct file *anon_inode_getfile(const char *name,
				const struct file_operations *fops,
				void *priv, int flags)
				
// 匿名文件操作集
static const struct file_operations inotify_fops = {
	.show_fdinfo	= inotify_show_fdinfo,
	.poll		= inotify_poll,
	.read		= inotify_read,
	.fasync		= fsnotify_fasync,
	.release	= inotify_release,
	.unlocked_ioctl	= inotify_ioctl,
	.compat_ioctl	= inotify_ioctl,
	.llseek		= noop_llseek,
};

const struct fsnotify_ops inotify_fsnotify_ops = {
	.handle_inode_event = inotify_handle_inode_event,
	.free_group_priv = inotify_free_group_priv,
	.free_event = inotify_free_event,
	.freeing_mark = inotify_freeing_mark,
	.free_mark = inotify_free_mark,
};



// 核心就是创建一个匿名我文件句柄
ret = anon_inode_getfd("inotify", &inotify_fops, group,
				  O_RDONLY | flags);
	
	
	
```

整理一下:

创建匿名inode
每建立一个inotify 实例, 内核会分配一个fsnotify_group
每一个监测会创建一个mark结构,挂入相应链表
inode 与 group 通过 mark 进行关联

### 匿名fd

```
guo@gg:/proc/sys/fs/inotify$ inotifywait -m /home
Setting up watches.
Watches established.

guo@gg:/proc/sys/fs/inotify$ ls -l /proc/203716/fd
总用量 0
lrwx------ 1 guo guo 64  3月  1 09:17 0 -> /dev/pts/3
lrwx------ 1 guo guo 64  3月  1 09:17 1 -> /dev/pts/3
lrwx------ 1 guo guo 64  3月  1 09:17 2 -> /dev/pts/3
lr-x------ 1 guo guo 64  3月  1 09:17 3 -> anon_inode:inotify

```

### inotify tool
inotifywatch,  统计事件
inotifywait, 监控文件和目录的事件


### sample

```
inotifywait -m /tmp

inotifywait -mrq -e create,delete /rsync/

#!/bin/bash
a="/usr/local/inotify-tools/bin/inotifywait -mrq -e modify,move,create,delete /rsync/"
b="/usr/bin/rsync -avz /rsync/* testfedora@192.168.100.5:/home/testfedora/"
$a | while read directory event file
    do
        $b &>> /tmp/rsync.log
    done

```
### 参考


[github](https://github.com/inotify-tools/inotify-tools)


## fanotify

inotify 的替换框架, 在5.1 的内核中, 增加了更多的事件.

基本功能:
 - 文件系统事件通知, 与inotify 类似
 - 文件系统监管, inotify 的扩展;
   三个基本模式, directed, per-mount, global
 - 访问控制
 - 监听者级别划分
 - 监听者程序的PID过滤
 - 缓存, ignore marks

### 系统调用

int fanotify_init(unsigned int flags, unsigned int event_f_flags);

nt fanotify_mark(int fanotify_fd, unsigned int flags,
                  uint64_t mask, int dirfd, const char *pathname);

### 访问控制

handle_perm(fan_fd, metadata) 

struct fanotify_response response_struct;
response_struct.fd = metadata->fd;
response_struct.response = FAN_ALLOW;  // FAN_DENY
ret = write(fan_fd, &response_struct, sizeof(response_struct))
