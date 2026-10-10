

# 自己动手，弄了个字符串反混淆

> 原文：[`www.pediy.com/kssd/pediy10/71451.html`](https://www.pediy.com/kssd/pediy10/71451.html)

有时候会看到反编译后的程序里面，所有的字符串都是乱乱代码，然后要通过类似下面一样的函数进行类似的解析，才知道明文是啥。  
  

代码:

```
 
   byte\[\] bytes = Convert.FromBase64String(param0);
    return Encoding.Unicode.GetString(bytes, 0, bytes.Length);

```

后来就自己写了个，在 C#.NET 2005 下，  
下面是正则部分  
代码:

```
            string Patt = @"(?<line>IL\_\\w+:\\s+ldstr\\s+)(?<con>(""\\w+(\\=){0,2}"")(\\s+\\+\\s""\\w+(\\=){0,2}"")\*(\\s+\\+\\s""\\w+(\\=){0,2}"")?)(?<clear>\\s+IL\_\\w+:\\s+call\\s+string\\sNS005\\.c02000025::m06000155\\(string\\))";
            Regex regex = new Regex(Patt, RegexOptions.Compiled | RegexOptions.IgnoreCase | RegexOptions.Singleline);
            string strText = regex.Replace(strTxt, new MatchEvaluator(relp));

```

1、NS005\\.c02000025::m06000155 这个就是反编译后，程序带的字符串需要解析的方法名称  
2、我的 IL 的格式如下，这是需要解析的乱乱的麻，如果有不同的格式适当修正下正则  
代码:

```
    IL\_00e7:  ldstr      "MAAAwAH0AIAA3ADAIAAxAEIIAAAIAAwAEYAIAAxAEUA"
    IL\_00ec:  call       string NS005.c02000025::m06000155(string)

```

下面的替换部分  
代码:

```
        private string relp(Match m)
        {
            string strRe = "";
            string strLine = m.Groups\["line"\].Value;
            string strCon = m.Groups\["con"\].Value;
            string clear = m.Groups\["clear"\].Value;
            strCon = replc\_n\_r(strCon).Replace(@"""", "");
            strCon = Base64Format(strCon).Replace("\\n", "\\\\n").Replace("\\r", "\\\\r");//
            strCon = String.Format(@"""{0}""", strCon);
            return strLine + strCon;
        }
        private string replc\_n\_r(string str)
        {
            string strRe = "";
            string patten = @"""\\s+\\+\\s""";
            Regex regex = new Regex(patten, RegexOptions.Compiled | RegexOptions.IgnoreCase | RegexOptions.Singleline);
            strRe = regex.Replace(str, "");
            return strRe;
        }

```

我目前基本上能解决自己的问题，如果用起来不能解决你的问题，留言大家一起与论坛的同学研究