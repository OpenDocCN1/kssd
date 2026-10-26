
```
00C2CF91     FF76 08           push dword ptr ds:[esi+8]

00C2CF94     6A 00             push 0

00C2CF96     E8 2639FFFF       call 00C208C1

00C2CF9B     50                push eax

00C2CF9C     A1 E018C400       mov eax,dword ptr ds:[C418E0]

00C2CFA1     8B48 78           mov ecx,dword ptr ds:[eax+78]

00C2CFA4     3348 5C           xor ecx,dword ptr ds:[eax+5C]

00C2CFA7     3348 14           xor ecx,dword ptr ds:[eax+14]

00C2CFAA     2BF9              sub edi,ecx

00C2CFAC     FFD7              call edi   //直接到这下“硬件执行”断点，F9，取消断点，F7进入

004D12D4     55                push ebp   //到这里了，OEP！LordPE-纠正映像-Dump！

004D12D5     8BEC              mov ebp,esp

004D12D7     83C4 F0           add esp,-10

004D12DA     B8 0C0E4D00       mov eax,XDeskWea.004D0E0C

004D12DF     E8 004EF3FF       call XDeskWea.004060E4

004D12E4     A1 B06E4D00       mov eax,dword ptr ds:[4D6EB0]

004D12E9     8B00              mov eax,dword ptr ds:[eax]

004D12EB     E8 08C0F9FF       call XDeskWea.0046D2F8

004D12F0     E8 D7E5FFFF       call XDeskWea.004CF8CC

004D12F5     A1 B06E4D00       mov eax,dword ptr ds:[4D6EB0]

004D12FA     8B00              mov eax,dword ptr ds:[eax]
```
OD，不要关！打开import--选择进程--OEP输入`D12D4`--自动搜索IAT--获取输入表--显示无效函数--CUT！

这样就可以了！因为这个软件是半透明界面，大家在录象中可能看不到，那就这样看吧~~

******************************************************************************

******************************************************************************

2、`Armadillo 3.00a - 3.61`&单&标准方式=====》目标程序“查重软件 2.0版（MARC数据相关）”

******************************************************************************

方法：2次断点法，即是：

`bp GetModuleHandleA`/`he GetModuleHandleA`

`bp GetCurrentThreadId`

******************************************************************************

OD载入。。。。。

```
00469000 g>  60                pushad  //入口

00469001     E8 00000000       call gz_LibIn.00469006

00469006     5D                pop ebp
```
`he GetModuleHandleA`下断，`F9`，注意堆栈！

```
7C80B529 k>  8BFF              mov edi,edi   //入口

7C80B52B     55                push ebp

7C80B52C     8BEC              mov ebp,esp

7C80B52E     837D 08 00        cmp dword ptr ss:[ebp+8],0

7C80B532     74 18             je short kernel32.7C80B54C
```
```
0046B4A3     F0:               prefix lock:           //第一次典型异常，Shift+F9过！

0046B4A4     F0:C7             ???                    ；未知命令

0046B4A6     C8 6033C9         enter 3360,0C9

0046B4AA     75 02             jnz short gz_LibIn.0046B4AE

0046B4AC     EB 15             jmp short gz_LibIn.0046B4C3

0046B4AE     EB 33             jmp short gz_LibIn.0046B4E3

0046B4B0     C9                leave

0046B5BC     F0:               prefix lock:           //第二次典型异常，Shift+F9过！

0046B5BD     F0:C7             ???                    ; 未知命令

0046B5BF     C8 64678F         enter 6764,8F

0046B5C3     06                push es

0046B5C4     0000              add byte ptr ds:[eax],al

0046B5C6     83C4 04           add esp,4

0046B5C9     8B85 5E3E0000     mov eax,dword ptr ss:[ebp+3E5E]

0046B5CF     60                pushad
```
```
7C80B529 k>  8BFF              mov edi,edi       //硬件中断在这里

7C80B52B     55                push ebp

7C80B52C     8BEC              mov ebp,esp

7C80B52E     837D 08 00        cmp dword ptr ss:[ebp+8],0

7C80B532     74 18             je short kernel32.7C80B54C
```
******************************************************************************

每一次`F9`运行时候，所出现的堆栈情况！

******************************************************************************

```
0012C258    00AF5331  /CALL 到 GetModuleHandleA 来自 00AF532B

0012C25C    0012C394  \pModule = "kernel32.dll"

0012C260    00000002
```
这个只`F9`一次，`Shift+F9`两次就可以返回了。。。

此时取消断点！就可以返回了

```
00AF5331     8B0D 60D8B100     mov ecx,dword ptr ds:[B1D860]     //返回到这里

00AF5337     89040E            mov dword ptr ds:[esi+ecx],eax

00AF533A     A1 60D8B100       mov eax,dword ptr ds:[B1D860]

00AF533F     393C06            cmp dword ptr ds:[esi+eax],edi

00AF5342     75 16             jnz short 00AF535A

00AF5344     8D85 B4FEFFFF     lea eax,dword ptr ss:[ebp-14C]

00AF534A     50                push eax

00AF534B     FF15 B850B100     call dword ptr ds:[B150B8]   ; kernel32.LoadLibraryA

00AF5351     8B0D 60D8B100     mov ecx,dword ptr ds:[B1D860]

00AF5357     89040E            mov dword ptr ds:[esi+ecx],eax

00AF535A     A1 60D8B100       mov eax,dword ptr ds:[B1D860]

00AF535F     393C06            cmp dword ptr ds:[esi+eax],edi

00AF5362     0F84 AD000000     je 00AF5415   //Magic Jump！改jmp！跳转很大的。。只是没跳而已

00AF5368     33C9              xor ecx,ecx

00AF536A     8B03              mov eax,dword ptr ds:[ebx]

00AF536C     3938              cmp dword ptr ds:[eax],edi

00AF536E     74 06             je short 00AF5376
```
此时第一断点的任务就完成了！

******************************************************************************

下断`bp GetCurrentThreadId`，`F9`运行，注意堆栈！

```
7C809737 k>  64:A1 18000000    mov eax,dword ptr fs:[18]  //停在这

7C80973D     8B40 24           mov eax,dword ptr ds:[eax+24]

7C809740     C3                retn
```
******************************************************************************

每一次`F9`运行时候，所出现的堆栈情况！

******************************************************************************

```
0012BB18    73391E36  /CALL 到 GetCurrentThreadId 来自 73391E30

0012BB1C    00000001

0012BB20    73391C1A  返回到 73391C1A 来自 73391DE2

0012BAF8    7339353F  /CALL 到 GetCurrentThreadId 来自 73393539

0012BAFC    00000000

0012BB00    734A0470

0012BB04    734A04D8

0012BD54    73DC9AD7  /CALL 到 GetCurrentThreadId 来自 73DC9AD1

0012BD58    00000000

0012BD5C    73E08100

0012BD60    73D30000

0012F580    00B0CF2D  /CALL 到 GetCurrentThreadId 来自 00B0CF27   //这一次时间缓冲比较大！

0012F584    0012FF04

0012F588    00000000
```
取消断点，返回！

```
00B0CF2D     A3 F018B200       mov dword ptr ds:[B218F0],eax   //回到这里

00B0CF32     E8 2487FEFF       call 00AF565B

00B0CF37     6A 00             push 0

00B0CF39     E8 4BD9FEFF       call 00AFA889

00B0CF3E     59                pop ecx

00B0CF3F     E8 7D39FFFF       call 00B008C1

00B0CF44     8BF8              mov edi,eax

00B0CF46     A1 E018B200       mov eax,dword ptr ds:[B218E0]

00B0CF4B     8B48 74           mov ecx,dword ptr ds:[eax+74]

00B0CF4E     3348 5C           xor ecx,dword ptr ds:[eax+5C]

00B0CF51     3308              xor ecx,dword ptr ds:[eax]

00B0CF53     03F9              add edi,ecx

00B0CF55     8B0E              mov ecx,dword ptr ds:[esi]

00B0CF57     85C9              test ecx,ecx

00B0CF59     75 2E             jnz short 00B0CF89

00B0CF5B     8B78 5C           mov edi,dword ptr ds:[eax+5C]

00B0CF5E     E8 5E39FFFF       call 00B008C1

00B0CF63     8B0D E018B200     mov ecx,dword ptr ds:[B218E0]                 ;

gz_LibIn.00479258

00B0CF69     FF76 14           push dword ptr ds:[esi+14]

00B0CF6C     8B51 74           mov edx,dword ptr ds:[ecx+74]

00B0CF6F     FF76 10           push dword ptr ds:[esi+10]

00B0CF72     33D7              xor edx,edi

00B0CF74     3311              xor edx,dword ptr ds:[ecx]

00B0CF76     FF76 0C           push dword ptr ds:[esi+C]

00B0CF79     03C2              add eax,edx

00B0CF7B     8B51 78           mov edx,dword ptr ds:[ecx+78]

00B0CF7E     3351 14           xor edx,dword ptr ds:[ecx+14]

00B0CF81     33D7              xor edx,edi

00B0CF83     2BC2              sub eax,edx

00B0CF85     FFD0              call eax

00B0CF87     EB 25             jmp short 00B0CFAE

00B0CF89     83F9 01           cmp ecx,1

00B0CF8C     75 22             jnz short 00B0CFB0

00B0CF8E     FF76 04           push dword ptr ds:[esi+4]
```
