

# QSound Qmp3D V1.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50201.htm`](https://www.pediy.com/kssd/pediy05/pediy50201.htm)

Program          : QSound Qmp3D Version 1.0   
Type            : a multi-function audio enhancement plug-in for the Winamp MP3 player   
OS              : only for Win9x/2K   
Protection      : Vbox 4.3 Preview   
Homepage        : http://www.qsound.com/   
Download        :   
  
Cracked by      : zombieys\[CCG\]   
Release type    : full unpacked   
Release date    : Jul 22,2002   
Release name    : cr-dsp\_qsnd.dll (430,080 bytes)   
Release Download : http://8287.my.west163.com/cr-dsp\_qsnd.rar   
  
  
    第一次接触 dll 文件脱壳，极其不适应，花了 2 整天搞定。要感谢 ljttt 老大的指点了，大脱神呀^^   
  
  
1.找 oep   
  
    首先执行 icedump,运行 winamp,加载 dsp 插件 qsound，弹出 vbox 对话框，下 bpx getprocaddress，按 f5。点 continued 后被中断，按一下 f12 来到这里   
  
01A7:0700BB36  PUSH      DWORD PTR \[EBP\-20\]   
01A7:0700BB39  CALL      \[KERNEL32!GetProcAddress\]   
01A7:0700BB3F  MOV        EDI,EAX   
01A7:0700BB41  CMP        EDI,EBX   
01A7:0700BB43  JZ        0700BCE6   
01A7:0700BB49  MOV        ESI,\[ESI+16\]   
01A7:0700BB4C  ADD        ESI,\[EBP+08\]   
01A7:0700BB4F  CMP        \[EBP\-24\],EBX   
01A7:0700BB52  JZ        0700BB89    -〉jmp 700bb89   
  
上面代码用来控制是否破坏 it 表，所以必须要他跳过去。   
省略跟踪过成，原因是不记得怎么找到 oep 的了:(   
当来到   
  
01A7:7007f57  SETZ CL   
01A7:7007f5A  TEST EAX,EAX   
01A7:7007f5C  MOV \[EBP+08\],ECX   
01A7:7007f5F  JNZ 7007F71   
01A7:7007f61  PUSH DWORD PTR \[EBP+14\]   
01A7:7007f64  PUSH DWORD PTR \[EBP+10\]   
01A7:7007f67  PUSH DWORD PTR \[EBP+0C\]   
01A7:7007f6A  CALL EAX                  ->按 f8 进去就是程序的入口了   
01A7:7007f6C  MOV \[ESI+18\],EAX   
01A7:7007f6F  JMP 7007E75   
  
Entry Point:15480   
  
  
2.dump 程序,提取 it 表   
      
    在入口处下 a，jmp eip，用 lordpe 来 dump dsp\_qsnd.dll 进程(用 prodump 会死掉的),dump 出来的大约有 568kb。运行 ImportREC，oep 填 15480，找到 it 表后保存。   
  
  
3.修复程序(最麻烦最花时间的工作)   
  
    用 peditor 删掉除 Preview 以外的所有 sections，用 winhex 删除相应的数据只剩下大约 416kb，再用 ImportREC 添加一个 it 表。重新给 dll 文件分配 sections 并修改 Directory table 中的 Export table,Import table,Rescoures,Basereloce,Import address table 为正确的值,修改 Base of data 以及 Size of image 为正确的值，把 image base 修改成对应的值(一开始不知道要改这个，花了我好长时间, 笨呐)。   
  
4.测试正常运行 for Win9x/2K,not for Winme(程序比较老)   
  
ok 到此为止   
  
2002.7.22   
zombieys\[CCG\]/\[iPB\]  4dd466d0e1e0cc123afbe504da2462ca