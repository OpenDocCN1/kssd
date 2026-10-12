
```asm
:0055E8F1 8B55E4               mov edx, dword ptr [ebp-1C]
* Possible StringData Ref from Code Obj ->"loezem"

:0055E8F4 B874EE5500          mov eax, 0055EE74
:0055E8F9 E8B25BEAFF          call 004044B0
:0055E8FE 85C0               test eax, eax
:0055E900 0F8FCB040000        jg 0055EDD1
:0055E906 8D55D8             lea edx, dword ptr [ebp-28]
:0055E909 A11C385800         mov eax, dword ptr [0058381C]
:0055E90E E88179EDFF         call 00436294
:0055E913 8B45D8             mov eax, dword ptr [ebp-28]
:0055E916 8D55DC             lea edx, dword ptr [ebp-24]
:0055E919 E85EA7EAFF         call 0040907C
:0055E91E 8B55DC             mov edx, dword ptr [ebp-24]
* Possible StringData Ref from Code Obj ->"loezem"

:0055E921 B874EE5500         mov eax, 0055EE74
:0055E926 E8855BEAFF         call 004044B0
:0055E92B 85C0               test eax, eax
:0055E92D 0F8F9E040000       jg 0055EDD1
:0055E933 8D55D0             lea edx, dword ptr [ebp-30]
:0055E936 A11C385800         mov eax, dword ptr [0058381C]
:0055E93B E85479EDFF         call 00436294
:0055E940 8B45D0             mov eax, dword ptr [ebp-30]
:0055E943 8D55D4             lea edx, dword ptr [ebp-2C]
:0055E946 E831A7EAFF         call 0040907C
:0055E94B 8B55D4             mov edx, dword ptr [ebp-2C]
* Possible StringData Ref from Code Obj ->"破解"

:0055E94E B884EE5500         mov eax, 0055EE84
:0055E953 E8585BEAFF         call 004044B0----软件中是否含有"破解"
:0055E958 85C0               test eax, eax
:0055E95A 0F8F71040000       jg 0055EDD1
:0055E960 8D55C8             lea edx, dword ptr [ebp-38]
:0055E963 A11C385800         mov eax, dword ptr [0058381C]
:0055E968 E82779EDFF         call 00436294
:0055E96D 8B45C8             mov eax, dword ptr [ebp-38]
:0055E970 8D55CC             lea edx, dword ptr [ebp-34]
:0055E973 E804A7EAFF         call 0040907C
:0055E978 8B55CC             mov edx, dword ptr [ebp-34]
* Possible StringData Ref from Code Obj ->"破解"

:0055E97B B884EE5500         mov eax, 0055EE84
:0055E980 E82B5BEAFF         call 004044B0
:0055E985 85C0               test eax, eax
:0055E987 0F8F44040000       jg 0055EDD1
:0055E98D 8D55C0             lea edx, dword ptr [ebp-40]
:0055E990 A11C385800         mov eax, dword ptr [0058381C]
:0055E995 E8FA78EDFF         call 00436294
:0055E99A 8B45C0             mov eax, dword ptr [ebp-40]
:0055E99D 8D55C4             lea edx, dword ptr [ebp-3C]
:0055E9A0 E8D7A6EAFF         call 0040907C
:0055E9A5 8B55C4             mov edx, dword ptr [ebp-3C]
* Possible StringData Ref from Code Obj ->"破解"

:0055E9A8 B884EE5500         mov eax, 0055EE84
:0055E9AD E8FE5AEAFF         call 004044B0
:0055E9B2 85C0               test eax, eax
:0055E9B4 0F8F17040000       jg 0055EDD1
:0055E9BA B201               mov dl, 01
```

接下来说说，他以前版本的杀毒认证，由于找不到以前的版本了，只能口述。

1. 将扫到的木马的本机地址（如：木马在 `D:\SS\ss.exe`），则发送 `http://www.luosoft.com/cgi-bin/iparmor1.pl?name=用户名!D:\SS\ss.exe`，如果用户名不是他服务器里有的，就显示 `NO OK`，如果有，就返回 `D:\SS\ss.exe`，此时软件就将 `D:\SS\ss.exe` 删除。

2. 如果显示 `NO OK` 就说你不是注册用户，并在注册表中删除你的注册项。

3. 如果返回的不是 `D:\SS\ss.exe`，则返回什么他就删除什么。

4. 如果返回空就显示“从起计算机才能策底清除木马”（这是在玩你哈）。

但此方法有一个 BUG，就是当文件名中有 `%20` 时，就无法删除，因为 `%20` 变成网址就是空格。

解决方法：

打开记事本，打入下列代码：

```
%>

response.write request("name")

%>
```

`4dd466d0e1e0cc123afbe504da2462ca`
