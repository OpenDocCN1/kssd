
# % 超强灰鸽子 vip2005 检测器 % 检测原理简单分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-409.htm`](https://www.pediy.com/kssd/pediy07/pediy7-409.htm)

`% 超强灰鸽子 vip2005 检测器 % 检测原理简单分析`

`http://skyxnet.blogdriver.com/skyxnet/1013647.html`

前言，新款的灰鸽子总给人无处不在的感觉, 自己就曾在朋友主机中碰到多次,每次只能手工判断并清除.在看到此款检测器时，作了少许测试。效果很不错， 就产生了想了解她是如何工作的！^\_^

先来看看系统未感染 灰鸽子 时的执行情况=>>>

```
00459E2B    68 10A24500      push 超强灰鸽.0045A210                    

; ASCII "GPigeon5_Shared"  

00459E30    6A 00           push 0  

00459E32    6A 04           push 4  

00459E34    E8 E3C3FAFF     call <jmp.&kernel32.OpenFileMappingA>  

00459E39    A3 ACDC4500     mov dword ptr ds:[45DCAC],eax               

; Eax=0 表示无可操作句柄  

00459E3E    833D ACDC4500 0>cmp dword ptr ds:[45DCAC],0  

00459E45    0F84 70030000   je 超强灰鸽.0045A1BB       ;  jump  

OpenFileMappingA()函数执行后的堆栈情况：  

0012F5E4   00000004      |Access = FILE_MAP_READ  

0012F5E8   00000000      |InheritHandle = FALSE  

0012F5EC   0045A210      \\MappingName = "GPigeon5_Shared"  

0045A1BB    8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC] ;跳到此处  

0045A1C1    8B80 20020000   mov eax,dword ptr ds:[eax+220]  

0045A1C7    BA C8A34500     mov edx,超强灰鸽.0045A3C8                   

; 没有检测到 灰鸽子 Vip 2005 服务端  

0045A1CC    8B08            mov ecx,dword ptr ds:[eax]                  

; ecx=0x427c4c ASCII "4AA"  

0045A1CE    FF51 38        call dword ptr ds:[ecx+38]         ; Retn eax=0  

0045A1D1    833D ACDC4500 0>cmp dword ptr ds:[45DCAC],0  

0045A1D8    74 0B          je short 超强灰鸽.0045A1E5         ; Jump  

0045A1DA    A1 ACDC4500     mov eax,dword ptr ds:[45DCAC]  

0045A1DF    50             push eax  

0045A1E0    E8 F7BDFAFF     call <jmp.&kernel32.CloseHandle>  

0045A1E5    33C0            xor eax,eax  

0045A1E7    5A             pop edx  

0045A1E8    59             pop ecx  

0045A1E9    59             pop ecx  

0045A1EA    64:8910         mov dword ptr fs:[eax],edx  

0045A1ED    68 07A24500     push 超强灰鸽.0045A207  

0045A1F2    8D45 B8         lea eax,dword ptr ss:[ebp-48]  

0045A1F5    BA 12000000     mov edx,12  

0045A1FA    E8 F99CFAFF     call 超强灰鸽.00403EF8  

0045A1FF    C3              retn ;retn to 0x45a207  

剩下的是一些返回后的处理...  
```

这里可以看到,检测系统是否存在 "灰鸽子 Vip 2005 服务端" 是通过 `OpenFileMappingA()`打开一个现成的文件映射对象,如存在则返回成功打开的句柄,否则退出往下的检测清除程序段,用 C 简单可描述成：

```
hMap = OpenFileMapping(FILE_MAP_READ,FALSE,"GPigeon5_Shared");

if (hMap == NULL)
{
  "没有检测到 灰鸽子 Vip 2005 服务端!"
}
```

此时, 程序编辑中出来提示信息:"没有检测到 灰鸽子 Vip 2005 服务端!"

接着进入 灰鸽子 感染系统后 的检测及清除代码的分析...

同样地, 程序照常中断在此处=>>

```
00459E2B    68 10A24500        push 超强灰鸽.0045A210                      

; ASCII "GPigeon5_Shared"  

00459E30    6A 00              push 0  

00459E32    6A 04              push 4  

00459E34    E8 E3C3FAFF        call <jmp.&kernel32.OpenFileMappingA>  

00459E39    A3 ACDC4500        mov dword ptr ds:[45DCAC],eax             

; 如果检测到：eax=0xcc 0xb4 0xd8 句柄值，如无：eax=0  

00459E3E    833D ACDC4500 00   cmp dword ptr ds:[45DCAC],0  

00459E45    0F84 70030000      je 超强灰鸽.0045A1BB  ; not jump  

00459E4B    8B83 FC020000      mov eax,dword ptr ds:[ebx+2FC]        ; eax=01023f24  

00459E51    8B80 20020000      mov eax,dword ptr ds:[eax+220]        ; eax=01024208  

00459E57    BA 28A24500        mov edx,超强灰鸽.0045A228 

; edx=0x45a228 => 检测到 灰鸽子 Vip 2005 0105 服务端存在  

00459E5C    8B08               mov ecx,dword ptr ds:[eax]

; ecx=0x427c4c ; ASCII "4AA"  

00459E5E    FF51 38            call dword ptr ds:[ecx+38]  

; 编辑框中显示检测到的字符  

00459E61    6A 00              push 0  

00459E63    6A 00              push 0  

00459E65    6A 00              push 0  

00459E67    6A 04              push 4  

00459E69    A1 ACDC4500        mov eax,dword ptr ds:[45DCAC]  

; eax=0xcc \\0xb4 \\0xd8,句柄值  

00459E6E    50                 push eax  

; 将此句柄值压入堆栈,供下一函数用  

00459E6F    E8 98C3FAFF        call <jmp.&kernel32.MapViewOfFile>  ;映射文件名     
  
; Retn EAX=01330000=》ASCII "一大串数字"  

00459E74    8BF0               mov esi,eax  

堆栈值：  

eax=01310000, (ASCII "5F7E8111692AAA7694C721CB5300D7072A14D8CE7138EA0903AEAA23D8907C6072109D983725466507924E2237B9AF5BFBA7FC98060E8E620FE692E8DA4EE8D963D6241181D3988E9A13550DC7AF1E816F8FB154967BA939DDF9F6AC6F9B225CBDBDADFF3410875CA95DC8BA5C46BBC9A79DDA0F)  

esi=00429028 (超强灰鸽.00429028)  

???  不知道是什么数据来的 ???  检验码？  
  

00459E76    85F6               test esi,esi  

00459E78    74 62              je short 超强灰鸽.00459EDC  ; NoT jUMP  

00459E7A    8D45 F4            lea eax,dword ptr ss:[ebp-C]      

; EAX=0X12F640  

00459E7D    8BD6               mov edx,edx                       

; 将那串字符串传入 EDX 中  

00459E7F    E8 48A2FAFF        call 超强灰鸽.004040CC   ; ECX=00,EDX=00  

00459E84    B2 01              mov dl,1     ; edx=0x1  

00459E86    A1 70954500        mov eax,dword ptr ds:[459570]  

00459E8B    E8 48F7FFFF        call 超强灰鸽.004595D8  

00459E90    8BF0               mov esi,eax  

00459E92    8B45 F4            mov eax,dword ptr ss:[ebp-C]      ;

 将那串字符串再入传入  

00459E95    E8 FAA4FAFF        call 超强灰鸽.00404394  

00459E9A    8BD0               mov edx,eax  

00459E9C    8D45 EC            lea eax,dword ptr ss:[ebp-14]   

00459E9F    E8 28A2FAFF        call 超强灰鸽.004040CC   

00459EA4    8B45 EC            mov eax,dword ptr ss:[ebp-14]     

; EAX=01024398,ASCII"特征串"  

00459EA7    8D4D F0            lea ecx,dword ptr ss:[ebp-10] 

00459EAA    BA 5CA24500        mov edx,超强灰鸽.0045A25C                


; edx=0x45a25c ASCII "20050101"  

00459EAF    E8 A0F5FFFF        call 超强灰鸽.00459454   

00459EB4    8B55 F0            mov edx,dword ptr ss:[ebp-10] 

00459EB7    8BC6               mov eax,esi 

00459EB9    E8 FEF7FFFF        call 超强灰鸽.004596BC 

00459EBE    8D4D F8            lea ecx,dword ptr ss:[ebp-8]  

00459EC1    33D2               xor edx,edx  

00459EC3    8BC6               mov eax,esi 

00459EC5    E8 9AF8FFFF        call 超强灰鸽.00459764 

00459ECA    8BC6               mov eax,esi  

00459ECC    E8 0792FAFF        call 超强灰鸽.004030D8  

00459ED1    A1 ACDC4500        mov eax,dword ptr ds:[45DCAC]  

00459ED6    50                 push eax                      

; EAX=0XCC,句柄  

00459ED7    E8 00C1FAFF        call <jmp.&kernel32.CloseHandle>         

; 关闭操作的句柄  

00459EDC    8D45 E8            lea eax,dword ptr ss:[ebp-18]   

00459EDF    8B4D F8            mov ecx,dword ptr ss:[ebp-8] 

; ECX=010252F0=ASCII "83034" ,VIP 用户名  

00459EE2    BA 70A24500        mov edx,超强灰鸽.0045A270 

00459EE7    E8 F4A2FAFF        call 超强灰鸽.004041E0   

00459EEC    8B55 E8            mov edx,dword ptr ss:[ebp-18]     


; 传入编辑框中=》Vip 用户名：83034  

00459EEF    8B83 FC020000      mov eax,dword ptr ds:[ebx+2FC]  

00459EF5    8B80 20020000      mov eax,dword ptr ds:[eax+220]  

00459EFB    8B08               mov ecx,dword ptr ds:[eax]    

00459EFD    FF51 38            call dword ptr ds:[ecx+38]    

00459F00    E8 37FCFFFF        call 超强灰鸽.00459B3C                  

; 该函数检测是否存在隐藏模块,返回 eax=0x1 表示检测到隐藏模块  

00459F05    84C0               test al,al  

00459F07    0F84 96020000      je 超强灰鸽.0045A1A3   ; not jump  

00459F0D    E8 E6FAFFFF        call 超强灰鸽.004599F8

; 打开令牌环，设置系统调试权限  

00459F12    E8 61FCFFFF        call 超强灰鸽.00459B78 

; 检测隐藏模块,映射出具体的文件名  

00459F17    8B83 FC020000      mov eax,dword ptr ds:[ebx+2FC]  

00459F1D    8B80 20020000      mov eax,dword ptr ds:[eax+220]  

00459F23    BA 84A24500        mov edx,超强灰鸽.0045A284  

00459F28    8B08               mov ecx,dword ptr ds:[eax] 

00459F2A    FF51 38            call dword ptr ds:[ecx+38] 

; 该函数处理一些消息  

00459F2D    A1 C8DC4500        mov eax,dword ptr ds:[45DCC8]  

00459F32    33D2               xor edx,edx  

00459F34    52                 push edx  

00459F35    50                 push eax  
```



```assembly
00459F36 8D45 E0         lea eax,dword ptr ss:[ebp-20]  

00459F39 E8 EAE1FAFF     call 超强灰鸽.00408128  

; 获取隐藏进程 ID ?  

00459F3E 8B4D E0         mov ecx,dword ptr ss:[ebp-20]  

; ecx=00ee546c ascii "916"\\1804  

00459F41 8D45 E4         lea eax,dword ptr ss:[ebp-1C]  

00459F44 BA B4A24500     mov edx,超强灰鸽.0045A2B4  

00459F49 E8 92A2FAFF     call 超强灰鸽.004041E0  

00459F4E 8B55 E4         mov edx,dword ptr ss:[ebp-1C]  

00459F51 8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]  

00459F57 8B80 20020000   mov eax,dword ptr ds:[eax+220]  

00459F5D 8B08            mov ecx,dword ptr ds:[eax]  

00459F5F FF51 38         call dword ptr ds:[ecx+38]  

; 隐藏的进程 ID：1804  

00459F62 8D45 DC         lea eax,dword ptr ss:[ebp-24]  

00459F65 8B0D B8DC4500   mov ecx,dword ptr ds:[45DCB8]  

00459F6B BA CCA24500     mov edx,超强灰鸽.0045A2CC  

00459F70 E8 6BA2FAFF     call 超强灰鸽.004041E0  

00459F75 8B55 DC         mov edx,dword ptr ss:[ebp-24]  

00459F78 8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]  

00459F7E 8B80 20020000   mov eax,dword ptr ds:[eax+220]  

00459F84 8B08            mov ecx,dword ptr ds:[eax]  

00459F86 FF51 38         call dword ptr ds:[ecx+38]              

; 安装文件名：  LWVVKL\_  

00459F89 8D45 D8         lea eax,dword ptr ss:[ebp-28]  

00459F8C 8B0D BCDC4500   mov ecx,dword ptr ds:[45DCBC]  

00459F92 BA E4A24500     mov edx,超强灰鸽.0045A2E4  

00459F97 E8 44A2FAFF     call 超强灰鸽.004041E0  

00459F9C 8B55 D8         mov edx,dword ptr ss:[ebp-28]  

00459F9F 8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]  

00459FA5 8B80 20020000   mov eax,dword ptr ds:[eax+220]  

00459FAB 8B08            mov ecx,dword ptr ds:[eax]  

00459FAD FF51 38         call dword ptr ds:[ecx+38]              

; 主 DLL 文件名： 3WVVK+3  

00459FB0 8D45 D4         lea eax,dword ptr ss:[ebp-2C]  

00459FB3 E8 94FDFFFF     call 超强灰鸽.00459D4C  

; GetWindowsDirectoryA()  

00459FB8 FF75 D4         push dword ptr ss:[ebp-2C]              

; SS:[0012F620]=00EE54E4 ASCII "D:\\WINDOWS\\"  

00459FBB 8D45 D0         lea eax,dword ptr ss:[ebp-30]  

00459FBE 50              push eax  

00459FBF A1 B8DC4500     mov eax,dword ptr ds:[45DCB8]  

00459FC4 E8 CBA1FAFF     call 超强灰鸽.00404194  

00459FC9 8BC8            mov ecx,eax  

00459FCB 83E9 04         sub ecx,4  

00459FCE BA 01000000     mov edx,1  

00459FD3 A1 B8DC4500     mov eax,dword ptr ds:[45DCB8]   

; LWVVKL\_  

00459FD8 E8 17A4FAFF     call 超强灰鸽.004043F4  

00459FDD FF75 D0         push dword ptr ss:[ebp-30]   

 ; ASCII "LWVV"  

00459FE0 68 FCA24500     push 超强灰鸽.0045A2FC   

 ; ASCII "\_Hook.DLL"  

00459FE5 8D45 FC         lea eax,dword ptr ss:[ebp-4]  

00459FE8 BA 03000000     mov edx,3   

; 3 个字符串连接数  

00459FED E8 62A2FAFF     call 超强灰鸽.00404254  

; 将三个字符串连接起来 ASCII "D:\\WINDOWS\\LWVV\_Hook.DLL"  

00459FF2 8B45 FC         mov eax,dword ptr ss:[ebp-4]   

; EAX=00459FF2 SS:[0012F648]=00EE5510,ASCII "D:\\WINDOWS\\LWVV\_Hook.DLL"  

00459FF5 E8 9AA1FAFF     call 超强灰鸽.00404194   

 ; EAX=0X18=24 => ASCII "D:\\WINDOWS\\LWVV\_Hook.DLL" 字符长度数  

00459FFA 8BD0            mov edx,eax  

00459FFC 85D2            test edx,edx  

00459FFE 7E 18           jle short 超强灰鸽.0045A018  

0045A000 BE 01000000     mov esi,1  

0045A005 B8 CCDC4500     mov eax,超强灰鸽.0045DCCC   

; ASCII "D:\\WINDOWS\\LWVV\_Hook.DLL"  

0045A00A 8B4D FC         mov ecx,dword ptr ss:[ebp-4]  

0045A00D 8A4C31 FF       mov cl,byte ptr ds:[ecx+esi-1]  

0045A011 8808            mov byte ptr ds:[eax],cl  

0045A013 46              inc esi  

0045A014 40              inc eax  

0045A015 4A              dec edx  

0045A016  ^ 75 F2        jnz short 超强灰鸽.0045A00A  

0045A018 E8 47C1FAFF     call <jmp.&kernel32.GetVersion>  

0045A01D A9 00000080     test eax,80000000  

0045A022 74 13          je short 超强灰鸽.0045A037   ; JUMP  

0045A024 68 581B0000     push 1B58  

0045A029 68 CCDC4500     push 超强灰鸽.0045DCCC   

; ASCII "D:\\WINDOWS\\LWVV\_Hook.DLL"  

0045A02E 6A FD           push -3  

0045A030 E8 97E6FFFF     call 超强灰鸽.004586CC  

0045A035 EB 2B          jmp short 超强灰鸽.0045A062  

0045A037 8B45 FC         mov eax,dword ptr ss:[ebp-4]  

0045A03A E8 55A1FAFF     call 超强灰鸽.00404194  

0045A03F 8BC8            mov ecx,eax  

0045A041 03C9            add ecx,ecx  

0045A043 41              inc ecx  

0045A044 BA D4DD4500     mov edx,超强灰鸽.0045DDD4   

 ; UNICODE "D:\\WINDOWS\\LWVV\_Hook.DLL"  

0045A049 8B45 FC         mov eax,dword ptr ss:[ebp-4]  

0045A04C E8 1FACFAFF     call 超强灰鸽.00404C70   

; MultiByteToWideChar()  

0045A051 68 581B0000     push 1B58  

0045A056 68 D4DD4500     push 超强灰鸽.0045DDD4   

; UNICODE "D:\\WINDOWS\\LWVV\_Hook.DLL"  

0045A05B 6A FD           push -3  

0045A05D E8 96E6FFFF     call 超强灰鸽.004586F8  

0045A062 8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]  

0045A068 8B80 20020000   mov eax,dword ptr ds:[eax+220]  

0045A06E BA 10A34500     mov edx,超强灰鸽.0045A310  

0045A073 8B08            mov ecx,dword ptr ds:[eax]  

0045A075 FF51 38         call dword ptr ds:[ecx+38]  

0045A078 6A 00           push 0  

0045A07A 68 30A34500     push 超强灰鸽.0045A330   

 ; ASCII "TGVIP\_MainForm"  

0045A07F E8 78C5FAFF     call <jmp.&user32.FindWindowA>  

;查找隐藏进程窗口  

0045A084 8BF0            mov esi,eax  

0045A086 85F6            test esi,esi  

0045A088 74 61          je short 超强灰鸽.0045A0EB  

0045A08A 8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]  

0045A090 8B80 20020000   mov eax,dword ptr ds:[eax+220]  

0045A096 BA 48A34500     mov edx,超强灰鸽.0045A348  

0045A09B 8B08            mov ecx,dword ptr ds:[eax]  

0045A09D FF51 38         call dword ptr ds:[ecx+38]  

0045A0A0 6A 64           push 64  

0045A0A2 E8 8128FBFF     call <jmp.&kernel32.Sleep>  

0045A0A7 6A 00           push 0  

0045A0A9 6A 00           push 0  

0045A0AB 68 00340000     push 3400  

0045A0B0 56              push esi  

0045A0B1 E8 86C7FAFF     call <jmp.&user32.PostMessageA>  

0045A0B6 6A 64           push 64  

0045A0B8 E8 6B28FBFF     call <jmp.&kernel32.Sleep>  

0045A0BD 6A 00           push 0  

0045A0BF 6A 00           push 0  

0045A0C1 68 00340000     push 3400  

0045A0C6 56              push esi  

0045A0C7 E8 70C7FAFF     call <jmp.&user32.PostMessageA>  

0045A0CC 6A 64           push 64  

0045A0CE E8 5528FBFF     call <jmp.&kernel32.Sleep>  

0045A0D3 6A 00           push 0  

0045A0D5 6A 00           push 0  

0045A0D7 68 00340000     push 3400  

0045A0DC 56              push esi  

0045A0DD E8 5AC7FAFF     call <jmp.&user32.PostMessageA>  

0045A0E2 6A 64           push 64  

0045A0E4 E8 3F28FBFF     call <jmp.&kernel32.Sleep>  

0045A0E9 EB 16          jmp short 超强灰鸽.0045A101  

0045A0EB 8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]  

0045A0F1 8B80 20020000   mov eax,dword ptr ds:[eax+220]  

0045A0F7 BA 7CA34500     mov edx,超强灰鸽.0045A37C  

0045A0FC 8B08            mov ecx,dword ptr ds:[eax]  

0045A0FE FF51 38         call dword ptr ds:[ecx+38]  

0045A101 8D45 CC         lea eax,dword ptr ss:[ebp-34]  

0045A104 E8 43FCFFFF     call 超强灰鸽.00459D4C  

0045A109 8D45 CC         lea eax,dword ptr ss:[ebp-34]  
```



```
0045A10C    8B15 B8DC4500       mov edx,dword ptr ds:[45DCB8]  

0045A112    E8 85A0FAFF         call 超强灰鸽.0040419C  

0045A117    8B45 CC             mov eax,dword ptr ss:[ebp-34]  

0045A11A    33D2               xor edx,edx  

0045A11C    E8 9BE3FAFF         call 超强灰鸽.004084BC  

0045A121    8D45 C4             lea eax,dword ptr ss:[ebp-3C]  

0045A124    E8 23FCFFFF         call 超强灰鸽.00459D4C                      

; 取消所要处理文件的属性  

0045A129    8D45 C4             lea eax,dword ptr ss:[ebp-3C]  

0045A12C    8B15 B8DC4500       mov edx,dword ptr ds:[45DCB8]  

0045A132    E8 65A0FAFF         call 超强灰鸽.0040419C  

0045A137    8B45 C4             mov eax,dword ptr ss:[ebp-3C]  

0045A13A    E8 55A2FAFF         call 超强灰鸽.00404394  

0045A13F    8BD0               mov edx,eax  

0045A141    8D45 C8             lea eax,dword ptr ss:[ebp-38]  

0045A144    E8 839FFAFF         call 超强灰鸽.004040CC  

0045A149    8B45 C8             mov eax,dword ptr ss:[ebp-38]  

0045A14C    E8 93E3FAFF         call 超强灰鸽.004084E4  

; 删除病毒文件  

0045A151    8D45 C0             lea eax,dword ptr ss:[ebp-40]  

0045A154    E8 F3FBFFFF         call 超强灰鸽.00459D4C  

 ; 获取 WINDOWS 目录  

0045A159    8D45 C0             lea eax,dword ptr ss:[ebp-40]  

0045A15C    8B15 BCDC4500       mov edx,dword ptr ds:[45DCBC]  

0045A162    E8 35A0FAFF         call 超强灰鸽.0040419C  

0045A167    8B45 C0             mov eax,dword ptr ss:[ebp-40]  

0045A16A    33D2               xor edx,edx  

0045A16C    E8 4BE3FAFF         call 超强灰鸽.004084BC  

 ; 取消文件的属性  

0045A171    8D45 B8             lea eax,dword ptr ss:[ebp-48]  

0045A174    E8 D3FBFFFF         call 超强灰鸽.00459D4C  

0045A179    8D45 B8             lea eax,dword ptr ss:[ebp-48]  

0045A17C    8B15 BCDC4500       mov edx,dword ptr ds:[45DCBC]  

0045A182    E8 15A0FAFF         call 超强灰鸽.0040419C  

0045A187    8B45 B8             mov eax,dword ptr ss:[ebp-48]  

0045A18A    E8 05A2FAFF         call 超强灰鸽.00404394  

0045A18F    8BD0               mov edx,eax  

0045A191    8D45 BC             lea eax,dword ptr ss:[ebp-44]  

0045A194    E8 339FFAFF         call 超强灰鸽.004040CC  

0045A199    8B45 BC             mov eax,dword ptr ss:[ebp-44]  

0045A19C    E8 43E3FAFF         call 超强灰鸽.004084E4    

 ; 删除病毒体  

0045A1A1    EB 42               jmp short 超强灰鸽.0045A1E5  

0045A1A3    8B83 FC020000       mov eax,dword ptr ds:[ebx+2FC]  

0045A1A9    8B80 20020000       mov eax,dword ptr ds:[eax+220]  

0045A1AF    BA A4A34500         mov edx,超强灰鸽.0045A3A4  

0045A1B4    8B08               mov ecx,dword ptr ds:[eax]  

0045A1B6    FF51 38             call dword ptr ds:[ecx+38]  

0045A1B9    EB 2A               jmp short 超强灰鸽.0045A1E5  

处理完毕啦。。。。  

0045A1E5    33C0               xor eax,eax  

0045A1E7    5A                 pop edx  

0045A1E8    59                 pop ecx  

0045A1E9    59                 pop ecx  

0045A1EA    64:8910             mov dword ptr fs:[eax],edx  

0045A1ED    68 07A24500         push 超强灰鸽.0045A207  

0045A1F2    8D45 B8             lea eax,dword ptr ss:[ebp-48]  

0045A1F5    BA 12000000         mov edx,12  

0045A1FA    E8 F99CFAFF         call 超强灰鸽.00403EF8  

0045A1FF    C3                 retn  
```

### 总结

该检测器通过 `OpenFileMappingA()` 函数检测指定的映射对象是否存在作判断，如存在，获取 VIP 用户名，继续检测隐藏的进程模块，获取其进程 ID，映像出具体文件名（继而作相应的文件完整路径的组合），进行权限提升（打开令牌环，设置系统调试权限），`PostMessageA()` 发送消息关闭隐藏进程主窗口，处理服务进程，取消病毒文件的所有属性（系统，隐藏，只读），最后删除病毒体。

检测器代码采用 Borland Delphi 编写，看这些反汇编代码总会碰到层层调用这一现象，在这些主要 CALL 的注释中，其实其内部的调用还有很多较详细的代码调用，限于篇幅，就不一一列举了。

此外，由于本人能力的有限，错误及遗漏在所难免！或许检测器原理并没有这么简单，还请检测器程序的编写作者或其他高手作出指点。万分感谢！

`4dd466d0e1e0cc123afbe504da2462ca`
