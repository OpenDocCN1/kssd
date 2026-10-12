
# EncryptPE 2005.9.10 脱壳脚本,是不是 s 都可以

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-671.htm`](https://www.pediy.com/kssd/pediy07/pediy7-671.htm)

```
//用的 OllyMachine  

invoke msg, "哈哈!现在我要开始运行啦，保存好当前工作，有可能要关闭 explorer.exe 哦!"  

invoke BPHWS, 0x711E3FF6, 1  

invoke ESTO  

invoke BPHWC 0x711E3FF6  

invoke WriteMemHexes, 0x711E3FF6, "EB"  

invoke WriteMemHexes, 0x711E5958, "9090"  

invoke BPHWS, 0x711E596D, 1  

invoke ESTO  

invoke BPHWC 0x711E596D  

invoke WriteMemHexes, 0x711E3FF6, "74"  

invoke WriteMemHexes, 0x711E5958, "7513"  

invoke WriteMemHexes, 0x711E5976, "9090"  

invoke WriteMemHexes, 0x711E5981, "EB"  

invoke BPHWS, 0x711E5AED, 1  

invoke ESTO  

invoke BPHWC 0x711E5AED  

invoke WriteMemHexes, 0x711E5AED, "9090909090909090909090"  

invoke WriteMemHexes, 0x711E5B20, "9090"  

invoke WriteMemHexes, 0x711E5B2C, "9090909090909090909090"  

invoke WriteMemHexes, 0x711E5B5E, "9090"  

invoke WriteMemHexes, 0x711E5C38, "13"  

invoke WriteMemHexes, 0x711E5C4A, "13"  

invoke WriteMemHexes, 0x711E5C51, "909090909090"  

invoke WriteMemHexes, 0x711E5C63, "9090909090"  

invoke WriteMemHexes, 0x711E5C8D, "9090"  

invoke WriteMemHexes, 0x711E5CBD, "9090"  

invoke WriteMemHexes, 0x711DB93D, "E9FC9E0200"  

invoke WriteMemHexes, 0x7120583E,  
"895C82FC8BC35B66813B90E8750D66C703FF158B55FC895302EB15803B897410803B8B740B66C703FF258B55FC895302C3"  

invoke BPHWS, 0x711E5CC9, 1  

invoke ESTO  

invoke BPHWC 0x711E5CC9  
```

如果非 `S` 使用的时候,先结束 `explorer.exe` 进程

使用可能让程序没有焦点,从新启动从新运行就可以了...好象有点点 `Bug`,我用这种方法已经成功脱了主程序,如果有 `sdk`,手工在程序导入表加如 `v20050910.epe` 这个 `dll`,`SDK` 就和没有一样了...![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

这种类型的壳写脚本是巨爽的事情,直接用绝对地址...

`BTW`:老王的壳好象越来越往后了,`0910` 加密强度还没有 `3.14` 好,稳定性也不怎样比 `3.14` 好...不过似乎这个版本在兼容性上有些处理,可能以后也要向非注入发展,没办法,杀毒软件的误抱用户往往也会相信的...![](img/28f5e5b1c17d493bd22adb028ae8a71b.jpg)

查点忘了...`OEP` 这个东西,如果是 `S`,直接用 `PEID` 插件搞定,不是,关闭 `Explorer.exe` 进程,再用这个插件,搞定收工...

各位,中秋快乐...电影去了...

`4dd466d0e1e0cc123afbe504da2462ca`
