
```cpp
RegCreateKey(HKEY_CURRENT_USER, "Software\\友锋图像处理系统\\Application", &hk);
RegSetValueEx(hk, "Order", 0, REG_SZ, Order, 9);
RegSetValueEx(hk, "Set1", 0, REG_BINARY, set1, 30);
RegSetValueEx(hk, "Set2", 0, REG_BINARY, set2, 30);
RegSetValueEx(hk, "Hint", 0, REG_SZ, user, 107);
RegSetValueEx(hk, "Set3", 0, REG_BINARY, set3_2, 30);
RegCloseKey(hk);

RegOpenKey(HKEY_CURRENT_USER, "Software\\友锋图像处理系统", &hk);
RegCreateKey(HKEY_CURRENT_USER, "Software\\友锋图像处理系统\\User", &hk);
RegSetValueEx(hk, "Value", 0, REG_SZ, user, 107);
RegCloseKey(hk);

MessageBox("破解成功！", "恭喜", MB_OK);
UpdateData(false);
}
```

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007年06月21日 23:44:20

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
