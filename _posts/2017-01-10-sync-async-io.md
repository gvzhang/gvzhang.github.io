---
title: Socket的同步阻塞IO和IO多路复用对比
categories:
 - 网络编程
tags:
 - socket
 - io多路复用
---

## 同步阻塞模式
在这个模式中，用户空间的应用程序执行一个系统调用，并阻塞，直到系统调用完成为止(数据传输完成或*发生错误)

## IO多路复用
- 多路网络连接复用一个IO线程
- 单个线程通过记录跟踪每一个Sock(I/O流)的状态来同时管理多个I/O流

## 测试结果
### 同步阻塞IO：
![server-同步-阻塞](https://zgjian-pic.oss.cn-beijing.aliyuncs.com/post/58b117950c953.png "server-同步-阻塞")

###### 可见连接1、连接2、连接3是依次执行的，当其中一个连接出现阻塞，比如连接1阻塞5秒，那么下一个连接2也会被阻塞了5秒才能被执行。

### IO多路复用：
![server-IO复用](https://zgjian-pic.oss.cn-beijing.aliyuncs.com/post/58b1124a66b70.png "server-IO复用")
说明：由于是交替执行的，所以不好区分是哪个连接，不过可以通过下图来对比

##### 连接1-阻塞5秒：
![client1-阻塞5秒](https://zgjian-pic.oss.cn-beijing.aliyuncs.com/post/58b1126d11192.png "client1-阻塞5秒")

##### 连接2：
![client2](https://zgjian-pic.oss.cn-beijing.aliyuncs.com/post/58b11273db80e.png "client2")

##### 连接3：
![client3](https://zgjian-pic.oss.cn-beijing.aliyuncs.com/post/58b1127acff3f.png "client3")

###### 可以看到连接1每次都阻塞了5秒才继续执行的，但是连接2和连接3却并没有受到阻塞的影响，马上就执行完了。

## client端代码：
```c
/**
 * 同步阻塞模式
 * 在这个模式中，用户空间的应用程序执行一个系统调用，并阻塞，直到系统调用完成为止(数据传输完成或*
 * 发生错误)，在结果没有返回时，程序什么也不做
 */
#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <time.h>
#define SERVPORT 1234
#define MAXDATASIZE 100
#define IPADDRESS "127.0.0.1"

int main(void) {
  time_t rawtime;
  struct tm *timeinfo;
  int sockfd, recvbytes;
  char rcv_buf[MAXDATASIZE]; /*./client 127.0.0.1 hello */
  char snd_buf[MAXDATASIZE];

  struct sockaddr_in server_addr;

  server_addr.sin_family = AF_INET;
  server_addr.sin_port = htons(SERVPORT);
  inet_pton(AF_INET, IPADDRESS, &server_addr.sin_addr);
  memset(&(server_addr.sin_zero), 0, 8);

  for (int z = 0; z < 2; z++) {
    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
      perror("socket:");
      exit(1);
    }

    /* create the connection by socket
     * means that connect "sockfd" to "server_addr"
     * 同步阻塞模式
     */
    if (connect(sockfd, (struct sockaddr *)&server_addr,
                sizeof(struct sockaddr)) == -1) {
      perror("connect");
      exit(1);
    }
    time(&rawtime);
    timeinfo = localtime(&rawtime);
    printf("connect:%d     %d:%d\n", sockfd, timeinfo->tm_min,
           timeinfo->tm_sec);

    sprintf(snd_buf, "Hello%d", z);

    // sleep(5);

    /* 同步阻塞模式  */
    if (send(sockfd, snd_buf, sizeof(snd_buf), 0) == -1) {
      perror("send:");
      exit(1);
    }
    time(&rawtime);
    timeinfo = localtime(&rawtime);
    printf("send:%s     %d:%d\n", snd_buf, timeinfo->tm_min, timeinfo->tm_sec);

    /* 同步阻塞模式  */
    if ((recvbytes = recv(sockfd, rcv_buf, MAXDATASIZE, 0)) == -1) {
      perror("recv:");
      exit(1);
    }

    rcv_buf[recvbytes] = '\0';
    printf("recv:%s     %d:%d\n\n", rcv_buf, timeinfo->tm_min,
           timeinfo->tm_sec);

    close(sockfd);
  }
  return 0;
}
```

## server端同步阻塞代码：
```c
/**
 * server端同步阻塞
 */
#include <arpa/inet.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <time.h>
#define SERVPORT 1234
#define BACKLOG 10 // max numbef of client connection
#define MAXDATASIZE 100

int main(void) {
  time_t rawtime;
  struct tm *timeinfo;
  int sockfd, client_fd, addr_size, recvbytes;
  char rcv_buf[MAXDATASIZE];
  char *val;
  struct sockaddr_in server_addr;
  struct sockaddr_in client_addr;
  int bReuseaddr = 1;

  char IPdotdec[20];

  /* create a new socket and regiter it to os .
   * SOCK_STREAM means that supply tcp service,
   * and must connect() before data transfort.
   */
  if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
    perror("socket:");
    exit(1);
  }

  /* setting server's socket */
  server_addr.sin_family = AF_INET; // IPv4 network protocol
  server_addr.sin_port = htons(SERVPORT);
  server_addr.sin_addr.s_addr = INADDR_ANY; // auto IP detect
  memset(&(server_addr.sin_zero), 0, 8);

  setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (const char *)&bReuseaddr,
             sizeof(int));
  if (bind(sockfd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr)) ==
      -1) {
    perror("bind:");
    exit(1);
  }

  /*
   * watting for connection ,
   * and server permit to recive the requestion from sockfd
   */
  if (listen(sockfd, BACKLOG) ==
      -1) // BACKLOG assign thd max number of connection
  {
    perror("listen:");
    exit(1);
  }

  while (1) {
    addr_size = sizeof(struct sockaddr_in);

    /*
     * accept the sockfd's connection,
     * return an new socket and assign far host to client_addr
     */
    if ((client_fd = accept(sockfd, (struct sockaddr *)&client_addr,
                            &addr_size)) == -1) {
      /* Nonblocking mode */
      perror("accept:");
      continue;
    }

    /**
     * sin_port和sin_addr都必须是NBO，一般可视化的数字都是HBO（本机字节顺序）
     * 地址的表达格式通常是ASCII字符串，数值格式则是存放到套接字地址结构的二进制值
     *
     * const char * inet_ntop(int family, const void *addrptr, char *strptr,
     * size_t len);
     * 函数中p和n分别代表表达（presentation)和数值（numeric)。
     * 从数值格式（addrptr）转换到表达式（strptr)。
     * len参数是目标存储单元的大小，以免该函数溢出其调用者的缓冲区
     */
    inet_ntop(AF_INET, (void *)&client_addr, IPdotdec, 16);

    time(&rawtime);
    timeinfo = localtime(&rawtime);
    printf("adding client on fd %d     %d:%d\n\n", client_fd, timeinfo->tm_min,
           timeinfo->tm_sec);

    /**
     * 同步阻塞在单个连接上，直到接收到该连接数据
     */
    if ((recvbytes = recv(client_fd, rcv_buf, MAXDATASIZE, 0)) == -1) {
      perror("recv:");
      exit(1);
    }
    sleep(1);

    time(&rawtime);
    timeinfo = localtime(&rawtime);
    printf("serving client on fd %d     %d:%d\n\n", client_fd, timeinfo->tm_min,
           timeinfo->tm_sec);
    /**
     * 同步阻塞在单个连接上，直到可发送数据到该连接上
     */
    if (send(client_fd, rcv_buf, strlen(rcv_buf), 0) == -1) {
      perror("send:");
      exit(1);
    }

    close(client_fd);
    time(&rawtime);
    timeinfo = localtime(&rawtime);
    printf("removing client on fd %d     %d:%d\n\n", client_fd,
           timeinfo->tm_min, timeinfo->tm_sec);
  }

  close(sockfd);
  return 0;
}
```
## server端异步非阻塞代码：
```c
/**
 * server端异步非阻塞
 */
#include <arpa/inet.h>
#include <assert.h>
#include <errno.h>
#include <netinet/in.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/select.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <time.h>
#include <unistd.h>

#define IPADDR "127.0.0.1"
#define PORT 1234
#define MAXLINE 1024
#define LISTENQ 5
#define SIZE 10

time_t rawtime;
struct tm *timeinfo;

typedef struct server_context_st {
  int cli_cnt;      /*客户端个数*/
  int clifds[SIZE]; /*客户端的个数*/
  fd_set allfds;    /*句柄集合*/
  int maxfd;        /*句柄最大值*/
} server_context_st;
static server_context_st *s_srv_ctx = NULL;
/*===========================================================================
 * ==========================================================================*/
static int create_server_proc(const char *ip, int port) {
  int fd;
  struct sockaddr_in servaddr;
  fd = socket(AF_INET, SOCK_STREAM, 0);
  if (fd == -1) {
    fprintf(stderr, "create socket fail,erron:%d,reason:%s\n", errno,
            strerror(errno));
    return -1;
  }

  /*一个端口释放后会等待两分钟之后才能再被使用，SO_REUSEADDR是让端口释放后立即就可以被再次使用。*/
  int reuse = 1;
  if (setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &reuse, sizeof(reuse)) == -1) {
    return -1;
  }

  bzero(&servaddr, sizeof(servaddr));
  servaddr.sin_family = AF_INET;
  inet_pton(AF_INET, ip, &servaddr.sin_addr);
  servaddr.sin_port = htons(port);

  if (bind(fd, (struct sockaddr *)&servaddr, sizeof(servaddr)) == -1) {
    perror("bind error: ");
    return -1;
  }

  listen(fd, LISTENQ);

  return fd;
}

static int accept_client_proc(int srvfd) {
  struct sockaddr_in cliaddr;
  socklen_t cliaddrlen;
  cliaddrlen = sizeof(cliaddr);
  int clifd = -1;

ACCEPT:
  clifd = accept(srvfd, (struct sockaddr *)&cliaddr, &cliaddrlen);

  if (clifd == -1) {
    if (errno == EINTR) {
      goto ACCEPT;
    } else {
      fprintf(stderr, "accept fail,error:%s\n", strerror(errno));
      return -1;
    }
  }

  time(&rawtime);
  timeinfo = localtime(&rawtime);
  printf("adding client on fd %d     %d:%d\n\n", clifd, timeinfo->tm_min,
         timeinfo->tm_sec);

  //将新的连接描述符添加到数组中
  int i = 0;
  for (i = 0; i < SIZE; i++) {
    if (s_srv_ctx->clifds[i] < 0) {
      s_srv_ctx->clifds[i] = clifd;
      s_srv_ctx->cli_cnt++;
      break;
    }
  }

  if (i == SIZE) {
    fprintf(stderr, "too many clients.\n");
    return -1;
  }
}

static int handle_client_msg(int fd, char *buf) {
  assert(buf);

  sleep(1);
  time(&rawtime);
  timeinfo = localtime(&rawtime);
  printf("serving client on fd %d     %d:%d\n\n", fd, timeinfo->tm_min,
         timeinfo->tm_sec);

  write(fd, buf, strlen(buf) + 1);
  return 0;
}

static void recv_client_msg(fd_set *readfds) {
  int i = 0, n = 0;
  int clifd;
  char buf[MAXLINE] = {0};
  for (i = 0; i <= s_srv_ctx->cli_cnt; i++) {
    clifd = s_srv_ctx->clifds[i];
    if (clifd < 0) {
      continue;
    }
    /*判断客户端套接字是否有数据*/
    if (FD_ISSET(clifd, readfds)) {
      //接收客户端发送的信息
      n = read(clifd, buf, MAXLINE);
      if (n <= 0) {
        /*n==0表示读取完成，客户都关闭套接字*/
        FD_CLR(clifd, &s_srv_ctx->allfds);
        close(clifd);
        s_srv_ctx->clifds[i] = -1;
        continue;
      }
      handle_client_msg(clifd, buf);
    }
  }
}
static void handle_client_proc(int srvfd) {
  int clifd = -1;
  int retval = 0;
  fd_set *readfds = &s_srv_ctx->allfds;
  struct timeval tv;
  int i = 0;

  while (1) {
    /*每次调用select前都要重新设置文件描述符和时间，因为事件发生后，文件描述符和时间都被内核修改啦*/
    FD_ZERO(readfds);
    /*添加监听套接字*/
    FD_SET(srvfd, readfds);
    s_srv_ctx->maxfd = srvfd;

    tv.tv_sec = 30;
    tv.tv_usec = 0;
    /*添加客户端套接字*/
    for (i = 0; i < s_srv_ctx->cli_cnt; i++) {
      clifd = s_srv_ctx->clifds[i];
      /*去除无效的客户端句柄*/
      if (clifd != -1) {
        FD_SET(clifd, readfds);
      }
      s_srv_ctx->maxfd = (clifd > s_srv_ctx->maxfd ? clifd : s_srv_ctx->maxfd);
    }

    /**
     * 为什么第一个参数的值为所有文件描述符的最大值加1？
     * select是*轮询*监听处理服务端和客户端套接字（多个套接字）
     * 服务器套接字监听进来的请求（生成客户端套接字），客户端套接字监听请求是否发生改变
     * 所以select就需要传入 最大的文件描述符+1（0开始） 来进行*遍历*
     */
    retval = select(s_srv_ctx->maxfd + 1, readfds, NULL, NULL, &tv);
    if (retval == -1) {
      fprintf(stderr, "select error:%s.\n", strerror(errno));
      return;
    }
    if (retval == 0) {
      // fprintf(stdout, "select is timeout.\n");
      continue;
    }
    if (FD_ISSET(srvfd, readfds)) {
      /*监听客户端请求*/
      accept_client_proc(srvfd);
    } else {
      /*接受处理客户端消息*/
      recv_client_msg(readfds);
    }
  }
}

static void server_uninit() {
  if (s_srv_ctx) {
    free(s_srv_ctx);
    s_srv_ctx = NULL;
  }
}

static int server_init() {
  s_srv_ctx = (server_context_st *)malloc(sizeof(server_context_st));
  if (s_srv_ctx == NULL) {
    return -1;
  }

  memset(s_srv_ctx, 0, sizeof(server_context_st));

  int i = 0;
  for (; i < SIZE; i++) {
    s_srv_ctx->clifds[i] = -1;
  }

  return 0;
}

int main(int argc, char *argv[]) {
  int srvfd;
  /*初始化服务端context*/
  if (server_init() < 0) {
    return -1;
  }
  /*创建服务,开始监听客户端请求*/
  srvfd = create_server_proc(IPADDR, PORT);
  if (srvfd < 0) {
    fprintf(stderr, "socket create or bind fail.\n");
    goto err;
  }
  /*开始接收并处理客户端请求*/
  handle_client_proc(srvfd);
  server_uninit();
  return 0;
err:
  server_uninit();
  return -1;
}
```


|   参考文件|http://blog.chinaunix.net/uid-26000296-id-3755264.html |
| ------------ | ------------ |
|   |  http://www.cnblogs.com/Anker/p/3258674.html |
