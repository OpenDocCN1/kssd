

# Tag&Rename v2.0 beta 1 

> 原文：[`www.pediy.com/kssd/pediy04/kanxue016.htm`](https://www.pediy.com/kssd/pediy04/kanxue016.htm)

注册 Tag&Rename v2.0 beta 1   
  
    这个又步 wincmd 的后尘，换汤不换药。做法跟 1.9.6 差不多，细节部分稍有不同。   
1.unpack(2001.9.12)   
    首先是脱壳部分，利用快速寻找 aspr 入口的方法:   
执行 tag&rename.exe，用 prodump 选 dump(full)脱壳,存为 dump.exe。接着用 winhex 打开 dump.exe，选择搜索文本，填 runtime,执行搜索，搜到后,向前找到离 runtime 最近的机器码为 55 8B EC 的地方就是程序的 oep，而在 tag&rename 里这个位置在 offset:0016e8a0 处，用 peditor 的 flc 功能将它转换为 virtual address 就是 56f2a0----oep。   
    ok，下面要在入口处脱壳，打开 SuperBPM，点 erase，用 trw 载入 t&r2b1，下 g 56f2a0，下 suspend。用 prodump 选 t&r2b1 进程 dump(full)，再打 ctrl+n，f5。注:为什么 t&r1.9.6 我没有在入口处脱壳，因为那纯属巧合，刚好能用罢了，而 2.0b1 不能用呀。一般情况下要在入口处 dump，不过能省则省么，偶懒，哈哈!   
  
    接着修复 import table，不过这个有点意思：ImportREC1.2final 无法提取完整的 it，要手动修复超过 300 个函数，累死~~~，好在有 1.9.6 的 it，copy 过来就行了，不过也狂耗时间，最衰的是修复后无法运行,惨~~~。无奈之下，用 ImportREC1.2beta2 重建 it，居然成功了，hoho,good luck:)   
  
以下操作必须在 tag&rename 的地盘里探测 

\*Tag&Rename v2.0 beta   
  
修复 import table 部分   
  
(1)    00174208    KERNEL32.dll    0151    GetCommandLineA   
  
(2)    00174358    KERNEL32.dll    013D    FreeResource   
  
(3)    00174394    KERNEL32.dll    01E9    GetVersion   
  
(4)    001743f4    KERNEL32.dll    0163    GetCurrentProcessId   
  
(5)    00174400    KERNEL32.dll    024c    LockResource   
  
补充一点关于菜单功能限制：   
  
具体的用 dede 或 exescope 打开看看，aspack 的功能限制也可以按这样打开，不过对改 section name 的 form 无效，哪位大哥知道的话麻烦告诉小弟一声。   
  
这两个是由 enable 控制的 08 为 false，09 为 true   
  
language 由于没有此资源，打开也没用。   
  
offset:23ab75 08->09  //open->close   
  
goto languages download page 可以使用，不过没有任何意义了:(，等正式版吧   
  
offset:23abe7 08->09  //open->close    
  
这两个是我第 1/2 次脱 aspr 的壳，错误在所难免，请见谅 

  
  
  
    修复 it 后，运行 t&r 报错 Initialization Error，所以就载入看看啦。发现错误在这个地方：   
  
:0056F2BF 55                      push ebp   
:0056F2C0 6804F35600              push 0056F304   
:0056F2C5 64FF30                  push dword ptr fs:\[eax\]   
:0056F2C8 648920                  mov dword ptr fs:\[eax\], esp   
:0056F2CB A13C235700              mov eax, dword ptr \[0057233C\]   
:0056F2D0 8B00                    mov eax, dword ptr \[eax\]   
:0056F2D2 E8DD23EDFF              call 004416B4   
:0056F2D7 FF15643E5700            call dword ptr \[00573E64\]        -->call 56E9B4   
这个只能用原版跟进去看看啦，发现这里最后 call 的是 56E9B4，在对应的位置改一下就好了。   
:0056F2DD A13C235700              mov eax, dword ptr \[0057233C\]   
:0056F2E2 8B00                    mov eax, dword ptr \[eax\]   
  
至此运行正常，脱壳任务完成了，继续 crack 工作。   
  
  
2.crack(2001.9.12)   
这个判断的模式一直没变过。选注册，名字任意，注册码 78787878，ctrl+d 下 s 30:0 l ffffffff '78787878'，bpm 搜到的地址。跟踪到如下：   
  
(1)   
\* Possible StringData Ref from Code Obj ->"x!A"   
                                  |   
:0045B2F7 A130054100              mov eax, dword ptr \[00410530\]   
:0045B2FC E8D37CFAFF              call 00402FD4   
:0045B301 8945F8                  mov dword ptr \[ebp\-08\], eax   
:0045B304 8D45F8                  lea eax, dword ptr \[ebp\-08\]   
:0045B307 E87CFEFFFF              call 0045B188   
:0045B30C 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:0045B30F 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:0045B312 8B08                    mov ecx, dword ptr \[eax\]   
:0045B314 FF5150                  call \[ecx+50\]   
:0045B317 40                      inc eax   
:0045B318 7402->9090              je 0045B31C ->nop   
:0045B31A B301                    mov bl, 01   
  
最难的还是无提示的 crc 校验，花了我不少时间，这个没法细说，花时间跟吧。起初选择 nop 掉 56E933-56E934，没有成功，后来就接着跟踪看他如何处理这段代码，没多久就迷失在代码迷宫里:(。干脆直接打补丁，最后的出路了。用 topo1.2 找出可用空间的地址 0067C4A3/offset:0025F4A3，把那段用来做 crc 的数据直接复制过去好啦   
  
(2)   
\* Possible Reference to String Resource ID=00004: "603D7D9D717D8C9023A94720FC9345E0FC9345E0"   
                                  |   
:0056E92E B904000000->e970db1000  mov ecx, 00000004->jmp 67c4a3   
:0056E933 F3                      repz   
:0056E934 A5                      movsd   
:0056E935 33C0                    xor eax, eax   
:0056E937 5A                      pop edx   
:0056E938 59                      pop ecx   
:0056E939 59                      pop ecx   
:0056E93A 648910                  mov dword ptr fs:\[eax\], edx   
:0056E93D 6852E95600              push 0056E952   
  
  
67c4a3 c705543e5700842e0ceb mov dword ptr \[573e54\],eb0c2e84  //offset:0025f4a3   
67c4ad c705583e5700cd7614b3 mov dword ptr \[553e58\],b31476cd   
67c4b7 c7055c3e5700c2883e70 mov dword ptr \[553e5c\],703e88c2   
67c4c1 c705603e570061ad33d2 mov dword ptr \[55bcc8\],d233ad61   
67c4cb e96524efff          jmp 56e935   
  
完活，好像能正常运行了，有错误的话请指出:)   
  
  
最后控制 trg&rename 时间的东西就是这个，用 norton register tracker 监测到的   
\[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\CLASSES\\CLSID\\{B6BDA2D7-451C-586C-838F-E3ACC3BACDB1}\]   
@="rKzQ1ZQ67t14y19M"   
删掉就可以重新计时   
  
REGEDIT4   
  
\[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\CLASSES\\CLSID\\{B6BDA2D7-451C-586C-838F-E3ACC3BACDB1}\]   
@=""   
或用这个补丁，把它存成 reset.reg   
  
汉化不知道需不需要改动，宏伟兄~~~~~~~~~~~~~~   
  
顺便贴个 TagRename Ver1.9-1.9.51 的 sn   
51soft 放出的，对以后版本无效   
  
name: ENFUSiA   
serial: TR\-O1tCAYu5-NzL90WVT   
  
2001.9.12   
zombieys\[CCG\]   
  
  
———————————————————————————————>   
                .-"      "-.      Cracked by zombieys\[CCG\]  >   
                /　　　　　　\\      qq:1789655                >   
              |              |    http://zombieys.yeah.net  >   
              |,  .-.  .-.  ,|    http://zombieys.126.com  >   
                |)(\_\_/  \\\_\_)(|      zombieys.cn.hongnet.com  >   
                |/    /\\    \\|                                >   
      (@\_@)    (\_    ^^    \_)      Thanks for your supports  >   
    \_  )\\\_\_\_\_\_\_\_\\\_\_|IIIIII|\_\_/\_\_\_\_\_                          >   
\_)@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_\_China Crack Group\_zombieys\_\_\_> 

 4dd466d0e1e0cc123afbe504da2462ca