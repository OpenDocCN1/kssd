
# DLL代码

```c
#include<windows.h>
#include<stdio.h>
//调用函数
BOOL GrowPlant(DWORD BaseAddr, DWORD x, DWORD y, DWORD TypePlant) {
LPVOID PlantFunc = BaseAddr + 0x18D70;
\_\_asm {
pushad
push -1 //-1
push TypePlant //植物类型
mov eax, y //y
push x //x
mov ecx, BaseAddr
mov ecx, [ecx+0x355E0C]
mov ecx, [ecx + 0x868]
push ecx
call PlantFunc
popad
}
return TRUE;
}
BOOL GrowZombie(DWORD BaseAddr, DWORD x, DWORD y, DWORD ZombieType) {
LPVOID PlantZombieFunc = BaseAddr + 0x35390;
\_\_asm {
pushad
push x
push ZombieType
mov eax,y
mov ecx,BaseAddr
mov ecx,[ecx+0x355E0C]
mov ecx,[ecx+0x868]
mov ecx,[ecx+0x178] //ebp
call PlantZombieFunc
popad
}
return TRUE;
}
BOOL WINAPI DllMain(HMODULE hInstance, DWORD fdwReason, LPVOID lpReserved) {
DWORD BaseAddr = GetModuleHandle(NULL);
DWORD pid = GetCurrentProcessId();
HANDLE hProcess = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, pid);
switch (fdwReason)
{
case DLL\_PROCESS\_ATTACH:
MessageBoxW(0, L"ProcessAttachDll!", L"window2", 0);
GrowPlant(BaseAddr,5,3,23);
GrowZombie(BaseAddr, 6, 2, 23);
break;
/\* case DLL\_THREAD\_ATTACH:
printf("ThreadAttach!\n");
break;
case DLL\_THREAD\_DETACH:
if (lpReserved == NULL)
{
FreeLibrary(hInstance);
}
break;\*/
case DLL\_PROCESS\_DETACH:
MessageBoxW(0, L"ProcessDeTachDll!", L"window2", 0);
break;
}
return TRUE;
}
```

# 参考资料

1. [你能学会的Cheat Engine零基础入门教程](elink@f08K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4L8W2t1@1P5e0q4#2y4#2m8K9i4K6u0r3i4K6y4r3M7%4m8E0i4K6g2X3K9h3c8Q4y4h3k6X3M7X3!0E0i4K6y4p5x3K6x3K6i4K6u0W2z5e0V1&6i4K6u0W2x3q4)9J5k6e0m8Q4x3U0k6$3k6q4)9#2k6Y4y4G2N6i4u0U0k6g2)9K6c8o6S2U0x3e0R3J5k6o6q4W2y4r3p5^5x3r3y4U0z5h3j5K6y4r3c8X3k6e0V1&6y4U0p5K6y4h3x3J5j5K6t1K6)

2. [【补档】豪哥植物大战僵尸修改教程视频合集](elink@3b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4N6r3f1@1P5e0q4g2y4@1A6F1i4K6y4r3M7q4)9K6c8o6q4Q4x3U0k6$3k6q4)9#2k6Y4y4G2N6i4u0U0k6g2)9K6c8o6S2U0x3e0R3J5k6o6q4W2y4r3p5^5x3r3y4U0z5h3j5K6y4r3c8X3k6e0V1&6y4U0p5K6y4h3x3J5j5K6t1K6)

3. [公布我所找到的所有基址及各种功能实现方法](elink@51cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9h3g2T1j5g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3k6W2)9K6c8Y4A6Q4x3@1b7^5y4K6l9#2x3K6t1J5y4o6q4Q4x3U0k6U0N6q4)9K6c8o6x3K6y4e0f1@1y4o6x3J5x3q4)9J5y4X3I4E0i4K6y4p5x3q4)9J5y4Y4y4U0i4K6y4p5x3q4)9J5y4Y4u0F1i4K6y4p5x3K6m8Q4x3U0k6@1L8W2)9K6c8r3u0S2K9h3c8#2f1r3!0K6N6p5u0J5L8%4N6K6k6i4u0Q4x3U0k6%4L8%4u0V1i4K6y4p5i4K6t1#2c8o6k6Q4x3U0g2n7x3W2)9J5y4f1y4q4i4K6t1#2c8f1k6Q4x3U0g2n7y4q4)9J5y4f1j5K6i4K6t1#2c8o6g2Q4x3U0g2n7c8q4)9J5y4f1u0p5i4K6t1#2b7e0W2Q4x3U0g2o6b7g2)9J5y4f1q4o6i4K6t1$3M7r3&6Q4x3@1b7H3i4K6t1$3M7X3g2V1i4K6g2X3N6r3q4Y4i4K6y4p5x3o6x3%4x3U0p5I4y4K6V1@1z5b7`.`.)

4. [C/C++全栈软件安全课（调试、反调试、游戏反外挂、软件逆向）持续更新中~~~~](elink@819K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4b7Y4V1@1P5e0q4J5y4@1y4I4i4K6u0r3i4K6y4r3M7q4)9K6c8o6p5#2y4W2)9J5y4Y4k6V1i4K6g2X3M7$3!0#2M7X3y4W2i4K6y4p5z5r3x3I4z5o6u0V1x3h3f1@1j5e0R3H3j5$3x3&6k6U0x3@1k6r3k6W2z5e0V1$3x3e0x3#2j5K6u0U0x3U0x3`.)

5. [逆向工程实战 揭秘汇编/反汇编（win32+游戏逆向实战）](elink@7efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4d9X3t1@1x3e0q4H3y4$3k6g2i4K6u0r3i4K6y4r3M7q4)9K6c8o6t1#2i4K6t1$3N6X3c8Q4y4h3k6K6L8%4g2J5j5$3g2Q4x3@1b7^5j5K6p5^5x3X3b7I4k6e0c8S2z5o6m8U0j5K6W2X3x3K6c8V1k6X3f1&6z5e0j5I4x3K6g2U0x3X3x3J5x3H3`.`.)

6. [[原创]常见的几种DLL注入技术](https://bbs.kanxue.com/thread-269910.htm)

7. [C++ 调用dll的方法](elink@beaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2y4S2K9%4g2&6j5g2)9#2k6W2)9#2k6W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9#2y4o6l9K6x3e0M7J5)

8. [初探DLL注入](elink@9b5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5H3x3U0V1K6y4K6j5@1x3R3`.`.)

9. [Dll注入之远程线程注入](elink@e76K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1x3r3I4V1k6Y4u0W2N6W2)9J5k6h3N6A6N6r3u0G2L8$3E0Q4x3X3g2A6L8#2)9J5c8X3&6G2N6r3g2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6g2X3L8%4m8W2M7X3q4@1K9h3&6Y4i4K6g2X3M7%4W2K6N6r3g2E0i4K6u0r3k6r3I4D9i4K6u0V1P5X3S2#2i4K6u0V1M7Y4g2Q4x3X3c8*7K9r3W2Q4x3X3c8&6N6h3q4F1i4K6u0V1j5$3S2W2L8X3N6Q4x3X3c8^5K9h3q4F1i4K6u0V1j5$3S2W2L8X3N6Q4x3X3c8*7K9s2g2Q4x3X3c8J5N6b7`.`.)

目录

- [基础准备](https://bbs.kanxue.com/#基础准备)
- [基址偏移表](https://bbs.kanxue.com/#基址偏移表)
- [常规项目](https://bbs.kanxue.com/#常规项目)
- [卡槽植物](https://bbs.kanxue.com/#卡槽植物)
- [种植无冷却](https://bbs.kanxue.com/#种植无冷却)
- [无限阳光](https://bbs.kanxue.com/#无限阳光)
- [浓雾透视](https://bbs.kanxue.com/#浓雾透视)
- [基本原理](https://bbs.kanxue.com/#基本原理)
- [HOOK](https://bbs.kanxue.com/#hook)
- [除雾代码](https://bbs.kanxue.com/#除雾代码)
- [种植植物](https://bbs.kanxue.com/#种植植物)
- [基本原理](https://bbs.kanxue.com/#基本原理-1)
- [远程线程注入dll函数](https://bbs.kanxue.com/#远程线程注入dll函数)
- [远程线程卸载dll函数](https://bbs.kanxue.com/#远程线程卸载dll函数)
- [关键dll函数](https://bbs.kanxue.com/#关键dll函数)
- [失败代码](https://bbs.kanxue.com/#失败代码)
- [远程线程代码注入(推荐)](https://bbs.kanxue.com/#远程线程代码注入推荐)
- [种植僵尸](https://bbs.kanxue.com/#种植僵尸)
- [基本原理](https://bbs.kanxue.com/#基本原理-2)
- [种植僵尸函数--dll注入版](https://bbs.kanxue.com/#种植僵尸函数--dll注入版)
- [远程代码注入版](https://bbs.kanxue.com/#远程代码注入版)
- [完整代码](https://bbs.kanxue.com/#完整代码)
- [参考资料](https://bbs.kanxue.com/#参考资料)

# 基础准备

1. CheatEngine工具的基本使用

   推荐视频[你能学会的Cheat Engine零基础入门教程](https://bbs.kanxue.com/elink@f59K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4L8W2t1@1P5e0q4#2y4#2m8K9i4K6u0r3i4K6y4r3M7%4m8E0i4K6g2X3K9h3c8Q4y4h3k6X3M7X3!0E0i4K6y4p5x3K6x3K6i4K6u0W2z5e0V1&6i4K6u0W2x3q4)9J5k6e0m8Q4x3U0k6S2L8i4m8Q4x3@1u0$3k6q4)9#2k6Y4y4G2N6i4u0U0k6g2)9K6c8o6S2U0x3e0R3J5k6o6q4W2y4r3p5^5x3r3y4U0z5h3j5K6y4r3c8X3k6e0V1&6y4U0p5K6y4h3x3J5j5K6t1K6)

   将ce官方给的闯关游戏通关即可

2. C/C++和汇编语言基础

   附上[汇编代码转换网站](https://bbs.kanxue.com/elink@a5eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6h3k6#2M7$3g2Q4x3X3g2U0j5g2)9J5c8X3!0F1L8r3W2F1k6g2)9J5k6s2R3^5y4W2)9J5k6r3q4K6M7$3g2E0j5X3I4W2M7W2)9J5k6h3S2@1L8g2)9J5x3$3c8A6M7$3q4K6M7$3g2E0j5X3I4&6)

3. WIN32开发基础

   了解WIN32命名规则,会使用GPT和查找微软官方文档即可

   推荐通过看微软官方文档[Win32 和 C++ 入门](https://bbs.kanxue.com/elink@585K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6D9k6h3q4J5L8Y4N6A6L8U0x3J5i4K6u0r3L8r3g2S2M7X3&6Q4x3X3c8@1L8#2)9J5k6s2m8J5L8$3N6J5j5h3#2Q4x3X3c8X3L8%4u0Q4x3X3c8%4K9h3&6V1L8%4N6K6) 能创建第一个windows程序即可

示例游戏版本: 中文年度加强版1.1.0.1056

主要参考资料

1. [【补档】豪哥植物大战僵尸修改教程视频合集](https://bbs.kanxue.com/elink@927K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4N6r3f1@1P5e0q4g2y4@1A6F1i4K6y4r3M7q4)9K6c8o6q4Q4x3U0k6S2L8i4m8Q4x3@1u0$3k6q4)9#2k6Y4y4G2N6i4u0U0k6g2)9K6c8o6S2U0x3e0R3J5k6o6q4W2y4r3p5^5x3r3y4U0z5h3j5K6y4r3c8X3k6e0V1&6y4U0p5K6y4h3x3J5j5K6t1K6)

2. [C/C++全栈软件安全课（调试、反调试、游戏反外挂、软件逆向）持续更新中~~~~](https://bbs.kanxue.com/elink@27cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4b7Y4V1@1P5e0q4J5y4@1y4I4i4K6u0r3i4K6y4r3M7q4)9K6c8o6p5#2y4W2)9J5y4X3q4E0M7q4)9K6b7Y4k6V1i4K6g2X3M7$3!0#2M7X3y4W2i4K6y4p5z5r3x3I4z5o6u0V1x3h3f1@1j5e0R3H3j5$3x3&6k6U0x3@1k6r3k6W2z5e0V1$3x3e0x3#2j5K6u0U0x3U0x3`.)

3. [逆向工程实战 揭秘汇编/反汇编（win32+游戏逆向实战）](https://bbs.kanxue.com/elink@4ceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4d9X3t1@1x3e0q4H3y4$3k6g2i4K6u0r3i4K6y4r3M7q4)9K6c8o6t1#2i4K6t1$3j5h3#2H3i4K6y4n7N6X3c8Q4y4h3k6K6L8%4g2J5j5$3g2Q4x3@1b7^5j5K6p5^5x3X3b7I4k6e0c8S2z5o6m8U0j5K6W2X3x3K6c8V1k6X3f1&6z5e0j5I4x3K6g2U0x3X3x3J5x3H3`.`.)

# 基址偏移表

部分参考: 公布我所找到的所有基址及各种功能实现方法

基址 `0x00355E0C`

阳光 `+868` `+5578`

金钱 `+950` `+50`
