
```
:004CCF87 E8D882FBFF              call 00485264

:004CCF8C 8BD8

  mov ebx, eax

:004CCF8E BA00000080              mov edx,
80000000

:004CCF93 8BC3

  mov eax, ebx

:004CCF95 E86A83FBFF              call 00485304

:004CCF9A 8D85A8FEFFFF            lea eax, dword
ptr [ebp+FFFFFEA8]

:004CCFA0 E83FD1FEFF              call 004BA0E4

:004CCFA5 8B95A8FEFFFF            mov edx, dword
ptr [ebp+FFFFFEA8]

:004CCFAB 8BC3

  mov eax, ebx

:004CCFAD E8B683FBFF              call 00485368

:004CCFB2 8D85A4FEFFFF            lea eax, dword
ptr [ebp+FFFFFEA4]

:004CCFB8 E827D1FEFF              call 004BA0E4

:004CCFBD 8B95A4FEFFFF            mov edx, dword
ptr [ebp+FFFFFEA4]

:004CCFC3 B101

  mov cl, 01

:004CCFC5 8BC3

  mov eax, ebx

:004CCFC7 E87884FBFF              call 00485444

:004CCFCC 8D85A0FEFFFF            lea eax, dword
ptr [ebp+FFFFFEA0]

:004CCFD2 E8A9F4FEFF              call 004BC480

:004CCFD7 8B8DA0FEFFFF            mov ecx, dword
ptr [ebp+FFFFFEA0]

:004CCFDD 33D2

  xor edx, edx

:004CCFDF 8BC3

  mov eax, ebx

:004CCFE1 E8FA85FBFF              call 004855E0

:004CCFE6 8BC3

  mov eax, ebx

:004CCFE8 E8E782FBFF              call 004852D4

:004CCFED 8D859CFEFFFF            lea eax, dword
ptr [ebp+FFFFFE9C]

:004CCFF3 E8F0D9FEFF              call 004BA9E8

:004CCFF8 8B959CFEFFFF            mov edx, dword
ptr [ebp+FFFFFE9C]

:004CCFFE 8BC3

  mov eax, ebx

:004CD000 E86383FBFF              call 00485368

:004CD005 8D8598FEFFFF            lea eax, dword
ptr [ebp+FFFFFE98]

:004CD00B E8D8D9FEFF              call 004BA9E8

:004CD010 8B9598FEFFFF            mov edx, dword
ptr [ebp+FFFFFE98]

:004CD016 B101

  mov cl, 01

:004CD018 8BC3

  mov eax, ebx

:004CD01A E82584FBFF              call 00485444

:004CD01F 8D8594FEFFFF            lea eax, dword
ptr [ebp+FFFFFE94]

:004CD025 E8D6D4FEFF              call 004BA500

:004CD02A 8B8D94FEFFFF            mov ecx, dword
ptr [ebp+FFFFFE94]

:004CD030 33D2

  xor edx, edx

:004CD032 8BC3

  mov eax, ebx

:004CD034 E8A785FBFF              call 004855E0

:004CD039 8BC3

  mov eax, ebx

:004CD03B E89482FBFF              call 004852D4

:004CD040 8BC3

  mov eax, ebx

:004CD042 E8B55EF3FF              call 00402EFC

:004CD047 C605880E4E0001          mov byte ptr [004E0E88],
01

:004CD04E B87C0E4E00              mov eax,
004E0E7C

:004CD053 E86C6BF3FF              call 00403BC4

:004CD058 B8800E4E00              mov eax,
004E0E80

:004CD05D E8626BF3FF              call 00403BC4
```
* Possible StringData Ref from Code Obj ->`"恭喜！您已注册为付费用户了。"`

            |

:004CD062 B814D44C00              mov eax,
004CD414

:004CD067 E8C0CBF9FF              call 00469C2C

:004CD06C 8B8650050000            mov eax, dword
ptr [esi+00000550]
* Possible StringData Ref from Code Obj ->`"您已注册为付费用户了"`

可以使用`UltraEdit`进行修改，修改完毕后，就是真正的“破解版”了。

注册时需在线，点击用户注册选项，输入盘符(一般填“c”)，并随便输入假用户注册码(20位任意数字)，点击“付费用户注册”，注册成功！

太棒了！

By the way，这个版本(`V2.4`)的软件中有不少Bug，例如：发送出去的邮件的“发送时间”不对。

马震宇

2001.8.17.
