
# ACProtect 代码保护示例分析

## 源代码实现

```cpp
DYNAMIC_BEGIN; //u must include this line before the code protected

MessageBox(NULL, "First decrypt this code, then run ,it will be encrypted again after run!",
          "Dynamic En/Decryption of codes example", MB_OK);

DYNAMIC_END; //u must include this line after the code protected

//No 2  EMBEDDED PROTECTION
EMBEDDED_BEGIN; //u must include this line before the code protected

MessageBox(NULL, "The Cryptor encrypt the codes between the embedded lock header and embedded lock tail ,
                 then communicate with the loader, After checking, then decrypt the codes, and run the decrypted code.
                 After the Running ,all the codes (include the embedded cryptor and the crypted code) will
                 keep the original encrypted status..",    "Embedded Protection of codes example", MB_OK);

EMBEDDED_END; //u must include this line after the code protected

//No 3  GetRegistrationName
//Get User name from ACProtect loader
char usrname[255] = "";
GetRegistrationName(usrname);

if (usrname[0])
    MessageBox(NULL, usrname, "User Name is :", MB_OK);
else
    MessageBox(NULL, "UnRegistered Version", "User Name is :", MB_OK);

//No 4  RSA LOCK CODE
bool keyok = false;
RSALOCK_BEGIN; //u must include this line before the code protected
keyok = !keyok;
MessageBox(NULL, "If u have no correct license file, you can not see me!", "RSA Lock code sample", MB_OK);
RSALOCK_END; //u must include this line after the code protected

return 0;
}
```

## 二进制分析与逆向

用 `VC` 编译后，还没加壳时，用 `OD` 载入看看，很明显：

*   `Messagebox(-1,0,0,2)` `RSA_Lock` start
*   `Messagebox(-1,0,0,3)` `RSA_Lock` end
*   `Messagebox(-1,0,0,5)` `Embedded` start
*   `Messagebox(-1,0,0,4)` `Embedded` end

`dynamic` `encrypt` 部分，相当于一个普通压缩壳

```asm
0040100C  |.  60                          PUSHAD
0040100D  |.  E9 AA010000                 JMP ACP.004011BC
00401012  |.  64 79 6D 6C 63 6B 5F 5F 68 65 61 64 65 72 00   ASCII "dymlck__header",0
00401021  |   00                          DB 00
00401022  |   00                          DB 00
00401023  |   00                          DB 00
```

... ( 中间 0 加壳后变成解压部分 ) ...

```asm
004011AD  |   00                          DB 00
004011AE  |   00                          DB 00
004011AF  |   00                          DB 00
004011B0  |   64                          DB 64                  ;  CHAR 'd'
004011B1  |   79                          DB 79                  ;  CHAR 'y'
004011B2  |   6D                          DB 6D                  ;  CHAR 'm'
004011B3  |   6C                          DB 6C                  ;  CHAR 'l'
004011B4  |   63                          DB 63                  ;  CHAR 'c'
004011B5  |   6B                          DB 6B                  ;  CHAR 'k'
004011B6  |   5F                          DB 5F                  ;  CHAR '_'
004011B7  |   62                          DB 62                  ;  CHAR 'b'
004011B8  |   65                          DB 65                  ;  CHAR 'e'
004011B9  |   67                          DB 67                  ;  CHAR 'g'
004011BA  |   69                          DB 69                  ;  CHAR 'i'
004011BB  |   6E                          DB 6E                  ;  CHAR 'n'
004011BC  |>  61                          POPAD
004011BD  |.  6A 00                       PUSH 0                  ; /Style = MB_OK|MB_APPLMODAL
004011BF  |.  68 84A24000                 PUSH ACP.0040A284       ; |Title = "Dynamic En/Decryption of codes example"
004011C4  |.  68 3CA24000                 PUSH ACP.0040A23C       ; |Text = "First decrypt this code, then run ,
                                                              ;        it will be encrypted again after run!"
004011C9  |.  6A 00                       PUSH 0                  ; |hOwner = NULL
004011CB  |.  FF15 B4904000               CALL DWORD PTR DS:[<&USER32.MessageBoxA>>; MessageBoxA
004011D1  |.  60                          PUSHAD
004011D2  |.  EB 3A                       JMP SHORT ACP.0040120E
004011D4  |.  64 79 6D 6C 63 6B 5F 65 6E 64 00   ASCII "dymlck_end",0
004011DF  |   00                          DB 00
004011E0  |   00                          DB 00
004011E1  |   00                          DB 00
```

... ( 中间 0 加壳后变成压缩部分 ) ...

```asm
0040120B  |   00                          DB 00
0040120C  |   00                          DB 00
0040120D  |   00                          DB 00
0040120E  |>  61                          POPAD
```

`Embeded` `Protect` 部分，相当于一个反调试的壳

```asm
0040120F  |.  60    PUSHAD
00401210  |.  6A 05 PUSH 5                    ; /Style = MB_RETRYCANCEL|MB_APPLMODAL
00401212  |.  6A 00 PUSH 0                    ; |Title = NULL
00401214  |.  6A 00 PUSH 0                    ; |Text = NULL
00401216  |.  6A FF PUSH -1                   ; |hOwner = FFFFFFFF
00401218  |.  FF15 B4904000 CALL DWORD PTR DS:[<&USER32.MessageBoxA>>; MessageBoxA
0040121E  |.  61    POPAD
0040121F  |.  60    PUSHAD
00401220  |.  E9 1C320000 JMP ACP.0404441
00401225  |.  70 65 65 74 6 ASCII "peetles__header",0          ; // 作者喜欢披头士?
00401235  |   00    DB 00
00401236  |   00    DB 00
00401237  |   00    DB 00
00401238  |   00    DB 00
00401239  |   00    DB 00
0040123A  |   00    DB 00
0040123B  |   00    DB 00
0040123C  |   00    DB 00
```

. 加壳后用来放检测壳，检测调试器，解密，加密代码

. `!!!!!!!!!!!!!` `0404442` - `0401218` = `322A` ，以后从 `Call` `MessageBox` 这一行地址加 `322A` 就是用户代码开始处 `!!!!!!!!!!!!!`

```asm
00404432  |   00    DB 00
00404433  |   00    DB 00
00404434  |   70    DB 70                     ;  CHAR 'p'
00404435  |   65    DB 65                     ;  CHAR 'e'
00404436  |   65    DB 65                     ;  CHAR 'e'
00404437  |   74    DB 74                     ;  CHAR 't'
00404438  |   6C    DB 6C                     ;  CHAR 'l'
00404439  |   65    DB 65                     ;  CHAR 'e'
0040443A  |   73    DB 73                     ;  CHAR 's'
0040443B  |   5F    DB 5F                     ;  CHAR '_'
0040443C  |   62    DB 62                     ;  CHAR 'b'
0040443D  |   65    DB 65                     ;  CHAR 'e'
```
