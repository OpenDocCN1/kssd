
# 对 `api` 名字 `hash` 的一点理解

> 原文：[`www.pediy.com/kssd/pediy10/79326.html`](https://www.pediy.com/kssd/pediy10/79326.html)

`Hash` `windows` `api` `method`

为了控制 `shellcode` 大小，总将 `api` 的名字用一个算法求得对应的一个 `Dword` 数（32 位）。

2 的 32 次方可以表达 4 294 967 296 种可能。

网络常用的一个 `hash` 算法的 `c` 语言实现如下

```c
#include <windows.h>

#include <stdio.h>

DWORD GetHash(char *fun_name)

{

DWORD digest=0;

while(*fun_name)

{

  digest=((digest<<25)|(digest>>7));

  digest+=*fun_name;

  fun_name++;

}

return digest;

}

void main()

{

DWORD hash;

hash=GetHash("MessageBoxA");

printf("result of hash is %.8x\n",hash);

}
```

基本算法就是 `h=h<<25 | h>>7 +*fun_name`

用汇编语言实现

```assembly
xor edx,edx        ;ebx equals zero

mov esi,offset funcname      ;get the start of api name string

1oop:

Movsz eax,byte prt[esi]      ;get one byte from name string to eax

cmp al,ah                   ;if we get zero then Exit

jz Exit

ror  edx ,07h        ;this statement equals digest=((digest<<25)|(digest>>7));

add edx,eax        ; this statement equals digest+=*fun_name;

add esi,1        ; ask esi to point to the next byte

jmp 1oop

Exit:
```

一时间没有想明白 `ror  edx ,07h` 为什么等于 `digest=((digest<<25)|(digest>>7))`

好笨，昨天画了画明白了~

假如有这样一来一个 32 位二进制数

执行完 `digest<<25` 后把右边的 7 位移动到左边，然后右边七位补 25 个 0

执行完 `digest>>7` 后把左边 25 位移动到右边，然后左边 7 位补 0

把两个数或运算后 正好跟循环右移 7 位是相等的。
