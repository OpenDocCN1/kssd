
# 电子收藏家 4.0

> 原文：[`www.pediy.com/kssd/pediy03/forum671.htm`](https://www.pediy.com/kssd/pediy03/forum671.htm)

国产新软破解实录(二) -- 电子收藏家 4.0

下载地址`http://www.esoftware.com.cn/oload.php?url=ftp://ftp.eware.com.cn/pub/fmanager/other/DC400Inst.zip`

## 1. 脱壳反汇编分析，找关键 call

串式参考`"Invalid Register Number"`，来到如下代码处：

```
:00541F72 EB15                jmp 00541F89

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00541EC3(C)
************************

:00541F74 8D55DC            lea edx, dword ptr [ebp-24]

* Possible StringData Ref from Code Obj -> 

:00541F77 B820205400        mov eax, 00542020

:00541F7C E81BD5FDFF        call 0051F49C
```

`******`号处双击鼠标右键

```
:00541EB8 8B45F4            mov eax, dword ptr [ebp-0C]
:00541EBB 5A                pop edx
:00541EBC E88F2FFEFF        call 00524E50 关键 call
:00541EC1 84C0              test al, al    注册标志
:00541EC3 0F84AB000000      je 00541F74    出错跳转(******)
```

## 2. trw2000 载入，下断 `bpx 541ebc`

用户名 `guodong` 注册码 `123456789`

`F8` 追入 `call 00524e50` (关键 call)

按 `F10` 一直到下面(光条停在 `524f24` 时止))

```
0167:00524F1F  MOV       EDX,[EBP-1C]
0167:00524F22  MOV       EAX,ESI
0167:00524F24  CALL      00404338   
```

`d eax` `eax=123456789` (假码)

`d edx` `edx=008FCDC1CF4E6B962A03DD4338E1B04A`

这么长串的数字，抄下来的吗？？？不，我这样做。

`trw2000` 中复制顶部窗口内容到文件里。

```
D range(范围) > filename
```

然后粘贴。

## 3. 整理

用户名 `guodong`

注册码 `008FCDC1CF4E6B962A03DD4338E1B04A`

作者：风飘雪

主页 `http://duba.126.com`

`e-mail` `gd1@yeah.net` `4dd466d0e1e0cc123afbe504da2462ca`
