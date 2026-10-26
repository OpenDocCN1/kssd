
```
004896E1   8B80E0020000           mov     eax, [eax+$02E0]

\* Reference to: controls.TControl.GetText(TControl):TCaption;

|

004896E7   E82417FAFF             call    0042AE10

004896EC   837DF400               cmp     dword ptr [ebp-$0C], +$00

004896F0   740D                   jz      004896FF

004896F2   8B45F4                 mov     eax, [ebp-$0C]   ；EAX 0101B710 ASCII "1234567890"

\* Reference to: System.Proc\_00403DBC

|

004896F5   E8C2A6F7FF             call    00403DBC

004896FA   83F814                 cmp     eax, +$14                ；eax=0000000A，注册码长度

，这里很关键！！！！

；后面会看到，注册码长度必须为20

004896FD   7D38                   jnl     00489737                 ：关键，要跳才行，输
入"1234567890"没跳

004896FF   6A07                   push    $07                     //注意常数

00489701   68D7A1BD37             push    $37BDA1D7               //注意常数

00489706   8D85E8FEFFFF           lea     eax, [ebp+$FFFFFEE8]

\* Reference to: Unit\_004073B8.Proc\_004082F8

|

0048970C   E8E7EBF7FF             call    004082F8

   {

     004082F8   55                     push    ebp

     004082F9   8BEC                   mov     ebp, esp

     004082FB   83C4F8                 add     esp, -$08

     004082FE   6A00                   push    $00

     00408300   8D5508                 lea     edx, [ebp+$08]

；D EDX

；0012FA8C  D7 A1 BD 37 07 00 00 00  住?... 这是压入的参数

     00408303   8955F8                 mov     [ebp-$08], edx

     00408306   C645FC10               mov     byte ptr [ebp-$04], $10

     0040830A   8D4DF8                 lea     ecx, [ebp-$08]

     \* Possible String Reference to: '%d'

     |

     0040830D   BA28834000             mov     edx, $00408328

     \* Reference to: Unit\_004073B8.Proc\_00409030

     |

     00408312   E8190D0000             call    00409030     ；进去，但没观察到什么，可F8

     00408317   59                     pop     ecx

     00408318   59                     pop     ecx

     00408319   5D                     pop     ebp

     0040831A   C20800                 ret     $0008

   }

00489711   8B85E8FEFFFF           mov     eax, [ebp+$FFFFFEE8]   ；EAX 0101B728 ASCII
"30999945687"，这里有东东了
；运行calc，输入十进制30999945687(因为上面那个call中有格式串"%d")，转化为十六进制，737BDA1D7
，呵呵，知道了，刚压入了两个参数的，
；两个双字合并为64位的字，再转化为十进制了

00489717   50                     push    eax

00489718   8D95E4FEFFFF           lea     edx, [ebp+$FFFFFEE4]

0048971E   B821771906             mov     eax, $06197721                   //注意常数

\* Reference to: Unit\_004073B8.Proc\_004082C8

|

00489723   E8A0EBF7FF             call    004082C8

     {

      004082C8   83C4F8                 add     esp, -$08

      004082CB   6A00                   push    $00

      004082CD   89442404               mov     [esp+$04], eax

      004082D1   C644240800             mov     byte ptr [esp+$08], $00

      004082D6   8D4C2404               lea     ecx, [esp+$04]

      004082DA   8BC2                   mov     eax, edx

      \* Possible String Reference to: '%d'

      |

      004082DC   BAF4824000             mov     edx, $004082F4

      \* Reference to: Unit\_004073B8.Proc\_00409030

      |

      004082E1   E84A0D0000             call    00409030   ；还是上面转化64进制到10进制中的那
个call

      004082E6   59                     pop     ecx

      004082E7   5A                     pop     edx

      004082E8   C3                     ret

     }

00489728   8B95E4FEFFFF           mov     edx, [ebp+$FFFFFEE4]   ；EDX 0101B740 ASCII
"102332193" 102332193=$06197721

0048972E   8D45F4                 lea     eax, [ebp-$0C]

00489731   59                     pop     ecx

；EAX 0012FBC0    D [eax] 0101B710  31 32 33 34 35 36 37 38  12345678

                          0101B718  39 30 00                 90.

；ECX 0101B728 ASCII "30999945687"

；EDX 0101B740 ASCII "102332193"

\* Reference to: System.Proc\_00403E08

|

00489732   E8D1A6F7FF             call    00403E08          //这个call，见[分析四]

\*\*\*\*\*\*\*\*\*\*\*\*\*\*注册码长为0x14跳到这里，前面那一段没有执行

00489737   33DB                   xor     ebx, ebx          //计数器清零

00489739   8D85EEFEFFFF           lea     eax, [ebp+$FFFFFEEE]   //eax=0012FABA，设为String1
的地址

；这个地址和前面初始化$100个byte的首地址相同，

这句执行完后，看到CPU下面的提示面板里有：

Stack ss:[0012FBC0]=0101B758, (ASCII "10233219330999945687")  ===>果然不错啊！^\_^

edx=0101B761, (ASCII "30999945687")

Jump from 00489751

Loop：

0048973F   8B55F4                 mov     edx, [ebp-$0C]           ;EDX 0101B758 ASCII
"10233219330999945687"

；注册码长为20，EDX 0101B888 ASCII "12345678901234567890"，这是输入的注册码

00489742   8A141A                 mov     dl, byte ptr [edx+ebx]   ；依次取每个字符

00489745   80EA14                 sub     dl, $14

00489748   8810                   mov     [eax], dl                ；存入string1 ,byte to
word

//修正前面初始化的$100个byte

0048974A   43                     inc     ebx                      ；计数器++

0048974B   83C002                 add     eax, +$02                ;Move to the next word

0048974E   83FB14                 cmp     ebx, +$14                ;IS the END?

00489751   75EC                   jnz     0048973F                 ；Goto LOoP

00489753   8BC7                   mov     eax, edi                 ；eax=00000014，长度   //
取出前面压入的数

00489755   888525FFFFFF           mov     [ebp+$FFFFFF25], al      ；mov byte ptr ss:[ebp-
DB],al //习惯于看减法

                                                                   ；ebp-DB=0012FAF1

0048975B   8A45F2                 mov     al, byte ptr [ebp-$0E]   ；al=05

0048975E   888527FFFFFF           mov     [ebp+$FFFFFF27], al      ；mov     [ebp-D9], al

                                                                   ；ebp-D9=0012FAF3

00489764   8A45F0                 mov     al, byte ptr [ebp-$10]   ；al=04

00489767   88852AFFFFFF           mov     [ebp+$FFFFFF2A], al      ；mov byte ptr ss:[ebp-
D6],al

                                                                   ；ebp-D6=0012FAF6

0048976D   8A45EE                 mov     al, byte ptr [ebp-$12]   ；al=1D

00489770   88852DFFFFFF           mov     [ebp+$FFFFFF2D], al      ；mov byte ptr ss:[ebp-
D3],al

                                                                   ；ebp-D3=0012FAF9

\* Possible String Reference to: '2810'

|

00489776   B8D0984800             mov     eax, $004898D0

\* Reference to: Unit\_004073B8.Proc\_0040832C

|

0048977B   E8ACEBF7FF             call    0040832C              ///见后面的[分析三] ，
将"2810"转化为十六进制数0AFA

00489780   8BC8                   mov     ecx, eax              //ECX 00000AFA

00489782   8BC1                   mov     eax, ecx              //

00489784   BB64000000             mov     ebx, $00000064        //

00489789   99                     cdq                           //

0048978A   F7FB                   idiv    ebx

；EAX 0000001C

；ECX 00000AFA      ECX=EAX\*EBX+EDX

；EDX 0000000A

；EBX 00000064

0048978C   8BD8                   mov     ebx, eax              //商

0048978E   8BC3                   mov     eax, ebx              //商

00489790   0414                   add     al, +$14              //1C+14=30

00489792   88852EFFFFFF           mov     [ebp+$FFFFFF2E], al   ；mov byte ptr ss:[ebp-D2],al
，ebp-0d2=0012FAFA，0x100个byte首地址

00489798   2ACB                   sub     cl, bl               //FA-1C=DE，将版本信息保存到注
册信息数组中

0048979A   80C114                 add     cl, $14              //DE+14=F2

0048979D   888D38FFFFFF           mov     [ebp+$FFFFFF38], cl   ；mov byte ptr ss:[ebp-C8],cl
  ebp-C8=0012FB04

004897A3   8D45F8                 lea     eax, [ebp-$08]     //eax=0012FBC4，不是0x100byte中
的字

\* Reference to: System.Proc\_00403B40

|

004897A6   E895A3F7FF             call    00403B40

   {

     00403B40   8B10                   mov     edx, [eax]

     00403B42   85D2                   test    edx, edx                     //edx=0就跳了，看
来edx是个标志

     00403B44   741B                   jz      00403B61

     00403B46   C70000000000           mov     dword ptr [eax], $00000000  //不为0就置[eax]为
0

     00403B4C   8B4AF8                 mov     ecx, [edx-$08]
```
