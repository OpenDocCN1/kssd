
## 再准备的一个 Xposed 的项目

- 不会的可以先看我之前写的一篇笔记：[AndroidStudio使用Xposed](https://bbs.kanxue.com/elink@6adK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4@1M7X3W2$3k6h3#2S2M7X3W2G2i4K6u0W2N6$3!0J5K9#2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3K6b7&6y4o6q4Q4x3X3g2Z5N6r3#2D9)

- 在 app 的 `build.gradle` 添加依赖 `implementation 'com.virjar:sekiro-api:1.0.1'`

![52.png](https://bbs.kanxue.com/upload/attach/202006/773600_KY7RZAZCGTBHFYG.jpg)

- 在 Xposed 的 `handleLoadPackage` 中启动 client 链接 server，并添加处理事件的 handler, 用于调用 Add 函数

```
Log.i(TAG, "connect server....");

//服务端host

String testHost = "your_host";

//客户端标识

String clientId = UUID.randomUUID().toString();

//接口组名称

String groupName = "addDemoTest2";

//暴露的接口名称

String actionName = "myAdd";

//拿classloader

clzLoader = lpparam.classLoader;

//连接服务端并且注册处理的handler

SekiroClient.start(testHost, clientId, groupName)
        .registerHandler(actionName, new SekiroRequestHandler(){

            @Override
            public void handleRequest(SekiroRequest sekiroRequest, SekiroResponse sekiroResponse){

                //当服务端分配任务时, 这里处理逻辑, 并把结果返回给服务端, 服务端再返回给调用者
                Class<?> clz = XposedHelpers.findClass("com.example.administrator.adddemo.MainActivity", clzLoader);
                int arg1  = sekiroRequest.getInt("arg1");
                int arg2  = sekiroRequest.getInt("arg2");
                Log.i(TAG, String.format("arg1 : %d, arg2 : %d", arg1, arg2));
                Object result = XposedHelpers.callStaticMethod(clz, "Add", arg1, arg2);
                Log.i(TAG, "result : " + result);
                sekiroResponse.success(result);
            }

        });
```

- 编译错误 `More than one file was found with OS independent path` 解决

```
//build.gradle里android{}添加错误的path

packagingOptions {
    exclude 'META-INF/DEPENDENCIES'
    exclude 'META-INF/INDEX.LIST'
    exclude ('META-INF/io.netty.versions.properties')
}
```

## 0x4 尝试调用

- 打开 app 后, 使用浏览器查看 group 列表是否注册成功:

  `[your_server_ip]:[server.port]/groupList`

![54.png](https://bbs.kanxue.com/upload/attach/202006/773600_Q3CBD4WACT7TSQ7.jpg)

- 调用接口查看结果:

```
格式: your_server_ip:[natHttpServerPort]/[invoke_type]?group=[group_id]&action=[action_name]&param1=[arg]

例子: cb8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5i4K6u0W2P5q4)9J5k6i4S2Q4x3X3g2^5i4K6y4m8y4e0j5H3x3W2)9J5c8X3q4K6P5h3&6U0d9h3&6$3L8$3E0W2i4K6y4r3k6%4u0G2N6i4m8Q4x3@1c8S2k6r3c8p5k6h3#2G2g2r3g2K6N6o6u0Q4x3U0k6S2L8i4m8Q4x3@1u0S2j5%4c8A6L8$3&6Q4x3@1c8E0P5f1q4V1k6q4)9J5y4X3q4E0M7q4)9K6b7X3q4J5k6K6q4Q4x3@1b7K6x3o6m8Q4x3U0k6S2L8i4m8Q4x3@1u0S2M7X3M7J5i4K6y4p5x3K6l9H3
```

![55.png](https://bbs.kanxue.com/upload/attach/202006/773600_57V8CVVF4E3N867.jpg)

- client 的调用日志

![56.png](https://bbs.kanxue.com/upload/attach/202006/773600_HART2TPCTDDXJZJ.jpg)

## 感谢&参考资料

- [sekiro实践](https://bbs.kanxue.com/elink@e2eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0k6T1y4K6p5I4x3o6k6U0y4o6g2W2j5W2)9K6c8X3k6J5L8$3#2Q4x3@1c8@1K9h3#2W2L8r3W2F1k6b7`.`.)

- [[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
