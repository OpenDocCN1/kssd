
# WinISO v4.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1252.htm`](https://www.pediy.com/kssd/pediy03/forum1252.htm)

参考国外高手之 patch

```text
     ____ _ __ ____ ____
   /_/ __)\_/ |_/ |__  |__/  __)
   \ \__  ) \  _/ | \ --|__ __)\
.-' \__/| \_|_| \__\ \__| `-.
| Flying Horse Cracking Force _|
_|                          \
__ RelType: Patch           |
| Cracker: Charles Manson    |
|                            |
`--------------------[31/07/2001]-'
        WinISO v4.0
```

`http://www.winiso.com/download.htm`

## (1)

```text
:0040B48C 81FF00003000        cmp edi, 00300000
:0040B492 7630                jbe 0040B4C4================>改 jmp!!
```

```text
* Reference To: WinISO._fmMainWnd
                        |
:0040B494 8B15CC074C00     mov edx, dword ptr [004C07CC]
:0040B49A 6A10             push 00000010

* Possible StringData Ref from Data Obj ->"STOP"
                        |
:0040B49C 68AE414B00       push 004B41AE

* Possible StringData Ref from Data Obj ->"Unregistered Version can not extract "
                ->"large files(>3MB) "
                        |
:0040B4A1 6876414B00       push 004B4176
```

## (2)

```text
:0040BAE8 817D1400004006      cmp dword ptr [ebp+14], 06400000
:0040BAEF 762F               jbe 0040BB20================>改 jmp!!
```

```text
* Reference To: WinISO._fmMainWnd
                        |
:0040BAF1 8B15CC074C00     mov edx, dword ptr [004C07CC]
:0040BAF7 6A10             push 00000010

* Possible StringData Ref from Data Obj ->"STOP"
                        |
:0040BAF9 689C424B00       push 004B429C

* Possible StringData Ref from Data Obj ->"Unregistered Version can not process "
                ->"large image file(>100MB). "
                        |
:0040BAFE 685C424B00       push 004B425C
```

## (3)

```text
:0040C80E 81B9CC05000000004006   cmp dword ptr [ecx+000005CC], 06400000
:0040C818 762E               jbe 0040C848================>改 jmp!!
:0040C81A A1CC074C00         mov eax, dword ptr [004C07CC]
:0040C81F 6A10             push 00000010
```

```text
* Possible StringData Ref from Data Obj ->"STOP"
                        |
:0040C821 68A3434B00       push 004B43A3

* Possible StringData Ref from Data Obj ->"Unregistered Version can not process "
                ->"large image file(>100MB). "
                        |
:0040C826 6863434B00       push 004B4363
```

## (4)

```text
:0040FFEC 817F0800C80000      cmp dword ptr [edi+08], 0000C800
:0040FFF3 7620               jbe 00410015================>改 jmp!!
:0040FFF5 6A10             push 00000010
```

```text
* Possible StringData Ref from Data Obj ->"STOP"
                        |
:0040FFF7 68FE634B00       push 004B63FE

* Possible StringData Ref from Data Obj ->"Unregistered Version can not process "
                ->"large image file(>100MB). "
                        |
:0040FFFC 68BE634B00       push 004B63BE
```

```text
\=====================================================================
```

自诩为高手的 `cracker` 可试试其注册码的难度!!!!!!!!!!!!!!!!!!!!!!!!!!

自诩为高手的 `cracker` 可试试其注册码的难度!!!!!!!!!!!!!!!!!!!!!!!!!! `4dd466d0e1e0cc123afbe504da2462ca`
