
前面2个`nop`是为了要将 `[1000D7B2]` 堆到原来的位址，以防重定位出问题。

## 现在已是正常的

`FindWindow`没打勾: `OllyDbg` - `NOTEPAD.EXE` - `[CPU - main thread, module NOTEPAD]`

`FindWindow`有打勾: - `[CPU - main thread, module NOTEPAD]`

最后附上我用 `Stack` 改法改的完成品

![](https://bbs.kanxue.com/upload/attach/200906/152491_kmfck0v5uwldp91.jpg)

![](https://bbs.kanxue.com/upload/attach/200906/152491_5cp4um275kdq4v7.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

### 上传的附件

- [2.JPG](javascript:void(0);)
  （30.67kb，979次下载）
- [1.JPG](javascript:void(0);)
  （29.70kb，978次下载）
- [advancedolly.rar](javascript:void(0);)
  （281.30kb，479次下载）
