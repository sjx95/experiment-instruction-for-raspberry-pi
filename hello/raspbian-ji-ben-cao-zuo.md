# RPi 基本操作

## 连接至 RPi 控制台

在之前的设置中，我们成功安装了镜像，开启了 SSH 服务，并将树莓派的有线网卡的 IP 配置为 `192.168.226.3`。现在可以用网线将计算机和树莓派连接，尝试登陆到树莓派的控制台了。

### 修改计算机的 IP 地址

_如果之前将 IP 修改为 10.X.Y.Z 的模式，则不需要此步骤。_

按照[此说明](https://jingyan.baidu.com/article/359911f5bafd5157fe03060a.html)修改 IP 地址，其中：

* IP 地址： 192.168.226.2
* 子网掩吗： 255.255.255.0
* 网关： 留空
* DNS： 留空

### 连通性测试

启动命令提示符，执行`ping 192.168.226.3`（或者之前设置的10.X.Y.Z），如果像下图一样收到了回应，则说明配置正确。

```
jxsong@SJX-Desktop:~/Documents> ping 192.168.226.3
PING 192.168.226.3 (192.168.226.3) 56(84) bytes of data.
64 bytes from 192.168.226.3: icmp_seq=1 ttl=64 time=1.13 ms
64 bytes from 192.168.226.3: icmp_seq=2 ttl=64 time=0.846 ms
64 bytes from 192.168.226.3: icmp_seq=3 ttl=64 time=1.11 ms
64 bytes from 192.168.226.3: icmp_seq=4 ttl=64 time=0.783 ms
^C
--- 192.168.226.3 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3010ms
rtt min/avg/max/mdev = 0.783/0.968/1.133/0.155 ms
```

### 通过 SSH 登陆

[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/) 是一个多用途的远程登陆软件，支持 SSH、Telnet、Serial 等方式。在 Host Name 中填写 192.168.226.3，然后 Open 即可。

树莓派默认的用户名为 pi，密码为 raspberry。

**注意：不要使用中文版，某些中文版实际上存在后门或病毒。**

![](/assets/PuTTY_Session.png)

登陆之后会出现一些提示信息，打开一个命令控制台。

![](/assets/RPi_logged_in.png)

## Linux 文件与目录管理

Ref: [Linux 文件与目录管理 | 菜鸟教程](http://www.runoob.com/linux/linux-file-content-manage.html)

Linux 的目录结构为树状结构，最顶级的目录为根目录 `/`。
计算机中的各个磁盘/分区，可以随时挂载在某一个目录下，也可以随时取消挂载。这样我们就可以磁盘中的所有文件了。

与 Windows 相同，Linux 可以使用绝对路径和相对路径。
- 绝对路径：由根目录 `/` 写起，如 `/usr/share/doc`
- 相对路径：相对于当前所处目录的路径，如 `src/hello.c`
- `~/`：表示用户主文件夹，在树莓派中为 `/home/pi/`
- `./`：表示当前所处目录的路径
- `../`：表示当前所处目录的上一层文件夹

### 基本文件命令
- `ls`: 列出目录中的内容
- `cd`: 切换目录
- `pwd`: 显示目前的目录
- `mkdir`: 创建一个新的目录
- `cp`: 复制文件，指定 `-r` 参数时可以复制目录
- `mv`: 移动文件或目录
- `rm`: 移除文件，指定 `-r` 参数时可以删除目录

### 文件查看命令
- `cat`: 将文件内容打印至控制台
- `less`: 可翻页地显示文件内容
- `head`: 将文件头几行打印至终端
- `tail`: 将文件最后几行打印至终端，如果指定 `-f` 参数则持续将持续打印新写入文件的内容

### 信号相关
- `Ctrl + C`: 发送 `SIGINT` 信号，一般为优雅地通知程序退出
- `Ctrl + Z`: 将进程挂起，可以之后用 `fg` 命令再次启动

## 文本编辑
由于 Linux 大多使用终端作为控制台，因此需要一个合适的编辑器。
VIM 是一个很好的选择，一方面它不需要图形界面，另一方面它经过了多年的发展已经有了很多插件，非常适合编写代码。然而 VIM 的使用较为复杂，也没有被包含在 RASPBIAN STRETCH LITE ，因此这里使用一个更为简单的工具 NANO。
在下一次的实验中，我们将介绍 VIM 的使用方法。

Ref: [nano命令](http://man.linuxde.net/nano)

启动方式: `nano <FILE_NAME>`

启动之后可以编写文本文件的内容，方向键控制移动，跟记事本差不多。
可以看到，下方有着操作提示符，其中 '^' 指 Ctrl 按键。

在编辑完成后，按下 `Ctrl + O` 来进入保存界面，直接按回车则保存在打开的文件中。

*更加复杂的使用见参考资料。*

