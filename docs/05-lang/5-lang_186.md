
# 简繁通 4.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6156.htm`](https://www.pediy.com/kssd/pediy06/pediy6156.htm)

**目标软件**：简繁通 4.0 试用版

**下载地址**：`http://www.xdevelop.net/download/JFTV40.zip`

**软件大小**：236KB

**软件授权**：共享软件

**加密方式**：注册码

**适用平台**：WindowsXP/2000/98/95/Me/NT(4.0)+Java(TM) 2 Runtime Environment Version 1.4.0 以上

**破解工具**：Jbuilder9、小颖 JAVA 源代码反编译超级引挚 V1.4 标准版。

**文章作者**：dayone

**破解时间**：2003-07-11 夜

**E-mail**：dayone@wxxd.com

**关于本文**：本文主要目的在于教学，研究 java 程序的反编译，请勿将此教程用于商业目的，否则后果自负。

首先解压 `JFTV40.zip`，再解开其中的 `JFTServer4.jar`，可以看到在解压目录 `JFTServer4\net\xdevelop` 下是主类 `JFTServer.class`（可以由启动程序 `startJFT.bat` 的内容“`java -hotspot -ms64m -mx64m -classpath JFTServer4.jar net.xdevelop.JFTServer`”得知。

解压目录 `JFTServer4` 下的 class 加密过了，于是反主类 `JFTServer.class`,看一下这个文件的源代码,其中，“//”处是我自己加入的注释。

```java
//Source File Name: JFTServer.class （为省篇幅，。。。。。处代码省掉了）

package net.xdevelop;

。。。。。

public class JFTServer extends ClassLoader
    implements ActionListener
{
    。。。。。
    public JFTServer(String O000O0OoO0O0OoOOOO00o[])
    {
        O00oo0oO0o00O0o00O000 = new Hashtable();
        O0ooOooOoO00oOoO0OooO = false;
        O000OOoO0O0oO0O0O0o0o = true;
        O0oOOO00Oo00o0o0o00OO = null;
        try
        {
            O00OO0000OOOoo00O0OoO = O000O0OoO0O0OoOOOO00o;
            byte O00oOOOOO0OO0o000O0oO[] = Ooo0OoO000o0o0O00000o(getClass().getName());
            if(O00oOOOOO0OO0o000O0oO == null)
                return;
            O0O0o0oO00OoOOo00oO0o = new ByteArrayInputStream(O00oOOOOO0OO0o000O0oO);
            O00O00O0O0O0O000OOOo0 = O0o0O0OOOo000OOoooOOO(O0O0o0oO00OoOOo00oO0o);
            if(O00O00O0O0O0O000OOOo0 < 2L)
            {
                System.out.println("Error: there is something wrong with the class file " + getClass().getName());
                return;
            }
            O0OooOo0O0OOoOOoO0Oo0();
            O0OooOo0O0OOoOOoO0Oo0();
        }
        catch(Exception Ooooo0oO00oo0o000oOOo)
        {
            System.out.println("Error: " + Ooooo0oO00oo0o000oOOo.toString());
            System.exit(0);
        }
        try
        {
            Oo0oO0000O00OOOO00oO0 = ResourceBundle.getBundle("resource.template1Resource", Locale.getDefault(), O0Oo00OO0o0O00ooOoOoO);
        }
        catch(MissingResourceException Ooooo0oO00oo0o000oOOo)
        {
            System.out.println(Ooooo0oO00oo0o000oOOo.toString());
            System.exit(0);
        }
        try
        {
            int O0Oo00ooO00Oo0o0OO0oo = O0O0o0oO00OoOOo00oO0o.read();
            int OooOOoOoO0OOOO0oOO00o = O0O0o0oO00OoOOo00oO0o.read();
            String O0ooOo00O00OOOo00oO00 = null;
            if(O0Oo00ooO00Oo0o0OO0oo == 0)
            {
                O000OOoO0O0oO0O0O0o0o = false;
            } else
            {
                O000OOoO0O0oO0O0O0o0o = true;
                if(OooOOoOoO0OOOO0oOO00o != 0)
                {
                    byte O0oo00OOoOO0O0oOOO00o[] = new byte[16];
                    O0O0o0oO00OoOOo00oO0o.read(O0oo00OOoOO0O0oOOO00o);
                    O0ooOo00O00OOOo00oO00 = new String(O0oo00OOoOO0O0oOOO00o, "8859_1");
                }
                String O0OooOO0Ooo0o0O0OO000 = O0OooOo0O0OOoOOoO0Oo0();
                O0OooOo0O0OOoOOoO0Oo0();
                Class Oo0o0OoOO0OOoOo0oOOOO = loadClass(O0OooOO0Ooo0o0O0OO000, true);
                O0oOOO00Oo00o0o0o00OO = Oo0o0OoOO0OOoOo0oOOOO.newInstance();
                byte O0oo00OOoOO0O0oOOO00o[] = new byte[0];
                Class O0OOOOOO000o0oO0oooO0[] = {
                    O0oo00OOoOO0O0oOOO00o.getClass(), Integer.TYPE
                };
                O0OOOOOOo0oOO0OO0o0OO = Oo0o0OoOO0OOoOo0oOOOO.getMethod("update", O0OOOOOO000o0oO0oooO0);
                Class Oo0o0o00OO000Oo00OO0O[] = {
                    OooOoO0000o0ooOOo00oo == null ? (OooOoO0000o0ooOOo00oo = class$("java.io.InputStream")) : OooOoO0000o0ooOOo00oo, Oo0O0OO0OoOo00O0OO00O == null ? (Oo0O0OO0OoOo00O0OO00O = class$("java.lang.String")) : Oo0O0OO0OoOo00O0OO00O
                };
                O0OoooOo0OoOoooo000oO = Oo0o0OoOO0OOoOo0oOOOO.getMethod("init", Oo0o0o00OO000Oo00OO0O);
                if(O0oOOO00Oo00o0o0o00OO != null && OooOOoOoO0OOOO0oOO00o != 0)
                {
                    Object O00O0oO0oO0o00OoOo0Oo[] = {
                        O0O0o0oO00OoOOo00oO0o, O0ooOo00O00OOOo00oO00
                    };
                    O0OoooOo0OoOoooo000oO.invoke(O0oOOO00Oo00o0o0o00OO, O00O0oO0oO0o00OoOo0Oo);
                }
                if(OooOOoOoO0OOOO0oOO00o == 0)
                {
                    O0ooOooOoO00oOoO0OooO = true;
                    OooOO0ooOo000oOo00OO0();
                }
            }
            if(!O0ooOooOoO00oOoO0OooO)
                Oo000OOo0ooO00o00oO0o();
        }
        catch(Exception Ooooo0oO00oo0o000oOOo)
        {
            System.out.println(Oo0oO0000O00OOOO00oO0.getString("Error: ") + Ooooo0oO00oo0o000oOOo.toString());
            System.exit(0);
        }
    }
    。。。。。
    protected Class findClass(String OoO00OooOOo00o0Ooo0oo)
        throws ClassNotFoundException
    {
        Class Oo0o0O000oOOoOOOOOoOo = null;
        byte Oo00oooo0ooO000OoOOOo[] = (byte[])O00oo0oO0o00O0o00O000.get(OoO00OooOOo00o0Ooo0oo);
        if(Oo00oooo0ooO000OoOOOo != null && Oo00oooo0ooO000OoOOOo.length != 0)
            Oo0o0O000oOOoOOOOOoOo = defineClass(OoO00OooOOo00o0Ooo0oo, Oo00oooo0ooO000OoOOOo, 0, Oo00oooo0ooO000OoOOOo.length);
        if(Oo0o0O000oOOoOOOOOoOo == null)
        {
            byte O0oo00OOoOO0O0oOOO00o[] = Ooo0OoO000o0o0O00000o(OoO00OooOOo00o0Ooo0oo);
            if(O0oo00OOoOO0O0oOOO00o != null)
                Oo0o0O000oOOoOOOOOoOo = defineClass(OoO00OooOOo00o0Ooo0oo, O0oo00OOoOO0O0oOOO00o, 0, O0oo00OOoOO0O0oOOO00o.length);
        }
        return Oo0o0O000oOOoOOOOOoOo;
    }
    。。。。。
    public static void main(String O0OooOO0Ooo0o0O0OO000[])
    {
        try
        {
            new JFTServer(O0OooOO0Ooo0o0O0OO000);
        }
        catch(Exception Ooooo0oO00oo0o000oOOo)
        {
            System.out.println(Ooooo0oO00oo0o000oOOo.toString());
            System.exit(0);
        }
    }
    。。。。。
    public static long O0o0O0OOOo000OOoooOOO(InputStream Oo0oOoOooOo0OO0O0O00O)//自校验
    {
         。。。。。
    }
    private void Oo000OOo0ooO00o00oO0o()
    {
         。。。。。
        Class O0oOOoOOO000o000O0Ooo = null;
        try
        {
            O0oOOoOOO000o000O0Ooo = loadClass(OoooO0O000O0ooO0O0O0o, true);//关键断点
        }
        catch(ClassNotFoundException Ooooo0oO00oo0o000oOOo)
        {
            System.out.println(Ooooo0oO00oo0o000oOOo.toString());
            System.exit(0);
        }
        try
        {
            Class O0OOOOOO000o0oO0oooO0[] = {
                O00OO0000OOOoo00O0OoO.getClass()
            };
            Object O00O0oO0oO0o00OoOo0Oo[] = {
                O00OO0000OOOoo00O0OoO
            };
            Method OoO0Oo0oO0OoOOOooooO0 = O0oOOoOOO000o000O0Ooo.getMethod("main", O0OOOOOO000o0oO0oooO0);
            if(OoO0Oo0oO0OoOOOooooO0 == null)
            {
                System.out.println(Oo0oO0000O00OOOO00oO0.getString("main(String argv[]) method can not be found in") + OoooO0O000O0ooO0O0O0o);
                System.exit(0);
            }
            OoO0Oo0oO0OoOOOooooO0.invoke(null, O00O0oO0oO0o00OoOo0Oo);
        }
        catch(IllegalAccessException Ooooo0oO00oo0o000oOOo)
        {
            System.out.println(Ooooo0oO00oo0o000oOOo.toString() + "Can not invoke the main(String) methos.");
            System.exit(0);
        }
        catch(IllegalArgumentException Ooooo0oO00oo0o000oOOo)
        {
```



```java
            System.out.println(Ooooo0oO00oo0o000oOOo.toString() + "Can not invoke the main(String) methos.");
            System.exit(0);
        }
        catch(InvocationTargetException Ooooo0oO00oo0o000oOOo)
        {
            Ooooo0oO00oo0o000oOOo.printStackTrace();
            System.out.println("Fire this error to TDC, Sun(China)");
            System.exit(0);
        }
        catch(Exception Ooooo0oO00oo0o000oOOo)
        {
            System.out.println(Ooooo0oO00oo0o000oOOo.toString() + Oo0oO0000O00OOOO00oO0.getString(" maybe password is wrong!"));
            System.exit(0);
        }
    }
    ......
    }
}
```

如果用别的反编译器可能代码会有些出入，不过没多大关系。

因为经过了混淆处理，反编译过来会有点难以理解，而且会有点小错误，没关系，改掉就行了。我们把加密类解密后再保存成文件，接下来就把上面的 `findClass` 方法改成

```java
protected Class findClass(String OoO00OooOOo00o0Ooo0oo)
        throws ClassNotFoundException
    {
        Class Oo0o0O000oOOoOOOOOoOo = null;
        byte Oo00oooo0ooO000OoOOOo[] = (byte[])O00oo0oO0o00O0o00O000.get(OoO00OooOOo00o0Ooo0oo);
        if(Oo00oooo0ooO000OoOOOo != null && Oo00oooo0ooO000OoOOOo.length != 0) {
            Oo0o0O000oOOoOOOOOoOo = defineClass(OoO00OooOOo00o0Ooo0oo, Oo00oooo0ooO000OoOOOo, 0, Oo00oooo0ooO000OoOOOo.length);
            //add by dayone
            try {
                FileOutputStream fileoutputstream = new
FileOutputStream(String.valueOf(String.valueOf(OoO00OooOOo00o0Ooo0oo)).concat(".class"));
                fileoutputstream.write(O0oo00OOoOO0O0oOOO00o);
                fileoutputstream.close();
            }
            catch (IOException e) {
                Class cls = findSystemClass(OoO00OooOOo00o0Ooo0oo);
                return cls;
            }
            //
        }
        if(Oo0o0O000oOOoOOOOOoOo == null)
        {
            byte O0oo00OOoOO0O0oOOO00o[] = Ooo0OoO000o0o0O00000o(OoO00OooOOo00o0Ooo0oo);
            if(O0oo00OOoOO0O0oOOO00o != null) {
                Oo0o0O000oOOoOOOOOoOo = defineClass(OoO00OooOOo00o0Ooo0oo, O0oo00OOoOO0O0oOOO00o, 0, O0oo00OOoOO0O0oOOO00o.length);
            //add by dayone
            try {
                FileOutputStream fileoutputstream = new
FileOutputStream(String.valueOf(String.valueOf(OoO00OooOOo00o0Ooo0oo)).concat(".class"));
                fileoutputstream.write(O0oo00OOoOO0O0oOOO00o);
                fileoutputstream.close();
            }
            catch (IOException e) {
                Class cls = findSystemClass(OoO00OooOOo00o0Ooo0oo);
                return cls;
            }
            //
        }
        }
        return Oo0o0O000oOOoOOOOOoOo;
    }
```

当然别高兴太早，编译运行出现错误：

```text
Error: there is something wrong with the class file net.xdevelop.JFTServer
```

原想屏蔽掉相关代码，但仔细研究后发现作者对 `JFTServer.class` 进行了校验，改程序总会发生错误！（用二进制编辑器打开 `JFTServer.class`，可以看出这个 `class` 已经被工具修改过了，有好多信息都藏在里面）那就来的偷梁换柱吧，把我们的 `JFTServer.java` 改名，其中的主类也改掉。把原来的 `JFTServer.class` 放到 `\net\xdevelop` 下，修改 `public JFTServer(String O000O0OoO0O0OoOOOO00o[])` 方法中 `byte O00oOOOOO0OO0o000O0oO[] = Ooo0OoO000o0o0O00000o(getClass().getName());` 为 `byte O00oOOOOO0OO0o000O0oO[] = Ooo0OoO000o0o0O00000o("net.xdevelop.JFTServer");` 再次运行就 `OK` 了。（要对其 `ip` 请求才能得到全部 `class` 文件）把解密过的 `class` 文件用小颖 `JAVA` 源代码反编译超级引挚 `V1.4` 标准版批处理反编译，没办法，因为发现一个个反编译的话反编译器要死掉。现在你就有了全部的源代码了，我知道你接下来想干什么~

跟踪程序运行，在 `private void Oo000OOo0ooO00o00oO0o()` 方法的 `O0oOOoOOO000o000O0Ooo = loadClass(OoooO0O000O0ooO0O0O0o, true);` 处断点可以看到程序调用了类 `Io0o0oO0O11I`。也就是说我们找到真正的主类了。我们看其中的 `main` 方法：

```java
    public static void main(String l00OO0OlII0Il[])
    {
        System.out.println("JFT Server V4.0 (built 2003.6.30 10:00)");
        System.out.println("JFT server starting ...");
        if(!lIoo0O1IloOIO())
            System.exit(-1);
        if(!l00oOOl1Io0ll.l1IOOOo0lOO0l())//关键方法
        {
            System.out.println("JFT Server V4.0(trial) Started.");
            System.out.println("-------------------------NOTE-------------------------------------\r\nIt's a unregistered software.\r\n-----------------------------------------------------------------");
        } else
        {
            System.out.println("JFT Server V4.0 Started.");
        }
        try
        {
            lIo0o0oO0O11I l10oIll11o01O = new lIo0o0oO0O11I();
            l10oIll11o01O.start();
        }
        catch(Exception loIlOl01OlOIl)
        {
            System.out.println("JFT server could not load successful: ".concat(String.valueOf(String.valueOf(loIlOl01OlOIl.getMessage()))));
        }
    }
```

于是来到关键类 `l00oOOl1Io0ll`：

```java
package l1OOoOIoo10OI.l0l110Il0lI11.l10O0o0o0Olo1.lOolI0ll0II00;

import java.io.PrintStream;
import java.security.MessageDigest;
import java.util.Date;
import l1OOoOIoo10OI.l0l110Il0lI11.l10O0o0o0Olo1.lO01IO1I1l1OO.l01oIl11OI0Ol;

public class l00oOOl1Io0ll
{
    public static boolean l0l1l0l01l10l = false;
    public static boolean loI00lIlI1I00 = false;
    private l00oOOl1Io0ll()
    {
    }
    public static void main(String l00OO0OlII0Il[])
    {
        String l1O1lIlol0o0I = l01l0O01O10OI("[www.chinakingonline.com",](http://www.chinakingonline.com",) "standard");
        System.out.print(l1O1lIlol0o0I);
    }
    public static String l0O0lIo11ooIo()    //取得正确的注册码，做 java 版注册机可以直接拿来用。
    {
        String l1Ololl11Illl = l01oIl11OI0Ol.lIOlO00lIolOl;  //取得 WEB_SITE 的值 : 站点域名或 IP 地址
        String l1O1lIlol0o0I = l01l0O01O10OI(l1Ololl11Illl, l01oIl11OI0Ol.l10olIll0Il0o); // l01oIl11OI0Ol.l10olIll0Il0o 为 JFT_TYPE 的值
        return l1O1lIlol0o0I;
    }
    private static String l01l0O01O10OI(String l1Ololl11Illl, String lO0oOo01o0OIo) //计算注册码
    {
        String llII0O0IO1oII = String.valueOf(String.valueOf((new StringBuffer(String.valueOf(String.valueOf(l1Ololl11Illl)))).append(lO0oOo01o0OIo).append("4.0")));
        String l1O1lIlol0o0I = "";
        try
        {
            MessageDigest l1II1o00111Oo = MessageDigest.getInstance("SHA-1"); //采用 SHA-1 算法
            byte lO0101oOooolI[] = l1II1o00111Oo.digest(llII0O0IO1oII.getBytes());
            for(int l1lolIlo0l01l = 0; l1lolIlo0l01l < lO0101oOooolI.length; l1lolIlo0l01l++)
            {
                String lO010IIol010I = Integer.toHexString(lO0101oOooolI[l1lolIlo0l01l]);   //转为 16 进制
                //取 lO010IIol010I 值的最后一位加到 l1O1lIlol0o0I 值的后面
                l1O1lIlol0o0I = String.valueOf(l1O1lIlol0o0I) + String.valueOf(lO010IIol010I.substring(lO010IIol010I.length() - 1));
            }
            String s1 = l1O1lIlol0o0I;
            return s1;
        }
        catch(Exception loIlOl01OlOIl)
        {
            System.out.println("Start JFTServer LIC Error:".concat(String.valueOf(String.valueOf(loIlOl01OlOIl.getMessage()))));
        }
        String s = "";
        return s;
    }
    public static boolean l1IOOOo0lOO0l()
    {
        Date l001OI0O1l1OO;
        String l1O1lIlol0o0I = l0O0lIo11ooIo();
        if(l01oIl11OI0Ol.lI1llI0O1O0Io.equals(l1O1lIlol0o0I))
            break MISSING_BLOCK_LABEL_97;
        l001OI0O1l1OO = new Date();
        if(l001OI0O1l1OO.getTime() <= 0xf758bb6c00L) goto _L2; else goto _L1
_L1:
        System.out.println("The software was expiered, please download new version!");
        System.out.println("Press Ctrl+C to end the program!");
        Exception exception;
        try
        {
            Thread.sleep(0x186a0L);
        }
        finally
        {
            System.exit(-1);
        }
        goto _L2
        exception;
_L2:
        if(l01oIl11OI0Ol.lIooO0O0OIOol > 50)
            l01oIl11OI0Ol.lIooO0O0OIOol = 50;
        return false;
        return true;
    }
}
```

呵呵，终于完工了。总的来说此软件采用了多种加密方式来保护，值得学习。

```
4dd466d0e1e0cc123afbe504da2462ca
```
