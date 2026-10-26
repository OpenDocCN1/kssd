
```text
0046ED7F  |.  C745 F8 FFFFF>mov     dword ptr [ebp-8], -1          ;  注册标志赋值
0046ED86  |.  EB 05         jmp     short 0046ED8D
0046ED88  |>  33C0          xor     eax, eax
0046ED8A  |.  8945 F8       mov     dword ptr [ebp-8], eax         ;  注册标志得到0，注册失败
0046ED8D  |>  33C0          xor     eax, eax
0046ED8F  |.  5A            pop     edx
0046ED90  |.  59            pop     ecx
0046ED91  |.  59            pop     ecx
0046ED92  |.  64:8910       mov     dword ptr fs:[eax], edx
0046ED95  |.  68 AAED4600   push    0046EDAA
0046ED9A  |>  8D45 FC       lea     eax, dword ptr [ebp-4]
0046ED9D  |.  E8 DA54F9FF   call    0040427C
0046EDA2  \.  C3            retn
```

---

假设输入假注册码：

`1234567890123456`

简单处理之，如果是小写字母则以`FF`代替，否则原样输出：

即：

`01 02 03 04 05 06 07 08 09 00 01 02 03 04 05 06`

简单处理成：

`21 43 65 87 09 21 43 65`

我且依序取名为：

`N1 N2 N3 N4 N5 N6 N7 N8`

简单的`Xor`运算：

- `N1 xor N4==A6` 结果如果是`38`则通过
- `N2 xor N8==26` 结果如果是`6E`则通过
- `N3 xor N6==44` 结果如果是`4E`则通过
- `N5 xor N7==4A` 结果如果是`1A`则通过

判断：

- `A6==38`
- `26==6E`
- `44==4E`
- `4A==1A`

---

于是有简单的注册机：

```text
procedure TForm1.Button2Click(Sender: TObject);

begin

  showmessage('Code by wofan[OCN]'+#13+'QQ:339171218'+#13+'2007-3-27');

end;

procedure TForm1.Button1Click(Sender: TObject);

var

  SN:array[0..15] of byte;

  t:array[0..3] of byte;

  i:integer;

  str:string;

begin

  Randomize;

{

  SN_str:='';

  fillchar(sn,sizeof(sn),0);

  fillchar(t,sizeof(t),0);

}

  SN[0]:=random(10);   //随机生成第某些位置的注册码字符

  SN[1]:=random(10);   //Random(x)将生成一个0<=num<x的数

  SN[2]:=random(10);

  SN[3]:=random(10);

  SN[4]:=random(10);

  SN[5]:=random(10);

  SN[8]:=random(10);

  SN[9]:=random(10);

  t[0]:=(SN[1] shl 4+SN[0]) xor $38;  //捉对厮杀

  t[1]:=(SN[3] shl 4+SN[2]) xor $6E;

  t[2]:=(SN[5] shl 4+SN[4]) xor $4E;

  t[3]:=(SN[9] shl 4+SN[8]) xor $1A;

  SN[7]:=t[0] shr 4;         //从结果中提取出注册字符

  SN[6]:=t[0] and $F;

  SN[15]:=t[1] shr 4;

  SN[14]:=t[1] and $F;

  SN[11]:=t[2] shr 4;

  SN[10]:=t[2] and $F;

  SN[13]:=t[3] shr 4;

  SN[12]:=t[3] and $F;

  for i:=0 to 15 do

    SN_str:=SN_str+inttohex(SN[i],1);

  edit1.Text:=SN_str;      //输出注册码

end;
```

这应该算是一个灰色软件。

用Baidu一搜，居然有很多爆破版，晕死！这个简单的东西怎么要用爆破。软件作者还在里面记录了黑名单！现在好了，用注册机生成无限个注册码，不知这个记录会有多长。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
