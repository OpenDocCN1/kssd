

# 【看雪论坛.腾讯公司 2008 软件安全竞赛】我的第二题分析过程及注册机

> 原文：[`www.pediy.com/kssd/pediy10/74166.html`](https://www.pediy.com/kssd/pediy10/74166.html)

标 题: **【******看雪论坛.腾讯公司 2008 软件安全竞赛**】**第一阶段◇第二题分析及注册机****  
作 者: mstwugui  
  
  
点击确定按钮后程序执行到 403C00，读取 name 和 key 之后跳入 401113  
  
首先是调用 403897 校验 403271 至 40389F 是否有 0xCC INT 3 断点，如果有直接终止当前进程  
  
接下来继续校验从 403271 开始的 0x76E 个字节，如果有代码改动则终止当前进程  
  
接着开始 name 格式校验，长度为 12 的无重复小写字符串（不包含'z'），校验失败则显示"username?"对话框并回到返回函数  
  
下一步是排序 name, 将 name 的每个字节转变为他在原始 name 字符串从小到大排列后的位置  
  
然后校验 key 的格式，长度至少为 53 并且每个字符都在 0x30-0x33 之间，校验失败则显示"key?"对话框并回到返回函数  
  
接着调用 401087 计算根据 key 和排序过的 name 重新计算 name，计算出来的结果为长度为 24 的字符串  
  
如果上一步返回的字符串第 n 个字节不等于排序过的 name 的第 n 个字节（0<=n<12），或不等于第 12+n 个字节，则显示"wrong!"对话框并回到返回函数  
  
否则显示"ok!"  
  
关于 401087 也就是 4022A0 的算法粗看一下有些长，我也没有什么好办法，为了让自己省事一些，先把整个函数用 C 描述出来  
  

引用: CString CalcRealName(char name\[\], char key\[\])  
{  
    CString sRet = "";  
    int        n = 12/key\[0\];  
    int        i, j, offset;  
    int        temp\[12\], bak\[12\];  
    int        value1, value2;  
  
    for (j=0;j<2;j++)  
    {  
        for (offset=0;offset<12;offset++)  
        {  
            for (i=0;i<12;i++)  
                temp\[i\] = 0x1E;  
            temp\[name\[offset\]\] = (j==0)?0x28:0x14;  
            memcpy(bak, temp, sizeof(int)\*12);  
            value1 = 0;  
            value2 = 0;  
            for (i=0;i<4;i++)  
            {  
                value1 += temp\[(key\[1\]\*4)+i\];  
                value2 += temp\[(key\[2\]\*4)+i\];  
            }  
            if (value1 == value2)  
            {  
                if (temp\[key\[3\]\*4+key\[4\]\] == temp\[key\[5\]\*4+key\[6\]\])  
                {  
                    if (temp\[key\[7\]\*4+key\[8\]\] == temp\[key\[9\]\*4+key\[10\]\])  
                        sRet = sRet + 'l';  
                    else  
                        sRet = sRet + 'k';  
                }  
                else  
                {  
                    if (temp\[key\[7\]\*4+key\[8\]\] == temp\[key\[9\]\*4+key\[10\]\])  
                        sRet = sRet + 'j';  
                    else  
                        sRet = sRet + 'i';  
                }  
            }  
            else if (value1 > value2)  
            {  
                temp\[key\[1\]\*4 + key\[11\]\] = temp\[key\[12\]\*4+key\[13\]\];  
                temp\[key\[1\]\*4 + key\[14\]\] = temp\[key\[15\]\*4+key\[16\]\];  
                temp\[key\[1\]\*4 + key\[17\]\] = temp\[key\[18\]\*4+key\[19\]\];  
                temp\[key\[2\]\*4 + key\[20\]\] = temp\[key\[21\]\*4+key\[22\]\];  
                temp\[key\[2\]\*4 + key\[23\]\] = temp\[key\[24\]\*4+key\[25\]\];  
                temp\[key\[2\]\*4 + key\[26\]\] = temp\[key\[27\]\*4+key\[28\]\];  
  
                value1 = 0;  
                value2 = 0;  
                for (i=0;i<4;i++)  
                {  
                    value1 += temp\[(key\[1\]\*4)+i\];  
                    value2 += temp\[(key\[2\]\*4)+i\];  
                }  
                if (value1 > value2)  
                {  
                    if (temp\[key\[29\]\*4+key\[30\]\] == temp\[key\[31\]\*4+key\[32\]\])  
                        sRet = sRet + 'e';  
                    else  
                        sRet = sRet + 'a';  
                }  
                else if (value1 < value2)  
                {  
                    if (temp\[key\[33\]\*4+key\[34\]\] > temp\[key\[35\]\*4+key\[36\]\])  
                        sRet = sRet + 'g';  
                    else if (temp\[key\[33\]\*4+key\[34\]\] < temp\[key\[35\]\*4+key\[36\]\])  
                        sRet = sRet + 'f';  
                    else if (temp\[key\[33\]\*4+key\[34\]\] == temp\[key\[35\]\*4+key\[36\]\])  
                        sRet = sRet + 'h';  
                }  
                else if (value1 == value2)  
                {  
                    memcpy(temp, bak, sizeof(int)\*12);  
                    if (temp\[key\[37\]\*4+key\[38\]\] > temp\[key\[39\]\*4+key\[40\]\])  
                        sRet = sRet + 'b';  
                    else if (temp\[key\[37\]\*4+key\[38\]\] < temp\[key\[39\]\*4+key\[40\]\])  
                        sRet = sRet + 'c';  
                    else if (temp\[key\[37\]\*4+key\[38\]\] == temp\[key\[39\]\*4+key\[40\]\])  
                        sRet = sRet + 'd';  
                }  
            }  
            else if (value1 < value2)  
            {  
                temp\[key\[1\]\*4 + key\[11\]\] = temp\[key\[12\]\*4+key\[13\]\];  
                temp\[key\[1\]\*4 + key\[14\]\] = temp\[key\[15\]\*4+key\[16\]\];  
                temp\[key\[1\]\*4 + key\[17\]\] = temp\[key\[18\]\*4+key\[19\]\];  
                temp\[key\[2\]\*4 + key\[20\]\] = temp\[key\[21\]\*4+key\[22\]\];  
                temp\[key\[2\]\*4 + key\[23\]\] = temp\[key\[24\]\*4+key\[25\]\];  
                temp\[key\[2\]\*4 + key\[26\]\] = temp\[key\[27\]\*4+key\[28\]\];  
  
                value1 = 0;  
                value2 = 0;  
                for (i=0;i<4;i++)  
                {  
                    value1 += temp\[(key\[1\]\*4)+i\];  
                    value2 += temp\[(key\[2\]\*4)+i\];  
                }  
                if (value1 > value2)  
                {  
                    if (temp\[key\[41\]\*4+key\[42\]\] > temp\[key\[43\]\*4+key\[44\]\])  
                        sRet = sRet + 'f';  
                    else if (temp\[key\[41\]\*4+key\[42\]\] < temp\[key\[43\]\*4+key\[44\]\])  
                        sRet = sRet + 'g';  
                    if (temp\[key\[41\]\*4+key\[42\]\] < temp\[key\[43\]\*4+key\[44\]\])  
                        sRet = sRet + 'h';  
                }  
                else if (value1 < value2)  
                {  
                    if (temp\[key\[45\]\*4+key\[46\]\] == temp\[key\[47\]\*4+key\[48\]\])  
                        sRet = sRet + 'e';  
                    else  
                        sRet = sRet + 'a';  
                }  
                else if (value1 == value2)  
                {  
                    memcpy(temp, bak, sizeof(int)\*12);  
                    if (temp\[key\[49\]\*4+key\[50\]\] > temp\[key\[51\]\*4+key\[52\]\])  
                        sRet = sRet + 'c';  
                    else if (temp\[key\[49\]\*4+key\[50\]\] < temp\[key\[51\]\*4+key\[52\]\])  
                        sRet = sRet + 'b';  
                    else if (temp\[key\[49\]\*4+key\[50\]\] == temp\[key\[51\]\*4+key\[52\]\])  
                        sRet = sRet + 'd';  
                }  
            }  
        }  
    }  
      
    return sRet;  
} 

现在我们可以开始分析 key 的格式，首先是注意到有一个长度为 12 的局部数组，而且代码中有很多 a\*4+b 的定位操作，因此很显然为了覆盖到每一个字节并且不重叠，key 的第一个字节应该是 3  
  
由于字符"ijkl"的触发条件都是 value1==value2, value1 和 value2 分别是从 key\[1\]\*4 和 key\[2\]\*4 在临时数组中的连续 4 个值的和，因此很显然 key\[1\]和 key\[2\]只能是 0 或 1，并且这两个字节必须不同，考虑到后面的  
  

引用:                 temp\[key\[1\]\*4 + key\[11\]\] = temp\[key\[12\]\*4+key\[13\]\];  
                temp\[key\[1\]\*4 + key\[14\]\] = temp\[key\[15\]\*4+key\[16\]\];  
                temp\[key\[1\]\*4 + key\[17\]\] = temp\[key\[18\]\*4+key\[19\]\];  
                temp\[key\[2\]\*4 + key\[20\]\] = temp\[key\[21\]\*4+key\[22\]\];  
                temp\[key\[2\]\*4 + key\[23\]\] = temp\[key\[24\]\*4+key\[25\]\];  
                temp\[key\[2\]\*4 + key\[26\]\] = temp\[key\[27\]\*4+key\[28\]\];                   

如果 key\[1\]是 1，而 key\[0\]是 0 话，那么在 4<=offset<8 的情况下，temp\[0\]-> temp\[4\]的原始值为 0x1E, temp\[4\]->temp\[7\]当中的 4 个字节有一个不为 0x1E, 此时我们无法确保将 value1 的值转变为小于 value2, 这是因为我们先对 value1 需要用到的内存空间操作，随后才对 value2 需要用到的内存空间操作  
  
因此 key\[1\]应该是 0，key\[2\]应该是\[1\]  
  
接下来根据"ijkl"字符的触发条件，得到  
  

引用:     key\[3\] = 0x02;  
    key\[4\] = rand()%2;  
    key\[5\] = 0x02;  
    key\[6\] = (key\[4\]+1)%2;  
    key\[7\] = 0x02;  
    key\[8\] = 2\*(rand()%2);  
    key\[9\] = 0x02;  
    key\[10\] = (0x02+key\[8\])%4;                   

回过头来我们继续看一下在 value1 不等于 value2 时对临时数组所做的调整  
  

引用:                 temp\[key\[1\]\*4 + key\[11\]\] = temp\[key\[12\]\*4+key\[13\]\];  
                temp\[key\[1\]\*4 + key\[14\]\] = temp\[key\[15\]\*4+key\[16\]\];  
                temp\[key\[1\]\*4 + key\[17\]\] = temp\[key\[18\]\*4+key\[19\]\];  
                temp\[key\[2\]\*4 + key\[20\]\] = temp\[key\[21\]\*4+key\[22\]\];  
                temp\[key\[2\]\*4 + key\[23\]\] = temp\[key\[24\]\*4+key\[25\]\];  
                temp\[key\[2\]\*4 + key\[26\]\] = temp\[key\[27\]\*4+key\[28\]\];                   

考虑到'a'和'e'的触发条件是修改后的 value1==value2，因此此处修改应该只对偏移地址 1,2,3 起作用，所以得到  
  

引用:     key\[11\] = rand()%3;  
    key\[14\] = (key\[11\] + 1 + rand()%2)%3;  
    key\[17\] = (6 - key\[11\] - key\[14\])%3;  
    key\[11\] ++;  
    key\[14\] ++;  
    key\[17\] ++;  
  
    key\[20\] = rand()%3;  
    key\[23\] = (key\[20\] + 1 + rand()%2)%3;  
    key\[26\] = (6 - key\[20\] - key\[23\])%3;  
    key\[20\] ++;  
    key\[23\] ++;  
    key\[26\] ++;                   

这里再进一步考虑 key\[1\]和 key\[2\]的值已经确定，因此得到  
  

引用:     key\[12\] = 0x01;  
    key\[15\] = 0x01;  
    key\[18\] = 0x01;  
  
    key\[13\] = key\[11\];  
    key\[16\] = key\[14\];  
    key\[19\] = key\[17\];  
  
    key\[21\] = 0x02;  
    key\[24\] = 0x02;  
    key\[27\] = 0x02;  
  
    key\[22\] = key\[20\];  
    key\[25\] = key\[23\];  
    key\[28\] = key\[26\];                   

接下来根据 a 和 e 的触发条件得出  
  

引用:     if (rand()%2)  
    {  
        key\[29\] = 0;  
        key\[30\] = 0;  
        key\[32\] = rand()%4;  
        if (key\[32\] == 0)  
            key\[31\] = 2;  
        else  
            key\[31\] = rand()%3;  
    }  
    else  
    {  
        key\[31\] = 0;  
        key\[32\] = 0;  
        key\[30\] = rand()%4;  
        if (key\[30\] == 0)  
            key\[29\] = 2;  
        else  
            key\[29\] = rand()%3;  
    }                   

以及  

引用:     if (rand()%2)  
    {  
        key\[45\] = 0;  
        key\[46\] = 0;  
        key\[48\] = rand()%4;  
        if (key\[48\] == 0)  
            key\[47\] = 2;  
        else  
            key\[47\] = rand()%3;  
    }  
    else  
    {  
        key\[47\] = 0;  
        key\[48\] = 0;  
        key\[46\] = rand()%4;  
        if (key\[46\] == 0)  
            key\[45\] = 2;  
        else  
            key\[45\] = rand()%3;  
    }                   

再根据"bcd"的触发条件得出  
  

引用:     key\[37\] = 0;  
    key\[38\] = 1;  
    key\[39\] = 0;  
    key\[40\] = 2;  
  
    key\[49\] = 0;  
    key\[50\] = 1;  
    key\[51\] = 0;  
    key\[52\] = 2; 

最后根据"fgh"的触发条件得出  

引用:     key\[33\] = 0;  
    key\[34\] = 1;  
    key\[35\] = 0;  
    key\[36\] = 2;  
  
    key\[41\] = 0;  
    key\[42\] = 1;  
    key\[43\] = 0;  
    key\[44\] = 2;                   

呵呵，都要结束了还没有谈到 netwind 大侠的 h 圈套，这个 h 确实看起来很像是个 bug，但其实是有解的，只要我们把 name 中的第 8 大的字节放到第 7 大的字节之后这个问题也就迎刃而解了，仔细看一下源代码  
  

引用: .text:00402C59 mov     edx, \[eax+edx\*4\]  
.text:00402C5C cmp     edx, \[ecx+esi\*4\]  
.text:00402C5F jle     short loc\_402C73  
.text:00402C61  
.text:00402C61 char\_f\_0:  
.text:00402C61 mov     eax, \[ebp+arg\_realname\]  
......  
.text:00402CAB mov     eax, \[ecx+eax\*4\]  
.text:00402CAE cmp     eax, \[edx+esi\*4\]  
.text:00402CB1 jge     short loc\_402CC5  
.text:00402CB3  
.text:00402CB3 char\_g\_0:  
.text:00402CB3 mov     ecx, \[ebp+arg\_realname\]  
......  
.text:00402CFD mov     ecx, \[edx+ecx\*4\]  
.text:00402D00 cmp     ecx, \[eax+esi\*4\]  
.text:00402D03 jge     short loc\_402D17  
.text:00402D05  
.text:00402D05 char\_h\_0:  
.text:00402D05 mov     edx, \[ebp+arg\_realname\]                   

这里其实有两次输出操作，if...else 之后还有一个单独的 if，因此这个 h 放在 g 后面就一帆风顺的通过了  
  
从整个分析可以看出，name 和 key 其实是独立的，并没有互相依赖，也就是说可以随意选取各一个合法的 name 和 key 都能得到 ok!

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || pediykg102008.rar  
 |