
```
>       push 004099EC

>  :0040132E 68F0994000

>         push 004099F0

>  :00401333 6880894000

>           push 00408980

>  :00401338 C705F0994000626C6F77

>   mov dword ptr [004099F0], 776F6C62

>  :00401342 C705EC99400066697368

>   mov dword ptr [004099EC], 68736966

>  :0040134C E81FFDFFFF

>           call 00401070===>BF_Enc

>  =====>BF_Enc("blowfish",key=ProductID)

>

>  ======================================================================

> ======================================================================

>  =====>最后分析结果

>

>  ComputerID=BF_Enc("blowfish",key=ProductID)

>

>  x=BF_Dec(Code,key="CrackingForFun")

>

>  y=BF_Enc(ComputerID,key="ChinaCrackingGroup")

>  x=y则注册成功；

>

>  我们要得到正确的注册码，那么

>

>  Code=BF_Enc(x,key="CrackingForFun");

>    =BF_Enc(y,key="CrackingForFun");

>

>    =BF_Enc(BF_Enc(ComputerID,key="ChinaCrackingGroup"),key="CrackingForFun");

>

>    如果更进一步，那么

>    =BF_Enc(BF_Enc(BF_Enc("blowfish",

>

> key=ProductID),

> key="ChinaCrackingGroup"),

>            key="CrackingForFun");

>

>  这样我们便可以编写它的keygen了

>  =====>

>  ======================================================================
```
