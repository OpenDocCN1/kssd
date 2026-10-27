
```
.text:00407631 ; =============== S U B R O U T I N E =======================================
.text:00407631
.text:00407631
.text:00407631 _KiSystemService proc near ; CODE XREF: ZwAcceptConnectPort(x,x,x,x,x,x)+Cp
.text:00407631 ; ZwAccessCheck(x,x,x,x,x,x,x,x)+Cp ...
.text:00407631
.text:00407631 var_104 = dword ptr -104h
.text:00407631 var_100 = dword ptr -100h
.text:00407631 var_D0 = dword ptr -0D0h
.text:00407631 var_CC = dword ptr -0CCh
.text:00407631 var_C8 = dword ptr -0C8h
.text:00407631 var_B0 = dword ptr -0B0h
.text:00407631 var_AC = dword ptr -0ACh
.text:00407631 var_A8 = dword ptr -0A8h
.text:00407631 var_A3 = byte ptr -0A3h
.text:00407631 var_73 = byte ptr -73h
.text:00407631 arg_0 = dword ptr 4
.text:00407631 arg_64 = dword ptr 68h
.text:00407631 arg_69 = byte ptr 6Dh
.text:00407631
.text:00407631 push 0 ; _KTRAP_FRAME 0x064 ErrCode 错误码置为0；
.text:00407633 push ebp ; 保存3环寄存器的值
.text:00407634 push ebx
.text:00407635 push esi
.text:00407636 push edi
.text:00407637 push fs
.text:00407639 mov ebx, 30h ;30h段选择子 0011 0000：索引为6，GDT表，运行级别为0环
.text:0040763E mov fs, bx ; 设置 fs 为 0x30
.text:0040763E ; 根据段选择子查GDT表得到对应的段描述符 ffc093df`f0000001
.text:0040763E ; fs.base = ffdff000，指向当前CPU的KPCR结构
.text:00407640 assume fs:nothing
.text:00407640 push dword ptr ds:0FFDFF000h ; 保存旧的 ExceptionList，然后把新的清成-1，_KPCR+0X00->_NT_TIB->ExecotionList
.text:00407646 mov dword ptr ds:0FFDFF000h, 0FFFFFFFFh
.text:00407650 mov esi, ds:0FFDFF124h ; esi 指向 CurrentThread(0FFDFF124h:_KPCR+0x124=_KPCR+0x120+0x004->_KPRCB+0X004->CurrentThread)
.text:00407656 push dword ptr [esi+140h] ; 保存 CurrentThread.PreviousMode
.text:00407656 ; PreviousMode = 0 表示从0环调用过来
.text:00407656 ; PreviousMode != 0 表示从3环调用过来
.text:0040765C sub esp, 48h ; esp 指向 _KTRAP_FRAME
.text:0040765F mov ebx, [esp+68h+arg_0];arg_0:4,[esp+6c]->Segcs(三环原来的cs的值)
.text:00407663 and ebx, 1 ;0环最低位为0，3环最低位为1
.text:00407666 mov [esi+140h], bl ; 旧CS 与 1 的结果存入 PreviousMode
.text:0040766C mov ebp, esp ; ebp 指向 _KTRAP_FRAME
.text:0040766E mov ebx, [esi+134h] ;_ETHREAD+0x00+0x134->_KTHERAD+0x134-> TrapFrame
.text:00407674 mov [ebp+3Ch], ebx ; [ebp+3Ch]：_KTRAP_FRAME.Edx ， ebx是指向原来的TrapFrame
.text:00407677 mov [esi+134h], ebp ;_ETHREAD+0x00+0x134->_KTHERAD+0x134-> TrapFrame 指向当前 _KTRAP_FRAME
.text:0040767D cld ; df = 0
.text:0040767E mov ebx, [ebp+60h] ;[ebp+60h]:_KTRAP_FRAME.EBP(3环ebp) ,ebp 指向 _KTRAP_FRAME
.text:00407681 mov edi, [ebp+68h] ;[ebp+68h]:_KTRAP_FRAME.EIP(3环eip)
.text:00407684 mov [ebp+0Ch], edx ; _KTRAP_FRAME.DbgArgPointer = edx
.text:00407684 ; 这一步是保存3环API参数指针
.text:00407687 mov dword ptr [ebp+8], 0BADB0D00h; _KTRAP_FRAME.DbgArgMark
.text:0040768E mov [ebp+0], ebx ; _KTRAP_FRAME.DbgEbp = _KTRAP_FRAME.Ebp
.text:00407691 mov [ebp+4], edi ; _KTRAP_FRAME.DbgEip = _KTRAP_FRAME.Eip
.text:00407694 test byte ptr [esi+2Ch], 0FFh;_ETHREAD+0x00+0x2c->_KTHERAD+0x2c->DebugActive ，
.text:00407698 jnz Dr_kss_a ; 测试 CurrentThread.DebugActive
.text:0040769E ; 如果处于调试状态(结果不为-1)，跳转，跳转后的代码主要是保存调试相关的寄存器(DR0-DR7)到 _KTRAP_FRAME.DR0-_KTRAP_FRAME.DR7
.text:0040769E
.text:0040769E loc_4664EF: ; CODE XREF: Dr_kss_a+10j
.text:0040769E ; Dr_kss_a+7Cj
.text:0040769E sti ; 关闭中断
.text:0040769F jmp loc_407781
.text:0040769F _KiSystemService endp
```

解析：`00407631` -- `00407637`：设置`_KTRAP_FRAME`结构体的下图中的成员

![在这里插入图片描述](upload/attach/202107/921642_WUC58G5SMDXZ2HQ.jpg)

`00407639`：`fs.base` = `ffdff000`，指向当前CPU的`KPCR`结构

![在这里插入图片描述](upload/attach/202107/921642_BB88B75JA4QXFUN.jpg)

在3环`fs`指向`TEB`结构，进入0环，`fs`指向`KPCR`结构体（通过段描述符），

`00407650`: `esi` 指向 `CurrentThread`（当前cpu所执行线程的`_ETHREAD`）

`00407656`: `push dword ptr [esi+140h]`：`_ETHREAD+0x00+0x140->_KTHERAD+0x140->PreviousMode`（原来的先前模式）

`00407640` 压栈 `ExceptionList->ExceptionList`

`00407656` 压栈 `PreviousMode->PreviousPreviousMode`

![在这里插入图片描述](upload/attach/202107/921642_PK9R43SPR7VH96U.jpg)

`0040765C`: `sub esp, 48h` ;提升48h后 `esp` 指向 `_KTRAP_FRAME`

![在这里插入图片描述](upload/attach/202107/921642_48SQCC67M866Z9P.jpg)

`00407698` `jnz Dr_kss_a`;保存调试相关的寄存器(`DR0-DR7`)到 `_KTRAP_FRAME.DR0-_KTRAP_FRAME.DR7`

```
.text:0040752C Dr_kss_a proc near ; CODE XREF: _KiSystemService+67↓j
.text:0040752C test dword ptr [ebp+70h], 20000h
.text:00407533 jnz short loc_407542
.text:00407535 test dword ptr [ebp+6Ch], 1
.text:0040753C jz loc_40769E
.text:00407542
.text:00407542 loc_407542: ; CODE XREF: Dr_kss_a+7↑j
.text:00407542 mov ebx, dr0
.text:00407545 mov ecx, dr1
.text:00407548 mov edi, dr2
.text:0040754B mov [ebp+18h], ebx
.text:0040754E mov [ebp+1Ch], ecx
.text:00407551 mov [ebp+20h], edi
.text:00407554 mov ebx, dr3
.text:00407557 mov ecx, dr6
.text:0040755A mov edi, dr7
.text:0040755D mov [ebp+24h], ebx
.text:00407560 mov [ebp+28h], ecx
.text:00407563 xor ebx, ebx
.text:00407565 mov [ebp+2Ch], edi
.text:00407568 mov dr7, ebx
.text:0040756B mov edi, large fs:20h
.text:00407572 mov ebx, [edi+2F8h]
.text:00407578 mov ecx, [edi+2FCh]
.text:0040757E mov dr0, ebx
.text:00407581 mov dr1, ecx
.text:00407584 mov ebx, [edi+300h]
.text:0040758A mov ecx, [edi+304h]
.text:00407590 mov dr2, ebx
.text:00407593 mov dr3, ecx
.text:00407596 mov ebx, [edi+308h]
.text:0040759C mov ecx, [edi+30Ch]
.text:004075A2 mov dr6, ebx
.text:004075A5 mov dr7, ecx
.text:004075A8 jmp loc_40769E
.text:004075A8 Dr_kss_a endp
.text:004075A8
.text:004075A8 ; ---------------------------------------------------------------------------
.text:004075AD align 10h
.text:004075B0
.text:004075B0 ; =============== S U B R O U T I N E =======================================
```

`0040769F` `jmp loc_407781`;可以发现跳转到了`_KiFastCallEntry`函数内部（不同的方式保存寄存器，但是最后执行的代码是一样的），因为`_KiIntSystemCall`与`KiFastSystemCall`对于寄存器的处理不同，所以需要使用`_KiFastCallEntry`，`KiSystemService`两个函数填充`_KTRAP_FRAME`结构体。
