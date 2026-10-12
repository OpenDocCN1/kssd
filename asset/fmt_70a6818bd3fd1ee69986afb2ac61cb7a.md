
### 关键字符串合成逻辑

首先通过循环处理 `num2` 数组，代码示例如下：

```
int num2[32]={0};
int num2len=0;
for(int i=0;i<len;i++){
    int tmp=i;
    tmp=tmp & 0x80000001;
    if(tmp!=0){
        tmp=(i+1) % 0xFF;
        if(tmp!=0){
            num2[num2len]=tmp;
            num2len++;
        }
    }
}
```

接下来是对关键字符串的汇编层面合成，涉及对 `esp` 寄存器的操作及调用 `UILib8_M` 模块：

```
0046D566        |.  52                    push edx
0046D567        |.  8D4424 2C             lea eax,dword ptr ss:[esp+2C]
0046D56B        |.  50                    push eax
0046D56C        |.  8D4C24 1C             lea ecx,dword ptr ss:[esp+1C]
0046D570        |.  51                    push ecx
0046D571        |.  E8 6AF8FFFF           call UILib8_M.0046CDE0                  ;  合成字符
0046D576        |.  83C4 18               add esp,18
0046D579        |.  50                    push eax
0046D57A        |.  8D4C24 10             lea ecx,dword ptr ss:[esp+10]
0046D57E        |.  C68424 CC000000 1>mov byte ptr ss:[esp+CC],11
0046D586        |.  FF15 34E34A00         call dword ptr ds:[<&MFC71U.#903>]     ;  MFC71U.7C29B383
0046D58C        |.  8D4C24 08             lea ecx,dword ptr ss:[esp+8]
0046D590        |.  889C24 C8000000       mov byte ptr ss:[esp+C8],bl
0046D597        |.  FF15 1CE54A00         call dword ptr ds:[<&MFC71U.#578>]     ;  MFC71U.7C2868E9
```

通过观察与分析，首先是将上面两个大循环得到的结果，与软件名位错位相连，然后嫁接两个 `0x30`。再把假码的前 20 位与软件名连接到上面。首位不变，是 `0x31`。

C++ 代码实现如下：

```
strCode[0]=0x31;
int k=1,i=0,j=0;
while(j<num1len){
    strCode[k]=sName;
    i+=2;
    strCode[k+1]=num1[j];
    j++;
    k+=2;
}

j=0,i=1;
while(j<num2len){
    strCode[k]=sName;
    i+=2;
    strCode[k+1]=num2[j];
    j++;
    k+=2;
}

strCode[k]=0x30;
k++;
strCode[k]=0x30;
k++;

TCHAR szConst[]="11111A22222B33333C44";
for(int i=0;i<20;i++,k++)
    strCode[k]=szConst;

for(int i=0;i<len;i++,k++)
    strCode[k]=sName;
```

观察数据窗口中的内存分布：

```
03FC4018  31 49 01 54 03 4F 05 76 07 6D 09 65 0B 63 0D 6E  1ITOvm.e c.n
03FC4028  0F 65 11 74 13 72 15 6D 02 4F 04 61 06 69 08 70  etrmOaip
03FC4038  0A 67 0C 6F 0E 76 10 72 12 65 14 35 16 30 30 31  .g.ovre5001
03FC4048  31 31 31 31 41 32 32 32 32 32 42 33 33 33 33 33  1111A22222B33333
03FC4058  43 34 34 49 6D 54 4F 4F 61 76 69 6D 70 65 67 63  C44ImTOOavimpegc
03FC4068  6F 6E 76 65 72 74 65 72 35                       onverter5
```

### MD5 计算与后续处理

然后对上面的字符串进行 `MD5` 计算，汇编调用如下：

```
0046D5A1        |.  52                    push edx                              ; /Arg1
0046D5A2        |.  8D8C24 84000000       lea ecx,dword ptr ss:[esp+84]         ; |
0046D5A9        |.  E8 12E40200           call UILib8_M.0049B9C0               ; \UILib8_M.0049B9C0
```

接下来，继续对得到的 `MD5` 字符串进行处理，涉及位运算和循环逻辑：

```
0046D6B4        |>  /56                    /push esi
0046D6B5        |. |8D4C24 34              |lea ecx,dword ptr ss:[esp+34]
0046D6B9        |. |FF15 7CE34A00          |call dword ptr ds:[<&MFC71U.#861>] ;  MFC71U.7C29986D
0046D6BF        |. |8D4C24 0C              |lea ecx,dword ptr ss:[esp+C]
0046D6C3        |. |50                    |push eax
0046D6C4        |. |FF15 44E34A00         |call dword ptr ds:[<&MFC71U.#904>] ;  MFC71U.7C29B289
0046D6CA        |. |8BCE                   |mov ecx,esi
0046D6CC        |. |D1E9                   |shr ecx,1                           ;  <<1，将最低位移到 CF 中
0046D6CE        |. |41                     |inc ecx
0046D6CF        |. |81E1 03000080          |and ecx,80000003
0046D6D5        |. |79 05                  |jns short UILib8_M.0046D6DC
0046D6D7        |. |49                     |dec ecx
0046D6D8        |. |83C9 FC                |or ecx,FFFFFFFC
0046D6DB        |. |41                     |inc ecx
0046D6DC        |>  |75 0F                  |jnz short UILib8_M.0046D6ED
0046D6DE        |.  |68 38F94A00            |push UILib8_M.004AF938
0046D6E3        |.  |8D4C24 10             |lea ecx,dword ptr ss:[esp+10]
0046D6E7        |.  |FF15 60E34A00         |call dword ptr ds:[<&MFC71U.#907>] ;  MFC71U.7C29B867
0046D6ED        |>  |83C6 02               |add esi,2
0046D6F0        |.  |83FE 20               |cmp esi,20
0046D6F3        |.^\\7C BF                 \\jl short UILib8_M.0046D6B4
0046D6F5        |.   8D4C24 0C             lea ecx,dword ptr ss:[esp+C]          ;  对 MD5 值的字符串进行处理
0046D6F9        |.   FF15 74E34A00         call dword ptr ds:[<&MFC71U.#4079>] ;  隔位取，4 位一组，以－相连
```

移除末尾多余的连接符：

```
0046D706        |.  6A 01                 push 1
0046D708        |.  48                    dec eax
0046D709        |.  50                    push eax
0046D70A        |.  8D4C24 14             lea ecx,dword ptr ss:[esp+14]         ;  去掉最后多出的一个“－”
```

### 最终比对与标志位设置

将处理得到的字符串嫁接到假码的前 20 位后面，与假码进行比较，然后就开始设置标志位了。

```
0046D718        |.  50                    push eax
0046D719        |.  6A 00                 push 0
0046D71B        |.  8D4C24 14             lea ecx,dword ptr ss:[esp+14]
0046D71F        |.  FF15 78E34A00         call dword ptr ds:[<&MFC71U.#3844>]  ;  MFC71U.7C299F0A
0046D725        |.  8B4C24 34             mov ecx,dword ptr ss:[esp+34]         ;  将上面提所得字符串转大写
0046D729        |.  51                    push ecx                              ;  与假码的前 20 位相连
0046D72A        |.  8D4C24 10             lea ecx,dword ptr ss:[esp+10]          ;  然后与假码比较
0046D72E        |.  FF15 54E34A00         call dword ptr ds:[<&MFC71U.#1473>]  ;  MFC71U.7C29927C
0046D734        |.  F7D8                  neg eax
0046D736        |.  1AC0                  sbb al,al
0046D738        |.  FEC0                  inc al
```

完整的 `VC` 代码如下（这里将前 20 位固定下来了，没有模拟随机生成！）：
