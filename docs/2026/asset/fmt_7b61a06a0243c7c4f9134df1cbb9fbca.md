
# [原创]hookQQ-API拦截QQ聊天记录-有图有码

大家先对QQ采用ollydbg调试QQ，

![](https://bbs.kanxue.com/upload/attach/201206/252728_37q75f7xag95ve8.png)

分析出相关QQ内部函数

```
//?GetMsgTime@Msg@Util@@YA\_JPAUITXMsgPack@@@Z
//?GetSelfUin@Contact@Util@@YAKXZ
//?GetGroupName@Group@Util@@YA?AVCTXStringW@@K@Z
//?GetDiscussName@Group@Util@@YA?AVCTXStringW@@K@Z
//?GetGroupMemLongNickname@Group@Util@@YAHKKAAVCTXStringW@@@Z
//?GetGroupMemShowName@Group@Util@@YA?AVCTXStringW@@KK@Z
//?GetSelfUin@Contact@Util@@YAKXZ
```

然后我们写一个ＤＬＬ来注射到ＱＱ内部，调用ＱＱ相关函数，获取相关QQ聊天记录信息，然后将QQ聊天记录用`sendmessage`发送出来。

DLL代码如下
