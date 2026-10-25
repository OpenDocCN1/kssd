
# 某款试用版听力学习软件无限次使用的实现

> 原文：[`www.pediy.com/kssd/pediy10/95870.html`](https://www.pediy.com/kssd/pediy10/95870.html)

by 天易 love 2009.7.31

本人近日偶然遇到一款英语学习软件，试用之后发现所含听力材料异常丰富，很是喜欢。不足之处是仅能试用 20 次，非常无奈。钱倒不是问题，但作为“看雪”的人去邮局汇钱买软件应该会感到很自卑，所以在此对作者表示抱歉。同时谢谢你开发出如此具有创意的软件，顺便给你做一个广告。下面进入正题：

## 一、2006 版软件的大致保护原理

程序每次运行后都将会在注册表中的某处记录下已运行过的次数。下次运行如果发现注册表该处数据为 20 就立即提示试用结束；程序还将当前使用的次数加密后以某个特定的文件名保存在某个特定的系统目录中，下次运行时找到该文件并解密出使用过的次数，如果已经是 20 次了也会提示试用结束;这样就实现了软件的双重保护。

## 二、软件保护中采用的技巧

每次保存到系统目录中的文件内容都只有 3 个字节，只是这个系统目录在不停的变化，这 3 个字节的数据也在变。

假如这一次是保存在 `C:\WINDOWS\Help` 中，那么下一次就是在 `C:\WINDOWS\system32` 里，再下一次又是在 `C:\WINDOWS\Help` 中，如此循环。软件刚开始运行时，在这两个指定的目录中搜索是否含有这些包含使用次数信息的文件，如果存在就对文件中包含的 3 个字节数据进行解密出已经使用过的次数。

## 三、突破保护的方法

第一步：考虑到加密文件中的数据是固定的，其值只与文件名有关。这是一个很大的弱点，我可以在试用的前 20 次内的某一次执行程序前，找出该文件保存起来。

第二步：修改注册表中记录次数的地方，小于 20 即可。

试用了 20 次之后，要想重新开始计数只要恢复系统文件夹中保存加密信息的文件，然后修改一下注册表就行了。是不是很简单啊？

![](img/590e94dcd7060782d3103e584dabc870.jpg)

![](img/cd03bb9ed368327c3c0869ca7fba4206.jpg)

![](img/aa29bfd92a1dd67a70a18401a0a7b90e.jpg)

## 四、无限次使用的 delphi 实现

考虑到这是款国产软件，代码的关键部分省略一些。

```delphi
program Project1;

uses  SysUtils,Windows,shellapi,Registry,Classes;   

procedure  operation();

Var 

  hd,BinaryFile:THandle;

  reg:Tregistry;

  delfile:TStringList;

  i:integer;

  BinaryBuffer: pchar;

  BytesWritten: dword;

begin

     shellexecute(0,'open','xxxx.exe',nil,nil,1);

     BinaryFile:=0;

     reg:=tregistry.Create ;

     reg.RootKey :=HKEY\_LOCAL\_MACHINE;

     IF reg.OpenKey('SOFTWARE\\Classes\\CLSID\\{xxxxx-xxx-xxxx-xxxx-xxxxxx}',True) then 
        reg.WriteString('','1');

     reg.Free;

    delfile:=TStringList.Create;

    delfile.Add('C:\\WINDOWS\\system32\\xxxxx.DLL');

    delfile.Add('C:\\WINDOWS\\system32\\xxxxx.DLL');

    delfile.Add('C:\\WINDOWS\\system32\\xxxx.DLL');

    delfile.Add('C:\\WINDOWS\\system32\\xxx.DLL');

    delfile.Add('C:\\WINDOWS\\system32\\xxxx.DLL');

    delfile.Add('C:\\WINDOWS\\system32\\xxxx.DLL');

    delfile.Add('C:\\WINDOWS\\system32\\xxxxx.DLL');

    delfile.Add('C:\\WINDOWS\\system32\\xxxxx.DLL');

    delfile.Add('C:\\WINDOWS\\system32\\xxxxxxxxxx.DLL');

    delfile.Add('C:\\WINDOWS\\system32\\xxxxx.DLL');

    delfile.Add('C:\\WINDOWS\\Help\\xxxxxxxxxxx.HLP');

    delfile.Add('C:\\WINDOWS\\Help\\xxxxxxxxxx.HLP');

    delfile.Add('C:\\WINDOWS\\Help\\xxxxx.HLP');

    delfile.Add('C:\\WINDOWS\\Help\\xxxxxxxxx.HLP');

    delfile.Add( 'C:\\WINDOWS\\Help\\xxxxxxxxx.HLP');

    delfile.Add('C:\\WINDOWS\\Help\\xxxxxxxxxxx.HLP');

    delfile.Add('C:\\WINDOWS\\Help\\xxxxxxxxxxxx.HLP');

    delfile.Add('C:\\WINDOWS\\Help\\xxxxxxxxxx.HLP');

    delfile.Add('C:\\WINDOWS\\Help\\xxxxxxxxxxx.CHM');

     For i:=0 To  delfile.Count-1 do 
        begin 
          if fileexists(delfile\[i\]) then 
            begin 
                DeleteFile(pchar(delfile\[i\])); 
            end; 
        end; 

     delfile.free; 

     try 
       BinaryFile := CreateFile('C:\\WINDOWS\\system32\\xxxxx.DLL', GENERIC\_WRITE, FILE\_SHARE\_WRITE, nil, CREATE\_ALWAYS, FILE\_ATTRIBUTE\_NORMAL, 0); 

       if BinaryFile<>0 then 
         begin 
             GetMem(BinaryBuffer,3); 

             BinaryBuffer\[0\]:=#86; 

             BinaryBuffer\[1\]:=#233; 

             BinaryBuffer\[2\]:=#111; 

             WriteFile(BinaryFile,BinaryBuffer^, 3,BytesWritten, nil); 

         end; 

     finally 
        CloseHandle(BinaryFile); 

      end; 

     sleep(1600); 

     repeat 
          hd:=FindWindow(nil,'xxxxxx 2006 - 启动窗口'); 

      until hd <> 0; 

      hd:=FindWindowEx(hd,0,nil,'免费试用'); 

      if hd<>0  then 
       begin 
          sendmessage(hd,$0201,0,1);  
          sendmessage(hd,$0202,0,1);  
       end; 

end; 

begin 

  operation(); 

end.
```

## 五、2008 版软件的大致保护原理

作者在保护方面有所改进，在注册表中保存了两处使用次数的地方，这两处必须相等，此外还有几十处的垃圾信息；而 2006 版只有一处且只要小于 20 次即可。仍然使用了在系统目录中保存文件的办法，只是又多了几个垃圾文件，在试用完后不删除掉，就不能再使用了，同样注册表中的几十处的垃圾信息也是这个作用。

2009 版软件功能没有变化，没有研究的必要。

综上所述，该软件的试用版要想实现无限次的使用，只是对垃圾文件包括注册表中的垃圾信息的清除以及对系统目录写入制定内容的文件、注册表中指定的位置写入使用的次数即可。通过实践，均已实现。

by 天易 love 2009.7.31
