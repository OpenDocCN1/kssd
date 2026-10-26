
近来正好在自学Delphi编程，就顺手自己写个注册机，关键源码公开如下（Delphi 7编译通过）：

```
procedure TForm1.btn1Click(Sender: TObject);

var

b,c,Temp:integer;

begin

    randomize; //初始化随机数

    repeat

    b:=10000+Random(90000);//算出一个随机5位数，即N1

    c:=10000+Random(90000);//算出一个N2

    Temp:=16816 xor (b xor c);//计算出N3

    until Length(IntToStr(Temp))=5;//经测试发现N3必须是5位，否则会提示注册错误

    Form1.Edit1.Text:='Senhuanpediy'+ inttostr(b) + '-' + inttostr(c) + '-' + inttostr(Temp);

end;



end.
```

注意：软件的注册信息保存在这里

```
Windows Registry Editor Version 5.00

[HKEY\_LOCAL_MACHINE\SOFTWARE\1KeyClean]

"LastDate"="07-26-2008 11:11"

"VerifyKey"="playboysen8419876-54321-55333"
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [`Pc-washer.rar`](javascript:void(0);)

  （829.57kb，146次下载）

- [`注册机及其源码.rar`](javascript:void(0);)

  （214.24kb，141次下载）
