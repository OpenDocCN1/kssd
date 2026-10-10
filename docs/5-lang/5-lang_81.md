

# 坛中一个 C#.net Plese Find Password 的全面分析

> 原文：[`www.pediy.com/kssd/pediy11/116583.html`](https://www.pediy.com/kssd/pediy11/116583.html)

针对文:[`bbs.pediy.com/showthread.php?t=110507`](http://bbs.pediy.com/showthread.php?t=110507)  
  
见到该帖后受不得诱惑,从仅有的 28kx 币中拿出 2kx 下载学习,的确很有意思,但也存在疑问,在此一并提出.  
  
该帖中提及的代码叫 Crack Me Hamishebahar,从 GOOGLE 中搜索应该来自伊朗的一个黑客和破解论坛,该作者已经给出了 PASSWORD,但寻求求解它的方法.  
  
下面我们对代码进行分析:  
  
1)首先我们看到的程序是个 loader,它要求您输入一个可多行的 password,不如说是一个解码 KEY,然后通过这个 KEY 解码程序目录下的一个加密 DLL,并通过 Assembly.Load()加载后执行.正确加载执行后会弹出一个伊朗文的提示框,否则英文提示 PASSWORD 错误.  
  
2)下面我们看它的关键:如何验证 KEY 和解码加 DLL:  
  

代码:

```
/// <summary>
        /// 这可是关键！
        /// 1)先取头五个字节来验证 password(其实也就是解码 KEY)
        /// 2)文件最后五字节放着,正式文件开始五字节的解码结果,用来验证解码是否正确,不正确就不白费功夫了.
        /// 3)接下来按照解码 KEY 长度,一段段取来解码并显示进度,最后把剩下部分解码但去掉最后的八个字节,(三个一字节计算因子,一个五字节验证用值)
        /// 4)结果是个 DLL,把它返回
        /// </summary>
       /// <param name="LoadFile"></param>
       /// <param name="Password"></param>
       /// <returns></returns>
        public byte\[\] GetPasswordByte(string LoadFile, string Password)
        {
            List<byte> list = new List<byte>();
            if (File.Exists(LoadFile))
            {
                byte\[\] buffer = new byte\[Password.Length\];
                using (FileStream stream = new FileStream(LoadFile, FileMode.OpenOrCreate))
                {
                    if (this.MainPrg != null)
                    {
                        this.MainPrg.Maximum = Convert.ToInt32(stream.Length);
                        this.MainPrg.Value = 0;
                    }
                    stream.Seek(stream.Length - 8L, SeekOrigin.Begin);
                    this.Rnd1 = Convert.ToByte(stream.ReadByte());  //保存那个 DLL 尾-8 处的一个字节
                    stream.Seek(stream.Length - 7L, SeekOrigin.Begin);
                    this.Rnd2 = Convert.ToByte(stream.ReadByte());  //保存那个 DLL 尾-7 处的一个字节
                    stream.Seek(stream.Length - 6L, SeekOrigin.Begin);
                    this.Rnd3 = Convert.ToByte(stream.ReadByte());  //保存那个 DLL 尾-6 处的一个字节


                    byte\[\] buffer2 = new byte\[5\];
                    byte\[\] password = new byte\[5\];
                    stream.Seek(stream.Length - 5L, SeekOrigin.Begin);
                    stream.Read(buffer2, 0, 5);                     //取那个 DLL 尾-5 处的五个字节

                    stream.Seek(0L, SeekOrigin.Begin);
                    stream.Read(password, 0, 5);                    //取 DLL 开头的五个字节
                      
                    password = this.GetPassword(password.ToList<byte>(), Password); //取文件开头五字节解码


                    if (!this.Check(buffer2, password)) //和验证值比较看是否解码正确,以证明 Password 是对的
                    {
                        return null;
                    }

                    //下面开始真正的解码工作
                    stream.Seek(0L, SeekOrigin.Begin);
                    for (int i = 0; i < stream.Length; i++)
                    {
                        if ((i + Password.Length) >= (stream.Length - 8L))
                        {
                            int count = Convert.ToInt32(stream.Length) - i;
                            buffer = new byte\[count\];
                            stream.Seek((long) i, SeekOrigin.Begin);
                            stream.Read(buffer, 0, count);
                            buffer = this.GetPassword(buffer.ToList<byte>(), Password);
                            list.AddRange(this.SelectList(buffer.ToList<byte>(), 0, count - 8));
                            if (this.MainPrg != null)
                            {
                                this.MainPrg.Value = this.MainPrg.Maximum;
                                Application.DoEvents();
                            }
                            goto Label\_024C;
                        }
                        stream.Seek((long) i, SeekOrigin.Begin);
                        stream.Read(buffer, 0, Password.Length);
                        buffer = this.GetPassword(buffer.ToList<byte>(), Password);
                        list.AddRange(this.SelectList(buffer.ToList<byte>(), 0, Password.Length));
                        if (this.MainPrg != null)
                        {
                            this.MainPrg.Value += Password.Length;
                            Application.DoEvents();
                        }
                        i += Password.Length - 1;
                    }
                }
            }
        Label\_024C:
            return list.ToArray();
        }

```

下面我们看看它调用的两个方法:  
  
代码:

```
/// <summary>
        /// 看看它在做什么？按 Password 的长度逐段调用解码方法解码,开始或最后不够 Password 的长度时按实际长度解码
        /// </summary>
        /// <param name="RET"></param>
        /// <param name="Password"></param>
        /// <returns></returns>
        public byte\[\] GetPassword(List<byte> RET, string Password)
        {
            List<byte> list = new List<byte>();
            for (int i = 0; i < RET.Count; i += Password.Length)//步长为 password 文本的长度,第一次时 RET.Count 为五,其后则取 Password.Length 为 RET.Count,最后取文件剩下的长度
            {
                if ((i + Password.Length) > RET.Count)//如果当前完成位置再按步长增加超出要解码的字节数组长时
                {
                    list.AddRange(this.GetPasswordByte(this.SelectList(RET, i, RET.Count - i), Password));//取剩下的部分解码
                    break;
                }
                list.AddRange(this.GetPasswordByte(this.SelectList(RET, i, Password.Length), Password).ToList<byte>());//按 password 文本的长度,一段段解码
            }
            return list.ToArray();
        }

```

代码:

```
/// <summary>
        /// 解码方法:
        /// 
        /// 对每个字节在 0x00~0xff 范围内进行轮式加减操作，如下：
        /// (0~255 轮操作)加 214
        /// (0~255 轮操作)减 199
        /// 加第二个参数对应字节 (0~255 轮操作)
        /// 减第二人参数的长度 (0~255 轮操作)
        /// 加 3   (0~255 轮操作)
        /// 
        /// 其实是种按字节解码的方法,传入字节的多少没什么关系,只要字节数不大于 Password 长度,分多段解后合起来和一次解都一样
        /// </summary>
        /// <param name="MyByte"></param>
        /// <param name="Password"></param>
        /// <returns></returns>
        private byte\[\] GetPasswordByte(List<byte> MyByte, string Password)
        {
            List<byte> list = MyByte;
            for (int i = 0; i < list.Count; i++)
            {
                list\[i\] = this.GoPByte(this.Rnd3, list\[i\], true);
                list\[i\] = this.GoPByte(this.Rnd2, list\[i\], false);
                list\[i\] = this.GoPByte(Convert.ToByte(Password\[i\]), list\[i\], true);
                list\[i\] = this.GoPByte(Password.Length, list\[i\], false);
                list\[i\] = this.GoPByte(this.Rnd1, list\[i\], true);
               
            }
            return list.ToArray();
        }

```

其中的三个计算因子为：  
代码:

```
/// <summary>
        /// 加法因子:3
        /// 保存在那个 DLL 尾-8 处的一个字节
        /// </summary>
        private int Rnd1;
        /// <summary>
        /// 减法因子:199
        /// 保存在那个 DLL 尾-7 处的一个字节
        /// </summary>
        private int Rnd2;
        /// <summary>
        /// 加法因子:214
        /// 保存在那个 DLL 尾-6 处的一个字节
        /// </summary>
        private int Rnd3;

```

最后我们给出ＤＬＬ文件头和文件尾的五个字节，也即加密后和加密前的五个字节（程序中做验证ＫＥＹ用）：  
文件头：\[41 60 8b 06 fd\]  
文件尾：\[4d 5a 90 00 03\] (注：刚好是不加密的'MZ'头前五个字节)  
  
好了，现在总结一下我们可利用的东西：  
１）用户要提供一个可输入的ＫＥＹ（也即换行，ＴＡＢ外都应该是可见字符）,它的长度和每一位都要参与解码．  
２）我们可以放心确定的是那五个字节可以用来验证我们的算法．  
３）其它字节会是什么？如果作者给我们机会的话，前 64 字节是标准 dos 头，再 64 字节是标准 DOS sub，接下来一个'PE'，事实上除了那五个字节和一个 dos 头中的 e\_lfanew(只能知道它在什么位置，值是什么都不会清楚)都可能不知道，因为都可以变化．当然，这个例子没变．我们可以获得更多的可验证字节．  
４）这个例子中，key 的长度并不很长，没超过我们可验证的前１２８＋２+2 个字节．  
  
只有这么多了，真可以求出它的ＫＥＹ吗？很是质疑！  
首先，这个ＫＥＹ是非可逆的，首先是求长度，其次是逐位求值，可验证的却只有那五个字节．(如果严谨的话)通过上面的分析，它的算法是逐位的，故知道了长度可逐位求解．  
  
我们寄希望于下面两个假设：  
１、dll 前 132 字节都为标准值，作者没变过。  
２、password 长度少于 132 字节。  
  
下面我们给出代码：  
代码:

```
public partial class Form3 : Form
    {
        //从它的 loader 程序中读入,假设都是标准的,用来验证我们的 KEY
        byte\[\] validateByte = new byte\[132\];//包括'PE'及 Machine,假设 CPU 也指定的一样,多了几个可验证字节
        byte\[\] objprogramByte = new byte\[132\];
        
        int\[\] lengthlist;//存求出的 password 可能的长度;

        bool\[\] finded = new bool\[132\]; //保存在该位上是否验证过的字节都可以找到值,以确定最后可能的位;
        public Form3()
        {
            InitializeComponent();
        }

        private void button1\_Click(object sender, EventArgs e)
        {
            //
            if (readValidateByte("Perplex Password Find.exe", "Crypt.dll"))
            {
                Initbyte();
                findPass();
            }
        }

        private void Initbyte()
        {
            for (int i = 0; i < 132;i++ )
            {
                finded\[i\] = true;
            }
        }

        private int findLength()
        {
            int j = 0;
            for (int i = 0; i < 132; i++)
            {
                if (finded\[i\])
                    j++;
            }
            return j;
        }

        private bool readValidateByte(string filename,string dllname)
        {
            bool result = false;
            bool result2 = false;
            if (File.Exists(filename))
            {

                using (FileStream stream = new FileStream(filename, FileMode.OpenOrCreate))
                {
                   
                    stream.Seek(0, SeekOrigin.Begin);
                    stream.Read(validateByte, 0, 132);
                    result = true;
                }
            }
            if (File.Exists(dllname))
            {

                using (FileStream stream2 = new FileStream(dllname, FileMode.OpenOrCreate))
                {

                    stream2.Seek(0, SeekOrigin.Begin);
                    stream2.Read(objprogramByte, 0, 132);
                    result2 = true;
                }
            }
            return result && result2;
        }

        private void findPass()
        {
            TestMe me = new TestMe();
           
            
            for (int x = 0; x < 132; x++)
            {

                for (int i = 0; i < 132; i++) //可验证的只有这么多位
                {
                    bool findValue = false;
                    for (int j = 0; j < 127; j++)//ASCLL 全进来吧,省不了多少,相比我往出摘的话.(真懒!什么人!鄙视一下!)
                    {
                        byte value = objprogramByte\[x\];
                        value = me.GoPByte(3, value, true);
                        value = me.GoPByte(i, value, false);
                        value = me.GoPByte(j, value, true);
                        value = me.GoPByte(199, value, false);
                        value = me.GoPByte(214, value, true);
                        if (value == validateByte\[x\])
                        {
                            //this.textBox1.Text += "第"+x.ToString()+"字节 值为:"+j.ToString()+"位数:"+i.ToString()+"   ";
                            findValue = true;
                            
                        }
                    }
                    if (!findValue)
                    {
                        finded\[i\] = false;
                    }
                }
                //this.textBox1.Text +=  + Environment.NewLine;
            }

            MessageBox.Show(findLength().ToString());
            lengthlist = new int\[findLength()\];
            int vv = 0;
            for (int ii = 0; ii < 132; ii++)
            {
                if (finded\[ii\])
                {
                    this.textBox1.Text += ii.ToString() + Environment.NewLine;
                    lengthlist\[vv\] = ii;
                    vv++;
                }
            }
            writeSerialFile();
            MessageBox.Show("It Cracked!");
        }

        private void writeSerialFile()
        {
            //copy PasswordByte.cs 原作者的文件,只把方法变成 public
            TestMe me = new TestMe();


            for (int x = 0; x < lengthlist.Length; x++)
            {

                byte\[\] buffer = new byte\[lengthlist\[x\]\];
                for (int i = 0; i < lengthlist\[x\]; i++)
                {
                    for (int j = 0; j < 127; j++)//ASCLL 全进来吧,省不了多少,相比我往出摘的话.(真懒!什么人!鄙视一下!)
                    {
                        byte value = objprogramByte\[i\];
                        value = me.GoPByte(3, value, true);
                        value = me.GoPByte(lengthlist\[x\], value, false);
                        value = me.GoPByte(j, value, true);
                        value = me.GoPByte(199, value, false);
                        value = me.GoPByte(214, value, true);
                        if (value == validateByte\[i\])
                        {
                          buffer\[i\] = (byte)j;
                          break;
                        }
                    }
                }
                writeFile(x, buffer);
                
            }
        }

        private void writeFile(int x, byte\[\] buffer)
        {
            string filename = x.ToString() + ".dll";
            using (FileStream stream = new FileStream(filename, FileMode.OpenOrCreate))
            {

                stream.Seek(0, SeekOrigin.Begin);
                stream.Write(buffer, 0, buffer.Length);
                
            }
        
        }

    }

```

说明：  
１)读入其 loader 程序的前 132 字节，或其它任意 exe 前 130(or 132)字节，作为验证标准。  
2)读入加密后的 dll 前 132 字节,以便用作用于我们的方法后和验证标准比对。  
３）先求可能的 password 长度，思路为：０～１２７个可用ＡＳＣＬＬ码，１～132 位字符串儿，代入解码算法比较加密前后的一个字节，找出这一个第一个字节有多少种匹配的可能。接下来看第二个字节。。。我们会发现有些位在某种长度下无论什么ＡＳＣＬＬ码都验证不回去，把它去掉吧，最后，我们找出可验证的 password 长度的交集，我们发现可能的长度只有 15 种。  
４）长度确定了（只有１５种），那我们求它的 password 内容是什么，结果也只有 15 种。  
５）好了，拷入原程序的密码输入框试试吧，其实除了一种之外都含有不可见字符。  
  
最后给出第十种，也就是正确的结果：  
代码:

```
salam Chetori? 
mikhay Mano Crack Koni? 
Mituni? 
Midoni Man Kiam?
Man Passwordam.
Movafagh v sarboland bashi azizam.

```

结论：作者没有修改ＰＥ文件前 132 字节内容，且ＫＥＹ小于这个长度，原本不可逆的ＫＥＹ被用穷举加验证的方法在十几秒内运算出来。（代码见附件，代码的效率还望见谅）  
  
另外：  
本坛的帖子中网友 sessiondiy 很早地给出了结果，遗憾没有 Solution,这里补上并给予致意,也希望如有更好的方法能以指点和交流。  
  
src1.rar

引用: *最初由 **opensrc**发布* ![查看帖子](img/showthread.php?p=834328#post834328) 另外：  
本坛的帖子中网友 sessiondiy 很早地给出了结果，遗憾没有 Solution 

有点久了, 大致上只记得当时我是放弃 DOS 头猜法, 就如你讲的复杂又真搞得出来吗?  
而且网上并无这 CM 的资讯所以无法得知者是否变太.  
  
我用的方法:  一般 dll 文件后面皆是 00  
  1. 找出 Block 长度, 否则免谈.  
     .dll(最后 8byte 不看) 最后是 1A F8 F1 06, 往前找同其值,  
     找到后位置相减即为 Block Len.  (1BFC-1B82=7A)  
  2. 有了长度就可算出 Block 起点  
      看一下起点是落在 1B82 ~ 1BFC 的何处  
     ((1B82 div 7A) + 1)\*7A = 1BA4  
     所以一个完整的被编码 Block 位置在 1BA4 ~ 最后 因 byte 不够, 可参看上一个 Block 即知  
     假设解密后为 00 00 00 00 00 ...............  
  
如此破之