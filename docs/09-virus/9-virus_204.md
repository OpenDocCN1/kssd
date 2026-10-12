
# Delphi 编写的 `LPK.DLL` 专杀，可清理 `RAR`

> 原文：[`www.pediy.com/kssd/pediy12/137601.html`](https://www.pediy.com/kssd/pediy12/137601.html)

病毒背景：

它是一种杀毒软件很难清除的一种木马，会随着系统启动而启动，会自动生成文件到系统目录或是程序目录，有可能还会在 `RAR` 文件目录中生成，一般杀毒软件查杀到 `RAR` 里面有此文件会提示删除文件而导致客户的文件丢失。如果发现电脑中几乎所有的目录都存在 `lpk.dll`，甚至压缩包中也存在，则极有可能中了利用操作系统自动加载 `lpk` 的蠕虫(而且是木马)。

大部分杀毒软件没法完美的处理 `RAR` 中病毒，（据说瑞星可以，但是速度很慢，不知道是不是采用的解压方法）

为了应急对付 `LPK.DLL` 的大量感染或者说是挽救大量的 `RAR`，熬夜写了这样一个东西（如果清理不干净，稍不留神会再次激活）

软件仅能删除 `LPK.DLL`（不会误删），可以删除带有系统、只读属性的，也可以删除 `RAR` 里（不管是在 `RAR` 根目录下还是里面的文件夹里）

采用体积匹配，应该能对付一些变种。

附源代码以及 `lpk.dll` 样本，本来想用 `unrar.dll` 来处理 `RAR`，不过来不及了。

有什么建议留言就好。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `LPK` 杀.rar  
|

支持 `usp10.dll` 清理吗？ 

你自己加上去就可以啦，有源代码，

```
if Match(FilePath + FileRec.Name, MaskList) then 
    begin 
        ce:=FilePath + FileRec.Name; 
        Form1.lbl3.Caption := ce; 
        Application.ProcessMessages; 
            if LowerCase(RightStr(ce,9)) = 'usp10.dll' then 
            begin 
                Form1.mmo1.Lines.Add('发现 usp10.dll:' + ce); 
                SetFileAttributes(PChar(ce),0); 
                if (filerec.Size>30000) and (filerec.Size<48000) then//这儿你看一下文件大小，自己改一下，为了对付变种，看一下它的体积区间，单位是字节
                if WinExecAndWait32('cmd /c del /f "' + ce +'"', SW_HIDE)=0 
                then 
                begin 
                Form1.mmo1.Lines.Add('删除：' + ce); 
                Inc(sum); 
                end; 
            end; 
            if LowerCase(RightStr(ce,3)) = 'rar' then 
            begin 
                Form1.mmo1.Lines.Add('发现 RAR 文件:' + ce); 
                Form1.mmo1.Lines.Add('分析' + ce); 
                WinExecAndWait32('cmd /c rar.exe vb "' + ce + 
                    '" >TEMP.TXT', SW_HIDE); 
                system.Assign(p, 'TEMP.TXT'); 
                system.Reset(p); 
                repeat 
                    Readln(p, s); 
                    l := Length(s); 
                    if (l > 8) and (LowerCase(RightStr(s, 9)) = 'usp10.dll') then 
                    begin 
                        Form1.mmo1.Lines.Add('发现 USP10.DLL 文件:' + s); 
                        if WinExecAndWait32('rar.exe d -sm30000 -sl48000 "' + 
                            ce + '" "' + s + '"', SW_HIDE) = 0 then//注意体积 -sm 后跟的是最小体积，-sl 后是最大体积 
                        begin 
                        Form1.mmo1.Lines.Add('已清理:' + s); 
                        Inc(sum); 
                        end; 
```

当然，重点就是 `RAR` 的清理。
