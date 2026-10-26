
```
/ / 生成密码

void CMy010EditorRegisterDlg::OnBnClickedButtonGenerate()
{
    // TODO: 在此添加控件通知处理程序代码
    m_edit_password.SetWindowText(L"");

    CString str;
    srand(time(NULL));

    BYTE k[8] = { 0x12, 0x34, 0x56, 0x9C, 0x90, 0x09, 0x87, 0x65 };

    while (TRUE)
    {
        BYTE k0 = rand() % 0xFF;
        BYTE k6 = rand() % 0xFF;

        // AL = (k[0]^k[6]^0x18 + 0x3D)^0xA7
        BYTE AL = (k0 ^ k6 ^ 0x18 + 0x3D) ^ 0xA7;

        if (AL > 0)
        {
            k[0] = k0;
            k[6] = k6;
            break;
        }
    }

    // ESI = (0x100*(k[1]^k[7] & 0xFF) + k[2]^k[5] & 0xFF)&0xFFFF
    // EAX = ((ESI^0x7892+0x4D30)^0x3421)&0xFFFF / 0xB
    // 判断余数是否为0，为0返回商，不为0返回0
    while (TRUE)
    {
        BYTE k1 = rand() % 0xFF;
        BYTE k7 = rand() % 0xFF;
        BYTE k2 = rand() % 0xFF;
        BYTE k5 = rand() % 0xFF;

        DWORD ESI = (0x100 * (k1 ^ k7 & 0xFF) + k2 ^ k5 & 0xFF) & 0xFFFF;
        DWORD EAX = (((ESI ^ 0x7892) + 0x4D30) ^ 0x3421) & 0xFFFF;

        if (EAX % 0xB == 0 && EAX / 0xB <= 0x3E8)
        {
            k[1] = k1;
            k[7] = k7;
            k[2] = k2;
            k[5] = k5;
            break;
        }
    }

    str.Format(L"%02x%02x-%02x%02x-%02x%02x-%02x%02x", k[0], k[1], k[2], k[3], k[4], k[5], k[6], k[7]);
    m_edit_password.SetWindowText(str.MakeUpper());
}
```

以上代码只是让你遇到那两个`JE`和一个`JA`跳转指令可以使它不成立，这个雏形生成的密码并没有与用户名进行关联，在下一小节当中会分析密码与用户名之间的联系。

## 0x03 深入分析算法

在上一小节当中，通过穷举法来找出符合要求的密码。下面将会为大家分析下用户名和密码之间的关系。

![](https://bbs.kanxue.com/upload/attach/202001/754946_5YXBEA4HBAGAEVN.jpg)

咱继续，接下来这一小段将`ECX`与`0x2`进行对比，其实好像也没什么用，我们看 JMP 跳转后的指令，如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_NED8X34MWG8M2QC.jpg)

这一段主要是将用户名的字符串转为`ASCII`版，在接下来的一个调用`CALL`后，`EAX`中为用户名字符串地址，如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_FA28ZKN5R7Y3V9A.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/754946_GSUZBH5M4NS8GMG.jpg)

在上图中的`ds:[edi+0x1C]`处其实是前面处理`k[0]和k[6]`那个`CALL`后的返回值（即`AL`的值），所以我们需要将代码中的

```
if (AL > 0)
{
    k[0] = k0;
    k[6] = k6;
    break;
}
```

改为如下：

```
if (AL > 0xA)
{
    k[0] = k0;
    k[6] = k6;
    break;
}
```

接下来就是要做的就是那个对用户名处理的`CALL`，通过分析知道，这个`CALL`的返回值类似于哈希值，我们需要得到这个值，回为这个值需要和`k[4]、k[5]、k[6]、k[7]`发生关系，我们不妨声明这样一个函数，用来处理`ASCII`码版的用户名字符串，这里有个取巧的办法，那就是利用`IDA`，将`010 Editor`载入`IDA`中，我们需要先获取处理这个用户名字符串的`CALL`的地址，这个地址可以通过`OD`找到，找到之后，切换到`IDA`，按`g`输入地址回车，再按下`F5`进行翻译，如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_2PGNB4HQSHK4KGT.jpg)

大致浏览一下，并不是太复杂，其中用到了一个数组，也就是上图圈出来的，双击进去，如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_YNVXFNHDXYN3R3A.jpg)

这里显示的比较乱，我们可把地址`02EDD840`转到`OD`里查看，如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_9QNB3U83EWC7SXT.jpg)

利用`OD`的插件将这一段进行数据转换并以`C++`形式复制出来，并将其命名为`dwEncodeArray`，如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_MQKAD6GT6Z55G7T.jpg)

并将原来的代码做下调整，如下：

```
/ / 生成密码

void CMy010EditorRegisterDlg::OnBnClickedButtonGenerate()
{
    using namespace std;

    // 先判断用户名输入是否为空
    if (m_edit_name.GetWindowTextLength())
    {
        m_edit_password.SetWindowText(L"");

        CString str;
        srand(time(NULL));

        int nRet = 0x3E8;
        BYTE k[8] = { 0x12, 0x34, 0x56, 0x9C, 0x90, 0x09, 0x87, 0x65 };

        m_edit_name.GetWindowText(str);

        CStringA stra(str.GetBuffer(0));
        str.ReleaseBuffer();

        string st(stra.GetBuffer(0));
        const char* cs = st.c_str();

        // 用户名进行加密
        DWORD dwKey = EncodeUserName(cs, 1, 0, nRet);

        // CMP k[4],RetValue&0xFF
        // CMP k[5],RetValue>>8&0xFF
        // CMP k[6],RetValue>>16&0xFF
        // CMP k[7],RetValue>>24&0xFF
        k[4] = dwKey & 0xFF;
        k[5] = dwKey >> 8 & 0xFF;
        k[6] = dwKey >> 16 & 0xFF;
        k[7] = dwKey >> 24 & 0xFF;

        while (TRUE)
        {
            BYTE k0 = rand() % 0xFF;
            BYTE k6 = k[6];

            // AL = (k[0]^k[6]^0x18 + 0x3D)^0xA7
            BYTE AL = (k0 ^ k6 ^ 0x18 + 0x3D) ^ 0xA7;

            if (AL > 0xA)
            {
                k[0] = k0;
                k[6] = k6;
                break;
            }
        }

        // ESI = (0x100*(k[1]^k[7] & 0xFF) + k[2]^k[5] & 0xFF)&0xFFFF
        // EAX = ((ESI^0x7892+0x4D30)^0x3421)&0xFFFF / 0xB
        // 判断余数是否为0，为0返回商，不为0返回0
        while (TRUE)
        {
            BYTE k1 = rand() % 0xFF;
            BYTE k7 = k[7];
            BYTE k2 = rand() % 0xFF;
            BYTE k5 = k[5];

            DWORD ESI = (0x100 * (k1 ^ k7 & 0xFF) + k2 ^ k5 & 0xFF) & 0xFFFF;
            DWORD EAX = (((ESI ^ 0x7892) + 0x4D30) ^ 0x3421) & 0xFFFF;

            if (EAX % 0xB == 0 && EAX / 0xB == 0x3E8)
            {
                k[1] = k1;
                k[7] = k7;
                k[2] = k2;
                k[5] = k5;
                break;
            }
        }

        str.Format(L"%02x%02x-%02x%02x-%02x%02x-%02x%02x", k[0], k[1], k[2], k[3], k[4], k[5], k[6], k[7]);
        m_edit_password.SetWindowText(str.MakeUpper());
    }
    else
    {
        MessageBox(L"Please enter name", L"warning", MB_OK);
    }
}
```

`EncodeUserName`函数如下：

```
int __cdecl CMyMy010EditorRegisterDlg::EncodeUserName(const char *a1, int a2, char a3, unsigned __int16 a4)
{
    const char *v4; // edx
    signed int v5; // esi
    signed int v6; // edi
    unsigned __int8 v7; // bl
    int v8; // eax
    int v9; // ecx
    int v10; // ecx
    int result; // eax
    unsigned __int8 v12; // [esp+8h] [ebp-10h]
    unsigned __int8 v13; // [esp+Ch] [ebp-Ch]
    unsigned __int8 v14; // [esp+10h] [ebp-8h]
    int v15; // [esp+14h] [ebp-4h]

    v4 = a1;
    v15 = 0;
    v5 = strlen(a1);
    v6 = 0;

    if (v5 <= 0)
        return 0;

    v12 = 0;
    v13 = 0;
    v7 = 15 * a4;
    v14 = 17 * a3;

    do
    {
        v8 = toupper((unsigned __int8)v4[v6]);
        v9 = v15 + dwEcodeArray[v8];

        if (a2)
            v10 = dwEcodeArray[v13]
                + dwEcodeArray[v7]
                + dwEcodeArray[v14]
                + dwEcodeArray[(unsigned __int8)(v8 + 47)] * (dwEcodeArray[(unsigned __int8)(v8 + 13)] ^ v9);
        else
            v10 = dwEcodeArray[v12]
                + dwEcodeArray[v7]
                + dwEcodeArray[v14]
                + dwEcodeArray[(unsigned __int8)(v8 + 23)] * (dwEcodeArray[(unsigned __int8)(v8 + 63)] ^ v9);

        result = v10;
        v15 = v10;
        v13 += 19;
        ++v6;
        v14 += 9;
        v7 += 13;
        v12 += 7;
        v4 = a1;
    } while (v6 < v5);

    return result;
}
```

编译运行后如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_8UUDGSJX3UEV9JW.jpg)

这样就达到了任意用户名注册！

## 0x04 去除网络验证

已加去除网络验证这一小节，感谢`yezhulove`给出的建议！在看到`yezhulove`的建议后，并没立即分析网络验证这块，原因是之前验证成功了，无法弹出网络验证失败错误提示。但今天在打开`010 Editor`时，发现它弹出如下界面：

![](https://bbs.kanxue.com/upload/attach/202001/754946_JST6RRFHMND5ZD8.jpg)

正好把这一部分补上，出现这种情况是`010 Editor`会把该密码上传到它的服务器，如果发现不是通过正常渠道获取得的密码，它会把这个密码拉黑，出现上述这种情况你也不要慌，接下来我会为大家分析怎么来去除网络验证。

打开`OD`，把`010 Editor`拖进来，`F9`运行起来，点击`Check License`，会断在如下地方：

![](https://bbs.kanxue.com/upload/attach/202001/754946_E9CSVMB67UUFNRE.jpg)

因为之前下的断点我并没删除，`F8`来到`00E08229`处，如果按照正常来说，在调用完这两个`CALL`后，`EAX`中的值要为`0xDB`，而此时变成了`0x113`。第一个`CALL`调用后，`EAX`的值为`0x2D`，这个没问题，那么问题可能出现在第二个`CALL`内。这个`CALL`本应该把`EAX`的值修改为`0xDB`，而它却把它修改成了`0x113`，我们不妨跟进这个`CALL`内，如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_FFQ9PWRCXJ476R9.jpg)

`F8`运行到此处后，发现此处的`JE`指令跳转不成功，接下来的`mov eax,0x113`代码将被执行并返回，所以需将`JE`指令改为`JMP`。改完之后，该`CALL`最后返回`0xDB`，而不是`0x113`。在后面你会发现即使修改了这处，它还是一样会验证失败，原因如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_9YC248FQBX8M49S.jpg)

这个`JNS`指令它会跳到失败处，也就是那个字符串提示你的密码已被删除。注意图中的`EAX`的值，它为 -1。同时，在运行`00E0824B`的`CALL`指令时会卡一下，如下：

![](https://bbs.kanxue.com/upload/attach/202001/754946_88C3Y2DJZ364DZ8.jpg)
