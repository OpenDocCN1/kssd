
# QQMsgSpeaker 附源码。语音读出 QQ 好友发来的消息/ update 2010.10.01

> 原文：[`www.pediy.com/kssd/pediy11/120936.html`](https://www.pediy.com/kssd/pediy11/120936.html)

功能如题所说的。

`TTS` 取巧用的 `Google Translate` 的。

感谢 `QQSpy`

感谢踏雪流云

感谢 `Translate.google.com`

.....

代码:

```python
    def Speaker():
        while True:
            if MsgCookies:
                TinyTTS.SPEECH(MsgCookies.pop())
            else:
                time.sleep(1)

    @HOOKDecorator('KernelUtil.?SaveMsg@Msg@Util@@YAHPB\_WKKKPAUITXMsgPack@@PAUITXData@@@Z', ctypes.CFUNCTYPE)
    def SaveMsg(str, toQQNum, fromQQNum, data3, pmsg, pdata):
        xmsg = ctypes.c\_long()
        AbstractMsg(ctypes.pointer(xmsg), pmsg)
        msg = ctypes.wstring\_at(xmsg.value)
        MsgCookies.insert(0, '%d 说: %s' % (fromQQNum, msg.encode('gb2312')))
        global IsSpeakerRunning
        if not IsSpeakerRunning:
            IsSpeakerRunning = True
            thread.start\_new(Speaker, ())
        return OriginFuncion(str, toQQNum, fromQQNum, data3, pmsg, pdata)
```

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `QQMsgSpeaker.zip`  
|
| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `DEMO.zip`  
|
