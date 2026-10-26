
``` 
function HexTostr(Hex: String): string;
var
    I:integer; Res: int64;
    ch: Char;
begin
    Res := 0;
    if Length(Hex)>0 then
    for I := 0 to Length(Hex) - 1 do
    begin
      ch := Hex[I + 1];
      if (ch >= '0') and (ch <= '9') then
        Res := Res * 16 + Ord(ch) - Ord('0')
      else if (ch >= 'A') and (ch <= 'F') then
        Res := Res * 16 + Ord(ch) - Ord('A') + 10
      else if (ch >= 'a') and (ch <= 'f') then
        Res := Res * 16 + Ord(ch) - Ord('a') + 10
      else raise Exception.Create('Error: not a Hex String');
    end;
    Result := inttostr(Res);
end;
```

好了注册机主要部分我们做好了～在程序上放两个 Edit…（`edit1` 和 `edit2`），在 `edit1` 的 `OnKeyUp` 事件中加上：

`Edit2.text:=HexTostr(strtointex0(inttostr(strtointex(edit1.text))));`

好了注册机搞定，呵呵。。。。。。。。函数写的有什么不好的地方希望高手提出，我的QQ:9674103希望大家一起交流。我是个菜鸟～～

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
