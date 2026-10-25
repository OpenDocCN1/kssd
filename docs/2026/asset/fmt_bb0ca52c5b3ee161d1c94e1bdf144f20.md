
```
* Possible StringData Ref from Data Obj ->"xios"

            |

:004062B3 6880474100 push 00414780
:004062B8 52 push edx
:004062B9 FFD7 call edi
:004062BB 6A02 push 00000002
:004062BD 56 push esi
:004062BE 8BE8 mov ebp, eax
:004062C0 C60630 mov byte ptr [esi], 30
:004062C3 C6460100 mov [esi+01], 00
:004062C7 8B442418 mov eax, dword ptr [esp+18]
:004062CB 6A01 push 00000001
:004062CD 6A00 push 00000000

* Possible StringData Ref from Data Obj ->"really"

            |

:004062CF 6878474100 push 00414778
:004062D4 50 push eax
:004062D5 FFD7 call edi
:004062D7 85ED test ebp, ebp
:004062D9 7519 jne 004062F4
:004062DB 837C241401 cmp dword ptr [esp+14], 00000001
:004062E0 7512 jne 004062F4
:004062E2 55 push ebp

* Possible StringData Ref from Data Obj ->"  回音壁  "

            |

:004062E3 686C474100 push 0041476C

* Possible StringData Ref from Data Obj ->"  注册成功，感谢您选用并且注册小精灵 "

  这就不用说了吧！

            |

:004062E8 6844474100 push 00414744
:004062ED 8BCB mov ecx, ebx

* Reference To: MFC42.Ordinal:1080, Ord:1080h

            |

:004062EF E882680000 Call 0040CB76

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004062D9(C), :004062E0(C)
|

:004062F4 8B13 mov edx, dword ptr [ebx]
:004062F6 8BCB mov ecx, ebx
:004062F8 FF92CC000000 call dword ptr [edx+000000CC]
:004062FE 8B442410 mov eax, dword ptr [esp+10]

* Reference To: ADVAPI32.RegCloseKey, Ord:015Bh

            |

:00406302 8B3508F04000 mov esi, dword ptr [0040F008]
:00406308 50 push eax
:00406309 FFD6 call esi
:0040630B 8B4C2418 mov ecx, dword ptr [esp+18]
:0040630F 51 push ecx
:00406310 FFD6 call esi
:00406312 5F pop edi
:00406313 5E pop esi
:00406314 5D pop ebp
:00406315 5B pop ebx
:00406316 83C42C add esp, 0000002C
:00406319 C3 ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00406206(C)

              不祥之跳，赶快上去拦住它！！！

**************

|

:0040631A 83F80E cmp eax, 0000000E
:0040631D 0F8DEFFEFFFF jnl 00406212
:00406323 6A00 push 00000000

* Possible StringData Ref from Data Obj ->" 错误提示 "

            |

:00406325 6838474100 push 00414738

* Possible StringData Ref from Data Obj ->"  您输入了错误的注册码  "

            |

:0040632A 681C474100 push 0041471C
:0040632F 8BCB mov ecx, ebx

* Reference To: MFC42.Ordinal:1080, Ord:1080h

            |

:00406331 E840680000 Call 0040CB76
:00406336 5F pop edi
:00406337 5E pop esi
:00406338 5D pop ebp
:00406339 5B pop ebx
:0040633A 83C42C add esp, 0000002C
:0040633D C3 ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00406159(C), :00406165(C)
**************

|

:0040633E 51 push ecx  跳到这的是笨蛋！！！按要求输入。别生气呀！！！

* Possible StringData Ref from Data Obj ->" 错误提示 "

            |

:0040633F 68C4474100 push 004147C4

* Possible StringData Ref from Data Obj ->"  错误，不能输入空行  "

            |

:00406344 6804474100 push 00414704
:00406349 8BCB mov ecx, ebx

* Reference To: MFC42.Ordinal:1080, Ord:1080h

            |

:0040634B E826680000 Call 0040CB76
:00406350 5F pop edi
:00406351 5E pop esi
:00406352 5D pop ebp
:00406353 5B pop ebx
:00406354 83C42C add esp, 0000002C
:00406357 C3 ret
```

总结：

注册的方法：

一、向作者注册。

二、注册名：`LIHATIAO`，注册码：`RA2zHD2rAAJEAK`。

三、修改注册表：`[HKEY_CURRENT_USER\Software\Microsoft\Windows]`，设置 `"sos"="1"`。

最后要说的是写破解过程真的很累，看雪学苑那么多的教程都是前辈们辛苦整理的，向他们致敬！

LIHAITAO

2001.2.10
