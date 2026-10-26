
# QQ聊天记录文件解密方式

QQ聊天记录文件解密方式

by nbw

4 NE365

本文纯属虚构，如有雷同纯属巧合。

昨天半夜被人拉起来弄了个QQ聊天记录查看器，可以查看本地QQ聊天纪录，这个东西不是修改QQ的本地登陆，而是直接解析本地聊天纪录文件，以前也见过，感觉大哥比较有含量。

今天看了一下，主要看一下QQ本地聊天纪录文件的格式。当然本文主要说数据解密方法，真正的文件解析还比较麻烦。

这里分析的QQ聊天记录读取器5.1，为了不让更多的人悲伤所以大家也别那这个软件破来破去了。我也不会破，我很单纯的。谁要是愿意继续分析建议直接分析QQ自带的聊天纪录察看器，有牛人给修改bug，分析起来应该也爽一些。汉

假设我们的`MsgEx.db`内有下面数据：

`data[16] = 0x61 12 75 20 7A 8B 74 C6 05 9D 77 65 98 E3 73 B1 ; a.u z???e?s`

共4个`DWORD`，其中后面2个`DWORD`作为密钥，前面2个`DWORD`是密文，用密钥计算出来一个key，该key与前面的2个`DWORD`异或便得到明文。

下面看看怎么处理上面的`data[16]`数组。

## 1. 初次处理

输入：

`05 9D 77 65 98 E3 73 B1`

处理：
```
00D43E50           8A09          mov cl,byte ptr ds:[ecx]
00D43E52           6A 08         push 8
00D43E54           3008          xor byte ptr ds:[eax],cl
00D43E56           42            inc edx
00D43E57           5B            pop ebx
00D43E58           3BD3          cmp edx,ebx
00D43E5A         ^ 7C DF         jl short 00D43E3B
```
得到结果：

`45 9C A6 1C 5C 43 8B 3A`

## 2. 计算密钥

```
00D43816           6A 10         push 10
00D43818           B8 909B77E3   mov eax,E3779B90                        ;这里初始化一个密钥常量
00D4381D           59            pop ecx                                                        ; ecx = 0x10 ，16轮运算
```
下面是循环，初始时候：

`ebx = 0x459CA61C`

`edi = 0x5C438B3A`

就是上面的那个结果，也就是输入数据

密钥key，共32位，4个`DWORD`，也就是循环中的`[ebp-x]`：

`0012F248  C7 B3 FE C2 A5 6D C6 A1 CE E6 22 24 DF 2F BE 2D  浅?ロ啤捂"$??`
```
00D4381E           8BD3          mov edx,ebx
00D43820           8BF3          mov esi,ebx
00D43822           C1EA 05       shr edx,5
00D43825           0355 FC       add edx,dword ptr ss:[ebp-4]
00D43828           C1E6 04       shl esi,4
00D4382B           0375 F8       add esi,dword ptr ss:[ebp-8]
00D4382E           33D6          xor edx,esi
00D43830           8D3418        lea esi,dword ptr ds:[eax+ebx]
00D43833           33D6          xor edx,esi
00D43835           2BFA          sub edi,edx
00D43837           8BD7          mov edx,edi
00D43839           8BF7          mov esi,edi
00D4383B           C1EA 05       shr edx,5
00D4383E           0355 F4       add edx,dword ptr ss:[ebp-C]
00D43841           C1E6 04       shl esi,4
00D43844           0375 F0       add esi,dword ptr ss:[ebp-10]
00D43847           33D6          xor edx,esi
00D43849           8D3438        lea esi,dword ptr ds:[eax+edi]
00D4384C           33D6          xor edx,esi
00D4384E           05 4786C861   add eax,61C88647
00D43853           2BDA          sub ebx,edx
00D43855           49            dec ecx
00D43856         ^ 75 C6         jnz short 00D4381E
```
得到输出：

`addr0[] = 0x0F 7B 55 00 77 81 54 E6`

这个16轮运算很简单，可以google -> 本论坛 -> `blowfish` 斑竹的某回贴就弄到代码了。

## 3. 解密

输入(就是最开始的16位)：

`addr1[] = 0x61 12 75 20 7A 8B 74 C6`
```
00D43E11           8A0C08        mov cl,byte ptr ds:[eax+ecx]                                ;获得addr1[]
00D43E14           324C05 E4     xor cl,byte ptr ss:[ebp+eax-1C]                ;这里[ebp+eax-1C]指向上面得到的密钥addr0
00D43E18           FF45 14       inc dword ptr ss:[ebp+14]
00D43E1B           40            inc eax
00D43E1C           FF4D 18       dec dword ptr ss:[ebp+18]
00D43E1F           880A          mov byte ptr ds:[edx],cl                                                ;填写输出
```
得到输出：

`00D41C49  6E 69 20 20 0D 0A 20 20                          ni  ..`

然后进入下一个循环。下一轮解密还是处理4个`DWORD`，不过并不是继续从下面取，而是从上面的`data[8]`开始算。

就是说文件里不存在专门用来作为密钥的冗余数据，不过也没看到压缩，汉。

文中算法连接：

`http://bbs.pediy.com//showthread.php?threadid=862&goto=nextoldest`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
