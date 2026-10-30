
```text
WARNING: Stack unwind information not available. Following frames may be wrong.

0013db30 2d846775 Multimedia!PlugInMain+0x1640

0013dbf0 2d829d0c Multimedia!PlugInMain+0x451c7

0013dcb0 23806896 Multimedia!PlugInMain+0x2875e

0013dcbc 23806869 EScript!PlugInMain+0x54e6

0013dcd4 2382afde EScript!PlugInMain+0x54b9

0013dce0 2382afe6 EScript!PlugInMain+0x29c2e

0013dcf4 23803e9b EScript!PlugInMain+0x29c36

00000000 00000000 EScript!PlugInMain+0x2aeb
```

### (b)

```text
free :02b75958

 ChildEBP RetAddr

0013d274 78134c39 ntdll!RtlFreeHeap

0013d2c0 2d835f82 MSVCR80!free+0xcd [f:\sp\vctools\crt\_bld\self\_x86\crt\src\free.c @ 110]

WARNING: Stack unwind information not available. Following frames may be wrong.

0013db30 2d846775 Multimedia!PlugInMain+0x349d4

0013dbf0 2d829d0c Multimedia!PlugInMain+0x451c7

0013dcb0 23806896 Multimedia!PlugInMain+0x2875e

0013dcbc 23806869 EScript!PlugInMain+0x54e6

0013dcd4 2382afde EScript!PlugInMain+0x54b9

0013dce0 2382afe6 EScript!PlugInMain+0x29c2e

0013dcf4 23803e9b EScript!PlugInMain+0x29c36

00000000 00000000 EScript!PlugInMain+0x2aeb
```

哈哈，这次它的状态是`free`。这次和CVE所说的`use after free`是相吻合的。

现在只剩下一对分配和释放操作了，显然上次的“不合理”，是用以控制内存数据的函数`util.printd`所导致的。

通过观察（a）和（b）的调用栈信息，他们下边的部分是完全相同，这说明这两个操作应该都在一个函数中发生。

根据返回地址`2d846775`，我们找到这里：

```asm
.text:2D846770                 call    sub_2D846602
.text:2D846775                 add     esp, 14h
.text:2D846778                 push    [ebp+arg_C]
```

跟进`sub_2D846602`：

```asm
.text:2D846602 sub_2D846602    proc near               ; CODE XREF: sub_2D846674+FCp
.text:2D846602

.text:2D846602 var_18          = dword ptr -18h
.text:2D846602 var_14          = dword ptr -14h
.text:2D846602 var_4           = dword ptr -4
.text:2D846602 arg_0           = dword ptr  8
.text:2D846602 arg_4           = dword ptr  0Ch
.text:2D846602 arg_8           = dword ptr  10h
.text:2D846602 arg_C           = dword ptr  14h
.text:2D846602 arg_10          = dword ptr  18h

.text:2D846602                 push    0Ch
.text:2D846604                 mov     eax, offset loc_2D8BA0F3
.text:2D846609                 call    __EH_prolog3_catch
.text:2D84660E                 mov     esi, [ebp+arg_4]
.text:2D846611                 mov     eax, [esi]
.text:2D846613                 mov     ecx, [eax+4]
.text:2D846616                 mov     ecx, [ecx+4]
.text:2D846619                 lea     ecx, [ecx+eax+4]
.text:2D84661D                 mov     eax, [ecx]
.text:2D84661F                 push    4Ch
.text:2D846621                 call    dword ptr [eax+0Ch] ;分配内存
.text:2D846624                 and     [ebp+var_4], 0
.text:2D846628                 mov     [ebp+var_14], eax
.text:2D84662B                 mov     [ebp+var_18], eax
.text:2D84662E                 test    eax, eax
.text:2D846630                 mov     byte ptr [ebp+var_4], 1
.text:2D846634                 jz      short loc_2D84666C
.text:2D846636                 push    [ebp+arg_10]
.text:2D846639                 mov     ecx, eax
.text:2D84663B                 push    [ebp+arg_C]
.text:2D84663E                 push    [ebp+arg_8]
.text:2D846641                 push    esi
.text:2D846642                 push    [ebp+arg_0]
.text:2D846645                 call    sub_2D8462CE
.text:2D84664A                 jmp     short loc_2D84666E
.text:2D84664C ; ---------------------------------------------------------------------------
.text:2D84664C
.text:2D84664C loc_2D84664C:                           ; DATA XREF: .rdata:2D8F2170o
.text:2D84664C                 mov     eax, [ebp+arg_4]
.text:2D84664F                 mov     eax, [eax]
.text:2D846651                 mov     ecx, [eax+4]
.text:2D846654                 mov     ecx, [ecx+4]
.text:2D846657                 push    [ebp+var_14]
.text:2D84665A                 lea     ecx, [ecx+eax+4]
.text:2D84665E                 mov     eax, [ecx]
.text:2D846660                 call    dword ptr [eax+1Ch] ;释放内存
.text:2D846663                 push    0
.text:2D846665                 push    0
.text:2D846667                 call    _CxxThrowException ;抛出异常
```

下边的函数会获得执行：

```asm
.text:2D8430FC loc_2D8430FC:                           ; DATA XREF: sub_2D8462CE+203o
.text:2D8430FC                 push    esi
.text:2D8430FD                 mov     esi, [esp+8]
.text:2D843101                 test    esi, esi
.text:2D843103                 jz      short loc_2D843127
.text:2D843105                 push    esi
.text:2D843106                 call    sub_2D842C86
.text:2D84310B                 test    eax, eax
.text:2D84310D                 pop     ecx
.text:2D84310E                 jz      short loc_2D843127
.text:2D843110                 mov     edx, [eax]
.text:2D843112                 mov     ecx, eax
.text:2D843114                 call    dword ptr [edx+4] ;夺取控制权
.text:2D843117                 push    0
.text:2D843119                 push    offset aMediaplayer_th ; "MediaPlayer_This"
.text:2D84311E                 push    esi
.text:2D84311F                 call    sub_2D81DF66
.text:2D843124                 add     esp, 0Ch
.text:2D843127
.text:2D843127 loc_2D843127:                           ; CODE XREF: .text:2D843103j
.text:2D843127                                         ; .text:2D84310Ej
.text:2D843127                 mov     ax, 1
.text:2D84312B                 pop     esi
.text:2D84312C                 retn
```

## 3. 漏洞利用

```text
object pointer   -->   fake object   -->   fake vtable     -->     fake virtual function

addr: xxxx             addr: yyyy          addr: cccc              addr: dddd

data: yyyy             data: cccc          data: +0 xxxx           data: nop slide

                                                 +4 dddd                 shellcode
```

在这里`cccc`是可控的，`cccc`必须确保`dddd`能指向我们可以用堆喷射覆盖到的内存区。

just for fun

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
