
```
:004863C9 E86ED6F7FF              call 00403A3C
:004863CE C3                      ret
:004863CF E9FCDDF7FF              jmp 004041D0
:004863D4 EBEE                    jmp 004863C4
:004863D6 8D55CC                  lea edx, dword ptr [ebp-34]
:004863D9 B8BC614800              mov eax, 004861BC
:004863DE E80106F8FF              call 004069E4
:004863E3 8B45CC                  mov eax, dword ptr [ebp-34]
:004863E6 8A10                    mov dl, byte ptr [eax]
:004863E8 8D45D0                  lea eax, dword ptr [ebp-30]
:004863EB E898E5F7FF              call 00404988
:004863F0 8B45D0                  mov eax, dword ptr [ebp-30]
:004863F3 50                      push eax
:004863F4 8D45D4                  lea eax, dword ptr [ebp-2C]
:004863F7 50                      push eax
:004863F8 8D55C8                  lea edx, dword ptr [ebp-38]
:004863FB B8B4614800              mov eax, 004861B4
:00486400 E8DF05F8FF              call 004069E4
:00486405 8B4DC8                  mov ecx, dword ptr [ebp-38]
:00486408 BAB4644800              mov edx, 004864B4
:0048640D A12CDD4800              mov eax, dword ptr [0048DD2C]
:00486412 E8B1FCFFFF              call 004860C8
:00486417 8B55D4                  mov edx, dword ptr [ebp-2C]  //哈哈..这次是不是快很多...
:0048641A A130DD4800              mov eax, dword ptr [0048DD30]
```

`name:ssljx[BCG]`

`code:2ceddf9b`

把注册表`HKEY_CURRENT_USER\Software\iSplit`又是未注册版.

这篇破文没讲算法,没有具体取分析程序.我只是将我破解过程的到的小小经验跟各位分享..断点设得好,破解容易搞,,哈...各位大侠指正!!!!

`ssljx[BCG]` 4dd466d0e1e0cc123afbe504da2462ca
