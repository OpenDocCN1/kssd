
```cpp
sprintf((char*)outx, "a= %d  renwu:0x%0.8x", YIJiPianYi, *((int*)RenWuJiZhi));

//OutputDebugStringA((char*)outx);

//调试输出用===================================

int ErJiPianYi = *((int*)(YIJiPianYi + 0xB0)); //二级偏移
int SanJiPianYi = *((int*)(YIJiPianYi + 0xB4)); //三级偏移
int GuaiWuShuLiang = (SanJiPianYi - ErJiPianYi) / 4; //怪物数量

//调试输出用===================================

sprintf(outx, "怪物数量:%d", GuaiWuShuLiang);

//OutputDebugStringA((char*)outx);

//调试输出用===================================

if (GuaiWuShuLiang > 0)
{
    Is_XiGuai = FALSE;

    int DD = 0; //偏移递增用
    for (int X = 0; X < GuaiWuShuLiang; X++)
    {
        int GuaiWuJiZhi = *((int*)(ErJiPianYi + DD)); //怪物基址
        DD = DD + 4; //递增4

        if (GuaiWuJiZhi == NULL)
        {
            break;
        }

        sprintf(outx, "怪物基址:%d", GuaiWuJiZhi);
        //OutputDebugStringA((char*)outx);

        int GuaiWuLeiXing = *((int*)(GuaiWuJiZhi + 0x8c)); //怪物类型【怪物基址的一级偏移】
        sprintf(outx, "怪物类型:%d", GuaiWuLeiXing);
        //OutputDebugStringA((char*)outx);

        int GuaiWuZhenYing = *((int*)(GuaiWuJiZhi + 0x74c)); //怪物阵营【怪物基址的一级偏移】
        sprintf(outx, "怪物阵营:%d", GuaiWuZhenYing);
        //OutputDebugStringA((char*)outx);

        //GuaiWuLeiXing==289 && GuaiWuZhenYing==200物品
        //(GuaiWuLeiXing==545 && GuaiWuZhenYing==100)  建筑物
        //(GuaiWuLeiXing==1057 && GuaiWuZhenYing==200)  没用的石头装饰 会卡人

        if (GuaiWuLeiXing == 529 || /*(GuaiWuLeiXing==289 && GuaiWuZhenYing==200) || */ (GuaiWuLeiXing == 545 && GuaiWuZhenYing == 100) || (GuaiWuLeiXing == 129 && GuaiWuZhenYing == 0) || (GuaiWuLeiXing == 100 && GuaiWuZhenYing == 529) || (GuaiWuLeiXing == 33 && GuaiWuZhenYing == 100) || (GuaiWuLeiXing == 273 && GuaiWuZhenYing == 100))
        {
            test1++;

            /*                                if(test1>3)//只吸3个怪物
            {
                Is_XiGuai=TRUE;
                return ;
            }*/

            DWORD RENWU_X = HuoQuRenWuX();
            DWORD RENWU_Y = HuoQuRenWuY();

            __asm
            {
                mov ecx, GuaiWuJiZhi
                mov edx, dword ptr ds:[ecx]
                push 0x0
                push RENWU_Y
                push RENWU_X
                call dword ptr ds:[edx+0x9C]
            }
        }
    }
}

Is_XiGuai = TRUE;

// 离开临界区
//Sleep(10);

}

extern bool Is_XiWU;

void XI_WU()
{// 进入临界区

    if (Is_XiWU == FALSE)
    {
        return ;
    }

    int g;
    int test1 = 0;

    if ((DWORD*)RenWuJiZhi == NULL)
    {
        return ; //人物基址等于空则返回
    }

    int YIJiPianYi = *(int*)(*((int*)RenWuJiZhi) + 0xB4); //一级偏移

    if (YIJiPianYi == NULL)
    {
        return; //一级偏移等于空则返回
    }

/*
    //调试输出用===================================
    char outx[256] = {0};
    sprintf( (char*)outx, "a= %d  renwu:0x%0.8x", YIJiPianYi, *((int*)RenWuJiZhi));
    //OutputDebugStringA((char*)outx);
*/
    //调试输出用===================================
    int ErJiPianYi = *((int*)(YIJiPianYi + 0xB0)); //二级偏移
    int SanJiPianYi = *((int*)(YIJiPianYi + 0xB4)); //三级偏移
    int GuaiWuShuLiang = (SanJiPianYi - ErJiPianYi) / 4; //怪物数量

    //调试输出用===================================
/*
    sprintf(outx, "怪物数量:%d", GuaiWuShuLiang);
    //OutputDebugStringA((char*)outx);
*/
    //调试输出用===================================

    if (GuaiWuShuLiang > 0)
    {
        Is_XiWU = FALSE;
        int DD = 0; //偏移递增用

        for (int X = 0; X < GuaiWuShuLiang; X++)
        {
            int GuaiWuJiZhi = *((int*)(ErJiPianYi + DD)); //怪物基址
            DD = DD + 4; //递增4

            if (GuaiWuJiZhi == NULL)
            {
                break;
            }

    /*        sprintf(outx, "怪物基址:%d", GuaiWuJiZhi);
            //OutputDebugStringA((char*)outx);*/

            int GuaiWuLeiXing = *((int*)(GuaiWuJiZhi + 0x8c)); //怪物类型【怪物基址的一级偏移】
    /*                sprintf(outx, "怪物类型:%d", GuaiWuLeiXing);
            //OutputDebugStringA((char*)outx);*/

            int GuaiWuZhenYing = *((int*)(GuaiWuJiZhi + 0x74c)); //怪物阵营【怪物基址的一级偏移】
/*
            sprintf(outx, "怪物阵营:%d", GuaiWuZhenYing);
            //OutputDebugStringA((char*)outx);*/

            //GuaiWuLeiXing==289 && GuaiWuZhenYing==200物品
            //(GuaiWuLeiXing==545 && GuaiWuZhenYing==100)  建筑物
            //(GuaiWuLeiXing==1057 && GuaiWuZhenYing==200)  没用的石头装饰 会卡人

            if ((GuaiWuLeiXing == 289/* && GuaiWuZhenYing==200*/) )
            {
                test1++;

                DWORD RENWU_X = HuoQuRenWuX();
                DWORD RENWU_Y = HuoQuRenWuY();

                __asm
                {
                    mov ecx, GuaiWuJiZhi
                    mov edx, dword ptr ds:[ecx]
                    push 0x0
                    push RENWU_Y
                    push RENWU_X
                    call dword ptr ds:[edx+0x9C]
                }
            }
        }
    }

    Is_XiWU = TRUE;

    // 离开临界区
    //Sleep(10);

}

void XI_REN()
{
    int g;
    int test1 = 0;

    if ((DWORD*)RenWuJiZhi == NULL)
    {
        return ; //人物基址等于空则返回
    }

    DWORD X = Game_dx.DATA2->RENWU_XY.X;
    DWORD Y = Game_dx.DATA2->RENWU_XY.Y;
    DWORD RenwuJizhi_Er = *((DWORD*)RenWuJiZhi);
```
