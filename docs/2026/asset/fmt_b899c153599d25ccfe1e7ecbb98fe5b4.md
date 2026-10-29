
```cpp
free(key2);

}

else

{

result = do_xxtea_decrypt(data, data_len, key, ret_length);

}

return result;

}
```

或者直接`pip install xxtea-py`

然后直接`python`，自己写个循环实现批量解密吧

```python
import xxtea
import os
orig_path = ""//初始路径
new_path = ""//存储路径
xxtea_sign = ""
xxtea_key = ""
orig_file = open(orig_path, "rb")
encrypt_bytes = orig_file.read()
orig_file.close()
decrypt_bytes = xxtea.decrypt(encrypt_bytes[len(xxtea_sign):], xxtea_key)
new_file = open(new_path, "wb")
new_file.write(decrypt_bytes)
new_file.close()
```

下面开始分析`lua`，这个最简单，我们想分析子弹打到鱼的逻辑，直接找`src\views\layer\BulletLayer.luac`

直接找子弹打到鱼的函数，很清晰，第一个参数为子弹对象，第二个为鱼列表，这里看到只有`master_id == self_player_id`才会调用

`on_self_bullet_crash_fish`当自己的子弹击中了鱼最后会调用加金币等，否则只调用`on_bullet_crash_fish`显示效果，这里可以也改成`on_self_bullet_crash_fish`就可以加金币了

```lua
function BulletLayer:on_bullet_crash_fish(bullet_obj, t_fish_list)
local scene = self:get_scene()
local master_id = bullet_obj:get_master_id()
local bullet_uid = bullet_obj:get_bullet_uid()
local self_player_id = self:get_self_player_id()
local num = #t_fish_list
if(num >= 2) then
local fish_layer = self:get_fish_layer()
table.sort(t_fish_list, function(uid1, uid2)
local fish_obj1 = fish_layer:get_fish_by_uid(uid1);local fish_obj2 = fish_layer:get_fish_by_uid(uid2)
local zorder1 = fish_obj1:getLocalZOrder();local zorder2 = fish_obj2:getLocalZOrder()
if(zorder1 > zorder2) then return true end
end)
end
local fish_uid = t_fish_list[1]
if(master_id == self_player_id) then
scene:on_self_bullet_crash_fish(master_id, bullet_uid, fish_uid)
--scene:on_self_bullet_crash_fish_test(master_id, bullet_uid, fish_uid)
end
scene:on_bullet_crash_fish(master_id, bullet_uid, t_fish_list)
end
```

在定位一下加钱的函数，`fish_gold`是鱼的钱，`fish_odds`是鱼的剩余 ，这句 `local one_add_gold = math.floor(fish_gold/12)`就是一个鱼加的钱，我们把这个`/12`改为`*12`就可以修改倍数了

```lua
function AnimationLayer:play_catch_fish_earn_money_self(fish_gold, fish_odds)
local earn_money_node = self:reuse_new_earn_money_view_node('zhuanqianle_buyu', self, 0)
local pos = cc.p(sizeVisble.width/2, sizeVisble.height/2)
local one_add_gold = math.floor(fish_gold/12)
local min = self:get_earn_money_one_add_gold_min()
if(one_add_gold < min) then one_add_gold = min end
earn_money_node:setPosition(pos); earn_money_node:set_gain_gold(fish_gold)
earn_money_node:set_one_add_gold(one_add_gold); earn_money_node:set_cur_gold(0)
local delay_time = cc.DelayTime:create(4)
local call_back = cc.CallFunc:create(handler(self, self.call_back_reuse_obj))
local seq = cc.Sequence:create(delay_time, call_back);
earn_money_node:setScale(1)
earn_money_node:runAction(seq);
earn_money_node:play_ani()
end
```

入口代码在`AnimationLayer.lua`里，直接看接收到玩家捕到鱼的函数，分为网或者炸弹，里面还有一网打尽和大转盘之类的特效，这里可以把它全部改成高倍的特效

```lua
function AnimationLayer:on_recv_player_catch_fish(player_id, fish_uid, fish_gold)
local fish_layer = self:get_fish_layer()
local fish_obj = fish_layer:get_fish_by_uid(fish_uid)
if(fish_obj == nil) then return end
local data_center = self:get_data_center()
local fish_id = fish_obj:get_fish_id()
local fish_info = data_center:get_fish_info(fish_id)
if(fish_info == nil) then return end
local fish_type = fish_info.type
if(fish_type == GameDefine.fish_type.wang) then
local other_fish_obj_list = self:get_other_fish_yi_wang_da_jin_list(fish_obj, GameDefine.fish_type.wang)
local num = #other_fish_obj_list
if(num > 0) then
local fish_odds = self:get_catch_fish_odds(fish_obj, other_fish_obj_list)
self:on_player_catch_fish_yi_wang_da_jin(player_id, fish_obj, fish_gold, fish_odds, other_fish_obj_list)
end
self:on_player_catch_fish_drop_fish_gold(player_id, fish_obj, 0, 0)
return
end
if(fish_type == GameDefine.fish_type.bomb) then
play_drop_gold = 0
local other_fish_list = self:get_bomb_fish_effect_fish_list()
local num = #other_fish_list
if(num > 0) then
local fish_odds = self:get_catch_fish_odds(fish_obj, other_fish_list)
self:on_player_catch_fish_bomb(player_id, fish_obj, fish_gold, fish_odds, other_fish_list)
end
self:on_player_catch_fish_drop_fish_gold(player_id, fish_obj, 0, 0)
return
end
local fish_odds = fish_info.mulriple_max;local fish_name = fish_info.name
self:on_player_cath_fish_da_zhuan_pan(player_id, fish_type, fish_gold, fish_name)
self:on_player_catch_fish_drop_fish_gold(player_id, fish_obj, fish_gold, fish_odds)
end
```

`src\models\DataCenter.lua`里存储着鱼和子弹等参数，可以看到子弹最高9级，`odds_min`与`odds_max`应该就代表威力，改这里可以改子弹威力

```
{
room_style= 3,
cannon_id= 109,
odds_min= 7,
odds_max= 10,
level= 3,
res= "pao3_buyu",
bullet= "zd9",
net= "yuwang3_buyu",
time= 200,
sound= 109
}
```

`self.fish_client`存储不同鱼的参数，不同id对应不同鱼，`mulriple_min`与`mulriple_max`存储着金币的倍数最大与最小值，一般鱼都是2-3倍，大章鱼为300倍，改这里可以直接改鱼的倍数

```
{
id= 302,
name= "60倍组合鱼",
packet= "4lian_buyu",
crash_model= 101,
mulriple_min= 60,
mulriple_max= 60,
type= 101,
zoder= 21,
die_sound= "fish14_1",
die_type= 4,
copy_num= 4
},
{
id= 402,
name= "大章鱼",
packet= "yu22_buyu",
crash_model= 16,
mulriple_min= 300,
mulriple_max= 300,
type= 2,
zoder= 30,
bron_sound= "f_wb_3",
die_sound= "fish33_1",
die_type= 5,
copy_num= 4
}
```

其他的功能逻辑获得源码后都很清晰，改倍数改鱼改子弹什么都很简单，改完之后重新用`sign`和`key`加密之后push到相应文件夹下面就实现了破解

# 个人想法

如何实现cocos2d反逆向，我的一些不成熟想法，由浅到深分层如下

1. 修改`xxtea`的`key`和`sign`，就像这个游戏，需要分析`so`才能找到`key`

2. 直接修改`xxtea`算法为其他可逆算法，这样逆向的时候还得逆加密算法

3. 在进一步，修改`luajit`源码改动`lua`虚拟机，比如改变字节码指令顺序，或者改变数据读取顺序等，这样逆向的时候必须分析`lua`虚拟机的改动

4. 把`key`，加密算法，虚拟机改动等的关键函数封装放到其他`cpp`或者其他`so`里，在加密一层，调用的时候在解密，这样需要先动态调试先分析调用逻辑

5. 关键代码加入`ollvm`再编译或者直接`vm`掉，这样需要先去掉`ollvm`混淆或者分析`vm`

## 参考文章

<a href="elink@52cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8s2S2B7M7$3S2#2K9Y4g2Q4x3V1k6H3i4K6u0r3y4K6l9J5z5o6x3&6x3#2)9J5k6h3S2@1L8h3H3`.">cocos2d 3.3 lua 代码加密 luac</a>

<a href="elink@ea6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6N6h3q4F1M7X3g2F1P5s2g2W2i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8H3i4K6u0V1j5%4u0S2N6$3I4Q4x3V1k6D9N6h3q4U0i4K6u0W2K9s2c8E0L8l9`.`.">安卓逆向之Luac解密反编译</a>

目录

- [初步认识cocos2d-x](https://bbs.kanxue.com/#初步认识cocos2d-x)

- [从c++进入lua世界](https://bbs.kanxue.com/#从c++进入lua世界)

- [applicationDidFinishLaunching函数](https://bbs.kanxue.com/#applicationdidfinishlaunching函数)

- `LuaStack::executeScriptFile`

- `LuaStack::luaLoadBuffer`

- [总结](https://bbs.kanxue.com/#总结)

- [实战](https://bbs.kanxue.com/#实战)

- [个人想法](https://bbs.kanxue.com/#个人想法)

- [参考文章](https://bbs.kanxue.com/#参考文章)

# 初步认识cocos2d-x

先clone到本地

`git clone 1a6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3y4G2M7K6u0V1i4K6u0r3j5$3!0U0L8%4x3J5k6q4)9J5k6s2S2Q4x3X3g2Y4K9i4b7`.`.

Cocos2d-x是一个开源的移动2D游戏框架，底层支持各种平台，核心用`c++`封装了各种库，外面给了`lua`和`c++`的接口，所以关键代码可能在`lua`中，很多安卓游戏的逻辑也基本都在`lua`脚本里，盗用官网这张图

![](https://bbs.kanxue.com/upload/attach/202107/877073_8S5R433RFN2RNT7.png)

# 从c++进入lua世界

`lua`虚拟机相关代码在`cocos2d-x\cocos\scripting\lua-bindings\manual`里

`CCLuaEngine.h` lua引擎相关

`CCLuaStack.h` lua栈相关

进入虚拟机

`cocos2d-x\templates\lua-template-default\frameworks\runtime-src\Classes\AppDelegate.cpp\AppDelegate::applicationDidFinishLaunching`

## applicationDidFinishLaunching函数

应用结束加载中进入`lua`虚拟机
