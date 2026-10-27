
```
                    if (Convert.ToDateTime(strArray[2]) > DateTime.Now)

                    {

                        if ((strArray[3] == "0") ||(Datetime.DateTimeDiff(Convert.ToDateTime(strArray[3]), Convert.ToDateTime(strArray[4])) > 100L))

                        {

                            this.TempLoginUser.Username = this.TextUsername.Text;

                            this.TempLoginUser.Password = this.TextPassword.Text;

                            this.TempLoginUser.Type = (strArray[1] == "1") ? "收费用户" : "普通用户";

                            this.TempLoginUser.Expire = (strArray[1] == "1") ? Convert.ToDateTime(strArray[2]) : DateTime.Now.AddYears(1);

                            if (this.TempLoginUser.Type == "普通用户")

                            {                             MsgBox.ShowInfoBox("普通账号只可以“xxxx”\n若想无限制使用请联系客服购买。");

                            }

                            Reg.SaveReg("username", this.TextUsername.Text);

                            base.DialogResult = DialogResult.OK;

                        }

                        else

                        {                          MsgBox.ShowErrorBox("同一帐号同一时间只能一人登录！\n如果您上次异常退出程序，请1-2分钟后尝试重新登录。");

                        }

                    }

                    else

                    {

                        MsgBox.ShowErrorBox("您的账号使用时间已到，请联系客服续费。");

                        this.TextPassword.Text = "";

                        this.TextUsername.Focus();

                        this.TextUsername.SelectAll();

                    }

                }

                else

                {

                    MsgBox.ShowErrorBox("账号或密码错误");

                    this.TextPassword.Text = "";

                    this.TextPassword.Focus();

                }

            }

            else

            {

                MsgBox.ShowErrorBox("连接登录服务器错误，请稍后再试");

            }

        }

        catch

        {

            MsgBox.ShowErrorBox("程序出现致命错误。请重新安装程序。");

            Environment.Exit(0);

        }

    }

}
```

流程清楚多了，返回的数据结构是这样的

```
1                验证账号或密码错误
0                是否是收费用户
2012-8-24         到期时间
0                是否有人登陆
2012-8-24 7:51:31  请求时间
```

在启动程序调用了`Update.dll`的`ChecUpdate`函数检查版本信息

```
public static void CheckUpdate(int AppID)
{
    string str = GetUrlText(new Uri("a2dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5s2S2Q4x3X3g2U0L8$3#2Q4x3V1k6$3j5h3I4A6k6r3q4@1k6g2)9J5c8Y4k6W2M7W2)9J5k6h3q4K6M7q4)9K6c8Y4y4G2k6Y4c8A6k6q4)9K6c8s2S2^5P5s2R3`."), Encoding.Default, 30);
    if (str != null)
    {
        if (Convert.ToDouble(str) > 4.6)
        {
            MessageBox.Show("软件已升级到 " + str + " 版，点击确定后软件将自动升级到最新版本。");
            Execute(Application.StartupPath + @"\update.exe", "c38K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5s2S2Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4S2^5P5s2S2Q4y4h3k6#2M7r3c8S2N6r3g2Q4x3X3g2J5j5i4u0Q4y4@1x3`." + Path.GetFileName(Application.ExecutablePath), null);
            Environment.Exit(0);
        }
    }
    else
    {
        MessageBox.Show("不能连接到服务器,请联系管理员", "错误", MessageBoxButtons.OK, MessageBoxIcon.Hand);
        Environment.Exit(0);
    }
}
```

搭个asp服务器

建一个`validate/update.asp`文件

里面的数据为 `1 | 1 | 2012-9-24 | 0 | 2012-8-24 7:51:31`

再建一个`validate/ver.asp`文件

里面是版本信息

再把`C:\Windows\System32\drivers\etc`目录下的`hosts`文件改为

```
127.0.0.1   0d3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5s2S2Q4x3X3g2U0L8$3@1
```

测试时又遇到个问题，

![](https://bbs.kanxue.com/upload/attach/201208/470839_9u4pfmgkyj0teyv.png)

可能在启动程序有检查。

分析一下启动程序有个`CheckHosts`函数

![](https://bbs.kanxue.com/upload/attach/201208/470839_i7eal8szgnf1677.png)

![](https://bbs.kanxue.com/upload/attach/201208/470839_idfdx9uh3y8q4xf.png)

启动程序检查`hosts`文件是否改成了`xxxx.com`.

用`ildasm`反汇编一下，Dump保存为`main.il`

找到相应位置

![](https://bbs.kanxue.com/upload/attach/201208/470839_f69jclpx5dg3pqu.png)

保存一下，用`ilasm.exe`编译

再进行测试，成功。完

第一次写这东西，有点乱，大家见谅！

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.png](javascript:void(0);)（2.39kb，166次下载）
- [2.png](javascript:void(0);)（6.12kb，165次下载）
- [3.png](javascript:void(0);)（2.18kb，163次下载）
- [4.png](javascript:void(0);)（5.91kb，164次下载）
- [5.png](javascript:void(0);)（2.87kb，163次下载）
- [6.png](javascript:void(0);)（7.72kb，164次下载）
- [7.png](javascript:void(0);)（68.52kb，162次下载）
- [8.png](javascript:void(0);)（52.62kb，164次下载）
- [9.png](javascript:void(0);)（93.80kb，163次下载）
