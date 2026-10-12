
# 简单分配和释放内存脚本

> 原文：[`www.pediy.com/kssd/pediy06/pediy6717.htm`](https://www.pediy.com/kssd/pediy06/pediy6717.htm)

原文链接：http://bbs.pediy.com/showthread.php?s=&threadid=5974

``` 
/*  
                                        简单分配和释放内存脚本  
    受 playar 感染，一时冲动写了这个脚本，本脚本功能简单，建议要强大的功能还是找其它好的 OD 插件使用。本脚本功能：简单分配和释放指定的内存 Page。由于脚本提供的功能命令有限，无法精确检测你输入的数据是否合要求，但你一般规矩输入没问题的。估计适用 Ollydbg 1.10 和 OllyScript 0.92 运行平台，由于本脚本运用兼容性设计的 idea，有采用 stolen 方法（兼容了 98，NT 设计不用），所以会显得脚本比较长，CPU 窗口比较混乱时，请按“*”；NT 系统下设计会比较省事……Enjoy!若使用有 Bug  请反馈！另外关于 VirtualAlloc 的 type 参数还不熟悉，失误之处还请指出！  
*/  
```

`var askpage`

`var call`

`var input-addr`

`var input-size`

`var seip`

`var base`

`var size`

`var flag`

`var stolen1`

`var stolen2`

`var stolen3`

`var no`

`var p`

`mov no,0`

`mov flag,0`

`mov seip,eip`

`mov p,eip`

`mov stolen1,[p]`

`add p,4`

`mov stolen2,[p]`

`add p,4`

`mov stolen3,[p]`

`mov [eip],#606A00E80000000061#`

`mov p,eip`

`add p,3`

`asm p,"call kernel32.GetModuleHandleA"`

`add p,5`

`go p`

`cmp eax,0`

`je error`

`mov flag,1`

`mov base,eax`

`log base`

`sto`

`return:`

`mov p,seip`

`mov eip,p`

`mov [p],stolen1`

`add p,4`

`mov [p],stolen2`

`add p,4`

`mov [p],stolen3`

`cmp flag,1`

`je begin`

`msg "不明获取错误！"`

`ret`

`begin:`

`ask "请输入功能选择：0-退出，1-分配内存，2-释放内存"`

`cmp $RESULT,1`

`je Allocate_space`

`cmp $RESULT,2`

`je Release_space`

`jmp exit`

`Allocate_space:`

`find base,#000000000000000000000000#`

`cmp $RESULT,0`

`je error`

`mov p,$RESULT`

`log p`

`mov eip,p`

`mov call,p`

`mov [p],#6050515253E80000000061#  // VirtualAlloc`

`add call,5`

`asm call,"call KERNEL32.VirtualAlloc"`

`sto`

`mov no,1`

`ask "分配内存功能 - 输入申请 Size（16 进制值）:"`

`mov edx,$RESULT`

`mov input-size,edx`

`mov size,edx  //Size`

`mov eax,40  //flProtect`

`mov ecx,1000  //Type`

`ask "分配内存功能 - 请输入申请页地址（16 进制值，0 为自动）:"`

`mov ebx,$RESULT  //lpAddress`

`mov input-addr,ebx`

`add call,5`

`log "下面是申请输入信息："`

`log input-addr`

`log input-size`

`msgyn "你输入参数已经登记在 Log 窗口，继续执行点'Y'，暂停点'N'。"`

`cmp $RESULT,1`

`je continue`

`pause`

`continue:`

`go call`

`cmp eax,0`

`je error`

`mov askpage,eax`

`log "下面是成功申请信息："`

`log askpage`

`add size,0FFF`

`and size,0FFFFF000`

`log size`

`sto`

`mov eip,seip`

`mov [p],#000000000000000000000000#`

`msg "Successed get need size page!Look log Windows!"`

`mov [askpage],"I successed to get This Page!"`

`cmt eip,"Look the page top,we wrote my flag!"`

`msg "请求分配内存任务成功！"`

`ret`

`Release_space:`

`find base,#000000000000000000000000#`

`cmp $RESULT,0`

`je error`

`mov p,$RESULT`

`log p`

`mov eip,p`

`mov call,p`

`mov [p],#60505152E80000000061#  //VirtualFree`

`add call,4`

`asm call,"call KERNEL32.VirtualFree"`

`sto`

`mov no,1`

`ask "释放内存功能 - 输入释放页地址:（16 进制值）"`

`cmp $RESULT,0`

`je error`

`mov askpage,$RESULT`

`log "下面是输入请求释放页："`

`log askpage`

`mov eax,8000  // Type`

`mov ecx,0  // Size`

`mov edx,askpage  //lpAddress`

`add call,5`

`go call`

`mov no,eax`

`sto`

`mov eip,seip`

`mov [p],#000000000000000000000000#`

`cmp no,1`

`jne waa`

`log "下面是成功释放页："`

`log askpage`

`cmt eip,"若有需要请看 Log 窗口"`

`msg "请求释放内存任务成功！"`

`ret`

`error:`

`sto`

`cmp flag,1`

`jne return`

`cmp no,1`

`jne waa`

`mov eip,seip`

`mov [p],#000000000000000000000000#`

`waa:`

`msg "执行任务失败！"`

`ret`

`exit:`

`msg "No Mission!"`

`ret`

///////////////////////////////////

修复了一点错误检测！

to somebody:

NT 系统的确可以很简单写（`exec/ende`），98 你能用得了，现在玩兼容性，萝卜青菜，各有所爱，有时做些复杂的事会有一定的体会！比如：某某要 BT 跟壳，偶做不了这妖事;) 

4dd466d0e1e0cc123afbe504da2462ca
