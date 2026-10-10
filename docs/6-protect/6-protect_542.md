

# 基于流的加壳与脱壳技术实现

> 原文：[`www.pediy.com/kssd/pediy12/137546.html`](https://www.pediy.com/kssd/pediy12/137546.html)

0．  引言  
    软件加密技术是软件保护的重要手段。这些加密措施给非法拷贝或非法使用造成一定的障碍，在某种程度上增加了破解难度。因现在互联网的普及，各种破解软件可以从网上轻易下载，为增加破解难度，编写自己的加密程序是很有必要的。现在加密程序的重要手段之一就是对程序加壳。如果加壳程序使用压缩算法还可以减少程序的体积，防止破解的人非法修改程序。本文介绍一种利用流技术的加壳脱壳方法来实现对软件的加密保护。  
1.  关于流  
流（TStream），从某种意义来说，它就是一段数据或是一块内存。在进行流操作时, 我们不必关心流中的数据到底是什么，只需要知道流的大小和当前的指针位置。所以流只有两个属性: Size（流中数据大小）和 Position（流中存取指针位置）。TStream 是一个抽象的基类, 不能直接生成对象。在具体的应用中, 主要使用它的子孙类:TFileStream 文件流;TStringStream:字符串流;TMemoryStream 内存流;TResourceStream 资源文件流等。本文就是利用内存流对应用程序进行加壳脱壳的。TMemoryStream 类的有关方法主要有：  
Read 方法实现将数据从内存流中读出；Write 方法实现将数据写入内存流中；Seek 方法从内存流中读取指针的移动；CopyFrom 方法从其它内存流中拷贝数据； LoadFromStream 方法主要用于读取文件；SaveToStream 方法主要将数据写入文件。  
2.  加壳脱壳原理  
加壳的原理就是把一段外壳程序和密码信息附加到应用程序中，并把程序的执行入口指向附加的代码中。这样，程序装入内存之后，附加代码首先执行，检查密码是否正确，如果正确则转入原来的程序中执行。否则，则退出程序。  
外壳加密实际上是给可执行文件加上一个外壳。用户执行的实际上是这个外壳程序。附加代码、被加壳程序及密码信息的存放结构如图 1 所示。  
   
其中，附加代码必须放在起始位置，程序运行时就执行它。被加壳程序和密码信息位置可以互换，我们只要知道它们的大小和准确位置就可以了。密码验证正确后，就把被加壳程序进行脱壳分离出来并执行它，等待它执行完毕，主进程才结束。  
1.1 附加代码的处理流程  
在附加代码执行后，首先读取自身密码信息（包括附加的外壳程序长度、被加壳程序长度、密码等），同用户输入的密码进行比对，确认无误后，将被加壳程序脱壳分离出来，同时把自身隐藏起来，创建被加壳程序的进程，等待被加壳程序进程结束后，退出本应用程序。如果密码比对不成功，则直接退出。  
1.2 加壳处理流程  
    首先读取附加代码、被加壳程序文件名及路径，同时读取自己定义的信息结构和用户输入的密码（其实就是一个纪录）。创建一个内存流，将附加代码读入内存流中，定位在内存流尾端，再创建另外一内存流，将被加壳程序写入内存流。将被加壳程序的内存流拷贝在附加代码后面。最后将自定义的信息写入内存流，保存在临时文件中。删除被加壳的文件，将临时文件命名为被加壳程序的文件名。  
1.3 脱壳处理流程  
脱壳的处理流程同加壳处理流程恰好相反。它首先创建一个内存流，将已经加壳的文件读入内存流，内存流定位到密码信息部分，验证密码信息，如果比对不成功，则退出程序。如比对成功后，内存流定位到被加壳程序部分，将它拷贝给另外一个内存流并保存为一个临时文件，删除已加壳的文件，将临时文件保存为被加壳程序的文件名，完成脱壳处理。  
3.  程序的实现及解读  
要实现程序的加壳脱壳处理，需要编写两个程序，即外壳程序（即附加代码）和加壳程序。外壳程序主要供加壳程序使用。其主要功能是提供用户输入密码，并比对已经加壳程序中密码信息同用户输入的密码是否相同。如果不同，则退出。比对成功，则进行脱壳处理，并运行脱壳后的程序。加壳程序用来将附加代码和密码信息附加到被加壳程序上。被加壳后的程序需要输入正确密码才能运行。  
 在这里，只列出加壳脱壳程序的关键代码。  
3.1 相关定义：密码信息和读取文件长度函数  
 为方便内存流的准确定位，必须知道文件的长度大小。其函数为：  
function GetFilelen(filename:string):integer;  
var  
   sr:TSearchRec;  
begin  
   if (findfirst(filename,faAnyfile and not faDirectory,sr)<>0) then result:=0  
   else result:=sr.size;  
   findclose(sr);  
end;  
密码信息为一纪录结构，其原型为  
Const  
Flag='我的加密文件';  
PassSize=15;//密码信息最高字符数  
type  
 Tinfo=Record  
  Flag:string\[Length(Flag)\];// 加壳标志，由用户自定义  
  Name: ShortString;// 文件名  
  PassWord:string\[PassSize\];// 密码  
  ShellLen,SourceLen:integer;//附加代码长度和被加壳程序长度  
 end;  
其实在密码信息部分，可任意添加自己的敏感信息。  
3.2 加壳程序  
var  
  CodeName,myname: string;// 附加代码的文件名和临时文件名  
  myInfo: TInfo;  
  MsStream,SourceMem:TMemoryStream;//内存流  
begin  
    CodeName:= ExtractFilePath(paramstr(0))+'shellprj.exe';  //附加代码的文件名  
    with myInfo do  
    begin  
        Flag := Flag;//自定义的标志  
        Name := ExtractFileName(SourceName);//文件名  
        Password := Edit2.Text;//密码  
        ShellLen := GetFilelen(CodeName);//附加代码的长度  
        SourceLen:=GetFilelen(SourceName);//被加壳程序长度  
    end;  
  
    MsStream:=TMemoryStream.Create; //创建内存流  
    MsStream.LoadFromFile(CodeName);//将附加代码读入内存流  
    MsStream.Seek(0,2);//定位内存流于尾端  
  
    SourceMem:=TMemoryStream.Create;  
    SourceMem.LoadFromFile(SourceName);//读取被加壳程序到内存流  
    SourceMem.Seek(0,0); //定位内存流于首端  
    MsStream.CopyFrom(SourceMem,0);//在附加代码尾端添加被加壳程序  
  
    MsStream.Seek(0,2);  
    MsStream.Writebuffer(myInfo, SizeOf(myInfo)); //在附加代码、被加壳程序之后添加密码信息  
    myname:=ExtractFilePath(SourceName)+'my.exe'; //临时文件名  
    MsStream.SaveToFile(myname); //释放内存流  
    SourceMem.Free;  
    MsStream.Free;  
      
    DeleteFile(SourceName);//删除被加壳的文件  
    RenameFile(myName, SourceName); //把临时文件重命名为被加壳的文件  
    showmessage('加壳已完成');  
end;  
在加壳程序中，应在加壳前读取被加壳程序是否有加壳标志 Flag，如没有加壳标志 Flag，则进行加壳。否则，弹出对话框，提醒操作者。为求程序的简化，这里省去。  
3.3 脱壳程序  
var  
  FsName, FtName: string; // 已加壳的文件名和临时文件名  
  myInfo: TInfo;  
  MsStream,TmpStream:TMemoryStream; //内存流  
begin  
   FsName := SourceName;//已加壳的文件名  
   MsStream:=TMemoryStream.Create;// 创建内存流  
   MsStream.LoadFromFile(FsName);//读取已加壳文件到内存流  
   MsStream.Seek(-Sizeof(myInfo),2);//定位到密码信息部分  
   MsStream.Read(myInfo,Sizeof(myInfo));//读取密码信息  
   if myInfo.Password =Edit2.Text then  
   begin  
     FtName := ExtractFilePath(FsName) + 'Fei' + myInfo.Name; //临时文件是在已加壳文件名前加"Fei"  
     TmpStream:=TMemoryStream.Create;//创建内存流  
     MsStream.Seek(myInfo.ShellLen,0);//定位到附加代码部分尾端，被加壳程序首端  
     TmpStream.Seek(0,0);  
     TmpStream.CopyFrom(MsStream,myInfo.SourceLen);//拷贝被加壳程序  
     TmpStream.SaveToFile(FtName);//将被加壳程序命名为临时文件名  
     TmpStream.Free; //释放内存流  
   end;  
   MsStream.Free;  
   DeleteFile(FsName);  //删除已加壳的文件  
   RenameFile(FtName, FsName); //把临时文件改为已加壳  
   showmessage('脱壳已完成');  
end;  
在脱壳程序中，同样要在脱壳前读取程序是否有加壳标志 Flag，如有加壳标志 Flag，则进行脱壳。否则，弹出对话框，提醒操作者。在这里省去。  
4.  结束语  
     流技术在实际应用中有着非常广泛的用途，如结构化文件读取、文件的分割与合并、压缩与解压缩等。本文就是流技术的一个典型应用。相对与其它加壳脱壳技术手段，利用流进行加壳脱壳，程序更为简单和高效。当然，本文在进行加壳脱壳时，对加密密码没有进行复杂的运算，以增加破解的难度，但它不是本文重点，故没有进行介绍。扩展本程序也可实现读加密狗、读 CPU 或序列号等。本程序在 WindowsXP/Delphi7 环境下运行通过。