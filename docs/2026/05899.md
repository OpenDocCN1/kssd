
# 脱中国游戏中心大厅程序的壳

下载：`http://202.96.162.68/MyGames/DownLoad/Setup-Plaza.exe`

`Asprotect`已出到1.1了，怎N在1.0打D，我只v重建`ImportTable`的部份，跟技巧省略，自己看教程慢慢摸索，才能M步。@件有程式被`Asprotect`加，一是`iGame.exe`，一是`GameDev.dll`，我以`iGame.exe`槔，dll的し椒疽，只是要c小技巧，用`Procdump32`去分析`iGame.exe`。

你lF]有`.idata` section只有`.rdata` section，`Asprotect`χυ`.rdata` section辛硗庖环Na方式，我不需重建`Import Table`，只要Modify `Image_Thunk_Data`的矩Y即可。

那要如何Modify它呢？然是有力的工具`Imp_list`，@次我用`Imp_list` V1.09，可在老俄那下d，下`.rdata` section的`YVO=4C000`，`Vs=11000`。

步骤：

1. 用`BW2K`，得知`OEP=004224D6`。
2. 绦`Imp_list`，e得出`Address.txt`、`Import0.bin`、`Import1.bin`、`Import2.bin`。
3. 在`oSoft-ICE`下，按`Ctrl-N`叫出`Trw2K`，下`faults off`，再下`CreatefileA do "d \*(esp+4)"`，绦`iGame.exe`，r住後，按`F12/F10`，直到返回主程式，下`BD \*`，`G 004224D6`，下`Suspend`用`Procdump32(dump(full))`，存成`iGdump.exe`，改`EP=000224D6`。
4. 用`Hex Workshop`打`Import1.bin`，偏移`1000`开始方块`728`，⑵滟N至`iGdump.exe`的`.rdata` section的`Raw Offset`，即`004B600`。

@蛹纯擅f成功.

C-Pen  2000\10\08
