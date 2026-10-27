
# 汇编转储

```
004134BB: 657465                    jz 00413523
004134BE: 61                        popad
004134BF: 7365                      jnb 00413526
004134C1: 2E636F6D                  arpl word ptr cs:[edi+6D],bp
004134C5: 200D0A202020              and byte ptr [2020200A],cl
004134CB: 2020                      and byte ptr [eax],ah
004134CD: 2020                      and byte ptr [eax],ah
004134CF: 2020                      and byte ptr [eax],ah
004134D1: 2020                      and byte ptr [eax],ah
004134D3: 2020                      and byte ptr [eax],ah
004134D5: 2020                      and byte ptr [eax],ah
004134D7: 2020                      and byte ptr [eax],ah
004134D9: 2020                      and byte ptr [eax],ah
004134DB: 48                        dec eax
004134DC: 6F                        outsd dx,ds:[esi]
004134DD: 6D                        insd es:[edi],dx
004134DE: 657061                    jo 00413542
004134E1: 6765203A                  and byte ptr gs:[bp+si],bh
004134E5: 206874                    and byte ptr [eax+74],ch
004134E8: 7470                      jz 0041355A
004134EA: 3A2F                      cmp ch,byte ptr [edi]
004134EC: 2F                        das
004134ED: 667364                    jnb 00413554
004134F0: 622E                      bound ebp,dword ptr [esi]
004134F2: 7965                      jns 00413559
004134F4: 61                        popad
004134F5: 682E6E6574                push 74656E2E
004134FA: 2020                      and byte ptr [eax],ah
004134FC: 2020                      and byte ptr [eax],ah
004134FE: 2020                      and byte ptr [eax],ah
00413500: 2020                      and byte ptr [eax],ah
00413502: 2020                      and byte ptr [eax],ah
00413504: 2020                      and byte ptr [eax],ah
00413506: 200D0A0D0A00              and byte ptr [000A0D0A],cl
0041350C: 0000                      add byte ptr [eax],al
0041350E: 0000                      add byte ptr [eax],al
00413510: 0000                      add byte ptr [eax],al
00413512: 0000                      add byte ptr [eax],al
00413514: 0000                      add byte ptr [eax],al
00413516: 0000                      add byte ptr [eax],al
00413518: 0000                      add byte ptr [eax],al
0041351A: 0000                      add byte ptr [eax],al
0041351C: 0000                      add byte ptr [eax],al
0041351E: 0000                      add byte ptr [eax],al
00413520: 0000                      add byte ptr [eax],al
00413522: 0000                      add byte ptr [eax],al
00413524: 0000                      add byte ptr [eax],al
00413526: 0000                      add byte ptr [eax],al
00413528: 0000                      add byte ptr [eax],al
0041352A: 0000                      add byte ptr [eax],al
0041352C: 0000                      add byte ptr [eax],al
0041352E: 0000                      add byte ptr [eax],al
00413530: 0000                      add byte ptr [eax],al
00413532: 0000                      add byte ptr [eax],al
00413534: 0000                      add byte ptr [eax],al
00413536: 0000                      add byte ptr [eax],al
00413538: 0000                      add byte ptr [eax],al
0041353A: 0000                      add byte ptr [eax],al
0041353C: 0000                      add byte ptr [eax],al
0041353E: 0000                      add byte ptr [eax],al
00413540: 0000                      add byte ptr [eax],al
00413542: 0000                      add byte ptr [eax],al
00413544: 49                        dec ecx
00413545: 6E                        outsb dx,ds:[esi]
00413546: 7661                      jbe 004135A9
00413548: 6C                        insb es:[edi],dx
00413549: 6964202050617373          imul esp,dword ptr [eax+20],73736150
00413551: 57                        push edi
00413552: 6F                        outsd dx,ds:[esi]
00413553: 7264                      jb 004135B9
00413555: 0000                      add byte ptr [eax],al
00413557: 0000                      add byte ptr [eax],al
00413559: 0000                      add byte ptr [eax],al
0041355B: 0000                      add byte ptr [eax],al
0041355D: 0000                      add byte ptr [eax],al
0041355F: 0000                      add byte ptr [eax],al
00413561: 0000                      add byte ptr [eax],al
00413563: 0000                      add byte ptr [eax],al
00413565: 0000                      add byte ptr [eax],al
00413567: 0000                      add byte ptr [eax],al
00413569: 0000                      add byte ptr [eax],al
0041356B: 0000                      add byte ptr [eax],al
0041356D: 0000                      add byte ptr [eax],al
0041356F: 0000                      add byte ptr [eax],al
00413571: 0000                      add byte ptr [eax],al
00413573: 0000                      add byte ptr [eax],al
00413575: 0000                      add byte ptr [eax],al
00413577: 0000                      add byte ptr [eax],al
00413579: 0000                      add byte ptr [eax],al
0041357B: 0000                      add byte ptr [eax],al
0041357D: 0000                      add byte ptr [eax],al
0041357F: 0000                      add byte ptr [eax],al
00413581: 0000                      add byte ptr [eax],al
00413583: 0000                      add byte ptr [eax],al
00413585: 0000                      add byte ptr [eax],al
00413587: 0000                      add byte ptr [eax],al
00413589: 0000                      add byte ptr [eax],al
0041358B: 0000                      add byte ptr [eax],al
0041358D: 0000                      add byte ptr [eax],al
0041358F: 0000                      add byte ptr [eax],al
00413591: 0000                      add byte ptr [eax],al
00413593: 0000                      add byte ptr [eax],al
00413595: 0000                      add byte ptr [eax],al
00413597: 0000                      add byte ptr [eax],al
00413599: 0000                      add byte ptr [eax],al
0041359B: 0000                      add byte ptr [eax],al
0041359D: 0000                      add byte ptr [eax],al
0041359F: 0000                      add byte ptr [eax],al
004135A1: 0000                      add byte ptr [eax],al
004135A3: 0000                      add byte ptr [eax],al
004135A5: 0000                      add byte ptr [eax],al
004135A7: 005265                    add byte ptr [edx+65],dl
004135AA: 6769737465722053          imul esi,dword ptr [bp+di+74],53207265
004135B2: 7563                      jnz 00413617
004135B4: 636573                    arpl word ptr [ebp+73],sp
004135B7: 7366                      jnb 0041361F
004135B9: 756C                      jnz 00413627
004135BB: 6C                        insb es:[edi],dx
004135BC: 7900                      jns 004135BE
004135BE: 0000                      add byte ptr [eax],al
004135C0: 0000                      add byte ptr [eax],al
004135C2: 0000                      add byte ptr [eax],al
004135C4: 0000                      add byte ptr [eax],al
004135C6: 0000                      add byte ptr [eax],al
004135C8: 0000                      add byte ptr [eax],al
004135CA: 0000                      add byte ptr [eax],al
004135CC: 0000                      add byte ptr [eax],al
004135CE: 0000                      add byte ptr [eax],al
004135D0: 0000                      add byte ptr [eax],al
004135D2: 0000                      add byte ptr [eax],al
004135D4: 0000                      add byte ptr [eax],al
004135D6: 0000                      add byte ptr [eax],al
004135D8: 0000                      add byte ptr [eax],al
004135DA: 0000                      add byte ptr [eax],al
004135DC: 0000                      add byte ptr [eax],al
004135DE: 0000                      add byte ptr [eax],al
```
