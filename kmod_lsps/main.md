# kmod-lsprocesses

本次实验由 Jim Zhong (@github/jimzhong) 提供参考代码。

## 实验目的

- 了解内核模块
- 学会编写最简单的内核模块
- 尝试编写一个内核模块，能够通过 /proc/psinfo 列出当前运行的所有进程，并对各状态的进程进行统计。

## 理论基础

本次实验需要一定的有关 Linux 内核的基础知识，如果您不能回答思考题中的问题，请先带着问题阅读以下材料：  
[IBM developerWorks: Linux 内核剖析](https://www.ibm.com/developerworks/cn/linux/l-linux-kernel/)  
[IBM developerWorks: Linux 可加载内核模块剖析](https://www.ibm.com/developerworks/cn/linux/l-lkm/)

## 实验步骤

### 配置镜像源

在第二次实验中，我们只配置了 Raspbian 的镜像源，这个镜像源中包含了由 Raspbian 衍生自 Debian 的软件包。
由于内核相关的包是由 Raspberry Pi 官方构建的，而不是来自 Debian ，因此我们需要将这个新的源也加入列表。
受限于使用范围，这个源没有对世界范围的线路进行优化，镜像也比较少。
因此我在我自己的服务器上配置了反向代理，也就能够相对快速的获取软件包了。

参考之前配置的方法，在连接之前，添加一个端口映射，将 8081 映射到 himmel.tech:80 ，然后连接到树莓派，将之前重命名过的 `raspi.list.bak` 重命名回 `raspi.list`，并修改内容为：

```
deb http://127.0.0.1:8081/debian/ stretch main ui
```

然后运行 `sudo apt update` 来更新软件源列表，在列表更新完毕后，
使用 `sudo apt install raspberrypi-kernel` 来将内核更新到最新版本，
然后安装 raspberrypi-kernel-header 这个内核编译支持包。

### 内核模块的使用

与内核模块相关的几个主要命令有：

- insmod - 插入内核模块
- rmmod - 移除内核模块
- modinfo - 打印内核模块信息
- lsmod - 列出已载入的模块
- modprobe - 另一个加载/卸载内核模块的工具

详细的使用方法，参照参考资料或 Manual 手册 (例如 `man insmod`)。

参考资料：  
[CHAPTER 31. WORKING WITH KERNEL MODULES](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/ch-working_with_kernel_modules)

### 编译第一个内核模块

下面，我们来编写第一个内核模块，在这个模块中，我们只做一件事：加载和卸载时在内核日志中输出问候信息。

内核模块与用户态程序的执行方式不同，因此程序的结构也不太一样。
回顾一下之前写过的程序，无论是执行什么样的功能，都必须有一个 main 函数。
当我们要求操作系统调入并执行这个程序的时候，代码将从 main 函数开始执行，即程序沿着一个非常明显的控制流执行。

内核模块则不一样，它没有 main 函数，也没有一个明显的控制流，而是由一组回调函数构成，
每一个回调能够处理某一类具体的事件，如 GPIO 中断发生、某个设备文件被访问等。
其中，init\_module 和 cleanup\_module 是两个最重要的函数，它们分别处理模块被加载和卸载的事件。
一般来说，在模块被加载时，会进行一些初始化操作，并在系统中将回调函数注册到相应的事件上，然后返回。
如果之后系统中发生了注册过的事件，那么相应的回调函数则会被执行。
当模块被卸载时，则需要将注册过的回调函数撤销掉，否则在事件再次发生时会产生未定义的错误。

这些差异与内核模块和用户空间程序的设计目的不同有关。
对于用户空间程序，其设计目的在于完成某种任务，比如计算圆周率、执行人脸检测等，因此存在一个具体的执行流程。
而内核模块则不是这样，内核模块用于控制硬件，并为用户空间程序提供服务。
因此，内核模块实际上是乱序执行的，因为无法确定用户空间的程序何时会请求服务，
或来自硬件设备的某个事件何时发生，甚至于一个函数可能会被同时多次执行。
因此，内核模块与普通用户程序的结构也就很不一样。

更多的内容，请阅读相关资料：  
[The Linux Kernel Module Programming Guide](http://www.tldp.org/LDP/lkmpg/2.6/html/)

下面我们来开始正式编写第一个内核模块，示例程序如下：

```C
#include <linux/kernel.h>
#include <linux/module.h>

int init_module(void) {
  printk(KERN_INFO "kmod say hi\n");
  return 0;
}
void cleanup_module(void) {
  printk(KERN_INFO "kmod say bye\n");
}

```

与普通的应用程序不同，内核模块不允许去调用任何 glibc 之类的用户空间的库，也不能去包含这些头文件，
因此我们需要告诉编译器，不要到默认路径找头文件，也不要链接默认的库，而是到内核源码目录下去找这些库。
显然，这是一件很麻烦的事情，因为内核源码中的文件数量是成千上万的。
不过幸好内核开发人员已经考虑过这个问题了，我们只需要在 Makefile 再调用一次 make ，
用 `-C` 告诉 make 程序到这里读取内核源码中提供的 Makefile ，
然后用 `M=$(shell pwd)` 来告诉 make 继续执行本目录的 Makefile。

因此整个 Makefile 看起来应该像这样：

```Makefile
obj-m := lsprocess.o			# Target

KVERSION := $(shell uname -r) 		# Check kernel version
KDIR := /lib/modules/$(KVERSION)/build	# Define kernel modules source dir
PWD := $(shell pwd)			# Define current dir

default:
        $(MAKE) -C $(KDIR) M=$(PWD) modules

clean:
        $(MAKE) -C $(KDIR) M=$(PWD) clean

install:
        $(MAKE) -C $(KDIR) M=$(PWD) modules_install

```

下面我们要用 `insmod` 命令将内核插入，再用 `rmmod` 命令将其卸载（注意需要 sudo 命令提权）。
之后使用 `dmesg` 命令打印内核日志，可以在其中看到类似下面的输出。

```
[18872.035441] kmod say hi
[18876.127173] kmod say bye
```

### 实现 kmod-lsprocess

下面，我们正式开始实现 kmod-lsprocess 的功能。需要实现的函数有如下几个：

- 初始化函数
  - 创建 /proc/psinfo 文件，并设置回调函数
  - 打印日志，告知初始化已完成
- 注销函数
  - 撤销回调函数，移除 /proc/psinfo 文件
  - 打印日志，告知注销已完成
- 触发读 /proc/psinfo 时的回调函数
  - 遍历 Linux 进程列表，输出进程信息至文件缓冲区并统计
  - 输出统计信息至缓冲区

其中，在用户态与内核态交互上，我们使用了 /proc 文件系统，
参考资料如下：  
[IBM developerWorks: 使用 /proc 文件系统来访问 Linux 内核的内容](https://www.ibm.com/developerworks/cn/linux/l-proc.html)

下面我们给出这一模块的参考代码：

```C
/*
This source code is originally written by Jim Zhong and posted on GitHub
*/


#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/proc_fs.h>
#include <linux/seq_file.h>
#include <linux/sched.h>
#include <linux/sched/signal.h>

MODULE_LICENSE("GPL");

// print all processes
// m: pointer to seq_file
// v: not used
static int show_process(struct seq_file *m, void *v)
{
    // pointer to task_struct, use in iteration
    struct task_struct *p;
    // buffer for process command name and parent process command name
    char name[TASK_COMM_LEN], pname[TASK_COMM_LEN];

    // below are counters for processes in different states
    long running = 0;
    long inter = 0;
    long uninter = 0;
    long zombie = 0;
    long stop = 0;
    long traced = 0;
    // process in other states
    long other = 0;
    // total process counter
    long total = 0;

    int i;
    // State in C style string
    const char* str_state;

    // ======
    for (i=0; i<80; ++i) 
        seq_printf(m, "=");
    seq_printf(m, "\n");
    // Print header
    seq_printf(m, "%-5s | %-30s | %-15s | %s \n", "PID", "NAME", "STAT", "PARENT");
    // ------
    for (i=0; i<80; ++i) 
        seq_printf(m, "-");
    seq_printf(m, "\n");
    
    // iterate through all processes
    for_each_process(p)
    {
        total++;
        // copy the process' command name into name buffer
        get_task_comm(name, p);
        if (p->parent)
            // copy the process' parent's command name into name buffer if there is a parent
            get_task_comm(pname, p->parent);
        else
            strcpy(pname, "None");
        // increment counters according to process state
        switch(p->state)
        {
            case TASK_RUNNING:
                running++;
                str_state = "RUNNING";
                break;
            case TASK_INTERRUPTIBLE:
                str_state = "INTERRUPTIBLE";
                inter++;
                break;
            case TASK_UNINTERRUPTIBLE:
                str_state = "UNINTERRUPTIBLE";
                uninter++;
                break;
            case TASK_WAKEKILL | __TASK_STOPPED:    // a stopped process has both flags set
                stop++;
                str_state = "STOPPED";
                break;
            case TASK_WAKEKILL | __TASK_TRACED:
                traced++;
                str_state = "TRACED";
                break;
            default:
                str_state = "OTHERS";
                other++;
        }
"for_each_process"        // zombie state is stored in p->exit_state
        if (p->exit_state & EXIT_ZOMBIE)
            str_state = "ZOMBIE";
        // print to seq_file m
        seq_printf(m, "%5d | %-30s | %-15s | %s \n", p->pid, name, str_state, pname);
    }
    // ------
    for (i=0; i<80; ++i) 
        seq_printf(m, "-");
    seq_printf(m, "\n");
    // print statistical information to seq_file
    seq_printf(m, "RUNNING:%ld \tINTERRUPTIBLE: %ld\tUNINTERRUPTIBLE: %ld\tSTOPPED: %ld\nTRACED: %ld\tZOMBIE: %ld\t\tOTHERS: %ld\nTOTAL: %ld\n",
        running, inter, uninter, stop, traced, zombie, other, total);
    // ======
    for (i=0; i<80; ++i) 
        seq_printf(m, "=");
    seq_printf(m, "\n");
    return 0;
}

// callback for open
static int ls_process_open(struct inode *inode, struct file *file)
{
  return single_open(file, show_process, NULL);
}

// file operations
static struct file_operations fops = {
    .owner    = THIS_MODULE,
    .open = ls_process_open,    // point to open routine
    .read = seq_read,           // seqfile read handler
    .llseek = seq_lseek,        // seqfile seek handler
    .release = seq_release,     // seqfile close handler
};

// module init handler
static int __init ls_process_init(void)
{
    // create /proc/psinfo directory
    struct proc_dir_entry *entry = proc_create("psinfo", 0666, NULL, &fops);
    if (entry == NULL)
    {
        // error handling
        printk(KERN_ERR "Couldn't create proc entry\n");
        return -ENOMEM;
    }
    // load successfully
    printk(KERN_INFO "Module loaded successfully\n");
    return 0;
}

// module exit callback
static void __exit ls_process_exit(void)
{
    // remove /proc/psinfo
    remove_proc_entry("psinfo", NULL);
    // print message
    printk(KERN_INFO "Module unloaded successfully\n");
}

module_init(ls_process_init);   // register loading handler
module_exit(ls_process_exit);   // register unloading handler

```

依照前述方法，编译该内核模块，然后将其加载入内核。
现在，用命令 `cat /proc/psinfo` 来访问内核模块给出的文件接口，可以看到形如下表的进程列表。

```
================================================================================
PID   | NAME                           | STAT            | PARENT 
--------------------------------------------------------------------------------
    1 | systemd                        | INTERRUPTIBLE   | swapper/0 
    2 | kthreadd                       | INTERRUPTIBLE   | swapper/0 
    4 | kworker/0:0H                   | OTHERS          | kthreadd 
    6 | mm_percpu_wq                   | OTHERS          | kthreadd 
    7 | ksoftirqd/0                    | INTERRUPTIBLE   | kthreadd 
    8 | rcu_sched                      | OTHERS          | kthreadd 
    9 | rcu_bh                         | OTHERS          | kthreadd 
   10 | migration/0                    | INTERRUPTIBLE   | kthreadd 
	.	.	.
	.	.	.
	.	.	.
 1220 | kworker/3:0                    | OTHERS          | kthreadd 
 2540 | kworker/0:1                    | OTHERS          | kthreadd 
 3676 | kworker/0:2                    | OTHERS          | kthreadd 
 3967 | kworker/u8:1                   | OTHERS          | kthreadd 
 4597 | kworker/u8:0                   | OTHERS          | kthreadd 
 4610 | kworker/0:0                    | OTHERS          | kthreadd 
 4934 | cat                            | RUNNING         | bash 
--------------------------------------------------------------------------------
RUNNING:1       INTERRUPTIBLE: 54       UNINTERRUPTIBLE: 0      STOPPED: 0
TRACED: 0       ZOMBIE: 0               OTHERS: 41
TOTAL: 96
================================================================================
```

最后不要忘了用 `rmmod` 将加载过的内核模块卸载掉。


## 思考题

1. 内核模块一定要有 init\_module 和 cleanup\_module 吗？什么情况下可以没有？
2. 简要回答什么是内核态，什么是用户态？它们最明显的区别是什么？
3. 为什么内核模块不能够使用 printf 、 sleep 之类的函数？
4. 加载 lsprocess 模块后，每次从 /proc/psinfo 中读出的信息相同吗？为什么？
5. 注释掉 lsprocess 模块源码中移除 /proc/psinfo 的一行，重新编译并加载内核，测试功能是否正常。
将模块卸载后，该文件还在吗？读取这一文件会出现什么现象？对其进行简要分析。
（建议在完成所有其他实验内容，妥善截图并写好报告后再做尝试。
如果出现了失去控制的错误，先尝试断电重启，如果还是不行就只能重新制作 SD 卡了。）

