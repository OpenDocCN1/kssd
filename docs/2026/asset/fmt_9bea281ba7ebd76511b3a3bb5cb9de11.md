
```
004297A3    80F9 23         cmp cl,23                                       ; 要求KEY[6]='#'
004297A6    5F              pop edi
004297A7    5E              pop esi
004297A8    5D              pop ebp
004297A9    0F94C0          sete al
004297AC    83C4 30         add esp,30
004297AF    C3              retn                                            ; 返回程序
```

## 算法总结

注册验证非常简单：

1. 注册码长度要求至少为16位，并记为`KEY`
2. 注册码要求 `KEY[3]='$'`（第3位必须为`$`）
3. 注册码要求 `KEY[4]='$'`（第4位必须为`$`）
4. 注册码要求 `KEY[6]='#'`（第6位必须为`#`）
5. 注册码要求 `atoi(KEY[16])+atoi(KEY[1])=0A`（第16位与第1位的和必须为10）
6. 注册码要求 `atoi(KEY[15])+atoi(KEY[2])=0A`（第15位与第2位的和必须为10）

Email未参加计算，注册码只要符合这个规律即可注册成功。

以规律算出的一组可用注册码：`98$$5#3210123421`

## 注册信息

- Email：`gb_1227@163.com` （任意）
- Registration Key：`98$$5#3210123421`

---

版权所有（C）2005 KuNgBiM[DFCG]         Copyright (C) 2005 KuNgBiM[DFCG]

---

Cracked By KuNgBiM[DFCG]  
2005-09-08  
01:20:30 AM

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
