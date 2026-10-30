
- 最终确认 `brushControl` 对应 `chunk_1.appservice.js`，而 `brushControlPlus` 对应 `chunk_2.appservice.js`
- 但是进入 `chunk_2.appservice.js` 却发现其代码仅有 51 行，应该是一个未启用的页面。
- 而在 `chunk_1.appservice.js` 中发现代码引用了 `ble` 蓝牙相关的库函数，看来这个代码文件应该就是我们想要的了。

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_C7AM7RZ7M7DYKCZ.png)

## 定位蓝牙控制函数

- 在该页面中继续寻找开关按钮，定位到了 `开启` 关键字。
- 这一点比较意外，在真正的控制界面中，开启电动牙刷按钮下写的是 `开启`，而非展示页面下显示的 `开关`，在这个地方浪费了点时间，刚开始就应该进入真实的页面去锁定 UI。
- 至此，基本定位到了真正的开启按钮。

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_TZAPRZVKTE5TN3D.png)

- `开启` 关键词是在 `j` 函数中（第 57 行），其功能看起来是控制该按钮的显示。
- 接下来需要查看该按钮所绑定的函数，于是尝试搜索 `j` 函数在哪里调用。
- 在 `j` 函数下方（第 66 行）发现了对 `j` 函数的调用。
- 同时，在同一个 `g` 函数中找到了绑定的控制函数 `onIsSwitch` 的相关代码，根据 `g` 函数的结构和 UI 的界面推断：第 61～63 行代码是对开关图片的事件绑定，而第 64～66 行是对开关下方文字的事件绑定。
- 所以 `onIsSwitch` 就是我们需要的函数。

## 定位开启与关闭函数

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_X3MB4574XH478MH.png)

- 在 `onIsSwitch` 函数中，可以明显看到 `onOff()` 和 `onOpen()` 函数。
- 继续分析这两个函数，但直接搜索后发现，这两个函数并未在 `chunk_1.appservice.js` 中定义，因此需要找到这两个函数所属对象（`o`）的定义位置。
- 向上查看代码，发现了 `o` 的包装位置，并且包装函数 `h` 也位于其下方。

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_FMKA7723YCEMS5A.png)

- 在整个项目中搜索模块 `bluetooth/protocol`，最终在 `appservice.app.js` 中定位到相关代码。

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_ZQHK8PT2JA2WQDV.png)

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_6Y73HDCQHZ9C9EV.png)

- 最终找到开启与关闭各有两个函数，因为有新旧协议的区别，具体使用哪套协议取决于电动牙刷型号，而身边的牙刷为 P 系列，对应 `oldBrushProtocol` 的协议，对应如下函数。

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_FSBMTYDGRYRM4NH.png)

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_UFKKJAYR2TXFN2G.png)

## 分析开关函数发送的蓝牙指令

- 分析上述两个函数，可以在第 5382 行和第 5404 行，找到发送蓝牙指令的调用。

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_ZNXR2JTS52Y2BHK.png)

- 再次搜索 `sendMessage` 函数后，可以确认上文第 5382 行和第 5404 行的字符串就是蓝牙指令。

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_6MSTND865RBEAPN.png)

## 逆向结果小结

- 至此，基本完成了对小程序发送蓝牙指令控制设备开关过程的解析，整个蓝牙交互过程没有认证步骤，蓝牙指令也未进行加密或混淆，下一步通过实际蓝牙流量验证逆向结果。

## AI 辅助加速逆向进度（以下内容由 AI 生成）

- 通过上文对开启电动牙刷的蓝牙控制过程的分析，发现该小程序中的蓝牙通信指令构造规则其实较为简单，未采用加密或混淆，因此可借助 AI 对整体代码进行辅助分析，定位其他功能的控制函数，以及蓝牙通信指令的构造规则。

```
AA [TYPE] [LEN] [FUNC] [DATA] [CRC]

│   │      │     │      │      └─ CRC校验码 (XOR异或)

│   │      │     │      └──────── 数据字节

│   │      │     └─────────────── 功能码

│   │      └───────────────────── 数据长度

│   └──────────────────────────── 指令类型

└──────────────────────────────── 帧头 (固定)

```

### 指令列表

#### 开关控制

- `onOpen()`：`aa0f010101a4`（类型：`0f`，数据：`01`，响应：`0f`） - 开启按钮。
- `onOff()`：`aa0f010100a5`（类型：`0f`，数据：`00`，响应：`0f`） - 开启按钮。

#### 飞行模式

- `onOpenAirplane()`：`aa07010101ac`（类型：`07`，数据：`01`，响应：`07`） - 旅行模式（开）
- `onOffAirplane()`：`aa07010100ad`（类型：`07`，数据：`00`，响应：`07`） - 旅行模式（关）

#### 震动提醒

- `onOpenVibration()`：`aa0b010101a0`（类型：`0b`，数据：`01`，响应：`0b`） - 换区提醒（开）
- `onOffVibration()`：`aa0b010100a1`（类型：`0b`，数据：`00`，响应：`0b`） - 换区提醒（关）

#### 高频模式

- `onOpenTreble()`：`aa0e010101a5`（类型：`0e`，数据：`01`，响应：`0e`） - 高频模式（开）
- `onOffTreble()`：`aa0e010100a4`（类型：`0e`，数据：`00`，响应：`0e`） - 高频模式（关）

## 蓝牙抓包验证逆向结果

### 确定电动牙刷的蓝牙 MAC 地址

- 首先使用 `nRF Connect` 扫描附近的蓝牙设备，并激活电动牙刷，发现信号最强的设备名称中，带有该品牌的拼音缩写，尝试连接后，电动牙刷的指示灯发生变化，由此确定电动牙刷的蓝牙 MAC 地址。

### 抓取并分析蓝牙通信流量

- 筛选出 ATT 层流量，定位到开关操作对应的蓝牙流量。

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_WVDADKVRH3QSUDT.png)

![image.png](https://bbs.kanxue.com/upload/attach/202607/1023727_SMZAFEMR762YPG8.png)

## 总结蓝牙通信协议并编写控制脚本

- 这里直接根据上文分析出的蓝牙指令规则，编写一个简单的蓝牙交互脚本。

```python
#!/usr/bin/env python3

# -*- coding: utf-8 -*-



import asyncio



from bleak import BleakClient, BleakScanner



WRITE_CHAR = "0000ff02-0000-1000-8000-00805f9b34fb"

NOTIFY_CHAR = "0000ff02-0000-1000-8000-00805f9b34fb"



CMD_OPEN = "aa0f010101a4"

CMD_OFF = "aa0f010100a5"

CMD_AIRPLANE_ON = "aa07010101ac"

CMD_AIRPLANE_OFF = "aa07010100ad"

CMD_VIBRATE_ON = "aa0b010101a0"

CMD_VIBRATE_OFF = "aa0b010100a1"

CMD_TREBLE_ON = "aa0e010101a5"

CMD_TREBLE_OFF = "aa0e010100a4"



class Controller:

    def __init__(self, mac):

        self.mac = mac.upper()

        self.client = None

        self.response = None

        self.waiting = False



    async def connect(self):

        print(f"搜索：{self.mac}")

        devices = await BleakScanner.discover(timeout=10.0)



        target = None

        for dev in devices:

            if dev.address.upper() == self.mac:

                target = dev

                print(f"找到：{dev.name}")

                break



        if not target:

            print("未找到设备")

            return False



        print("连接中……")

        self.client = BleakClient(target.address)

        await self.client.connect()



        if self.client.is_connected:

            print("连接成功")

            await self.client.start_notify(NOTIFY_CHAR, self._on_notify)

            return True



        return False



    def _on_notify(self, sender, data):

        self.response = [f"{byte:02x}" for byte in data]

        self.waiting = False

        print(" ".join(self.response))



    async def send(self, cmd, expected=None):

        if not self.client or not self.client.is_connected:

            return None



        print(cmd)

        self.waiting = True

        self.response = None



        await self.client.write_gatt_char(

            WRITE_CHAR,

            bytes.fromhex(cmd),

        )



        timeout = 0

        while self.waiting and timeout < 30:

            await asyncio.sleep(0.1)

            timeout += 1



        if timeout >= 30:

            return None



        if expected and self.response and len(self.response) > 1:

            return self.response[1] == expected



        return self.response



    async def disconnect(self):

        if self.client and self.client.is_connected:

            await self.client.disconnect()



    async def turn_on(self):

        result = await self.send(CMD_OPEN, "0f")

        print("已打开" if result else "操作失败")



    async def turn_off(self):

        result = await self.send(CMD_OFF, "0f")

        print("已关闭" if result else "操作失败")



    async def airplane(self, enable):

        cmd = CMD_AIRPLANE_ON if enable else CMD_AIRPLANE_OFF

        status = "开启" if enable else "关闭"

        result = await self.send(cmd, "07")

        print(f"旅行模式已{status}" if result else "操作失败")



    async def vibrate(self, enable):

        cmd = CMD_VIBRATE_ON if enable else CMD_VIBRATE_OFF

        status = "开启" if enable else "关闭"

        result = await self.send(cmd, "0b")

        print(f"换区提醒已{status}" if result else "操作失败")



    async def treble(self, enable):

        cmd = CMD_TREBLE_ON if enable else CMD_TREBLE_OFF

        status = "开启" if enable else "关闭"

        result = await self.send(cmd, "0e")

        print(f"高频模式已{status}" if result else "操作失败")



async def menu(ctrl):

    while True:

        print("\n" + "=" * 40)

        print("1. 打开    2. 关闭")

        print("3. 旅行    4. 换区")

        print("5. 高频    q. 退出")

        print("=" * 40)



        choice = input("选择：").strip()



        if choice == "1":

            await ctrl.turn_on()

        elif choice == "2":

            await ctrl.turn_off()

        elif choice == "3":

            await ctrl.airplane(input("1 开 / 0 关：") == "1")

        elif choice == "4":

            await ctrl.vibrate(input("1 开 / 0 关：") == "1")

        elif choice == "5":

            await ctrl.treble(input("1 开 / 0 关：") == "1")

        elif choice.lower() == "q":

            break



        input("按 Enter 继续……")



async def main():

    mac = input("MAC 地址：").strip()

    if not mac:

        return



    ctrl = Controller(mac)

    if await ctrl.connect():

        try:

            await menu(ctrl)

        finally:

            await ctrl.disconnect()



if __name__ == "__main__":

    try:

        asyncio.run(main())

    except KeyboardInterrupt:

        print("\n已中断")
```
