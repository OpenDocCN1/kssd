
\* Reference To: `VCL30.System.@LStrCat@51F89FF7`, Ord:0000h

            |

:004B9536 `E8557CF4FF`              Call

00401190              <-----String catenating

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B9501(U)

|

:004B953B `43`

    inc ebx

:004B953C `4E`

    dec esi

:004B953D `0F855EFFFFFF`            jne 004B94A1

          <---jump if SN[i]<>nil

上面的程序无非做了这些工作，把Serial Number的字符串中的每一个字符变成整数，然后乘以2F，结果再转换为字符串，以便连结字符串。我的Serial Number=`26313818`，按以上的规律则变成这样:

`IntToStr(2*2F)+IntToStr(6*2F)+IntToStr(3*2F)+IntToStr((1+3)*0x51)+IntToStr(3*2F)+IntToStr(8*2F)+IntToStr((1+3)*0x51)+IntToStr(8*2F)=>"94282141324141376324376"`

<--Now we call this StrA

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B9496(C)

|

:004B9543 `8B45F8`

mov eax, dword ptr [ebp-08] <---D eax, You'll Look StrA

\* Reference To: `VCL30.SysUtils.StrToFloat@044134E0`, Ord:0000h

            |

:004B9546 `E86D81F4FF`              Call

004016B8              <---StrA convert

To Float

:004B954B `DB7DEE`

fstp tbyte ptr [ebp-12]    <---Store at ebp-12

:004B954E `9B`

    wait

:004B954F `DB6DEE`

fld tbyte ptr [ebp-12]    <---Load StrA

:004B9552 `DB2DE4954B00`            fld tbyte

ptr [004B95E4]  <---Load some Float value

:004B9558 `DEC9`

  fmulp st(1), st(0)        <---st(0)=st(1)*st(0)

:004B955A `DB2DF0954B00`            fld tbyte

ptr [004B95F0]  <---Load another value

:004B9560 `DEE9`

  fsubp st(1), st(0)        <---st(0)=st(1)-st(0)

:004B9562 `DB7DEE`

fstp tbyte ptr [ebp-12]    <---Save st(0)

:004B9565 `9B`

    wait

:004B9566 `EB1A`

  jmp 004B9582

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B9590(C)

|

:004B9568 `8B45FC`

mov eax, dword ptr [ebp-04]

\* Reference To: `VCL30.SysUtils.StrToInt@0F6FDFF6`, Ord:0000h

            |

:004B956B `E83880F4FF`              Call

004015A8

:004B9570 `8945E4`

mov dword ptr [ebp-1C], eax <---eax=Serial Number

:004B9573 `DB45E4`

fild dword ptr [ebp-1C]    <---Load it

:004B9576 `DB6DEE`

fld tbyte ptr [ebp-12]      <---Load st(0)

\* Reference To: `VCL30.System.@FSafeDivideR@51F89FF7`, Ord:0000h

            |

:004B9579 `E8EA7CF4FF`              Call

00401268              <---st(0)=st(1)/st(0)

:004B957E `DB7DEE`

fstp tbyte ptr [ebp-12]    <---Save *result*

:004B9581 `9B`

    wait

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B9566(U)

|

:004B9582 `DB6DEE`

fld tbyte ptr [ebp-12]    <---Load st(0)

:004B9585 `DB2DFC954B00`            fld tbyte

ptr [004B95FC]  <---Load value

:004B958B `DED9`

  fcompp

  <---Compare

:004B958D `DFE0`

  fstsw ax

<---Store Status Word

:004B958F `9E`

    sahf

  <---Store AH Register into FLAGS

:004B9590 `72D6`

  jb 004B9568

:004B9592 `668B45F6`

mov ax, word ptr [ebp-0A]

:004B9596 `50`

    push eax

:004B9597 `FF75F2`

push [ebp-0E]

:004B959A `FF75EE`

push [ebp-12]

:004B959D `8B4508`

mov eax, dword ptr [ebp+08]

:004B95A0 `50`

    push eax

:004B95A1 `33C9`

  xor ecx, ecx

:004B95A3 `BA12000000`              mov edx,

00000012

:004B95A8 `B002`

  mov al, 02

\* Reference To: `VCL30.SysUtils.FloatToStrF@0DD792DD`, Ord:0000h

            |

:004B95AA `E80181F4FF`              Call

004016B0

:004B95AF `33C0`

  xor eax, eax

:004B95B1 `5A`

    pop edx

:004B95B2 `59`

    pop ecx

:004B95B3 `59`

    pop ecx

:004B95B4 `648910`

mov dword ptr fs:[eax], edx

:004B95B7 `68D9954B00`              push

004B95D9

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B95D7(U)

|

:004B95BC `8D45E8`

lea eax, dword ptr [ebp-18]

\* Reference To: `VCL30.System.@LStrClr@40929B27`, Ord:0000h

            |

:004B95BF `E87C7BF4FF`              Call

00401140

:004B95C4 `8D45F8`

lea eax, dword ptr [ebp-08]

:004B95C7 `BA02000000`              mov edx,

00000002

\* Reference To: `VCL30.System.@LStrArrayClr@51F89FF7`, Ord:0000h

            |

:004B95CC `E8777BF4FF`              Call

00401148

:004B95D1 `C3`

    ret

其实上面的:004B957E DB7DEE
  fstp tbyte ptr [ebp-12]    <---Save *result*
中的结果就是正确的注册码了。

1、注册码与Username,Organisation无关，只与Serial Number有关。

2、注册过程：

```pascal
procedure TForm1.Button3Click(Sender: TObject);
var I: Integer;  MyString,StrA: String;
begin
MyString:=edit1.text;
  i:=1;
  while I <=Length(MyString) do
  begin
  if StrToInt(MyString[I])<2 then
  StrA :=StrA+IntToStr((StrToInt(MyString[I])+3)*81)
else
    StrA :=StrA+IntToStr(StrToInt(MyString[I])*47);
    I := I + 1;
  end;
edit2.text:=IntToStr(Round((StrToFloat(StrA)*0.1428571428571428571-480547639)/StrToInt(MyString))
);
end;
```

3、该程序注册后信息在注册表中如下:

```
REGEDIT4

[HKEY_LOCAL_MACHINE\SOFTWARE\Simply Super Software\Trojan Remover\User]

"Name"="CoolBob"

"Organisation"="China Cracker Group"

"Serial Number"=hex:00,00,00,a0,45,18,79,41

"Registration"=hex:f0,32,4c,04,7b,18,fd,42
```

-----------------------------------------------------------written by CoolBob[CCG]

2001.4.16
