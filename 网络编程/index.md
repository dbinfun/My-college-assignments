# 实验一：TCP

## `tcp_server.c`

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#define PORT    11111//端口
#define BACKLOG 1
#define MAXDATASIZE 100
/**
 * 程序只考虑输入ascii码
*/
int main(void)
{
    int id[]={1,1,1,1,1,1,1,1,1,1};//id 学号
    int listenfd_d, connectfd_d;// 姓名首字母d结尾
    struct sockaddr_in server_d;
    struct sockaddr_in client_d;
    socklen_t addrlen;
    char buf[MAXDATASIZE];// 缓冲
    // 创建tcp/ipv4 面向tcp流的套接字
    if((listenfd_d = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
                perror("socket() error.\n");
                exit(1);
    }

    int opt = SO_REUSEADDR;//打开或关闭地址复用功能

    setsockopt(listenfd_d, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    bzero(&server_d, sizeof(server_d));//置0
    server_d.sin_family = AF_INET;// ipv4
    server_d.sin_port = htons(PORT); //设置端口,主机数转换为网络序
    server_d.sin_addr.s_addr = htonl(INADDR_ANY); // ip地址，此处是通配
    // 绑定到套接字体
    if (bind(listenfd_d, (struct sockaddr *)&server_d, sizeof(struct sockaddr)) == -1)
    {
                perror("bind() error.\n");
                exit(1);
    }
    // 监听套接字，监听队列长度1
    if (listen(listenfd_d, BACKLOG) == -1) 
    {
        perror("listen() error.\n");
        exit(1);
    }

    addrlen = sizeof(client_d);

    while(1) 
    {
        int n,m;
        // 接受连接
        if ((connectfd_d = accept(listenfd_d, (struct sockaddr *)&client_d, &addrlen)) == -1)
        {
            perror("accept() error.\n");
            exit(1);
        }
        // 网络地址转换为ascii码，网络序转换为主机序并输出。
        printf("客户端ip: %s, 端口: %d\n", inet_ntoa(client_d.sin_addr), ntohs(client_d.sin_port));
        // 循环监听数据,其中n表示的长度包含了\0
        while((n = read(connectfd_d, buf, MAXDATASIZE)) > 0)
        {
            // 计算分组是否不足
            char rever[n];
            for(int i=0,j=n-2;i<n-1;i++,j--){
                rever[i] = buf[j];
            }
            rever[n-1]='\0';//加上结束标志
            printf("收到信息:%s\n", buf);
            
            if (strcmp(buf, "quit") == 0)
            {
                exit(1);
            }
            printf("倒置后的信息:%s\n",rever);
            write(connectfd_d, rever, n);
        }
       close(connectfd_d);
    }
    close(listenfd_d);
}
```

## `tcp_client.c`

```c
/*client.c*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
#define PORT            11111
#define MAXDATASIZE     100

int main(int argc, char *argv[])
{
    argv[1] = "127.0.0.1";
    if(argc==1)argc++;
    int fd, numbytes, i;
    // 缓冲区
    char buf[MAXDATASIZE];
    char sendStr[MAXDATASIZE], recStr[MAXDATASIZE];
    struct hostent * he; //主机信息
    struct sockaddr_in server; // 套接字地址结构
    if (argc != 2)
    {
        printf("Usage: %s  <IP address>\n", argv[0]);
        exit(1);
    }
    // 查询主机信息
    if ((he = gethostbyname(argv[1])) == NULL) 
    {
        perror("gethostbyname() error.\n");
        exit(1);
    }

    // 创建tcp/ipv4套接字 tcp数据流 套接字
    if ((fd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
        perror("socket() error.\n");
        exit(1);
    }
    // 数据填充为0
    bzero(&server, sizeof(server));
    server.sin_family = AF_INET; // 设置族 ipv4
    server.sin_port = htons(PORT); // 端口
    server.sin_addr = *((struct in_addr *) he->h_addr); // 设置ip地址
    // 建立连接
    if (i = connect(fd, (struct sockaddr *)&server, sizeof(server)) == -1)
    {
        perror("connect() error\n.");
        exit(1);
    }
    while(i != -1)
    {
        printf("请输入消息:");
        scanf("%s", &sendStr);
        sendStr[strlen(sendStr)]='\0';
        if(strcmp(sendStr, "quit") == 0)
        {
            write(fd, sendStr, strlen(sendStr)+1);
            exit(1);
        }
        write(fd, sendStr, strlen(sendStr)+1);  
        read(fd, recStr, MAXDATASIZE);
        printf("收到倒置后的字符串:%s\n",recStr);
    }
    close(fd);
}
```

# 实验二:UDP

## `udp_server.c`

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#define PORT    11111//
#define BACKLOG 1
#define MAXDATASIZE 100
/**
 * 程序只考虑输入ascii码
*/
int main(void)
{
    int id[]={1,1,1,1,1,1,1,1,1,1};
    int listenfd_d;// 首字母d结尾
    struct sockaddr_in server_d;
    struct sockaddr_in client_d;
    socklen_t addrlen;
    char buf[MAXDATASIZE];// 缓冲
    // 创建tcp/ipv4 面向tcp流的套接字
    if((listenfd_d = socket(AF_INET, SOCK_DGRAM, 0)) == -1)
    {
                perror("socket() error.\n");
                exit(1);
    }

    int opt = SO_REUSEADDR;//打开或关闭地址复用功能

    setsockopt(listenfd_d, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    bzero(&server_d, sizeof(server_d));//置0
    server_d.sin_family = AF_INET;// ipv4
    server_d.sin_port = htons(PORT); //设置端口,主机数转换为网络序
    server_d.sin_addr.s_addr = htonl(INADDR_ANY); // ip地址，此处是通配
    // 绑定到套接字体
    if (bind(listenfd_d, (struct sockaddr *)&server_d, sizeof(struct sockaddr)) == -1)
    {
                perror("bind() error.\n");
                exit(1);
    }

    addrlen = sizeof(client_d);

    while(1) 
    {
        int n;
        
        // 循环监听数据,其中n表示的长度包含了\0
        while((n = recvfrom(listenfd_d,buf, MAXDATASIZE,0,(struct sockaddr *)&client_d, &addrlen)) > 0)
        {
            char rever[n];
            for(int i=0,j=n-2;i<n-1;i++,j--){
                rever[i] = buf[j];
            }
            rever[n-1]='\0';
            printf("收到信息:%s\n", buf);
            
            if (strcmp(buf, "quit") == 0)
            {
                exit(1);
            }
            printf("倒置后的信息:%s\n",rever);
            fflush(stdout);
            sendto(listenfd_d, rever,n,0,(struct sockaddr*)&client_d,sizeof(client_d));
        }
    }
    close(listenfd_d);
}
```

## `udp_client.c`

```c
/*client.c*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
#define PORT            11111
#define MAXDATASIZE     100

int main(int argc, char *argv[])
{
    argv[1] = "127.0.0.1";
    if(argc==1)argc++;
    int fd, numbytes, i;
    // 缓冲区
    char buf[MAXDATASIZE];
    char sendStr[MAXDATASIZE], recStr[MAXDATASIZE];
    struct hostent * he; //主机信息
    struct sockaddr_in server,perr; // 套接字地址结构
    if (argc != 2)
    {
        printf("Usage: %s  <IP address>\n", argv[0]);
        exit(1);
    }
    // 查询主机信息
    if ((he = gethostbyname(argv[1])) == NULL) 
    {
        perror("gethostbyname() error.\n");
        exit(1);
    }

    // 创建tcp/ipv4套接字 tcp数据流 套接字
    if ((fd = socket(AF_INET, SOCK_DGRAM, 0)) == -1)
    {
        perror("socket() error.\n");
        exit(1);
    }
    // 数据填充为0
    bzero(&server, sizeof(server));
    server.sin_family = AF_INET; // 设置族 ipv4
    server.sin_port = htons(PORT); // 端口
    server.sin_addr = *((struct in_addr *) he->h_addr); // 设置ip地址
    i=1;
    while(i>0)
    {
        printf("请输入消息:");
        scanf("%s", &sendStr);
        sendStr[strlen(sendStr)]='\0';
        if(strcmp(sendStr, "quit") == 0)
        {
            sendto(fd, sendStr, strlen(sendStr)+1,0,(struct sockaddr*)&server,sizeof(server));
            exit(1);
        }
        socklen_t len = sizeof(perr);
        sendto(fd, sendStr, strlen(sendStr)+1,0,(struct sockaddr*)&server,sizeof(server));
        int l = recvfrom(fd,buf, MAXDATASIZE,0,(struct sockaddr *)&perr, &len);
        printf("收到倒置后的字符串:%s\n",buf);
    }
    close(fd);
}
```

# 实验三:多进程

## `tcp_process_server.c`

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#define PORT    11111//
#define BACKLOG 1
#define MAXDATASIZE 100
/**
 * 程序只考虑输入ascii码
*/
void process(int connfd,struct sockaddr_in client);
const int id[]={1,1,1,1,1,1,1,1,1,1};
int main(void)
{
    
    int listenfd_d, connectfd_d; //首字母d结尾
    struct sockaddr_in server_d;
    struct sockaddr_in client_d;
    socklen_t addrlen;
    char buf[MAXDATASIZE];// 缓冲
    // 创建tcp/ipv4 面向tcp流的套接字
    if((listenfd_d = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
                perror("socket() error.\n");
                exit(1);
    }

    int opt = SO_REUSEADDR;//打开或关闭地址复用功能

    setsockopt(listenfd_d, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    bzero(&server_d, sizeof(server_d));//置0
    server_d.sin_family = AF_INET;// ipv4
    server_d.sin_port = htons(PORT); //设置端口,主机数转换为网络序
    server_d.sin_addr.s_addr = htonl(INADDR_ANY); // ip地址，此处是通配
    // 绑定到套接字体
    if (bind(listenfd_d, (struct sockaddr *)&server_d, sizeof(struct sockaddr)) == -1)
    {
                perror("bind() error.\n");
                exit(1);
    }
    // 监听套接字，监听队列长度1
    if (listen(listenfd_d, BACKLOG) == -1) 
    {
        perror("listen() error.\n");
        exit(1);
    }

    addrlen = sizeof(client_d);

    while(1) 
    {
        int n,m;
        // 接受连接
        if ((connectfd_d = accept(listenfd_d, (struct sockaddr *)&client_d, &addrlen)) == -1)
        {
            perror("accept() error.\n");
            exit(1);
        }
        // 网络地址转换为ascii码，网络序转换为主机序并输出。
        printf("客户端ip: %s, 端口: %d\n", inet_ntoa(client_d.sin_addr), ntohs(client_d.sin_port));
        //send(connectfd_d,"Welcome to my server\n",22, 0);
        //开始接收客户端的主机名字
        n = read(connectfd_d,buf,MAXDATASIZE);
        if(n>0){
            buf[n+1]='\0';
            printf("客户端的主机名是:%s\n",buf);
        }
        int pid;
        if(pid=fork()>0){
            close(connectfd_d);
            continue;
        }else if(pid==0){
            close(listenfd_d);
            process(connectfd_d,client_d);
            exit(0);
        }
    }
    close(listenfd_d);
}
void process(int connfd,struct sockaddr_in client){
    int n,m;
    char buf[MAXDATASIZE];
    while((n = read(connfd, buf, MAXDATASIZE)) > 0)
    {
        // 计算分组是否不足
        int m = 10-(n-1)%10;
        m %= 10;
        char rever[n+m];
        strcpy(rever,buf);
        // 补0
        for(int j=0;j<m;j++){
            rever[n-1+j]='0';
        }
        for (int i = 0,j=0; i < m+n-1; i++,j++)
        {
            rever[i] += id[j%10];// 直接可学号相加
        }
        rever[m+n-1] = '\0';
        printf("收到信息:%s\n", buf);
        
        if (strcmp(rever, "quit") == 0)
        {
            exit(1);
        }
        printf("加密后的信息:%s\n",rever);
        write(connfd, rever, n+m);
        bzero(&buf, sizeof(buf));
    }
    close(connfd);
}
```

## `tcp_process_client.c`

```c
/*client.c*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
#define PORT            11111
#define MAXDATASIZE     100

int main(int argc, char *argv[])
{
    argv[1] = "127.0.0.1";
    if(argc==1)argc++;
    int fd, numbytes, i;
    // 缓冲区
    char buf[MAXDATASIZE];
    char sendStr[MAXDATASIZE], recStr[MAXDATASIZE];
    struct hostent * he; //主机信息
    struct sockaddr_in server; // 套接字地址结构
    if (argc != 2)
    {
        printf("Usage: %s  <IP address>\n", argv[0]);
        exit(1);
    }
    // 查询主机信息
    if ((he = gethostbyname(argv[1])) == NULL) 
    {
        perror("gethostbyname() error.\n");
        exit(1);
    }

    // 创建tcp/ipv4套接字 tcp数据流 套接字
    if ((fd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
        perror("socket() error.\n");
        exit(1);
    }
    // 数据填充为0
    bzero(&server, sizeof(server));
    server.sin_family = AF_INET; // 设置族 ipv4
    server.sin_port = htons(PORT); // 端口
    server.sin_addr = *((struct in_addr *) he->h_addr); // 设置ip地址
    // 建立连接
    if (i = connect(fd, (struct sockaddr *)&server, sizeof(server)) == -1)
    {
        perror("connect() error\n.");
        exit(1);
    }
    // 输入主机名
    printf("连接成功，请输入主机名:");
    char* hostname = (char*) malloc(sizeof(char)*MAXDATASIZE);
    scanf("%s",hostname);
    write(fd, hostname, strlen(hostname)+1);
    free(hostname);//释放空间
    while(i != -1)
    {
        printf("请输入消息:");
        scanf("%s", &sendStr);
        sendStr[strlen(sendStr)]='\0';
        if(strcmp(sendStr, "quit") == 0)
        {
            write(fd, sendStr, strlen(sendStr)+1);
            exit(1);
        }
        write(fd, sendStr, strlen(sendStr)+1);  
        read(fd, recStr, MAXDATASIZE);
        printf("收到加密后的字符串:%s\n",recStr);
    }
    free(hostname);
    close(fd);
}
```

# 实验四:多线程

## `tcp_thread_server.c`

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <pthread.h>
#define PORT    11111//
#define BACKLOG 1
#define MAXDATASIZE 100
/**
 * 程序只考虑输入ascii码
*/
void process(int connfd,struct sockaddr_in client);
void* function(void* arg);
const int id[]={1,1,1,1,1,1,1,1,1,1,1};
struct ARG{
    int connfd;
    struct sockaddr_in client;
};
int main(void)
{
    pthread_t tid;
    int listenfd_d, connectfd_d; //首字母d结尾
    struct sockaddr_in server_d;
    struct sockaddr_in client_d;
    socklen_t addrlen;
    char buf[MAXDATASIZE];// 缓冲
    // 创建tcp/ipv4 面向tcp流的套接字
    if((listenfd_d = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
                perror("socket() error.\n");
                exit(1);
    }

    int opt = SO_REUSEADDR;//打开或关闭地址复用功能

    setsockopt(listenfd_d, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    bzero(&server_d, sizeof(server_d));//置0
    server_d.sin_family = AF_INET;// ipv4
    server_d.sin_port = htons(PORT); //设置端口,主机数转换为网络序
    server_d.sin_addr.s_addr = htonl(INADDR_ANY); // ip地址，此处是通配
    // 绑定到套接字体
    if (bind(listenfd_d, (struct sockaddr *)&server_d, sizeof(struct sockaddr)) == -1)
    {
                perror("bind() error.\n");
                exit(1);
    }
    // 监听套接字，监听队列长度1
    if (listen(listenfd_d, BACKLOG) == -1) 
    {
        perror("listen() error.\n");
        exit(1);
    }

    addrlen = sizeof(client_d);

    while(1) 
    {
        int n,m;
        // 接受连接
        if ((connectfd_d = accept(listenfd_d, (struct sockaddr *)&client_d, &addrlen)) == -1)
        {
            perror("accept() error.\n");
            exit(1);
        }
        printf("客户端ip: %s, 端口: %d\n", inet_ntoa(client_d.sin_addr), ntohs(client_d.sin_port));
        struct ARG* arg = (struct ARG*)malloc(sizeof(struct ARG));
        arg->connfd = connectfd_d;
        memcpy((void*)&(arg->client),&client_d,sizeof(client_d));
        if(pthread_create(&tid,NULL,function,(void*)arg)){
            perror("Pthread_create() error");
            exit(1);
        }
    }
    close(listenfd_d);
}
void* function(void* arg){
    struct ARG* info = (struct ARG*)arg;
    process(info->connfd,info->client);
    free(arg);
    pthread_exit(NULL);
}
void process(int connfd,struct sockaddr_in client){
    int n,m;
    char buf[MAXDATASIZE];
    //开始接收客户端的主机名字
    n = recv(connfd,buf,MAXDATASIZE,0);
    if(n>0){
        buf[n+1]='\0';
        printf("客户端的主机名是:%s\n",buf);
    }else{
        printf("??????");
    }
    while((n = recv(connfd, buf, MAXDATASIZE,0)) > 0)
    {
        // 计算分组是否不足
        int m = 10-(n-1)%10;
        m %= 10;
        char rever[n+m];
        strcpy(rever,buf);
        // 补0
        for(int j=0;j<m;j++){
            rever[n-1+j]='0';
        }
        for (int i = 0,j=0; i < m+n-1; i++,j++)
        {
            rever[i] += id[j%10];// 直接可学号相加
        }
        rever[m+n-1] = '\0';
        printf("收到信息:%s\n", buf);
        
        if (strcmp(rever, "quit") == 0)
        {
            close(connfd);
            exit(0);
        }
        printf("加密后的信息:%s\n",rever);
        send(connfd, rever, n+m,0);
        bzero(&buf, sizeof(buf));
    }
    close(connfd);
}
```

## `tcp_thread_client.c`

```c
/*client.c*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
#define PORT            11111
#define MAXDATASIZE     100

int main(int argc, char *argv[])
{
    argv[1] = "127.0.0.1";
    if(argc==1)argc++;
    int fd, numbytes, i;
    // 缓冲区
    char buf[MAXDATASIZE];
    char sendStr[MAXDATASIZE], recStr[MAXDATASIZE];
    struct hostent * he; //主机信息
    struct sockaddr_in server; // 套接字地址结构
    if (argc != 2)
    {
        printf("Usage: %s  <IP address>\n", argv[0]);
        exit(1);
    }
    // 查询主机信息
    if ((he = gethostbyname(argv[1])) == NULL) 
    {
        perror("gethostbyname() error.\n");
        exit(1);
    }

    // 创建tcp/ipv4套接字 tcp数据流 套接字
    if ((fd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
        perror("socket() error.\n");
        exit(1);
    }
    // 数据填充为0
    bzero(&server, sizeof(server));
    server.sin_family = AF_INET; // 设置族 ipv4
    server.sin_port = htons(PORT); // 端口
    server.sin_addr = *((struct in_addr *) he->h_addr); // 设置ip地址
    // 建立连接
    if (i = connect(fd, (struct sockaddr *)&server, sizeof(server)) == -1)
    {
        perror("connect() error\n.");
        exit(1);
    }
    // 输入主机名
    printf("连接成功，请输入主机名:");
    char* hostname = (char*) malloc(sizeof(char)*MAXDATASIZE);
    scanf("%s",hostname);
    write(fd, hostname, strlen(hostname)+1);
    free(hostname);//释放空间
    while(i != -1)
    {
        printf("请输入消息:");
        scanf("%s", &sendStr);
        sendStr[strlen(sendStr)]='\0';
        if(strcmp(sendStr, "quit") == 0)
        {
            send(fd, sendStr, strlen(sendStr)+1,0);
            exit(0);
        }
        send(fd, sendStr, strlen(sendStr)+1,0);  
        recv(fd, recStr, MAXDATASIZE,0);
        printf("收到加密后的字符串:%s\n",recStr);
    }
    free(hostname);
    close(fd);
}
```

