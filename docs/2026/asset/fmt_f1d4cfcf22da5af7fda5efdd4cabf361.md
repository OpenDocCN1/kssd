
# 注册机挑战游戏（A Keygen Challenge for Android）第一第二第三关攻略

今天找到肖大前段时间推荐的一个链接，关于Android的一个apk，详见：

> A Keygen Challenge for Android
> http://www.kanxue.com/bbs/showthread.php?t=185648

目前就玩到了第三关，特发此贴记录一下过程，有兴趣的一起玩起~![](https://bbs.kanxue.com/view/img/face/28.gif)

●

[*]Beginner

[*]Easy

[*]Hard

首先先对界面做一个展示吧，该软件分为5个等级，每个等级适用了不同的算法，当然难度随着等级提高也会提高.前两个算法还是挺简单的，各大牛可以忽略~

![](https://bbs.kanxue.com/upload/attach/201405/409887_0xcqpq929jbvnn5.jpg)

## 1.Beginner

第一关较为简单，采用VTS直接逆向并查看java源码可以找到对应的算法：

![](https://bbs.kanxue.com/upload/attach/201405/409887_lt91xscidjkesy6.jpg)

`string1`，`string2`分别表示输入的用户名还有序列号

根据这段源码，我们很快就确定了算法1的计算流程。首先根据输入的用户名，取用户名的每一位进行一些操作，之后 `int k = Integer.parseInt(paramString2)`，意味着序列号是数字，最后判断是否通过验证就是判断`k`与`i`（用户名运算的值）是否相等。

那么注册机也就很简单了，直接复制出来，输入用户名把`i`输出就是序列号了

``` 
import java.io.IOException;
import java.util.Scanner;

//xbalien 78388
public class Challenge1Verifier {

	public static void main(String[] args) throws IOException{
		Scanner in = new Scanner(System.in);
		String userName = in.next();
		key(userName);
	}
	public static void key(String paramString1){
	    int i = 0;
	    for (int j = 0; j < paramString1.length(); j++){
	      int m = paramString1.charAt(j);
	      i = m ^ i + m * m;
	    }
	    System.out.println(i);

	}
}
```

## 2.Easy

第二关需要结合smali语法，才能解决，光看源码会有一些小细节不好处理~

![](https://bbs.kanxue.com/upload/attach/201405/409887_rhf4nm5hgj2wn9z.jpg)

算法很容易看懂，首先用户名要大于4位，将用户名转为大写字母后，取其中每一位进行运算，生成一个中间变量`str2`，之后对`str2`中的每一个字符进行运算，算出一个结果`j`。

而对于输入的序列，也参与了运算并重`j`中不断的减去其计算值，最后结果当`j`变为0的时候判别为输入正确。

当然，由于反编译结果，出现了一个不可见的字符，因此需要通过smali确定两者分别相加的到底是什么值。一个为`0x30`一个为`0x40`，两者相差16，因此输入的序列如果与`str2`长度相同的话只需要每一位比`str2`都大`0x10`，就可以找到正确序列，因此算法也就很简单了：

``` 
import java.io.IOException;
import java.util.Scanner;

//xbalien AIECIE
public class Challenge2Verifier {

	public static void main(String[] args) throws IOException{
		Scanner in = new Scanner(System.in);
		String userName = in.next();
		key(userName);
	}

	public static void key(String paramString1)
	{
		if (paramString1.length() >= 4){
			String str1 = paramString1.toUpperCase();
			long l = 0L;
			for (int i = 0; i < str1.length(); i++)
				l = 3L * (l + str1.charAt(i)) - 64L;
			String str2 = Long.toString(l);
			System.out.println(str2);
			byte keyByte[] = new byte[str2.length()];
			for (int k = 0; k < str2.length(); k++){
				keyByte[k] = (byte) (str2.charAt(k) + 16);
			}
			System.out.println(new String(keyByte));
		}

	}
}
```

## 3.Hard

这一题开始相对就有一些复杂了，因为编译为源码后只能看到一部分，因此其他部分就是分析smali了。

![](https://bbs.kanxue.com/upload/attach/201405/409887_z13di3e4aprt43r.jpg)

能看到就是`public static String bytesToHex(byte[] paramArrayOfByte)`函数，以及一些内置的关键值

`byte[] secretBytes = this.secretKey.getBytes(Charset.forName("US-ASCII"));`

`String secretKey = "KeygenChallengeNumber3";`

那么关键就是分析`public boolean isValid(String paramString1, String paramString2)`了，暂时没有找到更好的工具查看跳转之类的，于是使用了IDA作为静态分析工具，因此载入后进入`isValid`函数，进行分析：
