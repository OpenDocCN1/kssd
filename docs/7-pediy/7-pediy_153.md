

# VA\_X inline Patching

> 原文：[`www.pediy.com/kssd/pediy10/88341.html`](https://www.pediy.com/kssd/pediy10/88341.html)

【文章标题】: VA\_X inline Pathcing  
【文章作者】: yangjt  
【作者邮箱】: yangjietao123@163.com  
【作者主页】: http://blog.sina.com.cn/yangjt  
【作者 QQ 号】: 325002492  
【软件名称】: VA\_X  
【加壳方式】: Armadillo  
【操作平台】: Win Vista+ Win XP  
【前   传】: BRD 51 的时候很过分……  
【修改说明】: 自己做起来才发现不是那么回事……本着认真负责的态度……我来重写这篇文章^\_^  
\--------------------------------------------------------------------------------  
【详细过程】  
原理就是 Hook ArmAccess.dll 输入表里的 VirtualProtect，因为解码后程序会 N 次调用到这个函数……所以我们把它给改到我们的代码上，然后就可以为所欲为了……(^\_^)当然中间还需要处理 Arm 的 CRC 校验。所以如何处理……请听我慢慢说来  
First Step:  
prepare  
  patch 代码前要做一些准备工作  
  用 PeEditor 把程序的.text 段和.adata 段的属性加上可写属性，因为代码需要对这两段进行 SMC，如果不能写就会出现页保护错误  
  之后 OllyICE 打开程序，下那个经典的断点 VirtualProtect，锁定程序解码 ArmAccess.dll 时对其区段进行的页保护设定  
  这段代码长成这个样子：  
  

引用:   1F25A53C   .  C745 B8 04000>mov     dword ptr \[ebp\-48\], 4  
  1F25A543   .  EB 07         jmp     short 1F25A54C  
  1F25A545      C745 B8 02000>mov     dword ptr \[ebp\-48\], 2  
  1F25A54C   >  8B55 E0       mov     edx, dword ptr \[ebp\-20\]  
  1F25A54F   .  8B45 F4       mov     eax, dword ptr \[ebp\-C\]  
  1F25A552   .  0342 0C       add     eax, dword ptr \[edx+C\]  
  1F25A555   .  8945 A8       mov     dword ptr \[ebp\-58\], eax  
  1F25A558   .  8B4D E0       mov     ecx, dword ptr \[ebp\-20\]  
  1F25A55B   .  8B51 08       mov     edx, dword ptr \[ecx+8\]  
  1F25A55E   .  8955 B0       mov     dword ptr \[ebp\-50\], edx  
  1F25A561   .  8D45 BC       lea     eax, dword ptr \[ebp\-44\]  
  1F25A564   .  50            push    eax                              ; /pOldProtect  
  1F25A565   .  8B4D B8       mov     ecx, dword ptr \[ebp\-48\]          ; |  
  1F25A568   .  51            push    ecx                              ; |NewProtect  
  1F25A569   .  8B55 B0       mov     edx, dword ptr \[ebp\-50\]          ; |  
  1F25A56C   .  52            push    edx                              ; |Size  
  1F25A56D   .  8B45 A8       mov     eax, dword ptr \[ebp\-58\]          ; |  
  1F25A570   .  50            push    eax                              ; |Address  
  1F25A571   .  FF15 7030291F call    dword ptr \[<&KERNEL32.VirtualPro>; \\VirtualProtect  
   

  查一下 MSDN 就不难发现 0x02 是 PAGE\_READONLY，顺便也看下 PAGE\_EXECUTE\_READWRITE，嗯……这个值是 0x40  
  好下面让我们继续分析  
  1F25A545      C745 B8 02000>movdwordptr \[ebp\-48\], 2  
  这句代码很诡异……因为它的作用……很显然嘛，把页保护属性装载到某个神秘的地方，这句不 Patch 到最后 Hook ArmAccess.dll 里的 VirtualProtect 就要出错。  
  把 0x02Patch 成 0x40 就 OK 啦~~嗯……第一部分工作完成……保存此处修改……休息下眼睛^\_^  
  
  
  
  第二部分开工咯~~  
  patch 代码要写到.adata 段  
  original:  
  1F259A2C  |.  A1 04A1291F   moveax, dwordptr \[1F29A104\]  
  
  patch:  

引用:   call @patch  
      
      
  @patch:  
  mov     eax, dword ptr \[1F29A104\];恢复原始句  
  pushad  
  mov ds:Var1,eax  
  mov \[eax+572ec\],//push @@1&ret 的机器码  
  mov \[eax+572ec\]，//push 一不够两次  
  popad  
  ret  
  ////////////////////////////////////////////////  
  @@1:  
  pushad  
  mov eax,ds:Var1  
  mov \[eax+572ec\],8B10C483//恢复 var1+572ec 处的机器码  
  mov \[eax+572F0\], 5DE5  
  mov dword ptr \[eax+441FD\], 28306D68//push @@2&ret 的机器码  
  mov word ptr \[eax+44201\], 0C31F//push 一不够两次  
  add eax, 572EC  
  mov ds:Var2, eax//存入 Var2  
  popad  
  push ds:Var2//这里直接 jmp ds:Var2 咋样？  
  retn  
  //////////////////////////////////////////////////////  
  @@2:  
  pushad  
  mov eax,ds:Var1  
  mov dword ptr \[eax+441FD\], 33504233/////fix code  
  mov word ptr \[eax+44201\], 7C41////////////fix code  
  mov dword ptr \[eax+6E124\], 1F2830C5/////Offset @@3Hook VirtualProtect  
  add eax, 441FD  
  mov ds:Var2, eax/////重新保存 Var 值  
  popad  
  mov dword ptr \[ebp\-4\], EA0CDE28//recover 4th CRC5 to normal  
  mov dword ptr \[ebp\-C\], C47B06E4//recover 4th CRC4 to normal  
  mov dword ptr \[ebp\-10\], 9BEFF589//recover 4th CRC3 to normal  
  mov dword ptr \[ebp\-14\], A0901CA0//recover 4th CRC2 to normal  
  mov dword ptr \[ebp\-18\], 0BB997E9//recover 4th CRC1 to normal  
  push ds:Var2, eax//jmp ds:Var2  
  retn  
  //////////////////////////////////////////////////////  
  @@3:  
  pushad  
  pushfd  
  cmp     dword ptr \[MigrateDevColor+1072F0h\], 0///////1EDC8550 ; Exported entry  17. MigrateDevColor   
  je next  
  ;1EECF840    6A FF           push    -1  
  ;  
  ;1EECF840    33C0            xor     eax, eax  
  ;1EECF842    40              inc     eax  
  ;1EECF843    C3              retn  
  ;  
  ;mov eax,1  
    mov     dword ptr \[1EECF840\], C340C033///find call witch call it and patch it with bin code of 'mov eax,1'  
    mov     byte ptr \[EditControlW+0C1EFh\], 0EBh/// Exported entry  11. EditControlW  
    mov     byte ptr \[EditControlW+0C245h\], 0EBh//p.s.这里地址可能不一样……要自己找的……  
    mov     byte ptr \[EditControlW+0C282h\], 0EBh  
    mov     byte ptr \[EditControlW+0C2A3h\], 0EBh  
    mov     byte ptr \[EditControlW+0BF93h\], 0EBh  
   next:  
    popfd  
    popad  
    jmp **VirtualProtect** 

  
  比如拿 1727 做实例就应该这样写代码  

引用:   0x1F2A7360:  
    mov     eax, dword ptr \[1F2BE104\]  
    pushad  
    mov     dword ptr \[1F2A7500\], eax  
    mov     dword ptr \[eax+572EC\], 2a738768  
    mov     dword ptr \[eax+572F0\], 0C31F  
    popad  
    jmp     1F27DA31  
    pushad  
    mov     eax, dword ptr \[1F2A7500\]  
    mov     dword ptr \[eax+572EC\], 8B10C483  
    mov     dword ptr \[eax+572F0\], 0C35DE5  
    mov     dword ptr \[eax+441FD\], 2A73C868  
    mov     word ptr \[eax+44201\], 0C31F  
    add     eax, 572EC  
    mov     dword ptr \[1F2A7504\], eax  
    popad  
    jmp     dword ptr \[1F2A7504\]  
    pushad  
    mov     eax, dword ptr \[1F2A7500\]  
    mov     dword ptr \[eax+441FD\], 33504233  
    mov     word ptr \[eax+44201\], 7C41  
    mov     dword ptr \[eax+6E124\], 1F2A7422  
    add     eax, 441FD  
    mov     dword ptr \[1F2A7504\], eax  
    popad  
    mov     dword ptr \[ebp\-4\], 8A86794A  
    mov     dword ptr \[ebp\-C\], 0xC2123907  
    mov     dword ptr \[ebp\-10\], 0xA5BA06EE  
    mov     dword ptr \[ebp\-14\], 56F72581  
    mov     dword ptr \[ebp\-18\], 6109DF09  
    jmp     dword ptr \[1F2A7504\]  
    pushad  
    pushfd  
    cmp     dword ptr \[1EDD0EE0\], 0  
    je 1F2A7453  
    mov     dword ptr \[1EEE4800\], 0C340C033  
    mov     byte ptr \[1ED552FF\],0EB  
    mov     byte ptr \[1ED55355\],0EB  
    mov     byte ptr \[1ED55392\],0EB  
    mov     byte ptr \[1ED553B3\],0EB  
    popfd  
    popad  
    jmp dword ptr\[1F2B7070\]  
   

  
\--------------------------------------------------------------------------------  
【经验总结】  
  @@3 的部分可以用自己的方法代替，比如说用 y3y3y3 的方法，和我以前所写过的方法，最后 Patch 完了别忘了 jmp   
  btw:不能使用 SetEnvironmentVariable 的方法=.=当然脱了壳以后可以 Set 下面这两个  
  "DAYSLEFT"  
  "DAYSINSTALLED"  
  至于 VirtualProtect，这个 API Armadillo 自己就用到的，可以 jmp dword ptr\[.....\]  
  在这里说明下……那个 4th CRC 是用 ArmaCRC 找出来的，文件在附件里  
  
  附带附件为 Patch，直接打上即可。算是一个礼物吧……不收费啦~~  
  祝天下所有的父亲节日快乐哈~~![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")  
  后传：构建和谐社会，人人有责……^\_^  
  
\--------------------------------------------------------------------------------  
【版权声明】: 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2009 年 06 月 21 日 12:43:52

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ArmaCRC.rar \[解压密码：pediy\]   
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || visual.assist-patch.rar  
 |