# IP Camera

在上一次实验中，我们学习了如何使用树莓派进行简单的图像处理。
这次实验中，我们将学习如何利用树莓派进行 TCP/IP 通信。

## 实验目的

- 了解 TCP/IP 协议族
- 掌握最基本的 Socket 网络编程模型
- 了解 ASIO 网络库

## 实验准备

- USB 摄像头 (教九 226 可借用 / 亦可将某一视频文件放入相应目录代替之)
- Visual Studio (>=2013) / Linux (Released after 2016)

## 实验流程

### 安装 ASIO 网络库

Boost 库是一个可移植、提供源代码的 C++ 库，作为标准库的后备，是 C++ 标准化进程的开发引擎之一。
Boost 库由 C++ 标准委员会库工作组成员发起，其中有些内容有望成为下一代 C++ 标准库内容。
而 ASIO 是 Boost 库的一部分，常用于处理网络通信或其他异步任务。

ASIO 分为两个版本，一个是 Boost/ASIO ，另一个是 ASIO(Non-Boost)。
这两者内容基本一致，只是 Boost 版本需要链接一些 Boost 库，Non-Boost 版本可以独立编译无需链接，但牺牲了一点编译速度。
ASIO 的[官方页面](http://think-async.com/)提供了下载、文档和相应的示例。

首先，我们先在树莓派上安装 ASIO 库，与 OpenCV 库一样， Raspbian 官方已经为我们预先打包了 ASIO 库，包名称为 `libasio-dev` ，因此使用命令 `sudo apt install libasio-dev` 即可让 APT 包管理程序为我们装好。

然后在本地安装 ASIO 库，如果正在使用 Linux 操作系统，那么一般可以使用包管理工具自动安装，包名一般为 `libasio-dev` `libasio-devel` 之类。

如果你正在使用 Windows 系统，需要在官方页面下载 Non-Boost 版本的库，并将其中的 include 文件夹解压至项目目录。
在编写 Server 端程序时，我们将使用这些的头文件。

### 安装 OpenCV

之前我们在树莓派上安装了 OpenCV 库，这次我们还需要在本地安装 OpenCV 库。
Linux 下安装比较简单，只需要安装相应的包就好了，一般会叫 `opencv-devel` `libopencv-dev` 或别的什么名字。

Windows 下推荐在 Visual Studio 下使用 OpenCV 库，安装可参考以下文章：  
[How to build applications with OpenCV inside the "Microsoft Visual Studio"](https://docs.opencv.org/3.4.1/dd/d6e/tutorial_windows_visual_studio_opencv.html)  
[OpenCV学习笔记（一）——OpenCV3.1.0+VS2015开发环境配置](https://www.cnblogs.com/linshuhe/p/5764394.html)

### 编写第一个网络通信程序

如果对 TCP/IP 协议族不了解，首先阅读[TCP/IP 教程](http://www.runoob.com/tcpip/tcpip-tutorial.html)。

在网络通信中，使用 Socket 来作为进行基本的通信接口。
当需要进行网络通信时，通常需要向操作系统申请一个 Socket ，然后将其绑定在某个端口上。
之后，只要告诉这个 Socket 与 TCP/IP 网络上的哪个 Socket 建立“链接”，并在 Socket 上执行读写即可。
在使用完毕后，还应该关闭这个 Socket ，释放系统资源。
无论是申请 Socket ，Socket 之间的数据传输、流控、拥塞控制，丢包重传等均由操作系统统一接管，不需要自行处理。

对于 TCP 的 Client 来说，通常是这样的一个流程：

- 申请一个 Socket
- 将 Socket 连接到指定目标（通常使用 IP 地址和端口号指定）
- 读写数据
- 关闭 Socket

而对于 TCP 的 Server 来说，一般这样处理：

- 申请一个 Socket
- 将 Socket 绑定到指定端口，并开始接受 Connection
- 接受到一个 Connecion 后，将 Connection 交给子程序处理
- 每一个子程序独立的进行一些读写操作
- 子程序关闭 Connection

更具体的，在网络编程有三种模型，分别为：

- 同步阻塞模型：在任务完成后函数才返回，一般需要启动多个线程来处理多个连接
- 同步非阻塞模型：在任务提交后函数就返回，需要随后检查标志位确定任务是否完成
- 异步：提交任务的同时指定一个回调函数，当任务完成后会自动调用回调函数惊醒下一步操作

这几中模型各有利弊，同步多线程模型编写简单，一般来说足以应付大多数情况。但同步多线程模型会造成许多不必要的线程切换，如果应用在高并发场合，另外两种方式会拥有更高的效率，但相应的也更难编写。

下面，我们编写一个简单的 Client 测试程序，该程序连接到指定端口并发送指定内容，然后断开连接。
其中，目标地址、目标端口、内容从启动参数中获取，示例代码如下：


```c++
#include <iostream>
#include <string>
#include <asio.hpp>

using namespace std;
using asio::ip::tcp;

int main(int argc, char *argv[]) {
        // Init IO_Service
        asio::io_service ios;
        // Resolve target address and port
        tcp::resolver resolver(ios);
        tcp::resolver::query query(argv[1], argv[2]);
        tcp::resolver::iterator endpoint_iterator = resolver.resolve(query);
        // Create and connect Socket
        tcp::socket socket(ios);
        asio::connect(socket, endpoint_iterator);
        // Send data
        socket.send(asio::buffer(string(argv[3])));
        // Close socket
        socket.close();
        return 0;
}

```

由于使用了 ASIO，在编译时需要引入对 pthread 的支持，因此编译命令为：`g++ test.cpp -o test -pthread`。

想要测试这个程序，我们需要 NetCat 工具包，使用命令 `sudo apt install netcat` 安装，然后打开一个新的 PuTTY 终端，使用命令 `nc -tln 8783` 在本地 8783 端口上建立监听。
现在回到之前的终端，运行我们的程序，即 `./test 127.0.0.1 8783 "RPi say hi"`，如果一切顺利的话，就可以看到另一个终端上接收到了 "RPi say hi" 的字符串。

### 更加合理的工程组织方式

到目前为止，我们每次编译都要敲一大堆命令，非常不方便。
另外，如果工程的规模一大，不仅很不方便，每次全部编译整个工程还十分费时，有时候，各模块之间的依赖往往也难以手动处理。

不过还好，我们有 make 这一自动化工具来帮助我们做这些事情，详细的说明见：
- [Make 命令教程 - 阮一峰](http://www.ruanyifeng.com/blog/2015/02/make.html)  
- [跟我一起写Makefile](http://wiki.ubuntu.org.cn/%E8%B7%9F%E6%88%91%E4%B8%80%E8%B5%B7%E5%86%99Makefile)  

Makefile 可以实现非常丰富的功能，从像本实验一样简单的项目，到 Linux 内核这样超大型的工程，均基于 Makefile 构建。
现在，新建一个工程文件夹，将 test.cpp 移动进去，然后在工程文件夹下新建 Makefile 文件，并写入以下内容：

```Makefile
# Setting OpenCV dependent library
OPENCV_LIBS = $(shell pkg-config --libs opencv)
# Setting ASIO dependent library
ASIO_LIBS = -lpthread

# Default target ip-camera
ip-camera: ip-camera.cpp
        ${CXX} -o ip-camera ip-camera.cpp ${OPENCV_LIBS} ${ASIO_LIBS}

# Add-on target test
test: test.cpp
        ${CXX} -o test test.cpp ${ASIO_LIBS}

```

这个 Makefile 定义了两个变量，用于指明连接哪些 LIB ，然后定义了两个目标：
一个是 ip-camera ，我们之后将编写它，它依赖于 ip-camera.cpp ；
另一个是 test ，即我们刚刚编写的那个文件，它依赖于 test.cpp 。

现在，在工程文件夹下执行命令 `make test` ，即可看到 make 根据 Makefile 的内容自动完成了编译。

### IP Camera

现在，是时候正式编写我们的 IP Camera 了。
为了简化编程，这里使用了阻塞同步的方式，并忽略了大部分的错误处理。

由于 TCP 传输的是无结构的字节流，因此需要我们规定一种数据传输的结构，好区分每一帧的图像。
一种最简单的方法是，每次建立链接，发送一帧图像，然后断开连接。
显然这种方法效率非常低，因此我们规定，在每一帧传输前，首先通知每一帧的长度，
这样我们就可以通过计算已接收的长度，得知一幅图像何时传输结束了。

源文件如下：

```c++
#include <iostream>
#include <string>
#include <vector>
#include <cstdint>
#include <opencv2/opencv.hpp>
#include <asio.hpp>

using namespace cv;
using namespace std;
using asio::ip::tcp;

int main(int argc, char *argv[]) {
    // Open camera
    VideoCapture vin(0);
    if (!vin.isOpened()) {
        cerr << "Failed opening camera." << endl;
        return -1;
    }
    
    // Init IO_Service
    asio::io_service ios;
    // Resolve target address and port
    tcp::resolver resolver(ios);
    tcp::resolver::query query(argv[1], argv[2]);
    tcp::resolver::iterator endpoint_iterator = resolver.resolve(query);
    // Create and connect Socket
    tcp::socket socket(ios);
    asio::connect(socket, endpoint_iterator);
    
    do {
        // Reading and encoding images
        Mat img;
        vin >> img;
        vector<uint8_t> T_imgBuffer;
        imencode(".png", img, T_imgBuffer);
        cerr << "T_imgBuffer.size() = " << T_imgBuffer.size() << endl;
        // Calculate data size
        vector<uint8_t> size;
        size.push_back(T_imgBuffer.size());
        size.push_back(T_imgBuffer.size() >> 8);
        size.push_back(T_imgBuffer.size() >> 16);
        size.push_back(T_imgBuffer.size() >> 24);
        // Send size and data
        try {
                socket.send(asio::buffer(size));
                socket.send(asio::buffer(T_imgBuffer));
        } catch (exception e) {
                break;
        }
        waitKey(100);
    } while (true);
    // Close socket
    socket.close();
    return 0;
}

```

现在，执行命令 `make` 即可，因为 ip-camera 写在第一个目标的位置，因此会被当作默认目标。
再次强调，这里忽略了大部分的错误处理，这在实际工程中可能造成程序崩溃。

下面来编写 Server 端，这里主要讲 Windows 的 Server 端， Linux 使用与 Windows 相同的代码即可，编译方式与树莓派类相似。

首先在Visual Studio工程中添加ASIO头文件的路径，具体方式为：  
Properties -> VC++ Directories -> Include Directories。
然后添加Server端文件：

```c++
#define	ASIO_STANDALONE
#include <iostream>
#include <string>
#include <asio.hpp>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;
using asio::ip::tcp;

int main() {
	asio::io_service io_service;
	tcp::acceptor acceptor(io_service, tcp::endpoint(tcp::v4(), 8783));

	namedWindow("Camera");

	tcp::socket socket(io_service);
	acceptor.accept(socket);

	do {
		vector<uint8_t> buffer(4);
		socket.receive(asio::buffer(buffer));

		size_t length = *((uint32_t *)&buffer[0]);
		cout << "length = " << length << ", " << flush;

		vector<uint8_t> img_data;
		buffer.resize(length);
		while (img_data.size() != length) {
			int recv_len = socket.receive(asio::buffer(buffer, length - img_data.size()));
			img_data.insert(img_data.end(), buffer.begin(), buffer.begin() + recv_len);
		}
		cout << "img_data.size() = " << img_data.size() << "." << endl;

		Mat img = imdecode(img_data, IMREAD_COLOR);
		imshow("Camera", img);
	} while (waitKey(1) != 'q');

	socket.close();

	return 0;
}

```

最后对这两个程序进行测试，首先在本地启动 Server 端程序，然后在树莓派上启动远程端，即 `./ipcamera 192.168.226.2 8783` ，如果一切顺利的话，应该在本地看到有如下显示：

![](/assets/cap_ipcamera.png)

## 总结

在本次实验中，我们：

- 安装了 ASIO 网络库
- 了解了 TCP/IP 的工作原理
- 进行了 TCP/IP 通信实验

相比于 STM32/51 等平台需要通过大量的寄存器操作，RPi 在处理网络 IO 时只需要跟 Socket 这一统一的结构体交互，在编码时非常方便。
同时，这也是嵌入式 Linux 的一大优势之一。

## 思考题
1. 结合 TCP 连接建立和断开的过程，分析一下为何每张图像建立一次连接很低效。
2. 将三种网络编程模型与 STM32 中串口通信的三种方式 (查询、中断、DMA) 做类比。
2. 为什么要用 TCP 而不是 UDP ，好处有什么？何时需要使用 UDP ？
3. 如果传输过程中发生丢包，造成不可恢复的 TCP 错误，会引起图像数据长度计算错误，进一步导致图像解码失败。
试问应当如何判断这一类错误，并从错误中恢复？
4. 参考 OpenCV 和 ASIO 的文档，以及本实验中的前几个程序，尝试为服务端程序加上注释。

