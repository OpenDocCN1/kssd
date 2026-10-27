
`l += c;`

```
}
String s2 = (new StringBuilder()).append(String.valueOf(l % i)).append(String.valueOf(l % j)).append(String.valueOf(l % k)).toString();
return s.equals(s2) || s.equals(s2.replace('0', 'a').replace('1', 'b'));
}
```

### 关键部分的授权对象 `com.A.A.E.A`

```java
public class A
{
    private String licenseKey;
    private String B;
    private String E;
    private static final char C = 45;
    private int A;
    public A(String lic)
    {
        B = "";
        E = "";
        licenseKey = lic != null ? lic.trim() : null;
        if(!validLicense())
            A = -3;
        else
            switch(lic.charAt(0))
            {
            case 69: // 'E'  试用
            case 84: // 'T'
                A = B(lic);
                break;
            case 65: // 'A'
            case 76: // 'L'
            case 83: // 'S'
                A = -1;
                break;
            default:
                A = -3;
                break;
            }
    }
    protected int B(String s)      // 检查试用版本
    {
        int i = com.A.A.E.F.B(s);
        if(i < 4 || i > 5)
            return -3;
        Date date = F(s, i);
        Date date1 = new Date();
        if(date1.compareTo(date) > 0)
            return -2;
        else
            return (int)((date.getTime() - date1.getTime()) / 0x5265c00L);
    }
    private static Date F(String s, int i)
    {
        int j = s.indexOf('-') + 1;
        if(i > 4)
            j = s.indexOf('-', j) + 1;
        j = s.indexOf('-', j) + 1;
        int k = s.indexOf('.', j);
        int l = Integer.parseInt(s.substring(j, k));
        j = k + 1;
        k = s.indexOf('.', j);
        int i1 = Integer.parseInt(s.substring(j, k));
        j = k + 1;
        k = s.indexOf('-', j);
        int j1 = Integer.parseInt(s.substring(j, k));
        Calendar calendar = Calendar.getInstance();
        calendar.set(l, i1 - 1, j1, 0, 0);
        return calendar.getTime();
    }
    public String I()
    {
        return B;
    }
    public String E()
    {
        return E;
    }
    public String H()
    {
        return licenseKey;
    }
    public String G()
    {
        byte abyte0[] = E.getBytes();
        for(int i = 0; i < abyte0.length; i++)
            abyte0[i] = (byte)(abyte0[i] + 2);
        return new String(abyte0);
    }
    private boolean validLicense()   // 检查注册码格式是否正确
    {
        if(licenseKey == null || licenseKey.length() < 3 || licenseKey.charAt(1) != '-')
            return false;
        switch(licenseKey.charAt(0))
        {
        case 69: // 'E'
        case 84: // 'T'
            return fourSplitLicense();
        case 65: // 'A'
        case 76: // 'L'
        case 83: // 'S'
            return twoSplitLicense();  // 正式版
        }
        return false;
    }
    private boolean D(int splitCharLength)
    {
        int j = com.A.A.E.F.B(licenseKey);   // 查看 - 的个数
        if(j < splitCharLength || j > splitCharLength + 1)
            return false;
        int k = A(licenseKey, 0, j);
        if(k == -1 || A(licenseKey, k, 1) > -1)
        {
            return false;
        } else
        {
            B = licenseKey.substring(0, k);
            E = licenseKey.substring(k);
            return true;
        }
    }
    private boolean fourSplitLicense()
    {
        return D(4);
    }
    private boolean twoSplitLicense()
    {
        return D(2);
    }
    public int M()
    {
        return A;
    }
    public String C()
    {
        int i = E.indexOf('#');
        if(i > -1)
            return E.substring(0, i);
        else
            return E;
    }
    public String K()
    {
        int i = E.indexOf('#');
        if(i > -1)
            return E.substring(i + 1);
        else
            return null;
    }
    private static int A(String s, int i, int j)
    {
        int k = i;
        for(int l = 0; l < j; l++)
        {
            k = s.indexOf('-', k);
            if(k == -1)
                return -1;
            k++;
            if(s.length() <= k)
                return -1;
        }
        return k;
    }
}
```

## 最终针对 Java 部分的格式校验

生成的 License 格式说明如下：

`LicenseType-Jversion-user#UserID-nativeHash#JavaHashCode` 对于正式版

试用版

`LicenseType-Jversion-user#UserID-yyyy.mm.dd-days-nativeHash#JavaHashCode`

其中 Java 部分主要针对的是 `nativeHash` 生成 `JavaHashCode` 的校验

### 2. Java 部分算法分析

注意看 `D` 这个类的 `//        if(!H(s2, s1, 7, 38, 11))           // 这个地方是关键点 部分`

这个是可以理解为 `H(licCode, nativeHash, 7, 38, 11)`{

    int sum = 0;
    for (char ch : nativeHash){
        sum += ch;
    }
    String realCode = (sum % 7) append (sum % 38) append (sum % 11);
    compare( licJavaHashCode, realCode);
}

因此 Java 部分的算法分析就是按照指定的 License 格式，生成，最后进行 `nativeHash` 的验证。

通过即可启动程序。

### 2.2 Native 部分算法分析

单纯的实现 Java 注册部分，只能够启动程序，任何实际的 Profile 调试均不起作用。

启动后，发现 JProfiler 控制台 打印了 `JProfiler> Error Licese ....` 具体内容忘记了，

一般这样的话，我们可以定位这些代码都是通过 JNI 采用本地的 动态链接库实现的，

采用 `IDA Pro` 进行动态库的分析。

1）由于使用 Mac OS 操作系统，因此直接安装的是 Mac OS X 版本的 JProfiler

找到本地动态库 `jprofiler.jnilib` 这个文件，拷贝到 Windows 用 `IDA Pro 64bit` 打开。

待续

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
