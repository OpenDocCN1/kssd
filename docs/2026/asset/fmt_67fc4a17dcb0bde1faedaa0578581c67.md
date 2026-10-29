
```
main函数开始执行
return run_application (endpoint_params, options, on_ready);
private static int run_application (EndpointParameters endpoint_params, ControlServiceOptions options, ReadyHandler on_ready) {
TemporaryDirectory.always_use ((directory != null) ? directory : DEFAULT_DIRECTORY);
application = new Application (new ControlService (endpoint_params, options));
.....
....
return application.run ();
}
```

`application.run`函数

```
public int run () {
Idle.add (() => {
start.begin ();
return false;
});
exit_code = 0;
loop.run ();
return exit_code;
}
```

`start.begin()`调用

```
private async void start () {
try {
yield service.start (io_cancellable); //这里调用的是ControlService的start函数
} catch (GLib.Error e) {
if (e is IOError.CANCELLED)
return;
printerr ("Unable to start: %s\n", e.message);
exit_code = 5;
loop.quit ();
return;
}
Idle.add (() => {
ready ();
return false;
});
}
```

`src\control-service.vala`

```
public async void start (Cancellable? cancellable = null) throws Error, IOError {
if (state != STOPPED)
throw new Error.INVALID_OPERATION ("Invalid operation");
state = STARTING;
try {
yield service.start (cancellable); //这里是WebService 调用start
....
}
```

`frida-core\lib\base\socket.vala`

```
public async void start (Cancellable? cancellable) throws Error, IOError {
............
var start_request = new Promise<SocketAddress> ();
schedule_on_dbus_thread (() => { //闭包，异步调用handle_start_request 函数
handle_start_request.begin (start_request, cancellable);
return false;
});
_listen_address = yield start_request.future.wait_async (cancellable);
}
private async void handle_start_request (Promise<SocketAddress> start_request, Cancellable? cancellable) {
try {do_start
SocketAddress effective_address = yield do_start (cancellable); //调用 do_start等待这个函数返回
.........
}
private async SocketAddress do_start (Cancellable? cancellable) throws Error, IOError {
server = (Soup.Server) Object.new (typeof (Soup.Server),
"tls-certificate", endpoint_params.certificate);
server.add_websocket_handler ("/ws", endpoint_params.origin, null, on_websocket_opened); //这个应该是websocket打开时处理的
if (endpoint_params.asset_root != null)
server.add_handler ("/", on_asset_request); //看前面demo，web处理函数，这个也是
SocketConnectable connectable = (flavor == CONTROL)
? parse_control_address (endpoint_params.address, endpoint_params.port) //设置端口 27042
: parse_cluster_address (endpoint_params.address, endpoint_params.port);
Soup.ServerListenOptions listen_options = (endpoint_params.certificate != null)
? Soup.ServerListenOptions.HTTPS
: 0;
SocketAddress? first_effective_address = null;
var enumerator = connectable.enumerate ();
while (true) {
SocketAddress? address;
try {
address = yield enumerator.next_async (io_cancellable);
} catch (GLib.Error e) {
throw new Error.NOT_SUPPORTED ("%s", e.message);
}
if (address == null)
break;
SocketAddress? effective_address = null;
InetSocketAddress? inet_address = address as InetSocketAddress;
if (inet_address != null) {
uint16 start_port = inet_address.get_port ();
uint16 candidate_port = start_port;
do {
try {
server.listen (inet_address, listen_options); //在这里监听
effective_address = inet_address;
........
.........
}
```

pc端会访问27042端口，去连接frida-server，我们接着分析一下`on_websocket_opened`

```
private void on_websocket_opened (Soup.Server server, Soup.WebsocketConnection connection, string path,
Soup.ClientContext client) {
var peer = new WebConnection (connection); //WebConnection 是个 IOStream
SocketAddress remote_address = client.get_remote_address ();
schedule_on_frida_thread (() => {
incoming (peer, remote_address); 发送incoming 信号
return false;
});
}
```

WebService的incoming信号在WebService初始化函数中注册了,调用了`on_server_connection`

```
service.incoming.connect (on_server_connection);
private void on_server_connection (IOStream connection, SocketAddress remote_address) {
private async void handle_server_connection (IOStream raw_connection) throws GLib.Error {
var connection = yield new DBusConnection (raw_connection, null, DELAY_MESSAGE_PROCESSING, null, io_cancellable);
//通过web传过来的 IOStream，生成了一个DBusConnection
connection.on_closed.connect (on_connection_closed);
Peer peer;
AuthenticationService? auth_service = endpoint_params.auth_service;
if (auth_service != null)
peer = new AuthenticationChannel (this, connection, auth_service); //这个类在gadget,应该是frida-gadget.so的时候会调用
else
peer = setup_control_channel (connection); //这个在本地
peers[connection] = peer;
connection.start_message_processing ();
}
```

这两个调用位置都会去注册dbus函数，我们分析`setup_control_channel`

```
private ControlChannel setup_control_channel (DBusConnection connection) {
return new ControlChannel (this, connection);
}
ControlChannel初始化函数
construct {
try {
HostSession session = this;
registrations.add (connection.register_object (ObjectPath.HOST_SESSION, session));
AuthenticationService null_auth = new NullAuthenticationService ();
registrations.add (connection.register_object (Frida.ObjectPath.AUTHENTICATION_SERVICE, null_auth));
//注册了dbus对象,也就是说，pc端可以远程调用当前这个类的函数，这里完成了，
...........
}
```

pc端先通过web连接到frida-server的端口，然后frida通过web的数据流，注册了一个dbus服务，供pc端调用，这样就实现了远程通讯

看一下ControlChannel，调用的函数

```
public async void input (uint pid, uint8[] data, Cancellable? cancellable) throws GLib.Error {
yield parent.host_session.input (pid, data, cancellable);
}
```

这个函数parent就是ControlService，最终还是调用的ControlService的`host_session`，这个`host_session`，我们后面说

+ control-service 跨进程attach,调用frida-agent-main

前面我们分析了，pc端通过dbus可以远程调用ControlChannel，而ControlChannel，实际上调用的是ControlService。

`frida\frida-core\src\control-service.vala`

```
private async AgentSessionId attach (uint pid, HashTable<string, Variant> options, ControlChannel requester,
Cancellable? cancellable) throws Error, IOError {
AgentSessionId id;
try {
id = yield host_session.attach (pid, options, cancellable); 调用了host_session的attach
} catch (GLib.Error e) {
throw_dbus_error (e);
}
......
}
```

不同的平台实现了不同的`host_session`,通过BaseDBusHostSession来抽象实现相同功能，不同架构实现不同功能

```
#if WINDOWS
var tempdir = new TemporaryDirectory ();
host_session = new WindowsHostSession (new WindowsHelperProcess (tempdir), tempdir);
#endif
#if DARWIN
host_session = new DarwinHostSession (new DarwinHelperBackend (), new TemporaryDirectory (),
opts.report_crashes);
#endif
#if LINUX
var tempdir = new TemporaryDirectory ();
host_session = new LinuxHostSession (new LinuxHelperProcess (tempdir), tempdir, opts.report_crashes);
#endif
#if QNX
host_session = new QnxHostSession ();
#endif
```

我们分析linux平台,在LinuxHostSession初始化函数中，释放了frida-agent文件

前面的`host_session.attach`,调用的是BaseDBusHostSession的attach

```
public async AgentSessionId attach (uint pid, HashTable<string, Variant> options,
Cancellable? cancellable) throws Error, IOError {
var entry = yield establish (pid, options, cancellable); 调用了establish 函数
var id = AgentSessionId.generate ();
entry.sessions.add (id);
try {
yield entry.provider.open (id, options, cancellable);
} catch (GLib.Error e) {
entry.sessions.remove (id);
throw new Error.PROTOCOL ("%s", e.message);
}
agent_sessions[id] = new AgentSessionEntry (entry.connection);
return id;
}
```

BaseDBusHostSession类 establish 函数 401行，调用 `perform_attach_to`，调用的是各个平台的实现的`perform_attach_to`

`src\linux\linux-host-session.vala`
