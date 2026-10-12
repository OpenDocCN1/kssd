
# 从 IDA 中提取代码做汇编注册机之 VB 篇

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-501.htm`](https://www.pediy.com/kssd/pediy08/pediy8-501.htm)

看了 deroko 的《Ripping VB code and making keygen out of it》，觉得写得非常好，给写算法注册机带来了不少方便之处。但是作者在最后部分特别是写注册机方面过于烦琐，不直观，尤其是其中引用大部分的宏在 `TASM` 中编译时带来不少麻烦，所以我重新编辑了下，连翻带改写了这篇文章，里面的许多内容及一些处理方法会跟作者的有所出入，并且添加了许多内容。附上在 `RADASM` 中编译通过的 `MASM32` 代码。

## 一、找到关键算法处

这是个 VB 写的 `CRACKME`，先用 VB 的反编译利器 `P32Dasm`（`http://t4c.ic.cz/forum/showthread.php?t=67`）反编译这个软件，得到以下信息：

```
文件: C:\Documents and Settings\ crackme\crackme.exe

P32Dasm v2.3

VB6 编译类型: NCode

frm1

004028F4  1.1 cmdExit.Click()

004029C6  1.2 cmdRegister.Click()       注册按钮处理过程

00402BD3  1.3 Command1.Click()

00402D3D  1.4 Command2.Click()

0040315C  1.5 Form.Load()

00403332  1.6 txtReg.KeyPress(KeyAscii As Integer)

文件反编译结束。
```

从这里，我们得到信息，该 `CRACKME` 的按钮处理过程在 `004029C6` 处。

用 `OD` 载入，在 `004029C6` 处下断，运行程序后随便输入注册码，按注册后断下。单步跟踪，到下面代码：

```
00402AB2   .  68 24404000   PUSH crackme.00404024                 ;  00404024  001540A4  UNICODE "Serializer"

00402AB7   .  50            PUSH EAX                               ;  0012F4EC  0015A04C  UNICODE "6D6074621C090102057374017402 07"

00402AB8   .  E8 00090000   CALL crackme.004033BD                 ;  关键算法

00402ABD   .  8BD0          MOV EDX,EAX                             ;  得到注册码

00402ABF   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]

00402AC2   .  E8 99E7FFFF   CALL <JMP.&msvbvm60.__vbaStrMove>

00402AC7   .  50            PUSH EAX

00402AC8   .  E8 99E7FFFF   CALL <JMP.&msvbvm60.__vbaStrCmp>      ;  对比注册码

00402ACD   .  8BF0          MOV ESI,EAX
```

将 `OD` 中的内存数据窗口切换到“长型”-"地址”，我们在 `00402AB2` 和 `00402AB7` 处右击-在数据窗口中跟随，可以看到这两处分别是 `UNICODE` 字符串“`Serializer`”和机器码，`00402AB8` 处是关键算法，`00402ABD` 处得到真正的注册码，所以 `00402AB8` 处的 `CALL` 非常关键，但是我们现在不想了解其中的算法过程，只想拷贝 `CALL 004033BD` 里面的代码做注册机。

## 二、截取代码

启动 `IDA`，反汇编该 `CRACKME`，“跳转到指定地址”，填入 `004033BD`，将 `004033BD` 到 `00403613` 处代码涂黑（即 `CALL 004033BD` 的整个内容），选择菜单“文件”-创建文件-创建 `ASM` 文件，我们保存为 `crackme.asm`。

我们将这段代码随便找个汇编注册机模版放进去，这里将它放到我修改的 `ARTEAM` 的一个模版里。在 `RADASM` 中编译，让它列出错误。

## 三、分析代码

先讲一些基本的错误：

1、`seg000:004033C3` push (offset `loc_401175`+1)

从 `IDA` 中可以看出这是个压入异常句柄，置 0 即可，我们把它改为：

```
push   offset  dummy_seh

...

dummy_seh:      xor     eax, eax

                ret
```

2、`mov eax, large fs:0`

此处的 `large` 是修饰符，直接去掉，“`fs:0`” `MASM` 也认不出来，我们在 `.DATA` 区加入

```
assume  fs:nothing
```

3、`seg000:004033DF mov dword ptr [ebp-8], offset dword_401160`

我们在 `OD` 中可以看到，`dword_401160= 8000Eh`

所以在 `.DATA` 处加入：

```
dword_401160            dd 8000Eh
```

4、`seg000:00403572 push offset dword_4027BC`

这里其实 `IDA` 反汇编错误（因为 VB 的字符串是以 `UNICODE` 形式存在的），`dword_4027BC` 是字符串 `"0"` 和 `2` ，即：

```
dd 2 <-- --- 字符串长度

string_0 dd 30h <----- 字符串 0

所以在.DATA 处加入：

                 dd 2

dword_4027BC          dd 30h
```

5、`jo loc_403616`

`push offset loc_403602`

`jmp short loc_4035E9`

这几个可以在 `IDA` 中直接跟到代码处拷贝相应的内容：

```
loc_403616:        ; CODE XREF: seg000:00403488 j

          ; seg000:004034F2 j ...

    call    __vbaErrorOverflow

loc_403602:        ; DATA XREF: seg000:004035B9 o

    mov  ecx, [ebp-14h]

    mov  eax, [ebp-20h]

    pop  edi

    pop  esi

    mov  large fs:0, ecx

    pop  ebx

    leave

    retn  8

  loc_4035E9:        ; CODE XREF: seg000:004035BE j

    lea  ecx, [ebp-24h]

    call   __vbaFreeStr

    lea  ecx, [ebp-28h]

    call   __vbaFreeStr

    lea  ecx, [ebp-2Ch]

    call   __vbaFreeStr

    retn
```

现在来讲一个重点错误：

就是所有的 VB 内部函数，这个在 `MASM` 中当然不可能被识别。但是…

感谢 `Vortex` 为我们解决了这个问题，它的 `Dll2inc`（`http://www.vortex.masmcode.com/`）轻松解决了这个烦恼。`Vortex` 原本是想用来生成可以供 `MASM` 调用的 C 运行库的，可是我拿 VB 的运行库 `msvbvm60.dll` 测试了一下，居然成功了。

拿 `msvbvm60.dll` 经过 `Dll2inc` 转换后得到 `msvbvm60.def`、`msvbvm60.inc`、`msvbvm60.lib`，这样就可以在 `MASM32` 中调用了。他专门定义了个调用这些函数的宏：`cinvoke`。我们只要在 `include` 和 `includelib` 中加入这些 `INC` 文件和 `LIB` 文件，然后将所有的调用 VB 内部函数的 `CALL` 改为 `cinvoke` 就可以“光明正大”的用这些 VB 内部函数了。

`RADASM` 中编译，可是还是有错误。

我们用 `OD` 来跟一下到底问题出在哪里。`RADASM` 中设置好调试器关联后，选择“构建”-"在调试器中运行”，`OD` 自动反汇编了这个有问题的 `KEYGEN`。我们逐步跟踪，到下面的代码时，出现异常，问题就出在 `__vbaStrCat` 中。

```
004012BC   .  E8 2B020000   CALL <JMP.&msvbvm60.__vbaStrCat>
```

我们跟入 `__vbaStrCat`：

```
660E5F3A >  55              PUSH EBP

660E5F3B    8BEC            MOV EBP,ESP

660E5F3D    8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]

660E5F40    50              PUSH EAX

660E5F41    FF75 08         PUSH DWORD PTR SS:[EBP+8]

660E5F44    FF75 0C         PUSH DWORD PTR SS:[EBP+C]

660E5F47    FF15 18EE1066   CALL DWORD PTR DS:[6610EE18]             ; DS:[6610EE18]=00000000,问题出在这里
```

跟到 `660E5F47` 时出现异常，我们看到 `PTR DS:[6610EE18]` 为 0，我们拿原来的 `CRACKME` 或者直接拿 `msvbvm60.dll` 反汇编，跟入 `__vbaStrCat`：

```
660E5F47    FF15 18EE1066   CALL DWORD PTR DS:[6610EE18]             ; OLEAUT32.VarBstrCat
```

可以看出问题所在：`PTR DS:[6610EE18]` 在 `CRACKEME` 中有被初始化成 `VarBstrCat` 而在 `KEYGEN` 中没有。

我们用 `IDA` 反汇编 `msvbvm60.dl` 后，跟踪 `__vbaStrCat` 函数：

```
.text:660E5F40                   push    eax

.text:660E5F41                   push    [ebp+arg_0]

.text:660E5F44                   push    [ebp+arg_4]

.text:660E5F47                   call    dword_6610EE18
```

跟进 `CALL`：

```
.data:6610EE18 dword_6610EE18  dd 0                       ; DATA XREF: sub_66004F58+46C w

.data:6610EE18                                     ; __vbaStrCat+D r
```

继续跟踪：

```
.text:660053BA                   push    offset aVarbstrcat ; "VarBstrCat"

.text:660053BF                   push    edi

.text:660053C0                   call    esi ; GetProcAddress

.text:660053C2                   test    eax, eax

.text:660053C4                   mov     dword_6610EE18, eax
```

我们来看看哪个输出函数能够导出该功能，继续跟踪下去：

```
.text:66004D81 sub_66004D81    proc near                   ; CODE XREF: sub_66004D59+20 p

.text:66004D81                   ; CreateIExprSrvObj+3E p
```

`CreateIExprSrvObj` 觉得可疑，跟进看看：

```
.text:660EA734                   public CreateIExprSrvObj

.text:660EA734 CreateIExprSrvObj proc near

.text:660EA734

.text:660EA734 var_8               = dword ptr -8

.text:660EA734 var_4               = dword ptr -4

.text:660EA734 arg_0               = dword ptr  8

.text:660EA734 arg_4               = word ptr  0Ch

.text:660EA734 arg_8               = word ptr  10h

.text:660EA734

.text:660EA734                   push    ebp

.text:660EA735                   mov     ebp, esp

.text:660EA737                   push    ecx

.text:660EA738                   push    ecx

.text:660EA739                   push    ebx

.text:660EA73A                   xor     ebx, ebx

.text:660EA73C                   cmp     [ebp+arg_4], 4

.text:660EA741                   push    esi

.text:660EA742                   push    edi

.text:660EA743                   mov     [ebp+var_8], ebx

.text:660EA746                   mov     [ebp+var_4], ebx

.text:660EA749                   jnz     loc_660EA7F6

.text:660EA74F                   cmp     [ebp+arg_8], bx

.text:660EA753                   ja      loc_660EA7F6

.text:660EA759                   call    sub_660CCD82

.text:660EA75E                   test    eax, eax

.text:660EA760                   jnz     short loc_660EA782

.text:660EA762                   push    2

.text:660EA764                   push    ebx

.text:660EA765                   push    ebx

.text:660EA766                   push    9

.text:660EA768                   push    ebx

.text:660EA769                   push    ebx

.text:660EA76A                   push    2636h

.text:660EA76F                   push    ebx

.text:660EA770                   push    6

.text:660EA772                   call    sub_66004D81  <---- 就是这个 CALL
```

我们需要这个 `CALL`，`deroko` 对 `CALL` 进行了修改并使之运行正常：

```
push    0

push    4

push    0

call    CreateIExprSrvObj(记得改 call 为 cinvoke)
```

我们复制到代码区，`RADASM` 中编译已经通过了。



## 四、解决 UNICODE 字符串

我们注意到在调用这个算法 `CALL` 之前有压入机器码和字符串 "Serializer"，这些字符串在 `VB` 中都是以 `UNICODE` 形式存在的，所以在定义字符串 "Serializer" 时我们必须定义为 `UNICODE` 字符格式。

利用 `MASM32` 的宏 `ucMacros.asm` 可以轻易做到只要在定义的字符串变量前面加 `WSTR` 即可：

```
WSTR static_string,     "Serializer"
```

我们还需要在代码中加入压入机器码和字符串 "Serializer" 的代码，然后才能调用 `CALL`。

## 五、完善注册机

如果进一步分析，会发现软件机器码的由来。

```
0040322C   .  50            PUSH EAX
0040322D   .  E8 EEF3FFFF  CALL crackme.00402620               ; 此 CALL 跳到 GetComputerNameW 函数,返回计算机名
...
00403286   .  57            PUSH EDI                             ; 00404024  0013F63C  UNICODE "Serializer"
00403287   .  50            PUSH EAX                             ; 计算机名  UNICODE "OEM-MICRO"
00403288   .  E8 30010000  CALL crackme.004033BD               ; 算法 CALL
0040328D   .  8BD0         MOV EDX,EAX                          ; 得到机器码
```

也就是说计算机名和字符串 "Serializer" 经过这个算法 `CALL` 后得到机器码。

在注册机中我们就可以直接用该代码获取机器码。

好了，主要代码已经贴在下面了，其实关键 `CALL` 内的算法只是字符串间简单的异或操作。这里讨论的不是怎么去分析这个算法过程。我主要是想大家了解的是这种注册机写法的思路。若有差错，希望补充。

```asm
Generate proc

        push    0
        push    4
        push    0
        cinvoke   CreateIExprSrvObj  

; -----------------------------------------------------------   计算机器码                      
        mov  computer_name_size,sizeof computer_name     
        invoke   GetComputerNameW,addr computer_name,addr computer_name_size        
        xor     eax, eax
        mov     edi, offset computer_name
        stc
        sbb     ecx, ecx
        cld
        repnz   scasw
        not     ecx
        dec     ecx
        shl     ecx, 1
        mov     computer_name_size, ecx
        mov     eax, offset static_string
        mov     ptr1, eax
        mov     eax, offset computer_name
        mov     ptr2, eax
        push    offset ptr1                    ;压入 UNICODE 字符串"Serializer"
        push    offset ptr2                    ;压入计算机名
        call    loc_4033BD                     ;算法 CALL       
        mov     esi, eax                       ;得到机器码
        mov     edi, offset hardware_key
        xor     ecx, ecx

; -----------------------------------------------------------   将机器码放到 hardware_key 中供机器码框调用
copy_hardwarekey:       lodsw
        test    ax, ax
        jz      get_hardwarekey
        add     ecx, 2
        stosw
        jmp     copy_hardwarekey

; -----------------------------------------------------------   计算注册码      
get_hardwarekey:        mov     hardware_key_size, cx          
        mov     eax, offset static_string
        mov     ptr1, eax
        mov     eax, offset hardware_key
        mov     ptr2, eax          
        push    offset ptr1                   ;压入 UNICODE 字符串"Serializer"
        push    offset ptr2                   ;压入机器码          
        call    loc_4033BD                   ;算法 CALL           
        mov     esi, eax                      ;得到注册码
        mov     edi, offset SerialBuffer

; -----------------------------------------------------------   将注册码复制到 SerialBuffer 供注册码框调用    
get_SerialBuffer:       lodsw
        test    ax, ax
        jz      exit
        stosw
        jmp     get_SerialBuffer

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> 以下是算法 CALL 内容，拷自 IDA
loc_4033BD:
    push  ebp
    mov   ebp, esp
    sub   esp, 0Ch
    push  offset  dummy_seh
    mov   eax,  fs:0
    push  eax
    mov   fs:0, esp
    sub   esp, 64h
    push  ebx
    push  esi
    push  edi
    mov   [ebp-0Ch], esp
    mov   dword ptr [ebp-8], offset dword_401160
    mov   edi, [ebp+0Ch]
    push  1
    pop   esi
    xor   eax, eax
    push  dword ptr [edi]
    mov   [ebp-20h], eax
    mov   [ebp-24h], eax
    mov   [ebp-28h], eax
    mov   [ebp-2Ch], eax
    mov   [ebp-38h], eax
    mov   [ebp-3Ch], eax
    mov   [ebp-40h], eax
    mov   [ebp-50h], eax
    mov   [ebp-60h], eax
    mov   [ebp-1Ch], esi
    cinvoke  __vbaLenBstr
    push  2
    mov   [ebp-68h], eax
    mov   [ebp-18h], esi
    pop   ebx

; ---------------------------------------------------------------------------
loc_40341C:        ; CODE XREF: seg000:00403491j
    mov   eax, [ebp-18h]
    cmp   eax, [ebp-68h]
    jg    short loc_403493
    push  dword ptr [ebp-24h]
    lea   ecx, [ebp-50h]
    mov   [ebp-48h], esi
    mov   [ebp-50h], ebx
    push  ecx
    push  eax
    push  dword ptr [edi]
    cinvoke  rtcMidCharBstr
    mov   edx, eax
    lea   ecx, [ebp-3Ch]
    cinvoke  __vbaStrMove
    push  eax
    cinvoke  rtcAnsiValueBstr
    push  eax
    cinvoke  __vbaStrI2
    mov   edx, eax
    lea   ecx, [ebp-40h]
    cinvoke  __vbaStrMove
    push  eax
    cinvoke  __vbaStrCat
    mov   edx, eax
    lea   ecx, [ebp-24h]
    cinvoke  __vbaStrMove
    lea   eax, [ebp-40h]
    push  eax
    lea   eax, [ebp-3Ch]
    push  eax
    push  ebx
    cinvoke  __vbaFreeStrList
    add   esp, 0Ch
    lea   ecx, [ebp-50h]
    cinvoke  __vbaFreeVar
    push  1
    pop   eax
    add   eax, [ebp-18h]
    jo    loc_403616
    mov   [ebp-18h], eax
    jmp   short loc_40341C

; ---------------------------------------------------------------------------
loc_403493:        ; CODE XREF: seg000:00403422j
    mov   eax, [ebp+8]
    push  dword ptr [eax]
    cinvoke  __vbaLenBstr
    mov   [ebp-70h], eax
    mov   [ebp-18h], esi

; ---------------------------------------------------------------------------
loc_4034A3:        ; CODE XREF: seg000:004035A9j
    mov   edi, [ebp-18h]
    cmp   edi, [ebp-70h]
    jg    loc_4035AE
    lea   eax, [ebp-50h]
    mov   [ebp-48h], esi
    push  eax
    mov   [ebp-50h], ebx
    push  dword ptr [ebp-1Ch]
    push  dword ptr [ebp-24h]
    cinvoke  rtcMidCharBstr
    mov   edx, eax
    lea   ecx, [ebp-3Ch]
    cinvoke  __vbaStrMove
    push  eax
    cinvoke  rtcAnsiValueBstr
    movsx eax, ax
    lea   ecx, [ebp-3Ch]
    mov   [ebp-30h], eax
    cinvoke  __vbaFreeStr
    lea   ecx, [ebp-50h]
    cinvoke  __vbaFreeVar
    mov   eax, [ebp-1Ch]
    push  dword ptr [ebp-24h]
    add   eax, esi
    jo    loc_403616
    mov   [ebp-1Ch], eax
    cinvoke  __vbaLenBstr
    cmp   [ebp-1Ch], eax
    jle   short loc_403508
    mov   [ebp-1Ch], esi

; ---------------------------------------------------------------------------
loc_403508:        ; CODE XREF: seg000:00403503j
    lea   eax, [ebp-50h]
    mov   [ebp-48h], esi
    push  eax
    mov   eax, [ebp+8]
    push  edi
    mov   [ebp-50h], ebx
    push  dword ptr [eax]
    cinvoke  rtcMidCharBstr
    mov   edx, eax
    lea   ecx, [ebp-3Ch]
```



```asm
    cinvoke   __vbaStrMove  

    push   eax  

    cinvoke   rtcAnsiValueBstr  

    lea   ecx, [ebp-3Ch]  

    movsx   edi, ax  

    cinvoke   __vbaFreeStr  

    lea   ecx, [ebp-50h]  

    cinvoke   __vbaFreeVar  

    xor   edi, [ebp-30h]  

    lea   eax, [ebp-38h]  

    mov   [ebp-58h], eax  

    lea   eax, [ebp-60h]  

    push   eax  

    mov   [ebp-38h], edi  

    mov   dword ptr [ebp-60h], 4003h  

    cinvoke   rtcHexBstrFromVar  

    mov   edx, eax  

    lea   ecx, [ebp-28h]  

    cinvoke   __vbaStrMove  

    push   dword ptr [ebp-28h]  

    cinvoke   __vbaLenBstr  

    cmp   eax, ebx  

    jge   short loc_403589  

    push   offset dword_4027BC  

    push   dword ptr [ebp-28h]  

    cinvoke   __vbaStrCat  

    mov   edx, eax  

    lea   ecx, [ebp-28h]  

    cinvoke   __vbaStrMove  

; ---------------------------------------------------------------------------  

loc_403589:        ; CODE XREF: seg000:00403570j  

    push   dword ptr [ebp-2Ch]  

    push   dword ptr [ebp-28h]  

    cinvoke   __vbaStrCat  

    mov   edx, eax  

    lea   ecx, [ebp-2Ch]  

    cinvoke   __vbaStrMove  

    push   1  

    pop   eax  

    add   eax, [ebp-18h]  

    jo   short loc_403616  

    mov   [ebp-18h], eax  

    jmp   loc_4034A3  

; ---------------------------------------------------------------------------  

loc_4035AE:        ; CODE XREF: seg000:004034A9j  

    mov   edx, [ebp-2Ch]  

    lea   ecx, [ebp-20h]  

    cinvoke   __vbaStrCopy  

    push   offset loc_403602  

    jmp   short loc_4035E9  

; ---------------------------------------------------------------------------  

    test   byte ptr [ebp-4], 4  

    jz   short loc_4035CE  

    lea   ecx, [ebp-20h]  

    cinvoke   __vbaFreeStr  

; ---------------------------------------------------------------------------  

loc_4035CE:        ; CODE XREF: seg000:004035C4j  

    lea   eax, [ebp-40h]  

    push   eax  

    lea   eax, [ebp-3Ch]  

    push   eax  

    push   2  

    cinvoke   __vbaFreeStrList  

    add   esp, 0Ch  

    lea   ecx, [ebp-50h]  

    cinvoke   __vbaFreeVar  

    retn  

; ---------------------------------------------------------------------------    

loc_403616:        ; CODE XREF: seg000:00403488j  

      ; seg000:004034F2j ...  

    cinvoke   __vbaErrorOverflow    

; ---------------------------------------------------------------------------        

loc_4035E9:        ; CODE XREF: seg000:004035BEj  

    lea   ecx, [ebp-24h]  

    cinvoke   __vbaFreeStr  

    lea   ecx, [ebp-28h]  

    cinvoke   __vbaFreeStr  

    lea   ecx, [ebp-2Ch]  

    cinvoke   __vbaFreeStr  

    retn  

; ---------------------------------------------------------------------------  

loc_403602:        ; DATA XREF: seg000:004035B9o  

    mov   ecx, [ebp-14h]  

    mov   eax, [ebp-20h]  

    pop   edi  

    pop   esi  

    mov   fs:0, ecx  

    pop   ebx  

    leave  

    retn  8  

; ---------------------------------------------------------------------------  

dummy_seh:      xor     eax, eax  

                ret  

; ---------------------------------------------------------------------------  

exit:                              

                ret           

Generate endp      


```

完整的代码及相关软件看`附件`。下篇想写个`VC`类的，如果有时间。

```
--------------------------------------------------------------------------------
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

                                                  2006 年 12 月 14 日

 4dd466d0e1e0cc123afbe504da2462ca
```
