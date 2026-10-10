

# 第一阶段第一题分析+完整逆向代码(看雪金山 2007 逆向分析挑战赛)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-248.htm`](https://www.pediy.com/kssd/pediy09/pediy09-248.htm)

> 第一阶段第一题分析+完整逆向代码(看雪金山 2007 逆向分析挑战赛)  
> by aker  
> 8:31 2007-8-24  
>   
> 第一题已经结束了,放出分析和逆向代码先;)  
> 我很少做 crackme 分析，第一次写逆向分析文章，希望没有什么错误。给了个还原出来的 crackme 的代码，见附件，样子，行为和原来的一模一样;)  
>   
> od 载入，没有什么说的，总共才 1.6k 的程序，载入就看到下面接受输入的代码，检查名字和序列号是否为空，空则重新接受输入  
>   
> 
> 代码:
> 
> * * *
> 
> `00400499   |.  8D45 EC        lea eax,dword ptr ss:[ebp-14]        ;  name   0040049C   |.  6A 10          push 10                              ; /Count = 10 (16.)   0040049E   |.  50             push eax                             ; |Buffer   0040049F   |.  68 E9030000    push 3E9                             ; |ControlID = 3E9 (1001.)   004004A4   |.  FF75 08        push dword ptr ss:[ebp+8]            ; |hWnd   004004A7   |.  FFD6           call esi                             ; \GetDlgItemTextA   004004A9   |.  85C0           test eax,eax         004004AB   |.  74 6C          je short CrackMe.00400519   004004AD   |.  8D85 E8EFFFFF  lea eax,dword ptr ss:[ebp-1018]      ;  serial   004004B3   |.  68 00100000    push 1000                            ; /Count = 1000 (4096.)   004004B8   |.  50             push eax                             ; |Buffer   004004B9   |.  68 EA030000    push 3EA                             ; |ControlID = 3EA (1002.)   004004BE   |.  FF75 08        push dword ptr ss:[ebp+8]            ; |hWnd   004004C1   |.  FFD6           call esi                             ; \GetDlgItemTextA   004004C3   |.  85C0           test eax,eax   004004C5   |.  74 52          je short CrackMe.00400519`
> 
> * * *
> 
> 下面代码计算用户名字符数，用的 repne scas 指令，ecx 保存字符数，我们还原代码时要不用 strlen，要不保存 GetDlgItemTextA 的返回值都可以达到该效果。  
> 代码:
> 
> * * *
> 
> `004004C7   |.  8D7D EC        lea edi,dword ptr ss:[ebp-14]        ;  name   004004CA   |.  83C9 FF        or ecx,FFFFFFFF                      ;  ecx = ffffffff   004004CD   |.  33C0           xor eax,eax                          ;  eax = 0   004004CF   |.  33D2           xor edx,edx                          ;  edx = 0   004004D1   |.  F2:AE          repne scas byte ptr es:[edi]   004004D3   |.  F7D1           not ecx   004004D5   |.  49             dec ecx   004004D6   |.  85C9           test ecx,ecx                         ;  计数用户名字符数   004004D8   |.  7E 23          jle short CrackMe.004004FD`
> 
> * * *
> 
> 下面 004004DA 代码计算用户名特征码，这个后面有用，注意 ebx 在前面 0040046F 处已经初始化了一个常数，发现一个未知寄存器一定要往上找，看什么地方修改过。  
> 代码:
> 
> * * *
> 
> `0040046F   |.  BB 68245713    mov ebx,13572468                     ;  ebx = 13572468`
> 
> * * *
> 
> 代码:
> 
> * * *
> 
> `004004DA   |> /0FBE4415 EC    /movsx eax,byte ptr ss:[ebp+edx-14]  ;  对用户名每个字符操作   004004DF   |. |03C3           |add eax,ebx   004004E1   |. |69C0 73127203  |imul eax,eax,3721273   004004E7   |. |05 57136824    |add eax,24681357   004004EC   |. |8BF0           |mov esi,eax   004004EE   |. |C1E6 19        |shl esi,19   004004F1   |. |C1F8 07        |sar eax,7   004004F4   |. |0BF0           |or esi,eax   004004F6   |. |42             |inc edx                             ;  edx -- i   004004F7   |. |3BD1           |cmp edx,ecx   004004F9   |. |8BDE           |mov ebx,esi   004004FB   |.^\7C DD          \jl short CrackMe.004004DA`
> 
> * * *
> 
> 代码还原 c 如下，其中 name 为接受输入的用户名，对等起来看的话，addr 相当于开始的 eax，adder2 相当于 esi，ebx 保存 namecalc，这个值以后用。  
> 代码:
> 
> * * *
> 
> `    for(i=0; i<namelen; i++)       {   // 计算名字特征           adder = (name[i]+ namecalc)*0x3721273+0x24681357;           adder2 = adder<<0x19;           __asm sar adder,7           namecalc = adder2|adder;       }`
> 
> * * *
> 
> 好，上面计算出了输入用户名的特征码，底下就到了第一个关键函数调用。该调用将用户名特征作为第一个参数，序列号数组做为第二个参数。  
> 代码:
> 
> * * *
> 
> `004004FD   |> \8D85 E8EFFFFF  lea eax,dword ptr ss:[ebp-1018]      ;  serial   00400503   |.  50             push eax   00400504   |.  53             push ebx                             ;  ebx   00400505   |.  E8 C2FDFFFF    call CrackMe.004002CC`
> 
> * * *
> 
> 跟进去一看，这个函数好长，开始是老规矩，压栈，申请空间，数据初始化。  
> 首先找找，什么地方对我刚刚调用的参数操作了，因为我传进来的参数才是和用户名，序列号相关的，也就是 ebp+8，和 ebp+c，分别发现两个。  
>   
> 代码:
> 
> * * *
> 
> `00400306   |.  8B7D 0C        mov edi,dword ptr ss:[ebp+C]         ;  edi == 输入的序列号      00400367   |> /8B45 08        /mov eax,dword ptr ss:[ebp+8]        ;  namecalc      00400383   |> /8B45 0C        /mov eax,dword ptr ss:[ebp+C]        ;  eax = &serial[0]      004003A4   |.  8B45 08        |mov eax,dword ptr ss:[ebp+8]        ;  eax = namecalc`
> 
> * * *
> 
> 分析一下第一个地方 00400306 是计算序列号长度，最后 ecx = 序列号长度，还原该处和上面说的一样，我们不需要这样做了。另外该处将 ebx 置 1，这个 ebx 以后都不会动了，就把他当作 1 看。  
>   
> 第二个地方是个小关键，根据名字特征值 namecalc 即\[ebp+8\]的 1-8 位构造名字表  
>   
> 代码:
> 
> * * *
> 
> `//汇编代码如下：   00400365   |.  8BFB           mov edi,ebx                          ;  edi = 1   00400367   |>  8B45 08        /mov eax,dword ptr ss:[ebp+8]        ;  namecalc   0040036A   |.  8BCF           |mov ecx,edi   0040036C   |.  D3E8           |shr eax,cl                          ;  namecalc >>= i   0040036E   |.  22C3           |and al,bl                           ;  (byte)namecalc &= 1   00400370   |.  88443D DC      |mov byte ptr ss:[ebp+edi-24],al   00400374   |.  47             |inc edi   00400375   |.  83FF 09        |cmp edi,9   00400378   |.^ 7C ED          \jl short CrackMe.00400367   0040037A   |.  33FF           xor edi,edi                          ;  edi = 0   0040037C   |.  885D E5        mov byte ptr ss:[ebp-1B],bl          ;  bl == 1          // 还原 c 代码如下       for (i=1; i<9; i++)       {   //根据名字特征值的 1-8 位构造名字表           nametable[i] = (unsigned char)((unsigned char)(namecalc>>i)&1);       }       nametable[9] = 1;       `
> 
> * * *
> 
> 第三和第四在一个大段里面，其他地方没有了，不用说，肯定就是这儿判断是否序列号为真。这一段比较长，我们先放一下，看看到底是什么地方判断成功的。下翻看到一个 msgbox，看看就这一个地方调用 msgbox 的，那肯定是他跳出判断的，再看到 0040041B 处文本是个 eax，而 eax 指向\[ebp-128\]，所以\[ebp-128\]存放判断成功失败的文本。看代码发现和\[ebp-128\]相关的地方在这一段有 4 个，开始置 0，刚刚 msgbox 要取数据，另外两个分别在 004003FC，0040042B，可以看到最后都是作为 call CrackMe.00400240 的第二个参数，而第一个参数不一样，但是第一个参数都是一个 11 字节的数组，而且第一个字节为 FF，只是做检查用的。  
>   
> 代码:
> 
> * * *
> 
> `0040040D   |.  8D85 D8FEFFFF  lea eax,dword ptr ss:[ebp-128]       ;  此处为判断成功失败的文本   00400413   |.  59             pop ecx   00400414   |.  6A 00          push 0                               ; /Style = MB_OK|MB_APPLMODAL   00400416   |.  68 70054000    push CrackMe.00400570                ; |Title = ""   0040041B   |.  50             push eax                             ; |Text   0040041C   |.  6A 00          push 0                               ; |hOwner = NULL   0040041E   |.  FF15 34024000  call dword ptr ds:[<&USER32.MessageB>; \MessageBoxA      //[ebp-128] 相关 1，判断成功   004003FC   |.  8D85 D8FEFFFF  lea eax,dword ptr ss:[ebp-128]       ;  该处返回 OK!!   00400402   |.  50             push eax   00400403   |.  8D45 E8        lea eax,dword ptr ss:[ebp-18]        ;  成功   00400406   |>  50             push eax   00400407   |.  E8 34FEFFFF    call CrackMe.00400240      //[ebp-128] 相关 2，判断失败   0040042B   |> \8D85 D8FEFFFF  lea eax,dword ptr ss:[ebp-128]       ;  不是数字,或者其他什么都是失败 Fail!   00400431   |.  50             push eax                             ;  第二个参数   00400432   |.  8D45 F4        lea eax,dword ptr ss:[ebp-C]         ;  第一个参数   00400435   \.^ EB CF          jmp short CrackMe.00400406`
> 
> * * *
> 
> 跟到 call CrackMe.00400240 里面看了下，很简单，就是异或还原数据。此处可以不看，只要知道数据传送进去，会异或出 fail!和 OK!!字样的字符串就好了。  
> 代码:
> 
> * * *
> 
> `   //c 还原代码   void calc_alpha(char *xor, char *result_str)   {       int i;       unsigned char xor0[10] = {0x25,0x9a,0xf3,0x6f,0x82,0xda,0x72,0xfe,0xc9,0xb7};       for(i=0; i<10; i++)  result_str[i] = xor0[i]^xor[i];   }   //下面是调用者准备好的数据。       char result_str[0x40];       unsigned char xorfail[10] = {0x63,0xfb,0x9a,0x03,0xa3,0xda,0x72,0xfe,0xc9,0xb7};  //fail!       unsigned char xorok[10] = {0x6a,0xd1,0xd2,0x4e,0x82,0xda,0x72,0xfe,0xc9,0xb7};  //OK!!      //调用 calc_alpha(xorfail,result_str);会得到 fail!字样，calc_alpha(xorok,result_str);会得到 OK!!      ////注意此处代码没有还原，该处检查标志 FF   00400240   /$  57             push edi   00400241   |.  8B7C24 08      mov edi,dword ptr ss:[esp+8]         ;  edi = 第一个参数，一已定义数组   00400245   |.  803F FF        cmp byte ptr ds:[edi],0FF            ;  测试 0012EB30 是否为 0xff   00400248   |.  75 5F          jnz short CrackMe.004002A9`
> 
> * * *
> 
> 好了，上面说了一大通嗦的话，下面到了真正关键的代码了，也就是上面说到的第三和第四在一个大段里面，肯定就是这儿判断是否序列号为真。该段是个循环，判断序列号\[ebp+C\]是否为真。  
> 循环操作如下：  
> 1.依次加载序列号的每一个字节。  
> 2.判断是否为数字字符 if(serial\[i\]<'0' || serial\[i\]>'9') ，不是数字字符就跳转。  
> 3.循环计算 32 位整数就是刚才计算的用户名特征码的 1/2 模 10 的余数，每次都多计算其 1/2，所以可以看到这个数是以 32 位为循环的。  
> 4.该数字加上刚刚的数字字符串代表的数字再模 10，以这个余数为索引到开始计算的名字特征值的名字表中作变换。  
> 5.下面是变换过程  
> 5.1 如果为 1，则一号位异或 1，见 004003BE，cmp edx,ebx  
> 5.2 否则异或该位置，但是需要该位置的前一个位置为 1，再前面所有数据为 0,不满足就报错  
> 关键代码就这些  
> 代码:
> 
> * * *
> 
> `00400383   |> /8B45 0C        /mov eax,dword ptr ss:[ebp+C]        ;  eax = &serial[0]   00400386   |. |8A0407         |mov al,byte ptr ds:[edi+eax]        ;     00400389   |. |3C 30          |cmp al,30                           ;  if(serial[i]<'0' || serial[i]>'9') goto fail;   0040038B   |. |8845 FF        |mov byte ptr ss:[ebp-1],al          ;     0040038E   |. |0F8C 97000000  |jl CrackMe.0040042B   00400394   |. |3C 39          |cmp al,39   00400396   |. |0F8F 8F000000  |jg CrackMe.0040042B                 ;  如果不是数字跳转到该处，一定要是数字，否则 fail   0040039C   |. |8BC7           |mov eax,edi                         ;     0040039E   |. |6A 1F          |push 1F   004003A0   |. |99             |cdq   004003A1   |. |59             |pop ecx                             ;     004003A2   |. |F7F9           |idiv ecx                            ;  i/0x1f   004003A4   |. |8B45 08        |mov eax,dword ptr ss:[ebp+8]        ;  eax = namecalc   004003A7   |. |6A 0A          |push 0A   004003A9   |. |8BCA           |mov ecx,edx                         ;  ecx = 余数   004003AB   |. |33D2           |xor edx,edx                         ;     004003AD   |. |D3E8           |shr eax,cl                          ;     004003AF   |. |59             |pop ecx                             ;     004003B0   |. |F7F1           |div ecx                             ;  remainder = (namecalc >>= (byte)ecx)%0xa   004003B2   |. |0FBE45 FF      |movsx eax,byte ptr ss:[ebp-1]       ;     004003B6   |. |8D4402 D0      |lea eax,dword ptr ds:[edx+eax-30]   ;  eax = serial[i]数字值+余数   004003BA   |. |33D2           |xor edx,edx                         ;     004003BC   |. |F7F1           |div ecx                             ;     004003BE   |. |3BD3           |cmp edx,ebx                         ;  ebx 一直为 1   004003C0   |. |75 05          |jnz short CrackMe.004003C7          ;  如果 remainder != 1 则跳转   004003C2   |. |305D DD        |xor byte ptr ss:[ebp-23],bl         ;     004003C5   |. |EB 22          |jmp short CrackMe.004003E9   004003C7   |> |385C15 DB      |cmp byte ptr ss:[ebp+edx-25],bl     ;  nametable[remainder-1]!=1>fail!   004003CB   |. |75 5E          |jnz short CrackMe.0040042B          ;  fail!   004003CD   |. |8D42 FE        |lea eax,dword ptr ds:[edx-2]        ;  eax = remainder-2; //calc   004003D0   |. |8BCB           |mov ecx,ebx                         ;  ecx = 1;   004003D2   |. |3BC3           |cmp eax,ebx                         ;  calc >= 1   004003D4   |. |7C 0B          |jl short CrackMe.004003E1   004003D6   |> |385C0D DC      |/cmp byte ptr ss:[ebp+ecx-24],bl   004003DA   |. |74 4F          ||je short CrackMe.0040042B          ;  fail!   004003DC   |. |41             ||inc ecx   004003DD   |. |3BC8           ||cmp ecx,eax   004003DF   |.^|7E F5          |\jle short CrackMe.004003D6   004003E1   |> |305C15 DC      |xor byte ptr ss:[ebp+edx-24],bl     ;  nametable[remainder] ^= 1;   004003E5   |. |8D4415 DC      |lea eax,dword ptr ss:[ebp+edx-24]   ;  多余操作   004003E9   |> |47             |inc edi   004003EA   |. |3BFE           |cmp edi,esi   004003EC   |.^\7C 95          \jl short CrackMe.00400383`
> 
> * * *
> 
> 逆向出来的 c 代码如下  
> 代码:
> 
> * * *
> 
> `    // check       for (i=0; i<seriallen; i++)       {         if(serial[i]<'0' || serial[i]>'9') goto failinput;            remainder = (serial[i]-0x30+ ((namecalc>>(unsigned char)(i%(TABLE_SIZE-1)))%10) )%10;//remainder 在 0-9 之间                   if(remainder != 1)         {           if(nametable[remainder-1]==1)            {              int calc = remainder-2;             while(calc >=1)                 if(nametable[calc--]==1) goto fail;                    nametable[remainder]^=1;            }              else goto fail;         }         else nametable[1] ^=1;       }`
> 
> * * *
> 
> 最后一个判断是，判断特征表中所有数字都为 0，比较简单。  
> 代码:
> 
> * * *
> 
> `004003EE   |> \8BC3           mov eax,ebx                          ;  ebx = 1   004003F0   |>  385C05 DC      /cmp byte ptr ss:[ebp+eax-24],bl     ;  此时需要 table 中全为 0   004003F4   |.  74 35          |je short CrackMe.0040042B           ;  fail!   004003F6   |.  40             |inc eax   004003F7   |.  83F8 0A        |cmp eax,0A   004003FA   |.^ 7C F4          \jl short CrackMe.004003F0      // c 代码       for (i=1; i<10; i++)           if(nametable[i]==1) goto failedcheck;`
> 
> * * *
> 
> 最后给出所有的逆向出来的检查用户名和序列号的 c 代码,没有什么需要注释的,名字都很清楚了,窗体部分见附件。至此，整个代码的逆向过程都出来了。  
> 过会儿给出写注册机的分析，这个咚咚写注册机有些麻烦，而且不太说的清楚。各位要不先自己试着考虑下象这样的过程该怎么写注册机。昨天我看了半天才想出来怎么写注册机,而且开始走了很大的弯路.  
>   
> 代码:
> 
> * * *
> 
> `   #define TABLE_SIZE      32   #define SERIAL_SIZE     0x1000   char            name[0x10];   char            serial[SERIAL_SIZE];   unsigned char   nametable[10];         int namelen = 0, seriallen = 0;      void calc_alpha(unsigned char *xor, char *result_str)   {// 其实这个地方就是为了好玩才逆向的;)不需要，可以直接 sprintf();       int i;       unsigned char xor0[10] = {0x25,0x9a,0xf3,0x6f,0x82,0xda,0x72,0xfe,0xc9,0xb7};       for(i=0; i<10; i++)  result_str[i] = xor0[i]^xor[i];   }   char buf[0x20];   int checkserial( char *name,char *serial)   {       int i;       int remainder;       unsigned namecalc = 0x13572468;       unsigned adder,adder2;       unsigned char xorfail[10] = {0x63,0xfb,0x9a,0x03,0xa3,0xda,0x72,0xfe,0xc9,0xb7};  //fail!       unsigned char xorok[10] = {0x6a,0xd1,0xd2,0x4e,0x82,0xda,0x72,0xfe,0xc9,0xb7};  //OK!!          memset(nametable,0,0xa);       //namelen = strlen(name);       //seriallen= strlen(serial);// 我采用 GetDlgItemText 的返回值计算的。          for(i=0; i<namelen; i++)       {   // 计算名字特征           adder = (name[i]+ namecalc)*0x3721273+0x24681357;           adder2 = adder<<0x19;           __asm sar adder,7           namecalc = adder2|adder;       }          for (i=1; i<9; i++)           nametable[i] = (unsigned char)((unsigned char)(namecalc>>i)&1);       nametable[9] = 1;              // check       for (i=0; i<seriallen; i++)       {         if(serial[i]<'0' || serial[i]>'9') goto failinput;            remainder = (serial[i]-0x30+ ((namecalc>>(unsigned char)(i%(TABLE_SIZE-1)))%10) )%10;                   if(remainder != 1)         {           if(nametable[remainder-1]==1)            {                int calc = remainder-2;             while(calc >=1)                 if(nametable[calc--]==1) goto fail;                    nametable[remainder]^=1;            }              else goto fail;         }         else nametable[1] ^=1;       }       for (i=1; i<10; i++)           if(nametable[i]==1) goto failedcheck;       goto success;   failinput:   fail:   failedcheck:           calc_alpha(xorfail,buf);        return 1;   success:       calc_alpha(xorok,buf);         return 0;   }`
> 
> * * *
> 
> 稍微总结一下验证过程，循环读入序列号，根据序列号对 nametable 的对应位置取反（其中位置判断是否序列号有效），最后序列号结束后判断是否 nametable 全为 0，否则失败。  
> 另外这个题逆推回去有些不容易. 

> 第一阶段第一题注册机思路+完整注册机代码(看雪金山 2007 逆向分析挑战赛)  
>   
> by aker  
> 9:10 2007-8-24  
>   
> 上一篇中分析了看雪金山 2007 逆向分析挑战赛第一阶段第一题 crackme 的代码，并实际动手逆向实现了该 crackme。  
>   
> 主要函数为 int checkserial( char \*name,char \*serial);该函数接受用户名和序列号，通过一系列变换判断是否变换成功。  
>   
> 本篇中分析如何根据关键代码流程写出注册机，附件是注册机及代码。  
>   
> 关键代码的操作过程如下：  
>   
> 循环操作如下：  
> 1.依次加载序列号的每一个字节。  
> 2.判断是否为数字字符 if(serial\[i\]<'0' || serial\[i\]>'9') ，不是数字字符就跳转。  
> 3.循环计算 32 位整数就是刚才计算的用户名特征码的 1/2 模 10 的余数，每次都多计算其 1/2，所以可以看到这个数是以 32 位为循环的。  
> 4.该数字加上刚刚的数字字符串代表的数字再模 10，以这个余数为索引到开始计算的名字特征值的名字表中作变换。  
> 5.下面是变换过程  
> 5.1 如果为 1，则一号位异或 1，见 004003BE，cmp edx,ebx  
> 5.2 否则异或该位置，但是需要该位置的前一个位置为 1，再前面所有数据为 0,不满足就报错  
>   
> 循环结束后  
> 6 检查名字特征值的名字表中的值，需要都为 0 才表示序列号为真。  
>   
> 实际代码如下：  
> 
> 代码:
> 
> * * *
> 
> `    // check       for (i=0; i<seriallen; i++)       {         if(serial[i]<'0' || serial[i]>'9') goto failinput;            remainder = (serial[i]-0x30+ ((namecalc>>(unsigned char)(i%(TABLE_SIZE-1)))%10) )%10;                   if(remainder != 1)         {           if(nametable[remainder-1]==1)            {                int calc = remainder-2;             while(calc >=1)                 if(nametable[calc--]==1) goto fail;                    nametable[remainder]^=1;            }              else goto fail;         }         else nametable[1] ^=1;       }       for (i=1; i<10; i++)           if(nametable[i]==1) goto failedcheck;       goto success;`
> 
> * * *
> 
> 昨天这个地方我首先走了弯路，也介绍下：  
> 从该验证过程可以看到，每次读入一位，然后该位肯定为'0'-'9'之间的一位数，所以想最简单的就是写一个外壳，对序列号从 1-0x1000 验证，每次'0'-'9'变换就是了,原形如下，当时将 getserial 的返回值作了分类，fail 表示错误，不可接受，failedcheck;表示前面都对，但是最后验证错误，success 表示成功，这样，我就可以区分是不是需要对序列号数字加一。但是实际上却出问题了，序列号怎么都不能最终使 nametable 全为 0，但是做了这么久，又有点不想放弃这个思路。傍晚刚好停了一下电，出去吃了晚饭，回来电来了，网络断了，正好可以不看论坛:)换个思路做做看。  
>   
> 代码:
> 
> * * *
> 
> `char buf[ERRO_SIZE];   int shellgo(unsigned namecalc)   {       int i;       enum result = OTHER;;       while(result)       {           for (i=0; i<10 && result!=FAILEDCHECK && result!=SUCCESS;; i++)           {// 每次检查一位               serial[seriallen-1] = (0x30+i);               result = getserial(namecalc,serial);           }              if(result != SUCCESS;; ) result = OTHER;           seriallen++;           if(seriallen>=0x1000)            {               wsprintf(buf,"exceeds..............\n");               return -1;           }       }       return 0;   }`
> 
> * * *
> 
> 仔细看了代码，发现一个咚咚，就是((namecalc>>(unsigned char)(i%31))%10)得变化对于每个用户名都是一样的循环变化。  
> 也就是说可以拿出来，先生成。还有就是名字特征的 1-8 位的数组都是对每个用户名一样的。  
> 代码:
> 
> * * *
> 
> `// 定义一个 32 位的数组       unsigned char table[32];       for (i=0; i<32; i++)            table[i] = ((namecalc>>(unsigned char)(i%31))%10);   // 另外这个是名字特征的 1-8 位的数组，加上最后一位固定为 1       for (i=1; i<9; i++)           nametable[i] = (unsigned char)((unsigned char)(namecalc>>i)&1);       nametable[9] = 1;`
> 
> * * *
> 
> 现在就容易懂了，每次从 table 中循环选取一个数字，和当前取到的序列号相加，模 10 就可以得到名字特征表的位置。  
> 然后对该位置异或。  
> 但是该位置选取有很多限制：  
> 见步骤 5.1,5.2  
> 5.1 如果为 1，则一号位异或 1，见 004003BE，cmp edx,ebx  
> 5.2 否则异或该位置，但是需要该位置的前一个位置为 1，再前面所有数据为 0,不满足就报错  
>   
> 说简单点就是，如果位置为 1，则 1 号位取反，否则，要保证当前位置前面一位为 1，而且再前面所有数据为 0，都满足就可以对该位取反。  
>   
> 实例：对名字 aker 有  
>             name:      aker  
>             nametable: \*110000111  
>             pos:       0123456789  
>             table:     89942689999426310005789942631528  
>   
> 比如第一个序列号为 3，则，可以知道 (3+8)%10 == 1,所以对第一位取反。  
> nametable 为 \*010000111  
> 如果不为 3，假定为 5，则余数为 3，2 号位为 1，但是 1 号位也为 1，所以报错。  
> 如果不为 3，假定为 4，则余数为 2，1 号位为 1，前面是 0 号，所以也可以进去，但是我们可以发现，如果随便选取一个的话，有可能进入死循环，出现很多序列号前后完全重复的情况。  
>   
>   
> 序列号按照上面的步骤如果位置为 1，则 1 号位取反，否则，要保证当前位置前面一位为 1，而且再前面所有数据为 0，都满足就可以对该位取反。使得最后一个序列号完成的时候，nametable 中全为 0。这样就表示序列号为真。  
>   
> 一些细节：序列号一次和 table 中的数字相加，得到的余数就是在 nametable 中的位置。  
> 好了，我们现在应该清楚如何检验序列号正确性的问题了。  
>   
> 稍微总结一下上面所说的：  
> 上面介绍了序列号如何验证，主要就是通过序列号使对应 nametable 为 0。  
> 变换可以写成下式：  
> nametable\[(serial\[i\]+table\[i%31\])%10\]^=1;而且此时(serial\[i\]+table\[i%31\])%10 要么为 1，要么在最左边的 1 的下一位，持续该变换过程可使 nametable 为 0。  
>   
> 下面就是问题的关键了：  
> 问题是，我们如何通过 table 和 nametable 找 serial 呢？  
> 根据上面的变换过程我们可以知道，每次异或位置只有两个，1，或者是最左边的 1 的右边。  
> 那现在的问题就是判断怎么样选取位置了，估计很多人就卡在这里了:)  
>   
> 其实仔细想一下可以明白，肯定要交互选取，这次 1，下次就是最左边的 1 的右边，为什么呢？  
> 因为如果你不交互选取的话，等于是回退了，比如你这次选了 1，下次还选 1 异或，就变为原来的数字了；或者你这次选了最左边的 1 的右边，那么下次再选，肯定还是这个位置，你再异或不是又回去了不，估计不少看官都恍然大悟了;)  
>   
> 那么，最开始该选 1 好位置还是选最左边的 1 的右边呢？  
> 结论是根据 nametable 中 1 的个数而不同，奇数选 1，偶数选最左边 1 的右边。  
> 这个问题我不能明白的证明给大家看，自己能够理解，但是感觉说不出来，这样吧，我举例子：  
>   
> 比如，最简单的，只有一个 1 的情况，假设 1 在最右边，你说这个时候该怎么选择位置呢？你说白痴才不知道，根据上面的条件，只能选最左边的 1 啊，好，你说对了；再假定 1 在最左边，你说位置该怎么选呢？你说，"当然最左边啦，选 1 号位就直接得结果了，你不急我都要替你急了":P。再假定 1 个 1 在 2-8 的任何位置呢？你说还是一样啊，肯定选最左边的 1，选右边，数字又变大了，什么时候才能变回来啊。要变除非再选哪个位置;)  
> 好了 1 个 1 的情况很清楚了，肯定是选最左边的 1 号位。  
>   
> 再接着说有两个 1 的 nametable，还是从最简单的开始，最左边有两个 1 就不用说了，肯定是第二个 1，这样可以直接消掉，最右边两个 1，肯定还是右边;)如果左边一个 1，右边一个 1 呢？;)反证一下，如果你选了 1 号位，下次你还要选这个，肯定是不行的;)如果任意位两个 1 呢？其实是这样的，如果 1 的个数为偶数的话，左边的总要往右边靠，靠到了，就可以把他消掉。所  
> 以，一定是选 1 右边的位置。  
>   
> 感觉口都说干了，谁给我买点水吧，呵呵呵呵。其实这是一种感觉。3 个的也类似，反正就两个位置，要不 1，要不最左边 1 的右边，最左边 3 个，你说怎么最快消掉哦？最右边呢，肯定是把最左边的先去掉才能动后面的啊。类推吧。。。。。再说了，要是有 1 被异或掉了，那就变成了两个，这就是叫什么什么证明来着，高中的东西都忘光了。  
>   
>   
> 代码如下，直接从验证的倒过来就是了，有些累了，不想多说了。输入用户名后，直接 work()，结束后，序列号存放在 char serial\[SERIAL\_SIZE\];中，可以看到，整体框架和计算序列号是否有效是差不多的。  
> 代码应该比较容易看懂，函数名和变量名就是注释了。  
>   
> 代码:
> 
> * * *
> 
> `#define TABLE_SIZE      32   #define SERIAL_SIZE     0x1000   char            name[0x10];   char            serial[SERIAL_SIZE];   unsigned char   nametable[10];   unsigned char   nametablepos[10];   unsigned char   table[TABLE_SIZE];      int namelen = 0, seriallen = 0;      int findmodifypos()   {       int i,tableitem = 0;       for (i=1; i<10; i++)           if (nametable[i] == 1) nametablepos[tableitem++]=i;       if(tableitem > 0)       {           if (tableitem%2) return 1;            else return nametablepos[0]+1;       }else return 0;    }      void adjusttable( )   {       int pos;       while((pos = findmodifypos()) && seriallen<SERIAL_SIZE )       {           serial[seriallen]=(10+pos-table[seriallen%(TABLE_SIZE-1)])%10+'0';           seriallen++;           if(pos ==1 )nametable[pos]^=1;            else nametable[pos]^=1;        }   }      int work()   {       // 局部变量       int i;       unsigned adder,adder2;       unsigned namecalc = 0x13572468;          memset(nametable, 0, 0xa);       memset(serial, 0, SERIAL_SIZE);       seriallen = 0;       // 函数动作       for(i=0; i<namelen; i++)       {   // 计算名字特征           adder = (name[i]+ namecalc)*0x3721273+0x24681357;           adder2 = adder<<0x19;           __asm sar adder,7           namecalc = adder2|adder;       }          for (i=1; i<9; i++)       {   //根据名字特征值的 1-8 位构造名字表           nametable[i] = (unsigned char)((unsigned char)(namecalc>>i)&1);       }       nametable[9] = 1;              for (i=0; i<TABLE_SIZE; i++)            table[i] = ((namecalc>>(unsigned char)(i%(TABLE_SIZE-1)))%10);          adjusttable();       return 0;   }`
> 
> * * *
> 
> 结束语：偶是菜鸟，这个 crackme 中我学到了很多东西，如隐藏字符串，另外就是看大段汇编代码没有那么头疼了;)  
> 总的来说，这个 crackme 难易适中，适合我这种菜菜,毕竟只有 1.6k:)感觉自己杂七杂八的说了，都没说清楚。希望能给你帮助。 

> 关键词:格雷码  
>   
> 
> 代码:
> 
> * * *
> 
> `DWORD MsGen2(HWND hDlg)   {     char szName[0x100];     char szCode[0x400];     int Len;     int i;     int K;     DWORD N;        if (Len = GetDlgItemText(hDlg, IDC_EDIT_NAME, szName, 16))     {       K = 0x13572468;       for (i=0;i<Len;i++)       {         K = (K + szName[i]) * 0x3721273 + 0x24681357;         K = (K << 0x19) | (K >> 7);       }          Len = 0;       do        {         Len++;       } while((Len ^ (Len >> 1)) != ((K >> 1) & 0xFF | 0x100));          for (i=0;i<Len;i++)       {         N = Len - i;         N ^= N - 1;         N ^= N >> 1;         __asm         {           bsf eax,N           mov N,eax         }         szCode[i] = ((BYTE)(N + 11 - (((DWORD)K >> (i % 31)) % 10)) % 10) + 0x30;       }       szCode[Len] = 0;          SetDlgItemText(hDlg, IDC_EDIT_CODE, szCode);     }        return 1;   }`
> 
> * * *

> base =====>为 namehash  
>   
> char buffer\[9\] = {0};  
> unsigned int namehash = base;  
> for (int i=0; i<8; i++)  
> {  
>  buffer\[i\] = (namehash>>(i+1))&0x1;  
> }  
> buffer\[8\] = 1;  
>   
> int nseqidlen;  
> char seqid\[31\];  
> byte nowbyte;  
>   
> for (i=0; i<nseqidlen; i++)  
> {  
>  nowbyte = seqid\[i\];  
>  int t1 = (namehash>>(i%31))%10;  
>  int tem1 = (t1 + nowbyte -0x30)%10;  
>  if (tem1==1)  
>  {  
>   buffer\[0\] ^= 1;  
>   continue;  
>  }  
>  if (buffer\[tem1-2\]!=1)  
>  {  
>   error;  
>  }  
>  if (tem1-2>=1)  
>  {  
>   for (i=0; i<tem1-2; i++)  
>   {  
>    if (buffer\[i\] == 1)  
>    {  
>     error;  
>    }  
>   }  
>  }  
>  buffer\[tem1-1\] ^= 1;  
> }  
> 通过对 tem1 的一系列取值,可以达到对其中一位进行反位的功能,前提是这一位前面都为 0.  
> 这个取值是一个数列  
> 让第 1 位反位  1   
> 让第 2 位反位  1 2 1  
> 让第 3 位反位  1 2 1 3 1 2 1  
> 让第 4 位反位  1 2 1 3 1 2 1 4 1 2 1 3 1 2 1  
> 通项式是 f(n) = f(n-1)+n+f(n-1);f(1)=1;  
>   
> int count;  
> void get\_reserve\_string(int num, char \*buffer)  
> {  
>     if (num==1)  
>     {  
>         sprintf(buffer+count, "%1d", 1);  
>         count++;  
>         return;  
>     }  
>     get\_reserve\_string(num-1, buffer);  
>     sprintf(buffer+count, "%1d", num);  
>     count++;  
>     get\_reserve\_string(num-1, buffer);  
>     return;  
> }  
> 用来生成这个数列.  
>   
> //这里用的是最正常的办法,没有优化!!!! 对 buffer 中每一个 1 进行反位  
>             for (i=0; i<9; i++)  
>             {  
>                 if (buffer\[i\]==1)  
>                 {  
>                     memset(stringbuffer, 0, 520);  
>                     count = 0;//全局置 0  
>                     get\_reserve\_string(i+1, stringbuffer);  
>                       
>                     for (int j =0; j<count; j++)  
>                     {  
>                         t1 = (namehash>>((key\_cur+j)%31))%10;  
>                         if (t1 <= (stringbuffer\[j\]-0x30))  
>                         {  
>                             key\[key\_cur+j\] = stringbuffer\[j\] - t1;  
>                         }  
>                         else  
>                             key\[key\_cur+j\] = 10 + stringbuffer\[j\] - t1;  
>                     }  
>                     key\_cur += count;        
>                 }  
>             }  
>   
> 唉,我当时注册的时候,是 12 点以前,但是老是注册不成功,没有办法,运气不好,祝福你们拉  
>   
> ps   
> sar 就是带符号的右移,定义成 int 就可以了,如果是 shr 的话,定义为 unsigned int 

> BYTE Ninekey\[9\] = {0};  
> char OutPath\[1024\] = {0};  
> int  Pathlen = 0;  
>   
> void UpRing(int idx);  
> void DownRing(int idx);  
>   
> void DownRing(int idx)  
> {  
>   if( idx > 2)  
>     DownRing(idx-2);  
>   OutPath\[Pathlen\] = idx;  
>   Ninekey\[idx-1\] = !Ninekey\[idx-1\];  
>   Pathlen++;  
>   if( idx > 2 )  
>     UpRing(idx-2);  
>   if( idx > 1 )  
>     DownRing(idx-1);  
> }  
>   
> void UpRing(int idx)  
> {  
>   if(idx > 1)  
>     UpRing(idx-1);  
>   if(idx > 2)  
>     DownRing(idx-2);  
>   OutPath\[Pathlen\] = idx;  
>   Ninekey\[idx-1\] = !Ninekey\[idx-1\];  
>   Pathlen++;  
>   if( idx > 2 )   
>     UpRing(idx-2);  
> } 

> 关键词：递归  
>   
> 
> 代码:
> 
> * * *
> 
> `int steplist[1000]={0}; //走的步数   BYTE newtmp1[10] = {0}; //状态   void x(int n);//清空 n   void y(int n);//填上 n   void setstep(int n);//异或一次      void setstep(int n) //异或一次   {     steplist[pStep++]=n;     newtmp1[n] ^= 1;   }      void x(int n) //清空 n 到最前面的   {     if (n == 1)     {       if (newtmp1[1] == 1) setstep(1);     }     else     {         y(n-1);//填上 n-1,清空 n-1 前面的       if (newtmp1[n] == 1) setstep(n); //如果有值就清空       x(n-1);//清空前面的     }      }      void y(int n) //填上 n，并清空前面的 n-1 个   {     if (n == 1)     {       if (newtmp1[1] == 0) setstep(1);     }     else     {         y(n-1);//填上 n-1 的，并清空前面的 n-2 个       if (newtmp1[n] == 0) setstep(n); //当前如果没值就填上       x(n-1);//清空前面 n-1     }      }`
> 
> * * *
> 
> 调用的时候  
> 代码:
> 
> * * *
> 
> `x(9);`
> 
> * * *

> 是 gray 码 谁要早提醒我一下就好了 网上一搜一大堆![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")  
> 偶也偶的贴出来,虽然写的很滥.  
> 
> 代码:
> 
> * * *
> 
> `GetEdx proc;获取 crackme 里面那个 edx     mov ecx,1Fh     cdq     mov eax,Codelen     idiv ecx     mov eax,UserCode     mov ecx,edx     xor edx,edx     shr eax,cl     mov ecx,0ah     div ecx     mov eax,edx     inc Codelen     ret   GetEdx endp   GetCode proc ;开始计算注册码     mov esi,offset lala     @@:   or ecx,0FFFFFFFFH     mov edi,offset haha     mov eax,1     repne scasb      not ecx     dec ecx     push ecx     .if ecx>=9h        pop ecx       ret     .elseif ecx==8       pop ecx       mov edi,offset haha       jmp @1     .endif     mov edi,offset haha     xor byte ptr [edi+ecx+1],1h     call GetEdx     mov ebx,0ah     pop ecx     add ebx,ecx     add ebx,2     sub ebx,eax     .if ebx>=0ah       sub ebx,0ah     .endif     add ebx,30h     mov byte ptr [esi],bl     inc esi   @1:  xor byte ptr [edi],1h     call GetEdx     mov ebx,0Bh     sub ebx,eax     .if ebx>=0ah       sub ebx,0ah     .endif     add ebx,30h     mov byte ptr [esi],bl     inc esi     jmp @B     ret      GetCode endp`
> 
> * * *

> 哈哈,偶的长的很难看,大家将就看把  
> 
> 代码:
> 
> * * *
> 
> `BOOL GenRegCode(HWND hWnd)    {     int  len,i;       TCHAR szName[MAXINPUTLEN]={0};     TCHAR szSerial[MAXINPUTLEN]={0};     //TCHAR szBuffer[MAXINPUTLEN]={0};     long key=0x13572468, eax=0;     unsigned char checksum[10]={0};     unsigned char magic[1024]="1213121412131215121312141213121612131214121312151213121412131217121312141213121512131214121312161213121412131215121312141213121812131214121312151213121412131216121312141213121512131214121312171213121412131215121312141213121612131214121312151213121412131219";     unsigned char tmpstring[1024]={0};        len=GetDlgItemText(hWnd, IDC_NAME, szName, sizeof(szName)/sizeof(TCHAR)+1); // 取姓名        for(i=0; i<len; i++)     {       eax = ((DWORD)szName[i]+key)*0x3721273+0x24681357;       key = (eax<<25)|(eax>>7);     }        for(i=1; i<9; i++)     {       checksum[i] = (key>>i)&1;     }     checksum[9] = 1;        do     {       eax = findindex(checksum, 1, 10);       if(eax==-1)break;       if(checksum[eax+1]==1)       {         wsprintf(tmpstring, "%d", eax+1);         strcat(szSerial, tmpstring);         checksum[eax+1]=0;       }       else       {         checksum[eax]=0;         while(eax--)         {           memset(tmpstring, 0, sizeof(tmpstring));           strncpy(tmpstring, magic, 1<<(eax));           strcat(szSerial, tmpstring);         }       }     }while(1);        len=strlen(szSerial);     for(i=0; i<len; i++)     {       szSerial[i]=0x30+(10+(szSerial[i]-0x30)-((DWORD)key>>(i%0x1f))%10)%10;     }     SetDlgItemText(hWnd, IDC_OUT, szSerial);       // 显示正确的序列号       return TRUE;   }      int findindex(char * input, char ch, int len)   {     int i;     for(i=0; i<len; i++)       if(input[i]==ch)return i;     return -1;   }`
> 
> * * *

> \[+\] Name = pediy  
> \[+\] Hash is 0x94D44A29  
> 5 2 6 3 6 3 6 8 4 7 8 9 4 2 1 0 0 0 5 2 1 0 5 7 8 4 7 8 9 4 2   
> 00 00 00 01 00 01 00 00 00 01   
> 00 01 00 01 00 01 00 00 00 01   
> 00 01 01 01 00 01 00 00 00 01   
> 00 00 01 01 00 01 00 00 00 01   
> 00 00 01 00 00 01 00 00 00 01   
> 00 01 01 00 00 01 00 00 00 01   
> 00 01 00 00 00 01 00 00 00 01   
> 00 00 00 00 00 01 00 00 00 01   
> 00 00 00 00 00 01 01 00 00 01   
> 00 01 00 00 00 01 01 00 00 01   
> 00 01 01 00 00 01 01 00 00 01   
> 00 00 01 00 00 01 01 00 00 01   
> 00 00 01 01 00 01 01 00 00 01   
> 00 01 01 01 00 01 01 00 00 01   
>   
> 我的输出的一小段，代码就不贴了，用了递归 

> 我的想法很简单，nametable\[0\]是可以自由从 0 变成 1 或者从 1 变成 0 的，而要将 nametable\[n\]进行 0、1 的变化，需要 nametable\[n-1\]=1,nametable\[0..n-2\]=0，这样，其实就是一个很简单的递归过程，虽然效率有点低，但是很容易想到。  
> 即要将第 n 位将 1 变成 0，首先将 n-1 位变成 1，然后将 n 位变成 0，然后再将 n-1 位变成 0，这样就是很很简单的递归问题了 

> 偶也来献丑，同样是格雷码。  
>   
> 
> 代码:
> 
> * * *
> 
> `#define CODE_LENGTH     9   #define TABLE_SIZE      ((1 << CODE_LENGTH) - 1)      unsigned int table[TABLE_SIZE] = {       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 6,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 7,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 6,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 8,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 6,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 7,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 6,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 9,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 6,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 7,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 6,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 8,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 6,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 7,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 6,       1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1, 5, 1, 2, 1, 3, 1, 2, 1, 4, 1, 2, 1, 3, 1, 2, 1   };      char    name[32];   char    serial[TABLE_SIZE + 1];      unsigned int calc_name       (       char            name[32],       unsigned int    code[CODE_LENGTH]       )       {       int name_length = strlen(name);       int factor = 0x13572468;       int tmp1, tmp2;       unsigned char c;       int i;          for (i = 0; i < name_length; i++)           {           tmp1 = name[i];           tmp1 += factor;           tmp1 *= 0x03721273;           tmp1 += 0x24681357;           tmp2 = tmp1;           tmp2 <<= 0x19;           tmp1 >>= 0x07;           tmp2 |= tmp1;           factor = tmp2;           }          for (i = 1; i < CODE_LENGTH; i++)           {           tmp1= factor;           tmp1 = tmp1 >> i;           c = (unsigned char)tmp1;           c &= 0x01;           code[i - 1] = c;           }          code[8] = 1;          return (unsigned int)factor;       }      void calc_code1       (       int *   a,       int *   b       )       {       int i = CODE_LENGTH;       a += (CODE_LENGTH - 1);       b += (CODE_LENGTH - 1);       *b = *a;          for (i--; i > 0; i--, a--, b--)           {           *(b - 1) = (*b) ^ (*(a - 1));           }       }      int calc_code2       (       int *   a       )       {       int v = 0;       int i;          i = CODE_LENGTH;       a += (CODE_LENGTH - 1);       for (; i > 0; i--, a--)           {           v = (v << 1) + (*a);           }       return v;       }      int calc(char input[32])       {       int serial_length = 0;       unsigned int name_int = 0;       unsigned int name_code[CODE_LENGTH] = {0};       int tmp[CODE_LENGTH] = {0};       int i;          name_int = calc_name(input, name_code);       calc_code1(name_code, tmp);       serial_length = calc_code2(tmp);          for (i = 0; i < serial_length; i++)           {           serial[i] = table[serial_length - i - 1] - ((name_int >> (i % 31)) % 10);           if (serial[i] < 0)               serial[i] += 10;           serial[i] += 0x30;           }          return 0;       }`
> 
> * * *

> 计算机不是个好东西 

> #define M(i,x,fname,index) x\[index\]=(10+i- (fname>>(index%31))%10 )%10;printf("%d",x\[index++\]);  
>   
> #define f(i) if( i==1) M(i,x,fname,index) \\  
> else\\  
> f(i-1);M(i,x,fname,index);f(i-1)  
>   
> void main()  
> {  
>    calfname(inputName);//根据输入名字计算名字 hash 值  
>   calTable01(fname);//根据 hash 值计算 01 表  
>   //输出结果  
>   for(i=1;i<10;i++)  
>   {  
>       if( table01\[i\] )  
>             f(i);  
>   }  
> }  
> 上面几行代码就可以搞定第一题了 

> 贴一下我的解法,标准递归,很容易理解:  
>   
>   /\*\*  
>    \* 算法思路是: 依次使 9,8,7,...,1 位变成 0. 因为,假如使第 9 位变成 0 了,  
>    \* 那么再把第 8 位变成 0 时就不需要考虑第 9 位了. 同样,第 8 位变成 0 后,  
>    \* 再改变第 7 位时就不需要考虑 8,9 位. 问题难度将逐步降低,直到最终解决,  
>    \* 而且,这个思路可以递归实现.  
>    \*/  
>   for (i = 9; i; i--)  
>   {  
>     if ( bits\[i\] != 0 )  
>     {  
>       xor(bits,i);  
>     }  
>   }  
>   
> 下面是 xor 函数实现:  
>   
> /\*\*  
>  \* 将 bits 数组的第 pos 位取反,也就是 bits\[pos\] ^= 1.  
>  \*  
>  \* 为将 pos 位取反，必须先：  
>  \* 1. bits\[pos - 1\] = 1  
>  \* 2. bits\[1\] 到 bits\[pos - 2\] 都为 0  
>  \* 这个函数通过递归调用来满足这两个条件.  
>  \*  
>  \* @note 未做递归深度检查. CrackMe2 中序列号最长 4096 (就是前面定义的 MAX\_TIMES),这里假设不会超过这个值.  
>  \*/  
> static void xor(uint8\_t bits\[10\],int pos)  
> {  
>   int i;  
>   
>   /// 递归结束条件  
>   if (pos == 1)  
>   {  
>     bits\[1\] ^= 1;  
>     m\_codes\[m\_count++\] = 1;  
>     return;  
>   }  
>   
>   /// 先递归调用来满足条件 1  
>   if ( bits\[pos - 1\] == 0 )  
>   {  
>     xor(bits,pos - 1);  
>   }  
>   
>   /// 再递归调用来满足条件 2  
>   for (i = pos - 2; i; i--)  
>   {  
>     if ( bits\[i\] != 0 )  
>     {  
>       xor(bits,i);  
>     }  
>   }  
>     
>   /// 2 个条件满足了,现在可以将第 pos 位取反了  
>   bits\[pos\] ^= 1;  
>   
>   /// 记录第几步改变了第几位,后面根据这个生成序列号  
>   m\_codes\[m\_count++\] = pos;  
> } 

> 我的分析方法是不是北部对?  
> 分析的速度很慢  
> 步骤一：任意输入，用户名：aaaaa 注册码：bbbbb  
>         我们查找 getdlgitemtexta 首值，找到后下硬件访问断点，点击注册我们的程序将停在这里：  
>   
>   
> 00400499  |.  8D45 EC       lea     eax, dword ptr \[ebp-14\]          ;《--------这里！  
> 0040049C  |.  6A 10         push    10                               ; /Count = 10 (16.)  
> 0040049E  |.  50            push    eax                              ; |Buffer  
> 0040049F  |.  68 E9030000   push    3E9                              ; |ControlID = 3E9 (1001.)  
> 004004A4  |.  FF75 08       push    dword ptr \[ebp+8\]                ; |hWnd  
> 004004A7  |.  FFD6          call    esi                              ; \\GetDlgItemTextA  
> 004004A9  |.  85C0          test    eax, eax                         ;  字符是零吗？  
> 004004AB  |.  74 6C         je      short 00400519                   ;  是的话就跳  
> 004004AD  |.  8D85 E8EFFFFF lea     eax, dword ptr \[ebp-1018\]        ;  否则就把地址内容放 A 中  
> 004004B3  |.  68 00100000   push    1000                             ; /Count = 1000 (4096.)  
> 004004B8  |.  50            push    eax                              ; |Buffer  
> 004004B9  |.  68 EA030000   push    3EA                              ; |ControlID = 3EA (1002.)  
> 004004BE  |.  FF75 08       push    dword ptr \[ebp+8\]                ; |hWnd  
> 004004C1  |.  FFD6          call    esi                              ; \\GetDlgItemTextA  
>   
> 004004C3  |.  85C0          test    eax, eax                         ;  指向下一个窗口数 ，                                                                      ;窗口有数据吗？  
> 004004C5 >|.  74 52         je      short 00400519                   ;  没有数据就跳到下一                                                                       ;个窗口  
> 004004C7  |.  8D7D EC       lea     edi, dword ptr \[ebp-14\]          ;  字符的第一个放到 DI 里  
> 004004CA  |.  83C9 FF       or      ecx, FFFFFFFF                    ;  对 C 放 1  
>   
> 经过分析知道这是读取用户名，和用户注册码的地方  
> 步骤二：  
>   
> 004004DA  |> /0FBE4415 EC   /movsx   eax, byte ptr \[ebp+edx-14\]      ;  把用户名转换到 A 寄存                                                                     ;  器里  
> 004004DF  |. |03C3          |add     eax, ebx                        ;  AB 相加  
> 004004E1  |. |69C0 73127203 |imul    eax, eax, 3721273               ;  乘上 3721273 放到 A 里  
> 004004E7  |. |05 57136824   |add     eax, 24681357                   ;  再加上 24681357  
> 004004EC  |. |8BF0          |mov     esi, eax                        ;  转化结果放 SI 里  
> 004004EE  |. |C1E6 19       |shl     esi, 19                         ;  左移动 19 位  
> 004004F1  |. |C1F8 07       |sar     eax, 7                          ;  循环右移动 7 位  
> 004004F4  |. |0BF0          |or      esi, eax                        ;  结果相或  
> 004004F6  |. |42            |inc     edx                             ;  D 增加一  
> 004004F7  |. |3BD1          |cmp     edx, ecx                        ;  到 4 了吗？  
> 004004F9  |. |8BDE          |mov     ebx, esi                        ;  把 SI 的结果保存到 BX                                                                                                         ;  里  
> 004004FB  |.^\\7C DD         \\jl      short 004004DA  
>   
> 分析可知是一个用户名的函数变换：  
> 函数如下：  
> unsingned int b=0x13572468; //常数  
> unsingned int a=0;          //中间变量 1  
> unsingned int s=0;          //中间变量 2  
> read();//读入一个用户名  
> func1(){  //用户名转化函数  
>    for(i=0;i<4;i++){  
>         a=read();  
>         a=(a+b)\*3721273+24681357;  
>         s=a;  
>         s=s<<19;  
>         a=s>>7;  
>         s=s|a;  
>         b=s;  
>      }  
> }  
> \====================================  
> //参数 2 的调用  
> 004002CC  /$  55            push    ebp                              ;  传递了两个参数,用户名,运算,和注册玛的地址  
> 004002CD  |.  8BEC          mov     ebp, esp                         ;  保护堆栈指针  
> 004002CF  |.  81EC 28010000 sub     esp, 128                         ;  分配堆栈  
> 004002D5      8065 DC 00    and     byte ptr \[ebp-24\], 0             ;  对 12eab4 缓冲区清零  
> 004002D9  |.  53            push    ebx                              ;  保存结果用户运算结果  
> 004002DA  |.  56            push    esi                              ;  保存结果  
> 004002DB  |.  57            push    edi                              ;  保护将要用到的寄存器  
> 004002DC  |.  33C0          xor     eax, eax                         ;  对 A 清零  
> 004002DE  |.  8D7D DD       lea     edi, dword ptr \[ebp-23\]          ;  地址偏移量送 DI  
> 004002E1  |.  AB            stos    dword ptr es:\[edi\]               ;  串操作指令,对此地址清零  
> 004002E2  |.  80A5 D8FEFFFF>and     byte ptr \[ebp-128\], 0            ;  对低八位清零  
> 004002E9  |.  6A 40         push    40  
> 004002EB  |.  AB            stos    dword ptr es:\[edi\]               ;  保存节  
> 004002EC  |.  AA            stos    byte ptr es:\[edi\]                ;  保存字节  
> 004002ED  |.  59            pop     ecx  
> 004002EE  |.  33C0          xor     eax, eax                         ;  清零  
> 004002F0  |.  8DBD D9FEFFFF lea     edi, dword ptr \[ebp-127\]         ;  对一首址放 DI 里  
> 004002F6  |.  804D F4 FF    or      byte ptr \[ebp-C\], 0FF            ;  对地址 12edcc 的低字节的内容之一  
> 004002FA  |.  F3:AB         rep     stos dword ptr es:\[edi\]          ;  对一个指定的区域清零  
> 004002FC  |.  804D E8 FF    or      byte ptr \[ebp-18\], 0FF           ;  对底字节之一  
> 00400300  |.  83C9 FF       or      ecx, FFFFFFFF  
> 00400303  |.  66:AB         stos    word ptr es:\[edi\]                ;  对双字清陵  
> 00400305  |.  AA            stos    byte ptr es:\[edi\]                ;  清零  
> 00400306  |.  8B7D 0C       mov     edi, dword ptr \[ebp+C\]           ;  对注册码地址放到 DI 里  
> 00400309  |.  33C0          xor     eax, eax                         ;  A 清零  
> 0040030B  |.  F2:AE         repne   scas byte ptr es:\[edi\]           ;  把 AL 或 AX 的内容与目标串作比较,比较结果反映在标志位  
> 0040030D  |.  F7D1          not     ecx                              ;  厕得有几个字符？  
>   
> 00400313  |.  C645 F5 63    mov     byte ptr \[ebp-B\], 63             ;  对 12EACD 填充数据  
> 00400317  |.  C645 F6 FB    mov     byte ptr \[ebp-A\], 0FB  
> 0040031B  |.  C645 F7 9A    mov     byte ptr \[ebp-9\], 9A  
> 0040031F  |.  C645 F8 03    mov     byte ptr \[ebp-8\], 3  
> 00400323  |.  C645 F9 A3    mov     byte ptr \[ebp-7\], 0A3  
> 00400327  |.  C645 FA DA    mov     byte ptr \[ebp-6\], 0DA  
> 0040032B  |.  C645 FB 72    mov     byte ptr \[ebp-5\], 72  
> 0040032F  |.  C645 FC FE    mov     byte ptr \[ebp-4\], 0FE  
> 00400333  |.  C645 FD C9    mov     byte ptr \[ebp-3\], 0C9  
> 00400337  |.  C645 FE B7    mov     byte ptr \[ebp-2\], 0B7  
> 0040033B  |.  C645 E9 6A    mov     byte ptr \[ebp-17\], 6A  
> 0040033F  |.  C645 EA D1    mov     byte ptr \[ebp-16\], 0D1  
> 00400343  |.  C645 EB D2    mov     byte ptr \[ebp-15\], 0D2  
> 00400347  |.  C645 EC 4E    mov     byte ptr \[ebp-14\], 4E  
> 0040034B  |.  C645 ED 82    mov     byte ptr \[ebp-13\], 82  
> 0040034F  |.  C645 EE DA    mov     byte ptr \[ebp-12\], 0DA  
> 00400353  |.  C645 EF 72    mov     byte ptr \[ebp-11\], 72  
> 00400357  |.  C645 F0 FE    mov     byte ptr \[ebp-10\], 0FE  
> 0040035B  |.  C645 F1 C9    mov     byte ptr \[ebp-F\], 0C9  
> 0040035F  |.  C645 F2 B7    mov     byte ptr \[ebp-E\], 0B7  
>   
> \===============================================  
> 00400363  |.  8BF1          mov     esi, ecx                         ;  保存注册字的数目  
> 00400365  |.  8BFB          mov     edi, ebx                         ;  BX 到 DI  
> 00400367  |>  8B45 08       /mov     eax, dword ptr \[ebp+8\]          ;  一字节的数放 A 里  
> 0040036A  |.  8BCF          |mov     ecx, edi                        ;  DI 内容到 C  
> 0040036C  |.  D3E8          |shr     eax, cl                         ;  循环右移动 CL 个字节  
> 0040036E  |.  22C3          |and     al, bl                          ;  A 与 B 的低字节相与  
> 00400370  |.  88443D DC     |mov     byte ptr \[ebp+edi-24\], al       ;  把 A 低字节的内容放到一地址  
> 00400374  |.  47            |inc     edi  
> 00400375  |.  83FF 09       |cmp     edi, 9  
> 00400378  |.^ 7C ED         \\jl      short 00400367  
> 0040037A  |.  33FF          xor     edi, edi                         ;  循环 9 次后放 DI 清零  
> 0040037C  |.  885D E5       mov     byte ptr \[ebp-1B\], bl            ;  低字节送次  
> 0040037F  |.  85F6          test    esi, esi                         ;  SI 是零吗？  
> 00400381  |.  7E 6B         jle     short 004003EE                   ;  小于等于的话就跳  
> 00400383  |>  8B45 0C       /mov     eax, dword ptr \[ebp+C\]          ;  把注册码放 EA 内  
> 00400386  |.  8A0407        |mov     al, byte ptr \[edi+eax\]          ;  把注册玛送 BL  
> 00400389  |.  3C 30         |cmp     al, 30                          ;  注册玛合 30 比较，在 A 里是 0  
> 0040038B  |.  8845 FF       |mov     byte ptr \[ebp-1\], al            ;  注册码放到 12EAD7 内存里  
> 0040038E  |.  0F8C 97000000 |jl      0040042B                        ;  小于 30 的话，就跳  
> 00400394  |.  3C 39         |cmp     al, 39                          ;  和 39 比较  
> 00400396      0F8F 8F000000 jg      0040042B                         ;  大于 39 吗？大于就就跳在 ASC 里是是 9  
> 0040039C  |.  8BC7          |mov     eax, edi                        ;  否 DI 放 A  
> 0040039E  |.  6A 1F         |push    1F  
> 004003A0  |.  99            |cdq                                     ;  字扩展  
> 004003A1  |.  59            |pop     ecx  
> 004003A2  |.  F7F9          |idiv    ecx  
> 004003A4  |.  8B45 08       |mov     eax, dword ptr \[ebp+8\]          ;  送 A  
> 004003A7  |.  6A 0A         |push    0A  
>   
> \=====================================================  
> 就再也分析不下去了  
> 都是动态的 

 4dd466d0e1e0cc123afbe504da2462ca