
```
0040DB47     59                 pop ecx   

0040DB48     5A                 pop edx   

0040DB49     E8 EF000000        call 0040DC3D   

0040DB4E     8985 AE0B0000      mov dword ptr ss:\[ebp+BAE\],eax   

0040DB54     8B32               mov esi,dword ptr ds:\[edx\]   

0040DB56     890A               mov dword ptr ds:\[edx\],ecx   

0040DB58     8B7A 10            mov edi,dword ptr ds:\[edx+10\]   

0040DB5B     894A 10            mov dword ptr ds:\[edx+10\],ecx   

0040DB5E     0BF6               or esi,esi   

0040DB60     75 02              jnz short 0040DB64   

0040DB64     03B5 2B120000      add esi,dword ptr ss:\[ebp+122B\]   

0040DB6A     03BD 2B120000      add edi,dword ptr ss:\[ebp+122B\]   

0040DB70     8B06               mov eax,dword ptr ds:\[esi\]   

0040DB72     0BC0               or eax,eax   

0040DB74     74 62              je short 0040DBD8   

                                ====>G 0040DBD8 跳出下面的循环！   

0040DB76     890E               mov dword ptr ds:\[esi\],ecx   

0040DB78     79 05              jns short 0040DB7F   

0040DB7F     0385 2B120000      add eax,dword ptr ss:\[ebp+122B\]   

  …… ……  省 略  …… ……   

0040DBD6   ^ EB 98              jmp short 0040DB70   

                                ====>发现 0040DB74 可以跳出循环！   

0040DBD8     83C2 14            add edx,14   

0040DBDB   ^ E9 02FFFFFF        jmp 0040DAE2   

                                ====>发现 0040DAE7 可以跳出循环！   

0040DBE0    /EB 01             jmp short 0040DBE3   

0040DBE3     8B85 47120000      mov eax,dword ptr ss:\[ebp+1247\]   

0040DBE9     EB 02              jmp short 0040DBED   

0040DBED     3385 9F120000      xor eax,dword ptr ss:\[ebp+129F\]   

                                ====>EAX=000010CC   

0040DBF3     EB 02              jmp short 0040DBF7   

0040DBF7     0385 A7120000      add eax,dword ptr ss:\[ebp+12A7\]   

                                ====>EAX=000010CC + 00400000=004010CC   这就是 OEP 值  :-)   

0040DBFD     EB 01              jmp short 0040DC00   

0040DC00     894424 1C          mov dword ptr ss:\[esp+1C\],eax   

0040DC04     EB 03              jmp short 0040DC09   

0040DC09     8DBD 3D0C0000      lea edi,dword ptr ss:\[ebp+C3D\]   

0040DC0F     EB 03              jmp short 0040DC14   

0040DC14     B9 07070000        mov ecx,707   

0040DC19     EB 01              jmp short 0040DC1C   

0040DC1C     32C0               xor al,al   

0040DC1E     EB 02              jmp short 0040DC22   

0040DC22     F3:AA              rep stos byte ptr es:\[edi\]   

                                ====>清除那些“警告信息”   

0040DC24     EB 01              jmp short 0040DC27   

0040DC27     8BFD               mov edi,ebp   

0040DC29     EB 02              jmp short 0040DC2D   

0040DC2D     B9 350C0000        mov ecx,0C35   

0040DC32     EB 01              jmp short 0040DC35   

0040DC35     F3:AA              rep stos byte ptr es:\[edi\]   

0040DC37     61                 popad   

                                ====>很高兴看见这个 POPAD   :-D   

0040DC38    /EB 01             jmp short 0040DC3B   

0040DC3B   ^FFE0               jmp eax   

                                ====>飞向光明之巅！   

———————————————————————   

004010CC     55                 push ebp   

                                ====>在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程   

004010CD     8BEC               mov ebp,esp   

004010CF     83EC 44            sub esp,44   

004010D2     56                 push esi   

004010D3     FF15 E4634000      call dword ptr ds:\[4063E4\] ; kernel32.GetCommandLineA   

004010D9     8BF0               mov esi,eax   

004010DB     8A00               mov al,byte ptr ds:\[eax\]   

004010DD     3C 22              cmp al,22   

004010DF     75 1B              jnz short 004010FC   

004010E1     56                 push esi   

004010E2     FF15 F4644000      call dword ptr ds:\[4064F4\] ; USER32.CharNextA   

———————————————————————   

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `000010CC`，点 `IT AutoSearch`，点“Get Import”，   

有几个无效函数手动修复之，`FixDump`，正常运行！`60K` -> `72K`  用 `FileScan` 优化后是 `50.5K`。   

用 `Unpes.exe` 自动脱壳后是 `60K`。   

—————————————————————————————————   

        ,     \_/   
      /| \_.-~/           \_     ,        青春都一饷  
     ( /~   /              ~-.\_ |  
     \`\\  \_/                ~ )           忍把浮名  
  _-~~~-.)  )\_\_/;;,.        \_  //’   
 /'\_,   --~    ~~~-  ,;;\_\_\_( (.-~~~-.       换了破解轻狂  
\`~ \_ ( ,\_..-- (      ,;'' /    ~--   /.\_\`  
  /~~//'   /' \`~         ) /--..\_, )\_  \`~    
  "  \`~"  "      \`"      /~'\`    \`\\~~    
              "      "   "~'  ""  

        Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]  

             2003-10-12  20:20  

 4dd466d0e1e0cc123afbe504da2462ca
```
