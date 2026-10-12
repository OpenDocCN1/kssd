
# 逆向分析，怎么理解这个值?

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-304.htm`](https://www.pediy.com/kssd/pediy09/pediy09-304.htm)

> 如下

```
seg000:00443EDE             mov     eax, ds:dword_A14E68
seg000:00443EE3             mov     eax, [eax+2D4h]
seg000:00443EE9             mov     edx, [eax+32040h]
seg000:00443EEF             push    ebx
seg000:00443EF0             mov     ebx, [ecx]
seg000:00443EF2             push    esi
seg000:00443EF3             lea     ecx, [eax+32040h]
seg000:00443EF9             push    edi
seg000:00443EFA             call    dword ptr [edx+8]
```

> 对于这个全局变量`ds:dword_A14E68`地址存放的值经过一系列变换最后`call dword ptr [edx+8]`，我想知道对于这个`ds:dword_A14E68`地址里面的值应该怎么看,是理解为类指针吗?还是什么?

```cpp
class A14E68
{
   UnkownClass *cls2D4;
}

class UnkownClass
{
   UnkownClass2 cls32040;
}

class UnkownClass2
{
   virtual void func1();
   virtual void func2();
   virtual void func3(void arg1, void arg2, void arg3);
}
```

> `call [edx+8]`等于:
>
> `A14E68 test;`
>
> `test.cls2D4->cls32040.func3(arg1, arg2, arg3);`

`4dd466d0e1e0cc123afbe504da2462ca`
