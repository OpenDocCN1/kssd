
`void` `*``new_func_set(void` `*``arg, void` `*``arg1, void` `*``arg2, void` `*``arg3) {`  `LOGD(``"Enter new_func_set %d"``,current_get_index);`  `current_set_index``+``+``;`  `/``/``set``的时候第二个参数可能为``0``，就像get的时候返回值可能为``0``一样`  `if` `(arg1` `=``=` `0``)` `return` `old_func_set(arg, arg1, arg2, arg3);`  `try` `{`  `memset(header_set,` `0``, HeaderSize);`  `memcpy(header_set, arg1, HeaderSize);`  `memset(end,` `0``, EndSize);`  `memset(middle_set,` `0``, SplitSize);`  `/``/``以八个``0``作为结束，拷贝以返回值偏移``12``个字节的作为开始的内存数据，其实就是中间文字部分`  `memccpy(middle_set, (char` `*``) arg1` `+` `sizeof(char)` `*` `HeaderSize, reinterpret_cast<``int``>(end), SplitSize);`  `void``*` `p_le` `=``memchr(middle_set,reinterpret_cast<``int``>(end),SplitSize);`  `/``/``原返回值中间文字部分的长度`  `int` `src_length` `=` `(char``*``)p_le` `-` `(char``*``)middle_set;`    `int` `current_lines` `=` `0``;`  `/``/``初始化解析文本以“|”作为分割左边 右边部分缓存指针`  `char` `*``left` `=` `static_cast<char` `*``>(calloc(SplitSize, sizeof(char)));`  `char` `*``right` `=` `static_cast<char` `*``>(calloc(SplitSize, sizeof(char)));`    `/``/``读取文件后删除了源文件的，从这里的``buffer``拷贝一个备份来操作`  `char` `*``temp_buffer` `=` `(char` `*``) malloc(sizeof(char)` `*` `file_size` `+` `sizeof(``int``));`  `memcpy(temp_buffer,` `buffer``, sizeof(char)` `*` `file_size` `+` `sizeof(``int``));`  `char` `*``p` `=` `strtok(temp_buffer,` `"\r\n"``);`  `while` `(p !``=` `NULL) {`  `memset(left,` `0``, SplitSize);`  `memset(right,` `0``, SplitSize);`  `char` `*``s` `=` `strstr(p,` `"|"``);`  `static_cast<char` `*``>(memcpy(left, p, strlen(p)` `-` `strlen(s)));`  `right` `=` `strcpy(right, s` `+` `sizeof(char));`  `if` `(current_lines !``=` `0``) {`  `char` `*``convert_str` `=` `static_cast<char` `*``>(malloc(strlen(left)` `*` `2``));`  `memset(convert_str,` `0``, strlen(left)` `*` `2``);`  `int` `length` `=` `UTF8_to_Unicode(convert_str, left);`  `/``/``length` `=``=` `(src_length` `-` `EndSize) &&`  `/``/`                `LOGD(``"src_length - EndSize  %d"` `, src_length` `-` `EndSize);`  `/``/``内存字节的比较`  `if` `(memcmp(middle_set, convert_str, src_length` `-` `EndSize)` `=``=` `0``) {`  `LOGE(``"---> called set_text replace %s to %s   times:%d"``,left,right,current_set_index);`  `LOGD(``"Original str hex at %p === >"``,&middle_set);`  `hexDump(reinterpret_cast<const char` `*``>(middle_set), src_length);`  `void` `*``p` `=` `malloc(strlen(right)` `*` `2``);`  `int` `le` `=` `UTF8_to_Unicode(static_cast<char` `*``>(p), right);`  `LOGD(``"Replacement str hex at %p === >"``,&le);`  `hexDump(reinterpret_cast<const char` `*``>(p), le);`  `/``/``申请空间来重新组合返回值`  `void` `*``temp` `=` `malloc(static_cast<size_t>(HeaderSize` `+` `le` `+` `EndSize));`  `memset(temp,` `0``, static_cast<size_t>(HeaderSize` `+` `le` `+` `EndSize));`  `memcpy(temp, header_set, HeaderSize);`  `memcpy((char` `*``) temp` `+` `HeaderSize, p, static_cast<size_t>(le));`  `memcpy((char` `*``) temp` `+` `HeaderSize` `+` `le, end, EndSize);`  `LOGD(``"Return str hex at %p === >"``,&temp);`  `hexDump(static_cast<const char` `*``>(temp), static_cast<size_t>(HeaderSize` `+` `le` `+` `EndSize));`  `free(convert_str);`  `free(left);`  `free(right);`  `free(temp_buffer);`  `return` `old_func_set(arg, temp, arg2, arg3);`  `}`  `}`  `p` `=` `strtok(NULL,` `"\r\n"``);`  `current_lines``+``+``;`  `}`  `free(left);`  `free(right);`  `free(temp_buffer);`  `return` `old_func_set(arg, arg1, arg2, arg3);`  `}catch (...){`  `LOGE(``"ERRR MENORY"``);`  `return` `old_func_set(arg, arg1, arg2, arg3);`  `}`  `}`

其实这里还有问题没考虑到(随便举例一个)：

- 游戏中很多文本是组合的

  比如 "`Time：12:00`" ,time是一个常量后面的12:00是动态加上去的，按照目前的代码是处理不了这种情况的，可能需要在读取文本中单独加上一个标识，分类讨论嘛

  （后面的版本已经完善这种需求）

大概就这意思，平时空余时间写的demo，欢迎大佬来fork修改提交共同完善这个小工具，这里不适合贴太多代码，详细代码移步

[UnityGameLocalizationDemo](https://bbs.kanxue.com/elink@a1eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2P5r3S2D9P5Y4W2Q4x3V1k6g2L8X3W2@1P5f1N6S2L8h3g2x3L8$3y4S2L8r3W2*7j5i4c8A6L8$3&6p5k6h3#2G2i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3q4H3M7q4)9J5c8Y4y4J5j5#2)9J5c8X3#2S2K9h3&6Q4x3V1k6U0M7s2m8Q4x3V1k6A6L8X3A6W2j5%4c8Q4y4h3k6V1L8$3u0T1P5g2)9J5k6h3y4H3M7l9`.`.)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2020-11-24 15:37

被唱过阡陌编辑

，原因： 更新新的想法，完善文章
