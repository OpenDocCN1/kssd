
# MSN 8.1 版本多开工具程序实现代码和原理!

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-586.htm`](https://www.pediy.com/kssd/pediy09/pediy09-586.htm)

> 刚写了一篇 QQ 游戏 `QQGame2006Patch3` 版本多开.分析和补丁！

> `http://bbs.pediy.com/showthread.php?s=&threadid=41546`

> 累呀!这里我就不写 `MSN` 的分析步骤了，主要发一下程序的实现的代码!

> 实现原理 开始调试进入---->修改核心代码---->然后恢复线程

> 本代码为 `DELPHI` 代码，适用 `MSN` 版本为 `8.1` (`Build 8.1.0178.00`)

> 下面是主要的实现代码!写好的程序请看附件!

```
procedure TFrmMain.spSkinButton1Click(Sender: TObject);

const
  OldData : array[0..1] of byte = ($3D,$B7);
  NewData : array[0..1] of byte = ($3D,$B6);

var
  StartInfo: TStartupInfo;
  ProcInfo: TProcessInformation;
  BytesRead : DWORD;
  TmpData : array[0..1] of Byte;
  FilePath:String;

begin
  FilePath := SpSkinEdit1.Text;
  ZeroMemory(@StartInfo, SizeOf(TStartupInfo));
  StartInfo.cb := SizeOf(TStartupInfo);
  if not CreateProcess(nil,Pchar(FilePath), nil, nil, False, Create_Suspended, nil, nil, StartInfo, ProcInfo) then Exit;
  ReadProcessMemory(ProcInfo.hProcess,Pointer($543CEE),@TmpData,2,BytesRead);
  if (TmpData[0] = OldData[0]) and (TmpData[1] = OldData[1]) then
  begin
    WriteProcessMemory(ProcInfo.hProcess, Pointer($543CEE), @NewData, 2, BytesRead);
  end;
  ResumeThread(ProcInfo.hThread);
  CloseHandle(ProcInfo.hProcess);
  CloseHandle(ProcInfo.hThread);
end;
```
