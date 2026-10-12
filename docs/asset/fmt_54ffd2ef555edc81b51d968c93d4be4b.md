
```csharp
while (R < 19 && D < 19 && map[D, R] == side + 1)
{
    count++;
    R++;
    D++;
}
if (count >= num)
{
    x = i;
    y = j;
    result = true;
    goto End;
}
L = j - 1; R = j+1;
D = i + 1; U = i-1;
count = 0;
//左斜
while (D < 19 && L > 0 && map[D, L] == side + 1)
{
    count++;
    D++;
    L--;
}
count++;
while (U > 0 && R < 19 && map[U, R] == side + 1)
{
    count++;
    U--;
    R++;
}
if (count >= num)
{
    x = i;
    y = j;
    result = true;
    goto End;
}
// End label and closing braces from previous context (implied)
End: return result;
```

### 断四函数

```csharp
private bool killFour(int side, ref int x, ref int y)
{
    bool result = false;
    for (int i = 0; i < 19; i++)
    {
        for (int j = 0; j < 19; j++)
        {
            if (map[i, j] == 0)
            {
                int tempI = i;
                int tempJ = j;
                //横向判断
                int count = 0;
                while ((j < 19) && j >= 0 && Map[i, j] == side + 1)
                {
                    count++;
                    j++;
                }
                j = tempJ - 1;
                while (j < 19 && j >= 0 && Map[i, j] == side + 1)
                {
                    count++;
                    j--;
                }
                if (count >= 4)
                {
                    result = true;
                    x = i;
                    y = j;
                    goto end;
                }
                j = tempJ;
                i = tempI;
                //竖向判断
                count = 0;
                while (i < 19 && i >= 0 && Map[i, j] == side + 1)
                {
                    count++;
                    i++;
                }
                i = tempI - 1;
                while (i < 19 && i >= 0 && Map[i, j] == side + 1)
                {
                    count++;
                    i--;
                }
                if (count >= 4)
                {
                    result = true;
                    x = i;
                    y = j;
                    goto end;
                }
                //右
                count = 0;
                i = tempI;
                while ((i < 19 && i >= 0) && (j < 19 && j >= 0) && Map[i, j] == side + 1)
                {
                    count++;
                    i++;
                    j++;
                }
                i = tempI - 1;
                j = tempJ - 1;
                while ((i < 19 && i >= 0) && (j < 19 && j >= 0) && Map[i, j] == side + 1)
                {
                    count++;
                    i--;
                    j--;
                }
                if (count >= 4)
                {
                    result = true;
                    x = i;
                    y = j;
                    goto end;
                }
                //左
                i = tempI;
                j = tempJ;
                count = 0;
                while ((i < 19 && i >= 0) && (j < 19 && j >= 0) && Map[i, j] == side + 1)
                {
                    count++;
                    i--;
                    j++;
                }
                i = tempI + 1;
                j = tempJ - 1;
                while ((i < 19 && i >= 0) && (j < 19 && j >= 0) && Map[i, j] == side + 1)
                {
                    count++;
                    i++;
                    j--;
                }
                if (count >= 4)
                {
                    result = true;
                    x = i;
                    y = j;
                    goto end;
                }
            }
        }
    }
    end: return result;
}
```

### Ai 函数

```csharp
public void AI(ref int X, ref int Y)
{
    int x=0, y=0;
    int emery = (Turn + 1) % 2;//敌方
    if (count(Turn) == 0)
    {
        if (count(emery) == 0)
        {
            //如果敌人还没下，就随便下一步
            Random rom = new Random();
            x = (int)(rom.Next(7, 15));
            y = (int)(rom.Next(7, 15));
        }
        else
        {
            //如果敌人已经下一子，在敌人权值最大的空行下
            getPoint(ref x, ref y, emery);
        }
    }
    else
    {
        if (SearchWin(turn, ref x, ref y)==true)
        {
        }
        else
        {
            if(SearchWin(emery, ref x, ref y)==true)
            {
            }
            else
            {
                if(  SearchLinkNum(4, turn, ref x, ref y)==true)
                {
                }
                else
                {
                    if(SearchLinkNum(4, emery, ref x, ref y)==true)
                    {
                    }
                    else
                    {
                        if(doubleThree(turn, ref x, ref y)==true)
                        {
                        }
                        else
                        {
                            if(doubleThree(emery, ref x, ref y)==true)
                            {
                            }
                            else  
                            {
                                if(  SearchLinkNum(3, turn, ref x, ref y)==true)
                                {
                                }
                                else
                                {
```
