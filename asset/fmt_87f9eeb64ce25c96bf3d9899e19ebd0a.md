
```asm
:00447B68 E887BFFBFF           call 00403AF4
:00447B6D 8D55F4               lea edx, dword ptr [ebp-0C]
:00447B70 8B83D8020000         mov eax, dword ptr [ebx+000002D8]
:00447B76 E8DDBCFDFF           call 00423858
:00447B7B 8B55F4               mov edx, dword ptr [ebp-0C] //EDX 中为我输入的注册码
:00447B7E 8B45FC               mov eax, dword ptr [ebp-04] //EAX 中为刚才循环完后的累加值
:00447B81 E8BEBFFBFF          call 00403B44
:00447B86 7528                 jne 00447BB0  //如果不相等就死了 ：（
```

到此为止，整个 `CrackMe` 的算法已经完全弄明白了，为了使大家加深印象我再综合的复诉一边：

举例说明 `NYDoll` 是如何产生出正确的注册码 `78339` 的

1. 依次取 `NYDoll` 中的字符，并将它以 `ASCII` 码的形式保存
2. `ASCII` 码的值加上 `11111`，再保存起来
3. 若用户名没有取完则循环，第二次 `ASCII` 码值加上 `11111` 的结果和第一次的累加值相加
4. 若第一次循环，则累加值为 `11111`

即注册过程的中间数据为：`11111+(78+11111)+(89+11111)+(68+11111)+(111+11111)+(108+11111)+(108+11111)`
结果等于 `78339` 即为正确的注册码了

接下来我们来看看要如何编写一个注册机，`TC` 没有图形界面

```vb
Private Sub Text1_Change()
    Dim c_num, aa, i As Integer '定义变量
    Dim a, b, c, d, e         '定义变量

    c_num = Len(Trim(Text1.Text)) 'c_num 指注册名的字符个数

    If c_num = 0 Then         '若用户名一栏为空则提示
        MsgBox "Enter Your Name First!"     '提示信息
    Else
        For i = 1 To c_num    '若填写完毕则开始循环运算
            a = Mid(Text1, i, 1)            'a 指现在取的一个字符
            c = Asc(a)                  '将用户名依次转换为 ASCII 码形式
            If i = 1 Then
                d = c + 11111 + 11111     '第一次循环累计值为 11111
            Else
                d = c + 11111     '用户名的每位的 ASCII 码值加上 11111
            End If

            e = e + d     '将累加值存入变量 E 中
        Next i

        Text2.Text = e     '循环完成后 E 中存放的就是真正的注册码了
    End If
End Sub
```

```c
#include "stdio.h"

main()
{
    unsigned char string[98];
    int d,i;
    unsigned long c,e=11111,f=0;

    printf("*********CrackMe 1.0*********\nThis keygen is made by Pretender\nPlease input your name : ");
    gets(string);
    printf("Your Register code is  : ");
    d=strlen(string);
    for(i=0;i<d;i++)
    {
        c=string[i];
        f+=c;
    }
    f+=(d+1)*e;
    printf("%ld",f);
    printf("\n ******************************\n *Welcome to WWW.CRACKNOW.COM*     /     /     / --\n ******************************\n");
}
```
`4dd466d0e1e0cc123afbe504da2462ca`
