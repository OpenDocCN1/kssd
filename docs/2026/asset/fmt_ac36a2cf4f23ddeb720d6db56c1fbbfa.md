
```pascal
temp8 := int2hex(ord(cz[8]),2)+int2hex(ord(cz[7]),2)+
int2hex(ord(cz[6]),2)+int2hex(ord(cz[5]),2);
for i:=9 to length(cz) do
  temp12 := temp12 + int2hex(ord(cz[i]),2);
temp12 := temp12 + '00'+'20'+'20';
temp12 := copy(temp12,1,8);
temp12 := copy(temp12,7,2)+copy(temp12,5,2)+copy(temp12,3,2)+copy(temp12,1,2);//反顺序
if length(cz)=12
then temp16 := '20202000'  //如注册名长度为12
else temp16 := '20202020';
end;
13 .. 512: begin //12< 注册名长度 <=512 :
temp4 := int2hex(ord(cz[4]),2)+int2hex(ord(cz[3]),2)+
int2hex(ord(cz[2]),2)+int2hex(ord(cz[1]),2);
temp8 := int2hex(ord(cz[8]),2)+int2hex(ord(cz[7]),2)+
int2hex(ord(cz[6]),2)+int2hex(ord(cz[5]),2);
temp12 := int2hex(ord(cz[12]),2)+int2hex(ord(cz[11]),2)+
int2hex(ord(cz[10]),2)+int2hex(ord(cz[9]), 2);
for i:=13 to length(cz) do
  temp16 := temp16 + int2hex(ord(cz[i]),2);
temp16 := temp16 + '00'+'20'+'20';
temp16 := copy(temp16,1,8);
temp16 := copy(temp16,7,2)+copy(temp16,5,2)+copy(temp16,3,2)+copy(temp16,1,2);//反顺序
end;
else begin //注册名长度 > 512 :
ShowMessage('the length of your name must less than 513!');
Exit;
end;
end;
temp41  := Hex2Int('$' + temp4);//KOL的这个Hex2Int函数好呀!
temp85  := Hex2Int('$' + temp8);
tempB9  := Hex2Int('$' + temp12);
temp10D := Hex2Int('$' + temp16);
Teax := temp10D;
Tecx := temp85;
Tesi := temp41;
Tecx := Tecx + Teax;
Tecx := Tecx + tempB9;
Tesi := Tesi \* $7531;
Tecx := Tecx \* $7531;
Tesi := Tesi - Tecx;
i:=0;  //i的初始值
repeat //开始循环！
Teax := i;
Tecx := Teax;
Tecx := Tecx and $7F;
Tesi := Tesi + key[Tecx];
i := i + 1;
until i > ($0FFF - 1);
Result := Uint2str(Tesi); //返回结果；应为：无符号的十进制数，故用KOL的 Uint2str 函数!
end;
```

好的，就到这里结束吧！分析难免有不当之处，还望大家给我指出来！

放上一组可用的Key：

name: `aCaFeeL`

code: `936627489`

注册机的算法在多台机器上使用不同注册名通过验证，但没有时间再优化它了，应该不存在bug的问题，附件中为我打包后的注册机源代码。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [keygen.for.rb_151plus.src.rar](javascript:void(0);)

  （7.61kb，174次下载）
