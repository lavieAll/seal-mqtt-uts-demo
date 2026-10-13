# seal-mqtt-uts

`seal-mqtt-uts` 是一个跨平台 MQTT 客户端插件，支持 **鸿蒙 Next、Android、iOS、微信小程序、支付宝小程序和 Web**。插件提供连接、订阅、发布、断线重连、遗嘱消息、TLS 和 MQTT 5.0 属性等能力。业务代码只需要创建客户端并调用统一 API，即可在支持的平台上使用 MQTT。

👏👏👏欢迎W（BJGFCYY）或Q（2480621579）咨询。

## 快速开始

### 1. 安装插件

将插件安装到项目后，在业务代码中从插件根路径导入：

```ts
import {
  createMqttClient,
  getMqttCapabilities
} from '@/uni_modules/seal-mqtt-uts'
```

请使用与当前项目匹配的插件版本，并确保设备或浏览器可以访问 MQTT 服务端。

### 2. 创建客户端并监听事件

一个客户端实例对应一个 MQTT 连接。建议先注册事件，再调用 `connect`：

```ts
const client = createMqttClient()

client.onConnect((res) => {
  console.log('MQTT 已连接', res)
})

client.onMessage((message) => {
  console.log('收到消息', message.topic, message.payload)
})

client.onReconnecting((info) => {
  console.log('准备重连', info.attempt, info.delay)
})

client.onDisconnect((info) => {
  console.log('连接断开', info.reason, info.willReconnect)
})

client.onError((error) => {
  console.error('MQTT 错误', error.errCode, error.errMsg)
})
```

### 3. 连接、订阅和发布

```ts
client.connect({
  // App 可使用 mqtt/mqtts/ws/wss；Web 和小程序使用 ws/wss
  url: 'wss://mqtt.example.com:8084/mqtt',
  clientId: 'device-001',
  username: 'user',
  password: 'password',
  protocolVersion: 4, // 4 = MQTT 3.1.1；5 = MQTT 5.0
  clean: true,
  keepalive: 60,
  success: () => {
    client.subscribe({
      subscriptions: [{ topic: 'device/device-001/command', qos: 1 }],
      success: (res) => console.log('订阅成功', res.grantedQos),
      fail: (error) => console.error('订阅失败', error.errMsg)
    })

    client.publish({
      topic: 'device/device-001/status',
      payload: JSON.stringify({ online: true }),
      qos: 1,
      retain: true,
      success: () => console.log('发布成功'),
      fail: (error) => console.error('发布失败', error.errMsg)
    })
  },
  fail: (error) => console.error('连接失败', error.errCode, error.errMsg)
})
```

页面或业务模块销毁时，主动释放客户端：

```ts
client.disconnect()
client.destroy()
```

## 支持的平台和地址格式

| 平台 | 支持的地址 | MQTT 版本 | 自定义 CA / 客户端证书 |
| --- | --- | --- | --- |
| Android App | `mqtt://`、`mqtts://`、`ws://`、`wss://` | 3.1.1、5.0 | 支持 |
| iOS App | `mqtt://`、`mqtts://`、`ws://`、`wss://` | 3.1.1、5.0 | 支持 |
| HarmonyOS App | `mqtt://`、`mqtts://`、`ws://`、`wss://` | 3.1.1、5.0 | 支持 |
| Web | `ws://`、`wss://` | 3.1.1 | 由浏览器管理 |
| 微信小程序 | `wss://`（也支持 `ws://`，以平台网络规则为准） | 3.1.1 | 由平台管理 |
| 支付宝小程序 | `wss://`（也支持 `ws://`，以平台网络规则为准） | 3.1.1 | 由平台管理 |

Web 和小程序不能使用 `mqtt://` 或 `mqtts://`。Web 页面使用 HTTPS 时，应使用 `wss://`，否则可能被浏览器拦截混合内容请求。WebSocket 地址通常需要包含服务端配置的路径，例如 `/mqtt`。

可在运行时读取当前平台能力：

```ts
const capabilities = getMqttCapabilities()
console.log(capabilities.schemes) // 当前平台支持的地址协议
console.log(capabilities.mqtt5)   // 是否支持 MQTT 5.0
```

## 客户端 API

### `createMqttClient()`

创建并返回一个客户端实例。每个实例独立维护一条连接和自己的订阅列表。

### `connect(options)`

建立连接。只有客户端处于 `idle` 或 `closed` 状态时才能再次连接。

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `url` | `string` | 必填 | MQTT 地址，格式见上表 |
| `clientId` | `string` | 自动生成 | 客户端标识；需要持久会话时请固定填写 |
| `username` | `string` | 无 | 用户名 |
| `password` | `string` | 无 | 密码 |
| `protocolVersion` | `4 \| 5` | `4` | `4` 表示 MQTT 3.1.1，`5` 表示 MQTT 5.0 |
| `clean` | `boolean` | `true` | 是否使用清理会话 |
| `keepalive` | `number` | `60` | 心跳间隔，单位为秒；`0` 表示关闭心跳 |
| `connectTimeout` | `number` | `10000` | 连接超时时间，单位为毫秒 |
| `will` | `MqttWill` | 无 | 异常断开时由服务端发布的遗嘱消息 |
| `tls` | `MqttTlsOptions` | 无 | App 端的 TLS 和证书配置 |
| `reconnect` | `MqttReconnectOptions` | 开启 | 断线重连配置 |
| `properties` | `MqttProperties` | 无 | MQTT 5.0 连接属性 |
| `success` | function | 无 | 首次连接成功回调 |
| `fail` | function | 无 | 首次连接失败回调 |
| `complete` | function | 无 | 首次连接完成回调 |

`success`、`fail` 和 `complete` 是本次 `connect` 操作的回调；`onConnect`、`onDisconnect` 和 `onError` 是持续事件，适合处理后续重连和网络变化。

### `subscribe(options)`

订阅一个或多个主题。必须在 `connected` 状态调用。

```ts
client.subscribe({
  subscriptions: [
    { topic: 'sensor/+/temperature', qos: 1 },
    { topic: 'device/001/#', qos: 0 }
  ],
  success: (res) => console.log('服务端授予的 QoS', res.grantedQos),
  fail: (error) => console.error(error.errCode, error.errMsg)
})
```

订阅主题支持 MQTT 通配符：`+` 匹配一级，`#` 必须位于最后一级。`noLocal`、`retainAsPublished` 和 `retainHandling` 仅在 MQTT 5.0 下有意义。

`subscriptions` 中每一项支持以下字段：

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `topic` | `string` | 必填 | 订阅主题，可使用通配符 |
| `qos` | `0 \| 1 \| 2` | `0` | 最大接收 QoS |
| `noLocal` | `boolean` | `false` | MQTT 5.0：不接收本客户端发布的消息 |
| `retainAsPublished` | `boolean` | `false` | MQTT 5.0：保留服务端转发时的 retain 标志 |
| `retainHandling` | `0 \| 1 \| 2` | `0` | MQTT 5.0：控制建立订阅时是否发送保留消息 |

`subscribe` 还支持 `properties`、`success`、`fail` 和 `complete`。`success` 参数中的 `grantedQos` 与传入的订阅顺序对应。

### `unsubscribe(options)`

取消订阅：

```ts
client.unsubscribe({
  topics: ['sensor/+/temperature'],
  success: () => console.log('取消订阅成功'),
  fail: (error) => console.error(error.errMsg)
})
```

`topics` 为要取消的主题数组；`properties`、`success`、`fail` 和 `complete` 可选。取消订阅的主题不允许为空。

### `publish(options)`

发布消息。发布主题不能为空，也不能包含 `+` 或 `#`。

```ts
client.publish({
  topic: 'device/001/command',
  payload: JSON.stringify({ action: 'restart' }),
  qos: 1,
  retain: false,
  success: () => console.log('发布成功'),
  fail: (error) => console.error(error.errCode, error.errMsg)
})
```

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `topic` | `string` | 必填 | 发布主题，不支持 `+` 和 `#` |
| `payload` | `string` | 空字符串 | UTF-8 文本消息 |
| `payloadBase64` | `string` | 无 | Base64 编码的二进制消息 |
| `qos` | `0 \| 1 \| 2` | `0` | 消息 QoS |
| `retain` | `boolean` | `false` | 是否保留消息 |
| `properties` | `MqttProperties` | 无 | MQTT 5.0 发布属性 |
| `success` / `fail` / `complete` | function | 无 | 本次发布操作的回调 |

文本和二进制消息分别使用 `payload` 与 `payloadBase64`，两者应二选一。接收消息时，`payload` 是 UTF-8 文本，`payloadBase64` 始终提供，可用于无损处理二进制内容：

```ts
client.publish({
  topic: 'device/001/binary',
  payloadBase64: 'AAECAwQ=',
  qos: 0
})

client.onMessage((message) => {
  console.log(message.payload)         // UTF-8 解码结果
  console.log(message.payloadBase64)   // 原始字节的 Base64
})
```

### `getState()`

返回当前状态：

`idle`、`connecting`、`connected`、`reconnecting`、`disconnecting`、`closed`。

### `disconnect(options?)`

主动断开连接。主动断开不会触发自动重连。`force: true` 会立即关闭连接；默认会等待正在处理的操作。

```ts
client.disconnect({
  reasonCode: 0,
  success: () => console.log('已断开')
})
```

`reasonCode` 是 MQTT 5.0 断开原因码，默认值为 `0`；`force` 默认为 `false`，设为 `true` 时立即关闭连接。两者在使用 MQTT 3.1.1 时会被忽略或按平台规则处理。

### `destroy()`

释放客户端、取消重连、清除订阅和事件回调。释放后不要继续使用该实例；需要重新连接时请创建新的客户端。

## 事件回调

| 方法 | 参数 | 触发时机 |
| --- | --- | --- |
| `onConnect(callback)` | `{ sessionPresent, protocolVersion, properties? }` | 连接成功，包含重连成功 |
| `onMessage(callback)` | `{ topic, payload, payloadBase64, qos, retain, dup, properties? }` | 收到消息 |
| `onReconnecting(callback)` | `{ attempt, delay }` | 即将进行自动重连 |
| `onDisconnect(callback)` | `{ reasonCode?, reason, willReconnect }` | 连接关闭或主动断开 |
| `onError(callback)` | `{ errCode, errMsg, reasonCode? }` | 连接、订阅、发布或重连过程发生错误 |

消息回调只注册一个处理函数；再次调用同一 `onXxx` 方法会替换之前的处理函数。

## 断线重连

默认开启自动重连，默认参数如下：

```ts
reconnect: {
  enable: true,
  minDelay: 1000,
  maxDelay: 30000,
  factor: 2,
  jitter: 0.3,
  maxAttempts: 0 // 0 表示不限制次数
}
```

可以按业务需要关闭或限制重连次数：

```ts
client.connect({
  url: 'mqtts://mqtt.example.com:8883',
  reconnect: {
    enable: true,
    minDelay: 2000,
    maxDelay: 15000,
    maxAttempts: 5
  }
})
```

重连期间客户端不是 `connected` 状态，此时调用 `subscribe`、`unsubscribe` 或 `publish` 会通过 `fail` 返回 `9010007`，插件不会替业务缓存离线消息。连接恢复后，客户端会按服务端会话情况恢复之前成功订阅的主题；需要可靠恢复时建议使用固定 `clientId`，并根据业务设置 `clean: false`。

## TLS、遗嘱和 MQTT 5.0

### TLS

App 端可以使用系统证书，也可以传入 PEM 格式的 CA 或客户端证书：

```ts
client.connect({
  url: 'mqtts://mqtt.example.com:8883',
  tls: {
    ca: caPem,
    cert: clientCertPem,
    key: clientKeyPem,
    rejectUnauthorized: true,
    serverName: 'mqtt.example.com'
  }
})
```

`cert` 与 `key` 必须成对提供。`rejectUnauthorized: false` 仅适合临时调试，生产环境应保持默认的证书校验。Web 和小程序的证书校验由平台负责，不能通过 `tls` 配置自定义 CA 或客户端证书。

### 遗嘱消息

遗嘱由服务端在客户端异常断开时发布，正常调用 `disconnect` 不会触发：

```ts
client.connect({
  url: 'mqtts://mqtt.example.com:8883',
  will: {
    topic: 'device/001/status',
    payload: JSON.stringify({ online: false }),
    qos: 1,
    retain: true
  }
})
```

### MQTT 5.0 属性

App 端支持 `protocolVersion: 5`。常用属性包括 `sessionExpiryInterval`、`receiveMaximum`、`maximumPacketSize`、`contentType`、`responseTopic`、`correlationDataBase64`、`messageExpiryInterval` 和 `userProperties`。服务端不支持 MQTT 5.0 时，请改用 `protocolVersion: 4`。

## 错误处理

错误对象包含 `errCode`、`errMsg`，部分服务端或协议错误还会包含 `reasonCode`。

| 错误码 | 含义 | 常见处理 |
| --- | --- | --- |
| `9010001` | 参数错误，例如 URL、topic、QoS 或 Base64 不合法 | 修正参数后重新调用 |
| `9010002` | 当前平台不支持所使用的地址协议或能力 | 使用当前平台支持的协议 |
| `9010003` | 连接失败，例如网络不可达、域名解析失败或端口被拒绝 | 检查网络、地址和服务端状态 |
| `9010004` | 连接超时 | 检查网络和服务端响应时间 |
| `9010005` | 服务端认证或连接拒绝 | 检查用户名、密码、权限和 `reasonCode` |
| `9010006` | TLS 握手或证书校验失败 | 检查证书链、域名、有效期和私钥 |
| `9010007` | 未连接时调用了发布、订阅或取消订阅 | 等待 `onConnect` 后再操作 |
| `9010008` | 订阅或取消订阅失败 | 检查主题、权限和服务端返回值 |
| `9010009` | 发布失败 | 检查主题、QoS、权限和连接状态 |
| `9010010` | 其他操作错误 | 结合 `errMsg` 排查 |

示例：

```ts
client.onError((error) => {
  if (error.errCode === 9010005) {
    console.error('认证失败，请检查账号密码')
    return
  }
  console.error('MQTT 错误：', error.errCode, error.errMsg, error.reasonCode)
})
```

## 完整页面示例

下面的示例演示一个简单的连接、订阅、收消息和发布流程。将 `url`、账号和主题替换为自己的服务端配置即可：

```ts
import { createMqttClient } from '@/uni_modules/seal-mqtt-uts'

const client = createMqttClient()
const topic = 'demo/seal-mqtt-uts'

client.onConnect(() => {
  console.log('连接成功')

  client.subscribe({
    subscriptions: [{ topic, qos: 1 }],
    success: () => {
      client.publish({
        topic,
        payload: JSON.stringify({ text: 'hello MQTT' }),
        qos: 1
      })
    },
    fail: (error) => console.error('订阅失败', error.errMsg)
  })
})

client.onMessage((message) => {
  try {
    const data = JSON.parse(message.payload)
    console.log('收到 JSON', data)
  } catch (_error) {
    console.log('收到文本', message.payload)
  }
})

client.onError((error) => {
  console.error(error.errCode, error.errMsg)
})

client.connect({
  url: 'wss://mqtt.example.com:8084/mqtt',
  clientId: 'demo-client-001',
  username: 'demo-user',
  password: 'demo-password',
  protocolVersion: 4,
  reconnect: { enable: true }
})

// 页面离开或业务结束时调用
function closeMqtt(): void {
  client.disconnect()
  client.destroy()
}
```

## 使用注意事项

1. 发布和订阅必须等待连接成功；不要只根据 `connect` 方法调用返回判断连接结果。
2. 每个客户端实例只建议对应一条连接；需要连接多个服务端时创建多个实例，并分别管理生命周期。
3. 小程序和 Web 平台还需要在对应平台配置 WebSocket 合法域名、HTTPS 证书和网络权限。
4. 生产环境建议使用 `mqtts://` 或 `wss://`，并妥善保护用户名、密码和证书内容。
5. 不要把 `clientId` 同时分配给多个在线客户端，否则服务端可能让旧连接下线。

