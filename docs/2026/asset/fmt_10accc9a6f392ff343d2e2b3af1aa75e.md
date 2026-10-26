
```
0042D771     E8 EA64FDFF       call x.00403C60                            ; 与上面字符串相连
0042D776     90                nop
0042D777     90                nop
0042D778     90                nop
0042D779     90                nop
0042D77A     90                nop
0042D77B     90                nop
0042D77C     6A 21             push 21
0042D77E     68 14784600       push x.00467814                            ; ASCII "AF#$&^%$$##@@#GF><MM<<:fdgdsewf::"
0042D783     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D787     E8 D464FDFF       call x.00403C60                            ; 与上面字符串相连
0042D78C     90                nop
0042D78D     90                nop
0042D78E     90                nop
0042D78F     6A 1F             push 1F
0042D791     68 F4774600       push x.004677F4                            ; ASCII ":@dd#$^DS@#$$%&DSGA[]][]{}reT{{"
0042D796     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D79A     E8 C164FDFF       call x.00403C60                            ; 与上面字符串相连
0042D79F     90                nop
0042D7A0     90                nop
0042D7A1     90                nop
0042D7A2     90                nop
0042D7A3     90                nop
0042D7A4     90                nop
0042D7A5     6A 40             push 40
0042D7A7     68 B0774600       push x.004677B0                            ; ASCII "ess###sf|~~@!#``degfsdfVCNM<KJFHDYMGNsr354375uhyfgdsbnhgnfwe9257"
0042D7AC     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D7B0     E8 AB64FDFF       call x.00403C60                            ; 与上面字符串相连
0042D7B5     90                nop
0042D7B6     90                nop
0042D7B7     90                nop
0042D7B8     90                nop
0042D7B9     6A 22             push 22
0042D7BB     68 88774600       push x.00467788                            ; ASCII "453242fdljpewrpwtew345767i8u)\*\*^#%"
0042D7C0     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D7C4     E8 9764FDFF       call x.00403C60                            ; 与上面字符串相连
0042D7C9     90                nop
0042D7CA     90                nop
0042D7CB     90                nop
0042D7CC     90                nop
0042D7CD     90                nop
0042D7CE     90                nop
0042D7CF     6A 1E             push 1E
0042D7D1     68 68774600       push x.00467768                            ; ASCII "!#@\_+#$%#@#%#RWEQSADdsfg$#$DS2"
0042D7D6     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D7DA     E8 8164FDFF       call x.00403C60                            ; 与上面字符串相连
0042D7DF     90                nop
0042D7E0     90                nop
0042D7E1     90                nop
0042D7E2     6A 1F             push 1F
0042D7E4     68 48774600       push x.00467748                            ; ASCII "fdsg#^DSDF@fhlpoyui90932sag87ds"
0042D7E9     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D7ED     E8 6E64FDFF       call x.00403C60                            ; 与上面字符串相连
0042D7F2     90                nop
0042D7F3     90                nop
0042D7F4     90                nop
0042D7F5     6A 21             push 21
0042D7F7     68 24774600       push x.00467724                            ; ASCII "slkl34vo3245dsfa#^#%$#@#@!dg$#%54"
0042D7FC     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D800     E8 5B64FDFF       call x.00403C60                            ; 与上面字符串相连
0042D805     90                nop
0042D806     90                nop
0042D807     90                nop
0042D808     6A 20             push 20
0042D80A     68 00774600       push x.00467700                            ; ASCII "43535234|~~@!#``degfsdfVCNM<KJFH"
0042D80F     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D813     E8 4864FDFF       call x.00403C60                            ; 与上面字符串相连
0042D818     90                nop
0042D819     90                nop
0042D81A     90                nop
0042D81B     90                nop
0042D81C     6A 20             push 20
0042D81E     68 DC764600       push x.004676DC                            ; ASCII "DYMGNsr354375uhyfgdsbnhgnfwe9257"
0042D823     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D827     E8 3464FDFF       call x.00403C60                            ; 与上面字符串相连
......
0042D841     8B7424 48         mov esi,dword ptr ss:[esp+48]
0042D845     90                nop
0042D846     90                nop
0042D847     90                nop
0042D848     85ED              test ebp,ebp
0042D84A     0F8C 1D040000     jl x.0042DC6D
0042D850     3B6E F4           cmp ebp,dword ptr ds:[esi-C]
0042D853     0F8F 14040000     jg x.0042DC6D
0042D859     0FBE042E          movsx eax,byte ptr ds:[esi+ebp]           ; 取用户ID字符的ASCII值
0042D85D     03C5              add eax,ebp                               ; ebp为计数器
0042D85F     99                cdq
0042D860     B9 1A000000       mov ecx,1A
0042D865     F7F9              idiv ecx
0042D867     80C2 41           add dl,41
0042D86A     885424 24         mov byte ptr ss:[esp+24],dl               ; 运算结果
0042D86E     90                nop
0042D86F     90                nop
0042D870     90                nop
0042D871     8B5424 24         mov edx,dword ptr ss:[esp+24]
0042D875     6A 01             push 1
0042D877     52                push edx
0042D878     8D4C24 20         lea ecx,dword ptr ss:[esp+20]
0042D87C     E8 EFC9FFFF       call x.0042A270                           ; 存储结果
0042D881     C64424 3C 03      mov byte ptr ss:[esp+3C],3
0042D886     90                nop
0042D887     90                nop
0042D888     90                nop
0042D889     55                push ebp
0042D88A     8D4424 2C         lea eax,dword ptr ss:[esp+2C]
0042D88E     50                push eax
0042D88F     8D4C24 50         lea ecx,dword ptr ss:[esp+50]
0042D893     E8 2887FDFF       call x.00405FC0
0042D898     8B00              mov eax,dword ptr ds:[eax]
0042D89A     8B48 F4           mov ecx,dword ptr ds:[eax-C]
0042D89D     51                push ecx
0042D89E     50                push eax
0042D89F     8D4C24 20         lea ecx,dword ptr ss:[esp+20]
0042D8A3     C64424 44 04      mov byte ptr ss:[esp+44],4
0042D8A8     E8 B363FDFF       call x.00403C60                           ; 取的字符前的子字符串与运算结果连成新串
0042D8AD     8B4424 28         mov eax,dword ptr ss:[esp+28]
0042D8B1     83C0 F0           add eax,-10
0042D8B4     C64424 3C 03      mov byte ptr ss:[esp+3C],3
0042D8B9     8D48 0C           lea ecx,dword ptr ds:[eax+C]
0042D8BC     83CA FF           or edx,FFFFFFFF
0042D8BF     F0:0FC111         lock xadd dword ptr ds:[ecx],edx
0042D8C3     4A                dec edx
0042D8C4     85D2              test edx,edx
0042D8C6     7F 08             jg short x.0042D8D0
0042D8C8     8B08              mov ecx,dword ptr ds:[eax]
0042D8CA     8B11              mov edx,dword ptr ds:[ecx]
0042D8CC     50                push eax
0042D8CD     FF52 04           call dword ptr ds:[edx+4]
0042D8D0     90                nop
0042D8D1     90                nop
0042D8D2     90                nop
0042D8D3     8BC5              mov eax,ebp
0042D8D5     99                cdq
0042D8D6     B9 03000000       mov ecx,3
0042D8DB     F7F9              idiv ecx
0042D8DD     85D2              test edx,edx
0042D8DF     75 79             jnz short x.0042D95A                      ; 取字符的位数是否为3的倍数
0042D8E1     90                nop
0042D8E2     90                nop
0042D8E3     90                nop
0042D8E4     8B5424 18         mov edx,dword ptr ss:[esp+18]
0042D8E8     8B72 F4           mov esi,dword ptr ds:[edx-C]
0042D8EB     90                nop
0042D8EC     90                nop
0042D8ED     90                nop
0042D8EE     8B5424 18         mov edx,dword ptr ss:[esp+18]
0042D8F2     8B42 FC           mov eax,dword ptr ds:[edx-4]
0042D8F5     B9 01000000       mov ecx,1
0042D8FA     2BC8              sub ecx,eax
0042D8FC     8B42 F8           mov eax,dword ptr ds:[edx-8]
```
