
# 破解用`Vboxs420.dll`加密的WebDrive 2.2 (1千字)

WebDrive 2.2(主程序`webdrive.exe`长度106,828字节）

http://www.riverfrontsoftware.com

这个东西用VBox 4.20加密的，只有一个DLL即`vboxs420.dll`。我的SoftICE 4.05用SoftICE Backdoor Keeper打过补丁，没看见它检测到SoftICE。用`bpx DialogBoxParamaA`设断点，等它显示Nag时就会断下来，按一下f12，然后点击Try按钮，就会看见如下的代码：

```
:10002FA0  CALL      [EAX+2C]    //若未过期则在EAX中返回B164
:10002FA3  CMP      EAX,EBX
:10002FA5  JNZ      10002FBA    //这个地方如果不跳走就说明程序出错了
:10002FA7  MOV      EAX,[1006EDB8]
:10002FAC  PUSH      EDI
:10002FAD  PUSH      FF
:10002FAF  PUSH      ESI
:10002FB0  MOV      ECX,[EAX+04]
:10002FB3  CALL      100282F0
:10002FB8  JMP      10002FF1
:10002FBA  MOV      EAX,[1006EDB8]
:10002FBF  PUSH      00
:10002FC1  PUSH      1000322F
:10002FC6  PUSH      DWORD PTR [EAX+3C]
:10002FC9  MOV      EAX,[100696B0]
:10002FCE  PUSH      000000CB
:10002FD3  PUSH      DWORD PTR [EAX+08]
:10002FD6  CALL      [USER32!DialogBoxParamA]  //显示Nag
:10002FDC  MOV      ESI,EAX
          //若选Try则EAX为0，选Quit则为1
:10002FDE  CALL      100031B0
:10002FE3  PUSH      ESI
:10002FE4  CALL      10002FF5
:10002FE9  AND      DWORD PTR [1006EDB8],00
:10002FF0  POP      ECX
:10002FF1  POP      EDI
:10002FF2  POP      ESI
:10002FF3  POP      EBX
:10002FF4  RET
```

按几下F12就到如下的地方：

```
:00406B8A  PUSH      01
:00406B8C  CALL      004063D0      //在这里头显示Nag
:00406B91  ADD      ESP,04
:00406B94  TEST      EAX,EAX
:00406B96  JZ        00406DE8      //过期则跳走
:00406B9C  MOV      ECX,0040F320
:00406BA1  CALL      004016E0
:00406BA6  MOV      ECX,0040F348
```

很显然，只要不要让它跳走即可，由于`Vboxs420.dll`和`webdrive.exe`均变换过，而且结合得比较紧密（就是所吹的injection）写一个`process patcher`即可，它没有CRC校验。跳过`JZ`前面的那个`CALL`则连Nag也没有了。
