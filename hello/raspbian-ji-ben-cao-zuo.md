# RPi基本操作

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

