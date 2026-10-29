
- [service.py](<https://bbs.kanxue.com/elink@338K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4M7Y4m8U0i4K6u0r3k6%4u0H3j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5r3q4E0M7r3I4W2M7#2)9J5c8Y4m8&6N6r3S2G2L8W2)9J5c8X3S2W2L8r3I4G2N6$3!0J5L8r3c8Q4x3V1k6Y4M7X3g2W2N6r3g2J5i4K6g2X3M7$3g2J5N6X3g2J5i4K6u0W2M7s2V1`.)>)

### 常用的protobuf数据类型&特殊字符&pb文件&

> [官方语法介绍](<https://bbs.kanxue.com/elink@787K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3M7X3!0@1L8$3u0#2k6W2)9J5k6h3c8W2N6W2)9J5c8Y4m8J5L8$3N6J5j5h3#2E0K9h3&6Y4i4K6u0V1k6%4g2A6k6r3g2K6i4K6u0r3M7s2u0G2N6r3)9K6i4K6u0r3)>)

#### 常用的protobuf数据类型

1. `string` -> 字符串类型，要求 utf-8 或 7-bit 与 ascii 编码的字符串

2. `bytes` -> 比特类型

3. `bool` -> 布尔类型

4. `int32` -> 32位整形，对应 Py 是 `int`

5. `int64` -> 64位整形，对应 Py 是 `int`

6. `float` -> 浮点类型

7. `repeated` -> 数组(`list`)，如 `repeated string data = 1;` 表示 一个 `string` 的 `list`

8. `map` -> 字典类型(`Python`)，如 `map <string , string> data = 1;`；前者为 `key` 的 `string` 类型，后者为 `string` 类型的 `value`

#### 常用的protobuf特殊字符

1. `package` -> 包名

2. `syntax` -> `protobuf` 语法版本

3. `service` -> 定义一个服务

4. `rpc` -> 定义服务中的方法

5. `stream` -> 定义方法的传输为流传输

6. `message` -> 定义消息体 `message User{}`

7. `extend` -> 扩展消息体 `extend User{}`

8. `import` -> 导入一些插件

9. `//` -> 注释~

### 在Nexus6P KaliNetHunter 上 配置环境并启动客户端与PC上`server`进行通信 (可有可无....)

> 目标：
>
> 1. 在6P手机上配置好 `Python` 环境、`grpc` 环境
> 2. 运行 `client.py` 与 PC 上 `server.py` 进行交互
> 3. ~~(未复现出)抓包拦截请求数据,尝试分析.... ~~ 当初以为只要运行在手机上就算是 android 了，，，后来才反应过来，，，，我是运行在 linux 上 `py` 程序，不是 android
>
>    - `client.py` 和 android app 走的系统底库不同（后续实现了 App 抓 `socket` 却可以抓到而 `client.py` 却不能抓到）

#### Nexus6P手机上配置grpc环境~

1. Python3.10 环境下 直接安装 三件套：

```bash
pip3 install protobuf grpcio grpcio-tools -i 1bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3P5i4m8A6i4K6u0W2k6r3!0#2j5X3q4F1i4K6u0W2j5$3!0E0i4K6u0r3M7$3W2E0M7r3I4W2
```

   - ![](https://bbs.kanxue.com/upload/attach/202308/900788_XXKBABBXV2SC6N4.png)

   - ![](https://bbs.kanxue.com/upload/attach/202308/900788_8Y8Q869G3JZYU3N.png)

   - ![](https://bbs.kanxue.com/upload/attach/202308/900788_EQMMCU7V4QYHE2K.png)

2. 下载 linux 对应的 `protoc` 工具 push 到手机内。先不下载试试 win 生成的两份 `_pb2` 文件能否共用？是的，可直接可用。

3. 将 `client.py` 修改访问地址为局域网内 PC 的 ip，并 push 到 `/sdcard/protobuf` 目录下。

#### Nexus6P 与PC 通信

将服务端的监听端口由 `127.0.0.1` 修改为 `0.0.0.0`，之后先启动 `server`，再在 6P 手机上启动 `client.py` 实现交互。

- ![](https://bbs.kanxue.com/upload/attach/202308/900788_BMPSVREJ7BGC673.png)

- ![](https://bbs.kanxue.com/upload/attach/202308/900788_XZ7ZRYSSVD3NF9R.png)

- ![](https://bbs.kanxue.com/upload/attach/202308/900788_X6JJ8V7GZVFR6HY.png)

可以发现同一局域网时可以进行通信的。

#### 此处采用中间人抓包会报错

客户端启动失败！charles 抓的包为 Unknow，整体运行逻辑如下：

- ![](https://bbs.kanxue.com/upload/attach/202308/900788_SDZ9V8VUJDYA5VS.jpg)

- 不知为何原因~

### Android grpc-protobuf Dem0

> [demo下载地址](<https://bbs.kanxue.com/elink@9edK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5N6h3g2^5K9h3q4F1k6$3A6&6M7#2)9J5c8W2m8J5L8%4c8G2j5Y4g2X3i4K6u0V1k6#2u0b7b7#2)9J5k6p5q4F1k6s2u0G2K9h3b7`.)>)
>
> 目标：
>
> 1. 实现 Android App 与 PC 上建立的 Python 服务端进行通信
> 2. 弄清楚普通 `grpc` 请求的执行流程，以及生成对应的 Python 服务器版本代码
> 3. 最终实现效果可以使得二者可以正常通信，为抓包 `hook` 以及分析打下基础

#### 搞清楚 android-java `.proto` 的运行原理

1. [Java客户端 + Python后端 实现 通信](<https://bbs.kanxue.com/elink@1a6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2y4e0q4U0N6r3!0Q4x3X3g2U0L8$3#2Q4x3V1k6#2i4K6g2X3x3e0f1^5z5e0f1K6x3U0W2Q4x3V1j5#2z5o6V1@1x3U0V1&6)>)
2. [官方Java教程](<https://bbs.kanxue.com/elink@d32K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3M7X3!0@1L8$3u0#2k6W2)9J5k6h3c8W2N6W2)9J5c8X3N6W2N6s2c8A6L8X3N6Q4x3X3c8K6N6r3q4J5N6r3g2V1i4K6u0r3K9X3q4$3j5i4c8#2N6r3!0J5K9h3q4D9i4K6u0r3)>)
3. [官方Python教程](<https://bbs.kanxue.com/elink@853K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3M7X3!0@1L8$3u0#2k6W2)9J5k6h3c8W2N6W2)9J5c8X3N6W2N6s2c8A6L8X3N6Q4x3X3c8K6N6r3q4J5N6r3g2V1i4K6u0r3M7s2W2@1K9r3!0F1N6s2g2@1L8%4u0A6j5h3I4Q4x3V1j5`.)>)

#### 利用Demo 实现与Python 通信

> 只分析其中一个界面
>
> ![](https://bbs.kanxue.com/upload/attach/202308/900788_UVZGM72U4JQ7FAM.png)

##### 找到demo中`helloworle.proto`文件 pull到PC上 并生成Python `_pb2_grpc.py` 代码

- 进入对应解释器环境下，执行：

```bash
python -m grpc_tools.protoc -I./ --python_out=./ --grpc_python_out=./ helloworld.proto
```

- ![](https://bbs.kanxue.com/upload/attach/202308/900788_Q75Y4SS86B6XYJ9.png)

- 照猫画虎写出对应的 `service.py`：

```python
from datetime import datetime
import grpc
import helloworld_pb2 as pb2
import helloworld_pb2_grpc as pb2_grpc
from concurrent import futures

class Greeter(pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        name = request.name
        sendtime = datetime.now().strftime('%Y-%m-%d-%H:%M:%S')
        result = f"My Name is {name}!!! send time is {sendtime} "
        print(f"服务器接收到客户端参数 -> name = {name}")
        print(f"服务器响应时间: {sendtime}")
        return pb2.HelloReply(message = result)

def run():
    grpc_server = grpc.server(
        futures.ThreadPoolExecutor(max_workers=4),
    )
    pb2_grpc.add_GreeterServicer_to_server(Greeter(),grpc_server)
    # ip_port = "127.0.0.1:50051"
    ip_port = "0.0.0.0:50051"
    grpc_server.add_insecure_port(ip_port)
    grpc_server.start()
    print(f"service start at {ip_port}")
    grpc_server.wait_for_termination()

if __name__ == "__main__":
    run()
```

- 验证启动 apk 能否正常通信

- ![](https://bbs.kanxue.com/upload/attach/202308/900788_BH5D2XN49HJBCHA.png)

- ![](https://bbs.kanxue.com/upload/attach/202308/900788_UAMJXADGQD8J35E.png)

所有所有前期准备已经 ok，准备使用 hook 抓包看能否抓到包~

### objection动态分析 & Hook socket抓包

1. 使用 `objection` 动态分析，搜索类：`android hooking search classes protobuf`

   - 复制系统库保存为 txt 文件，执行脚本加上 hook：

```python
# 读取文件并处理
with open('my_protobuf.txt', 'r',encoding='utf-8') as file:
    lines = file.readlines()
    processed_lines = []
    for line in lines:
        # 进行处理操作
        processed_line = "android hooking watch class " + line
        # 将处理后的行插入到原来的列表中
        processed_lines.append(processed_line)

    # 将处理后的内容按行插入到原文件中
    with open('my_protobuf_new.txt', 'w',encoding='utf-8') as file:
        for processed_line in processed_lines:
            file.write(processed_line)
```

   - ![](https://bbs.kanxue.com/upload/attach/202308/900788_HDRJ6JVWKDSHYMR.png)

2. 所有的输出信息如下
