
```
    Table := '717073727574777679780003020504070609080B0A0D0C0F0E111013121514171619181B';
    // 输入机器码
    MachineCode := Text1.Text;
    // 计算注册码
    if  Length(MachineCode)>20 then begin MachineCode :=  Copy(MachineCode, 1, 20); end;
    n :=  Length(MachineCode);
    for i:=1 to n do begin
      tmp :=  Copy(MachineCode, i, 1);
      m :=  (1+Pos(tmp, PChar(Char)+1));
      Sn := Sn+ Copy(Table, 2*m-1, 2);
    end;
    // 输出注册码
    Text2.Text := Sn;
end;
```
\--------------------------------------------------------------------------------

### 【经验总结】
没什么好总结的，自创的“联想分析法”真好用！呵呵~

\--------------------------------------------------------------------------------

### 【版权声明】
本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢!

2007 年 02 月 16 日 PM 02:33:00

`4dd466d0e1e0cc123afbe504da2462ca`
