

```text
====>EDX=70
:00486D2C 8A5201                  mov dl, byte ptr [edx+01]
                ====>DL=30
   即：70的后一位字符
:00486D2F E854D5F7FF           call 00404288
:00486D34 8B45C4               mov eax, dword ptr [ebp-3C]
:00486D37 E8E818F8FF              call 00408624
:00486D3C 3B5DDC                  cmp ebx, dword ptr [ebp-24]
                ====>EBX=2
   试炼码的第2位
                ====>[ebp-24]=7
试炼码的第3位HEX值10进制值的末位
                ====>所以注册码的第2位应是7
:00486D3F 750F                    jne 00486D50
            ====>跳则OVER！
:00486D41 3B75D8                  cmp esi, dword ptr [ebp-28]
                ====>ESI=6
   试炼码的第6位
                ====>[ebp-28]=8
试炼码的第4位HEX值10进制值的末位
                ====>所以注册码的第6位应是8
:00486D44 750A                    jne 00486D50
            ====>跳则OVER！
:00486D46 3B7DD4                  cmp edi, dword ptr [ebp-2C]
                ====>EDI=7
   试炼码的第7位
                ====>[ebp-2C]=9
试炼码的第5位HEX值10进制值的末位
                ====>所以注册码的第7位应是9
:00486D49 7505                    jne 00486D50
            ====>跳则OVER！
:00486D4B 3B45E0                  cmp eax, dword ptr [ebp-20]
                ====>EAX=0
    试炼码的第9位HEX值10进制值的末位
 ====>[ebp-20]=8  试炼码的第8位
                       ====>所以注册码的第8位应是0
:00486D4E 740C                    je 00486D5C
         ====>不跳则OVER！

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00486D3F(C), :00486D44(C), :00486D49(C)|
:00486D50 33DB           xor ebx, ebx
:00486D52 33C0                 xor eax, eax
:00486D54 5A                      pop edx
:00486D55 59   pop ecx
:00486D56 59         pop ecx
:00486D57 648910           mov dword ptr fs:[eax], edx
:00486D5A EB1F                    jmp 00486D7B

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486D4E(C)|
:00486D5C 33C0             xor eax, eax
:00486D5E 5A                   pop edx
:00486D5F 59                      pop ecx
:00486D60 59   pop ecx
:00486D61 648910     mov dword ptr fs:[eax], edx
:00486D64 EB13               jmp 00486D79
:00486D66 E901CAF7FF              jmp 0040376C
:00486D6B 33DB                    xor ebx, ebx
:00486D6D E862CDF7FF              call 00403AD4
:00486D72 EB07   jmp 00486D7B
:00486D74 E85BCDF7FF     call 00403AD4

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486D64(U)|
:00486D79 B301                    mov bl, 01
             ====>置1则OK！

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00486C67(U), :00486D5A(U), :00486D72(U)|
:00486D7B 33C0           xor eax, eax
:00486D7D 5A                 pop edx
:00486D7E 59                      pop ecx
:00486D7F 59   pop ecx
:00486D80 648910     mov dword ptr fs:[eax], edx
:00486D83 68B76D4800           push 00486DB7

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486DB5(U)|
:00486D88 8D45C4                  lea eax, dword ptr [ebp-3C]
:00486D8B BA04000000   mov edx, 00000004
:00486D90 E82FD3F7FF     call 004040C4
:00486D95 8D45E4         lea eax, dword ptr [ebp-1C]
:00486D98 BA07000000             mov edx, 00000007
:00486D9D E822D3F7FF              call 004040C4
:00486DA2 8D4508                  lea eax, dword ptr [ebp+08]
:00486DA5 BA05000000   mov edx, 00000005
:00486DAA E815D3F7FF     call 004040C4
:00486DAF C3             ret
:00486DB0 E96BCCF7FF              jmp 00403A20
:00486DB5 EBD1                    jmp 00486D88
:00486DB7 8BC3   mov eax, ebx
                   ====>EBX值入EAX！！
:00486DB9 5F                      pop edi
:00486DBA 5E   pop esi
:00486DBB 5B         pop ebx
:00486DBC 8BE5             mov esp, ebp
:00486DBE 5D                   pop ebp
:00486DBF C21400                  ret 0014
```

---

## 程序的2个功能限制处

```text
\* Possible StringData Ref from Code Obj ->"Yafeisoft"
|
:00485DA6 BAD85F4800   mov edx, 00485FD8
:00485DAB E8B8E5F7FF     call 00404368
:00485DB0 8B55F0         mov edx, dword ptr [ebp-10]
:00485DB3 A15CBC4800             mov eax, dword ptr [0048BC5C]
:00485DB8 E8EFE6F7FF              call 004044AC
:00485DBD 740F                    je 00485DCE
         ====>应跳！爆破点②！

\* Possible StringData Ref from Code Obj ->"Sorry, Only Registered version "
                                     ->"can use this function."
|
:00485DBF B8EC5F4800              mov eax, 00485FEC
:00485DC4 E877B6FAFF              call 00431440
:00485DC9 E98A010000              jmp 00485F58

…… …… 省 略 …… ……

\* Possible StringData Ref from Code Obj ->"Yafeisoft"
|
:00486771 BA9C694800       mov edx, 0048699C
:00486776 E8EDDBF7FF         call 00404368
:0048677B 8B55F0             mov edx, dword ptr [ebp-10]
:0048677E A15CBC4800              mov eax, dword ptr [0048BC5C]
:00486783 E824DDF7FF              call 004044AC
:00486788 740F   je 00486799
                   ====>应跳！爆破点③！

\* Possible StringData Ref from Code Obj ->"Sorry, Only Registered version "
                                     ->"can use this function."
|
:0048678A B8B0694800              mov eax, 004869B0
:0048678F E8ACACFAFF              call 00431440
:00486794 E98C010000              jmp 00486925
```

---

## 算法总结

1. 注册码与用户名无关。需要12位。
   试炼码：Z2CDE678F255
   当然这个试炼码是调试了数十次而不断修改得到的。
2. 注册码最后3位固定为255
3. 注册码第一位应是大写字母，并且根据第一位字符的HEX值-41后的值而跳向不同的位置进行不同判断的运算！我选择第1位是Z，下面的其它注册码情况根据这个分支进行求逆的！
4. 注册码第3、4、5、9位字符的HEX值不能小于41。分别取这几位字符HEX值的10进制值。
5. 注册码第2、6、7、8位应是数字。
6. 注册码的第2位应等于试炼码的第3位HEX值10进制值的末位。
7. 注册码的第6位应等于试炼码的第4位HEX值10进制值的末位。
8. 注册码的第7位应等于试炼码的第5位HEX值10进制值的末位。
9. 注册码的第8位应等于试炼码的第9位HEX值10进制值的末位。

至此可以求逆出一组可用的注册码：Z7CDE890F255

当然，注册码的组合是很多很多的……   只要你愿意找 ^O^ ^O^ ^O^ ^O^

---

## 完美爆破

呵呵，这个东东的完美爆破不太好做呀！

如果仅仅从算法校验里改很容易非法操作的。哪位大侠搞定了请指教！

未注册版本有15天的试用期、NAG、还有2个功能限制。所以我取点巧，从下面几处改，改完后发现没有注册提示了，功能也和注册版本一样了，勉强算是一个完美爆破吧。^v^ ^v^

1. `00484E09 751F jne 00484E2A`
   改为：`9090` NOP掉
   去除时间限制和NAG

2. `00485DBD 740F je 00485DCE`
   改为：`EB0F jmp 00485DCE`
   去除功能限制

3. `00486788 740F je 00486799`
   改为：`EB0F jmp 00486799`
   去除功能限制

---

## 注册信息保存

```text
REGEDIT4

[HKEY_CURRENT_USER\Software\Yafei\ColorFolder]
"Name"="fly"
"RegisterCode"="Z7CDE890F255"
```

---

## 整理

Registartion Name：fly

Registartion Code：Z7CDE890F255   或者：Z0FLY699Y255 …… 很多……

……

---

```text
         ,     \_/
        /| \_.-~/            \\_
    ,        青春都一饷
       ( /~
  /              \~-.\_ |\
  `\\  \_/                \
~\ )          忍把浮名
   \_-~~~-.)  )\_\_/;;,.
         \\_  //'
  /'\_,\   --~
\ ~~~-  ,;;\\_\_\_(  (.-~~~-.        换了破解轻狂
`~ \_( ,\_..--\ (     ,;'' /    ~--   /.\_`\
  /~~//'
  /' `~\         ) /--..\_, )\_  `~
  "
 `~"  "      `"      /~'`\
   `\\~~\
          "     "   "~'
 ""
                 Cracked By 巢水工作坊――fly
[OCN][FCG]
                2003-05-07  22:00
```

