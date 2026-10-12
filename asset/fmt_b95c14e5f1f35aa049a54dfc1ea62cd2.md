
`CODE:00401D8F ; ///////////////////////////////////////////////////////////////////`  

`CODE:00401D8F`  

`CODE:00401D8F loc_401D8F:`  

`CODE:00401D8F                  lea     eax, [ebp+lpszInfoBuf]`  

`CODE:00401D95                  call    sub_4018D7      ; 把 Internet 返回的字符串"200"`  

`CODE:00401D95                                     ; 转换成数字 200(0xC8)`  

子函数 `sub_4018D7` 把 Http 请求返回的信息转换成数字 200（十六进制 `0xC8`），并作为函数的返回值返回。  

`CODE:00401D9A                  mov     [ebp+var_dwReturnValue], eax`  

`CODE:00401D9D`  

`CODE:00401D9D loc_401D9D:`  

`CODE:00401D9D                  push    [ebp+hUrlFile]`  

`CODE:00401DA3                  call    InternetCloseHandle`  

`CODE:00401DA8`  

`CODE:00401DA8 loc_401DA8:`  

`CODE:00401DA8                  mov     eax, [ebp+var_dwReturnValue] ; 函数最后返回值 200(0xC8)`  

`CODE:00401DAB                  pop     esi`  

`CODE:00401DAC                  leave`  

`CODE:00401DAD                  retn    10h`  

`CODE:00401DAD sub_401CF0      endp`  

函数返回：  

`CODE:00402082                  call    sub_401CF0`  

`CODE:00402087                  test    eax, eax     ;函数返回到这里`  

`CODE:00402089                  jz      loc_402264      ; sub_401CF0 调用失败则跳转`  

`CODE:0040208F                  cmp     eax, 0FFFFFFFFh`  

`CODE:00402092                  jz      loc_402264`  

`CODE:00402098                  cmp     eax, 0C8h`  

`CODE:0040209D                  jnz     loc_40228A`  

跳来跳去看的我眼都花了，今天先到这，记下各寄存器的值明天继续...  

函数返回到 `402087` 时各寄存器的值为：  

`\\============================`  

`EAX` = `000000C8`  

`EBX` = `00000000`  

`ECX` = `0007E220`  

`EDX` = `00000000`  

`ESI` = `0040602C`  

`EDI` = `0076F670`  

`EBP` = `0076FFB4`  

`ESP` = `0076F54C`  

`C` `0`  

`P` `1`  

`A` `0`  

`Z` `1`  

`S` `0`  

`T` `0`  

`D` `0`  

`O` `0`  

`I` `1`  

`\\============================`

`4dd466d0e1e0cc123afbe504da2462ca`
