# 《TCP/IP网络编程》学习笔记

:flags:此仓库是我的《TCP/IP网络编程》学习笔记及具体代码实现，代码部分请参考本仓库对应章节文件夹下的代码。

我的环境是：Ubuntu18.04 LTS

编译器版本：`g++ (Ubuntu 7.3.0-27ubuntu1~18.04) 7.3.0` 和 `gcc (Ubuntu 7.3.0-27ubuntu1~18.04) 7.3.0`

所以本笔记中只学习有关于 Linux 的部分。

## 第 1 章：理解网络编程和套接字

本章代码，在[TCP-IP-NetworkNote](https://github.com/riba2534/TCP-IP-NetworkNote)中可以找到，直接点连接可能进不去。

### 1.1 理解网络编程和套接字

#### 1.1.1构建打电话套接字

以电话机打电话的方式来理解套接字。

**调用 socket 函数（安装电话机）时进行的对话**：

> 问：接电话需要准备什么？
>
> 答：当然是电话机。

有了电话机才能安装电话，于是就要准备一个电话机，下面函数相当于电话机的套接字。

```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
//成功时返回文件描述符，失败时返回-1
```

**调用 bind 函数（分配电话号码）时进行的对话**：

> 问：请问我的电话号码是多少
>
> 答：我的电话号码是123-1234

套接字同样如此。就想给电话机分配电话号码一样，利用以下函数给创建好的套接字分配地址信息（IP地址和端口号）：

```c
#include <sys/socket.h>
int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);
//成功时返回0，失败时返回-1
```

调用 bind 函数给套接字分配地址之后，就基本完成了所有的准备工作。接下来是需要连接电话线并等待来电。

**调用 listen 函数（连接电话线）时进行的对话**：

> 问：已架设完电话机后是否只需链接电话线？
>
> 答：对，只需要连接就能接听电话。

一连接电话线，电话机就可以转换为可接听状态，这时其他人可以拨打电话请求连接到该机。同样，需要把套接字转化成可接受连接状态。

```c
#include <sys/socket.h>
int listen(int sockfd, int backlog);
//成功时返回0，失败时返回-1
```

连接好电话线以后，如果有人拨打电话就响铃，拿起话筒才能接听电话。

**调用 accept 函数（拿起话筒）时进行的对话**：

> 问：电话铃响了，我该怎么办？
>
> 答：接听啊。

```c
#include <sys/socket.h>
int accept(int sockfd,struct sockaddr *addr,socklen_t *addrlen);
//成功时返回文件描述符，失败时返回-1
```

网络编程中和接受连接请求的套接字创建过程可整理如下：

1. 第一步：调用 socket 函数创建套接字。
2. 第二步：调用 bind 函数分配IP地址和端口号。
3. 第三步：调用 listen 函数转换为可接受请求状态。
4. 第四步：调用 accept 函数受理套接字请求。

#### 1.1.2  编写`Hello World`套接字程序

**服务端**：

服务器端（server）是能够受理连接请求的程序。下面构建服务端以验证之前提到的函数调用过程，该服务器端收到连接请求后向请求者返回`Hello World!`答复。除各种函数的调用顺序外，我们还未涉及任何实际编程。因此，阅读代码时请重点关注套接字相关的函数调用过程，不必理解全过程。

服务器端代码请参见：[hello_server.c](ch01/hello_server.c)

**客户端**：

客户端程序只有`调用 socket 函数创建套接字` 和 `调用 connect 函数向服务端发送连接请求`这两个步骤，下面给出客户端，需要查看以下两方面的内容：

1. 调用 socket 函数 和 connect 函数
2. 与服务端共同运行以收发字符串数据

客户端代码请参见：[hello_client.c](ch01/hello_client.c)

**编译**：

分别对客户端和服务端程序进行编译：

```shell
gcc hello_server.c -o hserver
gcc hello_client.c -o hclient
```

**运行**：

```shell
./hserver 9190
./hclient 127.0.0.1 9190
```

运行的时候，首先再 9190 端口启动服务，然后 heserver 就会一直等待客户端进行响应，当客户端监听位于本地的 IP 为 127.0.0.1 的地址的9190端口时，客户端就会收到服务端的回应，输出`Hello World!`

### 1.2 基于 Linux 的文件操作

讨论套接字的过程中突然谈及文件也许有些奇怪。但是对于 Linux 而言，socket 操作与文件操作没有区别，因而有必要详细了解文件。在 Linux 世界里，socket 也被认为是文件的一种，因此在网络数据传输过程中自然可以使用 I/O 的相关函数。Windows 与 Linux 不同，是要区分 socket 和文件的。因此在 Windows 中需要调用特殊的数据传输相关函数。

#### 1.2.1 底层访问和文件描述符

分配给标准输入输出及标准错误的文件描述符。

| 文件描述符 |           对象            |
| :--------: | :-----------------------: |
|     0      | 标准输入：Standard Input  |
|     1      | 标准输出：Standard Output |
|     2      | 标准错误：Standard Error  |

文件和套接字一般经过创建过程才会被分配文件描述符。

文件描述符也被称为「文件句柄」，但是「句柄」主要是 Windows 中的术语。因此，在本书中如果设计 Windows 平台将使用「句柄」，如果是 Linux 将使用「描述符」。

#### 1.2.2 打开文件:

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int open(const char *path, int flag);
/*
成功时返回文件描述符，失败时返回-1
path : 文件名的字符串地址
flag : 文件打开模式信息
*/
```

文件打开模式如下表：

| 打开模式 |            含义            |
| :------: | :------------------------: |
| O_CREAT  |       必要时创建文件       |
| O_TRUNC  |      删除全部现有数据      |
| O_APPEND | 维持现有数据，保存到其后面 |
| O_RDONLY |          只读打开          |
| O_WRONLY |          只写打开          |
|  O_RDWR  |          读写打开          |

#### 1.2.3 关闭文件：

```c
#include <unistd.h>
int close(int fd);
/*
成功时返回 0 ，失败时返回 -1
fd : 需要关闭的文件或套接字的文件描述符
*/
```

若调用此函数同时传递文件描述符参数，则关闭（终止）响应文件。另外需要注意的是，此函数不仅可以关闭文件，还可以关闭套接字。再次证明了「Linux 操作系统不区分文件与套接字」的特点。

#### 1.2.4 将数据写入文件：

```c
#include <unistd.h>
ssize_t write(int fd, const void *buf, size_t nbytes);
/*
成功时返回写入的字节数 ，失败时返回 -1
fd : 显示数据传输对象的文件描述符
buf : 保存要传输数据的缓冲值地址
nbytes : 要传输数据的字节数
*/
```

在此函数的定义中，size_t 是通过 typedef 声明的 unsigned int 类型。对 ssize_t 来说，ssize_t 前面多加的 s 代表 signed ，即 ssize_t 是通过 typedef 声明的 signed int 类型。

创建新文件并保存数据：

代码见：[low_open.c](ch01/low_open.c)

编译运行：

```shell
gcc low_open.c -o lopen
./lopen
```

然后会生成一个`data.txt`的文件，里面有`Let's go!`

#### 1.2.5 读取文件中的数据：

与之前的`write()`函数相对应，`read()`用来输入（接收）数据。

```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t nbytes);
/*
成功时返回接收的字节数（但遇到文件结尾则返回 0），失败时返回 -1
fd : 显示数据接收对象的文件描述符
buf : 要保存接收的数据的缓冲地址值。
nbytes : 要接收数据的最大字节数
*/
```

下面示例通过 read() 函数读取 data.txt 中保存的数据。

代码见：[low_read.c](ch01/low_read.c)

编译运行：

```shell
gcc low_read.c -o lread
./lread
```

在上一步的 data.txt 文件与没有删的情况下，会输出：

```
file descriptor: 3
file data: Let's go!
```

关于文件描述符的 I/O 操作到此结束，要明白，这些内容同样适合于套接字。

#### 1.2.6 文件描述符与套接字

下面将同时创建文件和套接字，并用整数型态比较返回的文件描述符的值.

代码见：[fd_seri.c](ch01/fd_seri.c)

**编译运行**：

```shell
gcc fd_seri.c -o fds
./fds
```

**输出结果**:

```
file descriptor 1: 3
file descriptor 2: 15
file descriptor 3: 16
```

### 1.3 基于 Windows 平台的实现

暂略

### 1.4 基于 Windows 的套接字相关函数及示例

暂略

### 1.5 习题

> :heavy_exclamation_mark:以下部分的答案，仅代表我个人观点，可能不是正确答案

1. 套接字在网络编程中的作用是什么？为何称它为套接字？

   > 答：操作系统会提供「套接字」（socket）的部件，套接字是网络数据传输用的软件设备。因此，「网络编程」也叫「套接字编程」。「套接字」就是用来连接网络的工具。

2. 在服务器端创建套接字以后，会依次调用 listen 函数和 accept 函数。请比较二者作用。

   > 答：调用 listen 函数将套接字转换成可受连接状态（监听），调用 accept 函数受理连接请求。如果在没有连接请求的情况下调用该函数，则不会返回，直到有连接请求为止。

3. Linux 中，对套接字数据进行 I/O 时可以直接使用文件 I/O 相关函数；而在 Windows 中则不可以。原因为何？

   > 答：暂略。

4. 创建套接字后一般会给他分配地址，为什么？为了完成地址分配需要调用哪个函数？

   > 答：套接字被创建之后，只有为其分配了IP地址和端口号后，客户端才能够通过IP地址及端口号与服务器端建立连接，需要调用 bind 函数来完成地址分配。

5.  Linux 中的文件描述符与 Windows 的句柄实际上非常类似。请以套接字为对象说明它们的含义。

   > 答：暂略。

6. 底层 I/O 函数与 ANSI 标准定义的文件 I/O 函数有何区别？

   > 答：文件 I/O 又称为低级磁盘 I/O，遵循 POSIX 相关标准。任何兼容 POSIX 标准的操作系统上都支持文件I/O。标准 I/O 被称为高级磁盘 I/O，遵循 ANSI C 相关标准。只要开发环境中有标准 I/O 库，标准 I/O 就可以使用。（Linux 中使用的是 GLIBC，它是标准C库的超集。不仅包含 ANSI C 中定义的函数，还包括 POSIX 标准中定义的函数。因此，Linux 下既可以使用标准 I/O，也可以使用文件 I/O）。

7. 参考本书给出的示例`low_open.c`和`low_read.c`，分别利用底层文件 I/O 和 ANSI 标准 I/O 编写文件复制程序。可任意指定复制程序的使用方法。

   > 答：暂略。

## 第 2 章 套接字类型与协议设置

本章代码，在[TCP-IP-NetworkNote](https://github.com/riba2534/TCP-IP-NetworkNote)中可以找到，直接点连接可能进不去。

本章仅需了解创建套接字时调用的 socket 函数。

### 2.1 套接字协议及数据传输特性

#### 2.1.1 创建套接字

```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
/*
成功时返回文件描述符，失败时返回-1
domain: 套接字中使用的协议族（Protocol Family）
type: 套接字数据传输的类型信息
protocol: 计算机间通信中使用的协议信息
*/
```

#### 2.1.2 协议族（Protocol Family）

通过 socket 函数的第一个参数传递套接字中使用的协议分类信息。此协议分类信息称为协议族，可分成如下几类：

> 头文件 `sys/socket.h` 中声明的协议族
>

| 名称      | 协议族               |
| --------- | -------------------- |
| PF_INET   | IPV4 互联网协议族    |
| PF_INET6  | IPV6 互联网协议族    |
| PF_LOCAL  | 本地通信 Unix 协议族 |
| PF_PACKET | 底层套接字的协议族   |
| PF_IPX    | IPX Novel 协议族     |

本书着重讲 PF_INET 对应的 IPV4 互联网协议族。其他协议并不常用，或并未普及。**另外，套接字中采用的最终的协议信息是通过 socket 函数的第三个参数传递的。在指定的协议族范围内通过第一个参数决定第三个参数。**

#### 2.1.3 套接字类型（Type）

套接字类型指的是套接字的数据传输方式，是通过 socket 函数的第二个参数进行传递，只有这样才能决定创建的套接字的数据传输方式。**已经通过第一个参数传递了协议族信息，为什么还要决定数据传输方式？问题就在于，决定了协议族并不能同时决定数据传输方式。换言之， socket 函数的第一个参数 PF_INET 协议族中也存在多种数据传输方式。**

#### 2.1.4 套接字类型1：面向连接的套接字（SOCK_STREAM）

如果 socket 函数的第二个参数传递`SOCK_STREAM`，将创建面向连接的套接字。

传输方式特征整理如下：

- 传输过程中数据不会消失
- 按序传输数据
- 传输的数据不存在数据边界（Boundary）

这种情形适用于之前说过的 write 和 read 函数

> 传输数据的计算机通过调用3次 write 函数传递了 100 字节的数据，但是接受数据的计算机仅仅通过调用 1 次 read 函数调用就接受了全部 100 个字节。

收发数据的套接字内部有缓冲（buffer），简言之就是字节数组。只要不超过数组容量，那么数据填满缓冲后过 1 次 read 函数的调用就可以读取全部，也有可能调用多次来完成读取。

**套接字缓冲已满是否意味着数据丢失？**

> 答：缓冲并不总是满的。如果读取速度比数据传入过来的速度慢，则缓冲可能被填满，但是这时也不会丢失数据，因为传输套接字此时会停止数据传输，所以面向连接的套接字不会发生数据丢失。

套接字联机必须一一对应。面向连接的套接字可总结为：

**可靠地、按序传递的、基于字节的面向连接的数据传输方式的套接字。**

#### 2.1.5 面向消息的套接字（SOCK_DGRAM）

如果 socket 函数的第二个参数传递`SOCK_DGRAM`，则将创建面向消息的套接字。面向消息的套接字可以比喻成高速移动的摩托车队。特点如下：

- 强调快速传输而非传输有序
- 传输的数据可能丢失也可能损毁
- 传输的数据有边界
- 限制每次传输数据的大小

面向消息的套接字比面向连接的套接字更具哟传输速度，但可能丢失。特点可总结为：

**不可靠的、不按序传递的、以数据的高速传输为目的套接字。**

#### 2.1.6 协议的最终选择

socket 函数的第三个参数决定最终采用的协议。前面已经通过前两个参数传递了协议族信息和套接字数据传输方式，这些信息还不够吗？为什么要传输第三个参数呢？

> 可以应对同一协议族中存在的多个数据传输方式相同的协议，所以数据传输方式相同，但是协议不同，需要用第三个参数指定具体的协议信息。

本书用的是 Ipv4 的协议族，和面向连接的数据传输，满足这两个条件的协议只有 TPPROTO_TCP ，因此可以如下调用 socket 函数创建套接字，这种套接字称为 TCP 套接字。

```c
int tcp_socket = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);
```

SOCK_DGRAM 指的是面向消息的数据传输方式，满足上述条件的协议只有 TPPROTO_UDP 。这种套接字称为 UDP 套接字：

```c
int udp_socket = socket(PF_INET, SOCK_DGRAM, IPPROTO_UDP);
```

#### 2.1.7 面向连接的套接字：TCP 套接字示例

需要对第一章的代码做出修改，修改好的代码如下：

- [tcp_client.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch02/tcp_client.c)
- [tcp_server.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch02/tcp_server.c)

编译：

```shell
gcc tcp_client.c -o hclient
gcc tcp_server.c -o hserver
```

运行：

```shell
./hserver 9190
./hclient 127.0.0.1 9190
```

结果：

```
Message from server : Hello World! 
Function read call count: 13
```

从运行结果可以看出服务端发送了13字节的数据，客户端调用13次 read 函数进行读取。

### 2.2 Windows 平台下的实现及验证

暂略

### 2.3 习题

1. 什么是协议？在收发数据中定义协议有何意义？

   > 答：协议是对话中使用的通信规则，简言之，协议就是为了完成数据交换而定好的约定。在收发数据中定义协议，能够让计算机之间进行正确无误的对话，以此来交换数据。

2. 面向连接的套接字 TCP 套接字传输特性有 3 点，请分别说明。

   > 答：①传输过程中数据不会消失②按序传输数据③传输的数据不存在数据边界（Boundary）

3. 下面那些是面向消息的套接字的特性？

   - **传输数据可能丢失**
   - 没有数据边界（Boundary）
   - **以快速传递为目标**
   - 不限制每次传输数据大小
   - **与面向连接的套接字不同，不存在连接概念**

4. 下列数据适合用哪类套接字进行传输？

   - 演唱会现场直播的多媒体数据（UDP）
   - 某人压缩过的文本文件（TCP）
   - 网上银行用户与银行之间的数据传递（TCP）

5. 何种类型的套接字不存在数据边界？这类套接字接收数据时应该注意什么？

   > 答：TCP 不存在数据边界。在接收数据时，需要保证在接收套接字的缓冲区填充满之时就从buffer里读取数据。也就是，在接收套接字内部，写入buffer的速度要小于读出buffer的速度。

## 第 3 章 地址族与数据序列

本章代码，在[TCP-IP-NetworkNote](https://github.com/riba2534/TCP-IP-NetworkNote)中可以找到。

把套接字比喻成电话，那么目前只安装了电话机，本章讲解给电话机分配号码的方法，即给套接字分配 IP 地址和端口号。

### 3.1 分配给套接字的 IP 地址与端口号

IP 是 Internet Protocol（网络协议）的简写，是为手法网络数据而分配给计算机的值。端口号并非赋予计算机的值，而是为了区分程序中创建的套接字而分配给套接字的端口号。

#### 3.1.1 网络地址（Internet Address）

为使计算机连接到网络并收发数据，必须为其分配 IP 地址。IP 地址分为两类。

- IPV4（Internet Protocol version 4）4 字节地址族
- IPV6（Internet Protocol version 6）6 字节地址族

两者之间的主要差别是 IP 地址所用的字节数，目前通用的是 IPV4 , IPV6 的普及还需要时间。

IPV4 标准的 4 字节 IP 地址分为网络地址和主机（指计算机）地址，且分为 A、B、C、D、E 等类型。

![](https://i.loli.net/2019/01/13/5c3ab0eb17bbe.png)

数据传输过程：

![](https://i.loli.net/2019/01/13/5c3ab19174fa4.png)

某主机向 203.211.172.103 和 203.211.217.202 传递数据，其中 203.211.172 和 203.211.217 为该网络的网络地址，所以「向相应网络传输数据」实际上是向构成网络的路由器或者交换机传输数据，然后又路由器或者交换机根据数据中的主机地址向目标主机传递数据。

#### 3.1.2 网络地址分类与主机地址边界

只需通过IP地址的第一个字节即可判断网络地址占用的总字节数，因为我们根据IP地址的边界区分网络地址，如下所示：

- A 类地址的首字节范围为：0~127
- B 类地址的首字节范围为：128~191
- C 类地址的首字节范围为：192~223

还有如下这种表示方式：

- A 类地址的首位以 0 开始
- B 类地址的前2位以 10 开始
- C 类地址的前3位以 110 开始

因此套接字手法数据时，数据传到网络后即可轻松找到主机。

#### 3.1.3 用于区分套接字的端口号

IP地址用于区分计算机，只要有IP地址就能向目标主机传输数据，但是只有这些还不够，我们需要把信息传输给具体的应用程序。

所以计算机一般有 NIC（网络接口卡）数据传输设备。通过 NIC 接受的数据内有端口号，操作系统参考端口号把信息传给相应的应用程序。

端口号由 16 位构成，可分配的端口号范围是 0~65535 。但是 0~1023 是知名端口，一般分配给特定的应用程序，所以应当分配给此范围之外的值。

虽然端口号不能重复，但是 TCP 套接字和 UDP 套接字不会共用端接口号，所以允许重复。如果某 TCP 套接字使用了 9190 端口号，其他 TCP 套接字就无法使用该端口号，但是 UDP 套接字可以使用。

总之，数据传输目标地址同时包含IP地址和端口号，只有这样，数据才会被传输到最终的目的应用程序。

### 3.2 地址信息的表示

应用程序中使用的IP地址和端口号以结构体的形式给出了定义。本节围绕结构体讨论目标地址的表示方法。

#### 3.2.1 表示 IPV4 地址的结构体

结构体的定义如下

```c
struct sockaddr_in
{
    sa_family_t sin_family;  //地址族（Address Family）
    uint16_t sin_port;       //16 位 TCP/UDP 端口号
    struct in_addr sin_addr; //32位 IP 地址
    char sin_zero[8];        //不使用
};
```

该结构体中提到的另一个结构体 in_addr 定义如下，它用来存放 32 位IP地址

```c
struct in_addr
{
    in_addr_t s_addr; //32位IPV4地址
}
```

关于以上两个结构体的一些数据类型。

| 数据类型名称 |             数据类型说明             | 声明的头文件 |
| :----------: | :----------------------------------: | :----------: |
|   int 8_t    |           signed 8-bit int           | sys/types.h  |
|   uint8_t    |  unsigned 8-bit int (unsigned char)  | sys/types.h  |
|   int16_t    |          signed 16-bit int           | sys/types.h  |
|   uint16_t   | unsigned 16-bit int (unsigned short) | sys/types.h  |
|   int32_t    |          signed 32-bit int           | sys/types.h  |
|   uint32_t   | unsigned 32-bit int (unsigned long)  | sys/types.h  |
| sa_family_t  |       地址族（address family）       | sys/socket.h |
|  socklen_t   |       长度（length of struct）       | sys/socket.h |
|  in_addr_t   |       IP地址，声明为 uint_32_t       | netinet/in.h |
|  in_port_t   |       端口号，声明为 uint_16_t       | netinet/in.h |

为什么要额外定义这些数据类型呢？这是考虑扩展性的结果

#### 3.2.2 结构体 sockaddr_in 的成员分析

- 成员 sin_family

每种协议适用的地址族不同，比如，IPV4 使用 4 字节的地址族，IPV6 使用 16 字节的地址族。

> 地址族

| 地址族（Address Family） | 含义                               |
| ------------------------ | ---------------------------------- |
| AF_INET                  | IPV4用的地址族                     |
| AF_INET6                 | IPV6用的地址族                     |
| AF_LOCAL                 | 本地通信中采用的 Unix 协议的地址族 |

AF_LOACL 只是为了说明具有多种地址族而添加的。

- 成员 sin_port

  该成员保存 16 位端口号，重点在于，它以网络字节序保存。

- 成员 sin_addr

  该成员保存 32 为IP地址信息，且也以网络字节序保存

- 成员 sin_zero

  无特殊含义。只是为结构体 sockaddr_in 结构体变量地址值将以如下方式传递给 bind 函数。

  在之前的代码中

  ```c
  if (bind(serv_sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) == -1)
      error_handling("bind() error");
  ```

  此处 bind 第二个参数期望得到的是 sockaddr 结构体变量的地址值，包括地址族、端口号、IP地址等。

  ```c
  struct sockaddr
  {
      sa_family_t sin_family; //地址族
      char sa_data[14];       //地址信息
  }
  ```

  此结构体 sa_data 保存的地址信息中需要包含IP地址和端口号，剩余部分应该填充 0 ，但是这样对于包含地址的信息非常麻烦，所以出现了 sockaddr_in 结构体，然后强制转换成 sockaddr 类型，则生成符合 bind 条件的参数。

### 3.3 网络字节序与地址变换

不同的 CPU 中，4 字节整数值1在内存空间保存方式是不同的。

有些 CPU 这样保存：

```
00000000 00000000 00000000 00000001
```

有些 CPU 这样保存：

```
00000001 00000000 00000000 00000000
```

两种一种是顺序保存，一种是倒序保存 。

#### 3.3.1 字节序（Order）与网络字节序

CPU 保存数据的方式有两种，这意味着 CPU 解析数据的方式也有 2 种：

- 大端序（Big Endian）：高位字节存放到低位地址
- 小端序（Little Endian）：高位字节存放到高位地址

![big.png](https://i.loli.net/2019/01/13/5c3ac9c1b2550.png)
![small.png](https://i.loli.net/2019/01/13/5c3ac9c1c3348.png)

两台字节序不同的计算机在数据传递的过程中可能出现的问题：

![zijiexu.png](https://i.loli.net/2019/01/13/5c3aca956c8e9.png)

因为这种原因，所以在通过网络传输数据时必须约定统一的方式，这种约定被称为网络字节序，非常简单，统一为大端序。即，先把数据数组转化成大端序格式再进行网络传输。

#### 3.3.2 字节序转换

帮助转换字节序的函数：

```c
unsigned short htons(unsigned short);
unsigned short ntohs(unsigned short);
unsigned long htonl(unsigned long);
unsigned long ntohl(unsigned long);
```

通过函数名称掌握其功能，只需要了解：

- htons 的 h 代表主机（host）字节序。
- htons 的 n 代表网络（network）字节序。
- s 代表 short
- l 代表 long

下面的代码是示例，说明以上函数调用过程：

[endian_conv.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch03/endian_conv.c)

```cpp
#include <stdio.h>
#include <arpa/inet.h>
int main(int argc, char *argv[])
{
    unsigned short host_port = 0x1234;
    unsigned short net_port;
    unsigned long host_addr = 0x12345678;
    unsigned long net_addr;

    net_port = htons(host_port); //转换为网络字节序
    net_addr = htonl(host_addr);

    printf("Host ordered port: %#x \n", host_port);
    printf("Network ordered port: %#x \n", net_port);
    printf("Host ordered address: %#lx \n", host_addr);
    printf("Network ordered address: %#lx \n", net_addr);

    return 0;
}
```

编译运行：

```shell
gcc endian_conv.c -o conv
./conv
```

结果：

```
Host ordered port: 0x1234
Network ordered port: 0x3412
Host ordered address: 0x12345678
Network ordered address: 0x78563412
```

这是在小端 CPU 的运行结果。大部分人会得到相同的结果，因为 Intel 和 AMD 的 CPU 都是小端序为标准。

### 3.4 网络地址的初始化与分配

#### 3.4.1 将字符串信息转换为网络字节序的整数型

sockaddr_in 中需要的是 32 位整数型，但是我们只熟悉点分十进制表示法，那么改如何把类似于 201.211.214.36 转换为 4 字节的整数类型数据呢 ?幸运的是，有一个函数可以帮助我们完成它。

```C
#include <arpa/inet.h>
in_addr_t inet_addr(const char *string);
```

具体示例：

[inet_addr.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch03/inet_addr.c)

```c
#include <stdio.h>
#include <arpa/inet.h>
int main(int argc, char *argv[])
{
    char *addr1 = "1.2.3.4";
    char *addr2 = "1.2.3.256";

    unsigned long conv_addr = inet_addr(addr1);
    if (conv_addr == INADDR_NONE)
        printf("Error occured! \n");
    else
        printf("Network ordered integer addr: %#lx \n", conv_addr);

    conv_addr = inet_addr(addr2);
    if (conv_addr == INADDR_NONE)
        printf("Error occured! \n");
    else
        printf("Network ordered integer addr: %#lx \n", conv_addr);
    return 0;
}
```

编译运行：

```shell
gcc inet_addr.c -o addr
./addr
```

输出：

```
Network ordered integer addr: 0x4030201
Error occured!
```

1个字节能表示的最大整数是255，所以代码中 addr2 是错误的IP地址。从运行结果看，inet_addr 不仅可以转换地址，还可以检测有效性。

inet_aton 函数与 inet_addr 函数在功能上完全相同，也是将字符串形式的IP地址转换成整数型的IP地址。只不过该函数用了 in_addr 结构体，且使用频率更高。

```c
#include <arpa/inet.h>
int inet_aton(const char *string, struct in_addr *addr);
/*
成功时返回 1 ，失败时返回 0
string: 含有需要转换的IP地址信息的字符串地址值
addr: 将保存转换结果的 in_addr 结构体变量的地址值
*/
```

函数调用示例：

[inet_aton.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch03/inet_aton.c)

```c
#include <stdio.h>
#include <stdlib.h>
#include <arpa/inet.h>
void error_handling(char *message);

int main(int argc, char *argv[])
{
    char *addr = "127.232.124.79";
    struct sockaddr_in addr_inet;

    if (!inet_aton(addr, &addr_inet.sin_addr))
        error_handling("Conversion error");
    else
        printf("Network ordered integer addr: %#x \n", addr_inet.sin_addr.s_addr);
    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```

编译运行：

```c
gcc inet_aton.c -o aton
./aton
```

运行结果：

```
Network ordered integer addr: 0x4f7ce87f
```

可以看出，已经成功的把转换后的地址放进了 addr_inet.sin_addr.s_addr 中。

还有一个函数，与 inet_aton() 正好相反，它可以把网络字节序整数型IP地址转换成我们熟悉的字符串形式，函数原型如下：

```c
#include <arpa/inet.h>
char *inet_ntoa(struct in_addr adr);
```

该函数将通过参数传入的整数型IP地址转换为字符串格式并返回。但要小心，返回值为 char 指针，返回字符串地址意味着字符串已经保存在内存空间，但是该函数未向程序员要求分配内存，而是再内部申请了内存保存了字符串。也就是说调用了该函数候要立即把信息复制到其他内存空间。因此，若再次调用 inet_ntoa 函数，则有可能覆盖之前保存的字符串信息。总之，再次调用 inet_ntoa 函数前返回的字符串地址是有效的。若需要长期保存，则应该将字符串复制到其他内存空间。

示例：

[inet_ntoa.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch03/inet_ntoa.c)

```c
#include <stdio.h>
#include <string.h>
#include <arpa/inet.h>

int main(int argc, char *argv[])
{
    struct sockaddr_in addr1, addr2;
    char *str_ptr;
    char str_arr[20];

    addr1.sin_addr.s_addr = htonl(0x1020304);
    addr2.sin_addr.s_addr = htonl(0x1010101);
    //把addr1中的结构体信息转换为字符串的IP地址形式
    str_ptr = inet_ntoa(addr1.sin_addr);
    strcpy(str_arr, str_ptr);
    printf("Dotted-Decimal notation1: %s \n", str_ptr);

    inet_ntoa(addr2.sin_addr);
    printf("Dotted-Decimal notation2: %s \n", str_ptr);
    printf("Dotted-Decimal notation3: %s \n", str_arr);
    return 0;
}
```

编译运行：

```shell
gcc inet_ntoa.c -o ntoa
./ntoa
```

输出:

```c
Dotted-Decimal notation1: 1.2.3.4
Dotted-Decimal notation2: 1.1.1.1
Dotted-Decimal notation3: 1.2.3.4
```

#### 3.4.2 网络地址初始化

结合前面的内容，介绍套接字创建过程中，常见的网络信息初始化方法：

```c
struct sockaddr_in addr;
char *serv_ip = "211.217,168.13";          //声明IP地址族
char *serv_port = "9190";                  //声明端口号字符串
memset(&addr, 0, sizeof(addr));            //结构体变量 addr 的所有成员初始化为0
addr.sin_family = AF_INET;                 //制定地址族
addr.sin_addr.s_addr = inet_addr(serv_ip); //基于字符串的IP地址初始化
addr.sin_port = htons(atoi(serv_port));    //基于字符串的IP地址端口号初始化
```

### 3.5 基于 Windows 的实现

略

### 3.6 习题

> 答案仅代表本人个人观点，不一定正确

1. **IP地址族 IPV4 与 IPV6 有什么区别？在何种背景下诞生了 IPV6?**

   答：主要差别是IP地址所用的字节数，目前通用的是IPV4，目前IPV4的资源已耗尽，所以诞生了IPV6，它具有更大的地址空间。

2. **通过 IPV4 网络 ID 、主机 ID 及路由器的关系说明公司局域网的计算机传输数据的过程**

   答：网络ID是为了区分网络而设置的一部分IP地址，假设向`www.baidu.com`公司传输数据，该公司内部构建了局域网。因为首先要向`baidu.com`传输数据，也就是说并非一开始就浏览所有四字节IP地址，首先找到网络地址，进而由`baidu.com`（构成网络的路由器）接收到数据后，传输到主机地址。比如向 203.211.712.103 传输数据，那就先找到 203.211.172 然后由这个网络的网关找主机号为 172 的机器传输数据。

3. **套接字地址分为IP地址和端口号，为什么需要IP地址和端口号？或者说，通过IP地址可以区分哪些对象？通过端口号可以区分哪些对象？**

   答：有了IP地址和端口号，才能把数据准确的传送到某个应用程序中。通过IP地址可以区分具体的主机，通过端口号可以区分主机上的应用程序。

4. **请说明IP地址的分类方法，并据此说出下面这些IP的分类。**

   - 214.121.212.102（C类）
   - 120.101.122.89（A类）
   - 129.78.102.211（B类）

   分类方法：A 类地址的首字节范围为：0~127、B 类地址的首字节范围为：128~191、C 类地址的首字节范围为：192~223

5. **计算机通过路由器和交换机连接到互联网，请说出路由器和交换机的作用。**

   答：路由器和交换机完成外网和本网主机之间的数据交换。

6. **什么是知名端口？其范围是多少？知名端口中具有代表性的 HTTP 和 FTP 的端口号各是多少？**

   答：知名端口是要把该端口分配给特定的应用程序，范围是 0~1023 ，HTTP 的端口号是 80 ，FTP 的端口号是20和21

7. **向套接字分配地址的 bind 函数原型如下：**

   ```c
   int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);
   ```

   **而调用时则用：**

   ```c
   bind(serv_sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)
   ```

   **此处 serv_addr 为 sockaddr_in 结构体变量。与函数原型不同，传入的是 sockaddr_in 结构体变量，请说明原因。**

   答：因为对于详细的地址信息使用 sockaddr 类型传递特别麻烦，进而有了 sockaddr_in 类型，其中基本与前面的类型保持一致，还有 sa_sata[4] 来保存地址信息，剩余全部填 0，所以强制转换后，不影响程序运行。

8. **请解释大端序，小端序、网络字节序，并说明为何需要网络字节序。**

   答：CPU 向内存保存数据有两种方式，大端序是高位字节存放低位地址，小端序是高位字节存放高位地址，网络字节序是为了方便传输的信息统一性，统一成了大端序。

9. **大端序计算机希望把 4 字节整数型 12 传递到小端序计算机。请说出数据传输过程中发生的字节序变换过程。**

   答：0x12->0x21

10. **怎样表示回送地址？其含义是什么？如果向会送地址处传输数据将会发生什么情况？**

    答：127.0.0.1 表示回送地址，指的是计算机自身的IP地址，无论什么程序，一旦使用回送地址发送数据，协议软件立即返回，不进行任何网络传输。

## 第 4 章 基于 TCP 的服务端/客户端（1）

本章代码，在[TCP-IP-NetworkNote](https://github.com/riba2534/TCP-IP-NetworkNote)中可以找到。

### 4.1 理解 TCP 和 UDP

根据数据传输方式的不同，基于网络协议的套接字一般分为 TCP 套接字和 UDP 套接字。因为 TCP 套接字是面向连接的，因此又被称为基于流（stream）的套接字。

TCP 是 Transmission Control Protocol （传输控制协议）的简写，意为「对数据传输过程的控制」。因此，学习控制方法及范围有助于正确理解 TCP 套接字。

#### 4.1.1 TCP/IP 协议栈

![](https://i.loli.net/2019/01/14/5c3c21889db06.png)

TCP/IP 协议栈共分为 4 层，可以理解为数据收发分成了 4 个层次化过程，通过层次化的方式来解决问题

#### 4.1.2 链路层

链路层是物理链接领域标准化的结果，也是最基本的领域，专门定义LAN、WAN、MAN等网络标准。若两台主机通过网络进行数据交换，则需要物理连接，链路层就负责这些标准。

#### 4.1.3 IP 层

转备好物理连接候就要传输数据。为了再复杂网络中传输数据，首先要考虑路径的选择。向目标传输数据需要经过哪条路径？解决此问题的就是IP层，该层使用的协议就是IP。

IP 是面向消息的、不可靠的协议。每次传输数据时会帮我们选择路径，但并不一致。如果传输过程中发生错误，则选择其他路径，但是如果发生数据丢失或错误，则无法解决。换言之，IP协议无法应对数据错误。

#### 4.1.4 TCP/UDP 层

IP 层解决数据传输中的路径选择问题，秩序照此路径传输数据即可。TCP 和 UDP 层以 IP 层提供的路径信息为基础完成实际的数据传输，故该层又称为传输层。UDP 比 TCP 简单，现在我们只解释 TCP 。 TCP 可以保证数据的可靠传输，但是它发送数据时以 IP 层为基础（这也是协议栈层次化的原因）

IP 层只关注一个数据包（数据传输基本单位）的传输过程。因此，即使传输多个数据包，每个数据包也是由 IP 层实际传输的，也就是说传输顺序及传输本身是不可靠的。若只利用IP层传输数据，则可能导致后传输的数据包B比先传输的数据包A提早到达。另外，传输的数据包A、B、C中可能只收到A和C，甚至收到的C可能已经损毁 。反之，若添加 TCP 协议则按照如下对话方式进行数据交换。

> 主机A：正确接受第二个数据包
>
> 主机B：恩，知道了
>
> 主机A：正确收到第三个数据包
>
> 主机B：可我已经发送第四个数据包了啊！哦，您没收到吧，我给你重新发。

这就是 TCP 的作用。如果交换数据的过程中可以确认对方已经收到数据，并重传丢失的数据，那么即便IP层不保证数据传输，这类通信也是可靠的。

![](https://i.loli.net/2019/01/14/5c3c268b40be6.png)

#### 4.1.5 应用层

上述内容是套接字通信过程中自动处理的。选择数据传输路径、数据确认过程都被隐藏到套接字内部。向程序员提供的工具就是套接字，只需要利用套接字编出程序即可。编写软件的过程中，需要根据程序的特点来决定服务器和客户端之间的数据传输规则，这便是应用层协议。

### 4.2 实现基于 TCP 的服务器/客户端

#### 4.2.1 TCP 服务端的默认函数的调用程序

![](https://i.loli.net/2019/01/14/5c3c2782a7810.png)

调用 socket 函数创建套接字，声明并初始化地址信息的结构体变量，调用 bind 函数向套接字分配地址。

#### 4.2.2 进入等待连接请求状态

已经调用了 bind 函数给他要借资分配地址，接下来就是要通过调用 listen 函数进入等待链接请求状态。只有调用了 listen 函数，客户端才能进入可发出连接请求的状态。换言之，这时客户端才能调用 connect 函数

```c
#include <sys/socket.h>
int listen(int sockfd, int backlog);
//成功时返回0，失败时返回-1
//sock: 希望进入等待连接请求状态的套接字文件描述符，传递的描述符套接字参数称为服务端套接字
//backlog: 连接请求等待队列的长度，若为5，则队列长度为5，表示最多使5个连接请求进入队列            
```
#### 4.2.3 受理客户端连接请求

调用 listen 函数后，则应该按序受理。受理请求意味着可接受数据的状态。进入这种状态所需的部件是**套接字**，但是此时使用的不是服务端套接字，此时需要另一个套接字，但是没必要亲自创建，下面的函数将自动创建套接字。

```c
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
/*
成功时返回文件描述符，失败时返回-1
sock: 服务端套接字的文件描述符
addr: 保存发起连接请求的客户端地址信息的变量地址值
addrlen: 的第二个参数addr结构体的长度，但是存放有长度的变量地址。
*/
```

sccept 函数受理连接请求队列中待处理的客户端连接请求。函数调用成功后，accept 内部将产生用于数据 I/O 的套接字，并返回其文件描述符。需要强调的是套接字是自动创建的，并自动与发起连接请求的客户端建立连接。

#### 4.2.4 回顾 Hello World 服务端

- 代码：[hello_server.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch04/hello_server.c)

重新整理一下代码的思路

1. 服务端实现过程中首先要创建套接字，此时的套接字并非是真正的服务端套接字
2. 为了完成套接字地址的分配，初始化结构体变量并调用 bind 函数。
3. 调用 listen 函数进入等待连接请求状态。连接请求状态队列的长度设置为5.此时的套接字才是服务端套接字。
4. 调用 accept 函数从队头取 1 个连接请求与客户端建立连接，并返回创建的套接字文件描述符。另外，调用 accept 函数时若等待队列为空，则 accept 函数不会返回，直到队列中出现新的客户端连接。
5. 调用 write 函数向客户端传送数据，调用 close 关闭连接

#### 4.2.5 TCP 客户端的默认函数调用顺序

![](https://i.loli.net/2019/01/14/5c3c31d77e86c.png)

与服务端相比，区别就在于「请求连接」，他是创建客户端套接字后向服务端发起的连接请求。服务端调用 listen 函数后创建连接请求等待队列，之后客户端即可请求连接。

```c
#include <sys/socket.h>
int connect(int sock, struct sockaddr *servaddr, socklen_t addrlen);
/*
成功时返回0，失败返回-1
sock:客户端套接字文件描述符
servaddr: 保存目标服务器端地址信息的变量地址值
addrlen: 以字节为单位传递给第二个结构体参数 servaddr 的变量地址长度
*/
```

客户端调用 connect 函数候，发生以下函数之一才会返回（完成函数调用）:

- 服务端接受连接请求
- 发生断网等一场状况而中断连接请求

注意：**接受连接**不代表服务端调用 accept 函数，其实只是服务器端把连接请求信息记录到等待队列。因此 connect 函数返回后并不应该立即进行数据交换。

#### 4.2.6 回顾 Hello World 客户端

- 代码：[hello_client.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch04/hello_client.c)

重新理解这个程序：

1. 创建准备连接服务器的套接字，此时创建的是 TCP 套接字
2. 结构体变量 serv_addr 中初始化IP和端口信息。初始化值为目标服务器端套接字的IP和端口信息。
3. 调用 connect 函数向服务端发起连接请求
4. 完成连接后，接收服务端传输的数据
5. 接收数据后调用 close 函数关闭套接字，结束与服务器端的连接。

#### 4.2.7 基于 TCP 的服务端/客户端函数调用关系

![](https://i.loli.net/2019/01/14/5c3c35a773b8c.png)

关系如上图所示。

### 4.3 实现迭代服务端/客户端

编写一个回声（echo）服务器/客户端。顾名思义，服务端将客户端传输的字符串数据原封不动的传回客户端，就像回声一样。在此之前，需要解释一下迭代服务器端。

#### 4.3.1 实现迭代服务器端

在 Hello World 的例子中，等待队列的作用没有太大意义。如果想继续处理好后面的客户端请求应该怎样扩展代码？最简单的方式就是插入循环反复调用 accept 函数，如图:

![](https://i.loli.net/2019/01/15/5c3d3c8a283ad.png)

可以看出，调用 accept 函数后，紧接着调用 I/O 相关的 read write 函数，然后调用 close 函数。这并非针对服务器套接字，而是针对 accept 函数调用时创建的套接字。

#### 4.3.2 迭代回声服务器端/客户端

程序运行的基本方式：

- 服务器端在同一时刻只与一个客户端相连，并提供回声服务。
- 服务器端依次向 5 个客户端提供服务并退出。
- 客户端接受用户输入的字符串并发送到服务器端。
- 服务器端将接受的字符串数据传回客户端，即「回声」
- 服务器端与客户端之间的字符串回声一直执行到客户端输入 Q 为止。

以下是服务端与客户端的代码：

- [echo_server.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch04/echo_server.c)
- [echo_client.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch04/echo_client.c)

编译:

```shell
gcc echo_client.c -o eclient
gcc echo_server.c -o eserver
```

分别运行:

```shell
./eserver 9190
./eclient 127.0.0.1 9190
```

过程和结果：

在一个服务端开启后，用另一个终端窗口开启客户端，然后程序会让你输入字符串，然后客户端输入什么字符串，客户端就会返回什么字符串，按 q 退出。这时服务端的运行并没有结束，服务端一共要处理 5 个客户端的连接，所以另外开多个终端窗口同时开启客户端，服务器按照顺序进行处理。

server:
![server.png](https://i.loli.net/2019/01/15/5c3d523d0a675.png)

client:
![client.png](https://i.loli.net/2019/01/15/5c3d523d336e7.png)

#### 4.3.3 回声客户端存在的问题

以上代码有一个假设「每次调用 read、write函数时都会以字符串为单位执行实际 I/O 操作」

但是「第二章」中说过「TCP 不存在数据边界」，上述客户端是基于 TCP 的，因此多次调用 write 函数传递的字符串有可能一次性传递到服务端。此时客户端有可能从服务端收到多个字符串，这不是我们想要的结果。还需要考虑服务器的如下情况：

「字符串太长，需要分 2 个包发送！」

服务端希望通过调用 1 次 write 函数传输数据，但是如果数据太大，操作系统就有可能把数据分成多个数据包发送到客户端。另外，在此过程中，客户端可能在尚未收到全部数据包时就调用 read 函数。

以上的问题都是源自 TCP 的传输特性，解决方法在第 5 章。

### 4.4 基于 Windows 的实现

暂略

### 4.5 习题

> 答案仅代表本人个人观点，不一定是正确答案。

1. **请你说明 TCP/IP 的 4 层协议栈，并说明 TCP 和 UDP 套接字经过的层级结构差异。**

   答：TCP/IP 的四层协议分为：应用层、TCP/UDP 层、IP层、链路层。差异是一个经过 TCP 层，一个经过 UDP 层。

2. **请说出 TCP/IP 协议栈中链路层和IP层的作用，并给出二者关系**

   答：链路层是物理链接领域标准化的结果，专门定义网络标准。若两台主机通过网络进行数据交换，则首先要做到的就是进行物理链接。IP层：为了在复杂的网络中传输数据，首先需要考虑路径的选择。关系：链路层负责进行一系列物理连接，而IP层负责选择正确可行的物理路径。

3. **为何需要把 TCP/IP 协议栈分成 4 层（或7层）？开放式回答。**

   答：ARPANET 的研制经验表明，对于复杂的计算机网络协议，其结构应该是层次式的。分册的好处：①隔层之间是独立的②灵活性好③结构上可以分隔开④易于实现和维护⑤能促进标准化工作。

4. **客户端调用 connect 函数向服务器端发送请求。服务器端调用哪个函数后，客户端可以调用 connect 函数？**

   答：服务端调用 listen 函数后，客户端可以调用 connect 函数。因为，服务端调用 listen 函数后，服务端套接字才有能力接受请求连接的信号。

5. **什么时候创建连接请求等待队列？它有何种作用？与 accept 有什么关系？**

   答：服务端调用 listen 函数后，accept函数正在处理客户端请求时， 更多的客户端发来了请求连接的数据，此时，就需要创建连接请求等待队列。以便于在accept函数处理完手头的请求之后，按照正确的顺序处理后面正在排队的其他请求。与accept函数的关系：accept函数受理连接请求等待队列中待处理的客户端连接请求。

6. **客户端中为何不需要调用 bind 函数分配地址？如果不调用 bind 函数，那何时、如何向套接字分配IP地址和端口号？**

   答：在调用 connect 函数时分配了地址，客户端IP地址和端口在调用 connect 函数时自动分配，无需调用标记的 bind 函数进行分配。

## 第 5 章 基于 TCP 的服务端/客户端（2）

本章代码，在[TCP-IP-NetworkNote](https://github.com/riba2534/TCP-IP-NetworkNote)中可以找到。

上一章仅仅是从编程角度学习实现方法，并未详细讨论 TCP 的工作原理。因此，本章将想次讲解 TCP 中必要的理论知识，还将给出第 4 章客户端问题的解决方案。

### 5.1 回声客户端的完美实现

#### 5.1.1 回声服务器没有问题，只有回声客户端有问题？

问题不在服务器端，而在客户端，只看代码可能不好理解，因为 I/O 中使用了相同的函数。先回顾一下服务器端的 I/O 相关代码：

```c
while ((str_len = read(clnt_sock, message, BUF_SIZE)) != 0)
    write(clnt_sock, message, str_len);
```

接着是客户端代码:

```c
write(sock, message, strlen(message));
str_len = read(sock, message, BUF_SIZE - 1);
```

二者都在村换调用 read 和 write 函数。实际上之前的回声客户端将 100% 接受字节传输的数据，只不过接受数据时的单位有些问题。扩展客户端代码回顾范围，下面是，客户端的代码:

```c
while (1)
{
    fputs("Input message(Q to quit): ", stdout);
    fgets(message, BUF_SIZE, stdin);

    if (!strcmp(message, "q\n") || !strcmp(message, "Q\n"))
        break;

    write(sock, message, strlen(message));
    str_len = read(sock, message, BUF_SIZE - 1);
    message[str_len] = 0;
    printf("Message from server: %s", message);
}
```

现在应该理解了问题，回声客户端传输的是字符串，而且是通过调用 write 函数一次性发送的。之后还调用一次 read 函数，期待着接受自己传输的字符串，这就是问题所在。

#### 5.1.2 回声客户端问题的解决办法

这个问题其实很容易解决，因为可以提前接受数据的大小。若之前传输了20字节长的字符串，则再接收时循环调用 read 函数读取 20 个字节即可。既然有了解决办法，那么代码如下：

- [echo_client2.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch05/echo_client2.c)

这样修改为了接收所有传输数据而循环调用 read 函数。测试及运行结果可参考第四章。

#### 5.1.3 如果问题不在于回声客户端：定义应用层协议

回声客户端可以提前知道接收数据的长度，这在大多数情况下是不可能的。那么此时无法预知接收数据长度时应该如何手法数据？这是需要的是**应用层协议**的定义。在收发过程中定好规则（协议）以表示数据边界，或者提前告知需要发送的数据的大小。服务端/客户端实现过程中逐步定义的规则集合就是应用层协议。

现在写一个小程序来体验应用层协议的定义过程。要求：

1. 服务器从客户端获得多个数组和运算符信息。
2. 服务器接收到数字候对齐进行加减乘运算，然后把结果传回客户端。

例：

1. 向服务器传递3,5,9的同事请求加法运算，服务器返回3+5+9的结果
2. 请求做乘法运算，客户端会收到`3*5*9`的结果
3. 如果向服务器传递4,3,2的同时要求做减法，则返回4-3-2的运算结果。

请自己实现一个程序来实现功能。

我自己的实现：

- [My_op_server.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch05/My_op_server.c)
- [My_op_client.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch05/My_op_client.c)

编译：

```shell
gcc My_op_client.c -o myclient
gcc My_op_server.c -o myserver
```

结果：

![](https://i.loli.net/2019/01/15/5c3d966b81c03.png)

其实主要是对程序的一点点小改动，只需要再客户端固定好发送的格式，服务端按照固定格式解析，然后返回结果即可。

书上的实现：

- [op_client.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch05/op_client.c)
- [op_server.c](https://github.com/riba2534/TCP-IP-NetworkNote/blob/master/ch05/op_server.c)

阅读代码要注意一下，`int*`与`char`之间的转换。TCP 中不存在数据边界。

编译：

```shell
gcc op_client.c -o opclient
gcc op_server.c -o opserver
```

运行:

```shell
./opserver 9190
./opclient 127.0.0.1 9190
```

结果：

![](https://i.loli.net/2019/01/16/5c3ea297c7649.png)

### 5.2 TCP 原理

#### 5.2.1 TCP 套接字中的 I/O 缓冲

TCP 套接字的数据收发无边界。服务器即使调用 1 次 write 函数传输 40 字节的数据，客户端也有可能通过 4 次 read 函数调用每次读取 10 字节。但此处也有一些一问，服务器一次性传输了 40 字节，而客户端竟然可以缓慢的分批接受。客户端接受 10 字节后，剩下的 30 字节在何处等候呢？

实际上，write 函数调用后并非立即传输数据， read 函数调用后也并非马上接收数据。如图所示，write 函数滴啊用瞬间，数据将移至输出缓冲；read 函数调用瞬间，从输入缓冲读取数据。

![](https://i.loli.net/2019/01/16/5c3ea41cd93c6.png)

I/O 缓冲特性可以整理如下：

- I/O 缓冲在每个 TCP 套接字中单独存在
- I/O 缓冲在创建套接字时自动生成
- 即使关闭套接字也会继续传递输出缓冲中遗留的数据
- 关闭套接字将丢失输入缓冲中的数据

假设发生以下情况，会发生什么事呢？

> 客户端输入缓冲为 50 字节，而服务器端传输了 100 字节。

因为 TCP 不会发生超过输入缓冲大小的数据传输。也就是说，根本不会发生这类问题，因为 TCP 会控制数据流。TCP 中有滑动窗口（Sliding Window）协议，用对话方式如下：

> - A：你好，最多可以向我传递 50 字节
> - B：好的
> - A：我腾出了 20 字节的空间，最多可以接受 70 字节
> - B：好的

数据收发也是如此，因此 TCP 中不会因为缓冲溢出而丢失数据。

write 函数在数据传输完成时返回。



## License

本仓库遵循 CC BY-NC-SA 4.0（署名 - 非商业性使用） 协议，转载请注明出处。

[![CC BY-NC-SA 4.0](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-nc-sa/4.0/)

