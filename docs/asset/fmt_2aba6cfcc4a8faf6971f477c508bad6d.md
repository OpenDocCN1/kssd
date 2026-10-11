
`004B391C`    `C780 DC000000 1>MOV DWORD PTR DS:[EAX+DC],16`

`004B3926`    `C780 E0000000 0>MOV DWORD PTR DS:[EAX+E0],0B`

`004B3930`    `C780 E4000000 2>MOV DWORD PTR DS:[EAX+E4],20`

`004B393A`    `C780 E8000000 0>MOV DWORD PTR DS:[EAX+E8],7`

`004B3944`    `C780 EC000000 0>MOV DWORD PTR DS:[EAX+EC],6`

### 三、追码分析

1. 用户名、注册码输入注册框时，非数字字符程序自动转成大写，用户名位数大于 5、小于等于 40，注册码：`***-***-***`(固定格式)

2. 用户名与程序名连接(含 2 个空格)，`MD5` 加密

   `WZWGPA` --> `WZWGPAFlash ScreenSaver Maker` --> `MD5` --> `ab88f3f20059220e0a05494a2cfb6877`

3. 注册码第 4 位必须是 0…9 之间的数，根据注册码第 4 位数值查表得到指向 `MD5` 加密字符的定位值。

   `1234567891` --> `4` --> `10、1、16、9、1D` --> `e、a、9、0、6`

   `16、1、22、9、29`

4. `e、a、9、0、6` 成为注册码中的第 2、3、6、7、9 位。

我的用户名：`wzwgpa`

注册码：`*ea-4*90-*6*` (`*`为任意数)

注册信息保存在注册表：`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\ShellSaver`

表达能力差，破文不能写得言简意赅，感谢你看完。

`4dd466d0e1e0cc123afbe504da2462ca`
