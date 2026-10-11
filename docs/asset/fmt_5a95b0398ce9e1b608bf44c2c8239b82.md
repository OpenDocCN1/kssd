
# Delphi 中的参数传递约定概述

> 原文：[`www.pediy.com/kssd/pediy10/65973.html`](https://www.pediy.com/kssd/pediy10/65973.html)

由于 Delphi 太好用了以至于大多数 Delphi fans 对 Delphi 约定都没什么认识...

抱歉其实大部分人的确是这样的

这里写下一篇浊文仅供大家参考-转载保留版权.谢谢大家支持

## 1. `register` - Delphi 默认模式

参数传递模式...前三个数据 `eax`, `edx`, `ecx`...超过三个参数部分.放在堆栈传递

其他的方法和...和 `stdcall` 一样...函数自己恢复堆栈

按照这个传递模式,所以说..这样效果编译器会更容易优化一些?呵呵

```
procedure XorMemory(lpMemory: Pointer; bKey: BYTE; dwLen: DWORD);

begin

  while (dwLen > 0) do
  begin
    PBYTE(lpMemory)^ := PBYTE(lpMemory)^ xor bKey;

    Inc(PBYTE(lpMemory));

    Dec(dwLen);

  end;

end;
```

## 2. `pascal` - 目前基本上不使用了

## 3. `cdecl` - C 语言调用约定(从右向左压栈.调用者恢复堆栈)

这个模式在 Delphi 下是一个很争议的话题..

怎么说呢..比如说 `wsprintf` 等函数都是错误的翻译...

`C\C++`下是采用 `cdecl` 调用约定,而 Delphi 下全部翻译成 `stdcall` 模式...

而且 `C\C++`总是配合可变参数一起使用的...

而 Delphi 下也是有可变参数标记的一般用户很少去关注如何使用罢了

具体看一下 `windows.pas`

```
function wsprintf(Output: PChar; Format: PChar): Integer; stdcall;
```

这里的声明类型完全是错误的...如果想要和 C 一样的方式

代码:

```
function wsprintf(Output: PChar; Format: PChar): Integer; cdecl; varargs; external user32 name 'wsprintfA';

function DbgPrint(Format:PChar): NTSTATUS; cdecl; varargs; external NtKernel name 'DbgPrint';
function _snprintf(buffer: PChar; nsize: Integer; const fmt: PChar): Integer; cdecl; varargs; external NtKernel name '_snprintf';
```

自己单独写一个函数声明...即可...你可以变参调用了...

使用的时候...和 `C\C++`下使用完全一样

这里有一个窍门...这样的函数我们如何声明函数类型?

找了大量的资料还是没招.不知道如何测试居然测试成功了...这叫啥?不知道

代码:

```
program Project2;

uses
  Windows;

//  注意看下面--cdecl varargs;之间是没有;号的
type
  TwsprintfA = function(Output: PAnsiChar; Format: PAnsiChar): Integer; cdecl varargs;

var
  fnwsprintfA: TwsprintfA;
  szBuffer: Array[0..MAX_PATH] Of Char;
begin
  @fnwsprintfA := GetProcAddress(LoadLibrary('user32'), 'wsprintfA');
  fnwsprintfA(szBuffer, 'Id: %s, Age: %d', 'Anskya', 18);
  MessageBox(0, szBuffer, 'By Anskya', 0);
end.
```

## 4. `stdcall` - 标准调用约定(从右向左压栈.函数自己恢复堆栈)

这个...基本上 api 都是采用如此调用模式..编写动态运行库的

比较重要的约定

## 5. `safecall` - Delphi 不支持..唉～牧龙鼠大牛抱歉我解决半天也没搞定

这个约定 `C\C++`支持,其实和 `register` 约定出奇的相似.

支持传递参数的寄存器不一样.
