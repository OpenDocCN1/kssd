
```
015F:0040760E  50                       PUSH      EAX   

015F:0040760F  FF7510                  PUSH      DWORD PTR [EBP+10]   

015F:00407612  E8C0FBFFFF              CALL      004071D7   

015F:00407617  83C410                  ADD       ESP,10   

015F:0040761A  6A02                    PUSH      02   

015F:0040761C  58                      POP       EAX   

015F:0040761D  5E                      POP       ESI   

015F:0040761E  C9                      LEAVE   

015F:0040761F  C3                      RET   
```

............................

如果要使`EAX=2`，从上面可以看出，

将`00407562 6A01                 PUSH      01`
改`00407562 6A02                 PUSH      02`

就可以注册。

整里一下，用`Ultraedt`打开`Dynamic Desktop.exe`

找到`6A 01 58 5F 38 85`
改成`6A 02 58 5F 38 85`

保存修改文件，重新运行。

噢！已经注册了。

嗨！只是看不明白注册码、每次注册码都不相同，看来我太暴力了。

### 第二种破解方法：

1、
找到`004075B1 756A               JNZ       0040761D`
改成`004075B1 746A               JZ        0040761D`

2、
找到`004075DD 753E               JNZ       0040761D`
改成`004075DD 743E               JZ        0040761D`

3、
找到`004075F6 7525               JNZ       0040761D`
改成`004075F6 7425               JZ        0040761D`

同样用`Ultraedt`打开`Dynamic Desktop.exe`

1、
找到`75 6A 80 BD 05 FC FF`
改成`74 6A 80 BD 05 FC FF`

2、
找到`75 3E 8B 71 01`
改成`74 3E 8B 71 01`

3、
找到`75 25 8D 85 05 FC FF FF`
改成`74 25 8D 85 05 FC FF FF`

注册信息在注册表中

`[HKEY_CURRENT_USER\Software\Dancing Bits\Dynamic Desktop]`

`"szRegistered"="1234567890"   4dd466d0e1e0cc123afbe504da2462ca`
