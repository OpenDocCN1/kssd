
### 代码段

```
004C1F68   .  E8 9BBAFAFF   CALL <mp4.GetHandle>                   ;  得到窗口句柄  
004C1F6D   .  50            PUSH EAX                               ; |hOwner  
004C1F6E   .  E8 4954F4FF   CALL <JMP.&user32.MessageBoxA>        ; \\MessageBoxA  
004C1F73   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-0x4\]  
004C1F76   .  E8 2520FCFF   CALL <mp4.Forms.TCustomForm.Close>    ;  关闭注册窗口  
004C1F7B   .  33C0          XOR EAX,EAX  
004C1F7D   .  5A            POP EDX  
004C1F7E   .  59            POP ECX  
004C1F7F   .  59            POP ECX  
004C1F80   .  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  
004C1F83   .  EB 36         JMP SHORT mp4.004C1FBB  
004C1F85   .^ E9 161AF4FF   JMP mp4.004039A0  
004C1F8A   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-0x4\]  
004C1F8D   .  E8 0E20FCFF   CALL <mp4.Forms.TCustomForm.Close>  
004C1F92   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-0x4\]  
004C1F95   .  E8 1AFDFFFF   CALL mp4.004C1CB4  
004C1F9A   .  E8 691DF4FF   CALL mp4.00403D08  
004C1F9F   .  EB 1A         JMP SHORT mp4.004C1FBB  
004C1FA1   >  6A 40         PUSH 0x40                                ;  弹出错误注册码提示框  
004C1FA3   .  68 C4204C00   PUSH mp4.004C20C4                      ;  ASCII "Register"  
004C1FA8   .  68 D0204C00   PUSH mp4.004C20D0  
004C1FAD   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-0x4\]  
004C1FB0   .  E8 53BAFAFF   CALL <mp4.GetHandle>                   ;  得到窗口句柄  
004C1FB5   .  50            PUSH EAX                               ; |hOwner  
004C1FB6   .  E8 0154F4FF   CALL <JMP.&user32.MessageBoxA>        ; \\MessageBoxA  
004C1FBB   >  33C0          XOR EAX,EAX  
```

进入关键 `004C1EB9`
```
004C1EB9   .  E8 B2FEFFFF   CALL mp4.004C1D70  
004C1D70  /$  55            PUSH EBP                               ;  关键 CALL 入口  
004C1D71  |.  8BEC          MOV EBP,ESP  
004C1D73  |.  B9 04000000   MOV ECX,0x4  
004C1D78  |>  6A 00         /PUSH 0x0  
004C1D7A  |.  6A 00         |PUSH 0x0  
004C1D7C  |.  49            |DEC ECX  
004C1D7D  |.^ 75 F9         \\JNZ SHORT mp4.004C1D78  
004C1D7F  |.  51            PUSH ECX  
004C1D80  |.  53            PUSH EBX  
004C1D81  |.  56            PUSH ESI  
004C1D82  |.  8BF0          MOV ESI,EAX  
004C1D84  |.  33C0          XOR EAX,EAX  
004C1D86  |.  55            PUSH EBP  
004C1D87  |.  68 871E4C00   PUSH mp4.004C1E87  
004C1D8C  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
004C1D8F  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
004C1D92  |.  8D55 F8       LEA EDX,DWORD PTR SS:\[EBP-0x8\]       ;  Edit_RegCode  
004C1D95  |.  8B86 08030000 MOV EAX,DWORD PTR DS:\[ESI+0x308\]  
004C1D9B  |.  E8 8054FAFF   CALL <mp4.GetText>                    ;  获取用户输入的注册码明文  
004C1DA0  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-0x8\]  
004C1DA3  |.  8D55 FC       LEA EDX,DWORD PTR SS:\[EBP-0x4\]  
004C1DA6  |.  E8 296DF4FF   CALL <mp4.SysUtils.Trim>              ;  去掉注册码空格  
004C1DAB  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-0x4\]  
004C1DAE  |.  50            PUSH EAX  
004C1DAF  |.  8D55 EC       LEA EDX,DWORD PTR SS:\[EBP-0x14\]      ;  Edit_RegName  
004C1DB2  |.  8B86 04030000 MOV EAX,DWORD PTR DS:\[ESI+0x304\]  
004C1DB8  |.  E8 6354FAFF   CALL <mp4.GetText>                    ;  获取用户名  
004C1DBD  |.  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP-0x14\]  
004C1DC0  |.  8D55 F0       LEA EDX,DWORD PTR SS:\[EBP-0x10\]  
004C1DC3  |.  E8 0C6DF4FF   CALL <mp4.SysUtils.Trim>              ;  去掉用户名中包含的空格  
004C1DC8  |.  8B55 F0       MOV EDX,DWORD PTR SS:\[EBP-0x10\]  
004C1DCB  |.  8D4D F4       LEA ECX,DWORD PTR SS:\[EBP-0xC\]  
004C1DCE  |.  8BC6          MOV EAX,ESI  
004C1DD0  |.  E8 EBFCFFFF   CALL <mp4.MakeKey>                    ;  生成注册码  
004C1DD5  |.  8B55 F4       MOV EDX,DWORD PTR SS:\[EBP-0xC\]  
004C1DD8  |.  58            POP EAX  
004C1DD9  |.  E8 5A29F4FF   CALL <mp4.System.LStrCmp>             ;  注册码比较(关键)  
004C1DDE  |.  75 52         JNZ SHORT mp4.004C1E32  
004C1DE0  |.  B3 01         MOV BL,0x1  
004C1DE2  |.  8D55 E4       LEA EDX,DWORD PTR SS:\[EBP-0x1C\]      ;  Edit_RegName  
004C1DE5  |.  8B86 04030000 MOV EAX,DWORD PTR DS:\[ESI+0x304\]  
004C1DEB  |.  E8 3054FAFF   CALL <mp4.GetText>                    ;  获取用户名  
004C1DF0  |.  8B45 E4       MOV EAX,DWORD PTR SS:\[EBP-0x1C\]  
004C1DF3  |.  8D55 E8       LEA EDX,DWORD PTR SS:\[EBP-0x18\]  
004C1DF6  |.  E8 D96CF4FF   CALL <mp4.SysUtils.Trim>  
004C1DFB  |.  8B55 E8       MOV EDX,DWORD PTR SS:\[EBP-0x18\]  
004C1DFE  |.  8D86 1C030000 LEA EAX,DWORD PTR DS:\[ESI+0x31C\]  
004C1E04  |.  E8 5B25F4FF   CALL <mp4.System.LStrAsg>             ;  System.LStrAsg  
004C1E09  |.  8D55 DC       LEA EDX,DWORD PTR SS:\[EBP-0x24\]      ;  Edit_RegCode  
004C1E0C  |.  8B86 08030000 MOV EAX,DWORD PTR DS:\[ESI+0x308\]  
004C1E12  |.  E8 0954FAFF   CALL <mp4.GetText>                    ;  获取注册码  
004C1E17  |.  8B45 DC       MOV EAX,DWORD PTR SS:\[EBP-0x24\]  
004C1E1A  |.  8D55 E0       LEA EDX,DWORD PTR SS:\[EBP-0x20\]  
004C1E1D  |.  E8 B26CF4FF   CALL <mp4.SysUtils.Trim>  
004C1E22  |.  8B55 E0       MOV EDX,DWORD PTR SS:\[EBP-0x20\]  
004C1E25  |.  8D86 20030000 LEA EAX,DWORD PTR DS:\[ESI+0x320\]  
004C1E2B  |.  E8 3425F4FF   CALL <mp4.System.LStrAsg>  
004C1E30  |.  EB 02         JMP SHORT mp4.004C1E34  
```

这里看到 `004C1DD0` 处 `CALL <mp4.MakeKey>` 生成注册码，再与输入的伪码进行比较，然后将比较结果返回，进入 `004C1DD0` 看看。

### `<mp4.Make>`

```
<mp4.Make>/$  55            PUSH EBP                               ;  生成注册码 CALL  
004C1AC1  |.  8BEC          MOV EBP,ESP  
004C1AC3  |.  51            PUSH ECX  
004C1AC4  |.  B9 04000000   MOV ECX,0x4  
004C1AC9  |>  6A 00         /PUSH 0x0  
004C1ACB  |.  6A 00         |PUSH 0x0  
004C1ACD  |.  49            |DEC ECX  
004C1ACE  |.^ 75 F9         \\JNZ SHORT mp4.004C1AC9             ;  准备堆栈变量空间  
004C1AD0  |.  51            PUSH ECX  
004C1AD1  |.  874D FC       XCHG DWORD PTR SS:\[EBP-0x4\],ECX  
004C1AD4  |.  53            PUSH EBX  
004C1AD5  |.  56            PUSH ESI  
004C1AD6  |.  57            PUSH EDI  
004C1AD7  |.  8BF9          MOV EDI,ECX  
004C1AD9  |.  8955 FC       MOV DWORD PTR SS:\[EBP-0x4\],EDX  
004C1ADC  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-0x4\]  
004C1ADF  |.  E8 F82CF4FF   CALL <mp4.System.LStrAddRef>  
004C1AE4  |.  33C0          XOR EAX,EAX  
004C1AE6  |.  55            PUSH EBP  
004C1AE7  |.  68 811C4C00   PUSH mp4.004C1C81  
004C1AEC  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
004C1AEF  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
004C1AF2  |.  8BC7          MOV EAX,EDI  
004C1AF4  |.  E8 1728F4FF   CALL <mp4.System.LStrClr>  
004C1AF9  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-0x4\]  
004C1AFC  |.  E8 EB2AF4FF   CALL <mp4.System.LStrLen>           ;  计算用户名长度  
004C1B01  |.  8BF0          MOV ESI,EAX  
004C1B03  |.  85F6          TEST ESI,ESI                         ;  用户名为空则跳  
004C1B05  |.  7E 26         JLE SHORT mp4.004C1B2D  
004C1B07  |.  BB 01000000   MOV EBX,0x1                          ;  i=1  
004C1B0C  |>  8D4D EC       /LEA ECX,DWORD PTR SS:\[EBP-0x14\]  
004C1B0F  |.  8B45 FC       |MOV EAX,DWORD PTR SS:\[EBP-0x4\]  
004C1B12  |.  0FB64418 FF   |MOVZX EAX,BYTE PTR DS:\[EAX+EBX-0x1\] ;  name\[i-1\]  
004C1B17  |.  33D2          |XOR EDX,EDX  
004C1B19  |.  E8 3273F4FF   |CALL <mp4.IntToHex>                 ;  IntToHex  
004C1B1E  |.  8B55 EC       |MOV EDX,DWORD PTR SS:\[EBP-0x14\]   ;  将用户名转换成 16 进制  
004C1B21  |.  8D45 F8       |LEA EAX,DWORD PTR SS:\[EBP-0x8\]  
004C1B24  |.  E8 CB2AF4FF   |CALL <mp4.System.LStrCat>          ;  转换后的 16 进制字符串存入缓冲区  
004C1B29  |.  43            |INC EBX  
```
