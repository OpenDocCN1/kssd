
# 一个秒转换时间的c++小类,ntp可以用到,做时间转换可以参考

如果需要用时间加密也可以参考

```cpp
struct zxptime
{
    zxptime();
    UINT iyear,imonth,iday;
    UINT ihour,imin,isec;
    void GetTime70(UINT tt); // 从1970年开始
    void GetTime900(UINT tt); // 从1900年开始
    void print(char *buff);
};

zxptime::zxptime()
{
    iyear=imonth=iday=0;
    ihour=imin=isec=0;
}

void zxptime::GetTime70(UINT tt)
{
    UINT myear,mday,oday;
    UINT msec;
    UINT ttmonth[12]={
        31,28,31,30,31,30,
        31,31,30,31,30,31
    };
    mday=tt/(24*3600);
    msec=tt-mday*(24*3600);
    myear=mday/365;
    oday=mday-myear*365-(myear+1)/4;
    int y, i=0;
    UINT iimoth;
    y=(1970+myear);
    iimoth=((y%4==0)&&(y%100!=0))||(y%400==0);
    for(i=0;i<12;i++)
    {
        if((y=oday-ttmonth[i])>0)
            oday-=ttmonth[i]+(i==1?iimoth:0);
        else
            break;
    }
    iday=oday+1;
    imonth=i+1;
    iyear=1970+myear;
    ihour=msec/3600;
    imin=(msec-ihour*3600)/60;
    ihour+=8; // 中国8时区
    isec=msec%60;
};

void zxptime::GetTime900(UINT tt)
{
    UINT myear,mday,oday;
    UINT msec;
    UINT ttmonth[12]={
        31,28,31,30,31,30,
        31,31,30,31,30,31
    };
    mday=tt/(24*3600);
    msec=tt-mday*(24*3600);
    myear=mday/365;
    oday=mday-myear*365-(myear)/4;
    int y, i=0;
    UINT iimoth;
    y=(1900+myear);
    iimoth=((y%4==0)&&(y%100!=0))||(y%400==0);
    for(i=0;i<12;i++)
    {
        if((y=oday-ttmonth[i])>0)
            oday-=ttmonth[i]+(i==1?iimoth:0);
        else
            break;
    }
    iday=oday+1;
    imonth=i+1;
    iyear=1900+myear;
    ihour=msec/3600;
    imin=(msec-ihour*3600)/60;
    ihour+=8; // 中国8时区
    isec=msec%60;
};

void zxptime::print(char *buff)
{
    sprintf(buff,"%02d-%02d-%02d %02d:%02d:%02d ",iyear,imonth,iday,
            ihour,imin,isec);
    //printf("%s\n",buff);
};
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
