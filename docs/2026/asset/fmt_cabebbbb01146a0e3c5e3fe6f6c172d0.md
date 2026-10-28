
```
void test()

{

008E10F0  push        ebp  

008E10F1  mov         ebp,esp  

008E10F3  sub         esp,0CCh  

008E10F9  push        ebx  

008E10FA  push        esi  

008E10FB  push        edi  

008E10FC  lea         edi,[ebp-0CCh]  

008E1102  mov         ecx,33h  

008E1107  mov         eax,0CCCCCCCCh  

008E110C  rep stos    dword ptr es:[edi]  

008E110E  mov         ecx,offset _D3472036_test@cpp (097200Bh)  

008E1113  call        __CheckForDebuggerJustMyCode (08E14C0h)  

	Base base1(10);

008E1118  push        0Ah  

008E111A  lea         ecx,[base1]  

008E111D  call        Base::Base (08E1000h)      //调用构造函数

}

008E1122  lea         ecx,[base1]                //将base1的地址赋给ecx

008E1125  call        Base::~Base (08E1060h)     //调用析构函数

008E112A  push        edx                        //开始执行函数推出的代码

008E112B  mov         ecx,ebp  

008E112D  push        eax  

008E112E  lea         edx,ds:[8E1150h]  

008E1134  call        _RTC_CheckStackVars (08E1410h)  

008E1139  pop         eax  

008E113A  pop         edx  

008E113B  pop         edi  

008E113C  pop         esi  

008E113D  pop         ebx  

008E113E  add         esp,0CCh  

008E1144  cmp         ebp,esp  

008E1146  call        _RTC_CheckEsp (08E1480h)  

008E114B  mov         esp,ebp  

008E114D  pop         ebp  

008E114E  ret
```
由上可以看出，对于局部变量来说，在函数退出之前会首先调用相应的析构函数释放内存，随后才开始函数退出代码的执行。再看析构函数内容
```
	~Base()

	{

008E1060  push        ebp  

008E1061  mov         ebp,esp  

008E1063  push        0FFFFFFFFh  

008E1065  push        952990h  

008E106A  mov         eax,dword ptr fs:[00000000h]  

008E1070  push        eax  

008E1071  mov         dword ptr fs:[0],esp  

008E1078  sub         esp,0CCh  

008E107E  push        ebx  

008E107F  push        esi  

008E1080  push        edi  

008E1081  push        ecx                              //保存类变量地址

008E1082  lea         edi,[ebp-0D8h]  

008E1088  mov         ecx,33h  

008E108D  mov         eax,0CCCCCCCCh  

008E1092  rep stos    dword ptr es:[edi]  

008E1094  pop         ecx                              //恢复类变量地址

008E1095  mov         dword ptr [this],ecx             //将类变量地址赋给局部变量

008E1098  mov         ecx,offset _D3472036_test@cpp (097200Bh)  

008E109D  call        __CheckForDebuggerJustMyCode (08E14C0h)  

		printf("~Base()\n");

008E10A2  push        offset string "~Base()\n" (09531B0h)  

008E10A7  call        printf (08E1280h)               //打印输出

008E10AC  add         esp,4                            

		if (pTest)	free(pTest);

008E10AF  mov         eax,dword ptr [this]            //取出类变量地址

008E10B2  cmp         dword ptr [eax],0               //判断地址中的变量值是否为0

008E10B5  je          Base::~Base+65h (08E10C5h)      //为0则跳转

008E10B7  mov         eax,dword ptr [this]            //取出类变量地址

008E10BA  mov         ecx,dword ptr [eax]             //将类变量地址中的内容取出赋给ecx

008E10BC  push        ecx                             //此时ecx便是pTest

008E10BD  call        free (091A2A0h)                 //调用free函数

008E10C2  add         esp,4  

	}

008E10C5  mov         ecx,dword ptr [ebp-0Ch]  

008E10C8  mov         dword ptr fs:[0],ecx  

008E10CF  pop         edi  

008E10D0  pop         esi  

008E10D1  pop         ebx  

008E10D2  add         esp,0D8h  

008E10D8  cmp         ebp,esp  

008E10DA  call        _RTC_CheckEsp (08E1480h)  

008E10DF  mov         esp,ebp  

008E10E1  pop         ebp  

008E10E2  ret
```
可以看到析构函数的执行和我们的普通成员函数的执行是一样的，它并没有把函数地址作为返回值返回给用户。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2021-10-20 11:24

被1900编辑

，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
