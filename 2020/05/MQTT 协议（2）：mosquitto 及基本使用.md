Mosquitto 是 Eclipse 旗下的一个 MQTT 开源库，支持 V3 和 V5，同时提供了一个 C 语言链接库 libmosquitto，用于客户端编程。以下是相关链接：

- [Eclipse Mosquitto](https://mosquitto.org/)
- [Github Mosquitto](https://github.com/eclipse/mosquitto)

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

注意：Mosquitto 有两种编程模式，单线程和多线程模式，具体参考 [How to disconnect and close the struct mosquitto object safely?](https://github.com/eclipse/mosquitto/issues/1282)。

更多 API 的介绍请参考 [Mosquitto APIs](https://mosquitto.org/api/files/mosquitto-h.html)。

## 其它

关于 Linux、Arm 和 Windows 上的编译请参考 [编译 libmosquitto](https://github.com/EthsonLiu/personal-notes/blob/master/mqtt/编译 libmosquitto.md)，关于 Mosquitto 的更多的内容请参考 [MQTT 协议和 mosquitto](https://shaocheng.li/posts/2015/08/11/)。

## 参考

- [基于 MQTT 协议的 Mosquitto 的使用及 libmosquitto 客户端编程](https://blog.csdn.net/dancer__sky/article/details/77855249)