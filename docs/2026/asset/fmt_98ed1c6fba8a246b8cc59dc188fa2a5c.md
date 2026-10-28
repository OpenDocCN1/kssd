
### 3. INightElf 个人觉得逆向分析与软件编程一样，非常忌讳还没有设计就开始编码。一定要有自己的逆向思路，不能猫抓死耗子。

不过这样分析仍然困难重重，突然想到论坛上面有一个 Delphi 逆向神器，能解析出窗口资源和库函数。

当当当……`DarkDe 4` 登上了舞台。

用 `DarkDe 4` 直接反汇编 `dzip32.dll`，保存反汇编结果，查看 `dzip32.dpr` 工程文件。

我靠，竟然把 `0061DB6C` 函数识别出来，原来是库函数，省略了太多功夫了，直接重新命名该函数，那么现在的焦点就是查看 `off_689188` 地址是什么东西，直接导航进去，看起来有点规律，但是仍然不知道是什么东西。潜意识有可能是 Delphi 创建窗口的机制，因为很多真正处理事件的业务都会放到 Form 窗口里面进行处理。

`dzip32.dpr` 工程文件：

```
00689736   33C9                   xor     ecx, ecx
00689738   B201                   mov     dl, $01
0068973A   A188916800             mov     eax, dword ptr [$00689188]

* Reference to: Classes.TThread.Create(TThread;boolean;Boolean);
|
0068973F   E82844F9FF             call    0061DB6C
00689744   33C0                   xor     eax, eax
00689746   5A                     pop     edx
00689747   59                     pop     ecx
00689748   59                     pop     ecx
00689749   648910                 mov     fs:[eax], edx
```

### 4. 现在又被卡壳了。因为我是新手第一次反汇编 Delphi 代码，我不知道 Delphi 创建窗口会调用哪些函数，好吧。那就自己新建一个 Delphi 工程试试，我机器上面没有 Delphi 5，只有 Delphi 7，用 Delphi 7 编译一个工程吧，看看有什么样的结果。

有两种反汇编的结果，一种是调用：

```
Application.CreateForm(TForm1, Form1);
Application.Run;
```

生成的反汇编：

```
CODE:0044DD40                 push    ebp
CODE:0044DD41                 mov     ebp, esp
CODE:0044DD43                 add     esp, 0FFFFFFF0h
CODE:0044DD46                 mov     eax, offset dword_44DB58
CODE:0044DD4B                 call    @Sysinit@@InitExe$qqrpv ; Sysinit::__linkproc__ InitExe(void *)
CODE:0044DD50                 mov     eax, ds:off_44EFC0
CODE:0044DD55                 mov     eax, [eax]
CODE:0044DD57                 call    sub_44C20C
CODE:0044DD5C                 mov     ecx, ds:off_44F09C
CODE:0044DD62                 mov     eax, ds:off_44EFC0
CODE:0044DD67                 mov     eax, [eax]
CODE:0044DD69                 mov     edx, off_44D94C _cls_Unit1_TForm1
CODE:0044DD6F                 call    Forms::TApplication::CreateForm(System::TMetaClass *,void *)
CODE:0044DD74                 mov     eax, ds:off_44EFC0
CODE:0044DD79                 mov     eax, [eax]
CODE:0044DD7B                 call    @Forms@TApplication@Run$qqrv ; Forms::TApplication::Run(void)
CODE:0044DD80                 call    @System@@Halt0$qqrv ; System::__linkproc__ Halt0(void)
```

另一种是调用：

```
form2:= TForm2.Create(nil);
form2.show;
```

生成的反汇编结果：

```
xor     ecx, ecx
mov     dl, 1
mov     eax, off_44D7AC   ; _cls_Unit2_TForm2
call    Forms::TCustomForm::TCustomForm(Classes::TComponent *)
call    Forms::TCustomForm::Show(void)
retn
```

两种方法无一例外都要引用到即将创建的窗口类函数 `_cls_Unit2_TForm2` 和 `_cls_Unit2_TForm1`。

创建窗口的函数为：

`Forms::TApplication::CreateForm(System::TMetaClass *,void *)`

和

`Forms::TCustomForm::TCustomForm(Classes::TComponent *)`

现在我们来对比 `dzip32.dll` 看看是否有相同之处。

### 5. 分析 `Classes.TThread.Create(TThread;boolean;Boolean)` 函数，可以确定真正的线程函数为 `offset sub_689220`，怎么分析我就不展开了，假如不明白可以 email 我。

```
.code:00689188 off_689188      dd offset off_6891D4    ; DATA XREF: DllEntryPoint+156 r
.code:0068918C                 dd 7 dup(0)
.code:006891A8                 dd offset loc_6891DC
.code:006891AC                 dd 40h
.code:006891B0                 dd offset off_616480
.code:006891B4                 dd offset sub_6039C4
.code:006891B8                 dd offset sub_61DCA4
.code:006891BC                 dd offset nullsub_16
.code:006891C0                 dd offset sub_6039D8
.code:006891C4                 dd offset nullsub_17
.code:006891C8                 dd offset sub_603714
.code:006891CC                 dd offset sub_603730
.code:006891D0                 dd offset sub_61DC3C
.code:006891D4 off_6891D4      dd offset sub_61DD58    ; DATA XREF: .code:off_689188 o
.code:006891D8                 dd offset sub_689220
.code:00689220 sub_689220      proc near               ; DATA XREF: .code:006891D8o
.code:00689220                 mov     eax, ds:off_69006C
.code:00689225                 cmp     dword ptr [eax], 0
.code:00689228                 jnz     short locret_689265
.code:0068922A                 mov     eax, ds:off_68FF2C
.code:0068922F                 mov     eax, [eax]
.code:00689231                 call    sub_65A454
.code:00689236                 mov     ecx, ds:off_69006C
.code:0068923C                 mov     eax, ds:off_68FF2C
.code:00689241                 mov     eax, [eax]
.code:00689243                 mov     edx, off_6877E4
.code:00689249                 call    sub_65A46C
.code:0068924E                 mov     eax, ds:off_68FF2C
.code:00689253                 mov     eax, [eax]
.code:00689255                 mov     byte ptr [eax+5Bh], 0
.code:00689259                 mov     eax, ds:off_68FF2C
.code:0068925E                 mov     eax, [eax]
.code:00689260                 call    sub_65A4EC
.code:00689265
.code:00689265 locret_689265:                          ; CODE XREF: sub_689220+8j
.code:00689265                 retn
.code:00689265 sub_689220      endp
```

#### `sub_689220` 和第一种反汇编结果对比一下，看看有什么相同之处，请详细看。激动人心的时刻到了。

```
sub_689220                                        第一种反汇编结果
mov     ecx, ds:off_69006C               mov     ecx, ds:off_44F09C
mov     eax, ds:off_68FF2C               mov     eax, ds:off_44EFC0
mov     eax, [eax]                              mov     eax, [eax]
mov     edx, off_6877E4                    mov     edx, off_44D94C _cls_Unit1_TForm1
call    sub_65A46C   call    Forms::TApplication::CreateForm(System::TMetaClass *,void *)
mov     eax, ds:off_68FF2C
mov     eax, [eax]
mov     byte ptr [eax+5Bh], 0
mov     eax, ds:off_68FF2C               mov     eax, ds:off_44EFC0
mov     eax, [eax]                             mov     eax, [eax]
call   sub_65A4EC  call    @Forms@TApplication@Run$qqrv ; Forms::TApplication::Run(void)
```

`off_6877E4` 的截图：

![](https://bbs.kanxue.com/upload/attach/201304/467706_nre1vztdy9bzvga.jpg)

`off_44D94C _cls_Unit1_TForm1` 的截图：

![](https://bbs.kanxue.com/upload/attach/201304/467706_13otpb6cfaqldf5.jpg)

导航 `off_6877E4` 地址进去，跟 `off_44D94C _cls_Unit1_TForm1` 差不多。

而且 `sub_65A4EC` 函数和 `Forms::TApplication::Run(void)` 内容一模一样。

已经可以确定 `ff_6877E4` 是一个窗口类了。

此时此刻，多令人激动。因为接下去即将是分析 Form 的内容，而 `DarkDe` 已经完全把窗口的函数反汇编出来。而且 `_cls_Unit1_TForm1` 已经把窗口类的父类函数都标记出来，那么 `off_6877E4` 窗口类也同样可以参照其标记出来。终于踏上了光明大道了。这一步实在是太重要了。非常关键的一步。

接下来便是确定 `ff_6877E4` 是那个窗口类。

`DarkDe` 反汇编出来有 5 个窗口类：

![](https://bbs.kanxue.com/upload/attach/201304/467706_0oz6vqjglrbjjm6.jpg)

各位新手可以按照我的思路练练手，提高自己的反汇编分析能力。

今天分析就到此为止，请各位看官给点掌声吧。

看雪论坛，基本是只看不回复的。哈哈。

这个 `dzip32.dll` 去年就已经完全逆向出来了，这几天只是觉得不把它拿出来，让它烂在电脑里面，有点对不起我的付出。因为随着时间的推移，我可能没有精力，也没有兴趣去做这件事情，这个月打算辞职换工作，打算往软件上层的方向发展，往项目经理的方向。

但是我一定会把 `dzip32.dll` 逆向的过程完全发表出来，也做为一段疯狂的日子的见证。其实写这篇文章，是非常耗费精力和时间的，你破解的时候思维可以跳跃得很大，但是写文章就不可以了。

欲知后事如何，请听下回分解！

---

**未完待续**

---

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [1.jpg](javascript:void(0);) （63.88kb，39次下载）
- [2.jpg](javascript:void(0);) （38.25kb，23次下载）
- [劫持篇1.jpg](javascript:void(0);) （63.54kb，14次下载）
- [劫持篇2.jpg](javascript:void(0);) （71.47kb，18次下载）
- [DZIP32.zip](javascript:void(0);) （1.17MB，102次下载）
- [虚拟机1.jpg](javascript:void(0);) （44.57kb，21次下载）
- [反加载篇1.jpg](javascript:void(0);) （45.04kb，32次下载）
- [rose 流程图.jpg](javascript:void(0);) （76.14kb，62次下载）
- [Delphi 基础汇编1.jpg](javascript:void(0);) （42.66kb，5次下载）
- [Delphi 基础汇编2.jpg](javascript:void(0);) （71.40kb，9次下载）
- [Delphi 基础汇编-CreateThread.jpg](javascript:void(0);) （21.13kb，8次下载）
- [Delphi 基础汇编-CreateThread2.jpg](javascript:void(0);) （72.09kb，12次下载）
- [Delphi 基础汇编-CreateThread3.jpg](javascript:void(0);) （84.31kb，14次下载）
- [Delphi 基础汇编-自定义对比.jpg](javascript:void(0);) （119.32kb，7次下载）
- [Delphi 基础汇编-自定义对比2.jpg](javascript:void(0);) （73.61kb，8次下载）
