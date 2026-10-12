
的值从右到左取出乘以十并相加，此时计算出来的值为十六进制，最后把管个十六进制转会十进制就是我们想要的注册码了

### 【注册机制作】

现在我们可以来制作注册机了，呵呵，我只会用 Delphi 哟！好了开始开工了。

先来写三个函数吧！

#### 第一个函数模仿第一次计算

```delphi
function strtointex(s: String): integer;

var
  i, n: integer;

begin
  n := 0;
  if length(s) - 1 > 0 then
    for i := 1 to length(s) do
      n := (n + ord(s[i])) * 10;

  result := n;
end;
```

#### 第二个函数模仿第二次计算

```delphi
function strtointex0(s: String): string;

var
  i: integer;
  n: int64;
  x: string;

begin
  n := 0;
  result := '';
  if length(s) > 0 then
  begin
    for i := length(s) downto 1 do
      n := (n + ord(s[i])) * 10;
    x := inttohex(n, 16);
    result := copy(x, length(x) - 7, 8);
  end;
end;
```

#### 第三个函数把十六进制转为十进制

不知道为什么 Delphi 中没有内置这个函数，还是不知道！管他了，我们写一个！

```delphi
function HexTostr(Hex: String): string;

var
  I: integer;
  Res: int64;
  ch: Char;

begin
  Res := 0;
  if Length(Hex) > 0 then
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

好了注册机主要部分我们做好了～在程序上放两个 `Edit…`(edit1 和 edit2) 在 edit1 的 `OnKeyUp` 事件中加上：

```delphi
Edit2.text := HexTostr(strtointex0(inttostr(strtointex(edit1.text))));
```

好了注册机搞定，呵呵。。。。。。。。函数写的有什么不好的地方希望高手提出，我的 QQ:9674103 希望大家一起交流。我是个菜鸟～～

`4dd466d0e1e0cc123afbe504da2462ca`
