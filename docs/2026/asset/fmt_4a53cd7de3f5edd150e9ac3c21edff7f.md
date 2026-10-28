
```javascript
// Global Values

var Player = {

    m_walkingSpeed : 200,

};



// Cheat status

var cheatStatus = {

    walkingSpeed : 0,

};



//Teleport

var setPositionAddr = Module.findExportByName("libGameLogic.so", "_ZN5Actor11SetPositionERK7Vector3");

var setPosition = new NativeFunction(setPositionAddr, 'void', ['pointer', 'pointer']);

var Vector3 = Memory.alloc(16);



function teleport(thisReference, x, y, z) {

    Memory.writeFloat(Vector3, x);

    Memory.writeFloat(ptr(Vector3).add(4), y);

    Memory.writeFloat(ptr(Vector3).add(8), z);

    setPosition(thisReference, Vector3);

}





// Chat Helper

function chatHelper(msg, thisReference) {

    var token = msg.split(" ");

    if (token[0] === "!wspeed_on") {

        Player.m_walkingSpeed = parseInt(token[1]);

        cheatStatus.walkingSpeed = 1;

        console.log("[CHEAT]: Walking Speed Enabled (" + token[1] + ")");

    }

    if (token[0] === "!wspeed_off") {

        Player.m_walkingSpeed = 200;

        cheatStatus.walkingSpeed = 0;

        console.log("[CHEAT]: Walking Speed Disabled (200)");

    }

    if (token[0] === "!tp") {

        console.log("[CHEAT]: Teleporting to " + token[1] + " " + token[2] + " "+ token[3]);

        teleport(thisReference, parseInt(token[1]), parseInt(token[2]), parseInt(token[3]));

 }

}



//Find "Player::Chat"

var chat = Module.findExportByName("libGameLogic.so", "_ZN6Player4ChatEPKc");

console.log("Player::Chat() at  address: " + chat);



// Add our logger

Interceptor.attach(chat, {

    onEnter: function (args) { // 0 => this; 1 => cont char* (our text)

       var chatMsg = Memory.readCString(args[1]);

       console.log("[Chat]: " + chatMsg);

       chatHelper(chatMsg, args[0]);

    }



});



// Find Player::GetWalkingSpeed()

var walkSpeed = Module.findExportByName("libGameLogic.so", "_ZN6Player15GetWalkingSpeedEv");

console.log("Player::GetWalkingSpeed() at address: " + walkSpeed);



// Check Speed

Interceptor.attach(walkSpeed,

    {

        // Get Player * this location

        onEnter: function (args) {

            //console.log("Player at address: " + args[0]);

            this.walkingSpeedAddr = ptr(args[0]).add(736) // Offset m_walkingSpeed

            //console.log("WalkingSpeed at address: " + this.walkingSpeedAddr);

        },



        // Get the return value

        onLeave: function (retval) {

            if (Memory.readFloat(this.walkingSpeedAddr) != Player.m_walkingSpeed && cheatStatus.walkingSpeed == 0) {

                Memory.writeFloat(this.walkingSpeedAddr, 200);

            }

            if (cheatStatus.walkingSpeed == 1) {

                Memory.writeFloat(this.walkingSpeedAddr, Player.m_walkingSpeed);

            }

        }

    });
```

## 0x04 从天而降的魔法

更新于2018年7月8日：服务端会做魔法值检测，所以我这里失败了。事与愿违，我们只设置了HUD中的值:(

如果你走向太阳的方向（在海上），你会在某处找到一个有奶牛的岛屿，之后会触发一个任务。在不做手脚的情况下，这个岛上的一个NPC会给你一个武器。 这个武器会消耗魔法，我们不喜欢浪费魔法（即使会快速再生）。我们希望魔法值始终处于最大值！

```
mothra@kaiju:~/holydays|⇒  cat Player.class| grep -i mana

    int32_t m_mana;

    float m_manaRegenTimer;

    virtual int32_t GetMana();

    virtual bool UseMana(int32_t);

    void PerformSetMana(int32_t);
```

魔法值与上文的移动速度情况类似，因此我们的操作方式也是相同的。计算`m_mana`的偏移量，然后hook `GetMana`将魔法值覆盖为100：

```
mothra@kaiju:~/holydays|⇒  gdb -p  $(pidof PwnAdventure3-Linux-Shipping) --batch 

\ -ex "set verbose off" -ex "b _ZN6Player15GetWalkingSpeedEv" --ex "c" --ex "print &this->m_mana"

\ -ex "print this" -ex "print (int)\$1-(int)\$2" -ex "quit" 2>/dev/null | awk '/\$3/ {print $3 }'



544
```

Hook `GetMana`：

```
var getMana = Module.findExportByName("libGameLogic.so", "_ZN6Player7GetManaEv");

        console.log("Player::GetMana at address: " + getMana);

        Interceptor.attach(getMana,

        {

            onEnter: function (args) {

                if (cheatStatus.infiniteMana == 1) {

                    m_manaAddr = ptr(args[0]).add(544) // Offset m_mana

                    Memory.writeInt(m_manaAddr, 100);

                }

            }

        }

        );
```

火力全开，魔法值不再减少。

## 0x05 写在最后

使用Frida在这个游戏中作弊只是一个有趣的方式来解释有关这个框架的基本概念。如果你知道更好的方法（或如何优化代码），请随时在Twitter上联系我[@TheXC3LL](https://bbs.kanxue.com/elink@e10K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2W2P5r3x3K6L8r3H3`.)。最终的作弊代码参见在我的GitHub中的[PwnAdventure3-cheat.py](https://bbs.kanxue.com/elink@acfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6j5i4K6u0V1b7K6y4x3e0q4)9J5c8Y4y4F1K9i4m8H3k6i4c8K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8W2m8%4L8V1q4V1N6X3g2F1N6s2g2J5k6e0y4Q4x3X3c8U0K9r3g2S2N6q4)9J5k6i4m8&6)。

如果您发现拼写错误或其他错误，请与我联系：P。

原文链接：8b1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5i4K6u0V1j5K6y4D9L8q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3M7r3!0K6N6s2y4Q4x3V1k6r3M7X3W2V1j5g2)9J5k6q4m8%4L8W2)9J5k6p5q4V1N6X3g2F1N6s2g2J5k6g2)9J5k6o6y4Q4x3V1j5`.

编译：看雪翻译小组 SpearMint

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2018-8-13 19:49

被SpearMint编辑，原因：
