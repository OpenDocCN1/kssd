

# 菜鸟也做注册机(四)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-36.htm`](https://www.pediy.com/kssd/pediy07/pediy7-36.htm)

【作者声明】：初学破解，仅作学习交流之用，失误之处敬请大侠赐教!  
  
【破解工具】：Ollydbg1.10、Hiew6.81、JiurlPedumpAver0.1(很好用的 PE 工具)、CrackTools(计算器工具)  
  
这篇以后再发破文不会再用这个标题了,以免误导大家!虽然不是个系列的教程,但我还是希望能对广大菜鸟有所帮助!  
  
载入程序；任意填入假码:用户名:sharpair 和假码:651484556;搜索字符参考，定位到以下代码：  
  
0042DB26   mov eax,dword ptr ds:\[42F744\]  
0042DB2B   mov eax,dword ptr ds:\[eax\]  
0042DB2D   mov dword ptr ds:\[42F750\],eax       ;  给内存 ds:\[42F750\]处存入一定值 0x12E6296D  
0042DB32   lea edx,dword ptr ss:\[ebp-4\]  
0042DB35   mov eax,dword ptr ds:\[ebx+1E0\]  
0042DB3B   call echap519.0041A228              ;  取输入的用户名"sharpair"  
0042DB40   cmp dword ptr ss:\[ebp-4\],0          ;  比较是否输入了用户名  
0042DB44   jnz short echap519.0042DB60  
0042DB46   push 0                              ; /Arg1 = 00000000  
0042DB48   mov cx,word ptr ds:\[42DD1C\]         ; |  
0042DB4F   mov dl,2                            ; |  
0042DB51   mov eax,echap519.0042DD28           ; |ASCII "Please typ in your name !!"  
0042DB56   call echap519.0042CE40              ; \\echap519.0042CE40  
0042DB5B   jmp echap519.0042DCF0  
0042DB60   lea edx,dword ptr ss:\[ebp-4\]  
0042DB63   mov eax,dword ptr ds:\[ebx+1E0\]  
0042DB69   call echap519.0041A228              ;  这个 Call 同上面的一样,取输入的用户名  
0042DB6E   mov eax,dword ptr ss:\[ebp-4\]  
0042DB71   call echap519.004037C0  
0042DB76   cmp eax,6                           ;  比较用户名长度是否大于 6  
0042DB79   jge short echap519.0042DB95  
0042DB7B   push 0                              ; /Arg1 = 00000000  
0042DB7D   mov cx,word ptr ds:\[42DD1C\]         ; |  
0042DB84   mov dl,2                            ; |  
0042DB86   mov eax,echap519.0042DD4C           ; |ASCII "Type at least 6 chars for your name! !"  
0042DB8B   call echap519.0042CE40              ; \\echap519.0042CE40  
0042DB90   jmp echap519.0042DCF0  
0042DB95   lea edx,dword ptr ss:\[ebp-4\]  
0042DB98   mov eax,dword ptr ds:\[ebx+1E4\]  
0042DB9E   call echap519.0041A228              ;  取输入的假码"65148455"  
0042DBA3   cmp dword ptr ss:\[ebp-4\],0  
0042DBA7   jnz short echap519.0042DBC3         ;  如果没输入注册码这里就不跳了  
0042DBA9   push 0                              ; /Arg1 = 00000000  
0042DBAB   mov cx,word ptr ds:\[42DD1C\]         ; |  
0042DBB2   mov dl,2                            ; |  
0042DBB4   mov eax,echap519.0042DD7C           ; |ASCII "Please enter your serial !"  
0042DBB9   call echap519.0042CE40              ; \\echap519.0042CE40  
0042DBBE   jmp echap519.0042DCF0  
0042DBC3   mov eax,edi  
0042DBC5   call echap519.00403544  
0042DBCA   mov dword ptr ds:\[esi\],2  
0042DBD0   /lea edx,dword ptr ss:\[ebp-4\]  
0042DBD3   |mov eax,dword ptr ds:\[ebx+1E0\]  
0042DBD9   |call echap519.0041A228  
0042DBDE   |mov eax,dword ptr ss:\[ebp-4\]       ;  取用户名到 eax 中  
0042DBE1   |mov edx,dword ptr ds:\[esi\]         ;  ds:\[esi\]上面已赋初值 2  
0042DBE3   |movzx eax,byte ptr ds:\[eax+edx-1\]  ;  从用户名的第二位开始按位取字符  
0042DBE8   |lea edx,dword ptr ss:\[ebp-8\]  
0042DBEB   |call echap519.00406578             ;  这个 Call 把字符转换成十进制值形式  
0042DBF0   |mov edx,dword ptr ss:\[ebp-8\]       ;  我这里是 edx=104(对应"h")  
0042DBF3   |mov eax,edi  
0042DBF5   |call echap519.004037C8             ;  这个 Call 粗跟没发现什么,猜测可能把上面的值保存到内存中  
0042DBFA   |inc dword ptr ds:\[esi\]             ;  调整指针  
0042DBFC   |cmp dword ptr ds:\[esi\],7           ;  比较是否取完用户名  
0042DBFF   \\jnz short echap519.0042DBD0  
0042DC01   lea eax,dword ptr ss:\[ebp-8\]  
0042DC04   push eax  
0042DC05   mov ecx,3  
0042DC0A   mov edx,1  
0042DC0F   mov eax,dword ptr ds:\[edi\]          ;  果然,这里是上面的 Dec 值连成的字串,我这里为"1049711411297"  
0042DC11   call echap519.004039C4  
0042DC16   mov eax,dword ptr ss:\[ebp-8\]        ;  析出第一位的,我这里 eax="104"  
0042DC19   call echap519.004065A8              ;  这个 Call 把 Dec 形式转化成 Hex 形式  
0042DC1E   mov dword ptr ds:\[42F758\],eax       ;  结果存入内存 ds:\[42F758\]  
0042DC23   mov eax,edi  
0042DC25   call echap519.00403544  
0042DC2A   mov eax,ebx  
0042DC2C   call echap519.0042D8E4  
0042DC31   mov eax,dword ptr ds:\[42F750\]  
0042DC36   mov dword ptr ds:\[42F750\],eax  
0042DC3B   mov eax,ebx  
0042DC3D   call echap519.0042D934  
0042DC42   mov eax,dword ptr ds:\[42F758\]  
0042DC47   mov dword ptr ds:\[42F758\],eax  
0042DC4C   mov eax,ebx  
0042DC4E   call echap519.0042D988  
0042DC53   mov eax,ebx  
0042DC55   call echap519.0042D9D8  
0042DC5A   mov eax,dword ptr ds:\[42F758\]  
0042DC5F   mov dword ptr ds:\[42F758\],eax  
0042DC64   mov eax,ebx  
0042DC66   call echap519.0042DA1C  
0042DC6B   mov eax,ebx  
0042DC6D   call echap519.0042DA28  
0042DC72   mov eax,dword ptr ds:\[42F758\]  
0042DC77   mov dword ptr ds:\[42F758\],eax  
0042DC7C   mov eax,ebx  
0042DC7E   call echap519.0042DA34  
0042DC83   mov eax,ebx  
0042DC85   call echap519.0042DA7C  
0042DC8A   mov eax,ebx  
0042DC8C   call echap519.0042DA9C  
0042DC91   mov eax,dword ptr ds:\[42F750\]  
0042DC96   add dword ptr ds:\[42F758\],eax  
0042DC9C   lea edx,dword ptr ss:\[ebp-4\]  
0042DC9F   mov eax,dword ptr ds:\[ebx+1E4\]  
0042DCA5   call echap519.0041A228              ;  取输入的假码"65148455"  
0042DCAA   mov eax,dword ptr ss:\[ebp-4\]  
0042DCAD   call echap519.004065A8              ;  Dec 转换成 Hex 形式  
0042DCB2   mov dword ptr ds:\[42F760\],eax  
0042DCB7   mov eax,dword ptr ds:\[42F758\]       ;  通过用户名计算得到的真码 Hex 形式  
0042DCBC   cmp eax,dword ptr ds:\[42F760\]       ;  假码的 Hex 形式在 ds:\[42F760\]中  
0042DCC2   jnz short echap519.0042DCDB  
0042DCC4   push 0                              ; /Arg1 = 00000000  
0042DCC6   mov cx,word ptr ds:\[42DD1C\]         ; |  
0042DCCD   mov dl,2                            ; |  
0042DCCF   mov eax,echap519.0042DDA0           ; |ASCII "Good Serial, Thanks For trying this Crackme bY nIabI !"  
0042DCD4   call echap519.0042CE40              ; \\echap519.0042CE40  
0042DCD9   jmp short echap519.0042DCF0  
0042DCDB   push 0                              ; /Arg1 = 00000000  
0042DCDD   mov cx,word ptr ds:\[42DD1C\]         ; |  
0042DCE4   mov dl,2                            ; |  
0042DCE6   mov eax,echap519.0042DDE0           ; |ASCII "Bad Name Or Serial Number !!!!!"  
0042DCEB   call echap519.0042CE40              ; \\echap519.0042CE40  
0042DCF0   xor eax,eax  
  
很显然,中间那段没注释的才是算法的最后完成部分,为什么我没注释,因为我是个懒人,不想再跟进去(别砸我:)),而且本人重点是向你说明有时没必要把算法弄得一清二楚我们一样可以做出注册机,让我们看看如何来让程序自显注册码吧！(当然最省的办法是用 Keymaker 做个内存注册机,但我想还是自己动手过瘾些)  
  
因为程序的关键判断处出现了正确码的 Hex 形式,所以我的懒是有依据的:) !程序中正确注码的形式为 Hex 值,而后面的输出函数要求的参数为字符串形式的,所以这里我们要把 Hex 值转换成相应的 ASCII 字符形式存放!以下是我改从 0042DC9F 行处的代码实现显示注册码的 Hex 进制形式:  
  
PS:本来想把 Hex 形式转化成 Dec 再转换成字串存入到内存中,但本人编程太烂,加上正确码是个大数,而且程序空间限制,只好作罢(我可不想对文件的 PE 结构操作,能省就省吧!我们的目标是一切从简:)),但还是希望那位大侠能补充完成此功能,让小菜鸟学习学习!  
\---------------------------------------------------  
0042DC9F    mov ecx,4  
0042DCA4    lea edx,dword ptr ds:\[42F758\]  
0042DCAA    lea ebx,dword ptr ds:\[42DDE7\]  
0042DCB0    xor eax,eax  
0042DCB2    mov al,byte ptr ds:\[edx\]  
0042DCB4    push eax  
0042DCB5    and al,0F  
0042DCB7    or al,30                           
0042DCB9    cmp al,39  
0042DCBB    jbe short echap519.0042DCBF  
0042DCBD    add al,7  
0042DCBF    mov byte ptr ds:\[ebx\],al  
0042DCC1    dec ebx  
0042DCC2    pop eax  
0042DCC3    shr al,4  
0042DCC6    or al,30  
0042DCC8    cmp al,39  
0042DCCA    jbe short echap519.0042DCCE  
0042DCCC    add al,7  
0042DCCE    mov byte ptr ds:\[ebx\],al  
0042DCD0    dec ebx  
0042DCD1    inc edx  
0042DCD2    loopd short echap519.0042DCB2  
0042DCD4    xor edx,edx  
0042DCD6    mov ecx,660000  
0042DCDB    push 0  
0042DCDD    mov cx,word ptr ds:\[42DD1C\]  
0042DCE4    mov dl,2  
0042DCE6    mov eax,echap519.0042DDE0          
0042DCEB    call echap519.0042CE40  
\----------------------------------------------------  
      
保存运行后试试看,啊,异常,呵呵,写入内存 42DDE7 处错误,:( !不用怕,既然是写入错误肯定是那段地址在代码段中,而作者把该段的属性设为不可写了，用 JiurlPedumpAver 打开文件 echap519.exe 看一下它的.code 段属性,为 60000020,这说明该块代码包含可执行代码,可读可执行,我们增加它的可写属性，其值应为 20000000h|4000000h|80000000h|00000020h=E0000020h,记下它的地址，再用 Hiew 打开文件,修改此处的值为 E0000020(注意在 Hiew 显示此值是倒序放的噢),保存退出再看看,是不是能正确运行了!还不满意!是的,正确的注册码后面还跟有 Or Serial Number!!!!!,这个不用我再说了吧,在 Hiew 中把它改为"is Serial Hex Value!!!"不就 OK 了吗?  
  
PS:其实刚开始时候我想在把正确码写到内存 ds:\[42EDE0\]处(此处不用改源文件的 code 段属性),可是在最后的显示窗口中竟然为空白，并未如愿显示正确码的 Hex 值,所以只好把正确码写到原来的字串"Bad Name Or Serial Number !!!!!"位置,希望哪位大侠能给小弟说明一下,真是很困惑啊!  
  
欢迎 e-mail 到 sharpair@163.com 交流！ 

 4dd466d0e1e0cc123afbe504da2462ca