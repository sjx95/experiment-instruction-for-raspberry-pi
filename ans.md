# 思考题参考答案

本份参考答案仅供参考，只列出了关键点，如有异议欢迎讨论。

## 人脸检测

1. g++ 和 gcc 有什么区别？如何查看 g++ 的版本？  
g++ 是 C++ 的编译器，而 gcc 是 C 的。  
`g++ --version`
2. 为什么有时候浏览器刷新后，看到的图片不完整？  
当浏览器刷新时，树莓派上的 nginx 会读取相应的文件。由于没有加读写锁，这时人脸识别程序可能尚未完整写入图片。
3. 为什么人脸检测实验中，从摄像头中读取图片要读30次？如果不这么做会发生什么？  
不知道有没有人注意到，浏览器刷新出的图片可能会有较高的延迟。这是因为摄像头是具有缓存的，需要读一定的次数并丢弃，把缓存读空，才能拿到最新的画面。
4. 人脸检测的过程执行的很慢，有什么简单可行的办法可以提高速度？  
方法比较多，比如降采样可以一定程度上缓解这个问题，有效利用多线程也是个可行的方案。如果还是不行，可以将图片发送至上位机处理然后获取结果。

## IP Camera

1. 结合 TCP 连接建立和断开的过程，分析一下为何每张图像建立一次连接很低效。  
TCP 在建立连接时，需要经过三次握手，这一过程会造成一定的延迟。另外维护每个 TCP 连接都需要占用一部分系统资源。
2. 将三种网络编程模型与 STM32 中串口通信的三种方式 (查询、中断、DMA) 做类比。  
这里以串口接受为例：  
  - 同步阻塞：不停查寄存器标志位，直到有数据进来  
  - 同步非阻塞：看一下寄存器标志位，如果没有就算了，有空时再来查看
（注意 STM32 串口没有缓存，这样会丢失数据）  
  - 异步：类似中断，设置一个回调函数，即类比中断向量，每次有新数据即触发该回调函数。
3. 为什么要用 TCP 而不是 UDP ，好处有什么？何时需要使用 UDP ？  
TCP 是一种可靠传输，保证了信息的完整和有序。 UDP 不保证数据到达，更不保证数据有序，但可以提供更高的实时性。
4. 如果传输过程中发生丢包，造成不可恢复的 TCP 错误，会引起图像数据长度计算错误，进一步导致图像解码失败。
试问应当如何判断这一类错误，并从错误中恢复？  
由于编码方式无法确认真正长度和两包之间的间隔，一旦发现解码失败，只能断开 TCP 连接并重连。
5. 参考 OpenCV 和 ASIO 的文档，以及本实验中的前几个程序，尝试为服务端程序加上注释。  
略。

## kmod-lsprocesses

1. 内核模块一定要有 init\_module 和 cleanup\_module 吗？什么情况下可以没有？  
一定。由于内核模块一般需要将相应的函数注册到相应的事件下，因此必须要进行初始化，同理也必须要做清理工作。
2. 简要回答什么是内核态，什么是用户态？它们最明显的区别是什么？  
前略。最主要的区别是，内核态能够运行处理器特权指令，用户态不能，只能通过 Trap 来请求操作系统服务。
3. 为什么内核模块不能够使用 printf 、 sleep 之类的函数？  
printf 之类的函数定义在 glibc 库中，这个库是给用户态程序调用的，它会引发 Trap 并请求操作系统服务。但内核模块已经工作在内核态了，所以不应该也不能去调用用户态的库。
4. 加载 lsprocess 模块后，每次从 /proc/psinfo 中读出的信息相同吗？为什么？  
不一定，系统的状态可能会发生变化，例如某个程序执行完毕并退出等。
5. 注释掉 lsprocess 模块源码中移除 /proc/psinfo 的一行，重新编译并加载内核，测试功能是否正常。 将模块卸载后，该文件还在吗？读取这一文件会出现什么现象？对其进行简要分析。 （建议在完成所有其他实验内容，妥善截图并写好报告后再做尝试。 如果出现了失去控制的错误，先尝试断电重启，如果还是不行就只能重新制作 SD 卡了。）  
这一行为一般会发生严重错误，由于卸载内核模块时没有清理掉注册的回调函数，但回调函数本身在内存中的代码段已经随着模块卸载而释放掉了，因此当读取 /proc/psinfo 时会发生严重错误。在我的版本上，会触发内存越界并进一步引发保护，操作系统和内核正常执行。对于某些特定的回调函数，或特定的内核版本，可能会造成整个内核的崩溃。

\- EOF -
 
