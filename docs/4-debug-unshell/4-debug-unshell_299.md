

# aspack 2.000

> 原文：[`www.pediy.com/kssd/pediy02/forum-crack02.htm`](https://www.pediy.com/kssd/pediy02/forum-crack02.htm)

> 工具  : softice, FrogICE ,procdump, Process Patcher  , topo , Offset Converter 
> 
>   
> 
>        Process Patcher   到这边抓 
> 
>        http://156.17.4.138/~mwd/playtools/patchers/memory/ppatcher.zip  
> 
>         
> 
>        FrogICE   到这边抓 
> 
>        http://kontum.vietmedia.com/protools/files/debuggers/frogsice.zip 
> 
>   
> 
>        topo  到这边抓 
> 
>        http://156.17.4.138/~mwd/playtools/exe-tools/topo.zip 
> 
>   
> 
>        Offset Converter  到这边抓 
> 
>        http://kontum.vietmedia.com/protools/files/utilities/oc.zip 
> 
>           
> 
>       
> 
> 程式  : aspack 2.000 
> 
>        请到这边抓 http://kenchen.bizland.com/zip/aspack2000.zip 
> 
>   
> 
> 前言  :  这个  aspack 2.000  是一个加壳软体 , 专帮软体加壳 
> 
>         不过还是比不上  UPX,  加壳之后档案比  UPX  稍大一点 
> 
>         但也还不错用啦 . 
> 
>         有  30  天的限制 ,  没有地方输入注册码 ,  推测应该是读一个 
> 
>        keyfile,  类似  ultraedit  那样 , 程式一启动就读一个档案 
> 
>         不过我不知道是读哪一个 , 用  regmon, filemon  好像也不知道 
> 
>         看有没有人能够找到 ...  
> 
>         而且此程式本身有加壳  ,  很可惜的 ,  我脱不了壳 , 只好借助 
> 
>         其他的工具 ,  而我想出了三种破解的方法 ,  将一一展示给各位看官 
> 
>         我的  aspack.exe  大小是 231424 byte 
> 
>         若不一样大小 , 就是版本不同了 
> 
>        ,  ok, go!! 
> 
>   
> 
> 1:  安装完之后 ,  一执行  aspack,  就出现   有常驻  debugger  之类的讯息 
> 
>    hmm,  会侦测  softice,  好吧 ,  用  FrogICE  去欺骗他 ,  执行  FrogICE, 
> 
>     可执行  aspack 了 ,  关闭  aspack,  调日期 ,  往后调个一年好了 , 
> 
>     执行  aspack,  告诉我过期了  ...  将日期调回来 ,  还是不行 ... 
> 
>     那他是如何知道我把日期调回来呢  ?  是如何知道我过期呢  ? 
> 
>     我就开始猜了 ,  是纪录在  registry  吗  ?  那就执行  regedit.exe  
> 
>    ( 不会有人不知道这个档案在哪里吧  ?  在  c:\\windows  底下 ) 
> 
>     找  aspack,  结果在  \\HKEY\_CURRENT\_USER\\software\\aspack  找到了 
> 
>     干掉他吧 ,  整个目录干掉了  ...  
> 
>     执行  aspack,  没有日期的限制了  ,  这是我的第一个方法破解  aspack 
> 
>   
> 
> 2:  当然 , 这样只是没有日期的限制 ,  但他会侦测  debugger,  如  softice 
> 
>     在想办法把他干掉吧  ... 
> 
>     不过在安装一次  aspack ,  让他有日期的限制 , 
> 
>     再来就要用  softice  的  symbol loader  来载入 ,  不过一载入就跑出一个 
> 
>    MessageBox,  写著  debugger detection ,  无法载入 ... 
> 
>   
> 
> 3:  这种软体一定要载入才行 , 因为他有加壳 ,  执行  procdump,  按一下  PE Editor 
> 
>     的按钮 ,  开启  aspack.exe , 按一下  section  的按钮 , 出现  aspack.exe  的  section 
> 
>     将卷轴拉下 ,  点一下任何一个  section  , 按滑鼠右键 ,  选  Edit section 
> 
>    section character  的栏位是  C0000040,  改成  E0000020, 
> 
>     用此法 , 将所有的栏位通通改成  E0000020,  这样就可以用  symbol loader 载入了 , 
> 
>    ok,  载入之后 ,  
> 
>  :00466001  60                  PUSHAD 
> 
>  :00466002  E801000000          CALL      00466008 ;  到这边一定要按  F8,  
> 
>                                                    ;  不然就过去了 
> 
>  :00466007  90                  NOP 
> 
>  :00466008  5D                  POP       EBP 
> 
>   
> 
>   
> 
>   继续  trace,  一直按  F10,  
> 
>   
> 
>  :00466009  81EDF3C54400        SUB       EBP,0044C5F3 
> 
>  :0046600F  BBECC54400          MOV       EBX,0044C5EC 
> 
>  :00466014  03DD                ADD       EBX,EBP 
> 
>  :00466016  2B9D80D24400        SUB       EBX,\[EBP+0044D280\] 
> 
>   
> 
>  :0046601C  83BD68D1440000      CMP       DWORD PTR \[EBP+0044D168\],00 
> 
>  :00466023  899DCECE4400        MOV       \[EBP+0044CECE\],EBX 
> 
>  :00466029  0F8573090000        JNZ       004669A2 
> 
>  :0046602F  8D8570D14400        LEA       EAX,\[EBP+0044D170\] 
> 
>  :00466035  50                  PUSH      EAX 
> 
>  :00466036  FF95BCD24400        CALL      \[EBP+0044D2BC\] 
> 
>  :0046603C  89856CD14400        MOV       \[EBP+0044D16C\],EAX 
> 
>  :00466042  8BF8                MOV       EDI,EAX 
> 
>   
> 
>  :00466044  8D9D7DD14400        LEA       EBX,\[EBP+0044D17D\] 
> 
>  :0046604A  53                  PUSH      EBX 
> 
>  :0046604B  50                  PUSH      EAX 
> 
>  :0046604C  FF95B8D24400        CALL      \[EBP+0044D2B8\] 
> 
>  :00466052  898588D24400        MOV       \[EBP+0044D288\],EAX 
> 
>  :00466058  8D9D8AD14400        LEA       EBX,\[EBP+0044D18A\] 
> 
>  :0046605E  53                  PUSH      EBX 
> 
>  :0046605F  57                  PUSH      EDI 
> 
>   
> 
>  :00466060  FF95B8D24400        CALL      \[EBP+0044D2B8\] 
> 
>  :00466066  89858CD24400        MOV       \[EBP+0044D28C\],EAX 
> 
>  :0046606C  8B85CECE4400        MOV       EAX,\[EBP+0044CECE\] 
> 
>  :00466072  898568D14400        MOV       \[EBP+0044D168\],EAX 
> 
>  :00466078  6A04                PUSH      04 
> 
>  :0046607A  6800100000          PUSH      00001000 
> 
>  :0046607F  6875090000          PUSH      00000975 
> 
>  :00466084  6A00                PUSH      00 
> 
>   
> 
>  :00466086  FF9588D24400        CALL      \[EBP+0044D288\] 
> 
>  :0046608C  898584D24400        MOV       \[EBP+0044D284\],EAX 
> 
>  :00466092  8D9DAFC64400        LEA       EBX,\[EBP+0044C6AF\] 
> 
>  :00466098  50                  PUSH      EAX 
> 
>  :00466099  53                  PUSH      EBX 
> 
>  :0046609A  E899090000          CALL      00466A38 
> 
>  :0046609F  8BC8                MOV       ECX,EAX 
> 
>  :004660A1  8DBDAFC64400        LEA       EDI,\[EBP+0044C6AF\] 
> 
>   
> 
>  :004660A7  8BB584D24400        MOV       ESI,\[EBP+0044D284\] 
> 
>  :004660AD  F3A4                REPZ MOVSB 
> 
>      ;  执行这一行时 ,  画面会一闪 ,  因为他会改变自己的程式码 , 
> 
>      ;  看一下  ESI, 和  EDI,  和  ECX  的值吧 , SMC(self modifying code) 的技巧                                          
> 
>   
> 
> 4:  之后的  trace , 可真是  trace  的莫名其妙 ,  因为他会跳来跳去 ,  程式码随时在变 
> 
>     他总是跳到不太对的地方 ,  跳的真是怪怪的 , 相信有  trace   过的人都会知道 
> 
>     而且好几次 ,  遇到  call xxxxxxxx ,  就已经跑出来一个  debugger detection 
> 
>     只好重新再载入 ...  真的是相当繁琐 ,  只好用一些技巧了 ... 
> 
>     后来总算来到一个地方 ,  快要跳出这个  call  了 ,  若觉得太烦了 ,  可下   
> 
>    g 4664dd ,  可直接跑到  cs:4664dd  这边    
> 
>   
> 
>  :004664DD  6801F0C100          PUSH      00C1F001 
> 
>  :004664E2  C3                  RET 
> 
>   
> 
> 5:  走完了 ,  来到了 
> 
>  014F:00C1F001  60                  PUSHAD 
> 
>  014F:00C1F002  E844060000          CALL      00C1F64B 
> 
>  014F:00C1F007  EB44                JMP       00C1F04D 
> 
>  014F:00C1F009  0000                ADD       \[EAX\],AL 
> 
>   
> 
>      继续  trace  到了 
> 
>  014F:00C1F107  50                  PUSH      EAX 
> 
>  014F:00C1F108  C3                  RET 
> 
>   就走完这个  call  了 
> 
>   
> 
> 6: 
> 
>               到了这边 
> 
>  014F:00C1F30D  8B9D192A4400        MOV       EBX,\[EBP+00442A19\] 
> 
>  014F:00C1F313  0BDB                OR        EBX,EBX 
> 
>  014F:00C1F315  740A                JZ        00C1F321 
> 
>  014F:00C1F317  8B03                MOV       EAX,\[EBX\] 
> 
>  014F:00C1F319  87851D2A4400        XCHG      EAX,\[EBP+00442A1D\] 
> 
>  014F:00C1F31F  8903                MOV       \[EBX\],EAX 
> 
>       继续  trace  到了 
> 
>  014F:00C1F5D0  683C15C100          PUSH      00C1153C 
> 
>  014F:00C1F5D5  C3                  RET 
> 
>       就走完这个  call  了 
> 
>   
> 
> 7: 
> 
>               到了这边 ,  继续  trace 
> 
>   
> 
>  014F:00C1153C  55                  PUSH      EBP 
> 
>  014F:00C1153D  8BEC                MOV       EBP,ESP 
> 
>  014F:00C1153F  83C4F4              ADD       ESP,-0C 
> 
>  014F:00C11542  E8B91AFFFF          CALL      00C03000 
> 
>  014F:00C11547  0F854F29FFFF        JNZ       00C03E9C 
> 
>  014F:00C1154D  E8062EFFFF          CALL      00C04358 
> 
>  014F:00C11552  E82154FFFF          CALL      00C06978 
> 
>  014F:00C11557  E8F871FFFF          CALL      00C08754 
> 
>  014F:00C1155C  E8EFC7FFFF          CALL      00C0DD50 
> 
>  014F:00C11561  E8CAFFFFFF          CALL      00C11530 
> 
>  014F:00C11566  E83129FFFF          CALL      00C03E9C 
> 
>  ;  到了这边 ,  只要按一下  F10,  立刻出现  debugger detection  的讯息 
> 
>  014F:00C1156B  8BE5                MOV       ESP,EBP 
> 
>  014F:00C1156D  5D                  POP       EBP 
> 
>  014F:00C1156E  C20C00              RET       000C 
> 
>        
> 
> 8:  所以啦 ,  再  load  一次 ,  一  load,  立刻下  g c11566 ,  以节省时间 , 到了之后 
> 
>     立刻按  F8,  进入这个  call  去  trace     
> 
>     到了这边     
> 
>  014F:00C1132C  648920              MOV       FS:\[EAX\],ESP 
> 
>  014F:00C1132F  8B4508              MOV       EAX,\[EBP+08\] 
> 
>  014F:00C11332  E88DFBFFFF          CALL      00C10EC4  
> 
>  ;  一走到这边 ,  一按  F10,  立刻出现  debugger detection  的讯息 
> 
>  ;  所以这个  call  一定有问题 
> 
>  014F:00C11337  33C0                XOR       EAX,EAX 
> 
>  014F:00C11339  5A                  POP       EDX 
> 
>  014F:00C1133A  59                  POP       ECX 
> 
>         
> 
> 9:  
> 
>    再  load  一次 ,  一  load,  立刻下  g c11332 , 到了之后    
> 
>    立刻按  F8,  进入这个  call  去  trace    
> 
>    到了这边 ,    
> 
>  014F:00C10EFE  8BD3                MOV       EDX,EBX 
> 
>  014F:00C10F00  8B45F8              MOV       EAX,\[EBP-08\] 
> 
>  014F:00C10F03  E858F9FFFF          CALL      00C10860 
> 
>  014F:00C10F08  8A154466C100        MOV       DL,\[00C16644\] 
> 
>  014F:00C10F0E  8B45F8              MOV       EAX,\[EBP-08\] 
> 
>  014F:00C10F11  E88EF9FFFF          CALL      00C108A4 
> 
>  014F:00C10F16  8BD8                MOV       EBX,EAX 
> 
>  014F:00C10F18  85DB                TEST      EBX,EBX 
> 
>  014F:00C10F1A  7445                JZ        00C10F61 
> 
>  ;  只要这个能跳 ,  就没有  debugger detection  的讯息了 
> 
>  ;  所以到了这边 , 可以下  r fl z,  来跳过 
> 
>  014F:00C10F1C  6A00                PUSH      00 
> 
>  014F:00C10F1E  A17066C100          MOV       EAX,\[00C16670\] 
> 
>  014F:00C10F23  50                  PUSH      EAX 
> 
>     
> 
>      到了这边 
> 
>  014F:00C11046  7425                JZ        00C1106D 
> 
>  014F:00C11048  8D55E4              LEA       EDX,\[EBP-1C\] 
> 
>  014F:00C1104B  33C0                XOR       EAX,EAX 
> 
>  014F:00C1104D  E8D616FFFF          CALL      00C02728 
> 
>  014F:00C11052  8B45E4              MOV       EAX,\[EBP-1C\] 
> 
>  014F:00C11055  8D55E8              LEA       EDX,\[EBP-18\] 
> 
>  014F:00C11058  E8C33DFFFF          CALL      00C04E20 
> 
>  014F:00C1105D  8B45E8              MOV       EAX,\[EBP-18\] 
> 
>  014F:00C11060  E83323FFFF          CALL      00C03398 
> 
>  014F:00C11065  8B5303              MOV       EDX,\[EBX+03\] 
> 
>  014F:00C11068  E853F9FFFF          CALL      00C109C0 
> 
>   
> 
>  ;  走过这边 , 立刻出现  file corrupted  的讯息 
> 
>  ;  应该是  checksum  吧 ,  因为刚刚改了  section character 
> 
>   
> 
>  014F:00C1106D  8B45F8              MOV       EAX,\[EBP-08\] 
> 
>  014F:00C11070  E8AF18FFFF          CALL      00C02924 
> 
>   
> 
> 10:  那也要跳过  cs:00c11068  才行 ,  那乾脆到了   
> 
>  014F:00C10F1A  7445                JZ        00C10F61 
> 
>      这个本来是要跳过  debugger dection  的 ,  乾脆一次跳个够吧 
> 
>      用  symbol loader  载入 ,  下  G C10F1A,  到了这边 , 下  A,  下 
> 
>     JMP C1106D  吧 ,  抄下机械码 ,  是  E94E010000 
> 
>   
> 
> 11:  到了  CS:C1106D,  继续  TRACE, 到了这边 
> 
>  014F:00C1141D  8B4508              MOV       EAX,\[EBP+08\] 
> 
>  014F:00C11420  E87BFCFFFF          CALL      00C110A0 
> 
>  ;  这个  CALL  一走过就出现过期的讯息了   
> 
>  014F:00C11425  33C0                XOR       EAX,EAX 
> 
>  014F:00C11427  5A                  POP       EDX 
> 
>  014F:00C11428  59                  POP       ECX 
> 
>  014F:00C11429  59                  POP       ECX 
> 
>      
> 
> 12: 
> 
>    再  load  一次 ,  一  load,  立刻下  g c11420 , 到了之后    
> 
>    立刻按  F8,  进入这个  call  去  trace    
> 
>    到了这边 ,    
> 
>  014F:00C111E2  E80DEEFFFF          CALL      00C0FFF4 
> 
>  014F:00C111E7  84C0                TEST      AL,AL 
> 
>  014F:00C111E9  7456                JZ        00C11241  
> 
>   
> 
>  ; 可以从这边下手 
> 
>   
> 
>  014F:00C111EB  8B55F4              MOV       EDX,\[EBP-0C\] 
> 
>  014F:00C111EE  8B45E8              MOV       EAX,\[EBP-18\] 
> 
>  014F:00C111F1  E8A2EFFFFF          CALL      00C10198 
> 
>  014F:00C111F6  8D55C8              LEA       EDX,\[EBP-38\] 
> 
>  014F:00C111F9  8B45E8              MOV       EAX,\[EBP-18\] 
> 
>  014F:00C111FC  E81FF0FFFF          CALL      00C10220 
> 
>  014F:00C11201  8B55C8              MOV       EDX,\[EBP-38\] 
> 
>  014F:00C11204  B8F076C100          MOV       EAX,00C176F0 
> 
>  014F:00C11209  E8A21EFFFF          CALL      00C030B0 
> 
>  014F:00C1120E  8D55CE              LEA       EDX,\[EBP-32\] 
> 
>  014F:00C11211  8B45E8              MOV       EAX,\[EBP-18\] 
> 
>  014F:00C11214  E893F0FFFF          CALL      00C102AC 
> 
>  014F:00C11219  33C0                XOR       EAX,EAX 
> 
>  014F:00C1121B  8A45CF              MOV       AL,\[EBP-31\] 
> 
>  014F:00C1121E  50                  PUSH      EAX 
> 
>  014F:00C1121F  8D45D0              LEA       EAX,\[EBP-30\] 
> 
>  014F:00C11222  50                  PUSH      EAX 
> 
>  014F:00C11223  E8C0F0FFFF          CALL      00C102E8 
> 
>  014F:00C11228  84C0                TEST      AL,AL 
> 
>  014F:00C1122A  751A                JNZ       00C11246 
> 
>  014F:00C1122C  6A00                PUSH      00 
> 
>  014F:00C1122E  68DC12C100          PUSH      00C112DC 
> 
>  014F:00C11233  68E412C100          PUSH      00C112E4 
> 
>  014F:00C11238  6A00                PUSH      00 
> 
>  014F:00C1123A  E8CD32FFFF          CALL      USER32!MessageBoxA 
> 
>  014F:00C1123F  EB05                JMP       00C11246 
> 
>  014F:00C11241  E88AF4FFFF          CALL      00C106D0 
> 
>   
> 
>  ;  这个会出现   过期的讯息 
> 
>   
> 
>  014F:00C11246  33C0                XOR       EAX,EAX 
> 
>  014F:00C11248  5A                  POP       EDX 
> 
>  014F:00C11249  59                  POP       ECX 
> 
>  014F:00C1124A  59                  POP       ECX 
> 
>  014F:00C1124B  648910              MOV       FS:\[EAX\],EDX 
> 
>  014F:00C1124E  EB0F                JMP       00C1125F 
> 
>  014F:00C11250  E99F19FFFF          JMP       00C02BF4 
> 
>  014F:00C11255  E876F4FFFF          CALL      00C106D0 
> 
>  014F:00C1125A  E8011BFFFF          CALL      00C02D60 
> 
>  014F:00C1125F  33C0                XOR       EAX,EAX 
> 
>   
> 
>   
> 
> 13:  所以我们知道要改哪里了 
> 
>      在 
> 
>     014F:00C10F1A  7445                JZ        00C10F61 
> 
>      改成 JMP C1106D  吧 ,  
> 
>      也就是把机械码  74456A00A1 
> 
>                改成  E94E010000 
> 
>      第二个要改的地方是             
> 
>     014F:00C111E9  7456                JZ        00C11241  
> 
>      改成  JMP 00C1125F  就行了 
> 
>      所以是机械码  7456 
> 
>              改成  EB74 
> 
>      问题是要改吗 ,  此软体有加壳耶 , 所以无法直接用  ULTRAEDIT  改 
> 
>      那就用  Process Patcher  吧 
> 
> 14     
> 
>       将 ppatcher.ppc, ppatcher.exe copy  到  aspack 2.000  的目录底下 
> 
>       修改 ppatcher.ppc,  使他的内容如下 
> 
>   
> 
> #Process Patcher Configuration File 
> 
> Version=3.60 
> 
>   
> 
> DisplayName= 破  aspack2000 
> 
> Filename=aspack.exe 
> 
> Filesize=231424 
> 
> Arguments=/quiet 
> 
> WaitInfinite=true 
> 
> Address=0xc10f1a:0x74:0xe9 
> 
> Address=0xc10f1b:0x45:0x4e 
> 
> Address=0xc10f1c:0x6a:0x01 
> 
> Address=0xc10f1d:0x00:0x00 
> 
> Address=0xc10f1e:0xa1:0x00 
> 
> Address=0xc111e9:0x74:0xeb 
> 
> Address=0xc111ea:0x56:0x74 
> 
>   
> 
>   
> 
> #End of Configuration File 
> 
>   
> 
>   
> 
> 15:  这样执行  ppatcher , 就可以了 ,  但奇怪的是  debugger detection  有时会过不了 , 
> 
>      只好执行  FrogsICE,  在执行  ppatcher 了 
> 
>      但这样子仍是不满意 ,  我是想用  SMC(self modifying code)  的技巧来改 
> 
>      但是 ,  使用  SMC  有一个问题 , 要找哪里的 code  来放我所加入的  code  呢  ? 
> 
>      我的确是有找 , 但是找不到一个够大的地方来存放我的  code,  
> 
>      只好用  topo  来多一些空间来存放我的  code  了 
> 
>      这个  topo  真是太棒了 ,  可以多一些  section  出来 , 可以放入我所想要的   code 
> 
>      
> 
>      所以先   启动  topo,  开启 aspack.exe,  旁边有三个选项 
> 
>     Backup file,  等等 ,  通通打勾 
> 
>     Byte to be add,  随意填一个 ,  我是填  50, 
> 
>      在按  Do it!  好了之后 ,  我的  aspack.exe  档案变大了 
> 
>      从  231424 byte  变成  232960 , 好像大了一点喔 , 没关系 , 可以用就好了 
> 
>      下面还有一个框 , 写著   
> 
>     memory address : 478000h 
> 
>     file offset 38800h 
> 
>   
> 
> 16:  用  symbol loader  载入看看 
> 
>      一开始是  invalid,  没关系 , 按一下   F8, 如果画面乱掉了 , 就按两下  F4,  如果画面没乱掉 
> 
>       就不必了  . 一直  trace,   到了这边 ,  
> 
>  014F:0047802C  90                  NOP 
> 
>  014F:0047802D  E9CFDFFEFF          JMP       00466001 
> 
>  014F:00478032  0000                ADD       \[EAX\],AL 
> 
>  014F:00478034  0000                ADD       \[EAX\],AL 
> 
>  014F:00478036  0000                ADD       \[EAX\],AL 
> 
>       
> 
>       那就是说  cs:478032  以后可以任我使用了 ,  其实把  cs:478000  改成 
> 
>                JMP       00466001   
> 
>       也是可以啦 ,  反正只要执行没问题就  ok  了 
> 
>   
> 
> 17:  那要改哪里呢  ? 用  symbol loader  载入 , 下  g c1f5c6 
> 
>      到了  cs:c1f5c6 ,  下  a , 下  jmp 478032,  到了  cs:478032 
> 
>      下 a,  下  mov dword  ptr \[c10f1a\],00014ee9  
> 
>         再下  mov byte ptr \[c10f1e\],00 
> 
>         再下  mov word ptr \[c111e9\],74eb 
> 
>         再下  jmp c1f5d0 
> 
>         把机械码纪录下来 
> 
>         是  C7051A0FC100E94E0100 
> 
>           C6051E0FC10000 
> 
>           66C705E911C100EB74 
> 
>           E97F757A00 
> 
>        为什么要下这些指令呢  ?    
> 
>        因为我们要把  CS:c10f1a  改成  JMP C1106D, 机械码是 
> 
>       E94E010000,  但是要反过来 , dword   是  4 BYTE,  所以先取  E94E0100 
> 
>        但是要反过来 ,  所以变成了  00014ee9 , 
> 
>        其他的也是同样道理 ,  最后再跳回去  c1f5d0,  继续执行 
> 
>        用   Offset Converter 开启  ASPACK.EXE  输入    478032 
> 
>        跑出了  38A32,  所以用  ULTRA EDIT  开启 ASPACK.EXE  
> 
>        在  38A32  的地方  , 输入刚刚机械码 
> 
>        就是  C7051A0FC100E94E0100 
> 
>            C6051E0FC10000 
> 
>            66C705E911C100EB74 
> 
>            E97F757A00 
> 
>   
> 
> 18:  再来就是  cs:c1f5c6,  这个地方如何变成  jmp  478032   呢  ? 
> 
>      因为    cs:c1f5c6  这个地方 ,  是  aspack  用  SMC  产生的 , 
> 
>      所以无法直接修改他 ,  那就来观察是什么时候产生这边的  code, 
> 
>      用  symbol loader  载入 , 下  bpm c1f5c6,  按  F5,  会在这边停住 
> 
>     cs: 466310  ,  在按一次  F5,  会在  cs:c1f0eb  停住 
> 
>      共有两次会修改到 ,  在第二次的地方比较重要 
> 
>      所以 ,  到了  cs:c1f0eb , 下  BD\* , 按几下  F10, 
> 
>      到了这边 
> 
>      cs:c1f0eb   repz movsb 
> 
>      cs:c1f0ed   mov eax , \[ebp+442939\] 
> 
>      cs:c1f0f3   push  00008000 
> 
>      cs:c1f0f8   push 0 
> 
>       当走到了 cs:c1f0f3   push  00008000 , 
> 
>       这边的机械码是  6800800000,  把他改成  jmp 478051 
> 
>       所以下  a,  再下  jmp 478051   机械码是  E9598F85FF 
> 
>       
> 
>       到了  CS:478051  下 A ,  
> 
>         再下   PUSH 00008000,  按两下  ENTER, 
> 
>         仔细看一下 ,  怎么变成  PUSH 8000  了  ... 
> 
>         这可不行 ,  一定要  PUSH  00008000  才行啊 
> 
>         所以下  E 478051 ,  游标跑到上面去了 ,  改记忆体的值 , 改成 
> 
>        6800800000 ,  这样才对 , 
> 
>         好了 ,  按一下  F10,  在开始下  A, 
> 
>         再下  MOV DWORD PTR \[C1F5C6\] , 858A67E9 
> 
>         再下  MOV BYTE PTR \[C1F5CA\] , FF 
> 
>         再下  JMP C1F0F8 
> 
>         改了四行 , 机械码总共是 , 
> 
>        6800800000 
> 
>        C705C6F5C100E9678A85 
> 
>        C605CAF5C100FF 
> 
>        E98C707A00 
> 
>         一样的 ,  用    Offset Converter 开启  ASPACK.EXE  输入    478051 
> 
>         跑出  38A51 ,  
> 
>         所以用  ULTRA EDIT  开启 ASPACK.EXE ,  在  38A51  的地方输入 
> 
>        6800800000 
> 
>        C705C6F5C100E9678A85 
> 
>        C605CAF5C100FF 
> 
>        E98C707A00 
> 
>   
> 
> 19:        
> 
>      再来就是  c1f0f3 ,   这个地方如何变成   push  00008000  呢  ? 
> 
>      用  symbol loader  载入 , 下  BPM C1F0F3,  按  F5,  会在这边停住 
> 
>     CS:466310 REPZ MOVSD ,  但在这附近的程式码很奇怪 ,  所以不要在这边动手脚 
> 
>      那要在哪里呢  ? 先离开  ASPACK ,  再用  symbol loader  载入 ,  下  G 4664D7 
> 
>     CS: 4664D7 JMP 4664DA 
> 
>      就改这边好了 ,  把他改成  JMP 47806C,  机械码是  E9901B0100 
> 
>      跳到了 CS:47806C  时 
> 
>      改成  MOV DWORD PTR \[C1F0F3\], 858F59E9 
> 
>          MOV BYTE PTR \[C1F0F7\], FF 
> 
>          PUSH C1F001 
> 
>          RET  
> 
>           
> 
>      机械码是 
> 
>     C705F3F0C100E9598F85 
> 
>     C605F7F0C100FF 
> 
>     6801F0C100 
> 
>     C3 
> 
>      再用  ULTRAEDIT  在正确的位址填入这些值 
> 
>      
> 
> 20:  
> 
>     再来就是  4664D7 ,   这个地方如何变成  JMP 47806C  呢  ? 
> 
>     用  symbol loader  载入  ASPACK,  到了   
> 
>     
> 
>    CS:4660AD    REPZ MOVSB  
> 
>     ;  这行会改变一些程式码 
> 
>    CS:4660AF    MOV EAX, \[EBP+44D284\] 
> 
>    CS:4660B5    PUSH 000080000 
> 
>    ;  这行似乎不错 ,  就改这个吧 
> 
>     到了  CS:4660B5  时   
> 
>     下  A , 下  JMP 478083 , 机械码是  E9C91F0100 
> 
>     到了  CS:478083  时 
> 
>     下  A,  下 
> 
>    PUSH 00008000     
> 
>     按两下  ENTER 
> 
>     当然 , 和前面步骤一样 ,  
> 
>     所以下  E 478083 ,  游标跑到上面去了 ,  改记忆体的值 , 改成 
> 
>    6800800000 ,  这样才对 , 
> 
>   
> 
>    MOV DWORD PTR \[4664D7\],011B90E9 
> 
>    MOV BYTE PTR \[4664DB\],00 
> 
>    JMP 4660BA 
> 
>     这些机械码是 
> 
>     
> 
>    6800800000 
> 
>    C705D7644600E9901B01 
> 
>    C605DB64460000 
> 
>    E91CE0FEFF 
> 
>     用  ULTRAEDIT  在正确对应的地方输入这些值 
> 
>     
> 
> 21:  
> 
>      用 Offset Converter 开启  ASPACK.EXE  输入    4660B5 
> 
>      跑出  26AB5 
> 
>      所以用  ULTRA EDIT  开起  ASPACK.EXE,  在  26AB5  
> 
>      改成     E9C91F0100 
> 
>      改完了 ...  
> 
>      就大工告成了 , 
> 
>      执行一下  ASPACK,  没有日期限制 ,  也没有   侦测  SOFTICE  了 
> 
>      这就是用  SMC(self modifyiing code)  的技巧 ,  不错吧 
> 
>      只不过有点烦 ,  没办法 ,  因为  aspack.exe  本身一直改自己程式码 
> 
>      所以我才要改那么多东西 ,  因为 aspack 很复杂 ,  
> 
>      所以我也改的很复杂 ,  若能直接脱壳 ,  就不必用到  SMC  了 
> 
>      所以只要  SMC  用的好 ,  加壳的软体根本不怕 

 4dd466d0e1e0cc123afbe504da2462ca