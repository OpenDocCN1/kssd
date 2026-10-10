
```
75BD2E70     8B08                  mov     ecx, dword ptr [eax]

75BD2E72     FF75 10              push    dword ptr [ebp+10]

75BD2E75     50                    push    eax

75BD2E76     FF51 1C              call    dword ptr [ecx+1C] ; mshtml.7CCFA138
```

`Push eax` 会把 `0X012014B0` 压入（THIS 指针）

`ECX+1C` 的代码如下，主要是从 `0X012014B0` 偏移 `0x10` 取 `apfnVtblObject1`，然后从 `apfnVtblObject1`（`7CC8EFA4`） 偏移 `0x1c` 处取指针，最终 `jmp ecx`

```
7CCFA138         8B4424 04           mov     eax, dword ptr [esp+4]
7CCFA13C         50                   push    eax
7CCFA13D         F740 1C 80000000    test    dword ptr [eax+1C], 80
7CCFA144         0F85 ABA60800       jnz     7CD847F5
7CCFA14A         83C0 0C             add     eax, 0C
7CCFA14D         8B08                mov     ecx, dword ptr [eax]
7CCFA14F         894C24 08           mov     dword ptr [esp+8], ecx
7CCFA153         8B48 04             mov     ecx, dword ptr [eax+4]
7CCFA156         8B49 1C             mov     ecx, dword ptr [ecx+1C]
7CCFA159         58                   pop     eax
7CCFA15A         C740 20 07000000    mov     dword ptr [eax+20], 7
7CCFA161         FFE1                jmp     ecx
```

可是由于 `0X012014B0` 先前引用计数被减 0 过，引发了后面 `CreateTearOffThunk` 对它的 `0x10` 处的 `apfnVtblObject1` 修改，`apfnVtblObject1` 早已不是先前的 `apfnVtblObject1`，而现在却要使用 `apfnVtblObject1` 来索引它偏 `0X1C` 的函数地址，`apfnVtblObject1` 被改为 `0X7CC8EFA4`，`0X7CC8EFA4` 的结构如下：

```
7CC8EFA4  7CCF9CA0  mshtml.7CCF9CA0
7CC8EFA8  7CCD4129  mshtml.7CCD4129
7CC8EFAC  7CCD4165  mshtml.7CCD4165
7CC8EFB0  7CD33D21  mshtml.7CD33D21
7CC8EFB4  7CCFA0C9  mshtml.7CCFA0C9
7CC8EFB8  7CE14F2A  mshtml.7CE14F2A
7CC8EFBC  D48A6EC6  <-----------------------这里开始存放的是 IID CLSID_HTMLWindow2
7CC8EFC0  11CF6A4A  <---------------------现在 jmp ecx 会跳到这个地址去
7CC8EFC4  4544A794
7CC8EFC8  00005453
```

就这样，一 `JMP ECX`，就触发了漏洞，`eip` 直接奔向了 `11CF6A4A` ，而该地址早已经被 `heap spray` 给占领了，后面就一堆木马被下载下来了。

### 漏洞原因及过程已经清楚了，怎么修复该漏洞？

```
if (Expression.VARTYPE !=  VT_BSTR)
{
If (VT_DISPATCH == Expression.VARTYPE )
Expression.pdispVal->AddRef();
*Result = Expression;
goto exit;
}
else
{
//执行表达式的计算
...........................
//将运算结果送入*Result
*Result = xxxxxxxxxx;
}
```

虽然这漏洞已经出来大半个月了，但还有些同学没有弄明白，所以就发出来了，有错误的地方请大家指出

格式太乱了，上传个 DOC

上传的附件

| ![img](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) || Microsoft Office Web Components. CVE-2009-1136 分析.doc.zip |
