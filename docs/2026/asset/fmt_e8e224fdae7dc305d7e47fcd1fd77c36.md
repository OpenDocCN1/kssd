
```c
sn1= suanfa1( chengxu,constant);
sn2=suanfa2(gd,dd);
sn3= suanfa1( name,sn1);
sn4=suanfa2( name,sn2);
itoa(sn4, code, 10);
itoa(sn3, reg, 10);
strncpy(reg1,reg,5);
strncpy(code1,code+1,5);
strcat(code1,"-");
strcat(code1,reg1);
printf("注册码是：");
printf("%s",code1);
system("PAUSE");
}
```

---

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
