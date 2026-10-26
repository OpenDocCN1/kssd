
# [原创]IE收藏夹管理小精灵算法分析

【破文作者】 YuanQiao[BCG][]

【文章题目】 IE收藏夹管理小精灵

【下载地址】 26fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6S2L8X3S2G2L8#2)9J5k6h3y4G2L8g2)9J5c8Y4m8J5L8#2)9J5c8X3k6S2N6X3!0J5K9i4c8W2i4K6u0r3k6X3q4$3L8%4u0A6N6r3g2e0k6i4c8#2M7q4)9J5k6h3g2^5k6b7`.

----------------------------------------------------------------------------------------------

【破解工具】 `PEiD`, `OllyDbg`

【破解难度】 +++初级+++

【破解平台】 `WinXP SP2`

----------------------------------------------------------------------------------------------

## 【文章简介】

1.可以录入网址

2.可以查找网址

3.可以打开网址

4.可以对网址分类

5.可以从IE收藏夹导入网址

6.可以把网址导放到IE收藏夹

7.清空IE收藏夹

8.支持本地网络实名(中文上网,把你的收藏夹当做数据库,你根本不用花钱去买网络实名，自己录一下就可以了,要什么有什么,你要什么关键字都可以自已录入)

9.支持广告拦截

10.支持各种搜索引擎(baidu,google,yahoo,sogou,yisou ,只要你在IE的网址上敲入中文,便可以通过各种搜索引擎进行搜索)

----------------------------------------------------------------------------------------------

## 【破解过程】

用`PEiD`检测显示《Borland Delphi 6.0 - 7.0》无壳。

运行软件，注册无提示。

用`OLLYDBG`载入之后，使用字符串参考。

看到一些字符：

`\\.\PhysicalDrive0`

`regCode`

`regUser`

通过下面计算后，说明找的断点是正确的。

```
004EB0C4  /.  55            PUSH EBP
004EB0C5  |.  8BEC          MOV EBP,ESP
004EB0C7  |.  33C9          XOR ECX,ECX
004EB0C9  |.  51            PUSH ECX
004EB0CA  |.  51            PUSH ECX
004EB0CB  |.  51            PUSH ECX
004EB0CC  |.  51            PUSH ECX
004EB0CD  |.  33C0          XOR EAX,EAX
004EB0CF  |.  55            PUSH EBP
004EB0D0  |.  68 35B14E00   PUSH Favorite.004EB135
004EB0D5  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004EB0D8  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004EB0DB  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]
004EB0DE  |.  E8 592BFFFF   CALL Favorite.004DDC3C        ;  计算机器码(如有兴趣可以进去看一看)
004EB0E3  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004EB0E6  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
004EB0E9  |.  E8 AE33FFFF   CALL Favorite.004DE49C          ;  计算真注册码的MD5值
004EB0EE  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  ;  把真注册码的MD5值给EAX
004EB0F1  |.  50            PUSH EAX
004EB0F2  |.  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
004EB0F5  |.  B8 48B14E00   MOV EAX,Favorite.004EB148      ;  ASCII "regCode"
004EB0FA  |.  E8 D131FFFF   CALL Favorite.004DE2D0           ;  计算假注册码的MD5值
004EB0FF  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]  ;  把假注册码的MD5值给EAX
004EB102  |.  8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]
004EB105  |.  E8 D6E0F1FF   CALL Favorite.004091E0
004EB10A  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]
004EB10D  |.  58            POP EAX
004EB10E  |.  E8 1D9BF1FF   CALL Favorite.00404C30          ;  比较两个MD5值是否相等
004EB113      74 05         JE SHORT Favorite.004EB11A      ;  这个就是爆破口
004EB115  |.  E8 9EDDFFFF   CALL Favorite.004E8EB8
004EB11A  |>  33C0          XOR EAX,EAX
004EB11C  |.  5A            POP EDX
004EB11D  |.  59            POP ECX
004EB11E  |.  59            POP ECX
004EB11F  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
004EB122  |.  68 3CB14E00   PUSH Favorite.004EB13C
004EB127  |>  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
004EB12A  |.  BA 04000000   MOV EDX,4
004EB12F  |.  E8 1497F1FF   CALL Favorite.00404848
004EB134  \.  C3            RETN
```

计算机器码

---------004EB0DE  |.  E8 592BFFFF   CALL Favorite.004DDC3C ---------

```
004DDC3C  /$  55            PUSH EBP
004DDC3D  |.  8BEC          MOV EBP,ESP
004DDC3F  |.  33C9          XOR ECX,ECX
004DDC41  |.  51            PUSH ECX
004DDC42  |.  51            PUSH ECX
004DDC43  |.  51            PUSH ECX
004DDC44  |.  51            PUSH ECX
004DDC45  |.  51            PUSH ECX
004DDC46  |.  51            PUSH ECX
004DDC47  |.  51            PUSH ECX
004DDC48  |.  53            PUSH EBX
004DDC49  |.  8BD8          MOV EBX,EAX
004DDC4B  |.  33C0          XOR EAX,EAX
004DDC4D  |.  55            PUSH EBP
004DDC4E  |.  68 F2DC4D00   PUSH Favorite.004DDCF2
004DDC53  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004DDC56  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004DDC59  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004DDC5C  |.  E8 FF000000   CALL Favorite.004DDD60         ;  取硬盘物理序列号
004DDC61  |.  8D55 EC       LEA EDX,DWORD PTR SS:[EBP-14]
004DDC64  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004DDC67  |.  E8 74B5F2FF   CALL Favorite.004091E0
004DDC6C  |.  837D EC 00    CMP DWORD PTR SS:[EBP-14],0     ;  判断是否取到硬盘物理序列号
004DDC70  |.  75 08         JNZ SHORT Favorite.004DDC7A
004DDC72  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004DDC75  |.  E8 16030000   CALL Favorite.004DDF90          ;  取C盘逻辑序列号
004DDC7A  |>  8D55 E8       LEA EDX,DWORD PTR SS:[EBP-18]
004DDC7D  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004DDC80  |.  E8 5BB5F2FF   CALL Favorite.004091E0
004DDC85  |.  837D E8 00    CMP DWORD PTR SS:[EBP-18],0     ;  判断是否取到C盘逻辑序列号
004DDC89  |.  75 35         JNZ SHORT Favorite.004DDCC0
004DDC8B  |.  33C0          XOR EAX,EAX
004DDC8D  |.  E8 B2C2F2FF   CALL Favorite.00409F44
004DDC92  |.  8945 F0       MOV DWORD PTR SS:[EBP-10],EAX
004DDC95  |.  8955 F4       MOV DWORD PTR SS:[EBP-C],EDX
004DDC98  |.  68 08DD4D00   PUSH Favorite.004DDD08           ;  ASCII "h0u0"
004DDC9D  |.  FF75 F4       PUSH DWORD PTR SS:[EBP-C]        ; /Arg2
004DDCA0  |.  FF75 F0       PUSH DWORD PTR SS:[EBP-10]       ; |Arg1
004DDCA3  |.  8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]    ; |
004DDCA6  |.  E8 11BAF2FF   CALL Favorite.004096BC           ; \Favorite.004096BC
004DDCAB  |.  FF75 E4       PUSH DWORD PTR SS:[EBP-1C]
004DDCAE  |.  68 18DD4D00   PUSH Favorite.004DDD18           ;  ASCII "a9g5"
004DDCB3  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004DDCB6  |.  BA 03000000   MOV EDX,3
004DDCBB  |.  E8 E46EF2FF   CALL Favorite.00404BA4
004DDCC0  |>  8BC3          MOV EAX,EBX
004DDCC2  |.  B9 28DD4D00   MOV ECX,Favorite.004DDD28        ;  ASCII "21"
004DDCC7  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004DDCCA  |.  E8 616EF2FF   CALL Favorite.00404B30           ;  取到的序列号后加上21
004DDCCF  |.  33C0          XOR EAX,EAX
004DDCD1  |.  5A            POP EDX
004DDCD2  |.  59            POP ECX
004DDCD3  |.  59            POP ECX
004DDCD4  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
004DDCD7  |.  68 F9DC4D00   PUSH Favorite.004DDCF9
004DDCDC  |>  8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]
004DDCDF  |.  BA 03000000   MOV EDX,3
004DDCE4  |.  E8 5F6BF2FF   CALL Favorite.00404848
004DDCE9  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004DDCEC  |.  E8 336BF2FF   CALL Favorite.00404824
004DDCF1  \.  C3            RETN
```

计算真注册码的MD5值

---------004EB0E9  |.  E8 AE33FFFF   CALL Favorite.004DE49C------------

```
004DE49C  /$  55            PUSH EBP
004DE49D  |.  8BEC          MOV EBP,ESP
004DE49F  |.  83C4 B8       ADD ESP,-48
004DE4A2  |.  53            PUSH EBX
004DE4A3  |.  56            PUSH ESI
004DE4A4  |.  57            PUSH EDI
004DE4A5  |.  33C9          XOR ECX,ECX
004DE4A7  |.  894D B8       MOV DWORD PTR SS:[EBP-48],ECX
004DE4AA  |.  894D CC       MOV DWORD PTR SS:[EBP-34],ECX
004DE4AD  |.  894D D0       MOV DWORD PTR SS:[EBP-30],ECX
004DE4B0  |.  894D F4       MOV DWORD PTR SS:[EBP-C],ECX
004DE4B3  |.  894D F0       MOV DWORD PTR SS:[EBP-10],ECX
004DE4B6  |.  894D D8       MOV DWORD PTR SS:[EBP-28],ECX
004DE4B9  |.  8955 F8       MOV DWORD PTR SS:[EBP-8],EDX
004DE4BC  |.  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
004DE4BF  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004DE4C2  |.  E8 0D68F2FF   CALL Favorite.00404CD4
004DE4C7  |.  33C0          XOR EAX,EAX
004DE4C9  |.  55            PUSH EBP
004DE4CA  |.  68 C5E64D00   PUSH Favorite.004DE6C5
004DE4CF  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004DE4D2  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004DE4D5  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
004DE4D8  |.  8B15 74ED4E00 MOV EDX,DWORD PTR DS:[4EED74]   ;  取一固定字符串
004DE4DE  |.  E8 D963F2FF   CALL Favorite.004048BC
004DE4E3  |.  8D55 D0       LEA EDX,DWORD PTR SS:[EBP-30]
004DE4E6  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]    ;  取机器码
```
