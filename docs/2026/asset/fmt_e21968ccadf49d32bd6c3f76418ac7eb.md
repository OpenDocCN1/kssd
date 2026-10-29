
# [原创]【Android】利用爬虫将电影网站打包成一个APP

以前总是通过`HDMI`线将电脑连接电视看剧看电影，但是每次想快进啊进行一些操作过于麻烦，手机上的APP的资源又太少，于是想到将电影网站打包成一个APP！

APP已经差不多做好了，最近也比较忙，没什么时间去弄，界面很丑，这里我讲讲我的主要思路，自己动手才有乐趣！

在做之前整理下思路，怎样才能把网站搬到APP上呢？

有两个方法：

1. 通过`WebView`简单粗暴将整个网站搬过来，几十行代码搞定！但是用户体验不是很好，广告多，字体小，操作也不是很方便
2. 利用爬虫，需要什么爬什么！然后将数据显示在自己做的界面上！

Let'go

首先我们搭一个界面大概的界面出来

[Java]纯文本查看复制代码

```
ButterKnife.bind(this);

mTabBar.init(getSupportFragmentManager())
    .setImgSize(50, 50)
    .setFontSize(10)
    .setTabPadding(4, 6, 10)
    .setChangeColor(Color.RED, Color.DKGRAY)
    .addTabItem("首页", R.mipmap.ic_launcher_round, HomeFragment.class)
    .addTabItem("电影", R.mipmap.ic_launcher_round, MoveFragment.class)
    .addTabItem("电视剧", R.mipmap.ic_launcher_round, TVplayFragment.class)
    .addTabItem("动漫", R.mipmap.ic_launcher_round, CartoonFragment.class);
```

一个4个`Fragment`绑定一个`activity`，每个`fragment`：

上面是个`recyclerview`

底部是`BottomTabBar`

![](https://bbs.kanxue.com/upload/attach/201808/785309_DN5UYQBWW89EYAJ.png)

界面搭好了就差数据了

导入图片加载和网络请求依赖

```
compile 'com.squareup.picasso:picasso:2.5.2'
compile 'com.mcxiaoke.volley:library:1.0.19'
```

打开网站，`F12`查看当前网页源码，找到需要的数据

![](https://bbs.kanxue.com/upload/attach/201808/785309_XSEEJFHEA93GWBV.png)

首先通过网络请求获取整个页面数据，然后正则匹配找到我们需要的数据，图片、链接、标题和评分，新建一个`Bean`类将数据储存到里面

[Java]纯文本查看复制代码

```
public class DataBean {

    private String DataName;
    private String DataScore;
    private String DataImg;
    private String DataNetWork;

    public String getDataName() {
        return DataName;
    }

    public void setDataName(String dataName) {
        DataName = dataName;
    }

    public String getDataScore() {
        return DataScore;
    }

    public void setDataScore(String dataScore) {
        DataScore = dataScore;
    }

    public String getDataImg() {
        return DataImg;
    }

    public void setDataImg(String dataImg) {
        DataImg = dataImg;
    }

    public String getDataNetWork() {
        return DataNetWork;
    }

    public void setDataNetWork(String dataNetWork) {
        DataNetWork = dataNetWork;
    }
}
```

通过`recyclerView`将数据显示到界面上，通过接口回调设置点击事件点击图片跳转到详情页

[Java]纯文本查看复制代码

```
RequestQueue queue = Volley.newRequestQueue(getActivity());

MyStringRequest stringRequest = new MyStringRequest(getHosturl(), new Response.Listener<String>() {

    @Override
    public void onResponse(String response) {
        String regEx = "<li><div class=li-box><div class=img-box></div><a href=\"(.+?)\"><img src=\"(.+?)\" onerror=\".+?\"><span class=back></span><span>(.+?)</span></div><P><a href=\".+?\" target=\"_blank\">(.+?)</a></P></li>";
        Pattern pattern = Pattern.compile(regEx);
        mMatcher = pattern.matcher(response);
        if (mData != null) {
            mData.clear();
        }
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (mMatcher.find()) {
                    //Log.e(TAG, matcher.group());
                    /*Log.e(TAG, mMatcher.group(1));
                    Log.e(TAG, mMatcher.group(2));
                    Log.e(TAG, mMatcher.group(3));
                    Log.e(TAG, mMatcher.group(4));*/
                    DataBean dataBean = new DataBean();
                    dataBean.setDataNetWork(mMatcher.group(1));
                    dataBean.setDataImg(mMatcher.group(2));
                    dataBean.setDataName(mMatcher.group(4));
                    dataBean.setDataScore(mMatcher.group(3));
                    mData.add(dataBean);
                }
            }
        }).start();

        if (mHomeAdapter == null) {
            mHomeAdapter = new homeAdapter(getContext(), mData, mGridLayoutManager);
        }
        mRecyclerView.setAdapter(mHomeAdapter);
        mHomeAdapter.notifyDataSetChanged();
        mTvThisPage.setText("第"+mThisPage+"页");
        Toast.makeText(getContext(), "第" + mThisPage + "页", Toast.LENGTH_SHORT).show();

        mHomeAdapter.setItemClickListener(new homeAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(int position) {
                String url = mData.get(position).getDataNetWork();
                String title = mData.get(position).getDataName();
                String img_url = mData.get(position).getDataImg();
                String requestUrl = getHosturl() + url;
                //Toast.makeText(getContext(), requestUrl, Toast.LENGTH_SHORT).show();
                Intent intent = new Intent(getContext(), DetailsActivity.class);
                intent.putExtra("title", title);
                intent.putExtra("img_url", img_url);
                intent.putExtra("requestUrl", requestUrl);
                startActivity(intent);
            }
        });
        //Log.e(TAG, response);
    }
}, new Response.ErrorListener() {

    @Override
    public void onErrorResponse(VolleyError error) {
        Log.e(TAG, error.getMessage(), error);
        mThisPage--;
        Toast.makeText(getContext(), "网络不稳定!加载失败!请稍后重试!", Toast.LENGTH_SHORT).show();
    }
});

queue.add(stringRequest);
```

![](https://bbs.kanxue.com/upload/attach/201808/785309_E4X8WM2F5E4USX4.png)

为了便于维护将首页封装成基类，然后将电影、电视剧和动漫页面继承，稍加改动，这样4个页面就出来了

[Java]纯文本查看复制代码

```
public class MoveFragment extends HomeFragment implements View.OnClickListener {

    @Override
    protected void intitView() {
        hostUrl = "02cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2Q4x3X3g2&6K9i4W2T1j5W2)9J5k6h3y4G2L8g2)9J5c8X3c8A6j5h3&6&6K9h3&6Y4i4K6u0r3";
        super.intitView();
        mThisPage = 1;
        listTpye = "List_15_";
        mTvThisPage.setVisibility(View.VISIBLE);
    }

    @Override
    protected void initData() {
        super.initData();
    }

    @Override
    protected String setTitle() {
        return "电影";
    }

    @Override
    protected void setViibilly() {
        page.setVisibility(View.VISIBLE);
    }
}
```

第二大步：

然后我们来做详情页

通过`Intent`获取到上个页面传递过来的`URL`、图片链接和标题，通过`URL`获取整个页面数据，然后正则匹配，其他需要的数据

![](https://bbs.kanxue.com/upload/attach/201808/785309_65WTHT7NJMYBYGP.png)

[Java]纯文本查看复制代码

```
RequestQueue queue = Volley.newRequestQueue(this);

MyStringRequest stringRequest = new MyStringRequest(mRequestUrl, new Response.Listener<String>() {

    @Override
    public void onResponse(String response) {
        //Log.e(TAG, response );
        String regEx = "<li><a href=\"/(.+?)\" >(.+?)</a></li>";
        /*
        <div ><ul><div ><div ></div><img src="http://88.meenke.com/img_buyhi/201805/2018052876045761.jpg" alt="帝王攻略" border="0"></div><h1>帝王攻略</h1><li>更新至：[17]</li><li>年　代：2018</li><li>类　型：<a href="/dhp_lianzai/Index.html" target="_blank">动画连载</a></li><li ><a id="shoucang" href="#sc">收藏</a></li></ul></div>
        */
        Pattern pattern = Pattern.compile(regEx);
        mMatcher = pattern.matcher(response);
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (mMatcher.find()) {
                    //Toast.makeText(DetailsActivity.this, mMatcher.group() + "", Toast.LENGTH_SHORT).show();
                    //Log.e(TAG, mMatcher.group());
                    //Log.e(TAG, mMatcher.group(1));
                    //Log.e(TAG, mMatcher.group(2));
                    String url = mMatcher.group(1);
                    String title = mMatcher.group(2);
                    //Log.e(TAG, mMatcher.group(3));
                    //Log.e(TAG, mMatcher.group(4));
                    DetailsBean dataBean = new DetailsBean();
                    dataBean.setTitle(title);
                    dataBean.setUrl(url);
                    mData.add(dataBean);
                }
            }
        }).start();

        DetailAdapter adapter = new DetailAdapter(DetailsActivity.this, mData, mGridLayoutManager);
        mDaRecyView.setAdapter(adapter);

        adapter.setItemClickListener(new DetailAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(int position) {
                String url = "297K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2Q4x3X3g2&6K9i4W2T1j5W2)9J5k6h3y4G2L8g2)9J5c8R3`.`." + mData.get(position).getUrl();
                Intent intent = new Intent(DetailsActivity.this, PlayActivity.class);
                intent.putExtra("url", url);
                startActivity(intent);
            }
        });
    }
}, new Response.ErrorListener() {

    @Override
    public void onErrorResponse(VolleyError error) {
        Log.e(TAG, error.getMessage(), error);
        Toast.makeText(DetailsActivity.this, "网络不稳定!", Toast.LENGTH_SHORT).show();
    }
});

queue.add(stringRequest);
```

第三大步：

最重要就是播放页面了，这里费了点时间

刚开始我也是按照前2步骤去操作，但是发现怎么都爬不到获取不到链接，打开原网站才发现播放链接是动态加载的而且还3层加密了！MMP

打开播放页面，`F12`选择`sources`，查看源码

![](https://bbs.kanxue.com/upload/attach/201808/785309_2GTC2PNERN98GFE.png)

`<div >`就是播放页面，但是里面空的，证明这是动态加载了这个标签

继续看源码找`script`标签看下里面做了什么

```
<script language="javascript">var StrHtml;var url=set_code(unescape("JN0HT%250G%256B%256BJeN.Ty6l.167%256Bx%256BlDvoVW%256B1dTw8xE.mkyw",0,0));var nexturl="no";var nextpath="no";var Player={Url:url,Height:240,Width:600,Show:null};function $ShowPlayer(w,h){document.write($Showhtml());}</script>
<script language="javascript" src="Play/23.js"></script><script language="javascript">$ShowPlayer(600,240);</script>
```

`JN0HT%250G%256B%256BJeN.Ty6l.167%256Bx%256BlDvoVW%256B1dTw8xE.mkyw`是第一加密后的链接，至于怎么来的，我们不管他，这个数据是可以爬出来的

`unescape()`方法是`JS`自带解密方法

`set_code()`再次加密

找到了主要的逻辑，顺着思路往下走，

在`netWork`刷新页面看下加载了那些东西，主要看`js`，发现加载了这么几个东东
