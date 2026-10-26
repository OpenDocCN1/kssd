
```asm
005A4828 >|.  E8 5B06E6FF      call NetUSB_e.00404E88               ;  ->System.@LStrToPChar(String):PAnsiChar;
005A482D  |.  8BD0             mov edx,eax
005A482F  |.  A1 4C2A5C00      mov eax,dword ptr ds:[5C2A4C]
005A4834  |.  8B00             mov eax,dword ptr ds:[eax]
005A4836  |.  59               pop ecx
005A4837 >|.  E8 B4ABECFF      call NetUSB_e.0046F3F0               ;  ->Forms.TApplication.MessageBox(TApplication;PChar;PChar;Longint):Integer;
005A483C >|.  8B83 F8020000    mov eax,dword ptr ds:[ebx+2F8]       ;  \*LEt\_RegUser:N.A.
005A4842  |.  8B10             mov edx,dword ptr ds:[eax]
005A4844  |.  FF92 C0000000    call dword ptr ds:[edx+C0]
005A484A  |.  E9 9F010000      jmp NetUSB_e.005A49EE
005A484F  |>  B2 01            mov dl,1
005A4851 >|.  8B83 F8020000    mov eax,dword ptr ds:[ebx+2F8]       ;  \*LEt\_RegUser:N.A.
005A4857 >|.  E8 5073E9FF      call NetUSB_e.0043BBAC               ;  ->StdCtrls.TCustomEdit.SetReadOnly(TCustomEdit;Boolean);
005A485C  |.  B2 01            mov dl,1
005A485E >|.  8B83 FC020000    mov eax,dword ptr ds:[ebx+2FC]       ;  \*LEt\_HardCode:N.A.
005A4864 >|.  E8 4373E9FF      call NetUSB_e.0043BBAC               ;  ->StdCtrls.TCustomEdit.SetReadOnly(TCustomEdit;Boolean);
005A4869  |.  33C0             xor eax,eax
005A486B  |.  8945 F8          mov [local.2],eax
005A486E  |.  8945 FC          mov [local.1],eax
005A4871  |.  EB 19            jmp short NetUSB_e.005A488C
005A4873 >|>  E8 70E3E5FF      /call NetUSB_e.00402BE8              ;  ->System.Randomize;
005A4878  |.  B8 FFC99A3B      |mov eax,3B9AC9FF                    ;  产生一个随机数
005A487D  |.  E8 CAE9E5FF      |call NetUSB_e.0040324C
005A4882  |.  8945 E0          |mov [local.8],eax
005A4885  |.  DB45 E0          |fild [local.8]
005A4888  |.  DD5D F8          |fstp qword ptr ss:[ebp-8]
005A488B  |.  9B               |wait
005A488C  |>  DD45 F8           fld qword ptr ss:[ebp-8]
005A488F  |.  D81D A44A5A00    |fcomp dword ptr ds:[5A4AA4]
005A4895  |.  DFE0             |fstsw ax
005A4897  |.  9E               |sahf
005A4898  |.^ 72 D9            \jb short <NetUSB_e.->System.Randomi>
005A489A  |.  8D55 D8          lea edx,[local.10]
005A489D  |.  A1 4C2A5C00      mov eax,dword ptr ds:[5C2A4C]
005A48A2  |.  8B00             mov eax,dword ptr ds:[eax]
005A48A4 >|.  E8 8FAFECFF      call NetUSB_e.0046F838               ;  ->DdeMan.TDdeMgr.GetExeName(TDdeMgr):AnsiString;<+>
005A48A9  |.  8B45 D8          mov eax,[local.10]                   ;  当前应用程序路径
005A48AC  |.  8D55 DC          lea edx,[local.9]
005A48AF >|.  E8 8026FFFF      call NetUSB_e.00596F34               ;  ->:THttpCli.\_PROC\_00596F34()
005A48B4  |.  FF75 DC          push [local.9]                       ;  产品名称:'网络\*\*'
005A48B7  |.  68 B04A5A00      push NetUSB_e.005A4AB0               ;  '|'
005A48BC  |.  DD45 F8          fld qword ptr ss:[ebp-8]
005A48BF  |.  83C4 F4          add esp,-0C
005A48C2  |.  DB3C24           fstp tbyte ptr ss:[esp]              ; |
005A48C5  |.  9B               wait                                 ; |
005A48C6  |.  8D45 D4          lea eax,[local.11]                   ; |
005A48C9  |.  E8 926CE6FF      call NetUSB_e.0040B560               ; \NetUSB_e.0040B560
005A48CE  |.  FF75 D4          push [local.11]                      ;  随机数?
005A48D1  |.  68 B04A5A00      push NetUSB_e.005A4AB0               ;  '|'
005A48D6  |.  8D55 CC          lea edx,[local.13]
005A48D9 >|.  8B83 F8020000    mov eax,dword ptr ds:[ebx+2F8]       ;  \*LEt\_RegUser:N.A.
005A48DF >|.  E8 88A3EAFF      call NetUSB_e.0044EC6C               ;  ->Controls.TControl.GetText(TControl):TCaption;
005A48E4  |.  8B45 CC          mov eax,[local.13]                   ;  用户名
005A48E7  |.  8D55 D0          lea edx,[local.12]
005A48EA >|.  E8 F54BE6FF      call NetUSB_e.004094E4               ;  ->SysUtils.Trim(AnsiString):AnsiString;overload;
005A48EF  |.  FF75 D0          push [local.12]
005A48F2  |.  68 B04A5A00      push NetUSB_e.005A4AB0
005A48F7  |.  8D55 C4          lea edx,[local.15]                   ;  '|'
005A48FA  |.  A1 6C285C00      mov eax,dword ptr ds:[5C286C]
005A48FF  |.  8B00             mov eax,dword ptr ds:[eax]
005A4901  |.  8B80 64030000    mov eax,dword ptr ds:[eax+364]
005A4907  |.  E8 BCB8F4FF      call NetUSB_e.004F01C8               ;  SerialNumber()
005A490C  |.  8B45 C4          mov eax,[local.15]                   ;  得到HDSerialNumber
005A490F  |.  8D55 C8          lea edx,[local.14]
005A4912 >|.  E8 CD4BE6FF      call NetUSB_e.004094E4               ;  ->SysUtils.Trim(AnsiString):AnsiString;overload;
005A4917  |.  FF75 C8          push [local.14]
005A491A  |.  68 B04A5A00      push NetUSB_e.005A4AB0               ;  '|'
005A491F  |.  8D55 BC          lea edx,[local.17]
005A4922 >|.  8B83 FC020000    mov eax,dword ptr ds:[ebx+2FC]       ;  \*LEt\_HardCode:N.A.
005A4928 >|.  E8 3FA3EAFF      call NetUSB_e.0044EC6C               ;  ->Controls.TControl.GetText(TControl):TCaption;
005A492D  |.  8B45 BC          mov eax,[local.17]                   ;  机器码
005A4930  |.  8D55 C0          lea edx,[local.16]
005A4933 >|.  E8 AC4BE6FF      call NetUSB_e.004094E4               ;  ->SysUtils.Trim(AnsiString):AnsiString;overload;
005A4938  |.  FF75 C0          push [local.16]                      ;  机器码
005A493B  |.  68 B04A5A00      push NetUSB_e.005A4AB0               ;  '|'
005A4940  |.  8D45 F4          lea eax,[local.3]
005A4943  |.  BA 0A000000      mov edx,0A
005A4948 >|.  E8 0304E6FF      call NetUSB_e.00404D50               ;  ->System.@LStrCatN;
005A494D  |.  8D55 B8          lea edx,[local.18]                   ;  将上面的字符串连接起来
005A4950  |.  8B45 F4          mov eax,[local.3]                    ;  网络\*\*|随机数|用户名|HDSerialNumber|机器码|
005A4953  |.  E8 403DFFFF      call NetUSB_e.00598698               ;  加密注册信息
005A4958  |.  8B55 B8          mov edx,[local.18]
005A495B  |.  8D45 F4          lea eax,[local.3]
005A495E >|.  E8 FD00E6FF      call NetUSB_e.00404A60               ;  ->System.@LStrLAsg(void;void;void;void);
005A4963  |.  8D55 B0          lea edx,[local.20]
005A4966  |.  8B45 F4          mov eax,[local.3]
005A4969 >|.  E8 764BE6FF      call NetUSB_e.004094E4               ;  ->SysUtils.Trim(AnsiString):AnsiString;overload;
005A496E  |.  8B45 B0          mov eax,[local.20]
005A4971  |.  8D55 B4          lea edx,[local.19]
005A4974  |.  E8 373AFFFF      call NetUSB_e.005983B0
005A4979  |.  8B55 B4          mov edx,[local.19]
005A497C  |.  8D45 F4          lea eax,[local.3]                    ;  放到粘贴板中……
005A497F >|.  E8 DC00E6FF      call NetUSB_e.00404A60               ;  ->System.@LStrLAsg(void;void;void;void);
005A4984 >|.  E8 2F3CEAFF      call NetUSB_e.004485B8               ;  ->Clipbrd.Clipboard:TClipboard;<+>
005A4989  |.  8B10             mov edx,dword ptr ds:[eax]
005A498B  |.  FF52 18          call dword ptr ds:[edx+18]
005A498E  |.  8D55 AC          lea edx,[local.21]
005A4991  |.  8B45 F4          mov eax,[local.3]
005A4994 >|.  E8 4B4BE6FF      call NetUSB_e.004094E4               ;  ->SysUtils.Trim(AnsiString):AnsiString;overload;
005A4999  |.  8B45 AC          mov eax,[local.21]
005A499C  |.  50               push eax
005A499D >|.  E8 163CEAFF      call NetUSB_e.004485B8               ;  ->Clipbrd.Clipboard:TClipboard;<+>
005A49A2  |.  5A               pop edx
005A49A3 >|.  E8 F837EAFF      call NetUSB_e.004481A0               ;  ->Clipbrd.TClipboard.SetAsText(TClipboard;AnsiString);
005A49A8 >|.  E8 0B3CEAFF      call NetUSB_e.004485B8               ;  ->Clipbrd.Clipboard:TClipboard;<+>
005A49AD  |.  8B10             mov edx,dword ptr ds:[eax]
005A49AF  |.  FF52 14          call dword ptr ds:[edx+14]
005A49B2  |.  6A 30            push 30
005A49B4  |.  8D55 A8          lea edx,[local.22]                   ;  提示获取注册信息成功。
005A49B7  |.  B8 6C4A5A00      mov eax,NetUSB_e.005A4A6C            ;  ASCII "oJCFk\"
```
