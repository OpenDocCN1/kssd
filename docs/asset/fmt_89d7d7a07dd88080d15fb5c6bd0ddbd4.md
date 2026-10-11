
```
019F:00436F5B 85C9             TEST     ECX,ECX
019F:00436F5D 7E77             JNG      00436FD6
019F:00436F5F 8A0C06           MOV      CL,[ESI+EAX]
019F:00436F62 6A01             PUSH     BYTE +01
019F:00436F64 884C2444         MOV      [ESP+44],CL
019F:00436F68 8B542444         MOV      EDX,[ESP+44]
019F:00436F6C 52               PUSH     EDX
019F:00436F6D 8D4C242C         LEA      ECX,[ESP+2C]
019F:00436F71 E890BE0200       CALL     00462E06
019F:00436F76 8B00             MOV      EAX,[EAX]
019F:00436F78 50               PUSH     EAX
019F:00436F79 FFD7             CALL     EDI
019F:00436F7B 8D4C2424         LEA      ECX,[ESP+24]
019F:00436F7F 8BE8             MOV      EBP,EAX
019F:00436F81 E8893D0300       CALL     0046AD0F
019F:00436F86 8B4C2410         MOV      ECX,[ESP+10]
019F:00436F8A 6A01             PUSH     BYTE +01
019F:00436F8C 8B41F8           MOV      EAX,[ECX-08]
019F:00436F8F 2BC6             SUB      EAX,ESI
019F:00436F91 8A4408FF         MOV      AL,[EAX+ECX-01]
019F:00436F95 88442424         MOV      [ESP+24],AL
019F:00436F99 8B4C2424         MOV      ECX,[ESP+24]
019F:00436F9D 51               PUSH     ECX
019F:00436F9E 8D4C2424         LEA      ECX,[ESP+24]
019F:00436FA2 E85FBE0200       CALL     00462E06
019F:00436FA7 8B00             MOV      EAX,[EAX]
019F:00436FA9 50               PUSH     EAX
019F:00436FAA FFD7             CALL     EDI
019F:00436FAC 8D4C241C         LEA      ECX,[ESP+1C]
019F:00436FB0 8BD8             MOV      EBX,EAX
019F:00436FB2 E8583D0300       CALL     0046AD0F
019F:00436FB7 8D54AD00         LEA      EDX,[EBP+EBP*4+00]
019F:00436FBB 8D4C2418         LEA      ECX,[ESP+18]
019F:00436FBF 8A845310494A00   MOV      AL,[EBX+EDX*2+004A4910]
;。。。。。。
019F:00436FE5 E8BA3F0300       CALL     0046AFA4                         在前 10 位后添"-"
019F:00436FEA 8D4C2418         LEA      ECX,[ESP+18]
019F:00436FEE 8D542440         LEA      EDX,[ESP+40]
019F:00436FF2 51               PUSH     ECX
019F:00436FF3 B304             MOV      BL,04
019F:00436FF5 50               PUSH     EAX
019F:00436FF6 52               PUSH     EDX
019F:00436FF7 885C2440         MOV      [ESP+40],BL
019F:00436FFB E83E3F0300       CALL     0046AF3E                         连成完整的注册码
;。。。。。。
```

## 【表】

敲 `d 004A48AC,004A497C >c:\Tab.txt` 得到：

```
0030:004A48AC 30 31 32 33 34 35 36 37-38 39 64 72 65 61 6d 73 0123456789dreams
0030:004A48BC 6f 66 74 73 72 74 79 6a-71 70 36 30 71 6e 65 65 oftsrtyjqp60qnee
0030:004A48CC 72 66 73 62 4a 6b 6c 6d-61 66 68 6a 6e 66 68 6a rfsbJklmafhjnfhj
0030:004A48DC 6b 6e 6d 67 63 67 72 6b-74 62 6f 6f 6c 68 68 69 knmgcgrktboolhhi
0030:004A48EC 76 6a 67 74 63 70 78 69-6d 6a 75 74 72 76 78 66 vjgtcpximjutrvxf
0030:004A48FC 6c 64 79 77 65 6c 77 62-7a 6b 6d 37 35 34 33 32 ldywelwbzkm75432
0030:004A490C 31 30 68 65 23 40 25 5b-5e 5d 7b 7d 21 78 7e 21 10he#@%[^]{}!x~!
0030:004A491C 7e 61 68 73 3c 3e 3f 7c-73 40 23 6e 62 76 63 78 ~ahs<>?|s@#nbvcx
0030:004A492c 7a 65 23 2d 2b 21 2b 35-21 23 7c 3d 3d 24 2f 40 ze#-+!+5!#|==$/@
0030:004A493c 21 2f 39 35 33 36 39 40-37 32 7c 23 2d 2a 40 34 !/95369@72|#-*@4
0030:004A494c 31 2d 34 3d 35 36 2a 2f-24 2b 40 30 30 34 2a 2d 1-4=56\*/$+@004*-
0030:004A495c 24 21 2f 33 33 3d 23 39-7c 32 3d 35 40 30 2a 24 $!/33=#9|2=5@0*$
0030:004A496c 7c 33 67 6b 31 23 37 2b-b8 d0 d0 bb c4 e3 b5 c4 |3gk1#7+感谢你的
```

## 【总结】

为了节省篇幅，略掉了很多行；这个软件在路上搞了 N 多 `call`，如果见 `CALL` 就跟进，那真的会精神崩溃。。。看雪老大指出过：

1.  **对软件进行粗跟踪**
    所谓粗跟踪，即在跟踪时要大块大块地跟踪，也就是说每次遇到调用 `CALL` 指令、重复操作指令 `REP`、循环操作 `LOOP` 指令以及中断调用 `INT` 指令等，一般不要跟踪进去，而是根据执行结果分析该段程序的功能。

2.  **对关键部分进行细跟踪**
    对软件进行了一定程度的粗跟踪之后，便可以获取软件中我们所关心的模块或程序段，这样就可以针对性地对该模块进行具体而详细地跟踪分析。一般情况下，对关键代码的跟踪可能要反复进行若干次才能读懂该程序，每次要把比较关键的中间结果或指令地址记录下来，这样会对下一次分析有很大的帮助。(看雪教程·基础知识之分析技术)

## 【注册机】

```delphi
//<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
//迷你梦幻日记本 2003 注册机 bY lq7972，with Delphi 6
//bruceyu13@sina.com

procedure TForm1.Button1Click(Sender: TObject);

var
  i,l:integer;
  C1,C2:Cardinal;
  C3:String;
  C4,C5:Integer;
  T1,T2:String;

const
  C:String='410704800313101lxl80';
  A:String='0123456789dreamsoftsrtyjqp60qneerfsbJklmafhjnfhjknmgcgrktboolhhivjgtcpximjutrvxfldywelwbzkm7543210he#@%[^]{}!x~!~ahs<>?|s@#nbvcxze#-+!+5!#|==$/@!/95369@72|#-*@41-4=56\*/$+@004*-$!/33=#9|2=5@0*$|3gk1#7+';

begin
  C1:=StrToInt(Edit1.Text);

  for i:=1 to length(C) do
  begin
    C1:=C1 Xor ((Ord(C[i])-$78) And $FF);
  end;

  C2:=Sqr(C1)*$a9baef0d;
  C3:=IntToStr(C2);
  l:=Length(C3);
  for i:=1 to l do
  begin
    C4:=(StrToInt(Char(C3[i]))*5)*2+StrToInt(Char(C3[l-i+1]))+1;
    C5:=(StrToInt(Char(C3[i]))*5)*2+StrToInt(Char(C3[l-i+1]))+101;
    T1:=T1+A[C4];
    T2:=T2+A[C5];
  end;

  Edit2.text:=T1+'-'+T2;
end;

//<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
```

4dd466d0e1e0cc123afbe504da2462ca
