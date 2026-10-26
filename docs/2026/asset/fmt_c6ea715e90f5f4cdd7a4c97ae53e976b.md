
```
                {                                                                                      //所以现在读出来的内容其实是一个地址，我们还要继续读，直到读取到数据为止。

                        ShowErrorInfo("读取内存数据出错");

                        __leave;

                }

                dwMemAddr = dwSunny + dwOffset2; //the address offset is 0x5560 ^_^~~;

                if (!ReadProcessMemory(hProc, (LPVOID)dwMemAddr, &dwSunny, sizeof(DWORD), &dwReaded)) //由于游戏使用的是二级地址，所以只要再读一次就可以得到我们需要的数据了。

                {

                        ShowErrorInfo("读取内存数据出错");

                        __leave;

                }

                if (bIsRead)

                {

                        SetDlgItemInt(hDlg, IDC_EDT_SUNNY, dwSunny, FALSE);   //成功读出来以后把数据显示到界面。

                        //如果能成功获取数据，设置控件状态可用，防止误写入错误的内存地址

                        EnableWindow(GetDlgItem(hDlg, IDC_BTN_CHEAT), TRUE);

                        EnableWindow(GetDlgItem(hDlg, IDC_EDT_SUNNY), TRUE);

                        EnableWindow(GetDlgItem(hDlg, IDC_BTN_LOCK), TRUE);

                }

                else

                {

                        DWORD dwWritten = 0;

                        DWORD dwNewSunny = GetDlgItemInt(hDlg, IDC_EDT_SUNNY, NULL, FALSE);

                        if (dwReaded == dwNewSunny)  //如果不等于我们设定的值才进行修改。

                        {

                                __leave;

                        }

                        if (!WriteProcessMemory(hProc, (LPVOID)dwMemAddr, &dwNewSunny, sizeof(DWORD), &dwWritten))  //如果是要改变数据的值，只进行写操作。

                        {

                                ShowErrorInfo("写入内存数据出错");

                                __leave;

                        }

                }

        }

        __finally

        {

                if (dwOldProtect != 0)

                {

                        if (!VirtualProtectEx(hProc, (LPVOID)dwMemAddr, sizeof(DWORD), dwOldProtect, &dwOldProtect)) //做完要做的事以后，再把属性设置回去。这一步也可以省略.

                        {

                                ShowErrorInfo("恢复内存属性出错");

                        }

                }

                CloseHandle(hProc);

        }

}
```

[2010-04-12]

国内有汉化版,汉化的版本已经是无限制版本了,而且汉化的质量很棒.

这个游戏本身就是绿色免费的小游戏,只不过官方自己加上了一个loader来显示试用信息而已.

下面教大家一个魔术,把正式版从试用版里变出来,不使用任何修改工具.

从官方网站下载游戏安装程序.原版是英文的,默认安装路径是`C:\Program Files\PopCap Games\Plants vs. Zombies` 如果你更改了安装路径,那么就到对应的路径下.运行`PlantsVsZombies.exe`,会弹出来一个对话框,选择 "Play Trial Game" 来启动游戏,在游戏显示loading画面的时候,再到游戏的安装目录看一下,会发现一个新的文件`popcapgame1.exe`,文件大小是2.86MB,属性为隐藏.这个不是木马,也不是恶意文件,而是正式版的游戏主程序.该文件是用独占方式创建的,不能直接复制.我们使用强大的磁盘管理工具Disk Genius来提取它.我选择的是复制到桌面.OK了,退出游戏.把桌面上的`popcapgame1.exe`重命名为`PlantsVsZombies.exe`,然后覆盖游戏安装目录下的同名文件,再次运行游戏.怎么样?是不是什么提示信息也没有了,而且游戏一切正常,没有任何限制.

为什么一个商业休闲游戏如此脆弱?或者这正是它的卖点所在,想通过让玩家发现所谓的技巧来提升自己的人气么?亦或是游戏发现自己运行在中国境内,知道俺们china ren不好惹,自动就把程序贡献出来了?

![](https://bbs.kanxue.com/upload/attach/201004/48065_9eee2mp6qkfnijy.jpg)

![](https://bbs.kanxue.com/upload/attach/201004/48065_llborct4mfo9m99.jpg)

附件中的`PlantsVsZombies_en.rar`是我提取出来的主程序,解压到游戏安装目录并选择覆盖原来的文件就行了,版本必须是官方的英文原版.

附件中是我写的游戏助手,用来演示修改阳光数,定制用户存档文件.

界面如下

![](https://bbs.kanxue.com/upload/attach/201004/48065_2v6j8l2iwl9eptf.jpg)

![](https://bbs.kanxue.com/upload/attach/201004/48065_krxybwompata9zb.jpg)

游戏中效果

![](https://bbs.kanxue.com/upload/attach/201004/48065_rd2jbwu0ola6lg5.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`01.JPG`](javascript:void(0);) （6.72kb，2354次下载）
- [`02.JPG`](javascript:void(0);) （77.19kb，2358次下载）
- [`03.jpg`](javascript:void(0);) （34.97kb，2347次下载）
- [`游戏存档.rar`](javascript:void(0);) （0.23kb，337次下载）
- [`04.jpg`](javascript:void(0);) （8.52kb，2211次下载）
- [`05.jpg`](javascript:void(0);) （14.13kb，2220次下载）
- [`PlantsVsZombies_en.rar`](javascript:void(0);) （1.04MB，347次下载）
- [`Plants vs Zombies Helper.rar`](javascript:void(0);) （109.15kb，380次下载）
