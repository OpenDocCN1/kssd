
# SVKP 中的一些花指令

> 原文：`www.pediy.com/kssd/pediy06/pediy6831.htm`

在 `DeJunk.ini` 中加入如下信息，在跟踪 `SVKP` 可以轻松一点。

`JunkType=SVKP1,Common,TELock,UltraProtect,Custom`

`PatList_SVKP1=_SVKP_S1,_SVKP_S2,_SVKP1,_SVKP2,_SVKP3,_SVKP4,_SVKP97,_SVKP98,_SVKP99,_SVKP100`

`[CODE_SVKP1]`

```
; call loc_1
; _ONE_BYTE_JUNKCODE_
;loc_1: call loc_2
; _TWO_BYTE_JUNKCODE_
;loc_2: add dword ptr [esp], 0Bh
; add dword ptr [esp+04h], 13h
; ret
; _ONE_BYTE_JUNKCODE_

S = E801000000??E802000000????8304240B8344240413C3??
R = 909090909090909090909090909090909090909090909090
```

`[CODE_SVKP2]`

```
; call loc_1
; _TWO_BYTE_JUNKCODE_
;loc_1: add dword ptr [esp], 08h
; ret
; _ONE_BYTE_JUNKCODE_

S = E802000000????83042408C3??
R = 90909090909090909090909090
```

`[CODE_SVKP3]`

```
; pushad
; call loc_1
; _ONE_BYTE_JUNKCODE_
; jmp loc_4
;loc_1: pop eax
; jmp loc_2
; _ONE_BYTE_JUNKCODE_
;loc_2: inc eax
; jmp loc_3
; _ONE_BYTE_JUNKCODE_
;loc_3: jmp eax
;loc_4: popad

S = 60E803000000??EB0A58EB01??40EB01??FFE061
R = 9090909090909090909090909090909090909090
```

`[CODE_SVKP4]`

```
; jl loc_1
;loc_2: jmp loc_3
; _ONE_BYTE_JUNKCODE_
;loc_1: jz loc_2
;loc_3: ...

S = 7C03EB03??74FB
R = 90909090909090
```

`[CODE_SVKP97]`

```
; call loc_1
; _ONE_BYTE_JUNKCODE_
; add esp, 4h

S = E801000000??83C404
R = 909090909090909090
```

`[CODE_SVKP98]`

```
; jmp loc_1
; mov ..
;loc_1: ....

S = EB03C7????
R = 9090909090
```

`[CODE_SVKP99]`

```
; jmp loc_1
; int 20h
;loc_1: ....

S = EB02CD20
R = 90909090
```

`[CODE_SVKP100]`

```
; jmp loc_1
; _TWO_BYTE_JUNKCODE_
;loc_1: ...

S = EB020F??
R = 90909090
```
