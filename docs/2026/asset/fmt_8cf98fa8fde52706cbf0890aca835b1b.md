
# 记一次某推上的session利用trick

## 写在前面

在一次浏览某推中发现了发现了了一个web challenge的赏金ctf，这里从来学习一下由于使`session_start()`报错引发的危害

## 正文

题目环境地址

<mark class="encrypted">7cdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5^5i4K6u0W2x3e0R3#2i4K6u0W2x3e0c8Q4x3X3f1J5x3o6u0Q4x3V1k6U0K9r3q4D9L8o6q4Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8Y4m8S2k6$3g2Q4x3@1c8K6K9r3!0%4e0h3g2f1K9r3g2o6L8$3c8W2</mark>

### 程序分析

`index.php`

```php
<?php
define('DEV\_MODE', false);
class Session
{
public static $id = null;
protected static $isInit = false;
protected static $started = false;
public static function start()
{
self::$isInit = true;
if (!self::$started) {
if (!is\_null(self::$id)) {
session\_id(self::$id);
self::$started = session\_start();
} else {
self::$started = session\_start();
self::$id = session\_id();
}
}
}
public static function stop()
{
if (self::$started) {
session\_write\_close();
self::$started = false;
}
}
public static function destroy() {
session\_destroy();
}
public static function set($key, $value)
{
if (!isset($\_SESSION) || self::get($key) == $value) {
return;
}
if (!self::$started) {
self::start();
$\_SESSION[$key] = $value;
self::stop();
} else {
$\_SESSION[$key] = $value;
}
}
public static function get($key)
{
if (isset($\_SESSION)) {
return $\_SESSION[$key];
}
return null;
}
public static function isInit()
{
return self::$isInit;
}
}
class User {
private $users;
private $states = ['start', 'checkCreds', 'credsValid', 'userState', 'connected', 'error'];
private $banlist = ['blackhat', 'notkindguy'];
function \_\_construct(){
$userFile = '/users.txt';
$fp = fopen($userFile,'r');
while(($userLine = fgets($fp))!==false){
$user = explode(':',trim($userLine),2);
$this->users[] = $user;
}
}
function login($username, $password){
$state = Session::get('state');
if($state === 'connected' && Session::get('authenticated') === true) exit;
if(method\_exists($this,$state)){
$this->$state($username, $password);
} else {
$this->start($username, $password);
}
}
function start($username, $password) {
// NOT IN USE FOR NOW
Session::set('state', 'checkCreds');
$this->login($username, $password);
}
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
function credsValid($username, $password) {
Session::set('user', $username);
Session::set('state', 'userState');
$this->login($username, $password);
}
function userState($username, $password) {
if(in\_array($username, $this->banlist)) {
Session::set('user',null);
Session::set('state','error');
$this->login($username, $password);
return;
} else {
Session::set('state', 'connected');
$this->login($username, $password);
}
}
function connected($username, $password) {
Session::set('authenticated',true);
echo "Welcome $username, you're connected! Have a great day.";
}
function error($username, $password) {
echo "Your login or password is incorrect, or you're banned :(";
Session::destroy();
return;
}
function getFlag() {
if(Session::get('user') === 'admin' && Session::get('authenticated')) {
echo file\_get\_contents('/flag.txt');
} else {
echo "No flag for you";
}
}
}
Session::start();
$users = file\_get\_contents('/users.txt');
if(isset($\_GET['page'])) {
switch($\_GET['page']) {
case 'login':
$user = new User();
$user->login($\_GET['username'],$\_GET['password']);
break;
case 'flag':
$user = new User();
$user->getFlag();
break;
case 'showMeTheCode':
highlight\_file(\_\_FILE\_\_);
exit;
}
}
```

`users.txt`

```text
user:user
```

我们仅仅只有一个账户`username`为`user`, `password`为`user`

我们又怎么能够达到在登陆admin账户之后进行flag的获取

那么肯定是需要越权的实现了

简单分析一下代码吧

```php
class Session
{
public static $id = null;
protected static $isInit = false;
protected static $started = false;
public static function start()
{
self::$isInit = true;
if (!self::$started) {
if (!is\_null(self::$id)) {
session\_id(self::$id);
self::$started = session\_start();
} else {
self::$started = session\_start();
self::$id = session\_id();
}
}
}
public static function stop()
{
if (self::$started) {
session\_write\_close();
self::$started = false;
}
}
public static function destroy() {
session\_destroy();
}
public static function set($key, $value)
{
if (!isset($\_SESSION) || self::get($key) == $value) {
return;
}
if (!self::$started) {
self::start();
$\_SESSION[$key] = $value;
self::stop();
} else {
$\_SESSION[$key] = $value;
}
}
public static function get($key)
{
if (isset($\_SESSION)) {
return $\_SESSION[$key];
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

存在有一个`\_\_construct`这个魔术方法，在创建对象的时候将会进行调用

主要是从`/users.txt`中读取账户

存在有`login`函数

```php
function login($username, $password){
$state = Session::get('state');
if($state === 'connected' && Session::get('authenticated') === true) exit;
if(method\_exists($this,$state)){
$this->$state($username, $password);
} else {
$this->start($username, $password);
}
}
```

传入`username`和`password`参数，首先从session中获取`state`值，如果其为`connected` 并且已经被被认证了就会直接退出

对应的如果存在有从`state`中获得的方法，就会调用其方法

如果没有，就调用`start`函数

```php
function start($username, $password) {
// NOT IN USE FOR NOW
Session::set('state', 'checkCreds');
$this->login($username, $password);
}
```

他会创建一个`$_SESSION['state'] = checkCreds`，之后再次调用`login`方法，根据上面的描述将会调用`checkCreds`方法

```php
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

在这个方法中，进行了身份的校验，通过从`/users.txt`中获取的账户对传入的参数`username`和`password`进行了判断，这里进行了强比较，所以也就不存在php的弱比较绕过了

如果不满足校验将创建一个`$_SESSION['state'] = error`，之后调用`login`方法，进而调用了`error`方法

```php
function error($username, $password) {
echo "Your login or password is incorrect, or you're banned :(";
Session::destroy();
return;
}
```

在`error`方法中将会销毁掉session并返回null

如果通过了前面的校验

就会创建一个`$_SESSION['state'] = credsValid`， 之后再次调用`login`，进而调用了`credsValid`方法

```php
function credsValid($username, $password) {
Session::set('user', $username);
Session::set('state', 'userState');
$this->login($username, $password);
}
```

在这个方法中将会将传入的`username`参数创建一个`$_SESSION['user'] = $username` 和 `$_SESSION['state'] = 'userState'`，之后调用了`login`方法，进而调用了`userState`方法

```php
function userState($username, $password) {
if(in\_array($username, $this->banlist)) {
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

如果传入的`useranme`参数在`banlist`名单中将会出现异常(有一说一，我感觉没有任何作用)

```text
private $banlist = ['blackhat', 'notkindguy'];
```

如果不存在，就会调用`connected`方法

```php
function connected($username, $password) {
Session::set('authenticated',true);
echo "Welcome $username, you're connected! Have a great day.";
}
```

赋予`$_SESSION['authenticated'] = true`

那么我们最后需要达到的目标就是

```php
function getFlag() {
if(Session::get('user') === 'admin' && Session::get('authenticated')) {
echo file\_get\_contents('/flag.txt');
} else {
echo "No flag for you";
}
}
```

不仅需要`username` 为`admin`, 而且还是需要认证的`admin`才会得到`flag`

我们通过上面的分析似乎走进了死胡同，但是还是有可以突破的点

### 突破

我们通过上面的分析，相信我们能够注意到在`credsValid`方法中存在和我们传入参数进行交互的点

他在代码中将其传入给了session中的`user`值，如果我们能够在这一步使得传入的`username`为`admin`，是不是后面获取`flag`就是格外的轻松了呢？

那是直接传入`admin`还是有一个问题！

那就是调用`credsValid`方法之前还有一步通过调用了`checkCreds`判断了`username`和`password`的可用性

但是如果我们能够获取到`credsValid`那一步的`cookie`, 修改我们的`cookie`值在传入`username=admin`是不是就可以成功突破了呢？

基于这样的思路，我们翻阅`php manual`的文档

<mark class="encrypted">39fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3K9s2m8Q4x3X3g2F1k6i4c8Q4x3V1k6E0j5h3&6#2j5h3I4Q4x3V1k6W2L8W2)9J5c8X3k6#2L8X3y4@1K9h3!0F1i4K6u0W2M7$3g2K6M7$3W2G2L8W2)9J5k6r3W2V1i4K6u0W2M7r3S2H3</mark>

发现在这里存在有这样一段话

![](upload/attach/202209/962655\_CAN6B5F2DYFCA2V.png)
