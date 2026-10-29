
# [原创]Android4.4 wifi代理流程

## 首先简单说下http代理的原理和实现。

举例，比如手机访问`096K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0o6i4@1f1$3i4K6R3&6i4K6S2n7i4@1f1$3i4K6W2o6i4@1u0m8i4@1f1%4i4K6W2m8i4K6R3@1K9i4m8Q4c8e0c8Q4b7U0S2Q4b7V1p5I4z5e0u0Q4x3X3f1I4y4U0S2Q4x3X3f1J5i4K6u0W2x3W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7b7W2!0m8x3#2!0q4y4#2)9&6x3q4)9^5y4W2!0q4y4W2)9&6b7#2)9^5c8q4!0q4y4g2)9^5b7g2!0m8x3g2!0q4y4g2)9&6z5g2!0m8z5q4!0q4y4#2)9&6b7g2)9^5y4r3W2H3i4@1f1@1i4@1t1^5i4@1u0m8x3e0V1J5i4K6u0W2x3e0j5^5i4K6u0W2x3#2)9J5k6e0c8Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0N6Q4b7f1u0Q4b7f1k6Q4c8e0g2Q4z5p5k6Q4b7e0y4Q4c8e0c8Q4b7U0S2Q4b7V1p5^5x3o6R3%4i4@1f1K6i4K6R3H3i4K6R3J5`

代理服务器这边`socket`监听`8087`端口，手机和代理服务器建立`socket`连接，把`http`请求的内容通过`socket`发送到代理服务器，代理服务器解析出`Host`。

![](https://bbs.kanxue.com/upload/attach/201906/670707_SC3YAEMUEZJHCYD.jpg)

得到`host=68eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4@1f1#2i4K6V1J5i4K6S2o6M7r3!0J5N6q4)9K6c8o6R3H3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1%4i4K6R3@1i4@1t1$3i4@1f1#2i4K6V1H3i4K6S2q4i4@1f1^5i4@1p5%4i4@1p5K6i4@1f1$3i4K6W2q4i4K6V1H3N6%4N6%4i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4c8e0g2Q4b7f1k6Q4b7U0W2Q4c8e0g2Q4b7V1q4Q4z5e0c8Q4c8e0N6Q4z5f1q4Q4z5o6c8A6M7q4)9K6c8o6t1J5x3q4)9J5k6e0p5^5x3g2)9J5k6e0x3^5i4K6u0W2x3e0b7^5i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4@1u0n7i4@1u0m8i4@1f1%4i4@1q4n7i4K6S2n7M7$3!0U0K9$3g2@1i4@1f1^5i4@1u0r3i4K6W2q4i4@1f1$3i4K6S2q4i4@1p5#2i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4K6S2m8i4K6S2m8K9s2c8@1M7q4!0q4z5q4!0m8c8W2!0n7y4#2!0q4y4W2!0n7x3g2)9^5x3W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4g2)9^5y4W2)9^5y4g2!0q4y4g2!0m8c8g2!0n7z5g2!0q4z5g2)9^5x3q4)9&6b7g2!0q4z5q4!0n7c8W2)9^5y4%4y4G2j5$3E0W2N6q4!0q4y4g2)9^5c8W2)9&6x3g2!0q4z5g2)9^5x3q4)9^5x3g2!0q4y4g2)9^5z5q4!0n7x3o6t1J5x3q4)9J5k6e0p5^5x3g2)9J5k6e0x3^5i4K6u0W2x3e0b7^5i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1#2i4K6V1H3i4K6S2o6i4@1f1%4i4K6V1H3i4K6R3$3i4@1f1$3i4K6S2m8i4K6S2m8i4@1f1^5i4@1u0r3i4K6V1@1i4@1f1#2i4K6W2n7i4K6W2q4i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6R3$3i4K6R3#2i4@1f1#2i4@1q4q4i4@1t1&6i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1&6i4K6R3H3i4K6R3I4i4@1f1#2i4K6W2n7i4K6W2q4i4@1f1$3i4K6R3&6i4K6S2n7i4@1f1$3i4K6W2o6i4@1u0m8i4@1f1K6i4K6R3H3i4K6R3J5`

目前发现有两种代理方式，1是wifi代理，2是全局代理。

## 手机设置wifi代理

本来打算写全部的流程，但是从设置到wifi涉及到内容实在太多了，我追了两天，所以尽量列重点，忽略不重要到。

一般是设置中选择wifi长按，填入代理服务器，端口等。定位到实现代码

`packages/apps/Settings/src/com/android/settings/wifi/WifiConfigController.java`

```java
public WifiConfigController(

        WifiConfigUiBase parent, View view, AccessPoint accessPoint, boolean edit) {

        ...

        WifiInfo info = mAccessPoint.getInfo();
        if (info != null && info.getLinkSpeed() != -1) {
            addRow(group, R.string.wifi_speed, info.getLinkSpeed() + WifiInfo.LINK_SPEED_UNITS);
        }

        addRow(group, R.string.wifi_security, mAccessPoint.getSecurityString(false));

        boolean showAdvancedFields = false;
        if (mAccessPoint.networkId != INVALID_NETWORK_ID) {
            WifiConfiguration config = mAccessPoint.getConfig();
            if (config.ipAssignment == IpAssignment.STATIC) {
                mIpSettingsSpinner.setSelection(STATIC_IP);
                showAdvancedFields = true;
            } else {
                mIpSettingsSpinner.setSelection(DHCP);
            }
            //Display IP addresses
            for(InetAddress a : config.linkProperties.getAddresses()) {
                addRow(group, R.string.wifi_ip_address, a.getHostAddress());
            }

            //构造函数，从设置中取出状态，是否设置代理
            if (config.proxySettings == ProxySettings.STATIC) {
                mProxySettingsSpinner.setSelection(PROXY_STATIC);
                showAdvancedFields = true;
            } else if (config.proxySettings == ProxySettings.PAC) {
                mProxySettingsSpinner.setVisibility(View.GONE);
                TextView textView = (TextView)mView.findViewById(R.id.proxy_pac_info);
                textView.setVisibility(View.VISIBLE);
                textView.setText(context.getString(R.string.proxy_url) +
                        config.linkProperties.getHttpProxy().getPacFileUrl());
                showAdvancedFields = true;
            } else {
                mProxySettingsSpinner.setSelection(PROXY_NONE);
            }
        }

        ...
```

当设置代理的时候，显示给用户

```java
//展示代理信息
private void showProxyFields() {
    WifiConfiguration config = null;

    mView.findViewById(R.id.proxy_settings_fields).setVisibility(View.VISIBLE);

    if (mAccessPoint != null && mAccessPoint.networkId != INVALID_NETWORK_ID) {
        config = mAccessPoint.getConfig();
    }

    if (mProxySettingsSpinner.getSelectedItemPosition() == PROXY_STATIC) {
        mView.findViewById(R.id.proxy_warning_limited_support).setVisibility(View.VISIBLE);
        mView.findViewById(R.id.proxy_fields).setVisibility(View.VISIBLE);
        if (mProxyHostView == null) {
            mProxyHostView = (TextView) mView.findViewById(R.id.proxy_hostname);
            mProxyHostView.addTextChangedListener(this);
            mProxyPortView = (TextView) mView.findViewById(R.id.proxy_port);
            mProxyPortView.addTextChangedListener(this);
            mProxyExclusionListView = (TextView) mView.findViewById(R.id.proxy_exclusionlist);
            mProxyExclusionListView.addTextChangedListener(this);
        }
        if (config != null) {
            //获取设置过的代理信息
            ProxyProperties proxyProperties = config.linkProperties.getHttpProxy();
            if (proxyProperties != null) {
                mProxyHostView.setText(proxyProperties.getHost());
                mProxyPortView.setText(Integer.toString(proxyProperties.getPort()));
                mProxyExclusionListView.setText(proxyProperties.getExclusionList());
            }
        }
    } else {
        mView.findViewById(R.id.proxy_warning_limited_support).setVisibility(View.GONE);
        mView.findViewById(R.id.proxy_fields).setVisibility(View.GONE);
    }
}
```

```java
//保存代理设置
private boolean ipAndProxyFieldsAreValid() {
    mLinkProperties.clear();
    mIpAssignment = (mIpSettingsSpinner != null &&
            mIpSettingsSpinner.getSelectedItemPosition() == STATIC_IP) ?
            IpAssignment.STATIC : IpAssignment.DHCP;

    if (mIpAssignment == IpAssignment.STATIC) {
        int result = validateIpConfigFields(mLinkProperties);
        if (result != 0) {
            return false;
        }
    }

    mProxySettings = (mProxySettingsSpinner != null &&
            mProxySettingsSpinner.getSelectedItemPosition() == PROXY_STATIC) ?
            ProxySettings.STATIC : ProxySettings.NONE;

    if (mProxySettings == ProxySettings.STATIC && mProxyHostView != null) {
        String host = mProxyHostView.getText().toString();
        String portStr = mProxyPortView.getText().toString();
        String exclusionList = mProxyExclusionListView.getText().toString();
        int port = 0;
        int result = 0;
        try {
            port = Integer.parseInt(portStr);
            //验证host和port是否正确
            result = ProxySelector.validate(host, portStr, exclusionList);
        } catch (NumberFormatException e) {
            result = R.string.proxy_error_invalid_port;
        }
        if (result == 0) {
            ProxyProperties proxyProperties= new ProxyProperties(host, port, exclusionList);
            mLinkProperties.setHttpProxy(proxyProperties);
        } else {
            return false;
        }
    }
    return true;
}
```

保存和获取都是通过`LinkProperties`操作`ProxyProperties`，而`ProxyProperties`只是赋值和获取，没有其他操作。
