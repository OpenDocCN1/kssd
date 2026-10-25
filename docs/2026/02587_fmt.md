
# Soundnailsd的破解教程（一） (9千字)

Soundnailsd的破解教程（一）

这个软件是peterchen大哥推荐的,其实早就搞定了,由于近来事比较多,还有一个重要的原因,那就是俺比较的懒嘛,所以教程现在这才写好....嘿嘿 *^_^*

下载地址：http://www.esoftware.com.cn/file/photo/animator/2001092101.shtml

## 简介

SWF 是 Macromedia Flash 的专属档案格式，它原是用来制作 Flash 动画时所产生的动画档；
而 MP3 是档案小、音极佳的多媒体音乐格式的代表。不过 MP3 并无法像 Flash 动画或串流多媒体格式，直接在网络上播放，所以常常都只是透过 Internet 来传输档案，无法达到实时播放的功能，这是比较可惜的地方。
现在 Soundnails 能够以简单的步骤，将众多 MP3 档案轻易的批次转换为 SWF 档案，而且还能选择不同样板，输出制式化的 HTML 网页，让网友直接在网页上播放音乐，这结合了 MP3 及 SWF 两者的优点，表现优异当然是可想而知了。使用 Soundnails 的好处，是它并不需要先行安装 Macromedia Flash，而且要播放 SWF 档案也不需要安装其它的音乐播放程序，只要选择想要转换的 MP3 档案，然后 step by step 的跟着做，就可以输出网页 HTML 组件，然后再上传到 Web Server 中，网友就可以直接在网络上欣赏好听的音乐了。Soundnails 在制作 SWF 时，会自动抓出 MP3 的 ID3 Tag 信息，而呈现在所制作的 HTML 网页中 (或者也能在 Soundnails 中编辑 MP3 档案的 ID3 Tag)；网页上的播放按钮，内建有五组 Flash 动画可选择套用，而输出网页也有五套风格相异的网页样版可挑选。试用版有 30 天及一次处理只能 5 个 MP3 档案等功能限制。

先用`wdasm32`静态分析一下看看，略去了很多无关的代码有兴趣的可以自己看一看.

代码如下：

```
:00457572 C605D8F0470000          mov byte ptr [0047F0D8],00
:00457579 B890084800              mov eax,00480890
:0045757E E801040000              call 00457984
-------->
:00457583 8BF0
  mov esi, eax
:00457585 8D45C4
lea eax, dword ptr [ebp-3C]
:00457588 E8F7030000              call 00457984
---------->与上面的是同一个CALL.是个关键CALL
:0045758D 8BF8
  mov edi, eax
:0045758F 3BF7
  cmp esi, edi
:00457591 0F8DA1000000            jnl 00457638
:00457597 8BC7
  mov eax, edi
:00457599 48
  dec eax
:0045759A 7405
  je 004575A1------>注意这个跳转和下面那个跳转,
      这个跳转跳向 EASY Version
:0045759C 48
  dec eax
:0045759D 741C
  je 004575BB--->这个跳转跳向 PRO Version
:0045759F EB34
  jmp 004575D5
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045759A(C)
|
:004575A1 6A00
  push 00000000
* Possible StringData Ref from Code Obj ->"Registration"
            |
:004575A3 B9A8764500              mov ecx,004576A8
* Possible StringData Ref from Code Obj ->"EASY Version registered. Restart " --->* *一个名字对应两个注册码，
                  ->"Program to
activate Changes."            分别来对应 EASY Version
和下面
            |
                的 PRO Version
            |
:004575A8 BAB8764500              mov edx,004576B8
:004575AD A1E8F54700              mov eax,dword ptr [0047F5E8]
:004575B2 8B00
  mov eax, dword ptr [eax]
:004575B4 E8476FFFFF              call 0044E500
:004575B9 EB32
  jmp 004575ED
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045759D(C)
|
:004575BB 6A00
  push 00000000
* Possible StringData Ref from Code Obj ->"Registration"
            |
:004575BD B9A8764500              mov ecx,004576A8
* Possible StringData Ref from Code Obj ->"PRO Version registered. Restart "-->* *
                  ->"Program to
activate Changes."
            |
:004575C2 BAF8764500              mov edx,004576F8
:004575C7 A1E8F54700              mov eax,dword ptr [0047F5E8]
:004575CC 8B00
  mov eax, dword ptr [eax]
:004575CE E82D6FFFFF              call 0044E500
:004575D3 EB18
  jmp 004575ED
      :
            :
      :
            :(略去不少代码)
```

大体分析一下上面的代码,可断定:`0045757E  call 00457984` 里面有注册码的算法及比较过程,那我们就来动态跟踪分析一下这个call.用`TRW2000`.

(无关的代码没有作说明)

代码如下:

```
0167:00457984  PUSH    EBP
0167:00457985  MOV      EBP,ESP
0167:00457987  ADD      ESP,BYTE -1C
0167:0045798A  PUSH    EBX
0167:0045798B  PUSH    ESI
0167:0045798C  PUSH    EDI
0167:0045798D  XOR      EDX,EDX
0167:0045798F  MOV      [EBP-14],EDX
0167:00457992  MOV      [EBP-10],EDX
0167:00457995  MOV      [EBP-0C],EDX
0167:00457998  MOV      EBX,EAX
0167:0045799A  XOR      EAX,EAX
0167:0045799C  PUSH    EBP
0167:0045799D  PUSH    DWORD 00457B88
0167:004579A2  PUSH    DWORD [FS:EAX]
0167:004579A5  MOV      [FS:EAX],ESP
0167:004579A8  LEA      EAX,[EBX+24]
0167:004579AB  MOV      EDX,00457BA0
0167:004579B0  CALL    00403BE0
0167:004579B5  XOR      ESI,ESI
0167:004579B7  CMP      DWORD [EBX],BYTE +00  ----->拿指向用户名的指针与00比较
0167:004579BA  JZ      00457A1F
      ----->为0则跳转
0167:004579BC  CMP      DWORD [EBX+04],BYTE
```
