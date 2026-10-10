

# 菜鸟也做注册机(三)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-35.htm`](https://www.pediy.com/kssd/pediy07/pediy7-35.htm)

【作者声明】：初学破解，仅作学习交流之用，失误之处敬请大侠赐教!  
  
【破解工具】：Ollydbg1.10、Hiew6.81  
  
各位好，很感谢您能继续看我的破文!这次让大家看一下懒人如何做注册机!  
一如既往的三板斧，载入程序；任意填入注册信息用户名:sharpair 假码:65148455；搜索字符参考，定位到以下代码：  
  
00427B5C   lea edx,dword ptr ss:\[ebp-4\]  
00427B5F   mov eax,dword ptr ds:\[ebx+1DC\]  
00427B65   call echap515.00415D90          ;  取用户名  
00427B6A   mov eax,dword ptr ss:\[ebp-4\]    ;  eax="sharpair"  
00427B6D   call echap515.004037B0          ;  计算用户名的长度  
00427B72   dec eax  
00427B73   jl short echap515.00427BA5      ;  如果没有输入用户名则跳  
00427B75   lea edx,dword ptr ss:\[ebp-4\]  
00427B78   mov eax,dword ptr ds:\[ebx+1EC\]  
00427B7E   call echap515.00415D90          ;  同上，这个 Call 取用户输入的注册码  
00427B83   mov eax,dword ptr ss:\[ebp-4\]    ;  eax="65148455"  
00427B86   push eax  
00427B87   lea edx,dword ptr ss:\[ebp-8\]  
00427B8A   mov eax,dword ptr ds:\[ebx+1DC\]  
00427B90   call echap515.00415D90          ;  又取用户名，不管，往下看  
00427B95   mov eax,dword ptr ss:\[ebp-8\]  
00427B98   pop edx  
00427B99   call echap515.00427A20          ;  下面就是一个判断和跳转，看来这个 Call 要跟进去  
00427B9E   cmp eax,0BC614E                 ;  0xBC614E 的十进制形式为 12345678  
00427BA3   jge short echap515.00427BC3  
00427BA5   push 0  
00427BA7   push echap515.00427C08          ;  ASCII "ERROR"  
00427BAC   push echap515.00427C10          ;  ASCII "Wrong Serial Number !"  
00427BB1   mov eax,dword ptr ds:\[429744\]  
00427BB6   call echap515.004199FC  
00427BBB   push eax                        ; |hOwner  
00427BBC   call <jmp.&user32.MessageBoxA>  ; \\MessageBoxA  
00427BC1   jmp short echap515.00427BDF  
00427BC3   push 0  
00427BC5   push echap515.00427C28          ;  ASCII "Success"  
00427BCA   push echap515.00427C30          ;  ASCII "Congratulation ! You've Did It.  
Mail Us : ekhmail@egroups.com"  
00427BCF   mov eax,dword ptr ds:\[429744\]  
00427BD4   call echap515.004199FC  
00427BD9   push eax                        ; |hOwner  
00427BDA   call <jmp.&user32.MessageBoxA>  ; \\MessageBoxA  
00427BDF   xor eax,eax  
00427BE1   pop edx  
00427BE2   pop ecx  
00427BE3   pop ecx  
  
不用说，跟进 00427B99 处的 Call,我们看看里面都有些什么?  
00427A20   push ebp  
00427A21   mov ebp,esp  
00427A23   add esp,-10  
00427A26   push ebx  
00427A27   push esi  
00427A28   xor ecx,ecx  
00427A2A   mov dword ptr ss:\[ebp-10\],ecx       ;  一些初值操作  
00427A2D   mov dword ptr ss:\[ebp-C\],ecx  
00427A30   mov dword ptr ss:\[ebp-8\],edx        ;  用户名和注册码存放到堆栈中  
00427A33   mov dword ptr ss:\[ebp-4\],eax  
00427A36   mov eax,dword ptr ss:\[ebp-4\]  
00427A39   call echap515.00403964              ;  这个 Call 干什么呢?  
00427A3E   mov eax,dword ptr ss:\[ebp-8\]  
00427A41   call echap515.00403964  
00427A46   xor eax,eax  
00427A48   push ebp  
00427A49   push echap515.00427B2A  
00427A4E   push dword ptr fs:\[eax\]  
00427A51   mov dword ptr fs:\[eax\],esp  
00427A54   xor ebx,ebx  
00427A56   mov eax,dword ptr ss:\[ebp-4\]  
00427A59   call echap515.004037B0              ;  取用户名的长度  
00427A5E   mov esi,eax  
00427A60   test esi,esi  
00427A62   jle short echap515.00427AA0         ;  判断用户名是否为空  
00427A64   mov eax,1  
00427A69   /mov edx,eax  
00427A6B   |mov ecx,dword ptr ss:\[ebp-4\]       ;  ecx="sharpair"  
00427A6E   |movzx ecx,byte ptr ds:\[ecx+edx-1\]  ;  按位取用户名  
00427A73   |add ebx,ecx                        ;  每个字符的 Hex 进制值累加到 ebx  
00427A75   |jno short echap515.00427A7C        ;  这个判断有点意思，判断其值是否为偶?  
00427A77   |call echap515.00402A30  
00427A7C   |shl ebx,8  
00427A7F   |mov ecx,dword ptr ds:\[428880\]      ;  ds:\[428880\]中为一内置串"LANNYDIBANDINGINANAKEKHYANGNGENTOT"  
00427A85   |movzx edx,byte ptr ds:\[ecx+edx-1\]  ;  按位取内置串中的字符的 Hex 值到 edx 中  
00427A8A   |or ebx,edx  
00427A8C   |test ebx,ebx  
00427A8E   |jge short echap515.00427A9C        ;  这个是判断什么呢？ebx 为正?  
00427A90   |imul edx,ebx,-1  
00427A93   |jno short echap515.00427A9A  
00427A95   |call echap515.00402A30  
00427A9A   |mov ebx,edx  
00427A9C   |inc eax  
00427A9D   |dec esi  
00427A9E   \\jnz short echap515.00427A69        ;  判断用户名是否取完  
00427AA0   xor ebx,12345678                    ;  ebx 中的结果与 12345678 或  
00427AA6   lea edx,dword ptr ss:\[ebp-10\]  
00427AA9   mov eax,ebx  
00427AAB   call echap515.004063F4              ;  这个 Call 实现 Hex 值转换为 Dec 值  
00427AB0   mov eax,dword ptr ss:\[ebp-10\]       ;  eax=1466307270(上面 ebx 中值的 Dec 形式)  
00427AB3   call echap515.004037B0              ;  这个 Call 上面出现过，求字串长度  
00427AB8   mov esi,eax                         ;  此时 esi=eax=0A(10 位)  
00427ABA   test esi,esi  
00427ABC   jle short echap515.00427AF6  
00427ABE   /mov eax,ebx                        ;  又对上面第一次循环出的 ebx 进行运算  
00427AC0   |mov ecx,0A  
00427AC5   |cdq  
00427AC6   |idiv ecx  
00427AC8   |bound edx,qword ptr ds:\[427B3C\]    ;  这个 bound 指令检查数组边界  
00427ACE   |mov dl,byte ptr ds:\[edx+428884\]    ;  还是在那个内置串表中取值  
00427AD4   |lea eax,dword ptr ss:\[ebp-10\]  
00427AD7   |call echap515.004036D8             ;  下面这一小段初跟了一下,没什么发现,不管了(lazy!)  
00427ADC   |mov edx,dword ptr ss:\[ebp-10\]  
00427ADF   |lea eax,dword ptr ss:\[ebp-C\]  
00427AE2   |call echap515.004037B8  
00427AE7   |mov eax,ebx  
00427AE9   |mov ecx,0A  
00427AEE   |cdq  
00427AEF   |idiv ecx  
00427AF1   |mov ebx,eax  
00427AF3   |dec esi  
00427AF4   \\jnz short echap515.00427ABE  
00427AF6   mov eax,dword ptr ss:\[ebp-C\]        ;  这里是最后查内置表得到的真码"L4N4LN66YA"  
00427AF9   mov edx,dword ptr ss:\[ebp-8\]        ;  这里是我们输入的假码  
00427AFC   call echap515.004038C0              ;  下面不用我说了  
00427B01   jnz short echap515.00427B0A  
00427B03   mov ebx,0BC614E                     ;  作者有点意思，注册码正确则将 ebx 赋值 0xB614E,不是通常的标志位 0 或 1  
00427B08   jmp short echap515.00427B0F  
00427B0A   mov ebx,12D691  
00427B0F   xor eax,eax  
00427B11   pop edx  
00427B12   pop ecx  
00427B13   pop ecx  
00427B14   mov dword ptr fs:\[eax\],edx  
00427B17   push echap515.00427B31  
00427B1C   lea eax,dword ptr ss:\[ebp-10\]  
00427B1F   mov edx,4  
00427B24   call echap515.00403558  
00427B29   retn  
算法虽然不是太清楚地分析出来，但因为是明码比较，我想利用程序自身做注册机应该没什么问题！  
我们记下 00427AF6 行处的正确注册码的内存地址 ss:\[ebp-c\],在我的机子上为 ss:12F9A4，但因为后面有出栈和 retn 操作，堆栈数据可能会被破坏掉，所以我们要把正确的注册码想办法传出来!因为原程序在 ebx 中存放标志位,故此我们可以利用 ebx 中存放正确注册码传出来(原程序是不会破坏 ebx 的值的)故从 00427B0A 处更改代码如下:  
00427B0A   mov ebx,ss:\[ebp-c\]  
00427B0D   nop  
00427B0E   nop  
00427B0F   xor eax, eax  
00427B11   pop edx  
00427B12   pop ecx  
00427B13   pop ecx  
00427B14   mov dword ptr fs:\[eax\],edx  
然后把从 00427BAC 处改为:  
00427BAC   push eax    ;我们不是把正确码放在 ebx 中吗?呵呵,函数的调用返回值一般都在 eax 中的  
00427BAD   nop  
00427BAE   nop  
00427BAF   nop  
00427BB0   nop  
00427BB1   mov eax,dword ptr ds:\[429744\]  
  
保存为另一个文件,运行一下试试，是不是跳出正确的注册码了!呵呵,原来做注册机也不用把算法弄得太透彻的啊,实在是懒人的福音啊!  
  
欢迎 e-mail 到 sharpair@163.com 交流！ 

 4dd466d0e1e0cc123afbe504da2462ca