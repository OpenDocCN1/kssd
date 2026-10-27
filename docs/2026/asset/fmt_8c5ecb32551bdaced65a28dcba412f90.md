
```
                                                                                     object)

  IL_00e0:  ldloc.s    V_8

  IL_00e2:  ldstr      "KeyCode"

  IL_00e7:  ldloc.s    V_5

  IL_00e9:  callvirt   instance void [mscorlib]Microsoft.Win32.RegistryKey::SetValue(string,

                                                                                     object)

  IL_00ee:  ldloc.s    V_8

  IL_00f0:  callvirt   instance void [mscorlib]Microsoft.Win32.RegistryKey::Close()

  IL_00f5:  ldarg.0

  IL_00f6:  ldstr      "Registration Succeeded"          //这里是注册成功

  IL_00fb:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(class [System.Windows.Forms]System.Windows.Forms.IWin32Window,

                                                                                                                                                     string)

  IL_0100:  pop

  IL_0101:  ret                        //这里有返回

  IL_0102:  ldarg.0                      //一定有跳到这的，注册失败的时候跳到这，向上找

  IL_0103:  ldstr      "Registration Failed"            //看到了吧，这就是注册失败的地方

  IL_0108:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(class [System.Windows.Forms]System.Windows.Forms.IWin32Window,

                                                                                                                                                     string)

  IL_010d:  pop

  IL_010e:  ret

} // end of method A::A
```

### 破解总结

怎么样，简单吧，这个程序不适用于爆破，原因，是每次程序向注册表中写的都是用户输入的东西，所以在用控件的时候，控件自身验证总是不正确的注册码，所以就有问题了，而想把`V_4`代替`V_5`却有数据类型转换的问题，也是一件很麻烦的事情。注册机已经做成，是用C#编的，原因无它，可以直接借用`GetHashCode()`这个函数。写的不是很细致，有不明白的语句，去查Visual Studio.net 2003带的MSDN吧。

### 算法注册机

总结一下用C#源码描述如下：

```csharp
string str1,str2,str3,str4;
int hash;

str1 = textBox1.Text;          //得到Product
str2 = textBox2.Text;          //得到Computer ID
str3 = textBox3.Text;          //得到Order ID
str4 = string.Concat(str3, str2, str1);  //连接三个字符串
hash = str4.GetHashCode();        //得到连接好的字符串的哈希代码，即注册码
```

### 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!
