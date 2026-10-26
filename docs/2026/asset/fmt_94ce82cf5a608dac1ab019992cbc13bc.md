
```
0040C75B  |.  80C3 11          add bl,11
0040C75E  |.  03C1             add eax,ecx
0040C760  |.  0FBECB           movsx ecx,bl
0040C763  |.  8816             mov byte ptr ds:[esi],dl                     ;  填充入第一位
0040C765  |.  03C1             add eax,ecx
0040C767  |.  0FBED2           movsx edx,dl
0040C76A  |.  03C2             add eax,edx
0040C76C  |.  885E 01          mov byte ptr ds:[esi+1],bl                   ;  填充入第二位
0040C76F  |.  99               cdq
0040C770  |.  83E2 03          and edx,3
0040C773  |.  C646 05 00       mov byte ptr ds:[esi+5],0
0040C777  |.  03C2             add eax,edx
0040C779  |.  5F               pop edi
0040C77A  |.  C1F8 02          sar eax,2                                    ;  除以4，基本是前四位平均
0040C77D  |.  8846 04          mov byte ptr ds:[esi+4],al                   ;  填充入第五位
0040C780  |.  5E               pop esi
0040C781  |.  5D               pop ebp
0040C782  |.  5B               pop ebx
0040C783  \.  C2 0800          retn 8
```

### 第三处：

```
0040626D  |.  E8 1E650000   call 儿童快乐.0040C790                       ;  第二次计算注册码
```

对第一计算结果再次计算，算法完全一样，得到最终注册码。

### 总结：

算法非常简单，上面有分析。注册码形式为`XXXXX-XXXXX-XXXXX-XXXXX-XXXXX`，X全为大写字母。基本是明码比较，对输入码做了简单处理，为了方便找规律，输入`ABCDE-FGHIJ-KLMNO-PQRST-UVWXY`，输入码转化的规律为：

将后7位删除，每个-及前面一个字母删除，重新连接得到字符串，即得`ABCDFGHIKLMNPQRS`，再与真码比较。将真码每四个一组，插入一个字母和-，最后在补足7个字符即可。

从C盘获得信息得到一串字符，填充变为24位，循环取出6位，转化为四位数字，然后加11变为字母，第五位基本是前四位平均，将几组字母连接成16位字母，再算一次变成注册码。只大概看了一下，可能分析有些不准，谅解。

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2011年05月02日 上午 10:11:48

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
