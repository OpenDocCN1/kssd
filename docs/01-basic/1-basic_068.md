
# asm 的魅力（四）

> 原文：[`www.pediy.com/kssd/pediy10/101257.html`](https://www.pediy.com/kssd/pediy10/101257.html)

Asm 的魅力（四）Asm 是业界利器-----------------------某人

Date：2009.11.15

Author：charme

Index：http://hi.baidu.com/charme000

Email:charme000@gmail.com

很久没有研究技术的东西了，心静不下来。现在稍微好点了，接着以前的主题，希望大家不要厌烦，呼呼！！

为什么说 asm 是业界利器呢？实际上搞破解和逆向分析的人体会最深了。我以前并不研究什么破解啊什么的，主要是没兴趣，有时候倒是也逆向一些东西！最近稍微的接触了下，觉得挺有意思的，也不是那么难！但是说到底吧，这里面最根本的实际上还是 asm 的功底和一些调试技巧！希望不久可以写个拙略的破文出来。

在此特别感谢下啊 cr 和 zapline 的帮助。

最近写了篇关于 inline hook 的整体架构的东西，放空间了（严重鄙视百度的日志篇幅限制，害我分了 7 篇，丫的！）。师傅说不喜欢看 asm，我也没有办法，很俗的说:习惯了，呼呼！

很多人问我：为什么要叫“asm 魅力“。呼呼，因为我的名字叫 charme（英文意思：魅力）。

昨天一个朋友问我：asm 可以写驱动么？当然可以！！！！！

那么这篇呢我们就以这个为主题，我实现一个简单的驱动出来。

## 原理

我们简单的自己实现一个调用门，来查询 r0 状态是一些寄存器的值！

## 驱动部分

代码:

```
;名称：MyKmd.asm

;功能：一个很简单的 WDM（KDM）驱动程序，为自己的程序提供 Ring0。

;说明：需要由“LoadMyKmd.exe”来安装。


;>>>>>>>>>>>>>>>STRUCT>>>>>>>>>>>>>>>>>>>>>>>>>>>>
CALLGATE  STRUCT
  OFFSETL    DW  0
  SELECTOR  DW  0
  DCOUNT    DB  0
  GTYPE    DB  0
  OFFSETH    DW  0
CALLGATE    ENDS

DESCRIPTOR  STRUCT
  LIMITL    DW  0
  BASEL    DW  0
  BASEM    DB  0
  ATTRIBUTES  DW  0
  BASEH    DB  0
DESCRIPTOR  ENDS
;>>>>>>>>>>>>>>>>>equ>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
CODES\_SEL  =  3E8H
CODE\_TYPE  =  0CF9AH
GATE\_TYPE  =  0ECH

;>>>>>>>>>>>>>>>>>Code>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.code
  DriverEntry:
  pushfd
  pushad

  ;取得 GDTR 内容
  push  edx      ;压栈 edx，esp-4，构建 4 字节空间
  sgdt  \[esp-2\]      ;GDTR 是 48 位，这里利用了技巧，在栈中构建了 6 字节空间，同时读出 GDTR 内容
          ;而且还不破坏堆栈指针，巧巧巧！
  pop  edx      ;全局描述符表基地址这里只关心基址(在高双字，高址)，所以出栈即是
  mov  eax,edx      ;存放全局描述符开始地址

  ;在全句描述符表开始地址存放 retn 指令机器码， 全句描述符表第一个描述符为空
  ;表述符,被保留，没被用书上说这个描述符应该全为 0 这里看出 可以写东西进去。
  mov  BYTE ptr \[edx\],0C3H 

  mov  ecx,3E0h    ;定位自己的门描述符，
  add  edx,ecx      ;

     assume edx:ptr CALLGATE    ;让描述符符位置在全局描述符的 3E0 这个位置
  cmp  \[edx\].SELECTOR,CODES\_SEL
  jz  \_ToEnd

  mov  \[edx\].OFFSETL,ax  ;创建门描述符
  mov  \[edx\].SELECTOR,CODES\_SEL
  mov  \[edx\].DCOUNT,0
  mov  \[edx\].GTYPE,GATE\_TYPE
  shr  eax,16
  mov  \[edx\].OFFSETH,ax
  ;描述符的偏移地址其实就是上面说到的那个 retn 指令地址,当 call 这个门的时候
  ;只从堆栈弹出 EIP 地址,所以 CS 寄存器没被覆盖，所以还是 ring0 权限

  add  edx,8
     assume edx:ptr DESCRIPTOR
  mov  \[edx\].LIMITL,0FFFFH  ;创建自己的代码段描述符
  mov  \[edx\].BASEL,0
  mov  \[edx\].BASEM,0
  mov  \[edx\].ATTRIBUTES,CODE\_TYPE
  mov  \[edx\].BASEH,0
  \_ToEnd:
  popad
  popfd
  mov  eax,0C0000182h
  ret  8 
end DriverEntry

```

说明：

这个源文件实际上可以写成一个 `MyKmd.bat` 文件，对于其中的批处理，编译的时候按注释处理，忽略。这是一种很好的处理方法。或者你也可以单独的写个 bat 文件来编译！代码中作了详细的注释。一看就明白！

这其中几处巧妙的地方正是 asm 得魅力所在，希望大家能够真正的体会出来！

## Loader

我们也可以写个加载驱动的 loader！这里只是演示，就不涉及什么 xx 的东西了，用最普通的方法来加载。

代码:

```
;名称：LoadMyKmd.asm

;功能：安装驱动程序“MyKmd.sys”使用。

;说明：必须生成“MyKmd.sys”之后才能正确运行。

;编译方式见程序尾部


.data
  szError    db  '装载驱动失败',0
  szOK1    db  '驱动安装成功',0
  szOK    db  'OH YEAH.~~!',0
  szMyWdm    db  'MyKmd.sys',0
  szMyWdnName  db  'MyKmd',0
  szBuffer  db  MAX\_PATH dup (0)

  hSCManager  dd  ?
  hService  dd  ?

.code
 start:
  invoke  OpenSCManager,NULL,NULL,SC\_MANAGER\_CREATE\_SERVICE
  .if !eax
    invoke  MessageBox,0,addr szError,0,MB\_ICONERROR or MB\_OK
    jmp \_Err
  .endif
  mov hSCManager,eax
  push ecx
  invoke  GetFullPathName,addr szMyWdm,sizeof szBuffer,addr szBuffer,esp
  pop ecx
  invoke  CreateService,hSCManager,addr szMyWdnName,addr szMyWdm, \\
      SERVICE\_START or DELETE,SERVICE\_KERNEL\_DRIVER, \\
      SERVICE\_DEMAND\_START,SERVICE\_ERROR\_IGNORE, \\
      addr szBuffer,0,0,0,0,0
  .if !eax
    invoke  MessageBox,0,addr szError,0,MB\_ICONERROR
    jmp \_Err
  .endif
  mov hService,eax
  invoke  StartService, hService, 0, NULL
  invoke  DeleteService, hService
  invoke  CloseServiceHandle, hService
  invoke  CloseServiceHandle, hSCManager
  invoke  MessageBox,0,addr szOK1,addr szOK,0
 \_Err:
  invoke  ExitProcess,0
  end start

```

说明：写过驱动的人应该对这些基本的函数都很熟悉的！

现在的话，驱动文件和 loader 都有了。我们再写个测试程序出来！看看到底能不能出来我们想要的效果！

## Test 部分

代码:

```
;程序名：Mytest.asm

;功能：用来测试 MyKmd.sys 是否正确


.data
  szOK    db  'Test Ring0',0
  CallGate\_Sel  dd  0
      dw  03E3H    ;调用门的选择子
  szFormat  db  '通用寄存器：',0Dh,0Ah
      db  'EAX=%08X   EBX=%08X   ECX=%08X   EDX=%08X',0Dh,0Ah
      db  'ESI=%08X   EDI=%08X   EBP=%08X   ESP=%08X',0Dh,0Ah,0Dh,0Ah
      db  '段（选择符）寄存器：',0Dh,0Ah
      db  ' CS=%04X   DS=%04X  ES=%04X   SS=%04X   FS=%04X  GS==%04X',0Dh,0Ah,0Dh,0Ah
      db  '指令指针寄存器：EIP=%08X  标志寄存器：EFLAGS=%08X',0Dh,0Ah,0Dh,0Ah
      db  '控制寄存器：',0Dh,0Ah
      db  ' CR0=%08X   CR1:CPU 保留   CR2=%08X   CR3=%08X',0Dh,0Ah,0Dh,0Ah
      db  '系统地址寄存器：',0Dh,0Ah
      db  ' GDTR=%04X %08X  IDTR=%04X %08X TR=%04X LDTR=%04X',0Dh,0Ah,0Dh,0Ah
      db  '调试寄存器：',0Dh,0Ah
      db  ' DR0=%08X  DR1=%08X   DR2=%08X   DR3=%08X',0Dh,0Ah
      db  ' DR4：CPU 保留  DR5：CPU 保留   DR6=%08X   DR7=%08X',0,0
.data?
  szBuffer  db  500h dup (?)
  \_ddEAX    dd  ?
  \_ddEBX    dd  ?
  \_ddECX    dd  ?
  \_ddEDX    dd  ?
  \_ddESI    dd  ?
  \_ddEDI    dd  ?
  \_ddEBP    dd  ?
  \_ddESP    dd  ?
  \_ddCS    dd  ?
  \_ddDS    dd  ?
  \_ddES    dd  ?
  \_ddSS    dd  ?
  \_ddFS    dd  ?
  \_ddGS    dd  ?
  \_ddEIP    dd  ?
  \_ddEFLAGS  dd  ?
  \_ddCR0    dd  ?
  ;\_ddCR1    dd  00000000h  ;CPU 保留
  \_ddCR2    dd  ?
  \_ddCR3    dd  ?
  \_dfGDTR    dd  ?    ;低 16 位是界限
      dd  ?
  \_dfIDTR    dd  ?
      dd  ?
  \_ddTR    dd  ?
  \_ddLDTR    dd  ?
  \_ddDR0    dd  ?
  \_ddDR1    dd  ?
  \_ddDR2    dd  ?
  \_ddDR3    dd  ?
  ;\_ddDR4    dd  00000000h  ;CPU 保留
  ;\_ddDR5    dd  00000000h  ;CPU 保留
  \_ddDR6    dd  ?
  \_ddDR7    dd  ?

.code
start:
  call  FWORD ptr CallGate\_Sel

  ;进入 ring0
  ;有兴趣自己也可以创建一个 0 级堆栈，这里用 ring3 的堆栈
  mov  eax,esp        ;保存 ring0 堆栈
  mov  esp,\[esp+4\]      ;装入 ring3 堆栈地址
  ;解释下这里为什么是+4：  因为 call 调用门会把 ss esp cs eip 分别放入堆栈，在驱动
  ;程序里面已经有一句 ret 8 指令把 eip 弹出来了。所以这里就得加 4 以得到 ring3 堆栈地址
    
  push  eax

  mov  \_ddEAX,eax
  mov  \_ddEBX,ebx
  mov  \_ddECX,ecx
  mov  \_ddEDX,edx
  mov  \_ddESI,esi
  mov  \_ddEDI,edi
  mov  \_ddEBP,ebp
  mov  \_ddESP,esp
  mov  \_ddCS,DWORD PTR cs
  mov  \_ddDS,DWORD PTR ds
  mov  \_ddES,DWORD PTR es
  mov  \_ddES,DWORD PTR ss
  mov  \_ddFS,DWORD PTR fs
  mov  \_ddGS,DWORD PTR gs
  push  eax
  pop  \_ddEIP
  pushfd  
  pop  \_ddEFLAGS
  mov  eax,cr0
  mov  \_ddCR0,eax
  mov  eax,cr2
  mov  \_ddCR2,eax
  mov  eax,cr3
  mov  \_ddCR3,eax
  sgdt  FWORD PTR \_dfGDTR
  sidt  FWORD PTR \_dfIDTR
  xor  eax,eax
  str  ax
  mov  \_ddTR,eax
  xor  eax,eax
  sldt  ax
  mov  \_ddLDTR,eax
  mov  eax,dr0
  mov  \_ddDR0,eax
  mov  eax,dr1
  mov  \_ddDR1,eax
  mov  eax,dr2
  mov  \_ddDR2,eax
  mov  eax,dr3
  mov  \_ddDR3,eax
  mov  eax,dr6
  mov  \_ddDR6,eax
  mov  eax,dr7
  mov  \_ddDR7,eax      ;ring0 下才可以访问的寄存器

  pop  esp

  ;;;开发返回 ring3
  push  offset ring3

  ;正如刚刚所说，已经弹出了 eip 地址。
  ;用 retf 返回肯定出错。所以还是手动帮忙来恢复堆栈，压入 ring3 下的代码地址
  retf
ring3:
  mov  eax,\_dfGDTR
  mov  ebx,\_dfGDTR+4
  xchg  ax,bx
  rol  eax,16
  mov  \_dfGDTR,eax
  mov  \_dfGDTR+4,ebx
  mov  eax,\_dfIDTR
  mov  ebx,\_dfIDTR+4
  xchg  ax,bx
  rol  eax,16
  mov  \_dfIDTR,eax
  mov  \_dfIDTR+4,ebx

  invoke  wsprintf,addr szBuffer,addr szFormat,\\
    \_ddEAX,\_ddEBX,\_ddECX,\_ddEDX,\\
    \_ddESI,\_ddEDI,\_ddEBP,\_ddESP,\\
    \_ddCS,\_ddSS,\_ddDS,\_ddES,\_ddFS,\_ddGS,\\
    \_ddEIP,\_ddEFLAGS,\\
    \_ddCR0,\_ddCR2,\_ddCR3,\\
    \_dfGDTR+4,\_dfGDTR,\_dfIDTR+4,\_dfIDTR,\_ddTR,\_ddLDTR,\\
    \_ddDR0,\_ddDR1,\_ddDR2,\_ddDR3,\_ddDR6,\_ddDR7
  invoke  MessageBox,0,addr szBuffer,addr szOK,0
  invoke  ExitProcess,0
end start

```

说明：也是很简单的！呼呼！

最终的效果图：

这篇陋文不是要教你怎么写个调用门什么的！要写这样的东西的话，看看 combojiang（也不知道拼对了没有，呼呼）大牛的 rootkits 系列就可以了！ 我的目的是想让大家体会下 asm 的魅力，仅此而已！比起 N 多大牛的内核驱动程序来说，这个就很简陋了，呼呼！不过现在好多的大牛都简陋的不会写简陋的东西了！

## 建议

我个人是这么看的，我并不建议用 asm 来写驱动，除非你有足够的驾驭能力。

原因 1-----asm 本身很晦涩，即便他强大。Asm 写驱动就有点像用 mfc 开发 gui 程序和.net 开发 gui 程序的对比。前者可能在某些方面要强大一点，但是开发速度和效率没有后者强！这个看情况择优！



## 原因 2
`asm` 写的驱动不好调试。最普遍的就是三精口服液。

### 总结
其实任何的语言无非是规则，高手在于能够充分的理解规则并且试图突破！

对于 `asm` 写驱动，前辈已经做了尝试，大家可以到小罗的主页上下载俄罗斯大牛整理的 `masm` 开发驱动的开发包 `KmdKit`。相当于 `ddk` 一样的东西，很实用！

好了，`asm` 魅力四写完了！就像又完成了一件事情一样！也很开心！

昨天受人委托去辅导一个高中生的去过信息竞赛，不禁感慨一番，里面设计很多的数据结构还有算法的问题，一个动态规划的问题，忙活了我两个小时才写出程序来。看来还是欠火候，希望小妹妹可以一切顺利，也不枉我一番教导啊！呼呼！

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `MyKmd.rar` |
