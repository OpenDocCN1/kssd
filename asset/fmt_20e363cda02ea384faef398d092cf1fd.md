
```pascal
        $20,$21,$22,$23,$24,$25,$26,$27,$28,$29,$2A,$2B,$2C,$2D,$2E,$2F,

        $30,$31,$32,$33,$34,$35,$36,$37,$38,$39,$3A,$3B,$3C,$3D,$3E,$3F,

        $40,$61,$62,$63,$64,$65,$66,$67,$68,$69,$6A,$6B,$6C,$6D,$6E,$6F,

        $70,$71,$72,$73,$74,$75,$76,$77,$78,$79,$7A,$5B,$5C,$5D,$5E,$5F,

        $60,$61,$62,$63,$64,$65,$66,$67,$68,$69,$6A,$6B,$6C,$6D,$6E,$6F,

        $70,$71,$72,$73,$74,$75,$76,$77,$78,$79,$7A,$7B,$7C,$7D,$7E,$7F,

        $80,$81,$82,$83,$84,$85,$86,$87,$88,$89,$8A,$8B,$8C,$8D,$8E,$8F,

        $90,$91,$92,$93,$94,$95,$96,$97,$98,$99,$9A,$9B,$9C,$9D,$9E,$9F,

        $A0,$A1,$A2,$A3,$A4,$A5,$A6,$A7,$A8,$A9,$AA,$AB,$AC,$AD,$AE,$AF,

        $B0,$B1,$B2,$B3,$B4,$B5,$B6,$B7,$B8,$B9,$BA,$BB,$BC,$BD,$BE,$BF,

        $C0,$C1,$C2,$C3,$C4,$C5,$C6,$C7,$C8,$C9,$CA,$CB,$CC,$CD,$CE,$CF,

        $D0,$D1,$D2,$D3,$D4,$D5,$D6,$D7,$D8,$D9,$DA,$DB,$DC,$DD,$DE,$DF,

        $E0,$E1,$E2,$E3,$E4,$E5,$E6,$E7,$E8,$E9,$EA,$EB,$EC,$ED,$EE,$EF,

        $F0,$F1,$F2,$F3,$F4,$F5,$F6,$F7,$F8,$F9,$FA,$FB,$FC,$FD,$FE,$FF);

begin

  if RegName='' then

    begin

      showmessage('输入的 注册名 不能为空!');

      exit;

    end;

  for i:=0 to 3 do sum[i] := $00; //初始化变量 sum

  ii := 0; //初始化变量 ii

  for i:=1 to length(RegName) do  //开始循环计算:

    begin

      if (ord(RegName[i])=$20) or (ord(RegName[i])=$2E)

      then ii := (ii and $3)

      else begin

               sum[ii] := sum[ii] + key[ord(RegName[i]) and $FF];//取字符表中的相应值

               ii := ii + 1;

               ii := (ii and $3);

             end;

  end;

  sumAll := int2hex(sum[3],2)+int2hex(sum[2],2)+int2hex(sum[1],2)+int2hex(sum[0],2);

  TempA  := Hex2Int('$' + sumAll);        //得到的前部分的结果(RegName -> TempA)


  Key1 := hex2int('$' + KeyCode1);

  Tecx := Key1;

  Tepb := $2;

  Tedi := Key1;

  goto cz59;

cz50:

  Tepb := Tepb shl $1;

  if (Tebp - 0) = 0 then goto cz9F;

cz59:

  Tesi := $F527789F;

  Teax := Tedi;

  Tedx := Tedi;

  // Tedx_eax := Teax * Tedx;

  asm

    MOV  ESI,[Tesi]

    MOV  EAX,[Teax]

    MOV  EDX,[Tedx]

    MUL  EDX

    mov  [Teax],EAX

    mov  [Tedx],EDX

  end;

  if Tesi > Tedx then goto cz73;

  Tbex := Teax;

  Teax := Tedx;

  Tedx := Tedx xor Tedx;

  //Tedx := Tedx_eax mod Tesi;

  asm

    MOV  ESI,[Tesi]

    MOV  EAX,[Teax]

    MOV  EDX,[Tedx]

    DIV  ESI

    mov  [Teax],EAX

    mov  [Tedx],EDX

  end;

  Teax := Tbex;

cz73:

  //Tedx := Tedx_eax mod Tesi;

  asm

    MOV  ESI,[Tesi]

    MOV  EAX,[Teax]

    MOV  EDX,[Tedx]

    DIV  ESI

    mov  [Teax],EAX

    mov  [Tedx],EDX

  end;

  Tesi := $E8B8D413;

  Teax := Tedx;

  Tedi := Tedx;

  if (Tebp and Tesi)=0 then goto cz50;

  Tesi := $F527789F;

  Tedx := Tecx;

  //Tedx_eax := Teax * Tedx;

  asm

    MOV  ESI,[Tesi]

    MOV  EAX,[Teax]

    MOV  EDX,[Tedx]

    MUL  EDX

    mov  [Teax],EAX

    mov  [Tedx],EDX

  end;

  if Tesi > Tedx then goto cz99;

  Tbex := Teax;

  Teax := Tedx;

  Tedx := Tedx xor Tedx;

  //Tedx := Tedx_eax mod Tesi;

  asm

    MOV  ESI,[Tesi]

    MOV  EAX,[Teax]

    MOV  EDX,[Tedx]

    DIV  ESI

    mov  [Teax],EAX

    mov  [Tedx],EDX

  end;

cz99:

  //Tedx := Tedx_eax mod Tesi;

  asm

    MOV  ESI,[Tesi]

    MOV  EAX,[Teax]

    MOV  EDX,[Tedx]

    DIV  ESI

    mov  [Teax],EAX

    mov  [Tedx],EDX

  end;

  Tecx := Tedx;

  goto cz50;

cz9F:

  TempB := Tecx;                      //得到的后部分的结果(KeyCode1 ->TempB)


  Result := int2hex((TempA xor TempB),8); //最终返回的 KeyCode2 结果是由 TempA xor TempB 后得到的

end;


```

好了，就到这里结束吧！分析难免有不当之处，还望大家给我指出来！

放上一组可用的 Key：

```
name: aCaFeeL
key1: ABF8FB42
key2: 8DB47F89
```

注册机的算法在多台机器上使用不同注册名通过验证，但没有时间再优化它了，应该不存在 bug 的问题，附件中为我打包后的注册机源代码。

另：如果问我这次分析最大的收获是什么？我会说，那就是学会了如何将大数运算的高位数和低位数分配到相应的 `EDX` 和 `EAX` 中了！别笑，呵呵！

`4dd466d0e1e0cc123afbe504da2462ca`
