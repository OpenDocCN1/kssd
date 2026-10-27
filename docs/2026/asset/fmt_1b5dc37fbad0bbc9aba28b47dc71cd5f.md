
```
for(ULONG index2=0;index2<subkey->NumberOfKeys;index2++)

{

ListValue(keypath,(PCM_KEY_NODE)((&subkey->ValueData_Offset)[index2]+RootBin));

}
}
```

后注:在`dump hive`时,正常是没有问题的...但如果使用了`combojiang`那篇隐藏注册表里面的那个驱动程序来隐藏注册表...比如它里面隐藏了`Services`键下的子键`Beep`,那我们在`dump Services`时就会失败....也不知道是什么原因,,应该是被什么干扰了吧...不过你可以自己复制原始的`HIVE`文件,在解释它...这个程序再改改应该可以...不过到了这里你应该对`HIVE`文件有个大概的理解了..

![](https://bbs.kanxue.com/upload/attach/200804/121500_54kahw51anfws9f.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [未命名.jpg](javascript:void(0);)

  （26.49kb，1391次下载）

- [hive.rar](javascript:void(0);)

  （1.73kb，393次下载）
