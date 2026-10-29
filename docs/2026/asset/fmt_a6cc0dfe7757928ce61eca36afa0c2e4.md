
```
Imports System.Text
Imports System.Security.Cryptography

Public Class Utils

    Public Shared Function GetLogFilePath() As String
        Return IO.Path.Combine(Environment.CurrentDirectory, "Log.txt")
    End Function

    Public Shared Function DecryptString(EncryptedString As String) As String
        If String.IsNullOrEmpty(EncryptedString) Then
            Return String.Empty
        Else
            Return Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
        End If
    End Function

    Public Shared Function EncryptString(PlainString As String) As String
        If String.IsNullOrEmpty(PlainString) Then
            Return String.Empty
        Else
            Return Encrypt(PlainString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
        End If
    End Function

    Public Shared Function Encrypt(ByVal plainText As String, _
        ByVal passPhrase As String, _
        ByVal saltValue As String, _
        ByVal passwordIterations As Integer, _
        ByVal initVector As String, _
        ByVal keySize As Integer) _
        As String

        Dim initVectorBytes As Byte() = Encoding.ASCII.GetBytes(initVector)
        Dim saltValueBytes As Byte() = Encoding.ASCII.GetBytes(saltValue)
        Dim plainTextBytes As Byte() = Encoding.ASCII.GetBytes(plainText)
        Dim password As New Rfc2898DeriveBytes(passPhrase, _
            saltValueBytes, _
            passwordIterations)
        Dim keyBytes As Byte() = password.GetBytes(CInt(keySize / 8))
        Dim symmetricKey As New AesCryptoServiceProvider
        symmetricKey.Mode = CipherMode.CBC
        Dim encryptor As ICryptoTransform = symmetricKey.CreateEncryptor(keyBytes, initVectorBytes)

        Using memoryStream As New IO.MemoryStream()
            Using cryptoStream As New CryptoStream(memoryStream, _
                encryptor, _
                CryptoStreamMode.Write)

                cryptoStream.Write(plainTextBytes, 0, plainTextBytes.Length)
                cryptoStream.FlushFinalBlock()
                Dim cipherTextBytes As Byte() = memoryStream.ToArray()
                memoryStream.Close()
                cryptoStream.Close()
                Return Convert.ToBase64String(cipherTextBytes)
            End Using
        End Using
    End Function

    Public Shared Function Decrypt(ByVal cipherText As String, _
        ByVal passPhrase As String, _
        ByVal saltValue As String, _
        ByVal passwordIterations As Integer, _
        ByVal initVector As String, _
        ByVal keySize As Integer) _
        As String

        Dim initVectorBytes As Byte()
        initVectorBytes = Encoding.ASCII.GetBytes(initVector)

        Dim saltValueBytes As Byte()
        saltValueBytes = Encoding.ASCII.GetBytes(saltValue)

        Dim cipherTextBytes As Byte()
        cipherTextBytes = Convert.FromBase64String(cipherText)

        Dim password As New Rfc2898DeriveBytes(passPhrase, _
            saltValueBytes, _
            passwordIterations)

        Dim keyBytes As Byte()
        keyBytes = password.GetBytes(CInt(keySize / 8))

        Dim symmetricKey As New AesCryptoServiceProvider
        symmetricKey.Mode = CipherMode.CBC

        Dim decryptor As ICryptoTransform
        decryptor = symmetricKey.CreateDecryptor(keyBytes, initVectorBytes)

        Dim memoryStream As IO.MemoryStream
        memoryStream = New IO.MemoryStream(cipherTextBytes)

        Dim cryptoStream As CryptoStream
        cryptoStream = New CryptoStream(memoryStream, _
            decryptor, _
            CryptoStreamMode.Read)

        Dim plainTextBytes As Byte()
        ReDim plainTextBytes(cipherTextBytes.Length)

        Dim decryptedByteCount As Integer
        decryptedByteCount = cryptoStream.Read(plainTextBytes, _
            0, _
            plainTextBytes.Length)

        memoryStream.Close()
        cryptoStream.Close()

        Dim plainText As String
        plainText = Encoding.ASCII.GetString(plainTextBytes, _
            0, _
            decryptedByteCount)

        Return plainText
    End Function

End Class
```

先不管这解密代码。当我编译该 EXE 出来的时候，发现执行文件会报错。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_AY5B69PAA97H6UX.png)
![image](https://bbs.kanxue.com/upload/attach/202301/784203_2YNEUU3432BDVTG.png)

用 `CMD` 后得到了一个异常报错，需要将配置文件放到此目录然后再去执行

![image](https://bbs.kanxue.com/upload/attach/202301/784203_ES5WBYH9SXW98RS.png)

添加后执行不会回显任何东西，寄了。

为什么不输出结果啊，仔细一看原来是缺少了打印代码。

那么最终代码就变成了：

```
Module Module1

    Sub Main()
        Dim Config As ConfigFile = ConfigFile.LoadFromFile("RU_Config.xml")
        Dim test As New SsoIntegration With {.Username = Config.Username, .Password = Utils.DecryptString(Config.Password)}
        Console.WriteLine(Utils.DecryptString(Config.Password))

    End Sub

End Module
```

结果也输出了：

![image](https://bbs.kanxue.com/upload/attach/202301/784203_KM4N3SNSRZCQUQM.png)

获得了一个新的密码 `xRxRxPANCAK3SxRxRx`

但是还有另外一种简单的思路，就是直接从 EXE 入手，在了解完代码之后，知道他只是没有打印这个输出，最终的结果肯定要停留在系统的某个地方。

### `dnspy`

既然是 C# 的代码，直接用 `dnspy` 反编译 exe 即可。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_FRDJ6QHYQ7N56K5.png)

思路就是，从这里开始调用解密算法然后输出结果，那么我们可以在

```
ssoIntegration.Password = Utils.DecryptString(configFile.Password);
```

这一行给他打上断点，让他执行到这里然后断开，然后查看寄存器。

F9 打下断点之后，开始执行程序。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_EZ9RZ9YVCCXJ3SM.png)

断点之后，可以看到在内存中的 `password` 参数就是配置文件中的密码，此处还没有进行解密，需要让他再执行一步。

单步执行 F11 可以看到解密是怎么执行的，逐过程 F10 就是单步执行，就会看到解密完成的密码存在了内存里面。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_M3S44DBHKW3DRAQ.png)

也可以获得密码

## user

接上一步获得了 C.smith 的密码之后 使用 SMB 去连接

```
└─# smbmap -H 10.129.157.43  -u C.Smith -p xRxRxPANCAK3SxRxRx //查看共享
[+] IP: 10.129.157.43:445       Name: 10.129.157.43
Disk                                                    Permissions Comment
----                                                    ---------- --------
ADMIN$                                                  NO ACCESS   Remote Admin
C$                                                      NO ACCESS   Default share
Data                                                    READ ONLY
IPC$                                                    NO ACCESS   Remote IPC
Secure$                                                 READ ONLY
Users                                                   READ ONLY
```

直接去访问自己名字文件下面的文件，获得了第一个 flag。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_CYABWBJMPSX6Z54.png)

## ROOT

我继续搜索这台机器的文件，发现了一个 txt 文件

![image](https://bbs.kanxue.com/upload/attach/202301/784203_C45WJ8MCG7NT8JQ.png)

但是很奇怪的是 0 字节。

而另外一份配置文件是一个导出的备份配置。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_WNDPACJJPFTFDB4.png)

4386 端口……刚开始 `nmap` 扫描的不就是 4386 端口吗，猜想这里的信息应该和另外一个端口有关。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_RF2EB74NVWBTQFD.png)

同时，另外一个目录存在一个 `HqkLdap.exe`。

用 Windows 打开 `Hqkldap.exe` 试试。

```
zhangsan  Release  ♥ 14:48 .\HqkLdap.exe
Invalid number of command line arguments
zhangsan  Release  ♥ 14:48 .\HqkLdap.exe -help
Specified config file does not exist
zhangsan  Release  ♥ 14:48 .\HqkLdap.exe help
Specified config file does not exist
```

获得了两个提示。

1. 命令行参数的数量无效
2. 指定的配置文件不存在

说明运行这个程序需要带配置文件

我想到用刚刚那个 0 字节的文件尝试一下

![image](https://bbs.kanxue.com/upload/attach/202301/784203_6V4UUV3SURU5RWE.png)

第三个报错，请确保安装了可选的数据库导入模块。

发现也是 C# 编译的，丢入 `dnspy` 看看

### `dnspy`

![image](https://bbs.kanxue.com/upload/attach/202301/784203_3WJ8HD29AYC662N.png)

反编译之后，发现其中有如下提示：
