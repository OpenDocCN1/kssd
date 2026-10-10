

# 移除山口山客户端针对私服的一些限制。

> 原文：[`www.pediy.com/kssd/pediy10/85353.html`](https://www.pediy.com/kssd/pediy10/85353.html)

作者：无聊的菜鸟  
时间：2009 年 3 月  
声明：“知道不？其实懒惰才是人类进步的源动力！”  
  
    山口山，大家都知道的，这里就不介绍了。  
    先说下为什么我会写下这些。首先吧，我不会写的很详细，至少不会直接给出 Patch 之后的代码。因为拿来主义总让我觉得变扭；其次，不写出来吧，总觉得憋着荒……  
    山口上自某个补丁之后，加入了一些额外的代码，外在的表现是减少了#132 错误（违规访问内存）。但是，实际效果是私服自己制作的自定义物品（就是不是暴雪开放的装备）的图标全部显示为问号（图片路径为 INV\_MISC\_QestionMark），也无法直接在背包中右击装备，同时无法自己对自己的自定义装备进行附魔，以及左键点击拿起这些物品后图标显示为问号（图片路径同样为 INV\_MISC\_QestionMark）。而这些问题在官方物品上是不存在的。  
    当然，你会说也许私服传递的数据不正确才导致这样的。这句话有道理，在补丁和检验的过程中，不同的私服服务器端传递下来的数据的正确性差异很大。但是这并不是主要的，因为这些自定义装备在那个补丁之前同样没有这些问题，而现在，即使有这这些问题，这些物品依然可以通过拖放的方法将其装备在正确的位置，并且装备在角色身上之后，在角色信息页面中图标显示的也是正确的。所以我们得出的结论是，客户端中有限制代码。  
\--------------------------无辜的分割线---------------------------  
插件部分(Lua)：  
    其实最开始的时候，我是去 google 了解决方法，结果下下来了一个插件 ItemPatch，记得好像是在 zgwow.cn 下的，这个插件里面 Hook 了 GetContainerItemInfo 和 GetActionTexture 两个 API，然后自行处理的这个 API 中出现 INV\_MISC\_QestionMark 的情况，具体往下看注释，顺带扫扫盲：  

代码:

```
\-- Item's Patch For 2.4.2 Core by W.S :P -- 由 W.S 为山口山 2.4.2 版本编写。

WOW\_GetContainerItemInfo = GetContainerItemInfo;  --Hook
function GetContainerItemInfo(index, id)
  local texture, itemCount, locked, quality, readable;  --声明本地变量
  texture, itemCount, locked, quality, readable = WOW\_GetContainerItemInfo(index, id);
  if( texture and string.find(texture,"INV\_Misc\_QuestionMark") ) then
    -- DEFAULT\_CHAT\_FRAME:AddMessage("GetContainerItemInfo Item "..itemid);
    local itemlink = GetContainerItemLink(index, id);
  --调用 API\_GetContainerItemLink 获取这个显示为问号的物品的物品链接。
    local itemid = 0;
    if( itemlink ) then
      \_, \_, itemid = string.find(itemlink, "Hitem:(%d+):");
  --在物品链接字符串中查找物品 ID。实际为"Hitem:"字符串之后的数字。
      texture = string.gsub(texture,"INV\_Misc\_QuestionMark", My\_GetItemTexture(itemid) );
   --调用下一级函数
      -- DEFAULT\_CHAT\_FRAME:AddMessage( itemlink.." - ID: "..itemid.." - Texture: "..texture );
    end
  end
  return texture, itemCount, locked, quality, readable;
end
\--说到这里，其实在后面 IDA 山口山的时候，我们能发现其实有直接实现的 GetItemID 函数，
\--但是这个函数没有被导出为 API……而且嘛，过程其实是一样的……
function My\_GetItemTexture( itemid )
  if MY\_ITEM\_TEXTURE\[ itemid \] ~= nil and MY\_ITEM\_TEXTURE\[ itemid \] ~= "" then
   --枚举下面数组中的 ID 对比，典型的查表法。
    return MY\_ITEM\_TEXTURE\[ itemid \];
  end
  return "INV\_Misc\_QuestionMark";
end

MY\_ITEM\_TEXTURE = {};
MY\_ITEM\_TEXTURE = 
{
\-- \["物品 id"\] = "INV\_XXX\_XXX",
\["1"\] = "INV\_Qiraj\_JewelGlyphed",
\["2"\] = "INV\_Qiraj\_JewelGlyphed",
......中间省略 XX 千字节.....
\["6051190"\] = "INV\_Scroll\_04",
\["1401019"\] = "INV\_Trinket\_HonorHold",

}

```

     这个插件代码不多，但是由于是查表法，最后插件体积高达 604KB……  
  
\------------------------------------熟悉的分割线--------------------------------  
改进插件：  
    大家都看见了，枚举起来很麻烦，每次自己做了新物品都要修改这个插件的数组，不然就是一个大大的问号。作为用户而言，也很麻烦，时不时地要来更新一次。而且更大的缺点是，这个玩意不通用的，把它用在其它私服上之后，你的 1 号物品是不是和他的 1 号物品一样呢？这个问题很有研究意义啊。  
    在我对山口山的 API 手册研究了之后（google 到的），我觉得其实可以不用枚举法。再 Hook 了 GetActionTexture、GetContainerItemInfo、GetSpellTexture、GetInventoryItemTexture 之后，界面上明显的地方已经没有问号了，不过自定义物品的使用、自身附魔以及拿在鼠标上依然是问号的问题并没有得到解决。我还是举个修改之后的例子：  
    （什么？你想要全部的？这个么，嘛嘛。不要想了。我故意的……）  
  
代码:

```
function GetContainerItemInfo(index, id)  --我们依然以这个函数为例
  local texture, itemCount, locked, quality, readable = WOW\_GetContainerItemInfo(index, id);

  if( texture and string.find(texture,"INV\_Misc\_QuestionMark") ) then
    local itemlink = GetContainerItemLink(index, id);
    local itemid = 0;
    if( itemlink ) then --如果获得的物品链接不是空串
      \_, \_, itemid = string.find(itemlink, "Hitem:(%d+):"); --获取 itemID
      \_, \_, \_, \_, \_, \_, \_, \_, \_\_, texture = GetItemInfo(itemid);  --使用 API\_GetItemInfo 来获取路径
    end
  end
  return texture, itemCount, locked, quality, readable;  --返回需要的东西
end

```

    这里我们就没有使用枚举，因为想下也会知道，自定义物品在 NPC 窗口中出售，在身上装备了之后，他的图标是可以正确显示的，所以 API 中肯定有能返回正确图标的函数。  
  
\--------------------------------好熟悉的分割线----------------------------------------  
山口山部分：  
    下面我们该到主程序的修改部分了，因为插件虽然好，但是有些东西却不能用插件来实现。例如：当你使用山口山登录英国的私服(NaxpServer.com)，看见一片的服务器想选择一个，然后山口山告诉你：“你正在使用不同语言版本的山口山服务器”。其实这个检测也是在插件中实现的，调用的 API 是 IsInvalidLocale。不过这个函数你却无法用插件 Hook。为什么呢？人家是 BLZ 自己写的插件，加载的比你早，在你的插件被读取编译之前，这个检测就已经过了。而且 BLZ 的插件你还不能修改，改了他就不给你启动山口山。  
    然后这里需要介绍下山口山的 API 导出表：  
    用 OD 加载山口山之后，打开内存视图，找到.rdata 区段，然后再这个区段中搜索你要的 API 的名字，比如 IsInvalidLocale，找到：  
代码:

```
0093D370  49 73 49 6E 76 61 6C 69 64 4C 6F 63 61 6C 65 00  IsInvalidLocale.

```

    然后再内存视图中找到.data 区段，在其中搜索这个字符串指针 0093d370，找到：  
代码:

```
00FC1D68  70 D3 93 00                                      p.@

```

    在把视图转为长型 ASCII 数据地址，如下：  
代码:

```
00FC1D40  0093D3F0  鹩?    ASCII "SortRealms"
00FC1D44  00476D70  pmG.  WoW.00476D70
00FC1D48  0093D3DC  苡?    ASCII "GetSelectedCategory"
00FC1D4C  00476E40  @nG.  WoW.00476E40
00FC1D50  0093D3C0  烙?    ASCII "RealmListDialogCancelled"
00FC1D54  00475E10  ^G.  WoW.00475E10
00FC1D58  0093D39C  ?    ASCII "IsInvalidTournamentRealmCategory"
00FC1D5C  00476B10  kG.  WoW.00476B10
00FC1D60  0093D380  .   ASCII "IsTournamentRealmCategory"
00FC1D64  00476BC0  G.   WoW.00476BC0
00FC1D68  0093D370  p.   ASCII "IsInvalidLocale"  <<--这里这里
00FC1D6C  00476C40  @lG.  WoW.00476C40                 <<--这个就是 IsInvaliLocale 的实现代码的地址
00FC1D70  00930B40  @?   WoW.00930B40      <<--空字符串指针，这一段表结束的标记
00FC1D74  00000000  ....

```

    所以要找一个 API 的实现也就是这样子找的。  
    然后通过查阅 API 手册，及对可疑函数的确认之后，最后我们把问题锁定在了 UseContainerItem 上。  
    有人说插件 HooK 不行么？这个 API 又不是启动时调用的。话这么说没错，可是这个 API 的类型是 protected，代表这个 API 只能由 BLZ 的插件来调用。。不支持用户来重写这个函数，而且当你接触了这个 API 的实现之后，你更不会这么想。  
    这个 API 一共 9 个分支，通过下断对比自定义物品和标准物品的使用问题，可以得知自定义物品的装备问题是出在第 8 个分支上：  
  
代码:

```
0053BEA6  |> \\8B7D FC       mov     edi, \[local.1\]                   ; case 8
0053BEA9  |.  8B57 08       mov     edx, dword ptr \[edi+8\]
0053BEAC  |.  33C9          xor     ecx, ecx
0053BEAE  |.  51            push    ecx
0053BEAF  |.  51            push    ecx
0053BEB0  |.  51            push    ecx
0053BEB1  |.  894D E8       mov     \[local.6\], ecx
0053BEB4  |.  894D EC       mov     \[local.5\], ecx
0053BEB7  |.  8B42 0C       mov     eax, dword ptr \[edx+C\]
0053BEBA  |.  8D4D E8       lea     ecx, \[local.6\]
0053BEBD  |.  51            push    ecx
0053BEBE  |.  50            push    eax
0053BEBF  |.  B9 60F31A01   mov     ecx, 011AF360
0053BEC4  |.  E8 17FD0800   call    005CBBE0
0053BEC9  |.  8BCF          mov     ecx, edi
0053BECB  |.  8BF0          mov     esi, eax
0053BECD  |.  E8 CE311100   call    0064F0A0      ;!!非零则可以装备
0053BED2  |.  85C0          test    eax, eax
0053BED4  |.  74 50         je      short 0053BF26
0053BED6  |.  85F6          test    esi, esi
0053BED8  |.  74 09         je      short 0053BEE3
0053BEDA  |.  83BE AC010000>cmp     dword ptr \[esi+1AC\], 0
0053BEE1  |.  75 43         jnz     short 0053BF26
0053BEE3  |>  833D 78680A01>cmp     dword ptr \[10A6878\], 0
0053BEEA  |.  75 1B         jnz     short 0053BF07
0053BEEC  |.  8B55 F4       mov     edx, \[local.3\]
0053BEEF  |.  8B82 08010000 mov     eax, dword ptr \[edx+108\]
0053BEF5  |.  8B88 D0000000 mov     ecx, dword ptr \[eax+D0\]
0053BEFB  |.  C1E9 14       shr     ecx, 14
0053BEFE  |.  F6C1 01       test    cl, 1
0053BF01  |.^ 0F84 6AFEFFFF je      0053BD71
0053BF07  |>  8B55 F8       mov     edx, \[local.2\]
0053BF0A  |.  8B43 0C       mov     eax, dword ptr \[ebx+C\]
0053BF0D  |.  8B4B 08       mov     ecx, dword ptr \[ebx+8\]
0053BF10  |.  6A 00         push    0
0053BF12  |.  52            push    edx
0053BF13  |.  50            push    eax
0053BF14  |.  51            push    ecx
0053BF15  |.  8B4D F4       mov     ecx, \[local.3\]
0053BF18  |.  E8 33A30F00   call    00636250
0053BF1D  |.  5F            pop     edi
0053BF1E  |.  5B            pop     ebx
0053BF1F  |.  33C0          xor     eax, eax
0053BF21  |.  5E            pop     esi
0053BF22  |.  8BE5          mov     esp, ebp
0053BF24  |.  5D            pop     ebp
0053BF25  |.  C3            retn

```

    然后我们继续来看 0064F0A0：  
代码:

```
本地调用来自 004AB0C8, 004AB12F, 004AE0A0, 0053BECD, 0058248E, 006313DA, 0064DD14, 00650A14, 00650BD3, 007713A0

0064F0A0  /$  8B41 08       mov     eax, dword ptr \[ecx+8\]
0064F0A3  |.  8B40 0C       mov     eax, dword ptr \[eax+C\]  ;此时 eax=itemID
0064F0A6  |.  8B0D 9C01FD00 mov     ecx, dword ptr \[FD019C\]  ;\[FD019C\]=11h
0064F0AC  |.  3BC1          cmp     eax, ecx
0064F0AE  |.  7C 1B         jl      short 0064F0CB
0064F0B0  |.  3B05 9801FD00 cmp     eax, dword ptr \[FD0198\]  ;\[FD0198\]=AFEDh
0064F0B6  |.  7F 13         jg      short 0064F0CB
0064F0B8  |.  2BC1          sub     eax, ecx
0064F0BA  |.  8B0D AC01FD00 mov     ecx, dword ptr \[FD01AC\]  ;\[00FD01AC\]=06371F00h
0064F0C0  |.  8B0481        mov     eax, dword ptr \[ecx+eax\*4\]
0064F0C3  |.  85C0          test    eax, eax
0064F0C5  |.  74 04         je      short 0064F0CB
0064F0C7  |.  8B40 18       mov     eax, dword ptr \[eax+18\]
0064F0CA  |.  C3            retn
0064F0CB  |>  33C0          xor     eax, eax
0064F0CD  \\.  C3            retn

```

    到这里，根据现有资料，11h(17)表示为客户端数据库中最小的 itemid，而 AFEDh(45037)则是最大的 itemID，这里做出了判断，如果传递近来的物品的 ID 不再客户端所包含的物品 ID 范围内则视为未知物品返回 0，如果在范围内，则查表 1 获得一个非零数据返回，最后经验证，此数据为物品的 equipSlot，即装备位置：其取值可能为如下之一：  
代码:

```
INVTYPE\_2HWEAPON - Two-Hand  双手武器
INVTYPE\_AMMO - Ammo 弹药
INVTYPE\_BAG - Bag      背包
INVTYPE\_BODY - Shirt   衬衣
INVTYPE\_CHEST - Chest  胸部
INVTYPE\_CLOAK - Back   背部披风
INVTYPE\_FEET - Feet     脚
INVTYPE\_FINGER - Finger 手指戒指
INVTYPE\_HAND - Hands  手
INVTYPE\_HEAD - Head   头
INVTYPE\_HOLDABLE - Held In Off-hand  副手物品
INVTYPE\_LEGS - Legs     腿部
INVTYPE\_NECK - Neck    颈部
INVTYPE\_QUIVER - Quiver     箭袋
INVTYPE\_RANGED - Ranged   远程射击武器
INVTYPE\_RANGEDRIGHT - Ranged  魔杖
INVTYPE\_RELIC - Relic               圣物
INVTYPE\_ROBE - Chest              胸 2
INVTYPE\_SHIELD - Off Hand       盾
INVTYPE\_SHOULDER - Shoulder  肩
INVTYPE\_TABARD - Tabard        战袍
INVTYPE\_THROWN - Thrown     投掷
INVTYPE\_TRINKET - Trinket       饰品
INVTYPE\_WAIST - Waist            腰部
INVTYPE\_WEAPON - One-Hand   单手
INVTYPE\_WEAPONMAINHAND - Main Hand  主手武器
INVTYPE\_WEAPONOFFHAND - Off Hand      副手武器
INVTYPE\_WRIST - Wrist            手腕

```

    所以这个函数的名称推测为 GetItemEquipSlotForClientItem。而我们需要的则是把这个函数修改成适合所有物品的。  
    前面插件中我们注意到 GetItemInfo 函数是能获取到正确的物品信息，也就是说，这个函数是没有有效物品检查的，通过分析这个函数，我们注意到 011AF368 的表可以查询到正确的物品信息，所以我们仿写其中的查询方式，即可完美解决这个问题。同时装备附魔的问题也一并解决了，猜测这 2 个限制公用了这个判断函数  
  
\---------------------------------哎，休息休息------------------------  
    然后继续对 UseContainerItem 的分析中，发现鼠标拿起物品之后显示的问号则是发生在第三个分支上：  
代码:

```
0053B702  |> \\F687 CC030000>test    byte ptr \[edi+3CC\], 1
0053B709  |.  75 78         jnz     short 0053B783
0053B70B  |.  8B45 FC       mov     eax, \[local.1\]
0053B70E  |.  8B4E 0C       mov     ecx, dword ptr \[esi+C\]
0053B711  |.  8B56 08       mov     edx, dword ptr \[esi+8\]
0053B714  |.  8B75 E0       mov     esi, \[local.8\]
0053B717  |.  6A 01         push    1
0053B719  |.  6A 00         push    0
0053B71B  |.  6A 01         push    1
0053B71D  |.  50            push    eax
0053B71E  |.  51            push    ecx
0053B71F  |.  52            push    edx
0053B720  |.  56            push    esi
0053B721  |.  53            push    ebx
0053B722  |.  E8 A928F7FF   call    004ADFD0  
0053B727  |.  56            push    esi
0053B728  |.  53            push    ebx
0053B729  |.  E8 128EF6FF   call    004A4540
0053B72E  |.  83C4 28       add     esp, 28
0053B731  |.  5B            pop     ebx
0053B732  |.  5F            pop     edi
0053B733  |.  33C0          xor     eax, eax
0053B735  |.  5E            pop     esi
0053B736  |.  8BE5          mov     esp, ebp
0053B738  |.  5D            pop     ebp
0053B739  |.  C3            retn

```

    对这一段代码的跟入发现限制代码位于 00652290：  
代码:

```
本地调用来自 004DBCAF, 004DE933, 0050CFB4, 0050D091, 0053A58A, 0053B098, 00548062, 00583E0D

00652290  /$  8B41 08       mov     eax, dword ptr \[ecx+8\]
00652293  |.  8B40 0C       mov     eax, dword ptr \[eax+C\]
00652296  |.  8B0D 9C01FD00 mov     ecx, dword ptr \[FD019C\]
0065229C  |.  3BC1          cmp     eax, ecx
0065229E  |.  7C 24         jl      short 006522C4
006522A0  |.  3B05 9801FD00 cmp     eax, dword ptr \[FD0198\]
006522A6  |.  7F 1C         jg      short 006522C4
006522A8  |.  2BC1          sub     eax, ecx
006522AA  |.  8B0D AC01FD00 mov     ecx, dword ptr \[FD01AC\]
006522B0  |.  8B0481        mov     eax, dword ptr \[ecx+eax\*4\]
006522B3  |.  85C0          test    eax, eax
006522B5  |.  74 0D         je      short 006522C4
006522B7  |.  8B40 14       mov     eax, dword ptr \[eax+14\]  ;!!
006522BA  |.  50            push    eax
006522BB  |.  E8 D0FEFFFF   call    00652190
006522C0  |.  83C4 04       add     esp, 4
006522C3  |.  C3            retn
006522C4  |>  33C0          xor     eax, eax
006522C6  |.  50            push    eax
006522C7  |.  E8 C4FEFFFF   call    00652190
006522CC  |.  83C4 04       add     esp, 4
006522CF  \\.  C3            retn

```

    和前面一段代码不同的是这边!!标准的 006522B7 行，获取的成员偏移变了，下面还多了一个对 00652190 函数的调用。通过跟踪对比发现，+14h 的成员是 Icon 的索引号。所以推测这个函数名应该是 GetItemIconForClientItem。了解这个之后，我们同样参照前面的代码只要修改下在最后的成员偏移，这个限制也被成功解除。同时也解决了包裹中物品的显示问题。  
  
    到现在插件中只需要保留对 GetActionTexture 和 GetInventoryItemTexture 这 2 个函数的处理即可。其他 2 个处理可以被删掉。至于这 2 个函数，当然也可以用 patch 来解决，但是由于种种原因，能简单点何必那么复杂？  
  
\----------------Keep going, going, going, and going.---------------------  
    此时这个修改好的山口山主程序在 WLK 客户端上已经没有问题，但是处于偷懒的愿望，我并不想在布下 3.05 的客户端，所以我直接复制到了 3.05 客户端重执行，OK。然后登录的时候我得到了一个提示：“您的账号已经开启了《巫妖王之怒》的权限，但是您所使用的客户端不包括《巫妖王之怒》的内容。如果您想要在这台电脑上进行游戏，就必须安装《巫妖王之怒》客户端。请访问 www.wowchina.com 下载客户端。”  
    无奈的再次开启 OD，然后定位到 GetClientExpansionLevel，然后对第一个 call 指向的函数 004021A0 下断，再登录，很快就能定位相应的判断代码了，这边我就不写了。  
  
收工，好长好长，太膜拜我自己了……