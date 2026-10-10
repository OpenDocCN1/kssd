

# 第一阶段第二题：找规律玩填字游戏

> 原文：[`www.pediy.com/kssd/pediy10/74154.html`](https://www.pediy.com/kssd/pediy10/74154.html)

bp GetWindowTextA 下断，运行后随便输入 name 和 key 点击“确定”，中断下来后 ALT+F9 返回到用户领空：  
00403C22   call      <jmp.&MFC42.#3874\_CWnd::GetWindowTextA>  
00403C27   mov    ecx, dword ptr \[ebp-4\]  
向下不远处：  
00403C84   mov     edx, dword ptr \[ebp-8\]  
00403C87   push      edx  
00403C88    mov       eax, dword ptr \[ebp-C\]  
00403C8B   push      eax  
00403C8C   call      00401113  
跟进就是处理 name 和 key 的。  
由代码：  
00403335    cmp     eax, 0C      ;  用户名长度必须是 12  
00403338    je      short 0040334E  
知道用户名长度必须是 12；从后面的分析还知道 name 串中不能有重复的字符。  
向下跟到：  
00403598    cmp     eax, 35      ;  key 长度要大于 35H  
0040359B   jnb       short 004035B1  
知道 key 长度要大于等于 53，从后面的分析还知道 key 串中字符只可能取‘0’，‘1’，‘2’，‘3’中的一个。向下跟到：  
00403615     push      ecx  
00403616     lea       edx, dword ptr \[ebp-100\]  
0040361C    push      edx  
0040361D     lea       eax, dword ptr \[ebp-2B0\]  
00403623    push      eax          ;  key 每个字符减 30H 后按 DWORD 存放  
00403624     call     00401087  
……  
00401087     jmp       004022A0  
……  
便到了主要部分了，sub\_004022A0 函数是用来验证 key 的，其复杂程度相当大!!!好多人在此放弃了，sub\_004022A0 函数共有三个参数，第一个 dwKey 是输入的 key 串逐个字符转换成整型后存储的内存地址；第二个参数 buffer 是存储生成的字符串的缓冲区，大小为 24 字节；第三个参数 a3 为：{0,1,2,3,4,5,6,7,8,9,0x0A,0x0B}；现将整理后的 C 代码贴出来分析：  

引用: /\*\[第一阶段◇第二题\]看雪论坛.腾讯公司 2008 软件安全技术竞赛  
            拨开乌云见月明   
        key 验证函数反汇编代码  
            singsing 整理  
\*/  
  
typedef long DWORD;  
typedef char byte;  
  
  
int \_\_cdecl sub\_4022A0(DWORD dwKey\[\], byte buffer\[\], DWORD a3\[\])  
{  
    int result;    
    signed int v4;    
    int v5;   
    DWORD v6\[12\];    
    int v7;    
    signed int v8;    
    signed int v9;    
    int v10;    
    int v11;    
    byte v12\[192\];    
    int v13;    
    int v14;    
    int v15;    
    int v16;    
    int v17;    
    int v18;    
  
    v4 = 0;  
    v5 = 0;  
    while ( v4 < 12 )  
    {  
        result = v4;  
        v6\[v4++\] = 30;  
    }  
    v7 = 0;  
    v8 = 0;  
    if ( dwKey\[0\] > 0 )  
    {  
        result = 12 / dwKey\[0\];  
        v7 = 12 / dwKey\[0\];  
    }  
    v9 = 0;  
    while ( v8 < 2 )  
    {  
        if ( v9 > 0 )  
            v6\[a3\[v9-1\]\] = 30;  
        v9 = 0;  
        while ( v9 < 12 )  
        {  
            if ( v9 > 0 )  
                v6\[a3\[v9-1\]\] = 30;  
            if ( !v8 )  
                v6\[a3\[v9++\]\] = 40;  
            if ( v8 == 1 )  
                v6\[a3\[v9++\]\] = 20;  
            v10 = 0;  
            v4 = 0;  
            while ( v4 < dwKey\[0\] )  
            {  
                v11 = 0;  
                while ( v11 < v7 )  
                    v12\[v4\*4+v11++\]=v6\[v10++\];  
                ++v4;  
            }  
            v13 = 0;  
            v14 = 0;  
            v4 = 0;  
            while ( v4 < v7 )  
            {  
                v13 += v12\[4 \* dwKey\[1\] + v4\];  
                v14 += v12\[4 \* dwKey\[2\] + v4\];  
                v4++;  
            }  
            if ( v13 == v14 )  
            {  
                if ( v12\[4 \* dwKey\[3\] + dwKey\[4\]\] == v12\[4 \* dwKey\[5\] + dwKey\[6\]\] )  
                {  
                    if ( v12\[4 \* dwKey\[7\] + dwKey\[8\]\] == v12\[4 \* dwKey\[9\]+ dwKey\[10\]\] )  
                        buffer\[v5++\] = 'l';  
                    else  
                        buffer\[v5++\] = 'k';  
                }  
                else  
                {  
                    if ( v12\[4 \* dwKey\[7\] + dwKey\[8\]\] == v12\[4 \* dwKey\[9\]+ dwKey\[10\]\] )  
                        buffer\[v5++\] = 'j';  
                    else  
                        buffer\[v5++\] = 'i';  
                }  
            }  
            if ( v13 > v14 )  
            {  
                v4 = 0;  
                v15 = 0;  
                v16 = 0;  
                v12\[4 \* dwKey\[1\] + dwKey\[11\]\] = v12\[4 \* dwKey\[12\]+ dwKey\[13\]\];  
                v12\[4 \* dwKey\[1\] + dwKey\[14\]\] = v12\[4 \* dwKey\[15\]+ dwKey\[16\]\];  
                v12\[4 \* dwKey\[1\] + dwKey\[17\]\] = v12\[4 \* dwKey\[18\]+ dwKey\[19\]\];  
                v12\[4 \* dwKey\[2\] + dwKey\[20\]\] = v12\[4 \* dwKey\[21\]+ dwKey\[22\]\];  
                v12\[4 \* dwKey\[2\] + dwKey\[23\]\] = v12\[4 \* dwKey\[24\]+ dwKey\[25\]\];  
                v12\[4 \* dwKey\[2\] + dwKey\[26\]\] = v12\[4 \* dwKey\[27\]+ dwKey\[28\]\];  
                while ( v4 < v7 )  
                {  
                    v15 += v12\[4 \* dwKey\[1\] + v4\];  
                    v16 += v12\[4 \* dwKey\[2\] + v4++\];  
                }  
                if ( v15 > v16 )  
                {  
                    if ( v12\[4 \* dwKey\[29\] + dwKey\[30\]\] == v12\[4 \* dwKey\[31\]+ dwKey\[32\]\] )  
                        buffer\[v5++\] = 'e';  
                    else  
                        buffer\[v5++\] = 'a';  
                }  
                if ( v15 < v16 )  
                {  
                    if ( v12\[4 \* dwKey\[33\] + dwKey\[34\]\] > v12\[4 \* dwKey\[35\]+ dwKey\[36\]\] )  
                        buffer\[v5++\] = 'g';  
                    if ( v12\[4 \* dwKey\[33\] + dwKey\[34\]\] < v12\[4 \* dwKey\[35\]+ dwKey\[36\]\] )  
                        buffer\[v5++\] = 'f';  
                    if ( v12\[4 \* dwKey\[33\] + dwKey\[34\]\] ==v12\[4 \* dwKey\[35\]+ dwKey\[36\]\] )  
                        buffer\[v5++\] = 'h';  
                }  
                if ( v15 == v16 )  
                {  
                    v10 = 0;  
                    v4 = 0;  
                    while ( v4 < dwKey\[0\] )  
                    {  
                        v11 = 0;  
                        while ( v11 < v7 )  
                        v12\[4 \* v4 + v11++\] = v6\[v10++\];  
                        ++v4;  
                    }  
                    if ( v12\[4 \* dwKey\[37\] + dwKey\[38\]\] > v12\[4 \* dwKey\[39\]+ dwKey\[40\]\] )  
                        buffer\[v5++\] = 'b';  
                    if ( v12\[4 \* dwKey\[37\] + dwKey\[38\]\] < v12\[4 \* dwKey\[39\]+ dwKey\[40\]\] )  
                        buffer\[v5++\] = 'c';  
                    if ( v12\[4 \* dwKey\[37\] + dwKey\[38\]\] ==v12\[4 \* dwKey\[39\]+ dwKey\[40\]\] )  
                        buffer\[v5++\] = 'd';  
                }  
            }  
            if ( v13 < v14 )  
            {  
                v4 = 0;  
                v17 = 0;  
                v18 = 0;  
                v12\[4 \* dwKey\[1\] + dwKey\[11\]\] = v12\[4 \* dwKey\[12\]+ dwKey\[13\]\];  
                v12\[4 \* dwKey\[1\] + dwKey\[14\]\] = v12\[4 \* dwKey\[15\]+ dwKey\[16\]\];  
                v12\[4 \* dwKey\[1\] + dwKey\[17\]\] = v12\[4 \* dwKey\[18\]+ dwKey\[19\]\];  
                v12\[4 \* dwKey\[2\] + dwKey\[20\]\] = v12\[4 \* dwKey\[21\]+ dwKey\[22\]\];  
                v12\[4 \* dwKey\[2\] + dwKey\[23\]\] = v12\[4 \* dwKey\[24\]+ dwKey\[25\]\];  
                v12\[4 \* dwKey\[2\] + dwKey\[26\]\] = v12\[4 \* dwKey\[27\]+ dwKey\[28\]\];  
                while ( v4 < v7 )  
                {  
                    v17 += v12\[4 \* dwKey\[1\] + v4\];  
                    v18 += v12\[4 \* dwKey\[2\] + v4++\];  
                }  
                if ( v17 > v18 )  
                {  
                    if ( v12\[4 \* dwKey\[41\] + dwKey\[42\]\] > v12\[4 \* dwKey\[43\]+ dwKey\[44\]\] )  
                        buffer\[v5++\] = 'f';  
                    if ( v12\[4 \* dwKey\[41\] + dwKey\[42\]\] < v12\[4 \* dwKey\[43\]+ dwKey\[44\]\] )  
                        buffer\[v5++\] = 'g';  
                    if ( v12\[4 \* dwKey\[41\] + dwKey\[42\]\] < v12\[4 \* dwKey\[43\]+ dwKey\[44\]\] )  
                        buffer\[v5++\] = 'h';  
                }  
                if ( v17 < v18 )  
                {  
                    if ( v12\[4 \* dwKey\[45\] + dwKey\[46\]\] == v12\[4 \* dwKey\[47\]+ dwKey\[48\]\] )  
                        buffer\[v5++\] = 'e';  
                    else  
                        buffer\[v5++\] = 'a';  
                }  
                if ( v17 == v18 )  
                {  
                    v10 = 0;  
                    v4 = 0;  
                    while ( v4 < dwKey\[0\] )  
                    {  
                        v11 = 0;  
                        while ( v11 < v7 )  
                        v12\[4 \* v4 + v11++\] = v6\[v10++\];  
                        ++v4;  
                    }  
                    if ( v12\[4 \* dwKey\[49\] + dwKey\[50\]\] > v12\[4 \* dwKey\[51\]+ dwKey\[52\]\] )  
                        buffer\[v5++\] = 'c';  
                    if ( v12\[4 \* dwKey\[49\] + dwKey\[50\]\] < v12\[4 \* dwKey\[51\]+ dwKey\[52\]\] )  
                        buffer\[v5++\] = 'b';  
                    if ( v12\[4 \* dwKey\[49\] + dwKey\[50\]\] ==v12\[4 \* dwKey\[51\]+ dwKey\[52\]\] )  
                        buffer\[v5++\] = 'd';  
                }  
            }  
        }// end v9 < 12  
        result = v8++ + 1;  
    }//end while ( v8 < 2 )  
    return result;  
}  
  
void main()  
{  
    DWORD dwKey\[53\]={3,0,1,2,1,2,0,2,2,2,0,1,1,1,2,1,2,3,1,3,1,2,1,2,2,2,3,2,3,0,0,0,1,0,1,0,2,0,1,0,2,0,1,0,2,0,1,0,0,0,1,0,2};  
    DWORD a3\[12\]={0,1,2,3,4,5,6,7,8,9,0x0A,0x0B};  
    char  buffer\[50\];  
    int ret;  
  
    ret=sub\_4022A0(dwKey,buffer,a3);  
} 

  
  可以看到最终生成的 buffer 串字符是由字符‘a’到‘l’组成的，而且 buffer 最终长度是 24，因此最先想到构造的 buffer 串是：“abcdefghijklabcdefghijkl”。当然要先确定这个串是否合理，具体方法是在 sub\_004022A0 函数的出口“00402F27   retn”处 F4 运行至，然后观察 buffer 的内容，二进制编辑成“abcdefghijklabcdefghijkl”，F9 运行发现弹出对话框“ok！”，说明构造的这个 buffer 是合理的。  
  我“悟出”的算法思想是这样的：  
外层循环 2 次，内层循环 12 次，也就是说尽量在第一次大循环时就构造出“abcdefghijkl”；dwKey\[0\]不为空，我这里设置为 3；将 12 分割为 3 块，每块元素有四个（12 div 3）。  
  
  
![](img/3e2c55dfc29c01e3ac14031f35c5da2a.jpg)  
  
  每次循环元素 40 的位置下移一位，上图中把 V12 的 12 个关键元素分割为 3 块，分别标示为 0 块，1 块，2 块。如果设置 dwKey\[1\]=0，dwKey\[2\]=1，则 v13 和 v14 是分别求 0 块和 1 块 4 个元素之和的。  
  第一次循环中，40 在 0 块的第 0 个元素，因此 v13 > v14，在 C 代码里找到条件“if ( v13 > v14 )”进去分析。  
“  
v12\[4 \* dwKey\[1\] + dwKey\[11\]\] = v12\[4 \* dwKey\[12\]+ dwKey\[13\]\];  
v12\[4 \* dwKey\[1\] + dwKey\[14\]\] = v12\[4 \* dwKey\[15\]+ dwKey\[16\]\];  
v12\[4 \* dwKey\[1\] + dwKey\[17\]\] = v12\[4 \* dwKey\[18\]+ dwKey\[19\]\];  
v12\[4 \* dwKey\[2\] + dwKey\[20\]\] = v12\[4 \* dwKey\[21\]+ dwKey\[22\]\];  
v12\[4 \* dwKey\[2\] + dwKey\[23\]\] = v12\[4 \* dwKey\[24\]+ dwKey\[25\]\];  
v12\[4 \* dwKey\[2\] + dwKey\[26\]\] = v12\[4 \* dwKey\[27\]+ dwKey\[28\]\];  
”  
这段代码是对 v12 表元素换位用的，暂不分析，向下看 v15 和 v16 也是分别求 0 块和 1 块 4 个元素之和的。如果不对 v12 表的元素进行换位操作，则第一次 v15 > v16，满足条件“if ( v15 > v16 )”进去分析。  
“  
  if ( v12\[4 \* dwKey\[29\] + dwKey\[30\]\] == v12\[4 \* dwKey\[31\]+ dwKey\[32\]\] )  
            buffer\[v5++\] = 'e';  
  else  
            buffer\[v5++\] = 'a';  
”  
欲得到‘a’，则使 4 \* dwKey\[29\] + dwKey\[30\]不等于 4 \* dwKey\[31\]+ dwKey\[32\]，则令  
4 \* dwKey\[29\] + dwKey\[30\]=0（此时 v12\[0\]=40）,令 4 \* dwKey\[31\]+ dwKey\[32\]=1（此时 v12\[1\]=30）,故 dwKey\[29\]= dwKey\[30\]=0，dwKey\[31\]=0，dwKey\[32\]=1.  
  如果按上述分析结果重新设置 dwKey，则可以得到第一个字符‘a’。由于后面几轮循环仍是 v13 > v14，而又想得到字符‘b’，‘c’，‘d’，则要满足 v15 == v16，因此就要考虑上面提及的 v12 表元素换位代码。  
经过分析换位形式为：  
            0 块换用 1 块元素，换用元素索引为 1，2，3  
            1 块换用 2 块元素，换用元素索引为 1，2，3  
只有这样后面连续的 3 次循环 v15 才能等于 v16，故有：  
4 \* dwKey\[1\] + dwKey\[11\]=0/1  
4 \* dwKey\[1\] + dwKey\[14\]=0/2  
4 \* dwKey\[1\] + dwKey\[17\]=0/3  
4 \* dwKey\[12\]+ dwKey\[13\]=0/1  
4 \* dwKey\[15\]+ dwKey\[16\]=0/2  
4 \* dwKey\[18\]+ dwKey\[19\]=0/3  
4 \* dwKey\[2\] + dwKey\[20\]=0/1  
4 \* dwKey\[2\] + dwKey\[23\]=0/2  
4 \* dwKey\[2\] + dwKey\[26\]=0/3  
4 \* dwKey\[21\]+ dwKey\[22\]=0/1  
4 \* dwKey\[24\]+ dwKey\[25\]=0/2  
4 \* dwKey\[27\]+ dwKey\[28\]=0/3  
已知 dwKey\[1\]=0，dwKey\[2\]=1，进行简单求解得：  
dwKey\[12\]=dwKey\[15\]=dwKey\[18\]=1；  
dwKey\[21\]=dwKey\[24\]=dwKey\[27\]=2；  
简单求解得：  
dwKey\[11\]=1 dwKey\[13\]=1 dwKey\[20\] =1 dwKey\[22\] =1  
dwKey\[14\]=2 dwKey\[16\]=2 dwKey\[23\] =2 dwKey\[25\] =2  
dwKey\[17\]=3 dwKey\[19\]=3 dwKey\[26\] =3 dwKey\[28\] =3  
  
按照这个规则重写 dwKey，这样第 2，3，4 次循环便满足条件“if ( v15 == v16 )”，  
           if ( v12\[4 \* dwKey\[37\] + dwKey\[38\]\] > v12\[4 \* dwKey\[39\]+ dwKey\[40\]\] )  
            buffer\[v5++\] = 'b';  
          if ( v12\[4 \* dwKey\[37\] + dwKey\[38\]\] < v12\[4 \* dwKey\[39\]+ dwKey\[40\]\] )  
            buffer\[v5++\] = 'c';  
          if ( v12\[4 \* dwKey\[37\] + dwKey\[38\]\] ==v12\[4 \* dwKey\[39\]+ dwKey\[40\]\] )  
            buffer\[v5++\] = 'd';  
第 2 次循环 v12\[1\]=40，第 3 次循环 v12\[2\]=40，第 4 次循环 v12\[3\]=40，想要依次得到‘b’，‘c’，‘d’，则令 4 \* dwKey\[37\] + dwKey\[38\]=1；4 \* dwKey\[39\]+ dwKey\[40\]=2 即可。  
简单求解得：  
dwKey\[37\]=0；dwKey\[38\]=1；dwKey\[39\]=0；dwKey\[40\]=2；  
  
  第 5 次循环开始 40 元素便在 v12 表的第 1 块了，因此“v13 < v14”，下面要得到字符‘e’，就要使“v12\[4 \* dwKey\[45\] + dwKey\[46\]\] == v12\[4 \* dwKey\[47\]+ dwKey\[48\]\]”成立（v17 < v18 已经成立，因为代码的条件是对称，分析当如 v13 与 v14）。因为 v12 表中的 12 个元素 11 个是 30，只有一个是 40，因此随便填写 dwKey\[45\]，dwKey\[46\]，dwKey\[47\]，dwKey\[48\]满足上式成立即可。  
  再下面是字符‘f’，‘g’，‘h’，这里要注意一点是虽然第 6 次循环本来 v12\[5\]=40，但是经过换位处理后 v12\[1\]=40.所以有：  
4 \* dwKey\[41\] + dwKey\[42\]=1；4 \* dwKey\[43\]+ dwKey\[44\]=2；  
简单求解得：  
dwKey\[41\]=0，dwKey\[42\]=1，dwKey\[43\]=0，dwKey\[44\]=2.  
  
接下来是字符‘i’，‘j’，‘k’，‘l’，这个有点技巧。如果要按照字符大小顺序来计算的话，条件形式为：  
不等/不等；不等/相等；相等/不等；相等/相等。  
第 9 次循环： v12\[8\]=40，v12\[9\]=30，v12\[10\]=30，v12\[11\]=30。  
第 10 次循环：v12\[8\]=30，v12\[9\]=40，v12\[10\]=30，v12\[11\]=30。  
第 11 次循环：v12\[8\]=30，v12\[9\]=30，v12\[10\]=40，v12\[11\]=30。  
第 12 次循环：v12\[8\]=30，v12\[9\]=30，v12\[10\]=30，v12\[11\]=40。  
我选择的比较方式是 v12\[9\]和 v12\[8\]比较，v12\[10\]和 v12\[8\]比较，这样就能满足上述条件形式，便可以顺序得到字符‘i’，‘j’，‘k’，‘l’。所以有：  
4 \* dwKey\[3\] + dwKey\[4\]=9； 4 \* dwKey\[5\] + dwKey\[6\]=8；  
4 \* dwKey\[7\] + dwKey\[8\]=10；4 \* dwKey\[9\]+ dwKey\[10\]=8；  
得：dwKey\[3\]=2； dwKey\[4\]=1； dwKey\[5\]=2； dwKey\[6\]=0；  
 dwKey\[7\]=2； dwKey\[8\]=2； dwKey\[9\]=2； dwKey\[10\]=0；  
  
按照上述规则重写 dwKey，在第一次 12 轮循环之后便可以得到字符串“abcdefghijkl”。但是在第二轮的循环中还需要注意几点（关键元素 40 变为了 20），不过也很简单，主要是因为算法是对称的。  
在确定第二轮的‘b’，‘c’，‘d’字符时，令 4 \* dwKey\[49\] + dwKey\[50\]=1；4 \* dwKey\[51\]+ dwKey\[52\]=2；  
得：dwKey\[49\]=0，dwKey\[50\]=1，dwKey\[51\]=0，dwKey\[52\]=2。  
在确定第二轮的‘f’，‘g’，‘h’字符时，令 4 \* dwKey\[33\] + dwKey\[34\]=1；4 \* dwKey\[35\]+ dwKey\[36\]=2；  
得：dwKey\[33\]=0，dwKey\[34\]=1，dwKey\[35\]=0，dwKey\[36\]=2。  
  重写 dwKey，如：  
{3,0,1,2,1,2,0,2,2,2,0,1,1,1,2,1,2,3,1,3,1,2,1,2,2,2,3,2,3,0,0,0,1,0,1,0,2,0,1,0,2,0,1,0,2,0,1,0,0,0,1,0,2};  
输出的 buffer 串便是“abcdefghijklabcdefghijkl”.  
现在用 cm.exe 程序测试，输入序列：  
abcdefghijkl  
30121202220111212313121222323000101020102010201000102  
点击“确定”后弹出“ok!”:  
  
![](img/d6b0d1c293fc450136417fad55dc389e.jpg)正确的序列很多，只要 name 合理 key 就能用，感觉 netwind 所说的 keygen 似乎不合理，应该是 namegen 吧？而且 cm.exe 程序有个 bug：只要 key 合理，则后加 N 个合法字符（0，1，2，3）也合理，因为 sub\_004022A0 不验证 53 字节之后的字符。  
  
  本人对上述算法不懂，仅仅是靠找规律玩填字游戏而已，因此未能提出通用的解法。  
  
by  sing  
2008-10-7