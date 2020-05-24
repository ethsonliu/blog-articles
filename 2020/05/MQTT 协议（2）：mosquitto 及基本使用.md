Mosquitto 是 Eclipse 旗下的一个 MQTT 开源库，支持 V3 和 V5，同时提供了一个 C 语言链接库 libmosquitto 用于客户端编程，开源地址是 [Github Mosquitto](https://github.com/eclipse/mosquitto)。

## 编译及简单使用

关于 Linux、Arm 和 Windows 上的编译请参考[编译 libmosquitto](https://github.com/EthsonLiu/personal-notes/blob/master/mqtt/编译 libmosquitto.md)，完成后会找到以下几个可执行程序，

- mosquitto，MQTT Broker，也就是 MQTT  Server（以下简称 Broker）
- mosquitto_passwd，管理上述 mosquitto 密码文件的命令行工具
- mosquitto_sub，订阅者客户端
- mosquitto_pub，发布者客户端
- mosquitto_rr，用于请求/响应消息传递的客户端

现在测试一下客户端和服务端程序。为了测试方便，将客户端和服务端程序都部署在本机，使用 localhost 连接。执行 `mosquitto -v`  启动 Broker ，`-v` 表示打印出运行信息，可以看到默认使用的端口是 1883，

```shell
ethson@ethson-virtual-machine:/etc/mosquitto$ mosquitto -v
1590294257: mosquitto version 1.6.9 starting
1590294257: Using default config.
1590294257: Opening ipv4 listen socket on port 1883.
1590294257: Opening ipv6 listen socket on port 1883.
```

如果系统出现如下问题，就需要添加一个 mosquitto 用户，

```shell
ethson@ethson-virtual-machine:/etc/mosquitto$ mosquitto -v
1590294227: Error: Invalid user 'mosquitto'.
ethson@ethson-virtual-machine:/etc/mosquitto$ useradd mosquitto
ethson@ethson-virtual-machine:
```

然后在第二个终端启动订阅者程序：`mosquitto_sub -h localhost -t test -v`，用 `-h` 参数指定服务器 IP，用 `-t` 参数指定订阅的话题。

在第三个终端启动发布者程序：`mosquitto_pub -h localhost -t test -m "Hello world"`，用 `-m` 参数指定要发布的信息内容，然后在订阅者的终端就可以看到由 Broker 推送的信息 "Hello world"，同时在 Broker 的终端也可以看到处理信息的过程。

其它更多的介绍请参照[文档](https://mosquitto.org/)，Mosquitto 提供了丰富的文档说明，请善用文档！

## 客户端编程

**客户端（订阅消息）**

```c++
#include <stdio.h>
#include <stdlib.h>
#include <mosquitto.h>
#include <string.h>

#define HOST "localhost"
#define PORT  1883
#define KEEP_ALIVE 60

void my_message_callback(struct mosquitto *mosq,
                         void *userdata,
                         const struct mosquitto_message *message)
{
    if (message->payloadlen)
    {
        printf("topic=<%s>, payload=<%s>\n",
               message->topic,
               message->payload);
    }
    else
    {
        printf("topic=<%s>, payload is empty\n",
               message->topic);
    }
}

void my_connect_callback(struct mosquitto *mosq,
                         void *userdata,
                         int result)
{
    if (!result)
    {
        // 成功连接后订阅主题
        mosquitto_subscribe(mosq, NULL, "example/topic", 2);
    }
    else
    {
        printf("Connect failed\n");
    }
}

int main()
{
    // libmosquitto 库初始化
    mosquitto_lib_init();
    
    // 创建 mosquitto 客户端
    struct mosquitto *mosq = mosquitto_new(NULL, true, NULL);
    if (!mosq)
    {
        printf("create client failed.\n");
        mosquitto_lib_cleanup();
        return 1;
    }
    
    // 设置回调函数，需要时可使用
    mosquitto_connect_callback_set(mosq, my_connect_callback);
    mosquitto_message_callback_set(mosq, my_message_callback);
    
    // 连接服务器
    int ret = mosquitto_connect(mosq, HOST, PORT, KEEP_ALIVE);
    if (ret != MOSQ_ERR_SUCCESS)
    {
        printf("Unable to connect, %s\n", mosquitto_strerror(ret));
        return 1;
    }
    
    // 循环处理网络消息，会阻塞在此处
    mosquitto_loop_forever(mosq, -1, 1);

    mosquitto_destroy(mosq);
    mosquitto_lib_cleanup();
    return 0;
}
```

**客户端（发布消息）**

```c++
#include <stdio.h>
#include <stdlib.h>
#include <mosquitto.h>
#include <string.h>

#define HOST "localhost"
#define PORT  1883
#define KEEP_ALIVE 60
#define MSG_MAX_SIZE  512

int main()
{
    char buff[MSG_MAX_SIZE];
    memset(buff, 0, sizeof(buff));
    
    // libmosquitto 库初始化
    mosquitto_lib_init();
    
    // 创建 mosquitto 客户端
    struct mosquitto * mosq = mosquitto_new(NULL, true, NULL);
    if (!mosq)
    {
        printf("create client failed.\n");
        mosquitto_lib_cleanup();
        return 1;
    }
    
    // 连接服务器
    int ret = mosquitto_connect(mosq, HOST, PORT, KEEP_ALIVE);
    if (ret != MOSQ_ERR_SUCCESS)
    {
        printf("Unable to connect, %s\n", mosquitto_strerror(ret));
        return 1;
    }
    
    // 开启一个线程，在线程里不停的调用 mosquitto_loop() 来处理网络信息
    ret = mosquitto_loop_start(mosq);
    if (ret != MOSQ_ERR_SUCCESS)
    {
        printf("mosquitto loop error, %s\n", mosquitto_strerror(ret));
        return 1;
    }
    
    while (fgets(buff, MSG_MAX_SIZE, stdin) != NULL)
    {
         // 发布消息
         mosquitto_publish(mosq, NULL, "example/topic", strlen(buff), buff, 0, 0);
         memset(buff, 0, sizeof(buff));
    }
    
    mosquitto_destroy(mosq);
    mosquitto_lib_cleanup();
    return 0;
}
```

更多 API 的介绍请参考 [Mosquitto APIs](https://mosquitto.org/api/files/mosquitto-h.html)，还是那句话，善用文档！

注意：Mosquitto 有两种编程模式，单线程和多线程模式。这是我在编程时意外遇到的一个 bug 而发现的，官方的文档并没有说明这一点，可以参见 [How to disconnect and close the struct mosquitto object safely?](https://github.com/eclipse/mosquitto/issues/1282)。

## 参考

- [基于 MQTT 协议的 Mosquitto 的使用及 libmosquitto 客户端编程](https://blog.csdn.net/dancer__sky/article/details/77855249)
- [MQTT 协议和 mosquitto](https://shaocheng.li/posts/2015/08/11/)