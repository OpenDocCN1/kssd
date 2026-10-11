
### 汇编代码分析

```assembly
.text:100087D6                 add    ecx, eax                     ====> Regcode(13)*3+Regcode(17)*4+Regcode(9)+Regcode(18)

.text:100087D8                 movsx  eax, byte ptr [esp+40h+var_C+2]     ====> eax="X"

.text:100087DD                 add    ecx, ebx                     ====> Regcode(13)*3+Regcode(17)*4+Regcode(9)+Regcode(18)+Regcode(12)

.text:100087DF                 lea    esi, [ecx+edi+0Eh]           ====> Regcode(8)+Regcode(13)*3+Regcode(17)*4+Regcode(9)+Regcode(18)+Regcode(12)+0Eh

.text:100087E3                 mov    ecx, [esp+40h+var_C]         ====> ecx="DM"

.text:100087E7                 movsx  edx, ch                      ====> edx="M"

.text:100087EA                 movsx  ecx, cl                      ====> ecx="D"

.text:100087ED                 add    eax, edx

.text:100087EF                 add    eax, ecx                     ====> eax="D"+"M"+"X"=E9h

.text:100087F1                 mov    ecx, 3E8h                    ====> ecx=3E8h=1000

.text:100087F6                 lea    eax, [eax+eax*4]             ====> eax*5=48Dh

.text:100087F9                 shl    eax, 2                       ====> eax*4=1234h

.text:100087FC                 cdq

.text:100087FD                 idiv   esi                          ====> 整除

.text:100087FF                 add    eax, 73h                     ====> 商+73h

.text:10008802                 cdq

.text:10008803                 idiv   ecx                          ====> 除以 1000，取余数

.text:10008805                 mov    al, [ebp+0Dh]                ====> Regcode(14)

.text:10008808                 mov    cl, [ebp+12h]                ====> Regcode(19)

.text:1000880B                 mov    [esp+40h+var_13], al

.text:1000880F                 mov    al, [ebp+9]                  ====> Regcode(10)

.text:10008812                 mov    [esp+40h+var_12], cl

.text:10008816                 mov    cl, [ebp+0Eh]                ====> Regcode(15)

.text:10008819                 mov    [esp+40h+var_F], al

.text:1000881D                 mov    [esp+40h+var_E], cl

.text:10008821                 mov    esi, edx                     ====> esi=余数

.text:10008823                 mov    dl, [ebp+6]                  ====> Regcode(7)

.text:10008826                 mov    [esp+40h+var_14], dl

.text:1000882A                 mov    dl, [ebp+13h]                ====> Regcode(20)

.text:1000882D                 mov    [esp+40h+var_10], dl

.text:10008831                 lea    edx, [esp+40h+var_14]       ====> 位置顺序：R(7)R(14)R(19)

.text:10008835                 push   edx

.text:10008836                 call   _atoi                        ====> 转为整数

.text:1000883B                 mov    edi, eax                     ====> 放到 edi 中

.text:1000883D                 lea    eax, [esp+44h+var_10]       ====> 位置顺序：R(20)R(10)R(15)

.text:10008841                 push   eax

.text:10008842                 call   _atoi                        ====> 转为整数

.text:10008847                 add    esp, 20h

.text:1000884A                 mov    ecx, eax                     ====> 放到 ecx 中

.text:1000884C                 cmp    edi, esi                     ====> 与余数比较

.text:1000884E                 jnz    loc_100088D6                 ====> 不等则错误

.text:10008854                 mov    eax, 55555556h

.text:10008859                 imul   esi                           ====> 乘以余数

.text:1000885B                 mov    eax, edx                     ====> 取高位数放到 eax 中

.text:1000885D                 mov    esi, 3E8h

.text:10008862                 shr    eax, 1Fh                     ====> 右移 1F 位，即除以 2³¹

.text:10008865                 lea    eax, [edx+eax+159h]          ====> +345(159h)

.text:1000886C                 cdq

.text:1000886D                 idiv   esi                           ====> 除以 1000

.text:1000886F                 cmp    ecx, edx                     ====> 进行比较

.text:10008871                 jnz    short loc_100088D6           ====> 不等则错误

.text:10008873                 mov    edx, [esp+28h+arg_4]         ====> 用户名

.text:10008877                 or     ecx, 0FFFFFFFFh

.text:1000887A                 mov    edi, edx

.text:1000887C                 xor    eax, eax

.text:1000887E                 repne scasb

.text:10008880                 not    ecx

.text:10008882                 dec    ecx

.text:10008883                 test   ecx, ecx                     ====> 用户名是否为空

.text:10008885                 jle    short loc_100088D6           ====> 为空则错误

:

:

:

.text:100088C2                 mov    eax, 1                       ====> 成功的旗标 eax=1

.text:100088C7                 and    ecx, 3

.text:100088CA                 repe movsb

.text:100088CC                 pop    edi

.text:100088CD                 pop    esi

.text:100088CE                 pop    ebp

.text:100088CF                 pop    ebx

.text:100088D0                 add    esp, 18h

.text:100088D3                 retn   8

.text:100088D6                 pop    edi

.text:100088D7                 pop    esi

.text:100088D8                 pop    ebp

.text:100088D9                 xor    eax, eax                     ====> 失败的旗标 eax=0

.text:100088DB                 pop    ebx

.text:100088DC                 add    esp, 18h

.text:100088DF                 retn   8

.text:100088DF sub_100086A0   endp
```

### 注册码计算过程

注册码的前 5 个字符必须为`DMX20`。

```
设 X = Regcode(8) + Regcode(13) * 3 + Regcode(17) * 4 + Regcode(9) + Regcode(18) + Regcode(12) + 14

Y = (hex("D") + hex("M") + hex("X")) * 20

Z = ((Y \ X) + 115) mod 1000

Z = Regcode(7,14,19)

(Z * 55555556h 的高位 + 345) mod 1000 = Regcode(20,10,15)
```

根据上面的计算过程，随机生成第 8、9、12、13、17、18 和 6、11、16 位的注册码后，再推算出第 7、10、14、15、19、20 位的注册码即可，注册机已做成。

### 注册文件说明

注册成功后，会在`DirectMediaXtra.x32`所在的文件夹下生成一个名为`dmx2.lic`的隐藏注册文件，里面就是注册名和注册码，可以用文本编辑器打开，以后只要把这个注册文件复制到相应文件夹即可。

### 附记

该公司的另一个插件，`WebXtra` 3.0 的注册过程跟`DirectMediaXtra.x32`完全一样，只是稍有不同，有兴趣的可以试一试，至于下载 url，我没有，是`CrackerABC` email 给我的，可以找他联系 `4dd466d0e1e0cc123afbe504da2462ca`。
