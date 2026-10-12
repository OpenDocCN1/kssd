
# 呼吸小秘

> 原文：[`www.pediy.com/kssd/pediy01/forum051.htm`](https://www.pediy.com/kssd/pediy01/forum051.htm)

=====呼吸小秘（`javegirl`）的破解过程。

软件介绍：这是一个做主页特效的小软件。软件不注册有功能限制，感觉其注册检测部分同人物生物节律有点相像，不提示出错信息！

=====这个软件用`aspack1.06`版压过，我用`prodump`脱的壳！

软件将注册信息写入注册表

`HKEY_USER` / `SOFTWARE` / `BREATHSOFT` / `注册` / `javagirl` 下，

每次启动时都做比较。注册码与注册名无关，但必须是 13 位，而且只要前 8 位是 `BSJG08SN`，后 5 位就可以任意啦！

=====破解过程：

1、`trw2000`载入脱壳后的`javagirl.exe`

2、填写注册信息，`name`:`yubing`

`code`:`78787878`

&我第一次追时按习惯只输入了 8 位的注册码，后来发现不对，又改为 13 位，请注意听，我会慢慢地分析:-)

3、下 `ctrl-d`，进入 `trw`，下

```
:bpx hmemcpy
:g
```

4、按“注册键”，程序立即被拦，下指令

```
:bc *
:pmodule
```

5、按 `F10`，只到如下程式：

```
0167:0049A329  MOV EAX,[EBP-04] &下 d eax 可看到你输入的注册码 78787878
0167:0049A32C  CALL 00403C00
0167:0049A331  CMP  EAX,BYTE+0D  &此时 eax 中为注册码个数(8 个), 是否为 13 位。
              JNZ  NEAR 0049A4AF  &如果不相等，则出错。
              LEA  EAX,[EBP-04]
              PUSH EAX
              ...
0167:0049A36C  MOV  EAX,[EAX+0508]
0167:0049A372  CALL  0043BAA8
0167:0049A377  MOV  EDX,[EBP-08]  &下 d edx 可见到注册码 BSJG08SN
              POP  EAX    &下 d eax 可见到输入注册码的前 8 位。
              CALL 00403D10 &比较注册码的核心哟，f8 追入！
              JNZ  NEAR 0049A4AF  &不相等则出错。
              ...
```

6、看注册码比较的部分 `call 00403D10`

```
0167:00403D10   PUSH EBX
              PUSH ESI
              PUSH EDI
              MOV ESI,EAX              &下 d eax 为你输入的注册码
              MOV EDI,EDX             &下 d edx 为正确的注册码
              CMP EAX,EDX             &比较注册码.
0167:00403d19   JZ NEAR 00403DAE      &相等，则跳转哟，一定要跳哟。
      ...................(略)
```

7、至此程序追踪完成，由于注册码必须是 13 位，所以我们前 8 位输入 `BSJG08SN`之后，还必须要任意填 5 位。

cracker：十三少

`scyb@sina.com`

`http://scyb.yeah.net`
