
0CF5C28F . 56  push  esi

0CF5C290 . E8 BFDE0400  call  <jmp.&Lua5.lua_pushstring>

0CF5C295 . 8B4C24 4C mov  ecx, dword ptr [esp+4C]

0CF5C299 . 83C4 08 add  esp, 8

0CF5C29C . 5E  pop  esi

0CF5C29D . 33CC  xor  ecx, esp

0CF5C29F . B8 01000000  mov  eax, 1

0CF5C2A4 . E8 FDDF0400  call  0CFAA2A6

0CF5C2A9 . 83C4 44 add  esp, 44

0CF5C2AC . C3  retn

### 8. 分析 `GetPassword()`

得知：

```
0CF5C285 . E8 86A80300  call  0CF96B10
```

这个 `call` 返回了加密后的密码。

### 9. 再对 `call 0CF96B10` 分析

得知：

```
0CF96C37 . 894424 24 mov  dword ptr [esp+24], eax
0CF96C3B . 51  push  ecx ;
0CF96C3C . 8D4424 28 lea  eax, dword ptr [esp+28]
0CF96C40 . 66:895C24 1A mov  word ptr [esp+1A], bx
0CF96C45 . E8 76AEFBFF  call  0CF51AC0 ; 取单个字符"78787878"
0CF96C4A . C68424 D00000>mov byte ptr [esp+D0], 1
0CF96C52 . 8B5424 24 mov  edx, dword ptr [esp+24]
0CF96C56 . 52  push  edx ; 组成字符
0CF96C57 . 8D8C24 AC0000>lea ecx, dword ptr [esp+AC]
0CF96C5E . FF15 FCE1FA0C call dword ptr [<&MSVCP80.std::basic_string<char,std::ch>;  msvcp80.std::basic_string<char,std::char_traits<char>,std::allocator<char> >::operator+=
```
......
......

```
0CF96CF5 . 50  push  eax
0CF96CF6 . 52  push  edx
0CF96CF7 . FF15 3CE0FA0C call dword ptr [<&Engine_Lua5.KG_EDStringToMD5String>] ; Engine_L.KG_EDStringToMD5String
```

密码是由 `Engine_L.KG_EDStringToMD5String` 进行加密，很可能是 MD5。

跟进 `Engine_L.KG_EDStringToMD5String`，发现：

```
1000128A |. E8 11080000 call  md5_init
10001296 |. E8 35080000 call  md5_append
100012A5 |. E8 F6080000 call  md5_finish
```

### 10. 用 MD5 测试游戏加密后的密码

正确，可逆。

### 11.
