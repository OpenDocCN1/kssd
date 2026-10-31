
```c
void startGameImpl(game_t *_this)
{
    int ch;
    if (_this->checkCar(_this) == 0)
        return;

    initBoard();
    _this->userCar->col = USER_COL;
    _this->botCar->col = BOT_COL;

    road_t *road = _this->road;
    if (road != NULL)
    {
        road->buildRoad(road, ROAD_BLOCKS);
        road->printEnd(road);
    }

    _this->printBanner();
    _this->userCar->printCar(_this->userCar);
    _this->botCar->printCar(_this->botCar);

    while (1)
    {
        ch = getch();
        if (ch == 'q' || ch == 'Q')
            goto End;
        if (ch == ' ')
            break;
    }

    _this->compete(_this);

End:
    _this->finishGame(_this);
    clear();
    endwin();
}
```

并且我们很容易可以看出来，如果我们进来直接退出，也会调用这个`finishGame`函数，也可能触发一次`performance++`，既然不开始游戏的话我们短时间内还是可以操作很多次的。

我们发现其中两个分支会在某辆车到达终点的时候才会进入的，其它时候根本不会进入。

有一个分支会可能会进入：

```c
if (isSlip(p))
{
    char msg[] = "You are luck!\nYour car's perfomrance increased!";
    mvprintw(MSG_ROW + 1, (winCol - strlen(msg)) / 2, "%s", msg);

    steps += 0.1;
    if (steps == 1.0)
    {
        steps = 0;
        _this->userCar->step += 1;
    }

    if (_this->userCar->stability < 100)
        _this->userCar->stability++;

    _this->userCar->performance++;
}
```

`isSlip`是一个随机性函数，也就是`p%`的概率为 1。这里`p=10`，也就是 10% 的概率，但是这个有点不太够看，算期望的话，我们 1000 次也就多 200 块，刷的效率还是比较低的。

但是其它地方确确实实没问题了，在这个`new_game`当中，没有任何办法可以通过正常手段去改这个`iswon`和`wintimes`了。

止步不前了，我们回到刚刚分析金钱的地方。

还是发现了端倪。

它在算时间戳的时候，会让取出时间乘上一个`difficulty`。

```c
fetchTime = fetchHour * 3600 + fetchMin * 60 + fetchSec * fixDifficulty;  // 0
fixTime = fixHour * 3600 + fixMin * 60 + fixSec;
```

并且给了一个注释 0。

### `difficulty`分析

那么这里我们分析一下这个`difficulty`的字段。

![](https://bbs.kanxue.com/upload/attach/202212/919002_VSYU7YM3AHSJHK4.jpg)

从上到下，第一个是初始化，`difficulty`为 1，第二个是定义，第三个是刚刚的`finishGame`的时候`++`，最后一个是我们在修车的时候赋值，又是`finishGame`，我们继续跳到里面分析。

```c
void finishGameImpl(game_t *_this)
{
    car_t *car = _this->userCar;
    int fuelCost, healthCost;
    int p = 10;
    static double steps;

    fuelCost = car->col / 10;
    healthCost = car->col / 5;
    car->health -= healthCost;
    car->fuel -= fuelCost;

    if (car->health < 0)
        car->health = 0;
    if (car->fuel < 0)
        car->fuel = 0;

    car->fixDifficulty++;

    if (isSlip(p))
    {
        char msg[] = "You are luck!\nYour car's perfomrance increased!";
        mvprintw(MSG_ROW + 1, (winCol - strlen(msg)) / 2, "%s", msg);

        steps += 0.1;
        if (steps == 1.0)
        {
            steps = 0;
            _this->userCar->step += 1;
        }

        if (_this->userCar->stability < 100)
            _this->userCar->stability++;

        _this->userCar->performance++;
    }

    attron(COLOR_PAIR(1));

    if (_this->botCar->isWon)
    {
        int ch;
        char msg[] = "You lose! Press enter to quit...";
        _this->botCar->isWon = 0;

        if (car->stability < _this->botCar->stability)
        {
            car->stability += 5;
            car->performance++;
        }

        mvprintw(BANNER_ROW + 3, (winCol - strlen(msg)) / 2, "%s", msg);
        while ((ch = getch()) != '\n');
    }
    else if (_this->userCar->isWon)
    {
        int ch;
        _this->winTimes++;
        _this->userCar->isWon = 0;
        _this->money += 10;

        mvprintw(
            BANNER_ROW + 3,
            (winCol - 10) / 2,
            "%s",
            "You Won!!! Press enter to quit..."
        );

        while ((ch = getch()) != '\n');
    }

    attroff(COLOR_PAIR(1));
}
```

发现这个逻辑在最开始，也就是无论如何都会被调用的，而且我们发现它的类型为`uint8`，无符号的 8 位整数，那么我们可以让它刷 255 次，导致了它加到最后变成 0，然后我们再不停地修车，就会导致取的时候少加了秒数相减极有可能是负数（修车赚钱），并且我们还可以修很多次，这样的话就可以避免在零点才能刷钱的问题。

那么两个思路一结合就可以做出这一题了。

## `exp`编写

首先给一下交互函数
