
```
 0043E14F     FF35 25FC4100      push dword ptr ds:[41FC25]
 0043E155     890C24             mov dword ptr ss:[esp],ecx
 0043E158     8B3C24             mov edi,dword ptr ss:[esp]
 0043E15B     8F05 B9FC4100      pop dword ptr ds:[41FCB9]
 0043E161     8F05 19FE4100      pop dword ptr ds:[41FE19]
 0043E167     8905 89FD4100      mov dword ptr ds:[41FD89],eax
 0043E16D     FF35 89FD4100      push dword ptr ds:[41FD89]
 0043E173     57                 push edi
 0043E174     BF 19FE4100        mov edi,UnPacked.0041FE19
 0043E179     8BC7               mov eax,edi
 0043E17B     5F                 pop edi
 0043E17C     8B08               mov ecx,dword ptr ds:[eax]
 0043E17E     8F05 95FC4100      pop dword ptr ds:[41FC95]
 0043E184     8B05 95FC4100      mov eax,dword ptr ds:[41FC95]
 0043E18A     53                 push ebx
 0043E18B     90                 nop
 0043E18C     90                 nop
 0043E18D     8F05 5DFE4100      pop dword ptr ds:[41FE5D]
 0043E193     FF35 5DFE4100      push dword ptr ds:[41FE5D]
 0043E199     890C24             mov dword ptr ss:[esp],ecx
 0043E19C     893D 91FE4100      mov dword ptr ds:[41FE91],edi
 0043E1A2     FF35 91FE4100      push dword ptr ds:[41FE91]
 0043E1A8     8F05 81FC4100      pop dword ptr ds:[41FC81]
 0043E1AE     891D 89FE4100      mov dword ptr ds:[41FE89],ebx
 0043E1B4     FF35 89FE4100      push dword ptr ds:[41FE89]
 0043E1BA     68 81FC4100        push UnPacked.0041FC81
 0043E1BF     5B                 pop ebx
 0043E1C0     8B0B               mov ecx,dword ptr ds:[ebx]
 0043E1C2     8F05 C9FC4100      pop dword ptr ds:[41FCC9]
 0043E1C8     8B1D C9FC4100      mov ebx,dword ptr ds:[41FCC9]
 0043E1CE     57                 push edi
 0043E1CF     890424             mov dword ptr ss:[esp],eax
 0043E1D2     890C24             mov dword ptr ss:[esp],ecx
 0043E1D5     8B0424             mov eax,dword ptr ss:[esp]
 0043E1D8     90                 nop
 0043E1D9     90                 nop
 0043E1DA     8F05 D5FD4100      pop dword ptr ds:[41FDD5]
 0043E1E0     8B0C24             mov ecx,dword ptr ss:[esp]
 0043E1E3     8F05 4DFC4100      pop dword ptr ds:[41FC4D]
 0043E1E9     50                 push eax
 0043E1EA     891424             mov dword ptr ss:[esp],edx
 0043E1ED     8F05 BDFE4100      pop dword ptr ds:[41FEBD]
 0043E1F3     FF35 BDFE4100      push dword ptr ds:[41FEBD]
 0043E1F9     51                 push ecx
 0043E1FA     B9 DDFD4100        mov ecx,UnPacked.0041FDDD
 0043E1FF     8939               mov dword ptr ds:[ecx],edi
 0043E201     59                 pop ecx
 0043E202     FF35 DDFD4100      push dword ptr ds:[41FDDD]
 0043E208     C705 A9FE4100 6055> mov dword ptr ds:[41FEA9],UnPacked.0040>
 0043E212     FF35 A9FE4100      push dword ptr ds:[41FEA9]
 0043E218     8B3C24             mov edi,dword ptr ss:[esp]
 0043E21B     8F05 95FD4100      pop dword ptr ds:[41FD95]
 0043E221     891D 29FD4100      mov dword ptr ds:[41FD29],ebx
 0043E227     90                 nop
 0043E228     90                 nop
 0043E229     FF35 29FD4100      push dword ptr ds:[41FD29]
 0043E22F     8BDF               mov ebx,edi
 0043E231     8BD3               mov edx,ebx
 0043E233     5B                 pop ebx
 0043E234     8F05 E9FE4100      pop dword ptr ds:[41FEE9]
 0043E23A     8B3D E9FE4100      mov edi,dword ptr ds:[41FEE9]
 0043E240     52                 push edx
 0043E241     891C24             mov dword ptr ss:[esp],ebx
 0043E244     68 9DFE4100        push UnPacked.0041FE9D
 0043E249     5B                 pop ebx
 0043E24A     8913               mov dword ptr ds:[ebx],edx
 0043E24C     8B1C24             mov ebx,dword ptr ss:[esp]
 0043E24F     8F05 49FE4100      pop dword ptr ds:[41FE49]
 0043E255     8B1424             mov edx,dword ptr ss:[esp]
 0043E258     8F05 69FD4100      pop dword ptr ds:[41FD69]
 0043E25E     FF15 9DFE4100      call dword ptr ds:[41FE9D]
 0043E264     8965 E8            mov dword ptr ss:[ebp-18],esp
 0043E267     8925 C5FD4100      mov dword ptr ds:[41FDC5],esp
 0043E26D     891D 21FD4100      mov dword ptr ds:[41FD21],ebx
 0043E273     FF35 21FD4100      push dword ptr ds:[41FD21]
 0043E279     90                 nop
 0043E27A     90                 nop
 0043E27B     - E9 DC4DFCFF      jmp UnPacked.0040305C                   ; 直接跳向 FOEP
```

复制以上修改，保存为另外一个文件。

打开 `LordPE`，修正 `OEP` 为 `3E000`，只选上“验证 PE”重建 `PE`。到此脱壳完成。

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
