# Hello RPi

本实验的目的在于初步认识树莓派，知晓基本配置方法、如何连接到控制终端，并对 Linux 有一个基本的认识。

## 实验准备

本次实验需要如下设备：

- TF 卡读卡器一枚；
- 网线一根；
- 安装有 Windows/Linux 操作系统的计算机一台。

由于 ZJUWLAN 需要网页认证，且无法从外部查看树莓派分配到的 IP，因此无法通过无线的方式初始化树莓派。

如果存在难以解决的网络问题，教九 226 拥有具有合适的网络环境，请在这边完成该实验。

---

- 由于树莓派使用 Linux 系统，我们建议但不强制要求在 Linux 环境下完成本系列实验，这样可以方便很多。
- 请在前往实验室前，先邮件确认遇到的问题和我在实验室的时间。



## 安装 Raspbian 操作系统

Ref:[ https://www.raspberrypi.org/documentation/installation/installing-images/README.md](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

### 获取 Raspbian 映像

Download @[This page](https://www.raspberrypi.org/downloads/raspbian/). 使用 RASPBIAN STRETCH LITE 映像即可。

该系统不包含任何 GUI 组件支持（本课程实验也不需要），下载和烧写较快。  
如果后期需要 GUI 组件，可以在该系统上另行安装。

### 写入 Raspbian 映像

树莓派官方推荐使用 Etcher 来烧写映像，步骤如下：

1. 下载安装 [Etcher](https://etcher.io/) ；
2. 解开下载到的压缩档；
3. 将 TF 卡插入计算机；
4. 打开 Etcher，并选择要烧录的映像； 
5. 选择要烧写到的设备（注意千万不要选错）；
6. 启动烧写并等待完成。

_对于 Linux 用户：_  
`dd if=<IMAGE_PATH> of=/dev/sdX bs=4M; sync`

系统映像写入完成后，应当能够看到 TF 卡被分为两个分区：

* boot:
  * 分区大小约50MB；
  * FAT32 分区，Windows/Linux 下均可读写；
* rootfs \(Root File System\):
  * 该分区占满记忆卡剩余空间；
  * EXT4 分区，Linux 下可读写；

### 进行基本配置

Raspbian 映像默认配置下，需要外接显示器和 HDMI 才能登陆树莓派控制台，这显然很不方便，也不符合课程需要。  
因此需要在这里预先修改一部分系统配置。

Ref:  
[A SECURITY UPDATE FOR RASPBIAN PIXEL](https://www.raspberrypi.org/blog/a-security-update-for-raspbian-pixel/)  
[THE LATEST UPDATE TO RASPBIAN](https://www.raspberrypi.org/blog/another-update-raspbian/)

**注意： Windows 可能会帮你隐藏文件扩展名，请关掉此功能。**

1. 挂载 boot 分区；
2. 在 boot 分区下新建一个空的文本文档，并将其命名为 ssh；
3. 在 boot 分区下找到`cmdline.txt`，在文件开头写下`ip=192.168.226.3`（注意最后有个空格）；  
   该文件应该看起来像这样：  
   `ip=192.168.226.3 dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=d3aa9210-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait`

4. 弹出 TF 卡。

_备注：寝室申请过固定 IP 的同学，也可以将 IP 修改为 10.X.Y.Z，其中 X、Y 与申请到的 IP 一致，Z 不同。_

## RPi 基本操作

### 连接至 RPi 控制台

在之前的设置中，我们成功安装了镜像，开启了 SSH 服务，并将树莓派的有线网卡的 IP 配置为 `192.168.226.3`。现在可以用网线将计算机和树莓派连接，尝试登陆到树莓派的控制台了。

#### 修改计算机的 IP 地址

_如果之前将 IP 修改为 10.X.Y.Z 的模式，则不需要此步骤。_

按照[此说明](https://jingyan.baidu.com/article/359911f5bafd5157fe03060a.html)修改 IP 地址，其中：

* IP 地址： 192.168.226.2
* 子网掩吗： 255.255.255.0
* 网关： 留空
* DNS： 留空

#### 连通性测试

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

#### 通过 SSH 登陆

[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/) 是一个多用途的远程登陆软件，支持 SSH、Telnet、Serial 等方式。在 Host Name 中填写 192.168.226.3，然后 Open 即可。

树莓派默认的用户名为 pi，密码为 raspberry。

**注意：不要使用中文版，某些中文版实际上存在后门或病毒。**

![](/assets/PuTTY_Session.png)

登陆之后会出现一些提示信息，打开一个命令控制台。

![](/assets/RPi_logged_in.png)

### Linux 文件与目录管理

Ref: [Linux 文件与目录管理 \| 菜鸟教程](http://www.runoob.com/linux/linux-file-content-manage.html)

Linux 的目录结构为树状结构，最顶级的目录为根目录 `/`。  
计算机中的各个磁盘/分区，可以随时挂载在某一个目录下，也可以随时取消挂载。这样我们就可以磁盘中的所有文件了。

与 Windows 相同，Linux 可以使用绝对路径和相对路径。

* 绝对路径：由根目录 `/` 写起，如 `/usr/share/doc`
* 相对路径：相对于当前所处目录的路径，如 `src/hello.c`
* `~/`：表示用户主文件夹，在树莓派中为 `/home/pi/`
* `./`：表示当前所处目录的路径
* `../`：表示当前所处目录的上一层文件夹

#### 基本文件命令

* `ls`: 列出目录中的内容
* `cd`: 切换目录
* `pwd`: 显示目前的目录
* `mkdir`: 创建一个新的目录
* `cp`: 复制文件，指定 `-r` 参数时可以复制目录
* `mv`: 移动文件或目录
* `rm`: 移除文件，指定 `-r` 参数时可以删除目录

#### 文件查看命令

* `cat`: 将文件内容打印至控制台
* `less`: 可翻页地显示文件内容
* `head`: 将文件头几行打印至终端
* `tail`: 将文件最后几行打印至终端，如果指定 `-f` 参数则持续将持续打印新写入文件的内容

#### 文本编辑

由于 Linux 大多使用终端作为控制台，因此需要一个合适的编辑器。  
VIM 是一个很好的选择，一方面它不需要图形界面，另一方面它经过了多年的发展已经有了很多插件，非常适合编写代码。然而 VIM 的使用较为复杂，也没有被包含在 RASPBIAN STRETCH LITE ，因此这里使用一个更为简单的工具 NANO。  
在下一次的实验中，我们将介绍 VIM 的使用方法。

Ref: [nano命令](http://man.linuxde.net/nano)

启动方式: `nano <FILE_NAME>`

启动之后可以编写文本文件的内容，方向键控制移动，跟记事本差不多。  
可以看到，下方有着操作提示符，其中 '^' 指 Ctrl 按键。

在编辑完成后，按下 `Ctrl + O` 来进入保存界面，直接按回车则保存在打开的文件中。

_更加复杂的使用见参考资料。_

### 其他

* `Ctrl + C`: 发送 `SIGINT` 信号，一般为优雅地通知程序退出
* `Ctrl + Z`: 将进程挂起，可以之后用 `fg` 命令再次启动
* `exit` ：退出控制台

## Hello RPi

由于 Raspbian 是一个基于 Debian 定制的通用 Linux 系统，因此可以用任何通用的语言来编写代码，包括但不限于：

* C/C++
* Python2/Python3
* Java
* GoLang
* Shell Script

这里用几个非常简单的例子介绍一下最基本的编译方法。

### C/C++

C/C++ 是树莓派中常用语言之一，由于 Linux 及其 GNU/Linux 中的大量组件都是由 C/C++ 编写并导出了链接库，因此 C/C++ 非常适合用作树莓派的开发。此外， C/C++ 执行效率很高，能够充分利用硬件性能。

首先检查编译器是否存在及版本，使用命令 `g++ --version`，若返回信息如下，则说明正在使用 6.3 版本的 gcc。

```
pi@raspberrypi:~ $ g++ --version
g++ (Raspbian 6.3.0-18+rpi1+deb9u1) 6.3.0 20170516
Copyright (C) 2016 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

使用前文所述的 NANO 编辑器新建一个文件并写入如下内容：

```cpp
// Hello, RPi.
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, RPi." << endl;
    return 0;
}
```

然后使用命令 `g++ hello.cpp -o hello`进行编译，注意 Linux 系统中不要求程序具有扩展名。如果没有输出任何内容，那么说明 "0 Error, 0 Warning"，编译成功。接下来通过 `./hello`运行编译好的代码，如果没有意外的话：

```
pi@raspberrypi:~ $ ./hello 
Hello, RPi.
```

### Python3

Python 也是树莓派中的一种常用语言，尽管具有弱类型、低效率的缺点，但由于它的库很丰富，还可以无缝集成 C/C++ 编写的代码，因此也得到了广泛的应用。

_注意：无缝集成意味着可以先用 Python 进行开发，然后进行性能分析，对于关键部分用 C/C++ 重写。_

使用 NANO 将以下脚本写入 `hello.py`：

```py
#!/usr/bin/python3


def main():
        print('Hello RPi!')


if __name__ == "__main__":
        main()
```

运行有两种方式，第一种是显式指定 Python3 来处理这段脚本，即 `python3 hello.py`；第二种则告诉 Bash （即命令解释器）运行这个脚本，由于在第一行指明了需要 `/usr/bin/python3`来处理这段脚本，因此 Bash 会将它交给 Python3 解释器来处理，命令为：

```bash
chmod +x hello.py # 给予脚本执行权限
./hello.py
```

### Shell Script

类似于 Windows 下的 `.bat`  脚本，Bash 解释器也具有解释脚本的能力。Linux 中，许多软件的安装，环境的配置等都是基于 Shell Script 的。在这里我们编写一个最简单的 Bash Shell 来体验一下。

还是使用 NANO ，将以下代码写入 `hello.sh` ：

```bash
#!/bin/sh

echo "Hello, RPi!"
```

与 Python 脚本一样，可以通过 `bash hello.sh`  和 `./hello.sh` 两种方式执行脚本，不过要注意第二种需要赋予执行权限。


## 实验总结

在这次实验中，我们：

* 安装了 Raspbian 操作系统并进行了基本的配置；
* 了解了树莓派控制台的连接方法，了解了最基本的 Linux 文件及文件夹管理命令；
* 知道了在控制台中编写、编译、执行程序和脚本的基本方法。

有了这些基础后，我们将在下次实验中，学习如何用树莓派进行简单的图像处理。

