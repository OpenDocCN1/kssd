
# 一个简单的加 vmp1.63 的 crackme 正向追踪

> 原文：[`www.pediy.com/kssd/pediy10/69889.html`](https://www.pediy.com/kssd/pediy10/69889.html)

今天儿子较乖，让我能静下心来多打会电脑。

早就想试下 `heXer` 的 `key` 了，没找到 `vmp1.64`，只好装了个 `1.63`，竟然也注册成功，感受了 `key` 的强大。

之前跟过一个海风提供的用 `vmp1.60` 的 `notepad`，变形很多，`vmopcodetable` 也加密了，虽然 `rokyxue` 给出了简化方法，不过还是不敢跟进去，怕浪费太多的时间和精力。手工怎能和机器对抗？

**目的：** 希望能正向分析一个自己设计的加了 `vmp` 的代码，在 `vmp` 中寻找原始代码的痕迹。

根据 `rokyxue` 分析，`vmp1.63` 虽然加了很多变形，但与 `vmp1.2x` 虚拟机主体相同。确认在 `vmp` 引擎中仍保留的是 `vmp1.2x` 的 `opcode` 结构，同时 `esi` 为 `opcode` 指针。`ecx` 为 `opcode` 解密后的地址。

**过程：**

写了个最简单的明码比较的 `crackme`，在注册按钮事件里加了 `vmp` 的 `marker`。

代码:

```
#define  VMBEGIN 
 __asm     
    {   
      _emit 0xEB
      _emit 0x10   
      _emit 0x56   
      _emit 0x4D   
      _emit 0x50 
      _emit 0x72 
      _emit 0x6F 
      _emit 0x74 
      _emit 0x65 
      _emit 0x63 
      _emit 0x74 
      _emit 0x20 
      _emit 0x62 
      _emit 0x65 
      _emit 0x67 
      _emit 0x69 
      _emit 0x6E 
      _emit 0x00
    }
  namelen=GetDlgItemText(IDC_NAME,&name[0],15);
  regcodelen=GetDlgItemText(IDC_REGCODE,&regcode[0],4095);

  if (namelen==0||namelen>15||regcodelen!=16)
    return;
   
  strupr(&regcode[0]);//
  for(tmpi=0;tmpi<regcodelen;tmpi++)
  {
    if (regcode[tmpi]>0x39||regcode[tmpi]<0x30)
      if(regcode[tmpi]>'F'||regcode[tmpi]<'A')
        return;
  }
  if(!strcmp(&name[0],"wangdell")&&!strcmp(&regcode[0],"FEDCBA9876543210"))
    MessageBox("OK!",0,MB_OK);
  else
    MessageBox("FAILED!",0,MB_OK);
#define  VMEND
    __asm 
    {   
      _emit 0xEB
      _emit 0x0E   
      _emit 0x56    
      _emit 0x4D    
      _emit 0x50 
      _emit 0x72 
      _emit 0x6F 
      _emit 0x74 
      _emit 0x65 
      _emit 0x63 
      _emit 0x74 
      _emit 0x20 
      _emit 0x65 
      _emit 0x6E 
      _emit 0x64 
      _emit 0x00
    }

```

使用 `vmp1.63` 加了一个最大速度的（`vmptest-ms.exe`)，和一个最大保护的(`vmtest-mp.exe`)

## 一、最大速度的（`vmptest-ms.exe`）

### 1、寻找虚拟机引擎

在 `vmp` 段下断点。`shift-f9` 直达 `vmp` 引擎。(`0x42d91d`)

### 2、寻找虚拟机取指令处

`f7` 单步执行直到

代码:

```
0042DD96    8B0C85 A6F64200 mov     ecx, dword ptr [eax*4+42F6A6]    ; getcodeaddr

```

`ecx` 里为加密的 `opcode` 的 `addr`

继续 `f7`，观察 `ecx` 直到其变化为一个正常的位于 `vmp` 区段地址，此时 `opcode` 地址已解密。

继续 `f7`，观察代码窗口，当发现 `retn xx` 时，停止，并加标签 `vm_execute`，如下

代码:

```
0042F35E    C2 5400         retn    54                          ; vm_excute

```

### 3、寻找两个常用 `vmopcode`(`VM_push_CT`,`VM_pop_CT`)

通常如果第一次通过 `retn` 跳转进入的就是 `VM_push_CT` 了。不过我们用个统计的方法，使用如下 `OD` 脚本

代码:

```
/*
Script written by wangdell
record opcode
20080802
//0x42e30b
*/
//test for vmprotect 1.63 release

var tmp1         
var tmp2         
var tmp3         
var tmp4         
var tmp5         
var tmp6         
var tmp7         
var tmp8         
var tmp9
var tmp10         
var imgbase

var bpVMenginejmp
var count

cmp $VERSION, "1.47"    //比较版本是否>1.47
jb odbgver
dbh        //hide od
BPHWCALL  //clear hardware breakpoint
BC        //clear software breakpoint
BPMC      //clear Memory breakpoint

mov count,0

ask_jmp:
ask "Enter EIP of VM_execute"
mov bpVMenginejmp,$RESULT
cmp bpVMenginejmp,0
je  ask_jmp
bp  $RESULT

log "VM Trace start!"
run_to_bp:
EOB  bp_record
ESTO             //step to bp(vm_execute)

bp_record:
log ecx
jmp run_to_bp

odbgver:
msg "本脚本须配合 ODbgscript 1.47 或以上的版本"
jmp end

end:
ret 

```

运行脚本，输入第 2 步确定的 `vm_execute` 地址，脚本执行几秒后，终止脚本，观察 `log` 窗口。

代码:

```
Script Log Window
Address    Message
42F35E     VM Trace start!
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042DB4B
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042D95D
42F35E     ecx: 0042F3F0
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 0042D6A5
42F35E     ecx: 00401254
42F35E     ecx: 00401254
42F35E     ecx: 0042E4BC
........

```

看到 `0042D6A5` 在开始处有 10 余条，其后 `0042D95D` 也有 10 余条，隔一条后，又有 10 余条 `0042D6A5`...

根据以往 `vmp1.2x` 的经验，`0042D6A5` 为 `VM_push_CT`，`0042D95D` 为 `VM_pop_CT`，增加标签。

### 4、寻找两个关键 `vmopcode`(`VM_JMP`,`VM_RETN`)

重新运行程序，并重新运行脚本，终止后观察脚本日志窗口

代码:

```
42F35E     VM Trace start!
......
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042DED5  //////////////////此为 VM_retn
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
........
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042D95D | offset <vmptest1.VM_pop_CT>
42F35E     ecx: 0042F3F0 | ////////////////此为 VM_jmp
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
42F35E     ecx: 0042D6A5 | offset <vmptest1.VM_push_CT>
......

```

在日志窗中寻找上面的代码片段，即 `VM_push_CT` n 和 `VM_pop_CT` n。

在 `VM_pop_CT` n 和 `VM_push_CT` n 通常只隔有 1 条指令，搜索所有的这样代码段

只有 2 条指令符合这一要求，他们就是 `VM_JMP` 和 `VM_RETN`。

它们的区分方法是，分别设断后，跟进去，很明显其中一个有 `esi` 的赋值，此为 `VM_jmp`。另一个则在指令执行结尾处有 `retn xx`。

### 5、追踪比对原始代码

去除所有断点，然后分别在 `VM_jmp` 开始处设断，在 `VM_retn` 结尾处设断(也就是 `retn xx`)

代码:



```
0042F3F0 > 8D3455 4A022F33 lea esi, dword ptr [edx*2+332F024A]
```

代码：

```
0042E9A0    C2 3C00          retn    3C
```

重新`f9`运行，在断点处观察堆栈和寄存器，可观察到一些原始代码的痕迹。

## 二、加最大保护(`vmtest-mp.exe`)

类似上面方法。

## 总结

`vmp1.63`很强大，`key`也很强大。
