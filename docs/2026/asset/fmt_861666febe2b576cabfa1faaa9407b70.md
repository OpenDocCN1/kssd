
# 象棋世家

## RSA124+变形MD5+BASE64注册算法分析

仅用于技术交流!!!!!!!!

通过分析这个软件：

1. 复习了`MD5`
2. 复习了`RSA`
3. 复习了`Base64`

Feel sth. bad![](https://www.kanxue.com/images/smilies/biggrin.gif)![](https://www.kanxue.com/images/smilies/biggrin.gif)![](https://www.kanxue.com/images/smilies/biggrin.gif),

程序启动的时候从注册表的`HKCU\Software\www.ai-master.com\SGUI\sn`下读取键值，然后通过`Base64`解码得到注册码。也就是说这儿的值是正确的注册码经`Base64`编码后的。

程序启动的时候就要输入注册码，否则无法使用。注册码共36位，经过分析后得知前5位是单独的，后31位作为`RSA`参数中的`M`。前5位经过变形的`MD5`运算，得到128位的`Hash`，然后对这个`Hash`再进行处理得到一个中间整数，我称它为`TempInt2`。用`RSA`对注册码后面的31位序列号进行加密，得到一个中间整数，我称它为`TempInt1`，如果`TempInt1 == TempInt2`，那么就注册成功。

下面我们就具体看看吧，下面就是整个验证流程，其中有些函数日名子是我在OD中分析出函数日功能后在IDA中改的。

```
0041DA20 <>/$  64:A1 00000000     mov eax,dword ptr fs:[0]                       ;  Check
0041DA26   |.  6A FF              push -1
0041DA28   |.  68 8B744500        push <SGUI.loc_45748B>
0041DA2D   |.  50                 push eax
0041DA2E   |.  64:8925 00000000   mov dword ptr fs:[0],esp
0041DA35   |.  81EC 90010000      sub esp,190
0041DA3B   |.  53                 push ebx
0041DA3C   |.  56                 push esi
0041DA3D   |.  57                 push edi
0041DA3E   |.  8BD9               mov ebx,ecx
0041DA40   |.  E8 0BFEFFFF        call <SGUI.RSACompute>                         ;这里面是RSA的初始化，里面有不少的构造和析构函数
-----------------------------------------------------------------------------------------
0041D850 <>/$  6A FF              push -1                                        ;  RSACompute
0041D852   |.  68 50744500        push <SGUI.loc_457450>                         ;  SE handler installation
0041D857   |.  64:A1 00000000     mov eax,dword ptr fs:[0]
0041D85D   |.  50                 push eax
0041D85E   |.  64:8925 00000000   mov dword ptr fs:[0],esp
0041D865   |.  83EC 08            sub esp,8
0041D868   |.  53                 push ebx
0041D869   |.  56                 push esi
0041D86A   |.  8BF1               mov esi,ecx
0041D86C   |.  57                 push edi
0041D86D   |.  68 08D64600        push SGUI.0046D608                             ;  ASCII "343A6F971A5035321CDECBC4ECE1945"
0041D872   |.  8D4C24 10          lea ecx,dword ptr ss:[esp+10]
0041D876   |.  E8 C7610300        call <SGUI.CString::CString(char const *)>     ;  上面的这个是RSA参数n
0041D87B   |.  8DBE 10040000      lea edi,dword ptr ds:[esi+410]
0041D881   |.  50                 push eax
0041D882   |.  8BCF               mov ecx,edi
0041D884   |.  C74424 20 00000000 mov dword ptr ss:[esp+20],0
0041D88C   |.  E8 E7610300        call <SGUI.CString::operator=(CString const &)>;  jmp to MFC42.#858
0041D891   |.  8D4C24 0C          lea ecx,dword ptr ss:[esp+C]
0041D895   |.  C74424 1C FFFFFFFF mov dword ptr ss:[esp+1C],-1
0041D89D   |.  E8 C45E0300        call <SGUI.CString::~CString(void)>            ;  jmp to MFC42.#800
0041D8A2   |.  68 00D64600        push SGUI.0046D600                             ;  ASCII "10001"
0041D8A7   |.  8D4C24 14          lea ecx,dword ptr ss:[esp+14]
0041D8AB   |.  E8 92610300        call <SGUI.CString::CString(char const *)>     ;  上面的这个是公钥e
0041D8B0   |.  8D9E 0C040000      lea ebx,dword ptr ds:[esi+40C]
0041D8B6   |.  50                 push eax
0041D8B7   |.  8BCB               mov ecx,ebx
0041D8B9   |.  C74424 20 01000000 mov dword ptr ss:[esp+20],1
0041D8C1   |.  E8 B2610300        call <SGUI.CString::operator=(CString const &)>;  jmp to MFC42.#858
0041D8C6   |.  8D4C24 10          lea ecx,dword ptr ss:[esp+10]
0041D8CA   |.  C74424 1C FFFFFFFF mov dword ptr ss:[esp+1C],-1
0041D8D2   |.  E8 8F5E0300        call <SGUI.CString::~CString(void)>            ;  jmp to MFC42.#800
0041D8D7   |.  6A 10              push 10                                        ; /Arg2 = 00000010
0041D8D9   |.  57                 push edi                                       ; |Arg1
0041D8DA   |.  8D8E 34050000      lea ecx,dword ptr ds:[esi+534]                 ; |
0041D8E0   |.  E8 5BF8FFFF        call <SGUI.RSAMove>                            ; \SGUI.0041D140
0041D8E5   |.  6A 10              push 10                                        ; /Arg2 = 00000010
0041D8E7   |.  53                 push ebx                                       ; |Arg1
0041D8E8   |.  8D8E C4050000      lea ecx,dword ptr ds:[esi+5C4]                 ; |
0041D8EE   |.  E8 4DF8FFFF        call <SGUI.RSAMove>                            ; \SGUI.0041D140
0041D8F3   |.  B9 40000000        mov ecx,40
0041D8F8   |.  33C0               xor eax,eax
0041D8FA   |.  8D7E 04            lea edi,dword ptr ds:[esi+4]
0041D8FD   |.  F3:AB              rep stos dword ptr es:[edi]
0041D8FF   |.  B9 40000000        mov ecx,40
0041D904   |.  8DBE 04010000      lea edi,dword ptr ds:[esi+104]
0041D90A   |.  F3:AB              rep stos dword ptr es:[edi]
0041D90C   |.  B9 40000000        mov ecx,40
0041D911   |.  8DBE 04020000      lea edi,dword ptr ds:[esi+204]
0041D917   |.  F3:AB              rep stos dword ptr es:[edi]
0041D919   |.  B9 40000000        mov ecx,40
0041D91E   |.  8DBE 04030000      lea edi,dword ptr ds:[esi+304]
0041D924   |.  F3:AB              rep stos dword ptr es:[edi]
0041D926   |.  8B4C24 14          mov ecx,dword ptr ss:[esp+14]
0041D92A   |.  5F                 pop edi
0041D92B   |.  5E                 pop esi
0041D92C   |.  B8 01000000        mov eax,1
0041D931   |.  5B                 pop ebx
0041D932   |.  64:890D 00000000   mov dword ptr fs:[0],ecx
0041D939   |.  83C4 14            add esp,14
0041D93C   \.  C3                 retn
--------------------------------------------------------------------------------------------------------
0041DA45   |.  8BB424 AC010000    mov esi,dword ptr ss:[esp+1AC]
0041DA4C   |.  8B06               mov eax,dword ptr ds:[esi]                     ;这里是注册码
0041DA4E   |.  8B40 F8            mov eax,dword ptr ds:[eax-8]                   ;这里是注册码的长度
0041DA51   |.  83F8 24            cmp eax,24
0041DA54   |.  74 07              je short <SGUI.loc_41DA5D>
0041DA56   |.  33C0               xor eax,eax
0041DA58   |.  E9 8D010000        jmp <SGUI.loc_41DBEA>
0041DA5D <>|>  56                 push esi                                       ;  loc_41DA5D
0041DA5E   |.  E8 AD010000        call <SGUI.BalckList>                          ;  这里是检验黑名单
0041DA63   |.  83C4 04            add esp,4
0041DA66   |.  85C0               test eax,eax                                   ;  是黑名单中的eax=1 else eax=0
0041DA68   |.  74 07              je short <SGUI.loc_41DA71>
0041DA6A   |.  33C0               xor eax,eax
0041DA6C   |.  E9 79010000        jmp <SGUI.loc_41DBEA>
0041DA71 <>|>  B9 40000000        mov ecx,40                                     ;  loc_41DA71
0041DA76   |.  33C0               xor eax,eax
0041DA78   |.  8D7C24 0C          lea edi,dword ptr ss:[esp+C]
0041DA7C   |.  F3:AB              rep stos dword ptr es:[edi]
0041DA7E   |.  8B0E               mov ecx,dword ptr ds:[esi]
0041DA80   |.  8B41 F8            mov eax,dword ptr ds:[ecx-8]
0041DA83   |.  8BCE               mov ecx,esi
0041DA85   |.  50                 push eax
0041DA86   |.  E8 F35F0300        call <SGUI.CString::GetBuffer(int)>            ;  jmp to MFC42.#2915
0041DA8B   |.  8BF8               mov edi,eax
0041DA8D   |.  83C9 FF            or ecx,FFFFFFFF
0041DA90   |.  33C0               xor eax,eax
0041DA92   |.  8D5424 0C          lea edx,dword ptr ss:[esp+C]
0041DA96   |.  F2:AE              repne scas byte ptr es:[edi]
0041DA98   |.  F7D1               not ecx
0041DA9A   |.  2BF9               sub edi,ecx
0041DA9C   |.  8BC1               mov eax,ecx
0041DA9E   |.  8BF7               mov esi,edi
0041DAA0   |.  8BFA               mov edi,edx
0041DAA2   |.  C1E9 02            shr ecx,2
0041DAA5   |.  F3:A5              rep movs dword ptr es:[edi],dword ptr ds:[esi]
0041DAA7   |.  8BC8               mov ecx,eax
0041DAA9   |.  33C0               xor eax,eax
0041DAAB   |.  83E1 03            and ecx,3
```
