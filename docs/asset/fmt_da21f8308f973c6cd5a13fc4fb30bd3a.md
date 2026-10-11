
## 代码逻辑分析

```
for i = 1 to slen1

    tmpint = asc(mid(mynamestr,i,1))

    myarray5(var\_fc) = (myarray5(var\_fc) + tmpint) xor &h12

    s1sum = s1sum + myarray5(var\_fc)

    var\_fc = var\_fc + 1

    if var\_fc = 9 then var\_fc = 1

next i

for i = 1 to slen2

    tmpint = asc(mid(mynamestr2,i,1))

    myarray3(var\_fc) = (myarray3(var\_fc) + tmpint) xor &h19

    s2sum = s2sum + myarray3(var\_fc)

    var\_fc = var\_fc + 1

    if var\_fc = 9 then var\_fc = 1

next i

var\_fc = 1

var\_f0 = 1

var\_b0 = (s1sum + s2sum) mod 512

for i = 1 to var\_48

    myarray3(i) = myarray3(i) xor myarray1(i)

    var\_e4 = abs((myarray5(i) xor myarray3(i)) mod 512 - var\_b0)

    if var\_e8 = 3 then

        if var\_e4 < 16 then

            mid(var\_50,var\_fc,2) = ("0" & hex(var\_e4))

        else

            mid(var\_50,var\_fc,2) = hex(var\_e4)

        end if

        if var\_f0 = 2 and var\_fc < 18 then

            var\_fc = var\_fc + 1

            mid(var\_50,var\_fc+1,1) = "-"

        end if

        var\_fc = var\_fc + 2

        var\_f0 = var\_f0 + 1

        if var\_f0 = 3 then var\_f0 = 1

    else

        mid(var\_50,i,1) = chr(myarray2(var\_e4))

    end if

next i

if strcomp(var\_50,mycode) = 0 and strcomp(myname,mynamestr) = 0 then

    fun464ef0 = true

else

    fun464ef0 = false

end if

end function
```

呵呵。长了点。但还没有完。看看上面还有两个函数需要跟进，

第一个`fun4678D0`是对用户名做变换的。由于这个结果还要和用户名比较。所以能否算出注册码这个`fun4678D0`调用是关键。

第二个`fun468060`函数调用是对`fun4678d0`的结果再做一次变换，算法和外面这个函数调用是相同的。

先看看`fun468060`，我把对用户名变换的第一个函数调用放在后面，方便后面分析。

## 汇编代码分析

```
00468060 sub\_468060      proc near                  ; CODE XREF: sub\_464EF0+107p

.text:00468060

.text:00468060 var\_248             = dword ptr -248h

.text:00468060 var\_244             = dword ptr -244h

.text:00468060 var\_240             = dword ptr -240h

.text:00468060 var\_23C             = dword ptr -23Ch

.text:00468060 var\_238             = dword ptr -238h

.text:00468060 var\_234             = dword ptr -234h

.text:00468060 var\_230             = dword ptr -230h

.text:00468060 var\_22C             = dword ptr -22Ch

.text:00468060 var\_228             = dword ptr -228h

.text:00468060 var\_224             = dword ptr -224h

.text:00468060 var\_220             = dword ptr -220h

.text:00468060 var\_21C             = dword ptr -21Ch

.text:00468060 var\_218             = dword ptr -218h

.text:00468060 var\_214             = dword ptr -214h

.text:00468060 var\_210             = dword ptr -210h

.text:00468060 var\_20C             = dword ptr -20Ch

.text:00468060 var\_208             = dword ptr -208h

.text:00468060 var\_204             = dword ptr -204h

.text:00468060 var\_200             = dword ptr -200h

.text:00468060 var\_1FC             = dword ptr -1FCh

.text:00468060 var\_1F8             = dword ptr -1F8h

.text:00468060 var\_1F4             = dword ptr -1F4h

.text:00468060 var\_1F0             = dword ptr -1F0h

.text:00468060 var\_1EC             = dword ptr -1ECh

.text:00468060 var\_1E8             = dword ptr -1E8h

.text:00468060 var\_1E4             = dword ptr -1E4h

.text:00468060 var\_1E0             = dword ptr -1E0h

.text:00468060 var\_1DC             = dword ptr -1DCh

.text:00468060 var\_1D8             = dword ptr -1D8h

.text:00468060 var\_1D4             = dword ptr -1D4h

.text:00468060 var\_1D0             = dword ptr -1D0h

.text:00468060 var\_1CC             = dword ptr -1CCh

.text:00468060 var\_1C8             = dword ptr -1C8h

.text:00468060 var\_1C4             = dword ptr -1C4h

.text:00468060 var\_1C0             = dword ptr -1C0h

.text:00468060 var\_1BC             = dword ptr -1BCh

.text:00468060 var\_1B8             = dword ptr -1B8h

.text:00468060 var\_1B4             = dword ptr -1B4h

.text:00468060 var\_1B0             = dword ptr -1B0h

.text:00468060 var\_198             = word ptr -198h

.text:00468060 var\_194             = word ptr -194h

.text:00468060 var\_190             = word ptr -190h

.text:00468060 var\_18C             = word ptr -18Ch

.text:00468060 var\_188             = word ptr -188h

.text:00468060 var\_184             = word ptr -184h

.text:00468060 var\_180             = word ptr -180h

.text:00468060 var\_17C             = word ptr -17Ch

.text:00468060 var\_178             = word ptr -178h

.text:00468060 var\_174             = word ptr -174h

.text:00468060 var\_170             = word ptr -170h

.text:00468060 var\_16C             = word ptr -16Ch

.text:00468060 var\_168             = word ptr -168h

.text:00468060 var\_164             = word ptr -164h

.text:00468060 var\_160             = dword ptr -160h

.text:00468060 var\_15C             = dword ptr -15Ch

.text:00468060 index\_158           = dword ptr -158h

.text:00468060 var\_154             = dword ptr -154h

.text:00468060 var\_150             = dword ptr -150h

.text:00468060 var\_14C             = dword ptr -14Ch

.text:00468060 var\_148             = dword ptr -148h

.text:00468060 var\_144             = dword ptr -144h

.text:00468060 var\_140             = dword ptr -140h

.text:00468060 var\_138             = dword ptr -138h

.text:00468060 var\_130             = dword ptr -130h

.text:00468060 var\_128             = dword ptr -128h

.text:00468060 var\_120             = dword ptr -120h

.text:00468060 var\_110             = dword ptr -110h

.text:00468060 var\_108             = dword ptr -108h

.text:00468060 var\_100             = dword ptr -100h

.text:00468060 var\_FC              = word ptr -0FCh

.text:00468060 var\_F8              = dword ptr -0F8h

.text:00468060 var\_F4              = word ptr -0F4h

.text:00468060 time\_F0             = dword ptr -0F0h

.text:00468060 var\_EC              = word ptr -0ECh

.text:00468060 var\_E8              = dword ptr -0E8h

.text:00468060 var\_E4              = word ptr -0E4h

.text:00468060 array5\_DC           = dword ptr -0DCh

.text:00468060 var\_D0              = dword ptr -0D0h

.text:00468060 var\_C4              = word ptr -0C4h

.text:00468060 var\_C0              = word ptr -0C0h

.text:00468060 var\_BC              = word ptr -0BCh

.text:00468060 arg0Len\_B8          = word ptr -0B8h

.text:00468060 mysum\_B4            = word ptr -0B4h

.text:00468060 var\_B0              = dword ptr -0B0h

.text:00468060 array4\_A8           = dword ptr -0A8h

.text:00468060 var\_9C              = dword ptr -9Ch

.text:00468060 array3\_8C           = dword ptr -8Ch

.text:00468060 var\_80              = dword ptr -80h

.text:00468060 array2\_70           = dword ptr -70h

.text:00468060 var\_64              = dword ptr -64h

.text:00468060 var\_58              = word ptr -58h

.text:00468060 var\_54              = word ptr -54h

.text:00468060 var\_50              = dword ptr -50h

.text:00468060 var\_4C              = word ptr -4Ch

.text:00468060 var\_48              = word ptr -48h

.text:00468060 var\_44              = dword ptr -44h

.text:00468060 array1\_40           = dword ptr -40h

.text:00468060 var\_3C              = dword ptr -3Ch

.text:00468060 var\_34              = dword ptr -34h

.text:00468060 var\_30              = dword ptr -30h

.text:00468060 var\_28              = word ptr -28h

.text:00468060 constLen\_24         = word ptr -24h

.text:00468060 var\_20              = dword ptr -20h

.text:00468060 var\_18              = dword ptr -18h

.text:00468060 var\_14              = dword ptr -14h

.text:00468060 var\_10              = dword ptr -10h

.text:00468060 var\_C               = dword ptr -0Ch

.text:00468060 var\_8               = dword ptr -8

.text:00468060 var\_4               = dword ptr -4

.text:00468060 arg\_0               = dword ptr  8

.text:00468060 arg\_4               = dword ptr  0Ch

.text:00468060

.text:00468060                    push    ebp

.text:00468061                    mov     ebp, esp

.text:00468063                    sub     esp, 18h

.text:00468066                    push    offset loc\_4020A6

.text:0046806B                    mov     eax, large fs:0

.text:00468071                    push    eax

.text:00468072                    mov     large fs:0, esp
```
