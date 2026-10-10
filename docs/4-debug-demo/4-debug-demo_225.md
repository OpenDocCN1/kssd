

# Acdsee3.1

> 原文：[`www.pediy.com/kssd/pediy03/forum402.htm`](https://www.pediy.com/kssd/pediy03/forum402.htm)

 Acdsee3.1Build815 版   
来源：《电脑报配套光盘 2000-3》（上 M 的软件还是买光盘吧，毕竟老婆每月从电信缴费回来连续三天泡菜伺候的滋味不太好受。）   
  
安装后选试用，用满 90 天就有提示。   
  
先用 UnAspack 脱壳，再搜索 90 90 33 c0 c3 90 90 ,有两处，将后一处改为 90 90 b0 01 c3 90 90 即可。启动软件，它要我给它返回信息，还是永不吧。  

  

 使用工具：   
Fileinfo   
UnAspack   
procdump162   
W32dasm   
Trw2000   
UltraEdit   
  
Acdsee 3.1 Build815 安装时选试用，用满 90 天就会出现购买提示。这个版本与前些版本不同，它没有输入注册码的地方。于是决定对它进行反汇编。   
软件是用 Aspack 压缩过的，用 UnAspack 将其脱壳，再将脱壳过的文件用 procdump162 的 PE Editor 载入，修改 Sections 中的 Characteristics 值，由 C0000040 改为 E0000020,才能使用   
W32dasm 反汇编。   
反汇编出来后，查看 String Data References,内容还真多,找到"Trial Version"，发现有两处。   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004171ED(U)   
|   
:00417200 B9E8424F00              mov ecx, 004F42E8   
:00417205 E8361D0800              call 00498F40   
:0041720A 85C0                    test eax, eax   
:0041720C 7529                    jne 00417237   
:0041720E 8D8C2414010000          lea ecx, dword ptr \[esp+00000114\]   
:00417215 6A50                    push 00000050   
:00417217 51                      push ecx   
  
\* Reference To: KERNEL32.lstrlenA, Ord:0335h   
                                  |   
:00417218 FF1540834D00            Call dword ptr \[004D8340\]   
:0041721E 8D940418010000          lea edx, dword ptr \[esp+eax+00000118\]   
:00417225 A10C434F00              mov eax, dword ptr \[004F430C\]   
:0041722A 52                      push edx   
  
\* Possible Reference to String Resource ID=00195: " Trial Version"   
                                  |   
:0041722B 68C3000000              push 000000C3   
:00417230 50                      push eax   
  
  
  
和   
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004659EE(U)   
|   
:00465A16 B9E8424F00              mov ecx, 004F42E8   
:00465A1B E820350300              call 00498F40   
:00465A20 85C0                    test eax, eax   
:00465A22 7520                    jne 00465A44   
:00465A24 8D442408                lea eax, dword ptr \[esp+08\]   
:00465A28 6A50                    push 00000050   
:00465A2A 50                      push eax   
:00465A2B FFD6                    call esi   
:00465A2D 8B150C434F00            mov edx, dword ptr \[004F430C\]   
:00465A33 8D4C040C                lea ecx, dword ptr \[esp+eax+0C\]   
:00465A37 51                      push ecx   
  
\* Possible Reference to String Resource ID=00195: " Trial Version"   
                                  |   
:00465A38 68C3000000              push 000000C3   
:00465A3D 52                      push edx   
  
  
发现两处有共同特点，都是 call 00498f40 后,若 eax 不为 0 就可跳过，再看 00498f40 处:   
\* Referenced by a CALL at Addresses:   
|:00404502  , :0040469F  , :0040489B  , :00404A0E  , :0040873C    
|:00408833  , :004129AF  , :00417205  , :004182AB  , :004183C9    
|:0043D206  , :0043D2FB  , :0043D409  , :0043D4F5  , :0043D636    
|:0045A529  , :00465A1B    
|   
:00498F40 33c0                    xor eax,eax   
:00498F42 C3                      ret   
  
  
:00498F43 90                      nop   
:00498F44 90                      nop   
:00498F45 90                      nop   
:00498F46 90                      nop   
:00498F47 90                      nop   
:00498F48 90                      nop   
:00498F49 90                      nop   
:00498F4A 90                      nop   
:00498F4B 90                      nop   
:00498F4C 90                      nop   
:00498F4D 90                      nop   
:00498F4E 90                      nop   
:00498F4F 90                      nop   
  
\* Referenced by a CALL at Addresses:   
|:004990C1  , :00499169  , :0049925A  , :00499368    
|   
:00498F50 81ECA4000000            sub esp, 000000A4   
:00498F56 56                      push esi   
:00498F57 57                      push edi   
:00498F58 8BBC24B0000000          mov edi, dword ptr \[esp+000000B0\]   
  
  
共有 17 处调用它，而它仅仅是将 eax 值置 0，就直接跳到"Trial Version"去了，试着将   
:00498F40 33c0                    xor eax,eax   
:00498F42 C3                      ret   
改为   
:00498F40 B001                    mov al, 01   
:00498F42 C3                      ret   
  
用 UltraEdit 打开 Acdsee.exe,搜索 90 90 33 c0 c3 90 90 ,有两处，将后一处改为   
90 90 b0 01 c3 90 90(我不想搜索太长，那一串 90 太多，我担心数错。）   4dd466d0e1e0cc123afbe504da2462ca