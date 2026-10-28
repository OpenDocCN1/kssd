
```
function GetProcAddressEx(Proc: Pointer): Pointer;

var
  lpCallRet: Pointer;
  iCodeLen: Integer;

begin

  Result := nil;
  lpCallRet := nil;

  iCodeLen := SizeOfCode(Proc);
  //  判断第一行代码是否为push 0xXXXXXXXX
  if (iCodeLen = 5) and (Byte(Proc^) = $68) then
  begin
    //  获取0xXXXXXXXX
    Proc := Pointer(PDWORD(longword(Proc) + 1)^);
    while True do
    begin
      iCodeLen := SizeOfCode(Proc);
      //  判断是否为call dword ptf[0xXXXXXXXX]
      if (iCodeLen = 6) and (Byte(Proc^) = $FF) and (Byte(Pointer(longword(Proc) + 1)^) = $15) then
      begin
        //  获取0xXXXXXXXX
        lpCallRet := Proc;
        Break;
      end;
      //  函数结尾
      if (Byte(Proc^) = $C3) then Break;
      Proc := pointer(longword(Proc) + iCodeLen);
    end;
  end;
  if lpCallRet <> nil then
  begin
    Result := Pointer(PDWORD(PDWORD(longword(lpCallRet) + 2)^)^);
  end;
end;
```

不好意思...由于暂时没有编程工具.文章里面出现的代码是以前写的

这篇文章完全是一篇回忆录...以前写的代码和资料全部丢失了..

这里使用的是Ms-Rem的Opcode Length Disassembler Engine

这里描述的是如何bypass 堆跳转挂钩模式...`jmp`和`call`调转模式

都一样的原理和代码可以实现.大家可以自己模拟作一下.关键是思路

大家有什么好的方法希望提供...

OLD引擎您可以从Ms-Rem的`process_hunter` Src中获取到

92bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6S2M7$3#2Q4x3X3g2J5N6b7`.`.

再次感谢许许多多的知名的不知名的大侠们的文章和代码.感谢

thank:EliCZ,Madshi,Ms-Rem,Aphex...29A Group

转载请保留版权.谢谢

Anskya@Gmail.com

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
