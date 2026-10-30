
### 配置

`server_addr` = `"192.168.1.101"` -- 这里也可以直接写服务器的域名，比如 `"214K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4Z5K9h3&6W2j5X3I4A6L8X3E0Q4x3X3g2U0L8$3@1".`
`server_port` = `8080`
`ap_ssid` = `"mywifi"` -- 路由器账号
`ap_passwd` = `"abc123"` -- 路由器密码

-- 配置USB以虚拟串口模式工作，这样调用 `print()` 函数就会在电脑串口终端打印输出
`LIB_UsbConfig("CDC")`

-- 使能系统 10 毫秒定时器开始工作
`LIB_10msTimerConfig("ENABLE")`

-- 设置esp8266 Wifi模块占用TX0、RX0、D5引脚，TCP Client模式
-- 路由器账号:mywifi 路由器密码:abc123，服务器ip:`192.168`.`1.101` 端口号:`8080`
-- 心跳包间隔时间 0 秒(不使用心跳机制)。如果需要使用请在ApiDoc文档中查阅关于 `LIB_WifiTcpConfig` 函数的p7参数详细介绍
`LIB_WifiTcpConfig("UART0","D5",ap_ssid,ap_passwd,server_addr,server_port,0)`

-- 变量初始化
`cnt_10ms` = `0`
`send_tab` = `{1,2,3,4,5}` -- 需要发送给server端的数据

-- 定义 10ms 中断回调函数
```
function LIB_10msTimerCallback()
  cnt_10ms = cnt_10ms + 1
end
```

-- 开始大循环
```
while (GC(1) == true) do
  -- 查询是否收到server下发的数据，如果收到就 print 输出收到的数据
  recv_flag,recv_tab = LIB_WifiTcpRecv()
  if recv_flag == 1 then
    print(string.format("tcp client receive %d bytes", #recv_tab))
    for k,v in ipairs(recv_tab) do
      print(k,v)
    end
  end
  -- 每 5 秒发送一包数据给server
  if cnt_10ms >= 500 then  -- 5000ms
    cnt_10ms = 0
    LIB_WifiTcpSend(send_tab)
  end
end
```

<small>如果感兴趣，上面代码中出现的LIB开头的库函数可以在<a href="http://shineblink.com/#/?page=API%E6%96%87%E6%A1%A3" target="_blank">API文档</a>中通过<kbd>Ctrl</kbd>+<kbd>F</kbd>查询。</small>

### 代码运行结果

这里我们在局域网内的一台电脑(192.168.1.101)上运行《<a href="https://yunpan.360.cn/surl_yPvUkcgNuMg" target="_blank">网络调试助手</a>》软件模拟Tcp Server端进行调试。

**（1）Server端数据接收如下：**

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202012/910638_MMNCJ6UUY8XQDJA.jpg)

**（2）Client端(Core开发板)数据接收如下：**
![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202012/910638_KMQK6MH4EYEBXAE.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
