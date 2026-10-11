
```
if EasyPDFtoRTF_call(code6_9) = ((EasyPDFtoRTF_call(code1_4) + (EasyPDFtoRTF_call(code1_4) * $5 + $C0E4) * 2 + $2694) shl $1) mod $2710
then Result := code6_9    //正确结果回送
else if Form1.ProgressBar.Progress > 9
then Form1.ProgressBar.Progress := 0
else Form1.ProgressBar.Progress := Form1.ProgressBar.Progress + 1;
end;
```

```
//穷举 16_19 位数          //第二部份外部算法
function QJ16_19(code11_14: string): string;
var
i, pos : integer;
cz, code16_19 : string;
begin
for i := 1000 to 9999 do
begin
code16_19 := int2hex(i, 0);
if EasyPDFtoRTF_call(code16_19) = ((((EasyPDFtoRTF_call(code11_14) - $1A02) xor $0) - $0) * (EasyPDFtoRTF_call(code11_14) + $8E8)) mod $2710
then Result := code16_19    //正确结果回送
else if Form1.ProgressBar.Progress > 9
then Form1.ProgressBar.Progress := 0
else Form1.ProgressBar.Progress := Form1.ProgressBar.Progress + 1;
end;
end;
```

```
//EasyPDFtoRTF 注册算法
function EasyPDFtoRTF_sn(): string; //定义 string 函数
var
code1_4, code6_9, code11_14, code16_19, RegCode : string;
eax, ebx, ebp, ecx, edx, edi, esi : integer;
begin
{
//注册码的格式应为：
RegCode := '1111-1714-6666-1568';
esi := EasyPDFtoRTF_call(copy(RegCode, 1, 4));   //1_4
edi := EasyPDFtoRTF_call(copy(RegCode, 6, 9));
ebx := EasyPDFtoRTF_call(copy(RegCode, 11, 14)); //11_14
ebp := EasyPDFtoRTF_call(copy(RegCode, 16, 19));
//第一部份外部算法
//edx := ((esi + (esi * $5 + $C0E4) * 2 + $2694) shl $1) mod $2710;  //6_9
//第二部份外部算法
//edx := $0;
//edx := ((((ebx - $1A02) xor edx) - edx) * (ebx + $8E8)) mod $2710;
showmessage(int2hex(ebp, 0));
showmessage(int2hex(edx, 0));
}

code1_4   := '1111';             //任意  4  位数//这里我定义的为：1111，你可自己修改，但必须满足能被穷举出来哟！
code6_9   := QJ6_9(code1_4);     //穷举 6_9 位数
code11_14 := '6666';             //任意  4  位数//这里我定义的为：6666，你可自己修改，但必须满足能被穷举出来哟！
code16_19 := QJ16_19(code11_14);//穷举 16_19 位数
Result := code1_4 + '-' + code6_9 + '-' + code11_14 + '-' + code16_19; //最终结果

end;
```

```
procedure TForm1.ButtonRegClick(Sender: PObj);
begin
EditName.Text := EasyPDFtoRTF_sn(); //EasyPDFtoRTF 注册算法
end;
```

```
procedure TForm1.ButtonExitClick(Sender: PObj);
begin
Form.Close;
end;

end.
```

好的，就到这里结束吧！分析难免有不当之处，还望大家给我指出来！

放上几组可用的 Key：

`name: [任意注册名]`
`code: 1111-1714-6666-1568`

`name: [任意注册名]`
`code: 2020-1712-8080-1920`

`name: [任意注册名]`
`code: 2000-1272-7033-2375`

`name: [任意注册名]`
`code: 1122-1956-6665-2615`

`name: [任意注册名]`
`code: 1144-2440-7040-240F`

`name: [任意注册名]`
`code: 1111-1714-8086-2648`

因为想偷懒，注册机的部分算法使用了穷举的方式，应该不存在 bug 的问题，附件中为我打包后的注册机源代码。

也把 `Easy PDF to Html Converter` 的算法跟了下，发现它的子 call 算法没有变，联那张 Key 表的数据都没有变，只是变了下面这些地方：

```
00424D02  |.  8D8CB6 04AB00>lea     ecx, dword ptr [esi+esi*4+AB04]
00424D09  |.  8D844E 342200>lea     eax, dword ptr [esi+ecx*2+2234]
00424D10  |.  B9 10270000   mov     ecx, 2710
00424D15  |.  D1E0          shl     eax, 1
00424D17  |.  99            cdq
00424D18  |.  F7F9          idiv    ecx
00424D1A  |.  3BFA          cmp     edi, edx

00424D29  |> \8D83 CAEAFFFF lea     eax, dword ptr [ebx-1536]
00424D2F  |.  81C3 E2090000 add     ebx, 9E2
00424D35  |.  99            cdq
00424D36  |.  33C2          xor     eax, edx
00424D38  |.  B9 10270000   mov     ecx, 2710
00424D3D  |.  2BC2          sub     eax, edx
00424D3F  |.  C64424 30 04  mov     byte ptr [esp+30], 4
00424D44  |.  0FAFC3        imul    eax, ebx
00424D47  |.  99            cdq
00424D48  |.  F7F9          idiv    ecx
00424D4A  |.  8D4C24 38     lea     ecx, dword ptr [esp+38]
00424D4E  |.  3BEA          cmp     ebp, edx
00424D50  |.  74 63         je      short pdf2html.00424DB5
```

也即是，两部分的算法分别变为了：

`;1
edx := ((esi + (esi * $5 + $AB04) * 2 + $2234) shl $1) mod $2710;

;2
edx := ((((ebx - $1536) xor edx) - edx) * (ebx + $9E2)) mod $2710;`

也给出一组 Key：

`name: 随便`
`code: 1818-2628-8000-2100`

呵呵！
4dd466d0e1e0cc123afbe504da2462ca
