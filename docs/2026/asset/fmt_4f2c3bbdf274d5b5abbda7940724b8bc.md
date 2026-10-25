
## 反汇编分析

```
test dl, dl
```

`:0043160C`

`7426`                    `je 00431634`

*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00431632(C)`

|

`:0043160E`

`0FBED2`                  `movsx edx, dl`

`====>` 用户名的HEX值依次入`EDX`

`:00431611`  `41`

                     `inc ecx`

             `====>` `ECX`依次增1。

`:00431612`

`0FAFD1`                  `imul edx, ecx`

             `====>` 用户名字符值依次与次数相乘

       `====>` 1、`EDX`= `66`*1=`66`

 `====>` 2、`EDX`= `6C`*2=`D8`

                   `====>` 3、`EDX`= `79`*3=`16B`

`:00431615`

`03F2`                    `add esi, edx`

             `====>` 先加`BDF`，再累加！

       `====>` 1、`ESI`= `66` +`BDF`=`C45`

   `====>` 2、`ESI`= `D8` +`C45`=`D1D`

                       `====>` 3、`ESI`= `16B`+`D1D`=`E88`

`:00431617`

`81FEBE170000`            `cmp esi, 000017BE`

       `====>` 所得结果与`17BE`比较

`:0043161D`  `7E06`

               `jle 00431625`

       `====>` 小于则跳

`:0043161F`  `81EEBE170000`

       `sub esi, 000017BE`

                       `====>` 大于`17BE`则再减去`17BE`

*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0043161D(C)`

|

`:00431625`

`83F90A`                  `cmp ecx, 0000000A`

`:00431628`  `7E02`

   `jle 0043162C`

`:0043162A`  `33C9`

         `xor ecx, ecx`

*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00431628(C)`

|

`:0043162C`

`8A5701`                  `mov dl, byte ptr [edi+01]`

               `====>` 继续取下一位字符

`:0043162F`

`47`                      `inc edi`

`:00431630`  `84D2`

   `test dl, dl`

`:00431632`  `75DA`

         `jne 0043160E`

 `====>` 继续循环，直至取完所有用户名字符

*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0043160C(C)`

|

`:00431634`

`3BF0`                    `cmp esi, eax`

             `====>` 最后所得结果与`EAX`比较

       `EAX`就是试炼码第7位后的数字，所以？ `ESI`=1720我们的最后几位真码！

`:00431636`  `750A`

       `jne 00431642`

                       `====>` 跳则`OVER`！

`:00431638`

`5F`                      `pop edi`

`:00431639`  `5E`

   `pop esi`

`:0043163A`  `5D`

         `pop ebp`

`:0043163B`  `B801000000`

         `mov eax, 00000001`

 `====>` 喜欢看到这儿的1，呵呵！

`:00431640`  `5B`

           `pop ebx`

`:00431641`  `C3`

                 `ret`

*

Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:0043159C(C)`,

`:004315A6(C)`, `:004315B0(C)`, `:004315BA(C)`, `:004315DB(C)`

`:00431636(C)`

|

`:00431642`

`5F`                      `pop edi`

`:00431643`  `5E`

   `pop esi`

`:00431644`  `5D`

         `pop ebp`

`:00431645`  `33C0`

             `xor eax, eax`

     `====>` 跳到这儿！`EAX`清0，就`OVER`了。

          若你还想做个完美破解版，就改这儿最好了，只要改此一处！

`:00431647`  `5B`

                     `pop ebx`

`:00431648`  `C3`

   `ret`

―――――――――――――――――――――――――――――

### 完美爆破

只要改此一处即可！

`431645`

`33C0`    `xor eax, eax`

改为：`431645`

`INC eax`  即：`33C0` -->`FEC0`

让其返回的`EAX`值永远是1。

呵呵，与上面的`43163B`  `mov eax, 00000001`相映成趣。

―――――――――――――――――――――――――――――

### 总结

注册码的前4位固定为：`mg37`（第5位若是`s`，则为`Site License`版本）

第5、6、7位可为任意数，第7位后的数字为用户名与程序给的数字进行简单运算得出！

（第5位若是`s`，则第6、7、8位可为任意数，第8位后的数字为用户名与程序给的数字进行简单运算得出！）

―――――――――――――――――――――――――――――

### 注册信息保存

`[HKEY_CURRENT_USER\Software\gamani\GIFMovieGear\2.0]`

`"RegName3"="fly"`

`"RegCode3"="mg372463720"`

―――――――――――――――――――――――――――――

### 整理

`Name`：`fly`

`Code`：`mg372463720`

或者：

`Name=fly`

`Code=mg37s1233720`

`====>` `Site License`版本

―――――――――――――――――――――――――――――

`Cracked By`

巢水工作坊――`fly`【`OCN`】

`2003-2-6`   `21:40`
