

# PECompact2.70 脱壳

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-569.htm`](https://www.pediy.com/kssd/pediy08/pediy8-569.htm)

【  标题  】 PECompact2.70 脱壳  
【  作者  】 linxer  
【破解平台】 Win2k3 English Version  
【脱壳工具】 OllyDbg v1.10 + OllyDump 插件  
【待脱软件】 Win2k3 自带 NOTEPAD(用 PECompact2.70 加壳)  
【  声明  】 初学脱壳，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
  
OD 异常设置忽略所有异常  
  
01001000 N>  B8 64BA0101        mov eax,NOTEPAD.0101BA64    //OD 载入后停于此处  
01001005     50                 push eax                    //向堆栈中压入一个异常处理回调函数地址(底下发生内存访问异常，将会调用这个函数)  
01001006     64:FF35 00000000   push dword ptr fs:\[0\]  
0100100D     64:8925 00000000   mov dword ptr fs:\[0\],esp    //向 SEH 链中插入一个异常处理项  
01001014     33C0               xor eax,eax  
01001016     8908               mov dword ptr ds:\[eax\],ecx  //内存访问异常  
  
由上面的代码知，bp 0x0101BA64，F9 运行到该断点  
  
底下就是异常处理回调函数的代码了  
0101BA64     B8 2EA801F1        mov eax,F101A82E  
0101BA69     8D88 59120010      lea ecx,dword ptr ds:\[eax+10001259\]  //此指令执行后 ecx 为 0x0101ba87,注意这是 retn 指令后的第一条指令地址  
0101BA6F     8941 01            mov dword ptr ds:\[ecx+1\],eax         //修改 0x0101ba87 处 mov 指令的操作数  
0101BA72     8B5424 04          mov edx,dword ptr ss:\[esp+4\]         //取 EXCEPTION\_RECORD 结构地址  
0101BA76     8B52 0C            mov edx,dword ptr ds:\[edx+C\]         //取发生异常指令的地址(就是上面的 0x01001016)  
0101BA79     C602 E9            mov byte ptr ds:\[edx\],0E9            //修改引发异常处指令的 opcode 为 jmp(0xe9,它的操作数是四字节的)  
0101BA7C     83C2 05            add edx,5                            //edx 是引发异常处的下条指令地址  
0101BA7F     2BCA               sub ecx,edx                          //计算相对偏移  
0101BA81     894A FC            mov dword ptr ds:\[edx-4\],ecx         //修改引发异常处指令的操作数(它将使程序跳到 0x0101ba87 处)  
0101BA84     33C0               xor eax,eax  
0101BA86     C3                 retn  
0101BA87     B8 78563412        mov eax,12345678  
  
通过对异常处理回调函数的分析，知道程序流程在异常处理结束后会来到 0x0101ba87 处，bp 0x0101ba87,F9 到该断点处  
  
0101BA87     B8 2EA801F1        mov eax,F101A82E  
0101BA8C     64:8F05 00000000   pop dword ptr fs:\[0\]  
0101BA93     83C4 04            add esp,4             //删除刚才插入的异常处理项  
0101BA96     55                 push ebp  
0101BA97     53                 push ebx  
0101BA98     51                 push ecx  
0101BA99     57                 push edi  
0101BA9A     56                 push esi  
0101BA9B     52                 push edx              //这片是真正的解压代码了，往下找对应的 pop 系列指令  
0101BA9C     8D98 12120010      lea ebx,dword ptr ds:\[eax+10001212\]  
0101BAA2     8B53 18            mov edx,dword ptr ds:\[ebx+18\]  
0101BAA5     52                 push edx  
0101BAA6     8BE8               mov ebp,eax  
0101BAA8     6A 40              push 40  
0101BAAA     68 00100000        push 1000  
0101BAAF     FF73 04            push dword ptr ds:\[ebx+4\]  
0101BAB2     6A 00              push 0  
0101BAB4     8B4B 10            mov ecx,dword ptr ds:\[ebx+10\]  
0101BAB7     03CA               add ecx,edx  
0101BAB9     8B01               mov eax,dword ptr ds:\[ecx\]  
0101BABB     FFD0               call eax  
0101BABD     5A                 pop edx  
0101BABE     8BF8               mov edi,eax  
0101BAC0     50                 push eax  
0101BAC1     52                 push edx  
0101BAC2     8B33               mov esi,dword ptr ds:\[ebx\]  
0101BAC4     8B43 20            mov eax,dword ptr ds:\[ebx+20\]  
0101BAC7     03C2               add eax,edx  
0101BAC9     8B08               mov ecx,dword ptr ds:\[eax\]  
0101BACB     894B 20            mov dword ptr ds:\[ebx+20\],ecx  
0101BACE     8B43 1C            mov eax,dword ptr ds:\[ebx+1C\]  
0101BAD1     03C2               add eax,edx  
0101BAD3     8B08               mov ecx,dword ptr ds:\[eax\]  
0101BAD5     894B 1C            mov dword ptr ds:\[ebx+1C\],ecx  
0101BAD8     03F2               add esi,edx  
0101BADA     8B4B 0C            mov ecx,dword ptr ds:\[ebx+C\]  
0101BADD     03CA               add ecx,edx  
0101BADF     8D43 1C            lea eax,dword ptr ds:\[ebx+1C\]  
0101BAE2     50                 push eax  
0101BAE3     57                 push edi  
0101BAE4     56                 push esi  
0101BAE5     FFD1               call ecx  
0101BAE7     5A                 pop edx  
0101BAE8     58                 pop eax  
0101BAE9     0343 08            add eax,dword ptr ds:\[ebx+8\]  
0101BAEC     8BF8               mov edi,eax  
0101BAEE     52                 push edx  
0101BAEF     8BF0               mov esi,eax  
0101BAF1     8B46 FC            mov eax,dword ptr ds:\[esi-4\]  
0101BAF4     83C0 04            add eax,4  
0101BAF7     2BF0               sub esi,eax  
0101BAF9     8956 08            mov dword ptr ds:\[esi+8\],edx  
0101BAFC     8B4B 0C            mov ecx,dword ptr ds:\[ebx+C\]  
0101BAFF     894E 14            mov dword ptr ds:\[esi+14\],ecx  
0101BB02     FFD7               call edi  
0101BB04     8985 FA120010      mov dword ptr ss:\[ebp+100012FA\],eax  
0101BB0A     8BF0               mov esi,eax  
0101BB0C     8B4B 14            mov ecx,dword ptr ds:\[ebx+14\]  
0101BB0F     5A                 pop edx  
0101BB10     EB 0C              jmp short NOTEPAD.0101BB1E  
0101BB12     03CA               add ecx,edx  
0101BB14     68 00800000        push 8000  
0101BB19     6A 00              push 0  
0101BB1B     57                 push edi  
0101BB1C     FF11               call dword ptr ds:\[ecx\]  
0101BB1E     8BC6               mov eax,esi  
0101BB20     5A                 pop edx  
0101BB21     5E                 pop esi  
0101BB22     5F                 pop edi  
0101BB23     59                 pop ecx  
0101BB24     5B                 pop ebx  
0101BB25     5D                 pop ebp    //到这里真实的 NOTEPAD 代码被释放出来  
0101BB26     FFE0               jmp eax    //F4 运行到该处  
  
到这里只要在 F7 一下就到了真正的 NOTEPAD 程序了，dump 下来，就 OK 了。  
  
  
【  总结 】PECompact2.70 脱壳步骤：  
  1.OD 加载程序，记下 OD 停处指令的第二个操作数 xxxxxxxx,Ctrl + G 到 xxxxxxxx 处  
  2.从 xxxxxxxx 处 Ctrl + F 查找 jmp eax 指令  
  3.F4 运行到 jmp eax 处  
  4.F7 步过一条指令  
  5.Dump 内存镜像  
  6.完事收工 

 4dd466d0e1e0cc123afbe504da2462ca