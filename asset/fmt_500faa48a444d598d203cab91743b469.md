
# 迅雷 DownAndPlay.dll 模块溢出

> 原文：[`www.pediy.com/kssd/pediy10/64330.html`](https://www.pediy.com/kssd/pediy10/64330.html)

/*  
抱歉，没有仔细注意这个问题。这个洞的确没办法远程溢出。![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  
*/

迅雷 `DownAndPlay.dll` 随迅雷起来后(不知道这个模块在从那个版本开始出现，我这是最新版本)，在本地绑定 `36897` 端口，等待链接，

接受的数据格式是 `XLDAP|key|value|XLDAP`，其中的 `key` 由预定义的几个，

这里实现溢出选择的是 `savepath`，当 `value` 超长时就会在 `spintf` 处发生溢出。

附件的 `11.dat` 是在 临时构造的 一份数据，可以使用 `nc` 看看效果，运行命令，

迅雷会因为异常退出。

有兴趣试着写出完整攻击代码。![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")  

`C:\>nc 127.0.0.1 36897 <11.dat`

```
23132CB6    8D45 A0         lea     eax, dword ptr [ebp-60]
23132CB9    E9 84010000     jmp     23132E42
23132CBE    68 B4C61323     push    2313C6B4                         ; ASCII "savepath"
23132CC3    57              push    edi
23132CC4    FFD6            call    esi
23132CC6    59              pop     ecx
23132CC7    84C0            test    al, al
23132CC9    59              pop     ecx
23132CCA    74 5F           je      short 23132D2B
23132CCC    8B75 0C         mov     esi, dword ptr [ebp+C]
23132CCF    8B0D E0D21323   mov     ecx, dword ptr [2313D2E0]
23132CD5    56              push    esi
23132CD6    E8 EB E5FFFF    call    231312C6
23132CDB    8B46 04         mov     eax, dword ptr [esi+4]
23132CDE    8B0D B0E41323   mov     ecx, dword ptr [<&MSVCP60.`std::>; MSVCP60.`std::basic_string<char,std::char_traits<char>,std::allocator<char> >::\_Nullstr'::`2'::\_C
23132CE4    85C0            test    eax, eax
23132CE6    8BD1            mov     edx, ecx
23132CE8    74 02           je      short 23132CEC
23132CEA    8BD0            mov     edx, eax
23132CEC    8B7F 04         mov     edi, dword ptr [edi+4]
23132CEF    85FF            test    edi, edi
23132CF1    74 02           je      short 23132CF5
23132CF3    8BCF            mov     ecx, edi
23132CF5    B8 D4C61323     mov     eax, 2313C6D4                    ; ASCII "XLDAP"
23132CFA    50              push    eax
23132CFB    52              push    edx
23132CFC    51              push    ecx
23132CFD    50              push    eax
23132CFE    8D85 5CFEFFFF   lea     eax, dword ptr [ebp-1A4]
23132D04    68 C0C61323     push    2313C6C0                         ; ASCII "%s|%s|%s|%s"
23132D09    50              push    eax
```

溢出发生位置

```
23132D0A    FF15 54E51323   call    dword ptr [<&MSVCRT.sprintf>]    ; MSVCRT.sprintf
23132D10    8D85 5CFEFFFF   lea     eax, dword ptr [ebp-1A4]
23132D16    50              push    eax
```

![](img/c53a5b9ecb686070c04551d4f121a78a.jpg)

睡觉了。。

## 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 11.rar |
