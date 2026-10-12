
# 让 EXE 导出函数

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-682.htm`](https://www.pediy.com/kssd/pediy09/pediy09-682.htm)

初步搞定。

问题来源：偶然发现 `OllyDBG.exe` 导出了一堆函数，这些函数都是供其插件调用的。对这种体系结构很感兴趣，想弄清楚它的实现原理。后来又看到梁肇新的书《编程高手箴言》第 278 页提到的调用门，觉得都应该差不多。

### 三种不同的解决办法

（原理可能是一样的，:)）

1. 在导出函数声明之前加上 `__declspec(dllexport)`。例：

   ```cpp
   __declspec(dllexport) int Add(int a, int b);
   __declspec(dllexport) int Sub(int a, int b);
   __declspec(dllexport) int Mul(int a, int b);
   __declspec(dllexport) int Div(int a, int b);
   ```

2. 在链接器参数中设置。例：

   ```cpp
   #pragma comment(linker, "/EXPORT:_Add,@1,NONAME")
   #pragma comment(linker, "/EXPORT:_Sub,@2,NONAME")
   #pragma comment(linker, "/EXPORT:_Mul,@3,NONAME")
   #pragma comment(linker, "/EXPORT:_Div,@4,NONAME")
   ```

3. 添加一个 def 文件，例：

   ```text
   EXPORTS
     Add       @1  NONAME
     Sub       @2  NONAME
     Mul       @3  NONAME
     Div       @4  NONAME
   ```

   另需要在链接器命令行参数中指定 def 文件名：`/DEF:Callee.def`

   注意：在 def 文件中不要有 `LIBRARY [library][BASE=address]` 这样的语句。

相比较而言，后两种方法可以设置更多的参数。

### 函数举例

```cpp
extern "C"
{
  int Add(int a, int b)
  {
    return (a + b);
  }

  int Sub(int a, int b)
  {
    return (a - b);
  }

  int Mul(int a, int b)
  {
    return (a * b);
  }

  int Div(int a, int b)
  {
    if (b == 0)
      return 0;
    else
      return (a / b);
  }
}
```

编译时会自动生成相应的导出库（lib）文件，供调用者使用。

### 调用方法

调用方法和普通的动态链接库调用一样。调用者必须能够找到被调用者的位置，否则报错，被调用者是否运行不影响。

调用代码举例：

```cpp
extern "C"
{
  int Add(int a, int b);
  int Sub(int a, int b);
  int Mul(int a, int b);
  int Div(int a, int b);
}

#pragma comment (lib, "Callee.lib")

void CCallerDlg::OnBnClickedCalculate()
{
  // TODO: Add your control notification handler code here
  UpdateData(TRUE);

  switch (((CComboBox *)GetDlgItem(IDC_COMBO_OPERATOR))->GetCurSel())
  {
  case ADD:
    {
      m_iResult = Add(m_iNum1, m_iNum2);
      break;
    }
  case SUB:
    {
      m_iResult = Sub(m_iNum1, m_iNum2);
      break;
    }
  ...
  ...
}
```

我在 `OD` 中跟了一下，发现这跟调用动态链接库也差不多。不过那几个函数被映射到下面的地址处：

```assembly
003810F0 >   8B4424 08                   mov     eax, dword ptr [esp+8]
003810F4     8B4C24 04                   mov     ecx, dword ptr [esp+4]
003810F8     03C1                        add     eax, ecx
003810FA     C3                          retn
003810FB     CC                          int3
003810FC     CC                          int3
003810FD     CC                          int3
003810FE     CC                          int3
003810FF     CC                          int3
00381100 >   8B4424 04                   mov     eax, dword ptr [esp+4]
00381104     2B4424 08                   sub     eax, dword ptr [esp+8]
00381108     C3                          retn
00381109     CC                          int3
0038110A     CC                          int3
0038110B     CC                          int3
0038110C     CC                          int3
0038110D     CC                          int3
0038110E     CC                          int3
0038110F     CC                          int3
00381110 >   8B4424 04                   mov     eax, dword ptr [esp+4]
00381114     0FAF4424 08                 imul    eax, dword ptr [esp+8]
00381119     C3                          retn
0038111A     CC                          int3
0038111B     CC                          int3
0038111C     CC                          int3
0038111D     CC                          int3
0038111E     CC                          int3
0038111F     CC                          int3
00381120 >   8B4C24 08                   mov     ecx, dword ptr [esp+8]
00381124     85C9                        test    ecx, ecx
00381126     75 03                       jnz     short 0038112B
00381128     33C0                        xor     eax, eax
0038112A     C3                          retn
```

跟常规的动态链接库被映射到高地址处略有不同。还不知道是什么原因。

### 结论

EXE 完全可以和 DLL 一样导出函数，一样被调用。

进一步的工作：我发现这个例子跟 `OllyDbg.exe` 还是有些不同，跟“调用门”的说法也有不同。这里实际上还是跟 DLL 差不多的原理。下一步争取实现一个跟 `OllyDbg.exe` 差不多的例子。

### 致谢

感谢海风月影、北极星 2003、默数悲伤所提供的思路。

源代码和例子见附件。
