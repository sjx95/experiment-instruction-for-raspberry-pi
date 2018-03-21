# 安装 Raspbian 操作系统

Ref:[ https://www.raspberrypi.org/documentation/installation/installing-images/README.md](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

## 获取 Raspbian 映像

Download @[This page](https://www.raspberrypi.org/downloads/raspbian/). 使用 RASPBIAN STRETCH LITE 映像即可。

该系统不包含任何 GUI 组件支持（本课程实验也不需要），下载和烧写较快。  
如果后期需要 GUI 组件，可以在该系统上另行安装。

## 写入 Raspbian 映像

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

## 进行基本配置

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

