
# Web Scripting Editor V2.4

> 原文：[`www.pediy.com/kssd/pediy04/kanxue071.htm`](https://www.pediy.com/kssd/pediy04/kanxue071.htm)

**软件**：`Web Scripting Editor V2.4`

**工具**：`Wdasm89` 和 `Trw2000`

**作者**：`wlq`

**心得体会**：先用静态工具 `wdasm89` 的字符串参考可以快速找到注册码的准确位置，然后用动态工具 `trw2000` 设断点追注册码，真的省时又省力。

## 破解过程

1. 运行 `Web Scripting Editor V2.4`，填写 `Registration` 的三项内容：

   `key:00000000` `Name:wlq` `Company Name:104`

   接着单击 `"Enter key"`。

2. 弹出如下提示框：

   ```
   “You have entered the wrong registration key
    Please make sure that you enter the key correctly”
   ```

3. 启动 `Wdasm89` 装载 `Web Scripting Editor V2.4`, 点击 `Refs--String Data Reference`, 找到 `"You have entered the wrong registration"`，双击之，看到以下内容：

   ```
   ****************************************************************
   * Possible StringData Ref from Data Obj ->"You have entered the wrong registration "
                                                                 ->"key
   Please make sure that you "
                                                                 ->"enter the key correctly"

                                                                 |
   :0041192D BA01964700                  mov edx, 00479601
   :00411932 8D45D4                      lea eax, dword ptr [ebp-2C]
   :00411935 E832040200                  call 00431D6C
   :0041193A FF45CC                      inc [ebp-34]
   :0041193D 8B00                        mov eax, dword ptr [eax]
   :0041193F E850030200                  call 00431C94
   :00411944 FF4DCC                      dec [ebp-34]
   :00411947 8D45D4                      lea eax, dword ptr [ebp-2C]
   :0041194A BA02000000                  mov edx, 00000002
   :0041194F E878060200                  call 00431FCC
   * Referenced by a (U)nconditional or (C)onditional Jump at Address:
   |:00411925(C)
   |
   :00411954 8B4DB0                      mov ecx, dword ptr [ebp-50]
   :00411957 64890D00000000              mov dword ptr fs:[00000000], ecx
   :0041195E 5B                          pop ebx
   :0041195F 8BE5                        mov esp, ebp
   :00411961 5D                          pop ebp
   :00411962 C3                          ret
   ****************************************************************
   ```

4. 往上看，来到：

   ```
   * Possible StringData Ref from Data Obj ->"BRG149BLUE"
                                                           |
   :004118E8 BAF6954700                  mov edx, 004795F6
   :004118ED 8D45D8                      lea eax, dword ptr [ebp-28]
   :004118F0 E877040200                  call 00431D6C
   :004118F5 FF45CC                      inc [ebp-34]
   :004118F8 8D55D8                      lea edx, dword ptr [ebp-28]
   :004118FB 58                          pop eax
   :004118FC E8C1070200                  call 004320C2
   :00411901 50                          push eax
   :00411902 FF4DCC                      dec [ebp-34]
   :00411905 8D45D8                      lea eax, dword ptr [ebp-28]
   :00411908 BA02000000                  mov edx, 00000002
   :0041190D E8BA060200                  call 00431FCC
   ^^^^^^^^^^                            ^^^^^^^^^^^^^^^^^
   :00411912 FF4DCC                      dec [ebp-34]
   :00411915 8D45DC                      lea eax, dword ptr [ebp-24]
   :00411918 BA02000000                  mov edx, 00000002
   :0041191D E8AA060200                  call 00431FCC
   ^^^^^^^^^                             ^^^^^^^^^^^^^^^^
   :00411922 59                          pop ecx
   :00411923 84C9                        test cl, cl
   :00411925 742D                        je 00411954       (此处可跳过 上面弹出错误注册码提示框的那段程序）
   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
   :00411927 66C745C06800                mov [ebp-40], 0068
   ```

   判断上面两处 `call 00431FCC` 肯定是关键 `call`：

   ```
   :0041190D E8BA060200                  call 00431FCC
   :0041191D E8AA060200                  call 00431FCC
   ```

5. 启动 `Trw2000`,填写 `Registration` 的三项内容：`key:00000000` `Name:wlq` `Company Name:24`

6. `BPX 0041190D`，`F5`，接着单击 `"Enter key"`。

7. `bc *`

8. `:0041190D E8BA060200                  call 00431FCC ---------- ( 按 F8 进入 call)`

9. `:00431FCC 55                            push ebp`
    `:00431FE0 E8F7250200                  call 004545DC -----------(再按 F8 进入 call)`

10. `:004545DC 8B10                          mov edx,  [eax]`
    `:004545DE 85D2                          test edx, edx ---------- 下 d edx， 得到注册码。`
    `:004545E0 7419                          je 004545FB`

11. **总结**：`key:BRG149BLUE` `Name:wlq` `Company Name:104`
    姓名和公司名与注册码无关。 `4dd466d0e1e0cc123afbe504da2462ca`
