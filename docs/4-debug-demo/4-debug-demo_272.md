

# SmartDraw 5.03

> 原文：[`www.pediy.com/kssd/pediy02/forum283.htm`](https://www.pediy.com/kssd/pediy02/forum283.htm)

 SmartDraw 5.03   
http://www.newhua.com/smartdraw.htm   
绘制流程图和其他各种图表的工具软件，功能强大，简单易用。   
很好的软件，可惜有时间限制，各位大哥帮忙破解一下吧。  

  

 SmartDraw5 时间过期破解   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0041CA71(C)   
|   
:0041CA8C E89E080000              call 0041D32F   
:0041CA91 85C0                    test eax, eax   
:0041CA93 7423                    jnz 0041CAB8      //改为  jmp 0041CAB8   
:0041CA95 6830200000              push 00002030   
  
\* Possible Reference to String Resource ID=00041: "This trial version of SmartDraw has now expired. You may dow"   
                                  |   
:0041CA9A 6A29                    push 00000029   
:0041CA9C E8E0B2FEFF              call 00407D81   
:0041CAA1 83C408                  add esp, 00000008   
  
\* Possible Ref to Menu: MDMENU, Item: "Object"   
                                  |   
:0041CAA4 C705A00D590001000000    mov dword ptr \[00590DA0\], 00000001   
  
\* Possible Ref to Menu: MDMENU, Item: "Object"   
                                  |   
:0041CAAE C705A80D590001000000    mov dword ptr \[00590DA8\], 00000001  //设过期标志   
  
l    Referenced by a (U)nconditional or (C)onditional Jump at Address:   
  
查："This Trial Version License HAS EXPIRED!"   
找到判断过期标志的依据如下：   
Name:            CANTSAVE, # of Controls=010, Caption:"SmartDraw Trial Edition License Expired!", ClassName:""  // 注意这个 CANTSAVE    
    001 - ControlID:0006, Control Class:"BUTTON" Control Text:"No"   
    002 - ControlID:0009, Control Class:"BUTTON" Control Text:"&Purchase"   
    003 - ControlID:0007, Control Class:"BUTTON" Control Text:"&Continue"   
    004 - ControlID:FFFF, Control Class:"STATIC" Control Text:""   
    005 - ControlID:FFFF, Control Class:"STATIC" Control Text:"This Trial Version License HAS EXPIRED!"   
    006 - ControlID:FFFF, Control Class:"BUTTON" Control Text:""   
    007 - ControlID:FFFF, Control Class:"STATIC" Control Text:"YOU ARE NO LONGER ABLE TO SAVE DOCUMENTS"   
    008 - ControlID:1389, Control Class:"STATIC" Control Text:"15"   
    009 - ControlID:040C, Control Class:"BUTTON" Control Text:"Dummy"   
    010 - ControlID:FFFF, Control Class:"STATIC" Control Text:"To purchase SmartDraw, press the Purchase button."   
以下是由 CANTSAVE 所查到   
／／／//不能另存为提示   
  
\* Referenced by a CALL at Address:   
|:004099EE    
|   
:005356E4 55                      push ebp   
:005356E5 8BEC                    mov ebp, esp   
:005356E7 81EC48020000            sub esp, 00000248   
:005356ED C785C0FDFFFF00000000    mov dword ptr \[ebp+FFFFFDC0\], 00000000   
:005356F7 C785E8FEFFFF00000000    mov dword ptr \[ebp+FFFFFEE8\], 00000000   
:00535701 C745F400000000          mov \[ebp\-0C\], 00000000   
:00535708 833DA80D590000          cmp dword ptr \[00590DA8\], 00000000      //比较   
:0053570F 741A                    je 0053572B   
  
\* Possible Reference to String Resource ID=00002: "There is no active printer selected. Choose Page Setup from "   
                                  |   
:00535711 6A02                    push 00000002   
  
\* Possible StringData Ref from Data Obj ->"CANTSAVE"    //不能另存为提示    
                                  |   
:00535713 6800F25800              push 0058F200   
:00535718 A1201D5900              mov eax, dword ptr \[00591D20\]   
:0053571D 50                      push eax   
:0053571E E8F28F0000              call 0053E715   
:00535723 83C40C                  add esp, 0000000C   
:00535726 E94F030000              jmp 00535A7A   
  
  
／／//不能存为模板提示   
  
\* Referenced by a CALL at Address:   
|:004099F8    
|   
:00535BF4 55                      push ebp   
:00535BF5 8BEC                    mov ebp, esp   
:00535BF7 81EC38020000            sub esp, 00000238   
:00535BFD C785C8FDFFFF00000000    mov dword ptr \[ebp+FFFFFDC8\], 00000000   
:00535C07 833DA80D590000          cmp dword ptr \[00590DA8\], 00000000    //比较   
:00535C0E 741A                    je 00535C2A   
  
\* Possible Reference to String Resource ID=00002: "There is no active printer selected. Choose Page Setup from "   
                                  |   
:00535C10 6A02                    push 00000002   
  
\* Possible StringData Ref from Data Obj ->"CANTSAVE"      //不能保存为模板提示    
                                  |   
:00535C12 6814F25800              push 0058F214   
:00535C17 A1201D5900              mov eax, dword ptr \[00591D20\]   
:00535C1C 50                      push eax   
:00535C1D E8F38A0000              call 0053E715   
:00535C22 83C40C                  add esp, 0000000C   
:00535C25 E9DE010000              jmp 00535E08   
  
  
／／//不能保存文件提示   
\* Referenced by a CALL at Addresses:   
|:004099E1  , :005362C8  , :0053647E  , :00546B5B    
|   
:00535E0C 55                      push ebp   
:00535E0D 8BEC                    mov ebp, esp   
:00535E0F 81EC20010000            sub esp, 00000120   
:00535E15 C785E8FEFFFF00000000    mov dword ptr \[ebp+FFFFFEE8\], 00000000   
:00535E1F 833DA80D590000          cmp dword ptr \[00590DA8\], 00000000  //比较   
:00535E26 741C                    je 00535E44   
  
\* Possible Reference to String Resource ID=00002: "There is no active printer selected. Choose Page Setup from "   
                                  |   
:00535E28 6A02                    push 00000002   
  
\* Possible StringData Ref from Data Obj ->"CANTSAVE"  //不能保存文件提示   
  
                                  |   
:00535E2A 6820F25800              push 0058F220   
:00535E2F A1201D5900              mov eax, dword ptr \[00591D20\]   
:00535E34 50                      push eax   
  
由上可知\[00590DA8\]为过期标志   
  
修改方法：   
查 e8 9e 08 00 00 85 c0 74 23  改为：e8 9e 08 00 00 85 c0 eb 23   
  
查 c7 05 a8 0d 59 00 01 00 00 00 替换为：c7 05 a8 0d 59 00 00 00 00 00  （共６处）   
  
实际还有其它方法可解决问题，在此不列出了。   4dd466d0e1e0cc123afbe504da2462ca