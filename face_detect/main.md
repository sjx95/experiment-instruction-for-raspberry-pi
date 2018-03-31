# 人脸检测

在上一次实验中，我们学习连接到了树莓派并编写了一个 Hello World! 程序。
在本次实验中，我们将学习如何借助树莓派进行简单的图像处理。

## 实验目的
- 掌握软件仓库的使用方法
- 学习 Linux 下编译程序的方法
- 能够读取摄像头数据
- 学会进行简单的图像处理

## 实验准备
- USB 摄像头 (教九 226 可借用，请先邮件确认)

## 实验流程

### 配置软件仓库

由于 GNU/Linux 主张开源，因此绝大多数的应用程序、库会通过源代码的形式发布。如果需要使用某些应用程序，需要依次将所依赖的库编译好，然后才能正确的编译应用程序。

这是由 Linux 开放的特性决定的，每个人使用的操作系统和环境都是经过一定的定制的，因此一般来说无法提供一个通用的二进制程序来满足所有用户需求。尽管这听起来很酷，但实际上会很麻烦，因此形成了某些标准的发行版，如 Debian / Red Hat / SuSE Linux ，并在软件仓库中放置了编译好的二进制包。

这样，我们就可以借助自动化工具来为我们解决包依赖问题，也不需要自行编译大部分开源项目了。

注意：如果有连接即可用的 WLAN ，按 [说明](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md) 连接即可，无需进行本步骤。

由于无法为树莓派联网，这里需要经由主机连接至软件源。一般来说，有如下几个途径：
- Mirrors - 将整个软件源下载到本地，需要花费大量时间和磁盘空间为软件源做镜像，不合适；
- Reverse Proxy - 反向代理，将 HTTP 请求反向代理到镜像仓库，但配置过于繁琐；
- 端口转发 - 将发往某个端口的数据转发至镜像仓库，较为简单。

这里采用第三种方法。
将网线连接好， IP 设置完毕后，启动树莓派。在 PuTTY 登陆前，找到 SSH Tunnel 界面，如下所示：

![PuTTY Forwarding](/assets/PuTTY_Forward.png)

SSH 支持三种转发方式，Local/Remote/Dynamic，这里使用 Remote 方式将树莓派上的 8080 端口转发至 mirrors.zju.edu.cn:80。
因此在 Source port 处填写 8080，Destination 处填写 mirrors.zju.edu.cn:80 ，方向选择 Remote ，之后就可以连接了。

*SSH 转发功能详细介绍： [https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/)。*

Linux 用户注意： `ssh -R 8080:mirrors.zju.edu.cn:80 pi@192.168.226.3`

下述内容需要 root 权限，在启动 nano 进行文本编辑时，请用 sudo ，如 `sudo nano /etc/hosts`。

在连接至树莓派控制台后，需要“欺骗”树莓派让他认为本机即是 mirrors.zju.edu.cn ，因此在 `/etc/hosts` 中加入 

```
127.0.0.1	mirrors.zju.edu.cn
```

之后测试是否能够建立 HTTP 请求，使用命令 `curl mirrors.zju.edu.cn:8080` 来进行，如果出现了一大堆 HTML 代码，而不是错误提示什么的，说明端口映射成功了。

接下来我们指定包管理程序 APT 从该处获取包，该文件在 `/etc/apt/source.list`，修改后看起来应该像这样：
```
deb http://mirrors.zju.edu.cn:8080/raspbian/raspbian/ stretch main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://archive.raspbian.org/raspbian/ stretch main contrib non-free rpi
```
然后将 `/etc/apt/source.list.d/raspi.list` 重命名为 `raspi.list.bak`。*想一想，重命名应该使用哪条命令？*

接下来尝试获取软件仓库清单，使用命令 `sudo apt update`，如果像下面这样没有报错，说明配置成功了。
```
pi@raspberrypi:~ $ sudo apt update
Hit:1 http://mirrors.zju.edu.cn:8080/raspbian/raspbian stretch InRelease
Reading package lists... Done
Building dependency tree       
Reading state information... Done
7 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

### 安装 OpenCV 开发包及 Nginx 引擎
OpenCV 是一个开源的计算机视觉库，常用于图像处理方面。
Nginx 则是一个 HTTP 服务器，在这里用于在主机的浏览器中显示图片。

使用命令 `sudo apt install libopencv-dev` 来安装 OpenCV ，APT 包管理工具会自动安装依赖的包。
同理，使用命令 `sudo apt install nginx` 来安装 Nginx。

现在，启动 Chrome 浏览器，进入隐身浏览模式，键入 `http://192.168.226.3/ `，应该能够看到 Nginx 的欢迎页面了。

### 借助 OpenCV 读取摄像头

在准备好环境后，我们先来编写一个简单的小程序来测试整个环境能否正常工作。
在这个程序中，我们将：

- 打开摄像头
- 从摄像头读取画面
- 将画面写入 HTTP 目录下（Nginx 默认目录 `/var/www/html/`）
- 在主机上用浏览器读取图片

参考：  
[OpenCV documentation Index](https://docs.opencv.org/2.4.9/)  (2.4.X版本的文档基本通用)  
[Reading and Writing Images and Video](https://docs.opencv.org/2.4.9/modules/highgui/doc/reading_and_writing_images_and_video.html)


```c++
// cap.cpp
#include <iostream>
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace std;
using namespace cv;

int main() {
    Mat frame;

    VideoCapture cap(0);
    if (!cap.isOpened()) {
	cerr << "Failed open carema." << endl;
        return -1;
    }
    while (true) {
        cap >> frame;
        imwrite("/var/www/html/cap.png", frame);
    }
    return 0;
}
```

下面对源代码进行编译。
与上一个实验不同，由于我们调用了OpenCV，因此需要链接OpenCV的链接库。
在 GCC 中，参数 -l 用于指定链接库，如 -lm 指的 libm.so（即去掉链接库文件名的lib和后缀名）。
这里我们使用了 opencv\_core 和 opencv\_hightui 组件，因此需要使用 `-lopencv_core -l opencv_highgui` 进行正确链接。

参考：  
[C 语言编译过程详解](https://www.cnblogs.com/CarpenterLee/p/5994681.html)  
[GCC 编译命令](https://www.cnblogs.com/ibyte/p/5828445.html)  
[gcc的使用简介与命令行参数说明](http://www.cnblogs.com/testlife007/p/6555404.html)  
[GCC and Make: Compiling, Linking and Building C/C++ Applications](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html)  

因此编译命令为：

```bash
g++ cap.cpp -lopencv_core -lopencv_highgui -o cap
```

如果没有意外的话，我们可以在浏览器中打开 `http://192.168.226.3/cap.png ` 并看到画面了，这时周期性的按下刷新按钮并移动摄像头，可以看到图片会有一定的变化。
按下 `Ctrl + C` 则可以中断程序运行。

注意：由于没有关闭摄像头，下一次程序运行时可能无法打开摄像头，这时将摄像头拔出并重新插入，或者直接重启树莓派即可。

### 检测画面中的人脸

在这一环节中，我们将编写一个函数，用于检测给定图片中的人脸，并用矩形框框起来。

OpenCV为我们提供了串级分类器，并预置了用于人脸检测的模板，这里我们将使用这个函数来进行人脸检测。
因此对每一帧的图像处理如下：

- 获取图像
- 转换为灰度模式
- 执行人脸检测，获取人脸区域信息
- 在人脸区域绘制矩形框
- 将图像写入HTTP目录下

首先编写人脸检测的函数，

```c++
// detect.cpp
#include <iostream>
#include <opencv2/objdetect/objdetect.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>

using namespace std;
using namespace cv;

Mat detect(Mat img) {
        if (img.cols == 0 || img.rows == 0)
                return img;

        CascadeClassifier faceCascade("/usr/share/opencv/haarcascades/haarcascade_frontalface_alt2.xml");
        
        Mat imgGray;
        vector<Rect> faces;
        
        cvtColor(img, imgGray, CV_RGB2GRAY);
        faceCascade.detectMultiScale(imgGray, faces, 1.2, 6, 0, Size(0, 0));
        cout << "Detect finished, " << faces.size() << " face(s) found." << endl;
        
        for(int i =0; i<faces.size(); i++) {
                rectangle(img, Point(faces[i].x, faces[i].y), 
                                Point(faces[i].x + faces[i].width, faces[i].y + faces[i].height), 
                                Scalar(0, 255, 0), 1, 8); 
        }
        return img;
}
```

然后编写相应的头文件：

```c++
// detect.h
#pragma once
#include <opencv2/core/core.hpp>

cv::Mat detect(cv::Mat img);
```

最后，在上述的 `cap.cpp` 中引用刚刚编写的`detect.h`，并在获取图像之后，输出图像之前，加上：

```c++
frame = detect(frame);
```

接下来就可以进行编译并测试了，由于这里使用了 opencv\_imgproc 和 opencv\_objdetect 这两个组件，因此除了上个环节链接的库外，还需要 `-lopencv_imgproc`、`-lopencv_objdetect` 这两个参数。
此外，除了 cap.cpp 外，detect.cpp 也需要一起编译。

运行程序后，不停刷新浏览器，观察是否能框住出现的人脸。
预置的分类器性能会比较差，可能需要避免晃动并多尝试几个角度。
人脸检测的效果如图琐事：

![](/assets/cap.png)

## 实验总结

在本次实验中，我们：
- 连接到了 Raspbian 软件仓库
- 安装了 OpenCV 开发环境和 Nginx HTTP 服务器
- 读取了摄像头数据
- 尝试进行了简单的人脸检测

尽管跟通用计算机相比，人脸检测的执行速度还是很慢，但已经大大优于 STM32 F4/F7 系列的设备了。
因此在需要一定规模计算的嵌入式应用中，除了借助与上位机通信外，还可以考虑使用 Raspberry Pi 之类的方案来解决。

在下一次实验中，我们将介绍 Raspberry Pi 和 嵌入式 Linux 的另一优势，网络栈。

## 思考题

1. g++ 和 gcc 有什么区别？如何查看 g++ 的版本？
2. 为什么有时候浏览器刷新后，看到的图片不完整？
3. 为什么人脸检测实验中，从摄像头中读取图片要读30次？如果不这么做会发生什么？
4. 人脸检测的过程执行的很慢，有什么简单可行的办法可以提高速度？

