

# 速达 3000pro 网络版

> 原文：[`www.pediy.com/kssd/pediy04/kanxue384.htm`](https://www.pediy.com/kssd/pediy04/kanxue384.htm)

青岛阿牛解狗教程之一（速达 3000pro 网络版）   
  
目的：打破加密狗加密的软件神秘感----你也可以解狗（包括一些航母级软件）   
  
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004238B5(C)   
|   
\* Reference To: sdproserver.Dogtestpro10::\_ManCheckDlg10(void())   
                            \`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`（太典型的检测狗的调用呀！）   
        
:004238E4 E8A76CFEFF              call 0040A590   
:004238E9 84C0                    test al, al   
:004238EB 7417                    je 00423904  //\*\*\*简单的讲，这里跳过去，速达 3000pro 网络端程序就可以运行了（当然还有一处，实在更改帐套的时候要调用，如果你修改 call  0040a590 的内部流程，使得 al 不等于零的话就省事情的很！不然的话两处你都得改！！）   
  
  
\* Reference To: sdproserver.Dogtestpro10::\_GetDlgNumber10(void())   
                                  |\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`（此处为检查软件狗内部允许的用户数量）   
  
  
:004238ED E81A70FEFF              call 0040A90C  //\*\*\*此处是调用读狗程序，也可以深入下去进行修改使 eax 得值是你需要的数量！！）   
  
:004238F2 A3F4594A00              mov dword ptr \[004A59F4\], eax   
:004238F7 8B45CC                  mov eax, dword ptr \[ebp\-34\]  //此处修改 eax 得值，就可以随心所欲的大道修改客户数量的目的，当然还有一处，同样的办法处理）    
:004238FA E851050000              call 00423E50   
:004238FF E99E000000              jmp 004239A2   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004238EB(C)   
|   
:00423904 66C745E00800            mov \[ebp\-20\], 0008   
:0042390A 8D45FC                  lea eax, dword ptr \[ebp\-04\]   
:0042390D E846BFFEFF              call 0040F858   
:00423912 8BC8                    mov ecx, eax   
:00423914 FF45EC                  inc \[ebp\-14\]   
:00423917 BAE8594A00              mov edx, 004A59E8   
  
\* Possible StringData Ref from Data Obj ->"没有检测到"   
                                  |   
:0042391C B82F7B4A00              mov eax, 004A7B2F   
:00423921 E8027F0700              call 0049B828   
:00423926 8D55FC                  lea edx, dword ptr \[ebp\-04\]   
:00423929 52                      push edx   
:0042392A 8D45F4                  lea eax, dword ptr \[ebp\-0C\]   
:0042392D E826BFFEFF              call 0040F858   
:00423932 50                      push eax   
:00423933 FF45EC                  inc \[ebp\-14\]   
  
\* Possible StringData Ref from Data Obj ->"网络版使用的软件狗！"   
                                  |   
:00423936 BA3A7B4A00              mov edx, 004A7B3A   
:0042393B 8D45F8                  lea eax, dword ptr \[ebp\-08\]   
:0042393E E8117A0700              call 0049B354   
:00423943 FF45EC                  inc \[ebp\-14\]   
:00423946 8D55F8                  lea edx, dword ptr \[ebp\-08\]   
:00423949 59                      pop ecx   
:0042394A 58                      pop eax   
:0042394B E8BC7B0700              call 0049B50C   
:00423950 8D55F4                  lea edx, dword ptr \[ebp\-0C\]   
:00423953 FF32                    push dword ptr \[edx\]   
:00423955 E84290FFFF              call 0041C99C   
:0042395A 59                      pop ecx   
:0042395B FF4DEC                  dec \[ebp\-14\]   
:0042395E 8D45F4                  lea eax, dword ptr \[ebp\-0C\]   
:00423961 BA02000000              mov edx, 00000002   
:00423966 E8497B0700              call 0049B4B4   
:0042396B FF4DEC                  dec \[ebp\-14\]   
:0042396E 8D45F8                  lea eax, dword ptr \[ebp\-08\]   
:00423971 BA02000000              mov edx, 00000002   
:00423976 E8397B0700              call 0049B4B4   
:0042397B FF4DEC                  dec \[ebp\-14\]   
:0042397E 8D45FC                  lea eax, dword ptr \[ebp\-04\]   
:00423981 BA02000000              mov edx, 00000002   
:00423986 E8297B0700              call 0049B4B4   
  
  
  
  
  
\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`   
\* Referenced by a CALL at Addresses:   
|:004238E4  , :00423A16  , :00423C67    
|   
  
Exported fn(): Dogtestpro10::\_ManCheckDlg10(void()) - Ord:000Dh   
:0040A590 55                      push ebp   
:0040A591 8BEC                    mov ebp, esp   
:0040A593 83C4E4                  add esp, FFFFFFE4   
:0040A596 33C0                    xor eax, eax   
:0040A598 8945E4                  mov dword ptr \[ebp\-1C\], eax   
:0040A59B 8945E8                  mov dword ptr \[ebp\-18\], eax   
:0040A59E 8945F0                  mov dword ptr \[ebp\-10\], eax   
:0040A5A1 33C0                    xor eax, eax   
:0040A5A3 55                      push ebp   
:0040A5A4 6877A64000              push 0040A677   
:0040A5A9 64FF30                  push dword ptr fs:\[eax\]   
:0040A5AC 648920                  mov dword ptr fs:\[eax\], esp   
:0040A5AF C645FF00                mov \[ebp\-01\], 00   
\* Reference To: sdproserver.Dogtestpro10::InitValue(void())   
                                  |   
:0040A5B3 E87CFFFFFF              call 0040A534   
\* Reference To: sdproserver.Dogtestpro10::DIS(void())   
                                  |   
:0040A5B8 E89F020000              call 0040A85C   
:0040A5BD 8945F4                  mov dword ptr \[ebp\-0C\], eax   
:0040A5C0 837DF40B                cmp dword ptr \[ebp\-0C\], 0000000B   
:0040A5C4 7409                    je 0040A5CF   
:0040A5C6 817DF415270000          cmp dword ptr \[ebp\-0C\], 00002715   
:0040A5CD 7506                    jne 0040A5D5   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040A5C4(C)   
|   
:0040A5CF C645FF00                mov \[ebp\-01\], 00   
:0040A5D3 EB7F                    jmp 0040A654   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040A5CD(C)   
|   
:0040A5D5 837DF400                cmp dword ptr \[ebp\-0C\], 00000000   
:0040A5D9 7575                    jne 0040A650   
:0040A5DB C745F801000000          mov \[ebp\-08\], 00000001   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040A631(C)   
|   
:0040A5E2 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:0040A5E5 B90A000000              mov ecx, 0000000A   
:0040A5EA 99                      cdq   
:0040A5EB F7F9                    idiv ecx   
:0040A5ED 83FA07                  cmp edx, 00000007   
:0040A5F0 7538                    jne 0040A62A   
:0040A5F2 B8FF000000              mov eax, 000000FF   
:0040A5F7 99                      cdq   
:0040A5F8 F77DF8                  idiv \[ebp\-08\]   
:0040A5FB 52                      push edx   
:0040A5FC 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:0040A5FF 8A90FBDE4B00            mov dl, byte ptr \[eax+004BDEFB\]   
:0040A605 80F2FF                  xor dl, FF   
:0040A608 81E2FF000000            and edx, 000000FF   
:0040A60E 58                      pop eax   
:0040A60F 2BD0                    sub edx, eax   
:0040A611 81F2FF000000            xor edx, 000000FF   
:0040A617 8D45E8                  lea eax, dword ptr \[ebp\-18\]   
  
\* Reference To: VCL50.System::::LStrFromChar(()), Ord:0000h   
                                  |   
:0040A61A E8E3230900              Call 0049CA02   
:0040A61F 8B55E8                  mov edx, dword ptr \[ebp\-18\]   
:0040A622 8D45F0                  lea eax, dword ptr \[ebp\-10\]   
  
\* Reference To: VCL50.System::::LStrCat(void()), Ord:0000h   
                                  |   
:0040A625 E8B4230900              Call 0049C9DE   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040A5F0(C)   
|   
:0040A62A FF45F8                  inc \[ebp\-08\]   
:0040A62D 837DF865                cmp dword ptr \[ebp\-08\], 00000065   
:0040A631 75AF                    jne 0040A5E2   
:0040A633 8D55E4                  lea edx, dword ptr \[ebp\-1C\]   
:0040A636 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
  
\* Reference To: VCL50.Sysutils::Trim(()), Ord:0000h   
                                  |   
:0040A639 E838260900              Call 0049CC76   
:0040A63E 8B45E4                  mov eax, dword ptr \[ebp\-1C\]   
\* Reference To: sdproserver.Dogtestpro10::\_Cmpar(())   
                                  |   
:0040A641 E84A020000              call 0040A890   
:0040A646 84C0                    test al, al   
:0040A648 740A                    je 0040A654   
:0040A64A C645FF01                mov \[ebp\-01\], 01   
:0040A64E EB04                    jmp 0040A654   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040A5D9(C)   
|   
:0040A650 C645FF00                mov \[ebp\-01\], 00   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0040A5D3(U), :0040A648(C), :0040A64E(U)   
|   
:0040A654 33C0                    xor eax, eax   
:0040A656 5A                      pop edx   
:0040A657 59                      pop ecx   
:0040A658 59                      pop ecx   
:0040A659 648910                  mov dword ptr fs:\[eax\], edx   
:0040A65C 687EA64000              push 0040A67E   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040A67C(U)   
|   
:0040A661 8D45E4                  lea eax, dword ptr \[ebp\-1C\]   
:0040A664 BA02000000              mov edx, 00000002   
  
\* Reference To: VCL50.System::::LStrArrayClr(void()), Ord:0000h   
                                  |   
:0040A669 E8A6230900              Call 0049CA14   
:0040A66E 8D45F0                  lea eax, dword ptr \[ebp\-10\]   
  
\* Reference To: VCL50.System::::LStrClr(()), Ord:0000h   
                                  |   
:0040A671 E8A4230900              Call 0049CA1A   
:0040A676 C3                      ret   
  
  
  
  
\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`   
\* Referenced by a CALL at Addresses:   
|:004238ED  , :00423A1F  , :00423C70    
|   
  
Exported fn(): Dogtestpro10::\_GetDlgNumber10(void()) - Ord:0013h   
:0040A90C 55                      push ebp   
:0040A90D 8BEC                    mov ebp, esp   
:0040A90F 83C4F4                  add esp, FFFFFFF4   
:0040A912 C7058CDE4B0003000000    mov dword ptr \[004BDE8C\], 00000003   
:0040A91C C70590DE4B0001000000    mov dword ptr \[004BDE90\], 00000001   
:0040A926 8D45FB                  lea eax, dword ptr \[ebp\-05\]   
:0040A929 A394DE4B00              mov dword ptr \[004BDE94\], eax   
:0040A92E E8E4FBFFFF              call 0040A517   
:0040A933 8945F4                  mov dword ptr \[ebp\-0C\], eax   
:0040A936 837DF400                cmp dword ptr \[ebp\-0C\], 00000000   
:0040A93A 7518                    jne 0040A954   
:0040A93C 33C0                    xor eax, eax   
:0040A93E 8A45FB                  mov al, byte ptr \[ebp\-05\]   
:0040A941 83E803                  sub eax, 00000003   
:0040A944 35FF000000              xor eax, 000000FF   
:0040A949 83E830                  sub eax, 00000030   
:0040A94C 83C00A                  add eax, 0000000A   
:0040A94F 8945FC                  mov dword ptr \[ebp\-04\], eax   
:0040A952 EB05                    jmp 0040A959   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040A93A(C)   
|   
:0040A954 33C0                    xor eax, eax   
:0040A956 8945FC                  mov dword ptr \[ebp\-04\], eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040A952(U)   
|   
:0040A959 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:0040A95C 8BE5                    mov esp, ebp   
:0040A95E 5D                      pop ebp   
:0040A95F C3                      ret   
  
现在看到了！解这只狗好简单！而我却费了好长时间，究其原因，实在是我太大意，根本就没注意到象   
（\* Reference To: sdproserver.Dogtestpro10::\_GetDlgNumber10(void())）的语句！一直在客户端找问题。   
  
我不太会写教程，对不住各位了！如有不明白的地方可以和我联系：qdcrack@sina.com   
  
我的竹叶：   
http://qdcrack.363.net   
  
(363.net 提供的竹叶空间实在太小，我的很多程序都没有办法放上去，让很多朋友失望！实在是没有办法！如果哪位老兄有空间，请和我联系，大家一起做好一个专业研究狗的网站）当然更希望各位解狗高手赐教！造福各位网友！)   
  
下期预告：   
  
青岛阿牛解狗教程之二（管家婆 8.2 网络版）   
目的：熟悉简单解狗方法   
敬请期待（要过年了！可能会延后！呵呵！） 

注：这里反汇编的是服务器端的程序!用 wdasm 反汇编会出现死机的情况,你可以参看论坛精华三的有关 anti 的相关文章!有两处跳转需要修改!然后就可以顺利反汇编! 

 4dd466d0e1e0cc123afbe504da2462ca