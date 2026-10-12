
# Txt2Html V2.7.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6778.htm`](https://www.pediy.com/kssd/pediy06/pediy6778.htm)

`Software:Txt2Html V2.7.2`

将文本文件转换成 html 文件

[`alexey-pr.pisem.net`](http://alexey-pr.pisem.net)

`Tools:TRW2000`，微软 Win98

`Cracker:lq7972[bruceyu13@sina.com]`

**Notes**: 这段时间整理敲入的 TXT 文本，用上这个工具，觉得不错，顺手 PJ 了；不知有人写这个否？不管了，贴出来，主要还是交流、学习～

## 操作记录

载入主程序

`g`

打开 `About` 窗口，输入用户名和序列号：“1234506789”

`Ctrl+N`

`bpx hmemcpy`

`g`

点"注册"按钮，拦住

`bc *`

`pmodule`

来到 `TXT2HTML` 领空，按 6 次 `F12`，按 `F10` 来到：

```
019F:0049B817 8B45D4           MOV      EAX,[EBP-2C]           用户名
019F:0049B81A BAE0BC4900       MOV      EDX,0049BCE0
019F:0049B81F E8D893F6FF       CALL     00404BFC
019F:0049B824 0F8507020000     JNZ      NEAR 0049BA31

;。。。。。。

019F:0049BA31 8D55FC           LEA      EDX,[EBP-04]
019F:0049BA34 8B8618030000     MOV      EAX,[ESI+0318]
019F:0049BA3A E8959AFAFF       CALL     004454D4
019F:0049BA3F 8D45C8           LEA      EAX,[EBP-38]
019F:0049BA42 50               PUSH     EAX
019F:0049BA43 B902000000       MOV      ECX,02                  一次取 2 位
019F:0049BA48 BA02000000       MOV      EDX,02                  从第 2 位起取
019F:0049BA4D 8B45FC           MOV      EAX,[EBP-04]            输入的注册码
019F:0049BA50 E8BB92F6FF       CALL     00404D10                取出
019F:0049BA55 8B45C8           MOV      EAX,[EBP-38]            第 2、3 位
019F:0049BA58 E897D5F6FF       CALL     00408FF4                把取出的字符转为数值～a1
019F:0049BA5D 8BD8             MOV      EBX,EAX

;下面以此依次取第 4、5 位、第 6、7 位和第 8、9 位，并将它们转为数值～a2、a3、a4

019F:0049BA5F 8D45C4           LEA      EAX,[EBP-3C]
019F:0049BA62 50               PUSH     EAX
019F:0049BA63 B902000000       MOV      ECX,02
019F:0049BA68 BA04000000       MOV      EDX,04
019F:0049BA6D 8B45FC           MOV      EAX,[EBP-04]
019F:0049BA70 E89B92F6FF       CALL     00404D10
019F:0049BA75 8B45C4           MOV      EAX,[EBP-3C]
019F:0049BA78 E877D5F6FF       CALL     00408FF4
019F:0049BA7D 8BF8             MOV      EDI,EAX
019F:0049BA7F 8D45C0           LEA      EAX,[EBP-40]
019F:0049BA82 50               PUSH     EAX
019F:0049BA83 B902000000       MOV      ECX,02
019F:0049BA88 BA06000000       MOV      EDX,06
019F:0049BA8D 8B45FC           MOV      EAX,[EBP-04]
019F:0049BA90 E87B92F6FF       CALL     00404D10
019F:0049BA95 8B45C0           MOV      EAX,[EBP-40]
019F:0049BA98 E857D5F6FF       CALL     00408FF4
019F:0049BA9D 8945F8           MOV      [EBP-08],EAX
019F:0049BAA0 8D45BC           LEA      EAX,[EBP-44]
019F:0049BAA3 50               PUSH     EAX
019F:0049BAA4 B902000000       MOV      ECX,02
019F:0049BAA9 BA08000000       MOV      EDX,08
019F:0049BAAE 8B45FC           MOV      EAX,[EBP-04]
019F:0049BAB1 E85A92F6FF       CALL     00404D10
019F:0049BAB6 8B45BC           MOV      EAX,[EBP-44]
019F:0049BAB9 E836D5F6FF       CALL     00408FF4
019F:0049BABE 8945F4           MOV      [EBP-0C],EAX
019F:0049BAC1 8B45FC           MOV      EAX,[EBP-04]
019F:0049BAC4 E8EF8FF6FF       CALL     00404AB8                输入的注册码长度
019F:0049BAC9 83F80A           CMP      EAX,BYTE +0A            若小于 10，
019F:0049BACC 0F857C010000     JNZ      NEAR 0049BC4E           就 GameOver
019F:0049BAD2 83C30D           ADD      EBX,BYTE +0D            a1+Dh
019F:0049BAD5 7105             JNO      0049BADC                不溢出就跳
019F:0049BAD7 E8F87EF6FF       CALL     004039D4
019F:0049BADC 83C709           ADD      EDI,BYTE +09            a2+9h
019F:0049BADF 7105             JNO      0049BAE6
019F:0049BAE1 E8EE7EF6FF       CALL     004039D4
019F:0049BAE6 3BDF             CMP      EBX,EDI                 二者若不相等，
019F:0049BAE8 0F8560010000     JNZ      NEAR 0049BC4E           就 GameOver
019F:0049BAEE 8B45F8           MOV      EAX,[EBP-08]            a3
019F:0049BAF1 83C015           ADD      EAX,BYTE +15            a3+15h
019F:0049BAF4 7105             JNO      0049BAFB
019F:0049BAF6 E8D97EF6FF       CALL     004039D4
019F:0049BAFB 8B55F4           MOV      EDX,[EBP-0C]            a4
019F:0049BAFE 83C205           ADD      EDX,BYTE +05            a4+5h
019F:0049BB01 7105             JNO      0049BB08
019F:0049BB03 E8CC7EF6FF       CALL     004039D4
019F:0049BB08 3BC2             CMP      EAX,EDX                 二者若不相等，
019F:0049BB0A 0F853E010000     JNZ      NEAR 0049BC4E           就 GameOver
019F:0049BB10 B201             MOV      DL,01

;下面写注册信息
;。。。。。。

019F:0049BB7C B8CCBD4900       MOV      EAX,0049BDCC
019F:0049BB81 E87628FAFF       CALL     0043E3FC                成功注册
;。。。。。。

019F:0049BC4C EB2F             JMP      SHORT 0049BC7D
019F:0049BC4E 6A00             PUSH     BYTE +00
019F:0049BC50 668B0DC0BD4900   MOV      CX,[0049BDC0]
019F:0049BC57 B202             MOV      DL,02
019F:0049BC59 B830BE4900       MOV      EAX,0049BE30
019F:0049BC5E E89927FAFF       CALL     0043E3FC                注册失败
019F:0049BC63 33D2             XOR      EDX,EDX
019F:0049BC65 8B8614030000     MOV      EAX,[ESI+0314]
019F:0049BC6B E89498FAFF       CALL     00445504
019F:0049BC70 33D2             XOR      EDX,EDX
019F:0049BC72 8B8618030000     MOV      EAX,[ESI+0318]
019F:0049BC78 E88798FAFF       CALL     00445504
019F:0049BC7D 33C0             XOR      EAX,EAX
019F:0049BC7F 5A               POP      EDX
019F:0049BC80 59               POP      ECX
019F:0049BC81 59               POP      ECX
019F:0049BC82 648910           MOV      [FS:EAX],EDX
;。。。。。。
```

## 总结

注册算法还是比较简单。下面给出注册机(Delphi)

```
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

//The KeyGen by lq7972,with Delphi
//E-mail:bruceyu13@sina.com

procedure TForm1.Button1Click(Sender: TObject);

var
  S,S1,S2,S3,S4:String;

begin
  S1:=IntToStr(Random(9))+IntToStr(Random(9));
  S2:=IntToStr(StrToInt(S1)+$D-$9);
  S3:=IntToStr(Random(9))+IntToStr(Random(9));
  S4:=IntToStr(StrToInt(S3)+$15-$5);
  If Length(S2) < 2 then
    S2:='0'+S2;
  S:=S1+S2+S3+S4;
  Edit1.text:=IntToStr(Random(9))+S+IntToStr(Random(9));
end;

//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

`4dd466d0e1e0cc123afbe504da2462ca`
