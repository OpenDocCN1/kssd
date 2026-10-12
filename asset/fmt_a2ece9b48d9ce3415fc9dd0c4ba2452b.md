
# 工资表之星 9.0 注册算法分析(简单)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-206.htm`](https://www.pediy.com/kssd/pediy07/pediy7-206.htm)

工资表之星 9.0 注册算法分析

日期：2005 年 4 月 19 日  破解人：`Baby2008`

---

【软件名称】：工资表之星 9.0

【软件大小】：8.32 MB

【下载地址】：`http://www.gzb99.com`

【保护方式】：注册码保护,试用时间限制

【破解声明】：初学 `Crack`，只是感兴趣,失误之处敬请诸位大侠赐教!

【破解工具】：`OllyDbg.V1.10` 聆风听雨汉化第二版、`PeID` 0.93，`DeDe`。

【破解过程】:

`Borland` `Delphi` 6.0 - 7.0 程序未加壳，`OD` 直接载入 `C:\Program Files\salarystar9\salarystar.exe`,插件查找“注册成功，下次启动……”

，双击来到 `0067E500`，向上查找，在 `0067E3F0` 下断，`F9` 运行,输入注册码：`12345678901234567890`，点击确定，`OD` 中断在：

```asm
0067E3F0  >/.  55           push ebp                              ;  <-Tregform@BitBtn1Click
0067E3F1  |.   8BEC         mov ebp,esp
0067E3F3  |.   33C9         xor ecx,ecx
0067E3F5  |.   51           push ecx
0067E3F6  |.   51           push ecx
0067E3F7  |.   51           push ecx
0067E3F8  |.   51           push ecx
0067E3F9  |.   51           push ecx
0067E3FA  |.   53           push ebx
0067E3FB  |.   56           push esi
0067E3FC  |.   8BD8         mov ebx,eax
0067E3FE  |.   33C0         xor eax,eax
0067E400  |.   55           push ebp
0067E401  |.   68 4DE56700  push <salaryst.->System.@HandleFinally>
0067E406  |.   64:FF30      push dword ptr fs:[eax]
0067E409  |.   64:8920      mov dword ptr fs:[eax],esp
0067E40C  |.   8D55 F4      lea edx,[local.3]
0067E40F  >|.  8B83 24030000 mov eax,dword ptr ds:[ebx+324]       ;  *Edit1:N.A.
0067E415  >|.  E8 4A65E0FF  call salaryst.00484964                 ;  ->Controls.TControl.GetText(TControl):TCaption;
0067E41A  |.   837D F4 00   cmp [local.3],0
0067E41E  |.   0F84 F9000000 je salaryst.0067E51D                 ;  注册码不能为空
0067E424  |.   8D45 F8      lea eax,[local.2]
0067E427  |.   8B15 98CF7000 mov edx,dword ptr ds:[70CF98]        ;  salaryst.0070EC68
0067E42D  |.   8B12         mov edx,dword ptr ds:[edx]
0067E42F  >|.  E8 8065D8FF  call salaryst.004049B4                 ;  ->System.@LStrLAsg(void;void;void;void);
0067E434  |.   8D55 F0      lea edx,[local.4]
0067E437  >|.  8B83 24030000 mov eax,dword ptr ds:[ebx+324]       ;  *Edit1:N.A.
0067E43D  >|.  E8 2265E0FF  call salaryst.00484964                 ;  ->Controls.TControl.GetText(TControl):TCaption;
0067E442  |.   8B45 F0      mov eax,[local.4]                      ;  注册码
0067E445  |.   8D55 FC      lea edx,[local.1]
0067E448  >|.  E8 C7FEFFFF  call salaryst.0067E314                 ;  ->:Tregform._PROC_0067E314()
0067E44D  |.   8B45 FC      mov eax,[local.1]
0067E450  >|.  E8 5FFFFFFF  call salaryst.0067E3B4                 ;  ->WebAdapt.TAdapterUpdateField.ConvertValueCount
0067E455  |.   8BF0         mov esi,eax                             ;  Eax=$A
0067E457  |.   83FE 05      cmp esi,5
0067E45A  |.   0F8C BD000000 jl salaryst.0067E51D
0067E460  |.   8D45 EC      lea eax,[local.5]
0067E463  |.   50           push eax
0067E464  >|.  E8 7FFFFFFF  call salaryst.0067E3E8                 ;  ->ADODB.TParameters.GetAttrCount
0067E469  |.   8BD0         mov edx,eax                             ;  Eax=$4
0067E46B  |.   8BCE         mov ecx,esi                               ;  Esi=$A
0067E46D  |.   8B45 FC      mov eax,[local.1]                        ;  注册码
0067E470  >|.  E8 C769D8FF  call salaryst.00404E3C                 ;  ->System.@LStrCopy(注册码,4,10),记为 SN[4-13];
0067E475  |.   8B45 EC      mov eax,[local.5]
0067E478  |.   50           push eax
0067E479  |.   A1 C0D27000  mov eax,dword ptr ds:[70D2C0]
0067E47E  |.   8B00         mov eax,dword ptr ds:[eax]
0067E480  |.   8B80 DC000000 mov eax,dword ptr ds:[eax+DC]
0067E486  |.   33C9         xor ecx,ecx
0067E488  |.   8B55 F8      mov edx,[local.2]
0067E48B  >|.  E8 4CB0E6FF  call salaryst.004E94DC                 ;  注册验证关键，跟进
0067E490  |.   84C0         test al,al
0067E492  |.   74 7F        je short salaryst.0067E513            ;  验证爆破
0067E494  |.   A1 C0D27000  mov eax,dword ptr ds:[70D2C0]
0067E499  |.   8B00         mov eax,dword ptr ds:[eax]
0067E49B  |.   8B40 60      mov eax,dword ptr ds:[eax+60]
0067E49E  >|.  E8 2D8FE4FF  call salaryst.004C73D0                 ;  ->DB.TDataSet.Close(TDataSet);
0067E4A3  |.   A1 C0D27000  mov eax,dword ptr ds:[70D2C0]
0067E4A8  |.   8B00         mov eax,dword ptr ds:[eax]
0067E4AA  |.   8B40 60      mov eax,dword ptr ds:[eax+60]
0067E4AD  |.   8B80 48020000 mov eax,dword ptr ds:[eax+248]
0067E4B3  |.   8B10         mov edx,dword ptr ds:[eax]
0067E4B5  |.   FF52 44      call dword ptr ds:[edx+44]
0067E4B8  |.   A1 C0D27000  mov eax,dword ptr ds:[70D2C0]
0067E4BD  |.   8B00         mov eax,dword ptr ds:[eax]
0067E4BF  |.   8B40 60      mov eax,dword ptr ds:[eax+60]
0067E4C2  |.   8B80 48020000 mov eax,dword ptr ds:[eax+248]
0067E4C8  |.   BA 64E56700  mov edx,salaryst.0067E564             ;  ASCII "UPDATE USERINFO SET PROGRAMKEY=:PEKY"
0067E4CD  |.   8B08         mov ecx,dword ptr ds:[eax]
0067E4CF  |.   FF51 38      call dword ptr ds:[ecx+38]
0067E4D2  |.   A1 C0D27000  mov eax,dword ptr ds:[70D2C0]
0067E4D7  |.   8B00         mov eax,dword ptr ds:[eax]
0067E4D9  |.   8B40 60      mov eax,dword ptr ds:[eax+60]
0067E4DC  |.   8B80 50020000 mov eax,dword ptr ds:[eax+250]
0067E4E2  |.   33D2         xor edx,edx
0067E4E4  >|.  E8 6B68E4FF  call salaryst.004C4D54                 ;  ->ComCtrls.TListItems.Delete(TListItems;Integer);<+>
0067E4E9  |.   8B55 FC      mov edx,[local.1]
0067E4EC  >|.  E8 EB82E4FF  call salaryst.004C67DC                 ;  ->DB.TParam.SetAsString(TParam;AnsiString);
0067E4F1  |.   A1 C0D27000  mov eax,dword ptr ds:[70D2C0]
0067E4F6  |.   8B00         mov eax,dword ptr ds:[eax]
0067E4F8  |.   8B40 60      mov eax,dword ptr ds:[eax+60]
0067E4FB  >|.  E8 3C59E6FF  call salaryst.004E3E3C                 ;  ->DBTables.TTable.DeleteTable(TTable);<+>
0067E500  |.   B8 94E56700  mov eax,salaryst.0067E594             ;  注册成功……
0067E505  >|.  E8 62E5E2FF  call salaryst.004ACA6C                 ;  ->Dialogs.ShowMessage(AnsiString);
0067E50A  |.   8BC3         mov eax,ebx
0067E50C  >|.  E8 074EE2FF  call salaryst.004A3318                 ;  ->Forms.TCustomForm.Close(TCustomForm);
0067E511  |.   EB 0A        jmp short salaryst.0067E51D
0067E513  |>   B8 E8E56700  mov eax,salaryst.0067E5E8             ;  注册码错误
0067E518  >|.  E8 4FE5E2FF  call salaryst.004ACA6C                 ;  ->Dialogs.ShowMessage(AnsiString);
0067E51D  |>   33C0         xor eax,eax
0067E51F  |.   5A           pop edx
0067E520  |.   59           pop ecx
0067E521  |.   59           pop ecx
0067E522  |.   64:8910      mov dword ptr fs:[eax],edx
0067E525  |.   68 54E56700  push salaryst.0067E554
0067E52A  |>   8D45 EC      lea eax,[local.5]
0067E52D  >|.  E8 EA63D8FF  call salaryst.0040491C                 ;  ->System.@LStrClr(void;void);
0067E532  |.   8D45 F0      lea eax,[local.4]
0067E535  |.   BA 02000000  mov edx,2
0067E53A  >|.  E8 0164D8FF  call salaryst.00404940                 ;  ->System.@LStrArrayClr(void;void;Integer);
0067E53F  |.   8D45 F8      lea eax,[local.2]
0067E542  |.   BA 02000000  mov edx,2
```
