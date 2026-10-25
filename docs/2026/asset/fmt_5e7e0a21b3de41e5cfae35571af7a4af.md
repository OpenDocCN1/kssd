
# [原创]IDA SDK合并jmp乱序插件代码示例阅读

看了下<http://bbs.pediy.com/showthread.php?t=60798>这篇文章和[f14K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0H3k6h3&6J5j5$3g2Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6s2y4Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4o6m8Q4x3V1k6^5z5o6k6Y4M7Y4m8Z5i4K6g2X3k6X3!0J5i4K6g2X3P5o6R3`.](https://bbs.kanxue.com/elink@d21K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0H3k6h3&6J5j5$3g2Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6s2y4Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4o6m8Q4x3V1k6^5z5o6k6Y4M7Y4m8Z5i4K6g2X3k6X3!0J5i4K6g2X3P5o6R3`.)这个插件。

功能主要是:

A small utility that will generate a flowgraph from x86 code -- similar to IDA's built-in funtionality -- which is capable of graphing non-contigous functions (as created by Microsoft's internal optimization tools).

这样可以指定地址和结束地址，来显示这段代码的流程图，但有的变形代码里采用直接跳转来打乱代码显示次序，显得很乱和难以分析，所以我们要合并他们。

很巧IDA里有份合并乱序jmp的文章，这里只合并的绝对跳转，由于IDA资料好少，搞了半天，终于明白了合并jmp的步骤，分享出来，希望能抛砖引玉。
