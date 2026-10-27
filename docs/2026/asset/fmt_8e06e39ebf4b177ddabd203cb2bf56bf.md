
# [原创]cxk壳的流程实现和复盘

在开始正文之前, 说一些题外话。

1. 这是本人在论坛注册后的第二次发帖,如果有格式错误, 发布的版块错误, 请各位明示指出。管理员也可以删帖。
2. 因为本人水平有限, 某些地方写的可能是错的, 也请各位大佬指正, 不胜感激。
3. 以下文章提到的壳子是本人编写的第二个壳子。编写过程中, 面临毕业找工地搬砖，以及学习新知识，等一系列事情。所以做的并不好。
4. 这个壳子并没有起名字, 某些大佬称它为 cxk壳 。壳子中代码 回滚加密 和字符串加密 是我一个同学实现的, 这篇文章只写我自己实现和参与实现的部分。
5. 这个帖子是本人对写壳过程的一次复盘。会写一些经验教训, 错误总结, 整体实现思路,和一些没有实现但是觉得很有用的点。
6. 为什么要发这个帖子？
   1) 本人在学习的时候, 大量阅读了论坛的优秀文章, 受益良多,饮水思源, 自己也有一些经验也想分享出来,希望跟我一样的小白可以受益。
7. 各位看这个帖子能得到什么？
   1) 在写壳过程中我自己的一些思路(老师指点和个人总结)。
   2) 两个可用反调试。
   3) 写壳过程中,所有我走过的弯路, 踩过的坑, 大家可以引以为鉴。
   4) 写壳过程中, 一些虽然没有实现, 但是我觉得很棒的思路, 大佬们以后或者可以用以下。
   5) 加壳源码以及shellcode。

## 正文开始：

加壳语言：c++

目标软件文件格式：PE

加壳平台：win32 / win64 （最开始写的时候 只希望在w7 32位平台可以运行）

## 明确加壳宗旨：

软件加壳是为了更好的保护软件, 目前世面上各种成熟的商业壳很多。从技术上讲, 攻防无绝对, 很多很厉害的商业壳被更厉害的大佬脱的干干净净。

目前强度很高, 最富盛名的壳子是VMP。

当然VMP壳也有对应的脱壳方法, 有人做过工作量计算, 如果一个大佬一个人全职脱壳(体量不大的情况下), 大概半年能完全脱掉。

厉害的壳子并不是完全杜绝破解者破解, 因为大佬无穷多, 总是有对应的解决办法。厉害的壳子是像VMP壳一样, 拉长破解者破解的时间。大概是那种,我知道各位很厉害, 我也知道各位一定能破解的了, 我只是希望能多托各位几天。

在这种理念前提下, 写壳的时候, 更多就是计算攻防双方的时间成本。

以下是一个表格, 以攻防双方水平差不多为大前提.

举例不同手法下, 防守方实现防守手段时间，和攻击方成功破解所花费时间。`n`表示 > 1的未知数。

![](https://bbs.kanxue.com/upload/attach/202001/93_KVHQFUNT3SPC6VX.jpg)

由上可见, 在写壳的过程中：

- 最理想的情况是：设计出一套 高性价比手法的加壳方案（实现起来简单, 破解起来麻烦）。
- 退而求其次的是：设计一些 中性价比的方案, 争取攻防双方消耗同等时间。
- 极力要避免的是：低性价比的手法, 防守方实现起来无比麻烦, 攻击方三两下干掉。

## 确定写壳计划：

在明确加壳宗旨前的写壳计划:

![](https://bbs.kanxue.com/upload/attach/202001/93_GWZZTG9GTAV6YYX.jpg)

在经过老师指点,明确加壳宗旨后,修改的工作计划

1) 对原计划的更改

![](https://bbs.kanxue.com/upload/attach/202001/93_ENGS3FASXUWSKHD.jpg)

这里本应该放弃 编号2 编号3 两个低性价比的方案, 但是在调整方案前, 编号2第性价比的方案就已经写完。所以只放弃了编号3性价比低的方案。

2) 新增的工作计划

![](https://bbs.kanxue.com/upload/attach/202001/93_K9JS8JEP2MRKT8W.jpg)

## 具体实现手法：

### 导入表相关：

导入表这里一共做了3个操作:

1. 目标文件原导入表被清0, 运行时在堆区申请空间, 填写API地址, 把堆区地址写入IAT, 完成IAT混淆。
2. 堆区填写IAT地址的时候, 为防止破解直接跳转到关键点, 写了6段垃圾代码, 随机选用填充。
3. 在填写API时候, 并未直接出现函数名的字符串对比，而是对函数名算了个hash, 通过Hash对比填写API地址

把上面的操作一步一步展开说：

1) 原文件导入表清空填0 代码如下

（受限于篇幅 这里贴了部分代码 完整代码看附件）

```
//这里清空原文件导入表的操作 封装成了一个函数

bool Pe_Imp::clear_imp()

{

    if (0 == this->m_imp_lst.size())

    {

        this->get_imp_data();

    }



    DWORD dw_iat_foa = 0;

    DWORD dw_int_foa = 0;

    DWORD dw_name_foa = 0;

    DWORD dw_byname_foa = 0;



    bool b_ret = false;

    char* p_name = 0;



    St_Pe_Int* p_iat = 0;

    St_Pe_Int* p_int = 0;

    St_Int_Name* p_int_name = 0;



    std::list<St_Imp_Data>::iterator it_imp_begin = m_imp_lst.begin();

    std::list<St_Imp_Data>::iterator it_imp_cur = m_imp_lst.begin();

    std::list<St_Imp_Data>::iterator it_imp_end = m_imp_lst.end();



    //清理iat byname

    for (; it_imp_cur != it_imp_end; it_imp_cur++)

    {

        //清空dll名

        dw_name_foa = it_imp_cur->dw_name_foa;

        p_name = this->m_p_data_buf + dw_name_foa;

        memset(p_name, 0, strlen(p_name));





        //获取 int iat表的 foa

        dw_int_foa = it_imp_cur->dw_int_foa;

        dw_iat_foa = it_imp_cur->dw_iat_foa;



        std::list<St_Int_Data>::iterator it_int_cur = it_imp_cur->int_lst.begin();

        std::list<St_Int_Data>::iterator it_int_end = it_imp_cur->int_lst.end();



        for (; it_int_cur != it_int_end; it_int_cur++)

        {

            p_int = (St_Pe_Int*)(this->m_p_data_buf + dw_int_foa);

            p_iat = (St_Pe_Int*)(this->m_p_data_buf + dw_iat_foa);



            //int清0  iat清0

            memset(p_int, 0, sizeof(St_Pe_Int));

            memset(p_iat, 0, sizeof(St_Pe_Int));

            dw_iat_foa += sizeof(St_Pe_Int);

            dw_int_foa += sizeof(St_Pe_Int);



            //如果是序号 就没啥事了

            if (true == it_int_cur->is_ords)

            {

                continue;

            }

                

            //定位到名称表 全部填0

            p_int_name = (St_Int_Name*)(this->m_p_data_buf 

                                      + it_int_cur->dw_byname_foa);

            p_int_name->Hint = 0;

            memset(&(p_int_name->Name), 0, it_int_cur->n_name_len);

        }

    }



    St_Pe_Imp* p_cur_imp = 0;

    p_cur_imp = this->m_p_imp_table;

    for (int n_index = 0; n_index < this->m_n_imp_count; n_index++)

    {

        p_cur_imp->OriginalFirstThunk = 0;  //int填0

        p_cur_imp++;

    }



    return 0;

}
```

2）垃圾代码填充 申请堆区空间 完成IAT混淆 代码如下

（受限于篇幅 这里贴了部分代码 完整代码看附件）
