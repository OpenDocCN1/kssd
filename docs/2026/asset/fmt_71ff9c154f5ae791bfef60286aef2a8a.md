
```
:0041F037 A5
  movsd
:0041F038 8BCA
  mov ecx, edx
:0041F03A 83E103
and ecx, 00000003
:0041F03D F3
  repz
:0041F03E A4
  movsb
:0041F03F 8D7C2414                lea
edi, dword ptr [esp+14]
:0041F043 83C9FF
or ecx, FFFFFFFF
:0041F046 F2
  repnz
:0041F047 AE
  scasb
:0041F048 F7D1
  not ecx
:0041F04A 49
  dec ecx
:0041F04B 80F908
cmp cl, 08                  //判UserName的长度
:0041F04E 884C2410                mov
byte ptr [esp+10], cl
:0041F052 732F
  jnb 0041F083                //大于等于8则取前8个字符
:0041F054 8B542410                mov
edx, dword ptr [esp+10] //否则拼也要拼8个字符出来
:0041F058 8D7C2414                lea
edi, dword ptr [esp+14]
:0041F05C 81E2FF000000            and edx, 000000FF
:0041F062 83C9FF
or ecx, FFFFFFFF
:0041F065 81C2DEAE4000            add edx, 0040AEDE
:0041F06B F2
  repnz
:0041F06C AE
  scasb
:0041F06D F7D1
  not ecx
:0041F06F 2BF9
  sub edi, ecx
:0041F071 8BC1
  mov eax, ecx
:0041F073 8BF7
  mov esi, edi
:0041F075 8BFA
  mov edi, edx
:0041F077 C1E902
shr ecx, 02
:0041F07A F3
  repz
:0041F07B A5
  movsd
:0041F07C 8BC8
  mov ecx, eax
:0041F07E 83E103
and ecx, 00000003
:0041F081 F3
  repz
:0041F082 A4
  movsb
```

最后就是利用`UserString[ ]`和`UserKey[ ]`进行判断。这也是一个循环。

```
:0041F028 33ED
  xor ebp, ebp
......................................................
:0041F083 C605E6AE400000          mov byte ptr [0040AEE6],
00
:0041F08A B9D4AE4000              mov ecx,
0040AED4
:0041F08F BE08000000              mov esi,
00000008      //循环控制变量k = 8
:0041F094 8A01
  mov al, byte ptr [ecx]  //取出a[k]
:0041F096 3C20
  cmp al, 20              //和0x20相比，分两种情况处理
:0041F098 730E
  jnb 0041F0A8
:0041F09A 33D2
  xor edx, edx
:0041F09C 25FF000000              and eax,
000000FF
:0041F0A1 8A510A
mov dl, byte ptr [ecx+0A] //取出UserString[k]
:0041F0A4 0BD0
  or edx, eax              //与a[k]相或
:0041F0A6 EB0C
  jmp 0041F0B4
:0041F0A8 33D2
  xor edx, edx
:0041F0AA 25FF000000              and eax,
000000FF
:0041F0AF 8A510A
mov dl, byte ptr [ecx+0A] //取出UserString[k]
:0041F0B2 33D0
  xor edx, eax              //与a[k]异或
:0041F0B4 03EA
  add ebp, edx              //累加到ebp中
:0041F0B6 41
  inc ecx
//下一个字节
:0041F0B7 4E
  dec esi
//k--;
:0041F0B8 75DA
  jne 0041F094              //继续循环
:0041F0BA 33C0
  xor eax, eax
:0041F0BC 5F
  pop edi
:0041F0BD 85ED
  test ebp, ebp            //累加和为0吗？
:0041F0BF 5E
  pop esi
:0041F0C0 5D
  pop ebp
:0041F0C1 0F94C0
sete al                  //为0则OK，否则bad
guy
:0041F0C4 5B
  pop ebx
:0041F0C5 83C42C
add esp, 0000002C
:0041F0C8 C3
  ret
```

上面这个循环可等价为：

```
long ebp = 0
for(k = 0; k < 8; k++)
{
    if (a[k] < 0x20)
    {
        ebp += UserString[k] | a[k];
    }
    else
    {
        ebp += UserString[k] ^ a[k];
    }
}
```

注册码正确与否的标准为：

```
if (ebp)
{
    printf("Wrong key");
}
else
{
    printf("good guy");
}
```

最后这个循环初看之下也比较难求逆，因为其中的或运算是不可逆的。但实际上我们只要让`a[k] < 0x20`这个条件永远不会满足即可避开这个或运算，此时上面这个循环简化为可逆的异或变换：

```
for(k = 0; k < 8; k++)
{
    ebp += UserString[k] ^ a[k];
}
```

这样要想让累加和`ebp`为0，只要满足`a[k] = UserString[k]`（`k` = 0, ..., 7）即可，由于`UserString[k]`是由可显示的用户名得来的，所以肯定满足`a[k] >= 0x20`。至此得到注册机如下：

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
void main(void)
{
    char        UserName[128];
    char        UserString[8];
    char        a[8];
    int          index, len;
    char        k, j;
    signed long  temp, guess;
    char        SuccessCounter;
    do
    {
        printf("Input your name(at least 5 chars):");
        gets(UserName);
        len = strlen(UserName);
    } while(len < 5);
    strupr(UserName);
    index = 0;
    for(k = 0; k < 8; k++)
    {
        a[k]  = UserString[k] = UserName[index++];
        index %= len;
    }
    //下面用穷举求出变换前的8个字节
    SuccessCounter = 0;
    for(k = 0; k < 8; k++)
    {
        for(guess = 0; guess <= 255; guess++)
        {
            temp = 1L;
            for(j = 7; j > 0; j--)
            {
                temp \*= guess;
                if (temp > 0x00008899L)
                {
                    temp %=
0x00008899L;
                }
            }
            if ((temp % 0x000000BBL) == a[k])
            {
                SuccessCounter++;
                a[k] = guess;
                break;
            }
        }
    }
    //判断8个字节是否全部穷举成功
    if (SuccessCounter != 8)
    {
        printf("Guess failed.\n");
        exit(-1);
    }
    for(k = 0; k < 8; k++)
    {
        a[k] ^= (k + 0x50);
    }
    printf("Your registration key is: ");
    for(k = 0; k < 4; k++)
    {
```
