
代码有点长，看着看着就迷失在代码的丛林里了。这里可以讨个巧，借助免费的反编译引擎来帮忙。

首先获得 `get_reg_no_true()` 这个函数的 `code` object，然后使用 `marshal.dump()` 保存成文件(从 `py2exe` 的源代码哪里学来的)。然后用 `WinHex` 加上 8 个字节的 `file` header。前 4 个字节代表 Python 版本号，2.4 是 `6DF20D0A`。后 4 个字节是 `timestamp`，随便写就是。

接着来到这里：[1a7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8W2M7s2W2@1K9r3!0F1i4K6u0W2L8X3g2@1i4K6u0r3](https://bbs.kanxue.com/elink@5d8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8W2M7s2W2@1K9r3!0F1i4K6u0W2L8X3g2@1i4K6u0r3)

这个据说是 [team509](https://bbs.kanxue.com/elink@d9cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8W2j5h3@1#2x3o6W2Q4x3X3g2U0L8$3#2Q4x3V1j5`.) 做的，它可以免费反编译小于 5KB 的文件。

反编译出的结果：

```
try:
    try:
        mac_address = ''
        strComputer = '.'
        objWMIService = win32com.client.Dispatch('WbemScripting.SWbemLocator')
        objSWbemServices = objWMIService.ConnectServer(strComputer, 'root\\cimv2')
        colItems = objSWbemServices.ExecQuery('Select * from Win32_NetworkAdapter')
        for objItem in colItems:
            if (objItem.MACAddress != None):
                if (('VEN_' in objItem.PNPDeviceID) and ('DEV_' in objItem.PNPDeviceID)):
                    mac_address = objItem.MACAddress
                    break

        if (mac_address == ''):
            mac_address = '00:0A:EB:F5:D4:14'
    except:
        mac_address = '00:0A:EB:F5:D4:14'
    try:
        s = md5.new(mac_address).hexdigest()
        self.regno = md5.new((s[16:] + s[:16])).hexdigest()[6:-6]
    except:
        self.regno = '00e6e95aff213b8e40ff'
    try:
        tmp_result = ''
        self.regno2 = self.conf_user.config.regnum
        s = md5.new(self.regno).hexdigest()
        md5_tmp = md5.new(((s[14:] + 'bb2') + s[:14])).hexdigest()
        reg_tmp = ((((((((md5_tmp[28:] + md5_tmp[16:20]) + md5_tmp[0:5]) + md5_tmp[7:9]) + md5_tmp[5:7]) + md5_tmp[9:16]) + md5_tmp[20:28]) + md5_tmp[8:12]) + md5_tmp[17:21])
        if (self.regno2[0:4] == reg_tmp[0:4]):
            if (self.regno2[4:8] == reg_tmp[4:8]):
                if (self.regno2[7:15] == reg_tmp[7:15]):
                    if (self.regno2[14:] == reg_tmp[14:]):
                        tmp_result = 'ok'
    except:
        pass
    if (tmp_result == 'ok'):
        return 'YES'
    else:
        return 'ERR'
except:
    return 'ERR'
```

算法一目了然！稍微改一下，注册机就可以出炉了。

综上，使用 Python 开发的商业软件，其安全性还值得商榷。抵御攻击的做法是使用第三方库编译成 `native code`，使用代码混淆器，或者修改 Python 源代码防止被反汇编。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
