
```
if RegName = '' then      //注册名不能为空!  
  begin  
    ShowMessage('first, you need input your name! :)');  
    Exit;  
  end;  

  cz := LowerCase(RegName); //注册名全部转为小写!  

  //注册名的长度可以为: (4x4)x32 = 512 位数的长度!  
  case length(cz) of        //分 5 种情况的不同对待  
    1 .. 4: begin //0< 注册名长度 <=4 :  
                for i:=1 to length(cz) do  
                  temp4 := temp4 + int2hex(ord(cz[i]),2);  
                temp4 := temp4 + '00'+'20'+'20';  
                temp4 := copy(temp4,1,8);  
                temp4 := copy(temp4,7,2)+copy(temp4,5,2)+copy(temp4,3,2)+copy(temp4,1,2);//反顺序  

                if length(cz)=4  
                then temp8 := '20202000'  //如注册名长度为 4  
                else temp8 := '20202020';  

                temp12 := '20202020';  
                temp16 := '20202020';  
              end;  
    5 .. 8: begin //4< 注册名长度 <=8 :  
                temp4 := int2hex(ord(cz[4]),2)+int2hex(ord(cz[3]),2)+  
                          int2hex(ord(cz[2]),2)+int2hex(ord(cz[1]),2);  

                for i:=5 to length(cz) do  
                  temp8 := temp8 + int2hex(ord(cz[i]),2);  
                temp8 := temp8 + '00'+'20'+'20';  
                temp8 := copy(temp8,1,8);  
                temp8 := copy(temp8,7,2)+copy(temp8,5,2)+copy(temp8,3,2)+copy(temp8,1,2);//反顺序  

                if length(cz)=8  
                then temp12 := '20202000'  //如注册名长度为 8  
                else temp12 := '20202020';  

                temp16 := '20202020';  
              end;  
     9 .. 12: begin //8< 注册名长度 <=12 :  
                temp4 := int2hex(ord(cz[4]),2)+int2hex(ord(cz[3]),2)+  
                          int2hex(ord(cz[2]),2)+int2hex(ord(cz[1]),2);  

                temp8 := int2hex(ord(cz[8]),2)+int2hex(ord(cz[7]),2)+  
                          int2hex(ord(cz[6]),2)+int2hex(ord(cz[5]),2);  

                for i:=9 to length(cz) do  
                  temp12 := temp12 + int2hex(ord(cz[i]),2);  
                temp12 := temp12 + '00'+'20'+'20';  
                temp12 := copy(temp12,1,8);  
                temp12 := copy(temp12,7,2)+copy(temp12,5,2)+copy(temp12,3,2)+copy(temp12,1,2);//反顺序  

                if length(cz)=12  
                then temp16 := '20202000'  //如注册名长度为 12  
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
  else        begin //注册名长度 > 512 :  
                ShowMessage('the length of your name must less than 513!');  
                Exit;  
              end;  
  end;  

  temp41  := Hex2Int('$' + temp4);//KOL 的这个 Hex2Int 函数好呀!  
  temp85  := Hex2Int('$' + temp8);  
  tempB9  := Hex2Int('$' + temp12);  
  temp10D := Hex2Int('$' + temp16);  
  Teax := temp10D;  
  Tecx := temp85;  
  Tesis := temp41;  

  Tecx := Tecx + Teax;  
  Tecx := Tecx + tempB9;  
  Tesis := Tesis * $7531;  
  Tecx := Tecx * $7531;  
  Tesis := Tesis - Tecx;  

  i:=0;  //i 的初始值  
  repeat //开始循环！  
    Teax := i;  
    Tecx := Teax;  
    Tecx := Tecx and $7F;  
    Tesis := Tesis + key[Tecx];  
    i := i + 1;  
  until i > ($0FFF - 1);  
  
  Result := Uint2str(Tesis); //返回结果；应为：无符号的十进制数，故用 KOL 的 Uint2str 函数!  
end;
```

好的，就到这里结束吧！分析难免有不当之处，还望大家给我指出来！

放上一组可用的 Key：

name: `aCaFeeL`
code: `936627489`

注册机的算法在多台机器上使用不同注册名通过验证，但没有时间再优化它了，应该不存在 bug 的问题，附件中为我打包后的注册机源代码。

`4dd466d0e1e0cc123afbe504da2462ca`
