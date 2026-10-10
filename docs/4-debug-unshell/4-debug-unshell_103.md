

# UPX 解压缩算法逆向

> 原文：[`www.pediy.com/kssd/pediy10/85348.html`](https://www.pediy.com/kssd/pediy10/85348.html)

逆解压缩算法之前 一直以为需要知道 N 多数学上的东西  逆了后才知道 只要知道基本的逻辑就可以了  
  
初次逆解压缩算法的同学 感到最困难的应该是  跟标志位有关的 加减法 以及条件跳转  
  
这些东西很难跟高级语言联系在一起 用高级语言表示特别麻烦 我在这个地方 也是绕了很大的一个圈子  
  
注：因为水平比较菜 所以我所说的逆向只是 功能相同 逻辑相似  而非指令一模一样  
  
面临失业  没有心情继续分析壳了  很久以前就想逆解压缩算法了  现在也有时间了  
  
  
  
仔细看下 UPX 的解压缩算法   里面最重要的就是两块指令  
  
1:  
  
0040ED1E    01DB            ADD EBX,EBX                                  ;EBX \* 2  
0040ED20    75 07           JNZ SHORT x.0040ED29                     ;检测 EBX 是否为 0  
0040ED22    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]           ;  
0040ED24    83EE FC        SUB ESI,-4                                      ;ESI + 4   
0040ED27    11DB            ADC EBX,EBX  
  
  
0040ED29  ^ 72 ED          JB SHORT x.0040ED18   
  
  
这块指令的 最阴的地方就是 SUB ESI, 4       SUB 指令影响 CF 标志  下面紧跟着就是一个 跟标志位有关加法  
  
C 语言中要想知道 SUB 执行后 CF 标志为多少 必须要把 ESI 的值 送到一个大于 4 字节的 数据中  这里我用了 unsigned   
  
\_\_int64 8 字节的数据来表示，而这块指令 我以 getbit 函数来表示(也可以用宏 或者 inline 但是我偏向于函数)  
  
int getbit(unsigned \_\_int64 \*pcom\_dword, unsigned char \*\*ppsrc)  
{  
    unsigned \_\_int64 temp;  
      
    \*pcom\_dword &= 0xffffffff;  
    (\*pcom\_dword) \*= 2;  
    if(!(\*pcom\_dword & 0xffffffff))  
    {  
        \*pcom\_dword = \*(unsigned int \*)(\*ppsrc);  
        temp = (unsigned int)(\*ppsrc);  
        temp -= (unsigned int)(-4);//temp 就是为了知道 SUB 后的 CF 标志 定义的变量  
        (\*ppsrc) += 4;  
        \*pcom\_dword = ((temp >> 32) & 1) + \*pcom\_dword + \*pcom\_dword;  
    }  
    return (int)((\*pcom\_dword >> 32) & 1);  
}  
  
几句汇编指令  变成 这么大一坨 惭愧  
\=================================================  
根据 4L 高手的提示 修改了 下  
int getbit(unsigned int \*pcom\_dword, unsigned int \*\*ppsrc)  
{  
  int temp;   
  
  temp = ((\*pcom\_dword)>>31)&1;//得到 符号位  
  (\*pcom\_dword) <<= 1;  
  if(0 == (\*pcom\_dword))  
  {  
    \*pcom\_dword = \*\*ppsrc;  
    temp = ((\*pcom\_dword) >> 31) & 1;  
    (\*pcom\_dword) <<= 1;  
    \*pcom\_dword += ((unsigned int)\*ppsrc >= 0xFFFFFFFC ? 0 : 1);  
    (\*ppsrc)++;  
  }  
  return temp;  
}  
  
最初 想得到标志位 也是通过 32 位表示的  方法是后 0~30 位相加  再把 2 个操作数 与 相加后的和  的第 31 位相加 就  
知道 CF 了    后来觉得 这样相加 实际上就是模拟 64 位相加  索性用 64 位表示算了  
这里修改成 32 位 简洁了很多   是因为 加法的 2 个操作数是一样的  向左移位就可以了  
  
getbit 函数改成这样 就可以直接替换原来的 getbit 解压了  果然我的思路过于狭窄  
再次膜拜下 4L 的高手  
\=========================================================  
2:  
  
0040ED30    01DB            ADD EBX,EBX  
0040ED32    75 07           JNZ SHORT x.0040ED3B  
0040ED34    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
0040ED36    83EE FC        SUB ESI,-4  
0040ED39    11DB            ADC EBX,EBX  
0040ED3B    11C0            ADC EAX,EAX  
0040ED3D    01DB            ADD EBX,EBX  
0040ED3F  ^ 73 EF          JNB SHORT x.0040ED30  
0040ED41    75 09           JNZ SHORT x.0040ED4C  
0040ED43    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]  
0040ED45    83EE FC        SUB ESI,-4  
0040ED48    11DB            ADC EBX,EBX  
0040ED4A  ^ 73 E4          JNB SHORT x.0040ED30  
  
这块指令 没有用函数表示  只是同样是一坨   再次惭愧  
  
//EBX 对应 com\_dword  ESI 对应 psrc  EAX 对应 off  
  
do  
{  
    do  
    {  
        off = off \* 2 + getbit(&com\_dword, &psrc);  
    }while(com\_dword =com\_dword + com\_dword, ((com\_dword >> 32) & 1) == 0);  
    if(com\_dword & 0xffffffff)  
    {  
        break;  
    }  
    com\_dword = \*(unsigned int \*)psrc;  
    temp = (unsigned int)psrc;  
    psrc += 4;  
    temp -= (unsigned int)(-4);              
}while(com\_dword = com\_dword + com\_dword + ((temp >> 32) & 1), ((com\_dword >> 32) & 1) == 0);  
  
这两块指令弄出来后 只剩下基本的逻辑了 没什么难度了  
逆这个东西 的作用就是知道怎么用 C 语言来表示标志位了 再来 就是 现在瞄下 压缩算法 就知道 从哪里跳出来了 不用靠猜的了    
  
UPX 的压缩级别不同 解压缩算法也是不同的  别指望 逆出来的能解压缩所有 UPX 加过壳的代码  
  
解压缩算法一直是心里的一根刺  本来想好好写写 逆向过程中的收获  逆完后 发现 什么都写不出来只能寥寥几句   
  
附件里 x.exe 就是 UPX 加过壳的 EXE， upx\_re.c 里存放的就是 根据 x.exe 逆出来的算法

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || upx.rar  
 |

应该不用这么麻烦，用 32 位足够了。  

代码:

```
int getbit(unsigned int \*pcom\_dword, unsigned int \*ppsrc)
{
    int temp;
    temp = ((\*pcom\_dword)>>31)&1;
    (\*pcom\_dword) <<= 1;
    if(0 != (\*pcom\_dword))
    {
        \*pcom\_dword = \*ppsrc;
        temp = ((\*pcom\_dword)>>31)&1;
        (\*pcom\_dword) <<= 1;
        \*pcom\_dword += ((unsigned int)ppsrc>=0xFFFFFFFC ? 0 : 1);
        (unsigned int)ppsrc += 4;
    }
    return temp;
}

```

![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")