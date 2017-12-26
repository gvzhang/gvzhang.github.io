---
title: 异步阻塞IO和异步非阻塞IO对比
categories:
 - 网络编程
tags:
 - socket
 - io多路复用
---

## 异步阻塞模式
- 带有阻塞通知的非阻塞 I/O
- 配置的是非阻塞 I/O，然后使用阻塞 select 系统调用来确定一个 I/O

## 异步非阻塞模式
- 一种处理与 I/O 重叠(并行)进行的模型

## 异步阻塞代码：
```c
/**
 * 异步阻塞模式
 * 带有阻塞通知的非阻塞 I/O
 * 配置的是非阻塞 I/O，然后使用阻塞 select 系统调用来确定一个 I/O
 * 描述符何时有操作
 * select
 * 调用非常有趣的是它可以用来为多个描述符提供通知，而不仅仅为一个描述符提供通知
 */
#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/select.h>
#include <sys/socket.h>
#include <sys/time.h>

#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#define SERVPORT 1234
#define MAXDATASIZE 100
#define IPADDRESS "127.0.0.1"
#define TFILE "data_from_socket_ab.txt"

int main(void) {
  int sockfd, recvbytes;
  char rcv_buf[MAXDATASIZE]; /*./client 127.0.0.1 hello */
  char snd_buf[MAXDATASIZE];
  struct sockaddr_in server_addr;

  fd_set readset, writeset;
  int check_timeval = 1;
  struct timeval timeout = {check_timeval, 0}; //阻塞式select, 等待1秒，1秒轮询
  int maxfd;
  int fp;
  int cir_count = 0;
  int ret;

  if ((fp = open(TFILE, O_WRONLY)) < 0) //不是用fopen
  {
    perror("fopen:");
    exit(1);
  }

  server_addr.sin_family = AF_INET;
  server_addr.sin_port = htons(SERVPORT);
  inet_pton(AF_INET, IPADDRESS, &server_addr.sin_addr);
  memset(&(server_addr.sin_zero), 0, 8);

  for (int z = 0; z < 8; z++) {
    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
      perror("socket:");
      exit(1);
    }

    /* create the connection by socket
     * means that connect "sockfd" to "server_addr"
     */
    if (connect(sockfd, (struct sockaddr *)&server_addr,
                sizeof(struct sockaddr)) == -1) {
      perror("connect");
      exit(1);
    }

    sprintf(snd_buf, "Hello%d", z);
    if (send(sockfd, snd_buf, sizeof(snd_buf), 0) == -1) {
      perror("send:");
      exit(1);
    }
    printf("send:%s\n", snd_buf);

    maxfd = sockfd > fp ? (sockfd + 1) : (fp + 1); //描述符最大值加1

    while (1) {
      /*
      下面是这些宏的详细描述：
        1、FD_ZERO(fd_set *
        set)，是把集合清空（初始化为0，确切的说，是把集合中的元素个数初始化为0，并不修改描述符数组).使用集合前，必须用FD_ZERO初始化，否则集合在栈上作为自动变量分配时，fd_set分配的将是随机值，导致不可预测的问题。
        2、FD_SET(int s,fd_set *
        set)，向集合中加入一个套接口描述符（如果该套接口描述符s没在集合中，并且数组中已经设置的个数小于最大个数时，就把该描述符加入到集合中，集合元素个数加1）。这里是将s的值直接放入数组中。
        3、FD_ISSET(int s,fd_set *
        set)，检查描述符是否在集合中，如果在集合中返回非0值，否则返回0.
        它的宏定义并没有给出具体实现，但实现的思路很简单，就是搜索集合，判断套接字s是否在数组中。
        4、FD_CLR(int s,fd_set *
        set)，从集合中移出一个套接口描述符（比如一个套接字连接中断后，就应该移除它）。实现思路是，在数组集合中找到对应的描述符，然后把后面的描述依次前移一个位置，最后把描述符的个数减1。

      下面是使用这些宏的基本方式。
        1、调用FD_ZERO来初始化fd_set；
        2、调用FD_SET将感兴趣的套接字描述符加入fd_set集合中（每次循环都要重新加入，因为select更新后，会将一些没有满足条件的套接字移除队列）；
        3、设置等待时间后，调用select函数--更新套接字的状态；
        4、调用FD_ISSET，来判断套接字是否有相应状态，然后做相应操作，比如，如果套接字可读，就调用recv函数去接收数据。
      */
      FD_ZERO(&readset); //每次循环都要清空集合，否则不能检测描述符变化
      FD_SET(sockfd, &readset); //添加描述符
      FD_ZERO(&writeset);
      FD_SET(fp, &writeset);

      /*
      int select(int maxfdp, fd_set *readfds, fd_set *writefds,
                fd_set *errorfds, struct timeval *timeout);
      参数列表
        int
        maxfdp是一个整数值，是指集合中所有文件描述符的范围，即所有文件描述符的最大值加1，不能错！在Windows中这个参数的值无所谓，可以设置不正确。
        　　
        fd_set
        *readfds是指向fd_set结构的指针，这个集合中应该包括文件描述符，我们是要监视这些文件描述符的读变化的，即我们关心是否可以从这些文件中读取数据了，如果这个集合中有一个文件可读，select就会返回一个大于0的值，表示有文件可读，如果没有可读的文件，则根据timeout参数再判断是否超时，若超出timeout的时间，select返回0，若发生错误返回负值。可以传入NULL值，表示不关心任何文件的读变化。
        　　
        fd_set
        *writefds是指向fd_set结构的指针，这个集合中应该包括文件描述符，我们是要监视这些文件描述符的写变化的，即我们关心是否可以向这些文件中写入数据了，如果这个集合中有一个文件可写，select就会返回一个大于0的值，表示有文件可写，如果没有可写的文件，则根据timeout参数再判断是否超时，若超出timeout的时间，select返回0，若发生错误返回负值。可以传入NULL值，表示不关心任何文件的写变化。
        　　
        fd_set *errorfds同上面两个参数的意图，用来监视文件错误异常。
        　　
        struct timeval*
        timeout是select的超时时间，这个参数至关重要，它可以使select处于三种状态：
        第一，若将NULL以形参传入，即不传入时间结构，就是将select置于阻塞状态，一定等到监视文件描述符集合中某个文件描述符发生变化为止；
        第二，若将时间值设为0秒0毫秒，就变成一个纯粹的非阻塞函数，不管文件描述符是否有变化，都立刻返回继续执行，文件无变化返回0，有变化返回一个正值；
        第三，timeout的值大于0，这就是等待的超时时间，即
        select在timeout时间内阻塞，超时时间之内有事件到来就返回了，否则在超时后不管怎样一定返回，返回值同上述。

      返回值：
        负值：select错误
        正值：某些文件可读写或出错
        0：等待超时，没有可读写或错误的文件
      */
      ret = select(maxfd, &readset, NULL, NULL, NULL);
      switch (ret) {
      case -1:
        exit(-1);
        break;
      case 0:
        //因为select采用阻塞模式，SOCKET肯定发生了变化
        //所以不会进入到这里
        break;
      default:
        if (FD_ISSET(sockfd, &readset)) //测试sock是否可读，即是否网络上有数据
        {
          recvbytes = recv(sockfd, rcv_buf, MAXDATASIZE, MSG_DONTWAIT);
          rcv_buf[recvbytes] = '\0';
          printf("recv:%s\n", rcv_buf);

          if (FD_ISSET(fp, &writeset)) {
            write(fp, rcv_buf, strlen(rcv_buf)); // 不是用fwrite
          }
          goto end;
        }
      }
      cir_count++;
      printf("CNT : %d \n", cir_count);
    }

  end:
    close(sockfd);
  }
  close(fp);
  return 0;
}
```

## 异步非阻塞代码：
```c
/**
 * 异步非阻塞模式
 * 一种处理与 I/O 重叠(并行)进行的模型
 * steps：
 * a. 调用read;
 * b. read请求会立即返回，说明请求已经成功发起了。
 * c. 在后台完成读操作这段时间内，应用程序可以执行其他处理操作。
 * d. 当
 * read的响应到达时，就会产生一个信号或执行一个基于线程的回调函数来完成这次
 * I/O处理过程。
 */
#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/select.h>
#include <sys/socket.h>
#include <sys/time.h>

#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#define SERVPORT 1234
#define MAXDATASIZE 100
#define IPADDRESS "127.0.0.1"
#define TFILE "data_from_socket_an.txt"

int main(void) {
  int sockfd, recvbytes;
  char rcv_buf[MAXDATASIZE]; /*./client 127.0.0.1 hello */
  char snd_buf[MAXDATASIZE];

  struct sockaddr_in server_addr;

  fd_set readset, writeset;
  int check_timeval = 1;
  struct timeval timeout = {check_timeval, 0}; //阻塞式select, 等待1秒，1秒轮询
  int maxfd;
  int fp;
  int cir_count = 0;
  int ret;

  if ((fp = open(TFILE, O_WRONLY)) < 0) //不是用fopen
  {
    perror("fopen:");
    exit(1);
  }

  server_addr.sin_family = AF_INET;
  server_addr.sin_port = htons(SERVPORT);
  inet_pton(AF_INET, IPADDRESS, &server_addr.sin_addr);
  memset(&(server_addr.sin_zero), 0, 8);

  for (int z = 0; z < 8; z++) {
    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
      perror("socket:");
      exit(1);
    }

    /* create the connection by socket
     * means that connect "sockfd" to "server_addr"
     */
    if (connect(sockfd, (struct sockaddr *)&server_addr,
                sizeof(struct sockaddr)) == -1) {
      perror("connect");
      exit(1);
    }

    sprintf(snd_buf, "Hello%d", z);
    if (send(sockfd, snd_buf, sizeof(snd_buf), 0) == -1) {
      perror("send:");
      exit(1);
    }
    printf("send:%s\n", snd_buf);

    maxfd = sockfd > fp ? (sockfd + 1) : (fp + 1); //描述符最大值加1

    while (1) {
      FD_ZERO(&readset); //每次循环都要清空集合，否则不能检测描述符变化
      FD_SET(sockfd, &readset); //添加描述符
      FD_ZERO(&writeset);
      FD_SET(fp, &writeset);

      ret = select(maxfd, &readset, NULL, NULL, &timeout); // 非阻塞模式
      switch (ret) {
      case -1:
        exit(-1);
        break;
      case 0:
        //轮询1秒后，SOCKET未发生变化
        break;
      default:
        //轮询1秒后，SOCKET发生了变化
        if (FD_ISSET(sockfd, &readset)) //测试sock是否可读，即是否网络上有数据
        {
          recvbytes = recv(sockfd, rcv_buf, MAXDATASIZE, MSG_DONTWAIT);
          rcv_buf[recvbytes] = '\0';
          printf("recv:%s\n", rcv_buf);

          if (FD_ISSET(fp, &writeset)) {
            write(fp, rcv_buf, strlen(rcv_buf)); // 不是用fwrite
          }
          goto end;
        }
      }
      timeout.tv_sec =
          check_timeval; // 必须重新设置，因为超时时间到后会将其置零

      cir_count++;
      printf("CNT : %d \n", cir_count);
    }

  end:
    close(sockfd);
  }
  close(fp);
  return 0;
}
```

## 测试结果
![client-asyn-block.png](https://zgjian-pic.oss-cn-beijing.aliyuncs.com/post/58b128709bfc2.png "client-asyn-block.png")

说明：因为在select中采用的是阻塞模式，SOCKET肯定发生了变化，所以不会执行`cir_count++;`

![client-asyn-nonblock.png](https://zgjian-pic.oss-cn-beijing.aliyuncs.com/post/58b128790a93d.png "client-asyn-nonblock.png")

说明：select不进行阻塞，在等待超时时返回0，然后执行`cir_count++;`

参考文件：http://blog.chinaunix.net/uid-26000296-id-3755268.html
