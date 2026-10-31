
# 《超级解霸2000》试用版的使用限制解除

《超级解霸2000》试用版只能使用 100 次，梁肇新将安装信息存放在两个地方：

## 1. 注册表中

```text
[HKEY-LOCAL-MACHINE\SOFTWARE\MICROSOFT\WINDOWS\CURRENTVERSION\SETUP]
```

`RUNTIME` 的值。

## 2. INI 文件中

路径：`C:\WINDOWS\STHVCD.INI`

```ini
[SETTING]

BERUN=
```
