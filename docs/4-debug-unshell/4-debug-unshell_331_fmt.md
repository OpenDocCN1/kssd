
# 3D Exploration

> 原文：[`www.pediy.com/kssd/pediy02/forum258.htm`](https://www.pediy.com/kssd/pediy02/forum258.htm)

哪位高手能脱 `3D Exploration` 的壳，它似乎是用 `ASProtect1.0` 加壳的。而且我追时发现 `3D Exploration` 的注册码判断部分好象在 `ASProtect` 里，不知是否是我水平不够，判断错误。望高手指教。

以下我只簡單的介紹,若有表達不善之處,還請見諒:

目標軟體: `3D Exploration V.1.3.05BETA`

使用工具: `Soft-ICE V.4.01`, `Procdump32`, `Icedump`, `Imp_list`

過程:

1. 先用 `Procdump32` 去分析它:
    - `Size of image:0011E000`
    - `Image Base:400000`

    以上資料待會 dump 的時候需要另在 section 當中並無發現 `.idata` 和一些 section 的 Name

2. 執行 `3D Exploration` 再執行 `Imp_list` 重建 Import Table, `Imp_list` 會警告你沒有發現 `.idata` 不過它還是會產生一個 `Address.txt`, 打開它你會發現 Import Table 的 RVA 在 `93000`。用 `Procdump32` 再去分析 `3D Exploration`, 你會發現在它的 section 當中有 `93000` 的 Virtual Offset, 此時你應該恍然大悟, 原來它有 `.idata` 只是 section name 被刪除而已, 不過這還需要驗證, 我們先用 `Procdump32` 的 Pe 編輯器將 `93000` 的 section name 補上 `.idata`

3. 刪去 `Imp_list` 所產生的檔案, 因為那是不能用的, 待會我們還要重新重建一次, 接下來我們要應付 `Asprotect` 的 CRC Check(因為我們已經動了它的 section)

4. 在 `Soft-ICE` 中加載 `Icedump`, 下 `bpx createfilea do "d *(esp+4)"` 執行 `3D Exploration` 攔住後按 F5 三次, 按 F12 返回, 再按 F10 Trace 來到:

```assembly
:005BA0C5 CMP EAX,[EBP-04]
:005BA0C8 JZ 005BA106        <--r fl z 使其不跳
```

繼續按 F10 來到第一次的一連串 Push, Mov

```assembly
:005BA493 MOV EAX,[EAX+08]
:005BA496 CALL 005B9764
:005BA49B XOR EAX,EAX
```

走到這裡下 `d 00493000` 你會看到內存都是 0, 再往下在 `004930e8` 的地方你會看到指針, 而 hint 的部份則在 `00493bf0~00495fd0`, 下 `f 493bf0 l 495fd0-493bf0 00` 將 hint 清為 0 以免妨礙 `Imp_list` 的工作

5. 再來就是找 OEP 了, 你可以下 `s 0 l ffffffff 8b 65 fc eb 01 e8 8b 45 f8` 找到後 G 到這個位址, 此時已快到程式進入點了, 我這裡是 `00401000`, 在此位址下 `/dump 400000 11E000 C:\3Ddump.exe`, 再按 F5 程式就會出現, 執行 `Imp_list` 重建 Import Table

最後的工作:

1. 將 OEP 改為 `00001000`
2. 將每個 section 的 Raw Offset 改為 Virtual Offset, Raw Size 改為 Virtual Size
3. Import Table RVA=`93000` SIZE=`21E0`
4. 再用 `Hex WorkShop` 打開 `Import0.bin`, `3Ddump.exe` 在 `Import0.bin` 選取 `21e0` 粘貼至 `3Ddump.exe` 的 `93000` 處

`JOHNSON 2000/08/18 4dd466d0e1e0cc123afbe504da2462ca`
