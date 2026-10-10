

# Advanced Ra-Renamer v.1.2 

> 原文：[`www.pediy.com/kssd/pediy04/kanxue017.htm`](https://www.pediy.com/kssd/pediy04/kanxue017.htm)

注册 Advanced Ra-Renamer v.1.2   
  
  今天去下 t&r2.0b1 的时候，看到了这个，它跟 t&r 是同一公司的产品，不过要比 t&r 要简单的多，顺便也搞定。   
  
1.unpack(2001.9.12)   
  
    首先是脱壳部分，利用快速寻找 aspr 入口的方法:   
执行 ra\_ren.exe，用 prodump 选 dump(full)脱壳,存为 dump.exe。接着用 winhex 打开 dump.exe，选择搜索文本，填 runtime,执行搜索，搜到后,向前找到离 runtime 最近的机器码为 55 8B EC 的地方就是程序的 oep，而在 ra\_ren.exe 里这个位置在 offset:000F40EC 处，用 peditor 的 flc 功能将它转换为 virtual address 就是 004F4AEC----oep。   
    ok，下面要在入口处脱壳，打开 SuperBPM，点 erase，用 trw 载入 ra\_ren，下 g 56f2a0，下 suspend。用 prodump 选 ra\_ren 进程 dump(full)，再打 ctrl+n，f5。   
  
    接着修复 import table，这个跟 t&r2b1 一样，用 ImportREC1.2final 无法提取完整的 it，用 ImportREC1.2beta2 能成功，hoho,interesting:)   
  
(1)    000F81F4    KERNEL32.dll    0151    GetCommandLineA   
  
(2)    000F8330    KERNEL32.dll    013D    FreeResource   
  
(3)    000F836C    KERNEL32.dll    01E9    GetVersion   
  
(4)    000F83BC    KERNEL32.dll    0163    GetCurrentProcessId   
  
(5)    000F83C4    KERNEL32.dll    024c    LockResource   
  
选 add new section，然后点 fix dump。   
  
      
    修复 it 后，运行 ra\_ren 报错 Initialization Error，所以就载入看看啦。发现错误在这个地方：   
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Program Entry Point  \*\*\*\*\*\*\*\*   
:004F4AEC 55                      push ebp   
:004F4AED 8BEC                    mov ebp, esp   
:004F4AEF 83C4F4                  add esp, FFFFFFF4   
:004F4AF2 B854464F00              mov eax, 004F4654   
:004F4AF7 E8541BF1FF              call 00406650   
:004F4AFC A10C654F00              mov eax, dword ptr \[004F650C\]   
:004F4B01 8B00                    mov eax, dword ptr \[eax\]   
:004F4B03 E8308CF4FF              call 0043D738   
:004F4B08 FF15D87E4F00            call dword ptr \[004F7ED8\] ->call 4f43e8   
:004F4B0E A10C654F00              mov eax, dword ptr \[004F650C\]   
:004F4B13 8B00                    mov eax, dword ptr \[eax\]   
  
至此运行正常，脱壳任务完成了，继续 crack 工作。   
  
  
2.crack(2001.9.12)   
选注册，名字任意，注册码 78787878，ctrl+d 下 s 30:0 l ffffffff '78787878'，bpm 搜到的地址。跟踪到如下：   
  
\* Possible StringData Ref from Code Obj ->"?A"   
                                  |   
:004572E3 A130F14000              mov eax, dword ptr \[0040F130\]   
:004572E8 E89BBBFAFF              call 00402E88   
:004572ED 8945F8                  mov dword ptr \[ebp\-08\], eax   
:004572F0 8D45F8                  lea eax, dword ptr \[ebp\-08\]   
:004572F3 E87CFEFFFF              call 00457174   
:004572F8 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004572FB 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:004572FE 8B08                    mov ecx, dword ptr \[eax\]   
:00457300 FF5150                  call \[ecx+50\]   
:00457303 40                      inc eax   
:00457304 7402->9090              je 00457308 ->nop   
:00457306 B301                    mov bl, 01   
  
  
hoho 没有 crc 校验，可能是这个不太出名的缘故吧   
完活，好像能正常运行了，有错误的话请指出:)   
  
  
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
\_)@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_\_China Crack Group\_zombieys\_\_\_>  4dd466d0e1e0cc123afbe504da2462ca