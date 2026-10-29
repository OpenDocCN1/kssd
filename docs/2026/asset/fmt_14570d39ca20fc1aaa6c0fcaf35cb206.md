
### 登录方法相关变量

```java
// login方法所属类中定义的变量

private Type DEFAULT_TYPE;

private EditText etMobile;

private EditText etPwd;

private long firstime;

private Map para;

private JsonRequest request;
```

### `login` 方法实现

```java
private void login(String userName, String pwd) {

        this.DEFAULT_TYPE = new TypeToken() {

        }.getType();

        this.para.clear();

        this.para.put("username", userName);

        this.para.put("userPwd", pwd);

        if(TextUtils.isEmpty(DodonewOnlineApplication.devId)) {

            DodonewOnlineApplication.devId = Utils.getDevId(DodonewOnlineApplication.getAppContext());

        }

        this.para.put("equtype", "ANDROID");

        this.para.put("loginImei", "Android" + DodonewOnlineApplication.devId);

        this.requestNetwork("user/login", this.para, this.DEFAULT_TYPE);

    }
```

以上代码是一个Android应用中的登录功能的实现。首先，有一个`login`方法，该方法接收用户名和密码作为参数。在`login`方法中，首先设置了`DEFAULT_TYPE`为一个`TypeToken`的类型。然后清空了`para`（参数）集合，并向其中添加了用户名和密码。接着判断了设备ID是否为空，如果为空，则获取设备ID并添加到参数中。然后设置了`equtype`为`"ANDROID"`，`loginImei`为`"Android"`加上设备ID。最后调用了`requestNetwork`方法来发起登录请求。

从`login`方法的两个参数名可以看出传入到`login`方法中的参数是用户输入的账号和密码，实在不确认我们可以再往上一层查看：

```java
@Override  // android.view.View$OnClickListener

    public void onClick(View v) {

        switch(v.getId()) {

            case 0x7F0D00C0: {  // id:btn_forget_password

                this.startActivity(new Intent(this, FindPasswordActivity.class));

                return;

            }

            case 0x7F0D00C1: {  // id:btn_login

                String s = this.etMobile.getText() + "";

                String s1 = this.etPwd.getText() + "";

                Utils.hideSoftInput(this, this.etPwd);

                if(this.checkInput(s, s1)) {

                    this.login(s, s1);

                    return;

                }

                return;

            }

            case 0x7F0D00C3: {  // id:btn_register_now

                this.startActivity(new Intent(this, RegisterActivity.class));

                return;

            }

            default: {

                return;

            }

        }

    }
```

可以很直白的看出以上代码是一个在Android应用中实现的`View.OnClickListener`的`onClick`方法。该方法根据被点击的视图的ID执行不同的操作。

- 当被点击的视图的ID为`0x7F0D00C0`（`btn_forget_password`）时，会启动`FindPasswordActivity`。
- 当被点击的视图的ID为`0x7F0D00C1`（`btn_login`）时，会获取`etMobile`和`etPwd`中的文本，隐藏软键盘，检查输入，然后调用`login`方法。
- 当被点击的视图的ID为`0x7F0D00C3`（`btn_register_now`）时，会启动`RegisterActivity`。
- 对于其他视图，方法则不执行任何操作。

总的来说，这个`onClick`方法处理了特定视图的点击事件，并根据视图的ID执行不同的操作，而要传入到`login`方法中的参数，在传入前会获取`etMobile`和`etPwd`中的文本，再结合`login`方法中的代码，可以断定传入到`login`方法中的参数八九不离十是账号和密码了，如果还不确认可以动态调试，将断点下在调用`login`方法的位置，一看寄存器的值便知这个猜测对不对咯！

现在我们再回到`login`方法中的代码，前面讲过清空了`para`集合，并向其中添加了用户名和密码，然后又压入了`"ANDROID"`和设备ID，最后将`para`集合作为参数传入到了`requestNetwork`方法中进行处理，我们去`requestNetwork`方法中去看看`para`集合对于最后的加密数据做出了什么贡献。

我们可以从前面的`requestNetwork`方法代码中看出，`para`集合被传入到了`addRequestMap`方法中，我们需要进入`addRequestMap`方法中看看进行了怎样的处理。

### `addRequestMap` 方法

```java
public void addRequestMap(Map map0, int a) {

    String s = System.currentTimeMillis() + "";

    if(map0 == null) {

        map0 = new HashMap();

    }

    ((HashMap)map0).put("timeStamp", s);

    String s1 = RequestUtil.encodeDesMap(RequestUtil.paraMap(((HashMap)map0), "sdlkjsdljf0j2fsjk", "sign"), this.desKey, this.desIV);

    JSONObject obj = new JSONObject();

    try {

        obj.put("Encrypt", s1);

        this.mRequestBody = obj + "";

    }

    catch(JSONException e) {

        e.printStackTrace();

    }

}
```

以上这段代码是一个方法，名为`addRequestMap`，它接受一个`Map`对象`map0`和一个整数`a`作为参数，而`map0`就是之前的`para`集合。

在这个方法中：
1. 首先，获取当前时间的毫秒数，并将其转换为字符串`s`。
2. 然后检查`map0`是否为`null`，如果是，则将`map0`实例化为一个新的`HashMap`。
3. 接下来，将键值对`"timeStamp", s`添加到`map0`中。
4. 使用`RequestUtil`工具类的`encodeDesMap`方法对`map0`进行加密处理，然后将其赋值给字符串`s1`。
5. 创建一个新的`JSONObject`对象`obj`，并将加密后的字符串`s1`放入`obj`中的`"Encrypt"`字段。
6. 最后，将`obj`转换为字符串并赋值给类成员变量`mRequestBody`。

经过分析最后发送给服务器的字符串就是在这进行处理的，即使不看整体代码，就看`obj.put("Encrypt", s1);`这一句代码就明白`s1`是最后那JSON数据中的加密值，而`s1`是经过以下代码进行处理得到的：

```java
String s1 = RequestUtil.encodeDesMap(RequestUtil.paraMap(((HashMap)map0), "sdlkjsdljf0j2fsjk", "sign"), this.desKey, this.desIV);
```

我们还可以从这句代码中的`this.desKey, this.desIV`两个参数看出`encodeDesMap`方法极大可能是DES对称加密，你们可能会疑惑`this.desKey, this.desIV`是什么玩意？这个是`addRequestMap`方法所属类初始化后的变量，而`desKey`应该是DES对称加密所使用的密钥，而`desIV`是CBC加密模式所要用到的初始向量，这些会在加解密的时候详解，这里我们就继续看代码了。这两个变量的值我们看一段`addRequestMap`方法所属类的一段代码就清楚了：

### `JsonRequest` 类初始化

```java
public class JsonRequest extends JsonBaseRequest {

    private static final Type DEFAULT_TYPE;

    private Context context;

    private String desIV;

    private String desKey;

    private Gson mGson;

    private Handler mHandler;

    private String mRequestBody;

    private Type typeOfT;

    private boolean useDes;

    static {

        JsonRequest.DEFAULT_TYPE = new TypeToken() {

        }.getType();

    }

    public JsonRequest(Context context, int method, String url, String requestBody, Listener response$Listener0, ErrorListener errorListener, Type typeOfT) {

        super(method, url, requestBody, response$Listener0, errorListener);

        this.useDes = true;

        this.mGson = new GsonBuilder().serializeNulls().create();

        this.desKey = "65102933";

        this.desIV = "32028092";

        this.mHandler = new Handler() {

            @Override  // android.os.Handler

            public void handleMessage(Message msg) {

                if(msg.what == 0) {

                    JsonRequest.this.intentLoginActivity();

                }

            }

        };

        this.context = context;

        this.typeOfT = typeOfT;

        this.mRequestBody = requestBody;

    }
```

`JsonRequest`类我只截取了一小段，可以看到`this.desKey, this.desIV`两个参数在构造函数中进行了初始化，我们现在知道了这两个值，前面我们讲到`encodeDesMap`方法极大可能是DES对称加密，那么`RequestUtil.paraMap(((HashMap)map0)`极有可能是需要进行DES加密的数据，而`paraMap`自然是对`map0`这个集合进行处理，经过这一路的追寻我们知道`map0`这个集合中被压入了哪些数据，那么我们进入`paraMap`方法中看看DES最后要加密的值是怎么处理的，以及DES要加密的值的明文是怎样的。

### `paraMap` 方法

```java
public static String paraMap(Map map0, String append, String sign) {

    try {

        Set set0 = map0.keySet();

        StringBuilder builder = new StringBuilder();

        ArrayList list = new ArrayList();

        for(Object object0: set0) {

            String keyName = (String)object0;

            list.add(keyName + "=" + ((String)map0.get(keyName)));

        }

        Collections.sort(list);

        int i;

        for(i = 0; i < list.size(); ++i) {

            builder.append(((String)list.get(i)));

            builder.append("&");

        }

        builder.append("key=" + append);

        map0.put("sign", Utils.md5(builder.toString()).toUpperCase());

        String s3 = new Gson().toJson(RequestUtil.sortMapByKey(map0));

        Log.w("yang", s3 + "   result");

        return s3;

    }

    catch(Exception e) {

        e.printStackTrace();

        return "";

    }

}
```

仔细看可以发现以上代码做了这些事情：
1. 获取`Map`中的所有`key`值，存入`Set`集合中。
2. 遍历`Set`集合，将每个`key`值及其对应的`value`值拼接成`"key=value"`的形式，存入`ArrayList`集合中。
3. 对`ArrayList`集合进行排序。
4. 遍历排序后的`ArrayList`集合，将每个`"key=value"`字符串使用`"&"`拼接成一个完整的URL参数字符串，并在最后添加上`"key=append"`，`append`变量为`addRequestMap`方法中传入的字符串`"sdlkjsdljf0j2fsjk"`。
5. 对拼接后的URL参数字符串进行MD5加密，并将加密结果存入`Map`中的`"sign"`键值对中。
6. 对`Map`中的键值对按照键名进行排序，并将排序后的`Map`转换成JSON格式的字符串。
7. 返回JSON格式的字符串。

从以上流程中可以发现在字符串`md5`消息摘要算法处理之前是需要进行排序的，这是因为像消息摘要算法是不可逆的，同样的数据不同的顺序得到的结果甚至会完全不一样，而消息摘要算法就是通过结果是否相等来判断数据是否被修改过，所以在进行消息摘要算法之前都是需要进行排序的。

如果我们想要获取明文数据，那么我们可以在`tostring()`之后数据处理之前进行下断，然后动态调试就可以获取到明文数据，但在这之前，我们先去认识一下常规的MD5消息摘要算法的代码大概长什么样，这一期我们简单了解一下加解密与算法，下一期我们再详细讲讲加解密与算法。
