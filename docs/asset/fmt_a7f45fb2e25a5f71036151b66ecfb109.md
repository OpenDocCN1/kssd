
# 从 C 语言到汇编，从汇编到 C 语言

> 原文：[`www.pediy.com/kssd/pediy10/89379.html`](https://www.pediy.com/kssd/pediy10/89379.html)

要从事“逆向工程”工作，或对其感兴趣，请必然要接触到汇编语言。然而我们在对计算机语言的了解大都是从高级语言（如 C、VB、Dephi）开始的。那当我们阅读由高级语言翻译成的机器语言时肯定会遇到很多障碍。下面是我在看了相关书籍后，做的一些笔记，希望有点参考价值！

## 一、循环语句与发汇编后的机器语言

### 1、for 循环

下面是一段 C 语言的代码，我们的目的是来看其反汇编的结果:

```
int myfunction(int a,int b)
{
  int c = a+b;
  int i;
  for(i=0;i<50;i++)
  {
      c = c+i;
  }
  return c;
}
```

前面的反汇编暂时不理它，这里从 for 的地方开始反汇编，结果如下:

```
for(i=0;i<50;i++)
00412BC7  mov        dword ptr [i],0   // i=0; 给循环变量赋初值
00412BCE  jmp         myfunction+39h (412BD9h)// 跳到第一次循环处
00412BD0  mov        eax,dword ptr [i]
00412BD3  add         eax,1  // i++;修改循环变量
00412BD6  mov        dword ptr [i],eax
00412BD9  cmp        dword ptr [i],32h //  比较 i 与 50 的关系, 检查循环条件
00412BDD  jge         myfunction+4Ah (412BEAh)  // 当 i>=50 [即 !(i<50) ] 时则跳出循环
{
    c = c+i;
00412BDF  mov         eax,dword ptr [c]  // 变量 c
00412BE2  add         eax,dword ptr [i]   // 变量 i
00412BE5  mov         dword ptr [c],eax  // c=c+i;
}
00412BE8  jmp         myfunction+30h (412BD0h)  // 跳回去修改循环变量
00412BEA  mov         eax,dword ptr [c]
}
```

可以看到 for 循环主要用这么几条指令来实现:`mov` 进行初始化。`jmp` 跳过循环变量改变代码。`cmp` 实现条件判断，`jge` 根据条件跳转。

用 `jmp` 回到循环改变代码进行下一次循环。所以 for 结构有以下的显著特征:

```
   mov <循环变量>,<初始值>   ; 给循环变量赋初值
jmp B       ;跳到第一次循环处
A: (改动循环变量)       ;修改循环变量。
…
B: cmp <循环变量>,<限制变量>  ;检查循环条件
   jgp  跳出循环
   (循环体)
…
jmp A       ;跳回去修改循环变量
```

### 2、do 循环

再看一下 do 循环，因为 do 循环没有修改循环变量的部分，所以比 for 循环要简单一些。

```
do
   {
          c = c+i;
    00411A55  mov        eax,dword ptr [c]
    00411A58  add         eax,dword ptr [i]
    00411A5B  mov         dword ptr [c],eax
    } while(c< 100);
    00411A5E  cmp        dword ptr [c],64h
    00411A62  jl          myfunction+35h (411A55h)
    return c;
```

do 循环就是一个简单的条件跳转回去。只有两条指令：

```
cmp <循环变量>,<限制变量>
jl <循环开始点>
```

### 3、while 循环

```
while(c<100){
    00411A55  cmp         dword ptr [c],64h
    00411A59  jge         myfunction+46h (411A66h)
       c = c+i;
    00411A5B  mov         eax,dword ptr [c]
    00411A5E  add         eax,dword ptr [i]
    00411A61  mov         dword ptr [c],eax
      }
    00411A64  jmp         myfunction+35h (411A55h)
    return c;
```

很明显，我们会发现 while 要更复杂一点。因为 while 除了开始的时候判断循环条件之外，后面还必须有一条无条件跳转回到循环开始的地方，共用三条指令实现:

```
     A: cmp <循环变量>,<限制变量>
         jge  B
        ( 循环体)
        …
   jmp A
     B: (循环结束了)
```

这样，我们对 C 语言中的循环结构的分析，就基本弄完了！当然，我们用同样的方法也可以分析出 C 语言在“分支语句”、以及其它的数据结构中的 C 代码与机器反汇编代码的关系了！

为了知识的完整性，这里补充一下 C 语言里的分支语句

## 二、分支语句

### 1、if-else 语句

为了观察其汇编语句，下面是一个简单的 if 判断结构：

```
  if(a>0 && a<10)
     {
          printf("a>0");
     }
     else if( a>10 && a<100)
     {
          printf("a>10 && a<100");
     }
     else
     {
          printf("a>10 && a<100");
     }
```

if 判断都是使用 `cmp` 再加上条件跳转指令。对于 if( A && B)的情况，一般都是使用否决法。如果 A 不成立，立刻跳下一个分支。依次，如果 B 不成立，同样跳下一分支。

```
   cmp 条件
   jle 下一个分支
```

所以开始部分的反汇编为:

```
   if(a>0 && a<10)
   00411A66  cmp       dword ptr [c],0
   00411A6A  jle         411A81h   ; 跳下一个 else if 的判断点
   00411A6C  cmp       dword ptr [c],0Ah
   00411A70  jge        411A81h   ; 跳下一个 else if 的判断点
   {
        printf("a>0");
   00411A72  push      offset string "a>0" (4240DCh)
   00411A77  call        @ILT+1300(_printf) (411519h)
   00411A7C  add       esp,4
   }
```

else if 的和 else 的特点是，开始都有一条无条件跳转到判断结束处，阻止前面的分支执行结束后，直接进入这个分支。这个分支能执行到的唯一途径只是,前面的判断条件不满足。

else 则在 `jmp` 之后直接执行操作。而 else if 则开始重复 if 之后的操作，用 `cmp` 比较，然后用条件跳转指令时行跳转。

```
   else if( a>10 && a<100)
     00411A7F  jmp          411AA9h     ;直接跳到判断块外
   00411A81  cmp         dword ptr [c],0Ah         ;比较+条件跳转，目标为下一个分支处
   00411A85  jle         411A9Ch
     00411A87  cmp         dword ptr [c],64h
     00411A8B  jge         411A9Ch
     {
           printf("a>10 && a<100");
     00411A8D  push        offset string "a>10 && a<100" (424288h)
     00411A92  call          @ILT+1300(_printf) (411519h)
     00411A97  add         esp,4
      }
     else
     00411A9A  jmp        411AA9h  ;这里是 else，所以只有简单的一条跳转。
   {
       printf("a>10 && a<100");
     00411A9C  push        offset string "a>10 && a<100" (424288h)
     00411AA1  call          @ILT+1300(_printf) (411519h)
     00411AA6  add         esp,4
      }
     return c;
```

### 2、switch-case 语句

switch 的特点是有多个判断。因为 swtich 显然不用判断大于小于，所以都是 `je`（因此，C 语言中 switch 语句不支持 float 类型的变量），分别跳到每个 case 处。最后一个是无条件跳转，直接跳到 default 处。以下的代码:

```
   switch(a)
    {
    case 0:
          printf("a>0");
    case 1:
    {
          printf("a>10 && a<100");
          break;
    }
   default:
          printf("a>10 && a<100");
   }
```

反汇编的 `switch(a)`

```
   00411A66  mov         eax,dword ptr [a]
   00411A69  mov         dword ptr [ebp-0E8h],eax
   00411A6F  cmp         dword ptr [ebp-0E8h],0  // case 0:
   00411A76  je            411A83h
   00411A78  cmp         dword ptr [ebp-0E8h],1  // case 1:
   00411A7F  je            411A90h
   00411A81  jmp         411A9Fh  // default:
   {
      …
```

显然是比较 a 是否是 0、1 这两个数字。汇编指令先把 a 移动到 `[ebp-0E8h]` 这个地址，然后再比较，这是调试版本编译的特点。可能是为了防止直接操作堆栈而导致堆栈破坏？最后一条直接跳转到 default 处。当然，如果没有 default，就会跳到 swtich{}之外。

从这里我们可以发现：switch 语句里，完成“比较判断”的指令会与“case”指令的两部分，在汇编中，不是按照 C 语句逐句翻译的，而是分开为两个指令模块来实现的！

```
   case 0:
           printf("a>0");
   00411A83  push        offset string "a>0"  (4240DCh)
   00411A88  call          @ILT+1300(_printf) (411519h)
   00411A8D  add         esp,4
   case 1:
   {
        printf("a>10 && a<100");
   00411A90  push        offset string "a>10 && a<100" (424288h)
   00411A95  call          @ILT+1300(_printf) (411519h)
   00411A9A  add         esp,4
      break;
   00411A9D  jmp         myfunction+8Ch (411AACh)
   }
   default:
        printf("a>10 && a<100");
   00411A9F  push        offset string "a>10 && c<100" (424288h)
   00411AA4  call          @ILT+1300(_printf) (411519h)
   00411AA9  add         esp,4
    }
```

至于 case 和 default 分支中，如果有 `break`，则会增加一个无条件跳转汇编指令。若没有 `break`，则就没有任何循环控制代码。

小结：如果在反汇编代码中发现连续多个“比较 cmp”和“相等跳转 je”就会让人联想到“switch”语句了！
