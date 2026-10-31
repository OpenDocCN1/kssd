
# [原创]通用键盘鼠标模拟(包括USB和PS2)

通过直接调用`Kbdclass`的回调函数`KeyboardClassServiceCallback`直接给上层发送键盘驱动。这个方法网上已经公开，参考Hook `KeyboardClassServiceCallback`实现键盘 Logger，其他的还有很多，可以到网上去查。

简单说一下没有公开的部分，就是按下和松开的模拟，已经扩展键的模拟。

模拟主要是构造`KEYBOARD_INPUT_DATA`结构，按下和松开的`Flags`分别对应`KEY_MAKE`、`KEY_BREAK`，然后调用`KeyboardClassServiceCallback`。这里直接用的`sudami`的代码，在此谢过，懒得改了。代码如下：

```
case IOCTL_KEY_DOWN :
  {
   if (ioBuf)
   {
    lKeyCode = *(ULONG*)ioBuf;
    dprintf("[KeyMouse] KeymouseDispatchDeviceControl IOCTL_KEY_DOWN = 0x%x\n", lKeyCode);
    dwSize = sizeof(KEYBOARD_INPUT_DATA);
    __asm {
     push eax
      mov kid.UnitId,0  ; 构造 KEYBOARD_INPUT_DATA
      mov eax,lKeyCode
      mov kid.MakeCode,ax
      mov kid.Flags,KEY_MAKE ;模拟按下
      mov kid.Reserved,0
      mov kid.ExtraInformation,0
      lea eax,dwRet
      push eax
      lea eax,kid
      add eax,dwSize
      push eax
      lea eax,kid
      push eax
      push g_kbDeviceObject
      call orig_KeyboardClassServiceCallback ;利用 KeyboardClassServiceCallback 模拟按键
      pop eax
    }
    status = STATUS_SUCCESS;
   }
   break;
  }
case IOCTL_KEY_UP:
  {
   if (ioBuf)
   {
    lKeyCode = *(ULONG*)ioBuf;
    dprintf("[KeyMouse] KeymouseDispatchDeviceControl IOCTL_KEY_UP = 0x%x\n", lKeyCode);
    dwSize = sizeof(KEYBOARD_INPUT_DATA);
    __asm {
     push eax
      mov kid.UnitId,0  ; 构造 KEYBOARD_INPUT_DATA
      mov eax,lKeyCode
      mov kid.MakeCode,ax
      mov kid.Flags,KEY_BREAK ;模拟松开
      mov kid.Reserved,0
      mov kid.ExtraInformation,0
      lea eax,dwRet
      push eax
      lea eax,kid
      add eax,dwSize
      push eax
      lea eax,kid
      push eax
      push g_kbDeviceObject
      call orig_KeyboardClassServiceCallback ;利用 KeyboardClassServiceCallback 模拟按键
      pop eax
    }
    status = STATUS_SUCCESS;
   }
   break;
  }
```

扩展键的区别是按下和松开的`Flags`分别对应`KEY_E0`、`KEY_E1`。其他和上面的一样，这里就不贴代码出来了。主要说一下扩展键有哪几个：（前面是`MakeCode`，后面代表按钮）

`0x1D-RIGHT CONTROL` `0x38-RIGHT ALT` `0x48-↑ 键` `0x50-↓ 键` `0x4b-← 键` `0x4d-→ 键` `0x5B-LEFT WIN` `0x5C-RIGHT WIN`

## 鼠标模拟

重点说一下鼠标的模拟，原理和键盘的一样。查找驱动`mouclass.sys`中的`MouseClassServiceCallback`函数，然后获取`\Device\PointerClass0`设备对象指针，构造`MOUSE_INPUT_DATA`结构，然后调用`MouseClassServiceCallback`。难点就在与构造`MOUSE_INPUT_DATA`结构上面。

```
typedef struct _MOUSE_INPUT_DATA {
    USHORT UnitId;
    USHORT Flags;
    union {
        ULONG Buttons;
        struct  {
            USHORT  ButtonFlags;
            USHORT  ButtonData;
        };
    };
    ULONG RawButtons;
    LONG LastX;
    LONG LastY;
    ULONG ExtraInformation;
} MOUSE_INPUT_DATA, *PMOUSE_INPUT_DATA;
```

通过调试操作系统调用`MouseClassServiceCallback`的参数，主要的标示有3个。

`Flags`标志是标示鼠标的坐标属性(即相对坐标、绝对坐标等)

`ButtonFlags`标志是左右中键按下和松开的标志

`LastX`是鼠标X坐标，与`Flags`标志有关

`LastY`是鼠标Y坐标，与`Flags`标志有关

其他几项可以填0。

具体模拟代码如下：

```
case IOCTL_MOUSE_LEFT_BUTTON_DOWN:
  {
   MouseFlags = MOUSE_LEFT_BUTTON_DOWN;
   goto __MouseCallBack;
  }
case IOCTL_MOUSE_LEFT_BUTTON_UP:
  {
   MouseFlags = MOUSE_LEFT_BUTTON_UP;
   goto __MouseCallBack;
  }
case IOCTL_MOUSE_RIGHT_BUTTON_DOWN:
  {
   MouseFlags = MOUSE_RIGHT_BUTTON_DOWN;
   goto __MouseCallBack;
  }
case IOCTL_MOUSE_RIGHT_BUTTON_UP:
  {
   MouseFlags = MOUSE_RIGHT_BUTTON_UP;
   goto __MouseCallBack;
  }
case IOCTL_MOUSE_MIDDLE_BUTTON_DOWN:
  {
   MouseFlags = MOUSE_MIDDLE_BUTTON_DOWN;
   goto __MouseCallBack;
  }
case IOCTL_MOUSE_MIDDLE_BUTTON_UP:
  {
   MouseFlags = MOUSE_MIDDLE_BUTTON_UP;
__MouseCallBack:
   mid.UnitId = 0;
   mid.Flags = MOUSE_MOVE_RELATIVE;
   mid.Buttons = 0;
   mid.ButtonFlags = MouseFlags;
   mid.RawButtons = 0;
   mid.LastX = *((ULONG*)ioBuf);
   mid.LastY = *((ULONG*)ioBuf+1);
   mid.ExtraInformation = 0;
   InputDataStart = &mid;
   InputDataEnd = InputDataStart+1;
   orig_MouseClassServiceCallback(
   g_mouDeviceObject,
   InputDataStart,
   InputDataEnd,
   &InputDataConsumed
   );
   status = STATUS_SUCCESS;
   break;
  }
case IOCTL_MOUSE_MOVE_RELATIVE:
  {
   mid.Flags = MOUSE_MOVE_RELATIVE; //相对坐标
   goto __MouseMoveCallBack;
  }
case IOCTL_MOUSE_MOVE_ABSOLUTE:
  {
   mid.Flags = MOUSE_MOVE_ABSOLUTE; //绝对坐标
   goto __MouseMoveCallBack;
  }
case IOCTL_MOUSE_VIRTUAL_DESKTOP:
  {
   mid.Flags = MOUSE_VIRTUAL_DESKTOP; //虚拟桌面
__MouseMoveCallBack:
   mid.UnitId = 1;
   mid.Buttons = 0;
   mid.RawButtons = 0;
   mid.LastX = *((ULONG*)ioBuf);
   mid.LastY = *((ULONG*)ioBuf+1);
   mid.ExtraInformation = 0;
   InputDataStart = &mid;
   InputDataEnd = InputDataStart+1;
   orig_MouseClassServiceCallback(
   g_mouDeviceObject,
   InputDataStart,
   InputDataEnd,
   &InputDataConsumed
   );
   status = STATUS_SUCCESS;
   break;
  }
```

驱动在`windows XP SP2`上测试通过。

本文来自CSDN博客，转载请标明出处：99fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3L8h3g2J5k6$3g2J5L8s2W2Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3U0l9H3z5g2)9J5c8U0p5I4i4K6u0r3x3e0W2Q4x3V1j5@1z5o6x3^5y4U0f1#2i4K6u0W2j5i4y4H3P5l9`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [KeyMouse.zip](javascript:void(0);)

  （20.41kb，1359次下载）
