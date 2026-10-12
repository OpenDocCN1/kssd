
# 虚函数表地址究竟放了啥！！！

> 原文：[`www.pediy.com/kssd/pediy12/131621.html`](https://www.pediy.com/kssd/pediy12/131621.html)

作者：`yulongying`

链接：`http://bbs.pediy.com/showthread.php?p=942469`

很久都没有用 `c++` 了，一直都是用 `c`，发现很多忽然生疏了不少，今天闲来无聊用 `OD` 跟踪 `c++` 类的函数表。很多人应该都知道了，所以再次声明，此文章是献给新手的，高手莫笑！

跟踪过程已经不是什么新鲜事了，这个网站就有介绍：

`http://blog.sina.com.cn/s/blog_511703010100m5rh.html`

我的源码如下：

```cpp
class CBase
{
public:
  int m_test1;
  int m_test2;
  CBase(void);
  virtual int add(int a, int b){return a+b;}
  virtual int sub(int a, int b){return a-b;}
public:
  ~CBase(void);
};

CBase::CBase(void)
{
  m_test1 = 1;
  m_test2 = 2;
}

CBase::~CBase(void)
{
}

class CFromBase :
  public CBase
{
public:
  int b;
  CFromBase(void);
  virtual int add(int a, int b){return a+b+1;}
  virtual int sub(int a, int b){return a-b-1;}
public:
  ~CFromBase(void);
};

CFromBase::CFromBase(void)
{
}

CFromBase::~CFromBase(void)
{
}

int main()
{
  CBase *base = new CFromBase();
  int a = sizeof(CBase);
  int c = sizeof(CFromBase);
  int b = base->add(1, 2);
  printf("%d hello world!, 0x%X, %d, %d", b, *(int *)base, ((int *)base)[1],((int *)base)[2]   );
  system("pause");
}
```

```asm
0041171D    6A 10           push    10
0041171F    E8 99FAFFFF     call    ClassAsm.004111BD                   ; 申请 sizeof(virtual table)+sizeof(m_test1)+sizeof(m_test2)大小的内存
00411724    83C4 04         add     esp, 4
00411727    8985 FCFEFFFF   mov     dword ptr ss:[ebp-104], eax
0041172D    C745 FC 0000000>mov     dword ptr ss:[ebp-4], 0
00411734    83BD FCFEFFFF 0>cmp     dword ptr ss:[ebp-104], 0
0041173B    74 13           je      short ClassAsm.00411750
0041173D    8B8D FCFEFFFF   mov     ecx, dword ptr ss:[ebp-104]
00411743    E8 D0F9FFFF     call    ClassAsm.00411118                  ; 子类 CFromBase 构造 跟进去
```

关键代码：

```asm
00411598    B8 CCCCCCCC     mov     eax, CCCCCCCC
0041159D    F3:AB          rep     stos dword ptr es:[edi]
0041159F    59             pop     ecx
004115A0    894D F8         mov     dword ptr ss:[ebp-8], ecx
004115A3    8B4D F8         mov     ecx, dword ptr ss:[ebp-8]
004115A6    E8 DBFBFFFF     call    ClassAsm.00411186                  ; 基 CBase 类构造
004115AB    8B45 F8         mov     eax, dword ptr ss:[ebp-8]
004115AE    C700 50664100   mov     dword ptr ds:[eax], ClassAsm.00416650      ; 虚函数表地址 00416650
```

在数据窗口查看下这个地址，在命令行窗口 `dd 00416650`：

```text
00416650  0041122B  ClassAsm.0041122B
00416654  00411230  ClassAsm.00411230
```

有两个地址 `0041122B` `00411230`。再分别查看下两个地址放的什么代码，在命令行窗口输入 `da 0041122B`：

```text
0041122B   /E9 B0030000     jmp     ClassAsm.004115E0
00411230   |E9 EB030000     jmp     ClassAsm.00411620
```

说明 `virtual table` 里面放的是一些 `jmp` 代码。跟踪不难发现 `004115E0` `00411620` 这两个地址就是 `CFromBase` 的两个虚函数 `add` `sub` 的地址。

如果在这里跟进去的时候：

```asm
004115A6    E8 DBFBFFFF     call    ClassAsm.00411186                  ; 基 CBase 类构造
```

也会类似一句：

```asm
00411486    C700 40664100   mov     dword ptr ds:[eax], ClassAsm.00416640
```

输入 `dd 00416640` 发现也放两个地址：

```text
00416640  004111E0  ClassAsm.004111E0
00416644  0041114A  ClassAsm.0041114A
```

其实这两个地址也是两个 `jmp` 代码，`jmp` 到 `CBase` 的两个虚函数 `add` `sub` 的地址。

调用 `int b = base->add(1, 2);` 的时候：

```asm
0041178A    8B45 EC         mov     eax, dword ptr ss:[ebp-14]
0041178D    8B10            mov     edx, dword ptr ds:[eax]                  ; 取出虚函数表的地址
0041178F    8B4D EC         mov     ecx, dword ptr ss:[ebp-14]             ; ecx 都知道了 放 this 指针的
00411792    8B02            mov     eax, dword ptr ds:[edx]                  ; eax 放的是 0041122b 也就是 jmp CFromBase::add 代码
00411794    FFD0            call    near eax                                ; 跳转到执行 jmp CFromBase::add 代码   可以跟进去跟进去验证下 呵呵。
```

希望跟我一样的新手们看了有点收获。呵呵！！！为了方便 `c++` 代码还是放附件里面。

补充：刚刚做了这样一个有趣的实验，代码如下：

```cpp
CBase *base = new CFromBase();
  int a = sizeof(CBase);
  int c = sizeof(CFromBase);
  int b = base->add(1, 2);
  typedef int (__stdcall *FUN)(int , int);
  FUN fun_add = (FUN)(*(int**) base)[0];
  FUN fun_sub = (FUN)(*(int**) base)[1];
  int d = fun_add(2,5);//==base->add(2, 5);
  int e = fun_sub(4,6);//==base->sub(4, 6);
  printf("0x%d hello world!, %d, %d, %d, %d", *(int *)base, ((int *)base)[1],((int *)base)[2], d, e);
```

上传的附件：

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | ClassAsm.rar |
|---|---|
