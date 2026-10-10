

# EXECryptor2.3.9 主程序脱壳【分享】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-607.htm`](https://www.pediy.com/kssd/pediy08/pediy8-607.htm)

一、破解目标：EXECryptor2.3.9 主程序  
  
二、破解工具：OllyDbg v1.10，ImportREC 1.6 Final，LordPE  
  
三、破解作者：DarkBull#126.com  
  
四、破解过程：  
  
1.寻找 OEP  
  
首先设置 OD 停在系统断点，隐藏 OD，忽略所有异常，运行以下脚本：  
  
data:  
    var hInstance  
    var codeseg  
    var vmseg  
    var ep  
    var oep  
    var temp  
code:  
    gpa "VirtualFree","kernel32.dll"  
    bphws $RESULT,"x"  
    run  
    bphwc $RESULT  
    rtu  
    gmi eip,MODULEBASE  
    mov hInstance,$RESULT  
    mov temp,$RESULT  
    add temp,3c  
    mov temp,\[temp\]  
    add temp,hInstance  
    add temp,28  
    mov temp,\[temp\]  
    add temp,hInstance  
    bc temp  
    mov ep,temp  
    gmemi eip,MEMORYBASE  
    mov codeseg,$RESULT  
    find $RESULT,#2ECC9D#  
    mov \[$RESULT\],#2ECC90#  
    gpa "EnumWindows","user32.dll"  
    mov \[$RESULT\],#8BC09C85C09D0578563412C20800#  
    gpa "CreateThread","kernel32.dll"  
    find $RESULT,#FF7518#  
    mov \[$RESULT\],#6A0490#  
    gpa "ZwCreateThread","ntdll.dll"  
    bp $RESULT  
loop1:  
    run  
    cmp eip,$RESULT  
    jne loop1  
    bc $RESULT  
    bp ep  
loop2:  
    run  
    cmp eip,ep  
    jne loop2  
    bc ep  
    mov temp,codeseg  
    sub temp,1  
    gmemi temp,MEMORYBASE  
    mov vmseg,$RESULT  
    gmemi temp,MEMORYSIZE  
    bprm vmseg,$RESULT  
    run  
    bpmc  
    mov oep,eax  
    sti  
    bprm oep,1  
loop3:  
    run  
    cmp eip,oep  
    jne loop3  
    bpmc  
    ret  
  
2.修复 IAT  
  
通过观察，可以确定 IAT 的起始地址为：004DD168，结束地址为：004DD988。IAT 修复脚本如下：  
data:  
    var base  
    var size  
    var iats  
    var iate  
    var fun  
    var cnt  
code:  
    gmi eip,MODULEBASE  
    mov base,$RESULT  
    gmi eip,MODULESIZE  
    mov size,$RESULT  
    add size,base  
    mov iats,4DD168          
    mov iate,4DD988  
    exec  
    push 004d70f0  
    push 004d70a0  
    push 004d7050  
    push 004d7000  
    ende  
loop1:  
    mov fun,\[iats\]  
    cmp fun,base  
    jb next  
    cmp fun,size  
    ja next  
    mov eip,fun  
    mov esp,0012ffb4  
    bphws iats,"w"  
    run  
    gn \[iats\]  
    cmp $RESULT,0  
    je pause1  
    bphwc iats  
    inc cnt  
    jmp next  
pause1:  
    pause                                                    ; 手动修复  
    bphwc iats  
next:  
    add iats,4  
    cmp iats,iate  
    ja end  
    jmp loop1  
end:  
    eval "Already Found {cnt} Function!"  
    msg $RESULT  
    ret  
  
3.跨平台  
清除已初始化的数据，修复方法基本同 EXECr2.2.6，大概修复 40 多处。  
以上脚本及脱壳程序在 WINXP 和 WIN2003 下测试通过。  
  
  
                2006.05.14 

 4dd466d0e1e0cc123afbe504da2462ca