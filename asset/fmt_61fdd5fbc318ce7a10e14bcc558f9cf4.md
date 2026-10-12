
`////计算暗证番号(注册码),算法一目了然吧~`

`int k = 0x7346322 + Math.abs((i / 133) \* 171 ^ 0x25cd97d);`

`//用 delphi 算注册码为 copy(120873762 +  Abs(((i div 133) \* 171) xor 39639421),0,8)`

`//以下比较注册码`

`if(registerdata.registerCode.equals(String.valueOf(k).substring(0, 8)))`

```
{
    nativeIndex = registerdata.nativeIndex;
    flag = true;
}
if(registerdata.loginName.equals("friend") && registerdata.registerCode.equals("1234567")) //靠,万能注册码,省得我写什么注册机了~
{
    nativeIndex = registerdata.nativeIndex;
    flag = true;
}
fileinputstream.close();
```

`catch(Exception exception) { }`

`return flag;` //改成 `return true;`就不用注册了~

`}`

如果主类 `JptStart.class` 也被加密就难破解了,但不知道如何实现。看来共享软件最好不要用 `java` 编,除非你真的有很好的加密方法。BTW,这个软件做的不错,对学 `java` 的朋友有很大的帮助。

`4dd466d0e1e0cc123afbe504da2462ca`
