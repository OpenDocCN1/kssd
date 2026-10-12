
# APK 反破解之二：Android APK 签名比对

> 原文：[`www.pediy.com/kssd/pediy12/137500.html`](https://www.pediy.com/kssd/pediy12/137500.html)

转载请注明出处：[`www.blogjava.net/zh-weir/arch...19/354663.html`](http://www.blogjava.net/zh-weir/archive/2011/07/19/354663.html)

## Android APK 签名比对

发布过 Android 应用的朋友们应该都知道，Android APK 的发布是需要签名的。签名机制在 Android 应用和框架中有着十分重要的作用。

例如，Android 系统禁止更新安装签名不一致的 APK；如果应用需要使用 system 权限，必须保证 APK 签名与 Framework 签名一致，等等。在[《APK Crack》](http://www.blogjava.net/zh-weir/archive/2011/06/11/352099.html)一文中，我们了解到，要破解一个 APK，必然需要重新对 APK 进行签名。而这个签名，一般情况无法再与 APK 原先的签名保持一致。（除非 APK 原作者的私钥泄漏，那已经是另一个层次的软件安全问题了。）

简单地说，签名机制标明了 APK 的发行机构。因此，站在软件安全的角度，我们就可以通过比对 APK 的签名情况，判断此 APK 是否由“官方”发行，而不是被破解篡改过重新签名打包的“盗版软件”。

## Android 签名机制

为了说明 APK 签名比对对软件安全的有效性，我们有必要了解一下 Android APK 的签名机制。为了更易于大家理解，我们从 Auto-Sign 工具的一条批处理命令说起。

在[《APK Crack》](http://www.blogjava.net/zh-weir/archive/2011/06/11/352099.html)一文中，我们了解到，要签名一个没有签名过的 APK，可以使用一个叫作 Auto-sign 的工具。Auto-sign 工具实际运行的是一个叫做 `Sign.bat` 的批处理命令。用文本编辑器打开这个批处理文件，我们可以发现，实现签名功能的命令主要是这一行命令：

```
java -jar signapk.jar testkey.x509.pem testkey.pk8 update.apk update_signed.apk
```

这条命令的意义是：**通过 `signapk.jar` 这个可执行 jar 包，以“`testkey.x509.pem`”这个公钥文件和“`testkey.pk8`”这个私钥文件对“`update.apk`”进行签名，签名后的文件保存为“`update_signed.apk`”。**

对于此处所使用的私钥和公钥的生成方式，这里就不做进一步介绍了。这方面的资料大家可以找到很多。我们这里要讲的是 `signapk.jar` 到底做了什么。

`signapk.jar` 是 Android 源码包中的一个签名工具。由于 Android 是个开源项目，所以，很高兴地，我们可以直接找到 `signapk.jar` 的源码！路径为 `/build/tools/signapk/SignApk.java`。

对比一个没有签名的 APK 和一个签名好的 APK，我们会发现，签名好的 APK 包中多了一个叫做 `META-INF` 的文件夹。里面有三个文件，分别名为 `MANIFEST.MF`、`CERT.SF` 和 `CERT.RSA`。`signapk.jar` 就是生成了这几个文件（其他文件没有任何改变。因此我们可以很容易去掉原有签名信息）。

通过阅读 signapk 源码，我们可以理清签名 APK 包的整个过程。

### 1、 生成 MANIFEST.MF 文件：

程序遍历 `update.apk` 包中的所有文件(`entry`)，对非文件夹非签名文件的文件，逐个生成 SHA1 的数字签名信息，再用 Base64 进行编码。具体代码见这个方法：

```
private static Manifest addDigestsToManifest(JarFile jar)
```

关键代码如下：

```
for (JarEntry entry: byName.values()) {
  String name = entry.getName();
  if (!entry.isDirectory() && !name.equals(JarFile.MANIFEST_NAME) &&
      !name.equals(CERT_SF_NAME) && !name.equals(CERT_RSA_NAME) &&
             (stripPattern == null ||!stripPattern.matcher(name).matches())) {
                
    InputStream data = jar.getInputStream(entry);
                while ((num = data.read(buffer)) > 0) {
                    md.update(buffer, 0, num);
                }
                Attributes attr = null;
                if (input != null) attr = input.getAttributes(name);
                attr = attr != null ? new Attributes(attr) : new Attributes();
                attr.putValue("SHA1-Digest", base64.encode(md.digest()));
                output.getEntries().put(name, attr);
    }
}
```

之后将生成的签名写入 `MANIFEST.MF` 文件。关键代码如下：

```
Manifest manifest = addDigestsToManifest(inputJar);
je = new JarEntry(JarFile.MANIFEST_NAME);
je.setTime(timestamp);
outputJar.putNextEntry(je);
manifest.write(outputJar);
```

这里简单介绍下 SHA1 数字签名。简单地说，它就是一种安全哈希算法，类似于 MD5 算法。它把任意长度的输入，通过散列算法变成固定长度的输出（这里我们称作“摘要信息”）。你不能仅通过这个摘要信息复原原来的信息。另外，它保证不同信息的摘要信息彼此不同。因此，如果你改变了 apk 包中的文件，那么在 apk 安装校验时，改变后的文件摘要信息与 `MANIFEST.MF` 的检验信息不同，于是程序就不能成功安装。

### 2、生成 CERT.SF 文件：

对前一步生成的 Manifest，使用 SHA1-RSA 算法，用私钥进行签名。关键代码如下：

```
Signature signature = Signature.getInstance("SHA1withRSA");
signature.initSign(privateKey);
je = new JarEntry(CERT_SF_NAME);
je.setTime(timestamp);
outputJar.putNextEntry(je);
writeSignatureFile(manifest,
new SignatureOutputStream(outputJar, signature));
```

RSA 是一种非对称加密算法。用私钥通过 RSA 算法对摘要信息进行加密。在安装时只能使用公钥才能解密它。解密之后，将它与未加密的摘要信息进行对比，如果相符，则表明内容没有被异常修改。

### 3、生成 CERT.RSA 文件：

生成 `MANIFEST.MF` 没有使用密钥信息，生成 `CERT.SF` 文件使用了私钥文件。那么我们可以很容易猜测到，`CERT.RSA` 文件的生成肯定和公钥相关。

`CERT.RSA` 文件中保存了公钥、所采用的加密算法等信息。核心代码如下：

```
je = new JarEntry(CERT_RSA_NAME);
je.setTime(timestamp);
outputJar.putNextEntry(je);
writeSignatureBlock(signature, publicKey, outputJar);
```

其中 `writeSignatureBlock` 的代码如下：

```
private static void writeSignatureBlock(
      Signature signature, X509Certificate publicKey, OutputStream out)
          throws IOException, GeneralSecurityException {
            SignerInfo signerInfo = new SignerInfo(
                new X500Name(publicKey.getIssuerX500Principal().getName()),
                publicKey.getSerialNumber(),
                AlgorithmId.get("SHA1"),
                AlgorithmId.get("RSA"),
                signature.sign());

        PKCS7 pkcs7 = new PKCS7(
          new AlgorithmId[] { AlgorithmId.get("SHA1") },
            new ContentInfo(ContentInfo.DATA_OID, null),
            new X509Certificate[] { publicKey },
            new SignerInfo[] { signerInfo });

        pkcs7.encodeSignedData(out);
}
```

好了，分析完 APK 包的签名流程，我们可以清楚地意识到：

1、 Android 签名机制其实是对 APK 包完整性和发布机构唯一性的一种校验机制。
2、 Android 签名机制不能阻止 APK 包被修改，但修改后的再签名无法与原先的签名保持一致。（拥有私钥的情况除外）。
3、 APK 包加密的公钥就打包在 APK 包内，且不同的私钥对应不同的公钥。换句话言之，不同的私钥签名的 APK 公钥也必不相同。所以我们可以根据公钥的对比，来判断私钥是否一致。

## APK 签名比对的实现方式

好了，通过 Android 签名机制的分析，我们从理论上证明了通过 APK 公钥的比对能判断一个 APK 的发布机构。并且这个发布机构是很难伪装的，我们暂时可以认为是不可伪装的。

有了理论基础后，我们就可以开始实践了。那么如何获取到 APK 文件的公钥信息呢？因为 Android 系统安装程序肯定会获取 APK 信息进行比对，所以我们可以通过 Android 源码获得一些思路和帮助。

源码中有一个隐藏的类用于 APK 包的解析。这个类叫 `PackageParser`，路径为 `frameworks\\base\\core\\java\\android\\content\\pm\\PackageParser.java`。当我们需要获取 APK 包的相关信息时，可以直接使用这个类，下面代码就是一个例子函数：

```
private PackageInfo parsePackage(String archiveFilePath, int flags){
         
  PackageParser packageParser = new PackageParser(archiveFilePath);
        DisplayMetrics metrics = new DisplayMetrics();
        metrics.setToDefaults();
        final File sourceFile = new File(archiveFilePath);
        PackageParser.Package pkg = packageParser.parsePackage(
                sourceFile, archiveFilePath, metrics, 0);
        if (pkg == null) {
            return null;
        }
        
        packageParser.collectCertificates(pkg, 0); 
        
        return PackageParser.generatePackageInfo(pkg, null, flags, 0, 0);
}
```

其中参数 `archiveFilePath` 指定 APK 文件路径；`flags` 需设置 `PackageManager.GET_SIGNATURES` 位，以保证返回证书签名信息。

具体如何通过 `PackageParser` 获取签名信息在此处不做详述，具体代码请参考 `PackageParser` 中的 `public boolean collectCertificates(Package pkg, int flags)` 和 `private Certificate[] loadCertificates(JarFile jarFile, JarEntry je, byte[] readBuffer)` 方法。至于如何在 Android 应用开发中使用隐藏的类及方法，可以参看我的这篇文章： [《Android 应用开发中如何使用隐藏 API》](http://www.blogjava.net/zh-weir/archive/2011/05/01/349360.html) 。

紧接着，我们就可以通过 `packageInfo.signatures` 来访问到 APK 的签名信息。还需要说明的是 Android 中 `Signature` 和 Java 中 `Certificate` 的对应关系。它们的关系如下面代码所示：

```
pkg.mSignatures = new Signature[certs.length];
for (int i=0; i<N; i++) {
    pkg.mSignatures[i] = new Signature(
    certs[i].getEncoded());
}
```

也就是说 `signature = new Signature(certificate.getEncoded());` `certificate` 证书中包含了公钥和证书的其他基本信息。公钥不同，证书肯定互不相同。我们可以通过 `certificate` 的 `getPublicKey` 方法获取公钥信息。所以比对签名证书本质上就是比对公钥信息。

OK，获取到 APK 签名证书之后，就剩下比对了。这个简单，功能函数如下所示:

```
private boolean IsSignaturesSame(Signature[] s1, Signature[] s2) {
        if (s1 == null) {
            return false;
        }
        if (s2 == null) {
            return false;
        }
        HashSet<Signature> set1 = new HashSet<Signature>();
        for (Signature sig : s1) {
            set1.add(sig);
        }
        HashSet<Signature> set2 = new HashSet<Signature>();
        for (Signature sig : s2) {
            set2.add(sig);
        }
        // Make sure s2 contains all signatures in s1.
        if (set1.equals(set2)) {
            return true;
        }
        return false;
}
```

## APK 签名比对的应用场景

经过以上的论述，想必大家已经明白签名比对的原理和我的实现方式了。那么什么时候什么情况适合使用签名对比来保障 Android APK 的软件安全呢？

个人认为主要有以下三种场景：

1、 程序自检测。在程序运行时，自我进行签名比对。比对样本可以存放在 APK 包内，也可存放于云端。缺点是程序被破解时，自检测功能同样可能遭到破坏，使其失效。
2、 可信赖的第三方检测。由可信赖的第三方程序负责 APK 的软件安全问题。对比样本由第三方收集，放在云端。这种方式适用于杀毒安全软件或者 APP Market 之类的软件下载市场。缺点是需要联网检测，在无网络情况下无法实现功能。（不可能把大量的签名数据放在移动设备本地）。



### 3、 系统限定安装
这就涉及到改 `Android` 系统了。限定仅能安装某些证书的 `APK`。软件发布商需要向系统发布上申请证书。如果发现问题，能追踪到是哪个软件发布商的责任。适用于系统提供商或者终端产品生产商。缺点是过于封闭，不利于系统的开放性。

以上三种场景，虽然各有缺点，但缺点并不是不能克服的。例如，我们可以考虑程序自检测的功能用 `native method` 的方法实现等等。软件安全是一个复杂的课题，往往需要多种技术联合使用，才能更好的保障软件不被恶意破坏。

## 参考资料

`Android` 源码

《Android 中的签名机制》

转载请注明出处：[`www.blogjava.net/zh-weir/arch...19/354663.html`](http://www.blogjava.net/zh-weir/archive/2011/07/19/354663.html)
