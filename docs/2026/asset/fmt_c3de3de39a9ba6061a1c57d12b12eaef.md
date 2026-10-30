
```c
char __cdecl shellcode_exec(int critical_class, int a2, int a3)

{

  int v3; // esi@1

  char v4; // bl@1

  bool v5; // zf@3

  int v6; // eax@9

  int v7; // esi@15

  int v8; // ecx@15

  int v9; // eax@17

  int v10; // esi@32

  int v11; // eax@44

  int v12; // eax@66

  int v13; // esi@66

  int v14; // eax@76

  int v15; // eax@90

  int v17; // eax@95

  int v18; // [sp+1Ch] [bp-54h]@58

  int v19; // [sp+24h] [bp-4Ch]@15

  int v20; // [sp+28h] [bp-48h]@15

  int v21; // [sp+2Ch] [bp-44h]@44

  int v22; // [sp+34h] [bp-3Ch]@15

  int v23; // [sp+38h] [bp-38h]@15

  int v24; // [sp+3Ch] [bp-34h]@14

  int v; // [sp+40h] [bp-30h]@1

  int v26; // [sp+44h] [bp-2Ch]@9

  int v27; // [sp+4Ch] [bp-24h]@49

  int v28; // [sp+50h] [bp-20h]@67

  int v29; // [sp+54h] [bp-1Ch]@1

  _DWORD *v30; // [sp+58h] [bp-18h]@1

  int v31; // [sp+5Ch] [bp-14h]@1

  int v32; // [sp+6Ch] [bp-4h]@1

  char v33; // [sp+70h] [bp+0h]@92

  int v34; // [sp+74h] [bp+4h]@86

  char v35; // [sp+78h] [bp+8h]@11

  int v36; // [sp+7Ch] [bp+Ch]@5

  char v37; // [sp+98h] [bp+28h]@14

  char v38; // [sp+B8h] [bp+48h]@15

  char v39; // [sp+1BCh] [bp+14Ch]@15

  char v40; // [sp+2C0h] [bp+250h]@15

  char v41; // [sp+3C4h] [bp+354h]@15

  char v42; // [sp+4C8h] [bp+458h]@15

  char v43; // [sp+5CCh] [bp+55Ch]@15

  char v44; // [sp+604h] [bp+594h]@76



  v3 = a2;

  v29 = a2;

  EnterCriticalSection(&CriticalSection);

  v4 = 0;

  v32 = 0;

  v = 0;

  v31 = 0;

  v30 = (_DWORD *)sub_801BAD4(critical_class);

  if ( v30

    && (unsigned __int8)shellcode_exec2(

                          (int (__stdcall ***)(_DWORD, _DWORD, _DWORD, _DWORD, _DWORD, _DWORD))v30,

                          critical_class,

                          (int)&v,

                          (int)&v31,

                          0,

                          0,

                          1) )

  {  //以下省略}

}



int __cdecl shellcode_exec2(int (__stdcall ***a1)(_DWORD, _DWORD, _DWORD, _DWORD, _DWORD, _DWORD), int critical_class, int a3, int a4, int a5, int a6, int a7)

{

  int (__thiscall **v7)(void *, _DWORD, _DWORD, _DWORD, _DWORD, _DWORD, _DWORD); // eax@1

  int result; // eax@1



  v7 = (int (__thiscall **)(void *, _DWORD, _DWORD, _DWORD, _DWORD, _DWORD, _DWORD))*a1;

  ++dword_823A6A0;

  result = ((int (__stdcall *)(int, int, int, int, int, int))*v7)(critical_class, a3, a4, a5, a6, a7);// shell_code execute3

  if ( !(_BYTE)result )

    --dword_823A6A0;

  return result;

}



char __thiscall shellcode_exec3(void *this, int critical_class, int *a3, _DWORD *a4, _DWORD *a5, unsigned int *a6, int a7)

{

  void *v7; // esi@1

  int v8; // eax@13

  int v9; // eax@14

  int v10; // eax@15

  unsigned int v11; // eax@17

  _DWORD *v12; // edx@20

  _DWORD *v13; // edx@21

  int *v14; // ecx@23

  int v15; // edx@24

  int v16; // eax@24

  int v17; // eax@25

  void (__cdecl **father_of_cc)(_DWORD, _DWORD, _DWORD, _DWORD); // eax@25

  unsigned int v20; // eax@36

  _DWORD *v21; // edx@40

  int v22; // eax@41

  _DWORD *v23; // edx@41

  void (__cdecl **v24)(_DWORD, _DWORD, _DWORD, _DWORD); // eax@43

  void (__cdecl **v25)(_DWORD, _DWORD, _DWORD, _DWORD); // ST00_4@49

  void (__cdecl **v26)(_DWORD, _DWORD, _DWORD, _DWORD); // ST00_4@50

  _DWORD *v27; // edi@60

  unsigned int v28; // [sp+Ch] [bp-4h]@25



  v7 = this;

  if ( !(unsigned __int8)sub_808AEE2(critical_class) || (_BYTE)a7 )

  {

    v17 = *(_DWORD *)v7;

    BYTE3(a7) = 0;

    (*(void (**)(void))(v17 + 112))();

    sub_801E4F8(critical_class);

    *((_BYTE *)v7 + 224) = 1;

    father_of_cc = *(void (__cdecl ***)(_DWORD, _DWORD, _DWORD, _DWORD))(critical_class + 60);// [[a2+60]] is a function

                                                // probaly the virtual function of multi extends class

    *((_DWORD *)v7 + 189) = father_of_cc;

    *((_DWORD *)v7 + 190) = 0;

    v28 = 0;

    if ( !father_of_cc )

      return 0;

    (*father_of_cc)(father_of_cc, 3, 0, &v28);  // ROP start

    *((_DWORD *)v7 + 203) = v28;

  }

  //以下省略

}
```

### PS

`critical_class`就是下面的`p`，一直往里面传进去的，我就逆着实参形参跟。不过貌似跟上去也没啥用，哈哈，就记住`[[p+60]]`会被`call`就是了

### 劫持 eip 的方法

1. 最后调用`shellcode_exec3`的时候会把传入的类指针`p`的`[[p+60]]`进行调用，而[`p+60`]是指向栈上的，而且是`strcat`可以影响到的内存，一开始觉得是多重继承的虚函数调用，但是虚表应该是在某个加载模块而不是栈上，我就很迷这个Adobe的程序员用的是什么野路子
2. 如何在茫茫大海中找到这个`call`？我总结的就是可以先全部填`AAAAAA`，然后看`exception`，遇到读写错误这些的就找个稳定可读写的内存解决了，然后最后看到个执行错误`eip=0x41414141`就谢天谢地了（虽然我觉得这种`call`栈上的地址这种事情不是一个正常的C++程序员能写出来的。。。）

### ROP

1. 为了绕过`ASLR`，`ROP`始终使用`icucnv36.dll`模块（这个模块不被`ASLR`保护）
2. 首先加`esp`，因为这个时候调用了三层子函数，`strcat`污染到的缓冲区在很上面，所以先把`esp`加高
   然后`pop esp`到`0x0c0c0c0c`，这个时候稳定是`ROP`的栈内容，然后进入各种`ROP`的轮回。。。
   具体好像是通过`file mapping`那套理论把`shellcode`拷到所建立的`RWX`内存里面
   这里注意一下，因为`ASLR`所以不能直接`retn kernel32`这种模块
   那想要调用`API`怎么办呢，利用`icucnv36.dll`的导入表
   栈中存`icucnv36.dll`的`IAT`，然后`pop`出来，找`ROP`类似`mov eax,[eax]`这种，函数指针取出来
   然后`ROP call eax`这样（或者直接找`call [eax]`，无所谓啦）
3. 最后跳到所建立的内存里面执行

具体可以`OD`跟一下

### javascript

1. 通过某种神秘方法，稳定将`ROP`复制到`0x0c0c0c0c`处（不知道怎么实现的，如果有大神可以指出）
2. 用`pdf stream`查看`JavaScript`发现很大的一部分是乱码，运行也报错，不知道是怎么实现的（报错还能实际运行成功，可能是什么加密方式，哈哈，不知道，求大神指出）
3. `js`代码从漏洞战争里看，有点类似堆喷射的代码，但又不是

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
