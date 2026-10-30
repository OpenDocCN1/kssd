
```c
fprintf(stderr, "Error: Cannot replace config file: %s\n", strerror(errno));

// 尝试删除临时文件
remove(temp_file);
result = 1;
}

// 尝试删除临时文件
remove(temp_file);
result = 1;
}

/**
* @brief 检查配置项是否存在
* @param key 配置键名
* @return true存在，false不存在
*/
bool acosNvramConfig_exist(const char *key)
{
char *value = nvm_get_value(key);
bool exists = (value != NULL);
printf("[DEBUG] Exist config: %s = %s\n", key, exists ? "true" : "false");
if (value) {
free(value);
}
return exists;
}

/**
* @brief 测试函数
*/
void test_config_functions(void)
{
printf("=== Testing Config Functions ===\n");
// 测试设置和获取
acosNvramConfig_set("server_ip", "192.168.1.1");
char *ip = acosNvramConfig_get("server_ip");
if (ip) {
printf("Got server_ip: %s\n", ip);
free(ip);
}
// 测试修改
acosNvramConfig_set("server_ip", "10.0.0.1");
ip = acosNvramConfig_get("server_ip");
if (ip) {
printf("Updated server_ip: %s\n", ip);
free(ip);
}
// 测试匹配
bool matched = acosNvramConfig_match("server_ip", "10.0.0.1");
printf("Match test: %s\n", matched ? "PASS" : "FAIL");
// 测试读取整数
acosNvramConfig_set("port", "8080");
int port;
if (acosNvramConfig_readAsInt("port", &port, 80) == 0) {
printf("Port: %d\n", port);
}
// 测试检查并设置
acosNvramConfig_check_set("new_setting", "value123");
// 测试存在性检查
bool exists = acosNvramConfig_exist("new_setting");
printf("new_setting exists: %s\n", exists ? "true" : "false");
// 测试删除
acosNvramConfig_unset("new_setting");
exists = acosNvramConfig_exist("new_setting");
printf("new_setting after unset exists: %s\n", exists ? "true" : "false");
printf("=== Test Complete ===\n");
}

/**
* @brief 主函数（测试用）
*/
int main(int argc, char *argv[])
{
// 如果带参数"-test"，运行测试
if (argc > 1 && strcmp(argv[1], "-test") == 0) {
test_config_functions();
return 0;
}
// 简单演示
printf("Config Management System\n");
printf("Config file: %s\n", ROUTE_CONFIG_PATH);
// 示例：设置和获取配置
acosNvramConfig_set("test_key", "test_value");
char *value = acosNvramConfig_get("test_key");
if (value) {
printf("test_key = %s\n", value);
free(value);
}
return 0;
}
```

```ini
fw_spi=1
usb_info_dev7=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
http_server_wan_enable=0
wifi_mac12=62:c0:02:11:22:33
wan_fix_dns=0
wifi_wep_on12=0
usb_info_dev83=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_auth_type3=3
wan_iptype=Dynamic
usb_info_dev8=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
usb_info_dev63=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_ssid4=NETGEAR_Guest3
usb_info_dev53=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
usb_info_dev16=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wan_vci_change=0
usb_info_dev37=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
interface=wan1,wan2,wan3,wan4,wan5,wan6,wan7,wan8,vlan1,vlan2,vlan3,vlan4,ssid1,ssid2,ssid3,ssid4,PhonePort(1&2)
usb_info_dev70=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wiz_language=English
usb_info_dev67=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wl0_channel=0
usb_info_dev32=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
port_trigger_dis=0
wifi_def_key2=1
usb_info_dev11=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_if_on12=0
usb_info_dev6=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
usb_info_dev28=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_def_key14=1
lan_ipaddr=192.168.1.1
upnp_enable=1
fw_response_ping=0
wan_dod=1
usb_info_dev79=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_wep_on13=0
mail_block=1
wifi_ssid1=NETGEAR-5G
usb_info_dev68=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
usb_info_dev45=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_if_on4=0
wan_vpi=0
usb_info_dev99=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
ntp_custom=0
usb_info_dev1=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
usb_info_dev30=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
ftp_server_enable=0
usb_info_dev75=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
upnp_ttl=4
mail_enable=0
ftp_server_wan_port=21
wifi_if_on2=0
fw_import=0
wan_vci=38
usb_info_dev61=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_def_key4=1
wps_config_state=1
wan_dslmode=MMODE
static_if_status=1
upnp_adv_time=30
wifi_broadcast_ssid14=1
ddns_use_wildcards=0
rt_rip_version=0
usb_info_dev0=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_key_len3=0
wifi_allow_guest4=0
fw_remote_port=8443
usb_info_dev59=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
1:macaddr=01:23:45:67:89:ab
wifi_if_on1=1
pppoe2_schedule_config=127:0:0:23:59
wifi_ssid3=NETGEAR_Guest2
wan_hwaddr_def=01:23:45:67:89:ab
port_forward_dis=0
wifi_allow_guest3=0
wl0_ssid=NETGEAR_11G
wifi_ssid12=NETGEAR-5G_Guest
usb_info_dev31=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
usb_info_dev15=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
qos_enable=0
wifi_ssid13=NETGEAR-5G_Guest2
lan_ifnames=eth1 eth2 eth3 eth4
wifi_if_on13=0
wifi_frag_thre=2346
wifi_key_len14=0
wifi_broadcast_ssid2=1
wifi_dot11_mode1=9
usb_info_dev54=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
ssidn=2
fw_block_trust_enable=0
usb_info_dev78=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
fw_block=0
lan_proto=dhcp
dhcp_end_ip=192.168.1.254
usb_info_dev100=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
usb_info_dev51=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_broadcast_ssid12=1
wifi_dot11_iso1=0
wifi_auth_type2=3
usb_info_dev58=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wanif=eth0
usb_info_dev38=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
usb_info_dev26=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
usb_info_dev35=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_allow_guest14=0
usb_info_dev73=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
wifi_dot11_mode=8
wifi_region1=Asia
wifi_ssid14=NETGEAR-5G_Guest3
wifi_mac=00:c0:02:11:22:33
wan_encap=1
usb_info_dev46=A200396E0402FF83@1@14.4G@U@1@USB_Storage;U:;0;0@
bs_trustedip_enable=0
```
