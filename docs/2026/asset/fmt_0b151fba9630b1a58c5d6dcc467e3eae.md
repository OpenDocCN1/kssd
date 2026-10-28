
经过排查，可以在调用堆栈的第二层找到一个疑似发消息的`call`。在这个地方下断点，让程序断下，分析附近代码。

## 分析发送消息的函数

### 普通消息

![](https://bbs.kanxue.com/upload/attach/201906/833057_UVAPXDF2JYJ85UQ.png)

此时`edx`指向微信ID，`[edx+4]`保存的是微信ID的长度。

![](https://bbs.kanxue.com/upload/attach/201906/833057_EAYJENKBWV7KK4V.png)

`ebx`指向消息内容，`[ebx+4]`保存的是消息内容的长度。那么这个很有可能就是我们要找的发送消息的`call`。

找到了发送消息的函数，那么怎么验证呢？利用微信ID。将`edx`指向的微信ID的地址和我们之前在CE中找到的当前窗口的微信ID对比，你会发现两个地址是一样的。

![](https://bbs.kanxue.com/upload/attach/201906/833057_JV57GEA9989KK8E.png)

改变这个地址的微信ID和内容，就能直接改变消息的接收者和内容，这个刚才我们已经实验过了。再结合这个函数传入的参数有当前消息的内容，就可以确定这个`call`就是微信发送消息的函数。

### 艾特某人消息

除了以普通文本的方式发送消息以外，还可以以艾特某人的方式发送消息。那么当发送的消息是艾特某人的时候，这个函数和发送普通文本消息有什么区别呢？区别就在于`eax`寄存器的值。

先发送一条普通消息程序断下：

![](https://bbs.kanxue.com/upload/attach/201906/833057_X49XMKY8Z5RRYWU.png)

此时`eax`的值为0，然后再发送一条艾特某人的消息：

![](https://bbs.kanxue.com/upload/attach/201906/833057_Z9KM4Y75VBXRKDK.png)

此时`eax`是有值的，数据窗口跟随，看看这个`14704C40`的地址保存的是什么内容：

![](https://bbs.kanxue.com/upload/attach/201906/833057_UUH9PRRV76UD2K6.png)

里面的被艾特的人的微信ID，**普通消息与艾特消息的区别就在于`eax`是否保存了被艾特人的微信ID**。大家可以用同样的方式分析处图片和表情在内存中的表现形式。

接下来我们只要记录下当前发送消息函数的地址+偏移，就能写一个dll注入到微信进程空间中，直接调用发送消息的函数，就能实现用自己写的程序给任何人发送消息。

## 总结

总结一下思路，寻找切入点->找地址->下断->栈回溯分析。跟接收消息的步骤是一致的。**找`call`的关键在于你能不能找到一个好的切入点，并且利用切入点与`call`之间的关系。**

## 代码实现

调用发送消息的函数代码如下：

```
void SendTextMessage(wchar_t* wxid, wchar_t* msg)

{

    //拿到发送消息的call的地址

    DWORD dwSendCallAddr = GetWeChatWinAddr() + 0x2EB4E0;



    //微信ID/群ID

    wxMsg id = {0};

    id.pMsg = wxid;

    id.msgLen = wcslen(wxid);

    id.buffLen = wcslen(wxid)*2;



    //消息内容

    wxMsg text = { 0 };

    text.pMsg = msg;

    text.msgLen = wcslen(msg);

    text.buffLen = wcslen(msg)*2;





    //取出微信ID和消息的地址

    char* pWxid = (char*)&id.pMsg;

    char* pWxmsg = (char*)&text.pMsg;





    char buff[0x81C] = { 0 };

    //调用微信发送消息call

    __asm {

        mov edx, pWxid;

        push 1;

        mov eax, 0;

        push eax;

        mov ebx, pWxmsg;

        push ebx;

        lea ecx, buff;

        call dwSendCallAddr;

        add esp, 0xC;

    }

}
```

## 最终效果

发送消息：

![](https://bbs.kanxue.com/upload/attach/201906/833057_T983Y7QPK6W6XH6.jpg)

接收消息：

![](https://bbs.kanxue.com/upload/attach/201906/833057_PKEJ2327DRQYY5C.png)

成品就暂时不发了，下回补上，有个bug调了好久没调出来，拜拜~

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-6-22 17:44 被鬼手56编辑，原因：[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
