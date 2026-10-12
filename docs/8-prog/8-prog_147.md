
# 给师弟做了刘涛涛题目

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-617.htm`](https://www.pediy.com/kssd/pediy09/pediy09-617.htm)

> 好像是刘涛涛面试的题目

```
// 刘涛涛题目
// 有一个字符串，里面包含一些数字，写一个函数， 把这些数字加起来。
// 比如“我 30 你 40 他 50”结果就是 120。
```

运行结果如下：

```
C:\tst\calc_str>cal

[+] Usage:   cal {#input string}
[-] Example: cal 我 30 你 40 他 50
[-] ---->Result: 120

C:\tst\calc_str>cal 我 30 你 40 他 50
input string length = 12
input string is:  我 30 你 40 他 50
copyed string is: 我 30 你 40 他 50
i = 3   current string: 30      sum =     30
i = 7   current string: 40      sum =     70
i = 11  current string: 50      sum =    120
---->Result: 120
```

代码如下：

```c
////////////////////////////////////////////////////////////////////////////////
// 
// 刘涛涛题目
// 有一个字符串，里面包含一些数字，写一个函数， 把这些数字加起来。
// 比如“我 30 你 40 他 50”结果就是 120。 
// coded by aker
//
////////////////////////////////////////////////////////////////////////////////

// 头文件
#include <stdio.h>

////////////////////////////////////////////////////////////////////////////////
// 全局变量

////////////////////////////////////////////////////////////////////////////////
// 参数表

////////////////////////////////////////////////////////////////////////////////
// 入口点
int main(int argc, char* argv[])
{
    char *cPtr=NULL,*cIter=NULL;
    char *buf;
    int len,i,sum=0;
    if (argc < 2)
    {
            printf("[+] Usage:   %s {#input string} \n"
                   "[-] Example: %s 我 30 你 40 他 50\n"
                   "[-] ---->Result: 120\n",
                   argv[0],argv[0]);
            exit(1);
    }
    
    // 复制字符串
    len = strlen(argv[1]);
    buf = (char*)malloc(len+1);  
    strcpy(buf,argv[1]);
    
    printf("input string length = %d\n",len);
    printf("input string is:  %s\n",argv[1]);
    printf("copyed string is: %s\n",buf);

    // 截断字符串，将其变为多个字符串，计算每个数字字串
    for(i=0;i<len;i++)
    {
        if(buf[i] >= '0' && buf[i] <= '9') // 字符为数字
        {
            cPtr = buf+i;
            cIter = cPtr+1;  
            
            while (*cIter >= '0' && *cIter <= '9') // 寻找数字字符串
            {
                cIter++;
                i++;
            }               
            *cIter = '\0';
            sum += atoi(cPtr);
            printf("i = %d\tcurrent string: %s\tsum = %6d\n",i,cPtr,sum);
        }
    }
    printf("---->Result: %d\n",sum);
    free(buf);
    return 0;
}
////////////////////////////////////////////////////////////////////////////////
```

`4dd466d0e1e0cc123afbe504da2462ca`
