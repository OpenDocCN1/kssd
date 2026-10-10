

# 我的第一阶段，第二题的解法。（未成功！！！）

> 原文：[`www.pediy.com/kssd/pediy10/74152.html`](https://www.pediy.com/kssd/pediy10/74152.html)

首先帖出我逆的关键函数：  

代码:

```
void CreateOutCode(DWORD \*snCode,char \*outCode,DWORD \*userCode)
{
    DWORD var\_4;
    DWORD var\_8;
    DWORD var\_C;
    DWORD var\_10;
    DWORD tempKey\[0xC\];
    DWORD tempCode\[0x30\]; 
    DWORD var\_104,var\_108,var\_10C,var\_110,var\_114,var\_118,var\_11C,var\_120,var\_124;
    var\_4 = 0;
    var\_8 = 0;
    var\_C = 0;
    var\_10 = 0;
    for(var\_4=0;var\_4<0xC;var\_4++)
    {
        tempKey\[var\_4\]=0x1E;
    }
    var\_104 = 0;
    var\_108 = 0;
    if (snCode\[0\]>0)
    {
        var\_104 = 0xC/snCode\[0\];
    }
    var\_10C = 0;
    while(var\_108<0x02)
    {
        if(var\_10C>0)
        {
            tempKey\[userCode\[var\_10C-0x01\]\] = 0x1E;
        }
        var\_10C = 0;
        while(var\_10C<0x0C)
        {
            if(var\_10C>0)
            {   //从第二位开始，保证上一位的临时值为 0x1E
                tempKey\[userCode\[var\_10C-0x01\]\] = 0x1E;
            }
            if(var\_108==0)
            {   //前半段，当前的临时值为 0x28
                tempKey\[userCode\[var\_10C\]\] = 0x28;
                var\_10C++;
            }
            if(var\_108==0x01)
            {   //前半段，当前的临时值为 0x14
                tempKey\[userCode\[var\_10C\]\] = 0x14;
                var\_10C++;
            }
            var\_4 = 0;
            var\_C = 0;
            while (var\_4<snCode\[0\])
            {   //根据输入的注册码的值，决定执行次数，第一位为 0 时，不执行。不位 0 时，执行 snCode\[0\] 次
                var\_8 = 0;
                while (var\_8<var\_104)
                {   //这里也是根据输入的注册码的值，决定执行次数，第一位为 0 时，不执行。 不位 0 时，执行 0xC/snCode\[0\] 次
                    tempCode\[var\_4\*0x04+var\_8\]=tempKey\[var\_C\]; //tempKey 只有前三位有效
                    var\_C++;
                    var\_8++;
                }
                var\_4++; 
            }//本循环执行 0C 次
            var\_110 = 0;
            var\_114 = 0;
            var\_4 = 0;
            while (var\_4<var\_104)
            {   
                //这里也是根据输入的注册码的值，决定执行次数，第一位为 0 时，不执行。 不位 0 时，执行 0xC/snCode\[0\] 次
                //可以决定输出哪些几个字符,控制方法，snCode\[0\]!=0 
                var\_110 += tempCode\[snCode\[0x01\]\*0x04+var\_4\];
                var\_114 += tempCode\[snCode\[0x02\]\*0x04+var\_4\];
                var\_4 ++;
            }
            if (var\_110==var\_114)
            {   //如果 snCode\[0\] == 0 这里永真执行，那么，输出就只有'i','j','k','l'这四个字符，所以 snCode\[0\] 不能为 0
                //那么，要输出 ijkl，就要保证 tempCode\[snCode\[0x01\]\*0x04+var\_4\] = tempCode\[snCode\[0x01\]\*0x04+var\_4\]
                //同时，需要保证 snCode\[0x01\]!=snCode\[0x01\] ，否则，也是永真执行
                if(tempCode\[snCode\[0x03\]\*0x04+snCode\[0x04\]\]!=tempCode\[snCode\[0x05\]\*0x04+snCode\[0x06\]\])
                {
                    if(tempCode\[snCode\[0x07\]\*0x04+snCode\[0x08\]\]!=tempCode\[snCode\[0x09\]\*0x04+snCode\[0xA\]\])
                    {
                        outCode\[var\_10\] = 'i'; 
                        var\_10 ++;
                    }
                    else
                    {
                        outCode\[var\_10\] = 'j'; 
                        var\_10 ++;
                    }
                }
                else
                {
                    if(tempCode\[snCode\[0x07\]\*0x04+snCode\[0x08\]\]!=tempCode\[snCode\[0x09\]\*0x04+snCode\[0xA\]\])
                    {
                        outCode\[var\_10\] = 'k'; 
                        var\_10 ++;
                    }
                    else
                    {
                        outCode\[var\_10\] = 'l'; 
                        var\_10 ++;
                    }
                }
            }
            if (var\_110>var\_114)
            {
                var\_4 = 0;
                var\_118 = 0;
                var\_11C = 0;
                tempCode\[snCode\[0x1\]\*0x4+snCode\[0x0B\]\] = tempCode\[snCode\[0x0C\]\*0x4+snCode\[0x0D\]\];
                tempCode\[snCode\[0x1\]\*0x4+snCode\[0x0E\]\] = tempCode\[snCode\[0x0F\]\*0x4+snCode\[0x10\]\];
                tempCode\[snCode\[0x1\]\*0x4+snCode\[0x11\]\] = tempCode\[snCode\[0x12\]\*0x4+snCode\[0x13\]\];
                tempCode\[snCode\[0x2\]\*0x4+snCode\[0x14\]\] = tempCode\[snCode\[0x15\]\*0x4+snCode\[0x16\]\];
                tempCode\[snCode\[0x2\]\*0x4+snCode\[0x17\]\] = tempCode\[snCode\[0x18\]\*0x4+snCode\[0x19\]\];
                tempCode\[snCode\[0x2\]\*0x4+snCode\[0x1A\]\] = tempCode\[snCode\[0x1B\]\*0x4+snCode\[0x1C\]\];
                while (var\_4<var\_104)
                {
                    //这里也是根据输入的注册码的值，决定执行次数，第一位为 0 时，不执行。 不位 0 时，执行 0xC/snCode\[0\] 次
                    //可以决定输出哪些几个字符,控制方法，snCode\[0\]!=0 

                    var\_118 += tempCode\[snCode\[0x01\]\*0x04+var\_4\];
                    var\_11C += tempCode\[snCode\[0x02\]\*0x04+var\_4\];
                    var\_4++;
                }
                if (var\_118>var\_11C)
                {
                    if(tempCode\[snCode\[0x1D\]\*0x04+snCode\[0x1E\]\]!=tempCode\[snCode\[0x1F\]\*0x04+snCode\[0x20\]\])
                    {
                        outCode\[var\_10\] = 'a'; 
                        var\_10 ++;
                    }
                    else
                    {
                        outCode\[var\_10\] = 'e'; 
                        var\_10 ++;
                    }
                }
                if (var\_118<var\_11C)
                {
                    if(tempCode\[snCode\[0x21\]\*0x04+snCode\[0x22\]\]>tempCode\[snCode\[0x23\]\*0x04+snCode\[0x24\]\])
                    {
                        outCode\[var\_10\] = 'g'; 
                        var\_10 ++;
                    }
                    if(tempCode\[snCode\[0x21\]\*0x04+snCode\[0x22\]\]<tempCode\[snCode\[0x23\]\*0x04+snCode\[0x24\]\])
                    {
                        outCode\[var\_10\] = 'f'; 
                        var\_10 ++;
                    }
                    if(tempCode\[snCode\[0x21\]\*0x04+snCode\[0x22\]\]==tempCode\[snCode\[0x23\]\*0x04+snCode\[0x24\]\])
                    {
                        outCode\[var\_10\] = 'h'; 
                        var\_10 ++;
                    }
                }
                if (var\_118==var\_11C)
                {
                    var\_4 = 0;
                    var\_C = 0;
                    while (var\_4<snCode\[0\])
                    {
                        var\_8 = 0;
                        while (var\_8<var\_104)
                        {
                            tempCode\[var\_4\*0x04+var\_8\]=tempKey\[var\_C\];
                            var\_C++;
                            var\_8++;
                        }
                        var\_4++; 
                    }
                    if(tempCode\[snCode\[0x25\]\*0x04+snCode\[0x26\]\]>tempCode\[snCode\[0x27\]\*0x04+snCode\[0x28\]\])
                    {
                        outCode\[var\_10\] = 'b'; 
                        var\_10 ++;
                    }
                    if(tempCode\[snCode\[0x25\]\*0x04+snCode\[0x26\]\]<tempCode\[snCode\[0x27\]\*0x04+snCode\[0x28\]\])
                    {
                        outCode\[var\_10\] = 'c'; 
                        var\_10 ++;
                    }
                    if(tempCode\[snCode\[0x25\]\*0x04+snCode\[0x26\]\]==tempCode\[snCode\[0x27\]\*0x04+snCode\[0x28\]\])
                    {
                        outCode\[var\_10\] = 'd'; 
                        var\_10 ++;
                    }
                }
            }
            if (var\_110<var\_114)
            {
                var\_4 = 0;
                var\_120 = 0;
                var\_124 = 0;

                tempCode\[snCode\[0x1\]\*0x4+snCode\[0x0B\]\] = tempCode\[snCode\[0x0C\]\*0x4+snCode\[0x0D\]\];
                tempCode\[snCode\[0x1\]\*0x4+snCode\[0x0E\]\] = tempCode\[snCode\[0x0F\]\*0x4+snCode\[0x10\]\];
                tempCode\[snCode\[0x1\]\*0x4+snCode\[0x11\]\] = tempCode\[snCode\[0x12\]\*0x4+snCode\[0x13\]\];
                tempCode\[snCode\[0x2\]\*0x4+snCode\[0x14\]\] = tempCode\[snCode\[0x15\]\*0x4+snCode\[0x16\]\];
                tempCode\[snCode\[0x2\]\*0x4+snCode\[0x17\]\] = tempCode\[snCode\[0x18\]\*0x4+snCode\[0x19\]\];
                tempCode\[snCode\[0x2\]\*0x4+snCode\[0x1A\]\] = tempCode\[snCode\[0x1B\]\*0x4+snCode\[0x1C\]\];
                while (var\_4<var\_104)
                {   
                    //这里可以决定输出哪些几个字符,控制方法，snCode\[0\]!=0 snCode\[1\]!=snCode\[2\]
                    var\_120 += tempCode\[snCode\[0x01\]\*0x04+var\_4\];
                    var\_124 += tempCode\[snCode\[0x02\]\*0x04+var\_4\];
                    var\_4++;
                }
                if (var\_120>var\_124)
                {
                    if(tempCode\[snCode\[0x29\]\*0x04+snCode\[0x2A\]\]>tempCode\[snCode\[0x2B\]\*0x04+snCode\[0x2C\]\])
                    {
                        outCode\[var\_10\] = 'f'; 
                        var\_10 ++;
                    }
                    if(tempCode\[snCode\[0x29\]\*0x04+snCode\[0x2A\]\]<tempCode\[snCode\[0x2B\]\*0x04+snCode\[0x2C\]\])
                    {   //这里是一个关键地方。看起来像是条件错误，仔细分析，却大有文章
                        //但 outCode\[var\_10\] = 'g' 时,var\_10++了。同时，下面的条件也执行了
                        //那么，可以说明 g 的后面，肯定是输出 h
                        outCode\[var\_10\] = 'g'; 
                        var\_10 ++;
                    }
                    if(tempCode\[snCode\[0x29\]\*0x04+snCode\[0x2A\]\]<tempCode\[snCode\[0x2B\]\*0x04+snCode\[0x2C\]\])
                    {   
                        outCode\[var\_10\] = 'h'; 
                        var\_10 ++;
                    }
                }
                if (var\_120<var\_124)
                {
                    if(tempCode\[snCode\[0x2D\]\*0x04+snCode\[0x2E\]\]!=tempCode\[snCode\[0x2F\]\*0x04+snCode\[0x30\]\])
                    {
                        outCode\[var\_10\] = 'a'; 
                        var\_10 ++;
                    }
                    else
                    {
                        outCode\[var\_10\] = 'e'; 
                        var\_10 ++;
                    }
                }
                if (var\_120==var\_124)
                {
                    var\_4 = 0;
                    var\_C = 0;
                    while (var\_4<snCode\[0\])
                    {
                        var\_8 = 0;
                        while (var\_8<var\_104)
                        {
                            tempCode\[var\_4\*0x04+var\_8\]=tempKey\[var\_C\];
                            var\_C++;
                            var\_8++;
                        }
                        var\_4++; 
                    }
                    if(tempCode\[snCode\[0x31\]\*0x04+snCode\[0x32\]\]>tempCode\[snCode\[0x33\]\*0x04+snCode\[0x34\]\])
                    {
                        outCode\[var\_10\] = 'c'; 
                        var\_10 ++;
                    }
                    if(tempCode\[snCode\[0x25\]\*0x04+snCode\[0x26\]\]<tempCode\[snCode\[0x27\]\*0x04+snCode\[0x28\]\])
                    {
                        outCode\[var\_10\] = 'b'; 
                        var\_10 ++;
                    }
                    if(tempCode\[snCode\[0x25\]\*0x04+snCode\[0x26\]\]==tempCode\[snCode\[0x27\]\*0x04+snCode\[0x28\]\])
                    {
                        outCode\[var\_10\] = 'd'; 
                        var\_10 ++;
                    }
                }
            }
        }
        var\_108++;
    }

}

```

代码没太大参考的地方。相信大家逆出来的都差不多。  
我对这一题的理解，主要是在 SN 的输入。下面的表达，有点乱，呵呵  
  
    char    szUser\[\]="aebcdfijklgh";  
  
    char    szSn\[0x36\]={'3','0','1',          
        //--------------------------------------   
        '2','0','2','1',    //1(==) 'i','j' (!=), 'k','l' (==)  3,4,5,6                          
        '2','0','2','2',    // 'i'(!=),'j'(==),'k'(!=),'l'(==)  7,8,9,A  
        //-------------------------------------- 这里对 tempCode 进行交换，达到流程的控制  
        '0','0','0',        //tempCode Edit 1\[1\]                B,C,D  
        '3','1','1',        //tempCode Edit 1\[2\]                E,F,10  
        '2','1','2',        //tempCode Edit 1\[3\]                11,12,13  
        //--------------------------------------  
        '0','1','0',        //tempCode Edit 2\[1\]                14,15,16  
        '1','0','1',        //tempCode Edit 2\[2\]                17,18,19  
        '2','2','1',        //tempCode Edit 2\[3\]                1A,1B,1C  
        //--------------------------------------  
        '0','0','0','2',     // 1(>) 2(>) OUT 'a'(!=),'e'       1D,1E,1F,20  
        //--------------------------------------  
        '0','2','0','3',    // 1(>) 2(<) OUT 'g'(>),'f'(<),'h'(==)  21,22,23,24  
        //--------------------------------------  
        '0','1','0','2',    // 1(<) 2(==) OUT 'b'(<),'d'(==)        25,26,27,28  
        // 1(>) 2(==) OUT 'b'(>),'c'(<),'d'(==)   
        //--------------------------------------  
        '0','3','0','2',    // 1(<) 2(>) OUT 'f'(>),'g''h'(<)       29,2A,2B,2C  
        //--------------------------------------  
        '0','1','1','3',    // 1(<) 2(<) OUT 'a'(!=),'e'            2D,2E,2F,30  
        //--------------------------------------  
        '0','1','1','0'};   // 1(<) 2(==) OUT 'c'(>)                31,32,33,34  
  
这是我一个失败的作品。  
我帖出来，是希望有人能指出我错误的地方。  
谢谢！ 

原来是 12 个钢球的问题。  
怎么没想到呢。哎。。。 

帖一个学习 C 语言时写的代码。仔细一比较，发现很多相同的地方。  

代码:

```
int main(int argc, char\* argv\[\])
{
  int d\[\]={1,1,1,1,1,1,1,1,1,1,1,1};
  int a\[4\],b\[4\],c\[4\];
  int nIndex;
  printf("输入 1－12 的一个数:");
  scanf("%d",&nIndex);
  printf("输入 0 或 2:");
  scanf("%d",&d\[nIndex-1\]);
  d\[nIndex-1\]=0; //也可以等于 2
  for (int i=0;i<4;i++)
  {
    a\[i\]=d\[i\];
    b\[i\]=d\[4+i\];
    c\[i\]=d\[8+i\];
  }
  if (a\[0\]+a\[1\]+a\[2\]+a\[3\]==b\[0\]+b\[1\]+b\[2\]+b\[3\])
  {
    if (a\[0\]+a\[1\]==c\[0\]+c\[1\])
    {
      if (a\[0\]==c\[2\])
      {
        printf("结果为: 12 (C\[3\])\\n");
      }
      else
      {
        printf("结果为: 11 (C\[2\])\\n");
      }
    }
    else
    {
      if (a\[0\]==c\[0\])
      {
        printf("结果为: 10 (C\[1\])\\n");
      }
      else
      {
        printf("结果为: 9 (C\[0\])\\n");
      }
    }
  }
  else if (a\[0\]+a\[1\]+a\[2\]+a\[3\]>b\[0\]+b\[1\]+b\[2\]+b\[3\]) 
  {
    if (a\[0\]+b\[0\]+b\[1\]==a\[1\]+b\[2\]+c\[0\])
    {
      if (a\[2\]==a\[3\])
      {
        printf("结果为: 8 (b\[3\])\\n");
      }
      else if (a\[2\]>a\[3\]) 
      {
        printf("结果为: 3 (a\[2\])\\n");
      }
      else
      {
        printf("结果为: 4 (a\[3\])\\n");
      }
    }
    else if (a\[0\]+b\[0\]+b\[1\]>a\[1\]+b\[2\]+c\[0\])  
    {
      if (a\[0\]>c\[0\])
      {
        printf("结果为: 1 (a\[0\])\\n");
      }
      else
      {
        printf("结果为: 7 (b\[2\])\\n");
      }
    }
    else
    {
      if (b\[0\]>b\[1\])
      {
        printf("结果为: 6 (b\[1\])\\n");
      }
      else if (b\[0\]<b\[1\])      
      {
        printf("结果为: 5 (b\[0\])\\n");
      }
      else
      {
        printf("结果为: 2 (a\[1\])\\n");
      }
    }
  }
  else
  {
    if (a\[0\]+b\[0\]+b\[1\]==a\[1\]+b\[2\]+c\[0\])
    {
      if (a\[2\]==a\[3\])
      {
        printf("结果为: 8 (b\[3\])\\n");
      }
      else if (a\[2\]>a\[3\]) 
      {
        printf("结果为: 4 (a\[3\])\\n");
      }
      else
      {
        printf("结果为: 3 (a\[2\])\\n");
      }
    }
    else if (a\[0\]+b\[0\]+b\[1\]>a\[1\]+b\[2\]+c\[0\])  
    {
      if (b\[0\]>b\[1\])
      {
        printf("结果为: 5 (b\[0\])\\n");
      }
      else if (b\[0\]<b\[1\])      
      {
        printf("结果为: 6 (b\[1\])\\n");
      }
      else
      {
        printf("结果为: 2 (a\[1\])\\n");
      }
    }
    else
    {
      if (a\[0\]<c\[0\])
      {
        printf("结果为: 1 (a\[0\])\\n");
      }
      else
      {
        printf("结果为: 7 (b\[2\])\\n");
      }
    }
  }
  return 0;
}

```