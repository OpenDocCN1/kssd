

# ActiveCube Ocx

> 原文：[`www.pediy.com/kssd/pediy05/pediy50009.htm`](https://www.pediy.com/kssd/pediy05/pediy50009.htm)

破解工具:Dasm89   
下载地址:http://www.datadynamics.com   
OS:Win2000 pro   
难度:易   
  
第一次写 Crack 教程,写的不好.请各位多指点!   
  
一个 Vb,Vc 下不错的数据控件.之所以破解它,因为觉得它有实用价值.   
运行该软件给出的例子有 Nag 弹出.该控件未加壳.可直接用 Dasm89 反汇编.   
  
我们可以在对话框参考中找到如下信息   
Name: DialogID\_008E, # of Controls=007, Caption:"ActiveCube Evaluation Copy", ClassName:""   
    001 - ControlID:0001, Control Class:"BUTTON" Control Text:"OK"   
    002 - ControlID:FFFF, Control Class:"STATIC" Control Text:""   
    003 - ControlID:FFFF, Control Class:"STATIC" Control Text:"Thank you for choosing to evaluate ActiveCube from Data Dynamics, Ltd. This ve"   
    004 - ControlID:FFFF, Control Class:"STATIC" Control Text:"Copyright 2001 (c) Data Dynamics, Ltd. All Rights Reserved."   
    005 - ControlID:FFFF, Control Class:"STATIC" Control Text:"2600 Tiller Lane"   
    006 - ControlID:FFFF, Control Class:"STATIC" Control Text:"Tel : (614) 895-3142"   
    007 - ControlID:FFFF, Control Class:"STATIC" Control Text:"To license the software, or to obtain assistance during the evaluation process"   
  
就从它入手.DialgoId\_008E   
  
:1008365C 90                      nop   
:1008365D 90                      nop   
:1008365E 90                      nop   
:1008365F 90                      nop   
  
\* Referenced by a CALL at Address:   
|:1004CAD4    
|   
:10083660 56                      push esi   
:10083661 8BF1                    mov esi, ecx   
  
\* Possible Reference to Dialog: DialogID\_008E   
                                  |   
  
\* Possible Reference to String Resource ID=00142: "Insufficient viewer area"   
                                  |   
:10083663 688E000000              push 0000008E    ->光标停在此处,我们向上回溯   
:10083668 E853D3F8FF              call 100109C0   
:1008366D C706081A0C10            mov dword ptr \[esi\], 100C1A08   
:10083673 8BC6                    mov eax, esi   
:10083675 5E                      pop esi   
:10083676 C3                      ret   
  
由\* Referenced by a CALL at Address::1004CAD4 回溯跟踪.    
  
:1004CAA5 8BF1                    mov esi, ecx   
:1004CAA7 3BC7                    cmp eax, edi   
:1004CAA9 8965F0                  mov dword ptr \[ebp\-10\], esp   
:1004CAAC 8975C8                  mov dword ptr \[ebp\-38\], esi   
:1004CAAF 897DFC                  mov dword ptr \[ebp\-04\], edi   
:1004CAB2 755B                    jne 1004CB0F   
:1004CAB4 397E7C                  cmp dword ptr \[esi+7C\], edi   
:1004CAB7 7556                    jne 1004CB0F   
  
\* Possible Reference to String Resource ID=00016: "Invalid dimension name"   
                                  |   
:1004CAB9 6A10                    push 00000010   
:1004CABB E8143C0600              call 100B06D4   
:1004CAC0 8BC8                    mov ecx, eax   
:1004CAC2 83C404                  add esp, 00000004   
:1004CAC5 894DCC                  mov dword ptr \[ebp\-34\], ecx   
:1004CAC8 BB01000000              mov ebx, 00000001   
:1004CACD 3BCF                    cmp ecx, edi   
:1004CACF 885DFC                  mov byte ptr \[ebp\-04\], bl   
:1004CAD2 7407                    je 1004CADB    
:1004CAD4 E8876B0300              call 10083660 '该 Call 弹出 Nag,需要跳过   
:1004CAD9 8BF8                    mov edi, eax   
  
把 1004CAD2 处的 7407 该为 7507,Nag 是没了,可该控件的表格也没了,不成功   
再向上看   
  
:1004CAB2 755B                    jne 1004CB0F   
将 755B 改为 745B(Je 1004CB0F),结果成功.   
  
该控件破解对有一定水平的人来说,很简单.但是控件的含'金'量不低.很有用的.我的报表一般都用它!   
  
感谢 Aming 给出的破解方法,非常好!对(ActiveBar2 sp3,SharpGrid 都有效).可是我到现在也不知道你的思路,是更改全局变量的值吗)能简单的说两句吗!  4dd466d0e1e0cc123afbe504da2462ca