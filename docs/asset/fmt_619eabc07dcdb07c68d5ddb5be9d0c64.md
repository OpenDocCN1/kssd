
三、OEP

MagicJmp 处理完了，可以看出与 `Armadillo V3.X-V4.X` 的处理没有大的不同。

其实也可以在下面的流程中避开输入表加密，并且去掉填充输入表 DLL 间的垃圾数据，这里就不写了，放脚本里面处理吧。

#### `BP CreateThread`

`Shift+F9` 中断后取消断点，`Alt+F9` 返回。

代码:

```
00DA3646     FF15 64E1DD00      call dword ptr ds:[DDE164] ; kernel32.CreateThread   
00DA364C     50                 push eax  
//返回这里   
00DA364D     FF15 84E2DD00      call dword ptr ds:[DDE284] ; kernel32.CloseHandle   
00DA3653     5E                 pop esi   
00DA3654     5B                 pop ebx   
00DA3655     8BE5               mov esp,ebp   
00DA3657     5D                 pop ebp   
00DA3658     C3                 retn   
//返回 00DBF1EF 处
```

`Ctrl+B` 在下面搜索：`FF D1 89 45 FC 8B 45 FC`。

找到在 `00DBF2F4` 处，设断，`Shift+F9`。

代码:

```
00DBF1EA     E8 2143FEFF        call 00DA3510   
00DBF1EF     83C4 04            add esp,4   
00DBF1F2     B9 B8B4DE00        mov ecx,0DEB4B8   
00DBF1F7     E8 B4BFFBFF        call 00D7B1B0   
00DBF1FC     0FB6D0             movzx edx,al   
00DBF1FF     85D2               test edx,edx   
00DBF201     74 0C              je short 00DBF20F   
00DBF203     6A 01              push 1   
00DBF205     B9 B8B4DE00        mov ecx,0DEB4B8   
00DBF20A     E8 11A6FCFF        call 00D89820   
00DBF20F     C705 3877DE00 081C > mov dword ptr ds:[DE7738],0DE1C08   
00DBF219     B9 04A6DE00        mov ecx,0DEA604   
00DBF21E     E8 5D36FBFF        call 00D72880   
00DBF223     C745 F0 00000000   mov dword ptr ss:[ebp-10],0   
00DBF22A     8D45 E8            lea eax,dword ptr ss:[ebp-18]   
00DBF22D     50                 push eax   
00DBF22E     68 10F3DB00        push 0DBF310   
00DBF233     FF15 74B1DE00      call dword ptr ds:[DEB174]   
00DBF239     83C4 08            add esp,8   
00DBF23C     8B0D 1CBBDE00      mov ecx,dword ptr ds:[DEBB1C]   
00DBF242     894D E4            mov dword ptr ss:[ebp-1C],ecx   
00DBF245     BA 07000000        mov edx,7   
00DBF24A     C1E2 02            shl edx,2   
00DBF24D     A1 04BBDE00        mov eax,dword ptr ds:[DEBB04]   
00DBF252     8B0D 04BBDE00      mov ecx,dword ptr ds:[DEBB04]   
00DBF258     8B35 04BBDE00      mov esi,dword ptr ds:[DEBB04]   
00DBF25E     8B76 04            mov esi,dword ptr ds:[esi+4]   
00DBF261     3371 34            xor esi,dword ptr ds:[ecx+34]   
00DBF264     333410             xor esi,dword ptr ds:[eax+edx]   
00DBF267     0375 E4            add esi,dword ptr ss:[ebp-1C]   
00DBF26A     8975 F4            mov dword ptr ss:[ebp-C],esi   
00DBF26D     8B55 08            mov edx,dword ptr ss:[ebp+8]   
00DBF270     833A 00            cmp dword ptr ds:[edx],0   
00DBF273     75 3E              jnz short 00DBF2B3   
00DBF275     A1 04BBDE00        mov eax,dword ptr ds:[DEBB04]   
00DBF27A     8B0D 04BBDE00      mov ecx,dword ptr ds:[DEBB04]   
00DBF280     8B50 68            mov edx,dword ptr ds:[eax+68]   
00DBF283     3351 34            xor edx,dword ptr ds:[ecx+34]   
00DBF286     A1 04BBDE00        mov eax,dword ptr ds:[DEBB04]   
00DBF28B     3350 60            xor edx,dword ptr ds:[eax+60]   
00DBF28E     8955 E0            mov dword ptr ss:[ebp-20],edx   
00DBF291     8B4D 08            mov ecx,dword ptr ss:[ebp+8]   
00DBF294     8B51 18            mov edx,dword ptr ds:[ecx+18]   
00DBF297     52                 push edx   
00DBF298     8B45 08            mov eax,dword ptr ss:[ebp+8]   
00DBF29B     8B48 14            mov ecx,dword ptr ds:[eax+14]   
00DBF29E     51                 push ecx   
00DBF29F     8B55 08            mov edx,dword ptr ss:[ebp+8]   
00DBF2A2     8B42 10            mov eax,dword ptr ds:[edx+10]   
00DBF2A5     50                 push eax   
00DBF2A6     8B4D F4            mov ecx,dword ptr ss:[ebp-C]   
00DBF2A9     2B4D E0            sub ecx,dword ptr ss:[ebp-20]   
00DBF2AC     FFD1               call ecx   
00DBF2AE     8945 FC            mov dword ptr ss:[ebp-4],eax   
00DBF2B1     EB 46              jmp short 00DBF2F9   
00DBF2B3     8B55 08            mov edx,dword ptr ss:[ebp+8]   
00DBF2B6     833A 01            cmp dword ptr ds:[edx],1   
00DBF2B9     75 3E              jnz short 00DBF2F9   
00DBF2BB     A1 04BBDE00        mov eax,dword ptr ds:[DEBB04]   
00DBF2C0     8B0D 04BBDE00      mov ecx,dword ptr ds:[DEBB04]   
00DBF2C6     8B50 68            mov edx,dword ptr ds:[eax+68]   
00DBF2C9     3351 34            xor edx,dword ptr ds:[ecx+34]   
00DBF2CC     A1 04BBDE00        mov eax,dword ptr ds:[DEBB04]   
00DBF2D1     3350 60            xor edx,dword ptr ds:[eax+60]   
00DBF2D4     8955 DC            mov dword ptr ss:[ebp-24],edx   
00DBF2D7     8B4D 08            mov ecx,dword ptr ss:[ebp+8]   
00DBF2DA     8B51 04            mov edx,dword ptr ds:[ecx+4]   
00DBF2DD     52                 push edx   
00DBF2DE     8B45 08            mov eax,dword ptr ss:[ebp+8]   
00DBF2E1     8B48 08            mov ecx,dword ptr ds:[eax+8]   
00DBF2E4     51                 push ecx   
00DBF2E5     6A 00              push 0   
00DBF2E7     8B55 08            mov edx,dword ptr ss:[ebp+8]   
00DBF2EA     8B42 0C            mov eax,dword ptr ds:[edx+C]   
00DBF2ED     50                 push eax   
00DBF2EE     8B4D F4            mov ecx,dword ptr ss:[ebp-C]   
00DBF2F1     2B4D DC            sub ecx,dword ptr ss:[ebp-24]   
00DBF2F4     FFD1               call ecx  ; Armadill.004010CC   
//设断，中断后取消断点，F7 进入   
//飞向光明之巅  ^Q^   
00DBF2F6     8945 FC            mov dword ptr ss:[ebp-4],eax   
00DBF2F9     8B45 FC            mov eax,dword ptr ss:[ebp-4]   
00DBF2FC     5E                 pop esi   
00DBF2FD     8BE5               mov esp,ebp   
00DBF2FF     5D                 pop ebp   
00DBF300     C3                 retn
```

代码:

```
004010CC     55                 push ebp   
//OEP   
004010CD     8BEC               mov ebp,esp   
004010CF     83EC 44            sub esp,44   
004010D2     56                 push esi   
004010D3     FF15 E0634000      call dword ptr ds:[4063E0] ; kernel32.GetCommandLineA   
004010D9     8BF0               mov esi,eax   
004010DB     8A00               mov al,byte ptr ds:[eax]   
004010DD     3C 22              cmp al,22   
004010DF     75 13              jnz short 004010F4
```

运行 `ImportREC`，选择这个进程。

把 `OEP` 改为 `000010CC`，点 `IT AutoSearch`，点 `Get Import`，`Cut` 掉填充在 DLL 间的无效指针 `FixDump`，正常运行。

Game Over

代码:

```
             ,     _/       /| _.-~/            \_     ,        青春都一晌
          ( /~   /             \~-._ |\          `\\  _/              \   ~\ )          忍把浮名 
    _-~~~-.)  )__/;;,.          \_  //‘     /'_,\   --~   \ ~~-  ,;;\___(  (.-~~~-.        换了破解轻狂 
    `~ _( ,_..--\ (     ,;'' /     ~--   /._`\      /~~//'   /' `~\         ) /--.._, )_  `~
    "  `~"  "      `"      /~'`\     \\~~\      "      "   "~'  ""
          fly [CUG] 
        http://unpack.cn
      http://www.unpack.cn
        2007.09.16 00:00
```
