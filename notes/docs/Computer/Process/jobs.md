# 任务

无论进程还是线程，其实都是以任务（jobs）的形态存在，任务是 Linux 里基本的调度单位

任务，可直译为作业，是一组相关进程的集合，通常由 Shell 命令启动

当 Shell 启动了某个任务，会调用某个系统 API（e.g. `tcsetpgrp`）把任务编号 PID 与 Shell 所属的终端关联起来

当终端需要发送信号时，会调用某个系统 API（e.g. `tcsetpgrp`）获取任务的编号

```shell
# 列出所有任务
jobs -l
# 默认只显示任务编号(job number)
#   -l显示任务对应的PID
#   -r只列出放在后台运行的
#   -s只列出放在后台暂停的

# 显示任务ID
$!

# 将任务丢到后台执行，在命令的最后加上&
# 但如果有stdout和stderr依旧会输出到前台，导致前台被影响，可以利用数据重定向将输出传送到文件中
cp file1 file2 > /tmp/log.txt 2>&1 &

# 如果想要脱机后依旧运行任务则需要使用nohup
nohup command  # 前台运行
nohup command &  # 后台运行
```

## job control

任务管理就是调用kill向进程发送一个信号

查看所有信号的编号和名字：`kill -l`

![20210826205955](http://image.zuoright.com/20210826205955.png)

进程对信号的处理其实就包括两个问题：

1. 进程如何发送信号：发送信号的系统调用是 kill()
2. 进程收到信号后如何处理：signal() 它可以给信号注册 handler

内核中对不同的信号有不同的缺省行为，一般会采用退出（terminate），暂停（stop），忽略（ignore）这三种行为中的一种。

![20210826212807](http://image.zuoright.com/20210826212807.png)

当我们运行 kill 1 这个命令的时候，希望把 SIGTERM 这个信号发送给 1 号进程，于是调用了kill()这个内核的调用接口（即系统调用），从而进入到了内核函数 sys_kill()，而内核在决定把信号发送给 1 号进程的时候，会调用 sig_task_ignored() 这个函数来做个判断，有以下三种情况：

1. 缺省(Default)，每个信号都有一个缺省行为，可使用`man 7 signal`查看，对于 SIGTERM 这个信号来说，它的缺省行为就是进程退出。
2. 捕获(Catch)，即进程可以在代码中针对某个信号，调用 signal() 注册相应的handler，这样进程在运行的时候，接收到指定信号，便不再使用 SIG_DFL 这个缺省的 handler，而是执行自定义的 handler。
3. 忽略(Ignore)，即不做任何处理

当 Linux 进程收到 SIGTERM 信号并且使进程退出，这时 Linux 内核对处理进程退出的入口点就是 do_exit() 函数，do_exit() 函数中会释放进程的相关资源，比如内存，文件句柄，信号量等等。

在做完这些工作之后，它会调用一个 exit_notify() 函数，用来通知和这个进程相关的父子进程等。

![20210829153533](http://image.zuoright.com/20210829153533.png)

### 特权信号

有两个特权信号：`SIGKILL` 和 `SIGSTOP`，它们是Linux 为 kernel 和超级用户去删除任意进程所保留的，不能被忽略，也不可以被捕获。

但对于初始化进程是可以忽略特权信号的，即 `kill -9 1` 是不会生效的

### 暂停任务

- 20 `SIGTSTP` 温和地暂停，执行Ctrl+z时即发送的就是这个信号
- 19 `SIGSTOP` 粗暴地暂停，不可屏蔽

```shell
kill -TSTP 进程编号
kill -STOP 进程编号
kill -CONT 进程编号  # 恢复进程

# 将任务暂并停放到后台
# 比如正在编辑一个文件或者执行的命令正在运行中时使用
Ctrl+z

# 让暂停到后台的任务在后台(background)继续运行
bg %job_id

# 将后台任务拿到前台(foreground)来执行
fg %job_id  # %可加可不加，为了与PID做区分建议加
```

### 终止任务

- 15 `SIGTERM` 以正常的进程方式结束一个任务（默认）
- 2 `SIGINT` 中断任务，完成善后工作再结束，执行Ctrl+c时即发送的就是这个信号
- 3 `SIGQUIT` 中断任务，结束前执行core dump操作，不常用
- 9 `SIGKILL` 立刻强制结束一个进程或任务（通常用来删除不正常的任务），不可屏蔽
- 1 `SIGHUP` 启动被终止的进程或任务，重新读取一次配置文件

```shell
# 发送信号
kill -signal_id_or_name 进程号
killall -signal_id_or_name 进程名称
# kill 与 killall 的区别是一个用进程号一个用进程名称
# 避免误杀同名进程，推荐优先使用kill

kill %job_id  # 默认发送编号为15的SIGTERM信号
kill -9 %job_id  # 指定发送编号为9的信号
kill -SIGKILL %job_id  # 指定发送名称为SIGKILL的信号
```

## 任务状态

![20210828204948](http://image.zuoright.com/20210828204948.png)

### 运行态(R stat)

- 运行中(即获得了CPU资源)
- 运行队列中

### 睡眠态(等待队列)

> 进程需要等待某个资源，可以是一个信号量(Semaphore), 或者是磁盘I/O

- TASK_INTERRUPTIBLE 可中断(S stat)
- TASK_UNINTERRUPTIBLE 不可中断(D stat)

### 退出态

- EXIT_ZOMBIE 进程退出前的状态（僵尸进程）
- EXIT_DEAD 进程结束退出时的状态

每一个 Linux 进程在退出的时候都会进入一个僵尸状态（EXIT_ZOMBIE），实际上它只是一个等待被父进程删除掉的条目，而它不是一个真正的进程，所以无法被杀死，而是需要父进程调用 `wait()` 或者 `waitpid()` 系统调用来清理，这也是容器中 init 进程必须具备的一个功能

> `wait()` 系统调用是一个阻塞的调用，也就是说，如果没有子进程是僵尸进程的话，这个调用就一直不会返回，那么整个进程就会被阻塞住，而不能去做别的事了  
> `waitpid()` 如果在调用的时候没有僵尸进程，那么函数就马上返回

## 孤儿进程

如果某个进程死了，而它的子进程还没死，那么这些子进程就被形象地称之为孤儿，然后会被初始进程领养，即变为初始进程的直接子进程

## 僵尸进程

单一僵尸进程虽然无害，但如果有过多的僵尸进程，就会消耗系统中的进程数资源，最坏的情况是导致新的进程无法启动。

```shell
# 给僵尸进程的父进程发送消息，让其清理掉子进程
kill -s SIGCHLD ppid
# 如果以上命令无效，则需要杀死父进程，使其变为孤儿进程，继而被init进程回收
# 注意，如果僵尸进程的父进程是1，请谨慎删除，否则将会重启
sudo kill -9 ppid
```
