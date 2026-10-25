
```assembly
:00424033 C3                      ret
```

<--上面的比较看上去很乱,跟一下很容易明白,为了盘和我的手的寿命,略去注释了,总之是把假码同上面那个`SVP1001234567890123456789`经过计算后得到的字串比较,相同的话就成功.

注册机:(Borland Pascal 7.0)

```pascal
Program CrackSWF2Video;

var st,code,mac:string;

    len,p,EAX,ECX,EDX,EDI,ESI:longint;

begin

     st:='4YKCV3Q85MT7EXWDF9SNZGA2BHRPJU6';

     code:='SVP1001234567890123456789';

     write('Please input your machine number:');

     readln(mac);

     len:=length(mac);

     for p:=1 to len do

     begin

     if (mac[p]>='a') and (mac[p]<='z') then mac[p]:=chr(ord(mac[p])+3)

     else if (mac[p]>='A') and (mac[p]<='Z') then mac[p]:=chr(ord(mac[p])+$23)

     else if (mac[p]>='0') and (mac[p]<='9') then mac[p]:=chr(ord(mac[p])+$4E);

     EDI:=EDI+ord(mac[p]);

     end;

     EDI:=EDI+len;

     EDI:=EDI mod $1F;

     ECX:=5;

     repeat

           EAX:=EDI;

           EDX:=EAX mod $1F;

           EAX:=ECX;

           code[ECX+1]:=st[EDX+1];

           EDX:=EAX mod len;

           ESI:=len-EDX;

           EAX:=ord(mac[EDX+1]);

           EAX:=EAX\*ESI\*ESI;

           EAX:=EAX shr 1;

           EDI:=EAX+EDI\*2;

           inc(ECX);

     until ECX=$19;

     writeln(code);

end.  {那一段计算太乱了,所以我干脆直接把汇编翻译了过来 :) }
```

我的机器码:3930761255

我的注册码:`SVP10-MHE73-XJ3D9-TPFNG-W6M2K`

注:分隔符可以随意输入,但注册成功后可以看到是五个字符一节的形式.
