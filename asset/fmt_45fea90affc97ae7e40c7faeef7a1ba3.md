
00442EAF  BA68304400        mov    edx, $00443068   

`Reference to: system.@LStrCmp;`   

00442EB4  E84F0DFCFF        call   00403C08   

00442EB9  7558              jnz    00442F13   

00442EBB  6A01              push   $01   

00442EBD  6A00              push   $00   

00442EBF  6A00              push   $00   

00442EC1  8D55F0            lea    edx, [ebp-$10]   

`Reference to TApplication instance`   

00442EC4  A1944C4400        mov    eax, dword ptr [$444C94]   

00442EC9  8B00              mov    eax, [eax]   

`Reference to: forms.TApplication.GetExeName(TApplication):System.AnsiString;`   

00442ECB  E84CC6FFFF        call   0043F51C   

00442ED0  8B45F0            mov    eax, [ebp-$10]   

00442ED3  8A10              mov    dl, byte ptr [eax]   

00442ED5  8D45F4            lea    eax, [ebp-$0C]   

`Reference to: system.@LStrFromChar(String;Char);`   
or: `system.@LStrFromWChar(String;WideChar);`   
or: `system.@WStrFromChar(WideString;Char);`   
or: `system.@WStrFromWChar(WideString;WideChar);`   

00442ED8  E8430BFCFF        call   00403A20   

00442EDD  8D45F4            lea    eax, [ebp-$0C]   

`Possible String Reference to: ':\\browse\\setup.exe'`   

00442EE0  BA78304400        mov    edx, $00443078   

`Reference to: system.@LStrCat;`   

00442EE5  E8160CFCFF        call   00403B00   

00442EEA  8B45F4            mov    eax, [ebp-$0C]   

`Reference to: system.@LStrToPChar;`   

00442EED  E8CA0DFCFF        call   00403CBC   

00442EF2  50                push   eax   

`Possible String Reference to: 'Open'`   

00442EF3  6858304400        push   $00443058   

00442EF8  8BC3              mov    eax, ebx   

`Reference to: controls.TWinControl.GetHandle(TWinControl):Windows.HWND;`   

00442EFA  E8B962FEFF        call   004291B8   

00442EFF  50                push   eax   

`Reference to: shell32.ShellExecuteA()`   

00442F00  E8EFD2FFFF        call   004401F4   

`Reference to TApplication instance`   

00442F05  A1944C4400        mov    eax, dword ptr [$444C94]   

00442F0A  8B00              mov    eax, [eax]   

`Reference to: forms.TApplication.Terminate(TApplication);`   

00442F0C  E8F7C1FFFF        call   0043F108   

00442F11  EB7B              jmp    00442F8E   

00442F13  FF0564584400      inc    dword ptr [$445864] <==单击 Ok 的次数   

00442F19  833D6458440001    cmp    dword ptr [$445864], +$01 <=== 是第 2 次输入密码吗?   

00442F20  750C              jnz    00442F2E  <========== 是则跳转   

`Possible String Reference to: '该序列号无效，请查看输入的是否与 CD 封套上的序列号一致！'`   

00442F22  B894304400        mov    eax, $00443094   

`Reference to: dialogs.ShowMessage(System.AnsiString);`   

00442F27  E8B0F8FFFF        call   004427DC  <==== 提示"密码错!"   

00442F2C  EB60              jmp    00442F8E   

`Possible String Reference to: '欢迎使用图片浏览器的测试版！'`   

00442F2E  B8D4304400        mov    eax, $004430D4   

`Reference to: dialogs.ShowMessage(System.AnsiString);`   

00442F33  E8A4F8FFFF        call   004427DC  <==== 执行冷雨浏览器   

00442F38  6A01              push   $01   

00442F3A  6A00              push   $00   

00442F3C  6A00              push   $00   

00442F3E  8D55F0            lea    edx, [ebp-$10]   

`Reference to TApplication instance`   

00442F41  A1944C4400        mov    eax, dword ptr [$444C94]   

00442F46  8B00              mov    eax, [eax]   

`Reference to: forms.TApplication.GetExeName(TApplication):System.AnsiString;`   

00442F48  E8CFC5FFFF        call   0043F51C   

00442F4D  8B45F0            mov    eax, [ebp-$10]   

00442F50  8A10              mov    dl, byte ptr [eax]   

00442F52  8D45F4            lea    eax, [ebp-$0C]   

`Reference to: system.@LStrFromChar(String;Char);`   
or: `system.@LStrFromWChar(String;WideChar);`   
or: `system.@WStrFromChar(WideString;Char);`   
or: `system.@WStrFromWChar(WideString;WideChar);`   

00442F55  E8C60AFCFF        call   00403A20   

00442F5A  8D45F4            lea    eax, [ebp-$0C]   

`Possible String Reference to: ':\\browse\\setup.exe'`   

00442F5D  BA78304400        mov    edx, $00443078   

`Reference to: system.@LStrCat;`   

00442F62  E8990BFCFF        call   00403B00   

00442F67  8B45F4            mov    eax, [ebp-$0C]   

`Reference to: system.@LStrToPChar;`   

00442F6A  E84D0DFCFF        call   00403CBC   

00442F6F  50                push   eax   

`Possible String Reference to: 'Open'`   

00442F70  6858304400        push   $00443058   

00442F75  8BC3              mov    eax, ebx   

`Reference to: controls.TWinControl.GetHandle(TWinControl):Windows.HWND;`   

00442F77  E83C62FEFF        call   004291B8   

00442F7C  50                push   eax   

`Reference to: shell32.ShellExecuteA()`   

00442F7D  E872D2FFFF        call   004401F4   

`Reference to TApplication instance`   

00442F82  A1944C4400        mov    eax, dword ptr [$444C94]   

00442F87  8B00              mov    eax, [eax]   

`Reference to: forms.TApplication.Terminate(TApplication);`   

00442F89  E87AC1FFFF        call   0043F108   

00442F8E  33C0              xor    eax, eax   

00442F90  5A                pop    edx   

00442F91  59                pop    ecx   

00442F92  59                pop    ecx   

00442F93  648910            mov    fs:[eax], edx   

FINALLY

`Possible String Reference to: '[嬪]?'`   

00442F96  68C02F4400        push   $00442FC0   

00442F9B  8D45EC            lea    eax, [ebp-$14]   

00442F9E  BA03000000        mov    edx, $00000003   

`Reference to: system.@LStrArrayClr;`   

00442FA3  E8F808FCFF        call   004038A0   

00442FA8  8D45F8            lea    eax, [ebp-$08]   

`Reference to: system.@LStrClr(String);`   

00442FAB  E8CC08FCFF        call   0040387C   

00442FB0  8D45FC            lea    eax, [ebp-$04]   

`Reference to: system.@LStrClr(String);`   

00442FB3  E8C408FCFF        call   0040387C   

00442FB8  C3                ret   

00442FB9  E97E03FCFF        jmp    0040333C   

00442FBE  EBDB              jmp    00442F9B   

END

00442FC0  5B                pop    ebx   

00442FC1  8BE5              mov    esp, ebp   

00442FC3  5D                pop    ebp   

00442FC4  C3                ret   

### 8. 看明白了吗?

(摇头)我-------还是--------------不懂!

我倒!!!!!!!!!!

上面的程序用 pascal 表示如下:

```pascal
write('输入序列号: ');
readln(str_inputed_sn);

if string_compare(str_inputed_sn, '238888') then
  display_hidden_files()
else
begin
  int_compare_counter := int_compare_counter + 1;
  if (int_compare_counter >= 2) or string_compare(str_inputed_sn, '3456') then
    display_CoolRain_browser();
end;
```

### 9. 小结

本例的真正序列号是: `238888`

### 10. 做这张光盘的 JS 真是 TMD, 封皮上写着 Delphi 5 正式版, 里面却是 trial 版本

`4dd466d0e1e0cc123afbe504da2462ca`
