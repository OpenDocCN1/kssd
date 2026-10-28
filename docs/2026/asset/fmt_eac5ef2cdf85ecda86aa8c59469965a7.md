
# 写了个脚本解释器

纯解释执行，不生成语法树和opcode

控制流程是yacc的状态矩阵，我觉得只要把lex文件改改，不要出现明文脚本文件，应该可以拿来加密算法![](https://bbs.kanxue.com/view/img/face/27.gif)

## 测试代码

```
a=10;

n=3;

while(a-- && n)
{
if(a<4)break;
print("while a="+a);
if(a==6)continue;
if(a==8)while(n>1.5)print("sub while n="+(n=n-0.5));
print("next");
}
print(a+n);
print("end");
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [testscript.rar](javascript:void(0);)

  （571.22kb，37次下载）
