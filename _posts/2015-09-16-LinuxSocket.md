---
layout:     post
title:      "Linux Socket编程基础知识"
date:       2015-09-16 12:00:00
author:     "邵正"
header-img: "img/post-bg-2015.jpg"
tags:
    - Linux
    - Socket  
---

| 主题       | 概要                  |
| ---------- | --------------------- |
| socket编程 | Linux Socket编程基础  |
| --------   | ---                   |
| **编辑**   | **时间**              |
| 新建       | 20150916              |
| --------   | ---                   |
| **序号**   | **参考资料**          |
| 1          | Linux程序设计(第四版) |


1.什么是套接字？
套接字(socket)是一种通信机制，凭借这种机制，客户/服务器系统的开发工作既可在本地单机上进行，也可以跨网络进行，linux所提供的功能（如打印服务，连接数据库和提供Web页面）和网络工具（ftp）通常都是通过socket来进行通信的。
2.套接字的连接
 可以把套接字想像为电话转接系统，一个电话打到公司，接线员接听电话并把它转到正确的部门（服务器进程），然后再从部门找到电话要找的人（服务器套接字），每个打入的电话（客户）都被转接到正确的人，而中间介入的接线员则可以空出来处理后续的电话。那么套接字应用程序是如何来建立维持一个连接的？
服务器端：
首先，服务器应用程序用系统调用socket来创建一个套接字，它是系统分配给该服务器进程的类似文件描述符的资源，它不能与其它进程共享。
接着，服务器进程会调用系统调用bind来给套接字命名，本地套接字是Linux文件系统中的文件名，对于网络套接字，它的名字是与客户连接的特定网络有关的服务标识符（地址和端口号）。
其次，服务器进程调用系统调用listen，创建一个队列用于存放来自客户的进入连接。
最后，服务器通过系统调用accept来接受客户的连接，它会创建一个与原有的命名套接字不同的新套接字。这个新的套接字只用与这个特定的客户进行通信，而命令套接字则被保留下来继续处理来自其他客户的连接。
客户端：
客户端的步骤更加简单，客户首先调用socket创建一个未命名套接字，然后将服务器的命名套接字作为一个地址来调用connect与服务器建立连接。

example1:

一个简单的客户端程序：client1.c

```c++
#include<sys/types.h>
#include<sys/socket.h>
#include<stdio.h>
#include<sys/un.h>
#include<unistd.h>
#include<stdlib.h>
int main()
{
	int sockfd,len;
	char ch='A';
	struct sockaddr_un address;
	int result;

	//--为客户创建套接字
	sockfd=socket(AF_UNIX,SOCK_STREAM,0);

	//--根据服务器情况给套接字命名
	address.sun_family=AF_UNIX;
	strcpy(address.sun_path,"server_socket");
	len=sizeof(address);

	//--连接到服务器套接字
	result=connect(sockfd,(struct sockaddr *)&address,len);

	if (-1==result)
	{
	   perror("oops:client1");
	   exit(0);
     }

	//--通过套接字进行读写
	printf("client:write char [ %c] to server \n",ch);
	write(sockfd,&ch,1);
	read(sockfd,&ch,1);
	printf("client:read char from server=%c \n",ch);

	//--关闭退出
	close(sockfd);
	exit(0);

}

```
运行 ./client1时，它会失败，因为还没有创建服务端的套接字。

一个简单的本地服务器程序：server 1.c

```c++
#include<sys/types.h>
#include<sys/socket.h>
#include<stdio.h>
#include<sys/un.h>
#include<unistd.h>
#include<stdlib.h>
int main()
{
	int server_sockfd,client_sockfd;	
	struct sockaddr_un server_addr;
	struct sockaddr_un client_addr;
	int server_len;
	int client_len;

	//--删除以前的套接字，为服务器创建一个未命名套接字
	unlink("server_socket");
	server_sockfd=socket(AF_UNIX,SOCK_STREAM,0);

	//--命名套接字
	server_addr.sun_family=AF_UNIX;
	strcpy(server_addr.sun_path,"server_socket");
	server_len=sizeof(server_addr);
	bind(server_sockfd,(struct sockaddr *) &server_addr,server_len);
	
	//--创建连接队列，最多接受5个连接
	listen(server_sockfd,5);
	
	while(1)
	{
	  char ch;	  
	  printf("server waiting\n");
	  client_len=sizeof(client_addr);

	  //--接受连接并进行读写操作
	  //client_sockfd=accept(server_sockfd, (struct sockaddr *)&client_addr, &client_len);
	  client_sockfd=accept(server_sockfd, NULL	, NULL);
	  
	  read(client_sockfd,&ch,1);
	  printf("server:read char from client=%c \n",ch);
	  ch++;
	  printf("server:write char [ %c] to client \n",ch);
	  write(client_sockfd,&ch,1);

	  //--关闭套接字
	  close(client_sockfd);		
	}

	return 0;
}

```
通过 ./server1 & 启动服务器程序后，再启动客户端程序，就会与服务器端进行通信。
3.套接字属性
套接字属性由3个属性确定，它们是：域(domain)、类型(type)、协议(protocol)
(1).套接字的域
域指定套接字通信中使用的网络介质，最常见的域为AF_INET，它指的是Internet网络。还有UNIX文件系统域AF_UNIX，基于IOS标准协议的网络所使用的AF_ISO域，用于施乐（Xerox）网络系统的AF_XNS域。
(2).套接字类型
一个套接字域可能有多种不同的通信方式，AF_INET域中提供了两种通信机制：流（stream）和数据报（datagram）。
流套接字：
提供一个有序、可靠、双向字节流的连接，由类型SOCK_STREAM实现。基于TCP/IP协议实现。
数据报套接字：
由SOCK_DGRAM指定的数据报套接字不建立和维护一个连接，基于UDP/IP实现，提供的是一种无序的不可靠服务，但是，相对来说开销较小，因为不用维护一个连接，速度也较快。
(3).套接字协议
如果底层的传输机制允许不止一个协议来提供要求的套接字，就可以为套接字选一个特定的协议，在网络套接字中，只需使用默认值即可。
4.套接字系统调用
(1).创建套接字
int socket(int domain,int type,int protocol)
--domain指定使用的协议簇
--type指定套接字的通信类型
--protocol指定使用的协议

(2).命名套接字
int bind(int sockfd, const struct sockaddr * address, size_t address_len)
bind系统调用把address中的地址分配给与文件描述符socket关联的未命名套接字。

(3).创建套接字队列
int listen(int socket,int backlog)
listen系统调用为服务器程序创建一个队列来保存未处理的连接请求，backlog设定为可以容纳的未处理连接的最大数目。

(4).接受连接
int accept(int sockfd, const struct sockaddr * address, size_t address_len)
服务器端通过accept系统调用来等客户建立对该套接字的连接。

(5).请求连接
int connect(int sockfd, const struct sockaddr * address, size_t address_len)
客户端通过connect系统调用为客户端未命名套接字与服务器监听套接字之间建立连接。

(6).关闭连接
调用close函数来终止服务器和客户端上的套接字连接。

example2:网络连接套接字

client2.c:

```c++
#include<sys/types.h>
#include<sys/socket.h>
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<netinet/in.h>
#include<arpa/inet.h>

int main()
{
	int sockfd,len;
	char ch='A';
	struct sockaddr_in address;
	int result;

	//--为客户创建套接字
	sockfd=socket(AF_INET,SOCK_STREAM,0);

	//--根据服务器情况给套接字命名
	address.sin_family=AF_INET;
	address.sin_addr.s_addr=inet_addr("127.0.0.1");
	address.sin_port=htons(9734);
	len=sizeof(address);

	//--连接到服务器套接字
	result=connect(sockfd,(struct sockaddr *)&address,len);

	if (-1==result)
	{
	   perror("oops:client1");
	   exit(0);
     }

	//--通过套接字进行读写
	printf("client:write char [ %c] to server \n",ch);
	write(sockfd,&ch,1);
	read(sockfd,&ch,1);
	printf("client:read char from server=%c \n",ch);

	//--关闭退出
	close(sockfd);
	exit(0);
}

```


server2.c：

```c++
#include<sys/types.h>
#include<sys/socket.h>
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<netinet/in.h>
#include<arpa/inet.h>

int main()
{
	int server_sockfd,client_sockfd;	
	struct sockaddr_in server_addr;
	struct sockaddr_in client_addr;
	int server_len;
	int client_len;
	
	//--为服务器创建一个未命名套接字
	server_sockfd=socket(AF_INET,SOCK_STREAM,0);

	//--命名套接字
	server_addr.sin_family=AF_INET;
	server_addr.sin_addr.s_addr=inet_addr("127.0.0.1");
	server_addr.sin_port=htons(9734);
	server_len=sizeof(server_addr);
	bind(server_sockfd,(struct sockaddr *) &server_addr,server_len);

	//--创建连接队列，最多接受5个连接
	listen(server_sockfd,5);
	
	while(1)
	{
	  char ch;	  
	  printf("server waiting\n");

	   //--接受连接并进行读写操作
	  client_len=sizeof(client_addr);
	  //client_sockfd=accept(server_sockfd, (struct sockaddr *)&client_addr, &client_len);
	  client_sockfd=accept(server_sockfd, NULL	, NULL);	  
	  read(client_sockfd,&ch,1);
	  printf("server:read char from client=%c \n",ch);
	  ch++;
	  printf("server:write char [ %c] to client \n",ch);
	  write(client_sockfd,&ch,1);
	  close(client_sockfd);		
	}

	return 0;
}

```


