

# SMC 之抛砖引玉(VC++6.0)

> 原文：[`www.pediy.com/kssd/pediy10/92270.html`](https://www.pediy.com/kssd/pediy10/92270.html)

SMC 是什么意思？它的英文名叫“Self Modifying Code”，顾名思义，就是“代码自修改”  
先来个简单的,来改数据  

代码:

```
CString cs1,cs2; 
cs1="123456789"; 
cs2="abcdefghi"; 
\_\_asm 
{ 
  mov esi,cs1//A 函数地址 
  mov edi,cs2//B 函数地址 
  mov ecx,9h 
  rep movsb 
} 
MessageBox(cs2); 

```

通过 MOVSB 命令来改变数据  
下面来看怎样代码:  
代码:

```
void CSMCDlg::OnOK() 
{
  msgboxB();
  BOOL (CSMCDlg::\*pa)(void)=&CSMCDlg::msgboxA;
  BOOL (CSMCDlg::\*pb)(void)=&CSMCDlg::msgboxB;
     \_\_asm
     {
       mov esi,pa
       mov edi,pb
      mov ecx,pb
      sub ecx,esi//长度要注意,这里因为 A.B 写得很近所以可以这样得出
      rep movsb
    }
  msgboxB();  
}
BOOL CSMCDlg::msgboxA()
{  
  ::MessageBox(m\_hWnd,"123456789","LILI",0);
  return true;
}
BOOL CSMCDlg::msgboxB()
{
  ::MessageBox(m\_hWnd,"987654321","LILI",0);
  return true;
}
#pragma comment(linker, "/SECTION:.text,ERW")//.text 段一定要改成可读写

```

这就是 SMC 的基本原理,用一段代码来覆盖别一段代码(注意代码的长度)  
希望大家给点建议! 

先占个位,以后有什么新的关于 SMC 的心得就写在这里  
当然也希望大家谈谈自己对 SMC 的心得!  
\-----------------------------------------------------------------------------  
今天想了个笨办法,来得到函数的(机器码的)长度:  

代码:

```
BOOL CSMCDlg::msgboxA()
{  
  ::MessageBox(m\_hWnd,"123456789","LILI",0);
  return true;
}
int CSMCDlg::Aend()
{
  return 0;
}

.......................
其它代码
.........................

BOOL CSMCDlg::msgboxB()
{
  ::MessageBox(m\_hWnd,"987654321","LILI",0);
  return true;
}
int CSMCDlg::Bend()
{
  return 1;
}

```

在一个函数(A)的结尾写一个无用的函数(Aend),并取得它的地址,两个地址相减,就得到了函数(A)的长度  
代码:

```
void CSMCDlg::OnOK() 
{
  int Alen,Blen;
  msgboxB();
  BOOL (CSMCDlg::\*pa)(void)=&CSMCDlg::msgboxA;
  BOOL (CSMCDlg::\*pb)(void)=&CSMCDlg::msgboxB;
  int (CSMCDlg::\*paend)(void)=&CSMCDlg::Aend;
  int (CSMCDlg::\*pbend)(void)=&CSMCDlg::Bend;//这两个函数的代码不能一样,否则会被 VC++优化,变成两个指针指向同一地址
  \_\_asm
  {
    mov eax,pa
    mov ebx,paend
    sub ebx,eax
    mov Alen,ebx
    mov eax,pb
    mov ebx,pbend
    sub ebx,eax
    mov Blen,ebx
     mov esi,pa
     mov edi,pb
    mov ecx,Alen//这里长度应该不会有错误了的
    rep movsb
  }
  msgboxB();  
}

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || SMC.rar  
 |