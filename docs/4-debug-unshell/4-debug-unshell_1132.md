
# SSReader

> 原文：[`www.pediy.com/kssd/pediy04/kanxue393.htm`](https://www.pediy.com/kssd/pediy04/kanxue393.htm)

工具：

`peditor1.7`

`Ultraedit 9.0(最新版～！呵呵！不过那个版本不重要)`

`fi2.45`

参考资料：看雪论坛精华 3

`SSReader` 中 `pdg2.dll` 被 `UPX` 加壳后修改，大多数为 `UPX0` 之类该为 `UPX?` 只要将对应该还原即可。但是 `pdg2.dll` 将 `UPX?` 全为 0 所以在 `fi` 里面 `sectionName` 里面为空。`Ultraedit` 也找到修改出。我用 `peditor` 修改 `sectionName`，随便该一个四个字节的符号。接着再在 `ultraedit` 里面将其对应的该为 `UPX0`～`UPX1`（注意大写），然后在 `0x3e0H` 处找到 `0c 09` 的代码，将其前面的四个字节该为 `UPX!`。最后用 `upx -d` 解决！收工！！！！

感谢： 所有写过心得的朋友！！！！希望你们再接再厉！！～～～

呵呵：）我捡现成的！！

```
4dd466d0e1e0cc123afbe504da2462ca
```
