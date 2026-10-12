
## 0044D3D9: 汇编代码分析

```
0044D3D9   .  74 06         je      short 0044D3E1  
0044D3DB   .  8B08          mov     ecx, [eax]  
0044D3DD   .  50            push    eax  
0044D3DE   .  FF51 08       call    [ecx+8]  
0044D3E1   >  33C0          xor     eax, eax  
0044D3E3   >  8B4D F4       mov     ecx, [ebp-C]  
0044D3E6   .  5F            pop     edi  
0044D3E7   .  5E            pop     esi  
0044D3E8   .  5B            pop     ebx  
0044D3E9   .  64:890D 00000>mov     fs:[0], ecx  
0044D3F0   .  C9            leave  
0044D3F1   .  C2 0C00       retn    0C  
```

函数有 3 个堆栈参数，没有寄存器参数。

我们要找的是第三个参数即`[ebp+10]`。

```
0044D249   .  8B45 10       mov     eax, [ebp+10]  
0044D24C   .  53            push    ebx  
0044D24D   .  56            push    esi  
0044D24E   .  57            push    edi  
0044D24F   .  C700 01000000 mov     dword ptr [eax], 1  
```

由上面的指令可知:`[ebp+10]`是一个整形数的地址,即我们要确定的`@var2`是个整形类型，改名为`@intVar2`。

## 分析 `_TempName`

返回`_TempName`继续分析。

```
lable1:  
  mov     eax, _arg3  
  mov     dword ptr [eax], 2  
```

可知:`_arg3`是个整形指针，改名为`_pIntArg3`。

```
  mov     eax, _arg2  
  push    edi  
  mov     edx, @var1  
  push    1  
  mov     ecx, [eax-8]  
```

可知:`_arg2`是一个指针，改名为`_pArg2`。

```
  invoke  CString::CString,_arg4  
```

`_arg4`是个字符串指针，改名为`_lpszArg4`。

```
  invoke  CString::CString,_arg1  
```

`_arg1`是个字符串指针，改名为`_lpszArg1`。

```
  invoke  CWnd::CWnd,addr @var3  
```

可知:`@var3`是个隐含的 `this` 指针，函数中原来的指令为：

```
00415C16  |.  8D4D AC       lea     ecx, [ebp-54]  
00415C19  |.  E8 569A0500   call    <jmp.&MFC42.#567_CWnd::CWnd>  
```

Visual C++ 编译器使用 `ecx` 用作 `this` 指针。

所以 `@var3` 是个 `CWnd` 的实例，改名为 `@wnd`。

```
invoke  CWnd::MessageBox,addr @wnd,_arg4,_arg1,MB_YESNO or MB_DEFBUTTON2 or MB_ICONHAND (114h)  ;这个就是登录失败的时候的标题和信息  
```

由上面可知:
- `_arg4` 为：内容 `"输入密码与上次成功登陆的密码不一致，(回车符）是否到服务器验证?"`
- `_arg1` 为：标题 `"错误"`

```
invoke  CString::~CString,_arg2  
```

`_arg2` 是个字符串指针，改名为`_lpszArg2`。

### 变量重命名总结

看看现在都确定了那些：

- `_regArg1` 是个函数指针的指针的指针，改名为:`_lpRegArg1` 吧，他终归是个指针
- `_arg1` 是个字符串指针，改名为：`_lpszArg1`
- `_arg2` 是个字符串指针，改名为`_lpszArg2`
- `_arg3` 是个整形指针，改名为`_pIntArg3`
- `_arg4` 是个字符串指针，改名为 `_lpszArg4`
- `@var1` 是个整形数，改名为 `@intVar1`
- `@var2` 是个整形数，改名为 `@intVar2`
- `@var3` 是个 `CWnd` 对象，改名为 `@wndVar3`

函数现在可以写成：

```asm
_TempName  Proc  _lpRegArg1,_lpszArg1,_lpszArg2,_pIntArg3,_arg?,_lpszArg4  (因为不知道倒数第二个为什么没有在函数中引用过，所以先用？表示)  

  local  @intVar1,@intVar2  

  local  @wndVar3:CWnd  

  push    ebx  

  push    esi  

  xor     ebx, ebx  

  mov     [ebp-4], ebx  

  mov     @intVar1, ebx  

  mov  ecx,_lpRegArg1  

  mov     eax, [ecx+84]  

  mov     ecx, [eax]  

  mov     byte ptr [ebp-4], 1  

  invoke  [ecx+1c],eax,004e7460,addr @intVar1  (这里不管 addr 会覆盖掉 eax 的值)  

  test    eax, eax  

  jnz     lable1  

  mov     eax, @intVar1  

  mov     ecx, [eax]  

  invoke  [ecx+14],eax,004e8940,addr @intVar2(这里不管 addr 会覆盖掉 eax 的值)  

  test    eax, eax  

  je      lable2  

lable1:  

  mov     eax, _lpIntArg3  

  mov     dword ptr [eax], 2  

  jmp     lable3  

lable2:  

  mov     eax, _arg2  

  push    edi  

  mov     edx, @intVar1  

  push    1  

  mov     ecx, [eax-8]  

  pop     esi  

  mov     edi, [edx]  

  invoke  [edi+1c],edx,_arg1,ebx,ecx,eax,esi  

  test    eax, eax  

  pop     edi  

  je      lable4  

  cmp     _lpszArg4, esi  

  je      lable5  

  mov     eax, _lpIntArg3  

  mov     [eax], ebx  

  jmp     lable3  

lable5:  

  invoke  CString::CString,_lpszArg4  

  mov     byte ptr [ebp-4], 2  

  invoke  CString::CString,_lpszArg1  

  mov     byte ptr [ebp-4], 3  

  invoke  BasicCtrlDll.BasicLoadStr,addr _lpszArg4,281h  

  invoke  BasicCtrlDll.BasicLoadStr,addr _lpszArg1,28dh  

  invoke  CWnd::CWnd,addr @wnd  

  mov     byte ptr [ebp-4], 4  

  invoke  CWnd::MessageBox,addr @wnd,_lpszArg4,_lpszArg1,MB_YESNO or MB_DEFBUTTON2 or MB_ICONHAND (114h)  ;这个就是登录失败的时候的标题和信息  

  cmp     eax, 6  

  mov     eax, _lpIntArg3  

  jnz     label6  

  mov     dword ptr [eax], 2  

  jmp     lable7  

lable6:  

  mov     [eax], ebx  

lable7:  

  mov     byte ptr [ebp-4], 3  

  invoke  CWnd::~CWnd,addr @wndVar3  

  mov     byte ptr [ebp-4], 2  

  invoke  CString::~CString,addr _lpszArg1  

  mov     byte ptr [ebp-4], 1  

  invoke  CString::~CString,_lpszArg4  

  jmp     lable3  

lable4:  

  mov     eax, _lpIntArg3  

  mov     [eax], esi  

lable3:  

  mov     eax, _lpIntArg3  

  mov     [ebp-4], bl  

  cmp     eax, ebx  

  je      lable8  

  mov     ecx, [eax]  

  invoke  [ecx+8],eax  

lable8:  

  or      dword ptr [ebp-4], FFFFFFFF  

  invoke  CString::~CString,_lpszArg2  

  pop     esi  

  pop     ebx  

  ret  

_TempName  endp  
```

今天到这里了，进度有点慢.....

2006-06-18 10:00

## 分析 `_TempName` 返回值

2006-06-21 7:10

找出`_TempName` 函数的返回值，这里的返回值，只考虑高级语言中的 `return` 语句中的值，不考虑其他引用返回值、等等情况。

`_TempName` 函数在结尾时，没有显示对 `eax`，`edx` 进行赋值，可以判断`_TempName` 函数的返回值为空，但也有可能`_TempName` 返回她调用函数的值。

比如：`return func(arg1,..)`，为了进一步确定，看看调用`_TempName` 函数的函数有没有在 `call 00415b55` 指令后紧跟着使用 `eax`、`edx` 的值。

```
00414FE8  |.  E8 680B0000   call    00415B55          
  
00414FED  |.  FFD6          call    esi                ;  Kernel32.GetTickCount  
```

结果表明：没有使用 `eax`，`edx` ，所以`_TempName` 的返回值应为 `void`。

C++表示的函数原型应为：

```cpp
void  TempName(void* regArg1,char* lpszArg1,char* lpszArg2,int* lpIntArg3,?,char* lpszArg4);  
```

## 还原高级语言表达式

下面将汇编语言指令还原成高级语言的表达式。

从函数调用语句或比较指令向上倒序还原，例如：

```asm
  push    ebx  
  push    esi  
  xor     ebx, ebx  
  mov     [ebp-4], ebx  
  mov     @intVar1, ebx  
  mov  ecx,_lpRegArg1  
  mov     eax, [ecx+84]  
  mov     ecx, [eax]  
  mov     byte ptr [ebp-4], 1  
  invoke  [ecx+1c],eax,004e7460,addr @intVar1  (这里不管 addr 会覆盖掉 eax 的值)  
  test    eax, eax  
```

就从 `invoke [ecx+1c],eax,004e7460,addr @intVar1` 开始，用局部变量，和参数替换掉寄存器，这样我们就去掉了临时的寄存器。

变量得到了高级语言表达式。(在高级语言中，我把局部变量名前面的`@`去掉，参数名前面的`_`去掉)

- `ecx=[eax]=[[ecx+84]]=[[lpRegArg1+84h]]=*(*(lpRegArg1+84h))`
- `eax=[ecx+84]=[lpRegArg1+84]=*(lpRegArg1+0x84)`

`invoke [ecx+1c],eax,004e7460,addr @intVar1` 用 C++ 表示为：

```cpp
*(*(*(lpRegArg1+0x84))+0x1c)(* (lpRegArg1+0x84),0x004e7460,&intVar1);  
```

删掉：

```asm
  mov  ecx,_lpRegArg1  
  mov     eax, [ecx+84]  
  mov     ecx, [eax]  
```

这几个临时的寄存器变量。

上面还剩的指令有：

```asm
  push    ebx  
  push    esi  
  ；这是保存寄存器值，与高级语言没什么关系删掉  
  xor     ebx, ebx  
  mov     [ebp-4], ebx  
  mov     @intVar1, ebx  
  mov     byte ptr [ebp-4], 1  
```

可以写成：

```cpp
  [ebp-4]=0;  
  intVar1=0;  
        byte ptr[ebp-4]=1;  
```

以上的指令还原为：

```cpp
  [ebp-4]=0;  
  intVar1=0;  
        byte ptr[ebp-4]=1;  
  *(*(*(lpRegArg1+0x84))+0x1c)(* (lpRegArg1+0x84),0x004e7460,&intVar1);  
```

跳转指令使用条件表达式和 `goto` 替换。

依次类推函数可写为

```cpp
void TempName(void* lpRegArg1,char* lpszArg1,char* lpszArg2,int* lpIntArg3,?,lpszArg4)  

{  

  int intVar1,intVar2;  

  CWnd wndVar3;  

  mov  [ebp-4],0  

  intVar1=0;  

  *(*(*(lpRegArg1+0x84))+0x1c)(* (lpRegArg1+0x84),0x004e7460,&intVar1);  

  if(eax) goto lable1  

  *(*(intVar1)+0x14)(intVar1,0x004e8940,&intVar2);  

  // 看来 var1 不是个整形变量，而是个指针 改名为 lpVar1,函数要改写了  

  intVar1=0;这条语句可能是空指针初始化   

}  
```

```cpp
void TempName(void* lpRegArg1,char* lpszArg1,char* lpszArg2,int* lpIntArg3,?,lpszArg4)  

{  

  void* lpVar1=NULL;  

  int intVar2;  

  CWnd wndVar3;  

  mov  [ebp-4],0  

  *(*(*(lpRegArg1+0x84))+0x1c)(* (lpRegArg1+0x84),0x004e7460,&lpVar1);  

  if(eax) goto lable1  

  *(*(lpVar1)+0x14)(intVar1,0x004e8940,&intVar2);  

  if(!eax) goto lable2;  

lable1:  

  *lpIntArg3=2;  

  goto lable3;  

lable2:  

  **lpVar1+0x1c)(lpVar1,0,* (lpszArg2-8),lpszArg2,1);  

  if(!eax) goto lable4;  

  if(lpszArg4==1) goto lable5;  

  *lpIntArg3=0;  

  goto lable3;  

lable5:  

  CString(lpszArg4);  

  mov  byte ptr[ebp-4],2;  

  CString(lpszArg1);  

  mov  byte ptr[ebp-4],3;  

  BasicCrtDll.BasicLoadStr(lpszArg4,0x281);  

  BasicCrtDll.BasicLoadStr(lpszArg1,0x28d);  

  wndVar3.MessageBox(lpszArg4,lpszArg1,MB_YESNO or MB_DEFBUTTON2 or MB_ICONHAND);  
```
