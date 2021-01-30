# coinbene-spot-rest 现货openapi rest接口说明

[TOC]

## 基本信息

- 本篇列出REST接口的baseurl: https://openapi-exchange.coinbene.com 
- 建议创建完API后，修改添加上自己服务器出口IP，进一步增强API安全性校验
- 需要科学上网，国内用户建议机器绑定host，104.16.127.19 openapi-exchange.coinbene.com
- 所有接口的响应都是JSON格式
- 所有时间、时间戳均为UNIX时间，单位为毫秒
- HTTP 4XX 错误码用于指示错误的请求内容、行为、格式。
- HTTP 429 错误码表示警告访问频次超限，即将被封IP
- HTTP 5XX 错误码用于指示Coinbene服务侧的问题。
- HTTP 504 表示API服务端已经向业务核心提交了请求但未能获取响应，特别需要注意的是504代码不代表请求失败，而是未知。很可能已经得到了执行，也有可能执行失败，需要做进一步确认。
- 每个接口都有可能抛出异常，异常响应格式如下：

```
{
  "code": 10001,
  "msg": "Invalid Paramater."
}
```

- 具体的错误码及其解释在错误代码汇总。
- GET方法的接口, 参数必须在query string中发送。
- POST方法的接口, 参数在 request body中发送(content type application/json)。
- 对参数的顺序不做要求。

## 访问限制

- 当访问接口超过频率限制时，将返回429状态：请求太频繁。
- 限制规则，如果传入有效的API key则用user id限速；如果没有则拿用户的公网IP限速。各个接口上有单独的说明。

## 接口类型

- 主要为两类接口，公共接口和私有接口。
- 公共接口无需认证即可调用。
- 私有接口用户订单和账户。每个私有请求必须使用规范的验证形式进行签名。私有接口需要使用您的API key进行验证。

## 签名方式

所有接口请求头必须包含以下内容：

- ACCESS-KEY 字符串类型的API key
- ACCESS-SIGN 使用Hex生成字符串返回，具体代码参考下面Java版本和Python版本代码
- ACCESS-TIMESTAMP 发起请求的时间戳
- 所有请求都应该含有application/json类型内容，并且是有效的JSON。

ACCESS-SIGN的值生成规则：

- 按照timestamp + method + requestPath + body字符串（+表示字符串连接），以及secret，使用HMAC SHA256方法加密，最后把加密串的字节数组转成十六进制字符串返回。
- 其中，timestamp的值与ACCESS-TIMESTAMP请求头相同，必须是UTC时区Unix时间戳的十进制秒数或ISO8601标准的时间格式，精确到毫秒。
- Method是请求方法，字母全部大写：GET/POST
- requestPath是请求接口路径，例如：/api/exchange/v2/market/orderBook
- body是指请求主体的字符串。GET请求没有body信息可省略；POST请求有body信息JSON串，例如{"symbol":"BTCUSDT","order_id":"7440"}
- secret为用户申请API时所生成的。

接口请求样例：

- GET协议接口两种情况: 

```
1. 不带参数：
签名串preHash: 2019-06-13T11:18:29.009ZGET/api/exchange/v2/market/tradePair/list
2. 带参数：
签名串preHash: 2019-06-13T11:18:29.009ZGET/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT
```


```
Url: http://127.0.0.1:8604/api/exchange/v2/market/tradePair/list
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: c5c7487225df0ee6ce333603f0cc2410870ba904641b068be74fec0ac36bf2ee
	ACCESS-TIMESTAMP: 2019-06-13T11:14:05.591Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-13T11:14:05.591ZGET/api/exchange/v2/market/tradePair/list


```


```
Url: http://127.0.0.1:8604/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 890f16e5dd3a50c2b6d19a15c5d303fca0f40470824ddae2ecbefd9cb3c49712
	ACCESS-TIMESTAMP: 2019-06-13T11:18:29.009Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-13T11:18:29.009ZGET/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT

```


- POST协议接口情况：

```
签名串preHash: 2019-06-13T11:27:14.493ZPOST/api/exchange/v2/order/place{"symbol":"BTC/USDT","price":"67.3","quantity":"38","direction":"1","clientId":"1560425234454"}
```


```
Url: http://127.0.0.1:8604/api/exchange/v2/order/place
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 237dbda384965cf78284bdb028ab05229343e635e46adca6323833ade6d8704a
	ACCESS-TIMESTAMP: 2019-06-13T11:27:14.493Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"symbol":"BTC/USDT","price":"67.3","quantity":"38","direction":"1","clientId":"1560425234454"}
preHash: 2019-06-13T11:27:14.493ZPOST/api/exchange/v2/order/place{"symbol":"BTC/USDT","price":"67.3","quantity":"38","direction":"1","clientId":"1560425234454"}

```

- 签名算法验证：


```
源串：2019-05-25T03:20:30.362ZGET/api/spot/v2/account/info
secret：9daf13ebd76c4f358fc885ca6ede5e27
生成sign串：a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2

样例代码（Java版本）：
**
   * 生成签名
   *
   * @param timeStamp   时间戳
   * @param method      请求方法：POST或者GET
   * @param requestUrl  url
   * @param requestBody 请求内容，没有传null
   * @param secret      密钥
   */
  private String signForContractOpenApi(String timeStamp, String method, String requestUrl, String requestBody, String secret) {
    String shaResource = timeStamp + method + requestUrl + (requestBody == null ? "" : requestBody);
    System.out.println(shaResource);
    String signStr = sha256_HMAC(shaResource, secret);
    return signStr;
  }

  /**
   * sha256_HMAC加密
   *
   * @param resource 签名源字符串
   * @param secret   秘钥
   * @return 加密后字符串
   */
  private static String sha256_HMAC(String resource, String secret) {
    String hash = "";
    try {
      Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
      SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
      sha256_HMAC.init(secret_key);
      byte[] bytes = sha256_HMAC.doFinal(resource.getBytes("UTF-8"));
      hash = byteArrayToHexString(bytes);
    } catch (Exception e) {
      System.out.println("Error HmacSHA256 ===========" + e.getMessage());
    }
    return hash;
  }

  /**
   * 将加密后的字节数组转换成字符串
   *
   * @param bytes 字节数组
   * @return 字符串
   */
  private static String byteArrayToHexString(byte[] bytes) {
    StringBuffer buffer = new StringBuffer();
    String stmp;
    for (int index = 0; bytes != null && index < bytes.length; index++) {
      stmp = Integer.toHexString(bytes[index] & 0XFF);
      if (stmp.length() == 1) {
        buffer.append('0');
      }
      buffer.append(stmp);
    }
    return buffer.toString().toLowerCase();
  }

样例代码（Python版本）：

import hashlib
import hmac
import unittest

def sign(message, secret):
    """
    gen sign
    :param message: message wait sign
    :param secret:  secret key
    :return:
    """
    secret = secret.encode('utf-8')
    message = message.encode('utf-8')
    sign = hmac.new(secret, message, digestmod=hashlib.sha256).hexdigest()
    return sign

class TestUtil(unittest.TestCase):
    def test_sign(self):
        sn = sign("2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
        self.assertEqual(sn, "a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2")


```

## 接口规范

### 公共接口-获取全部交易配置信息

```
获取交易所全部币对配置列表
限速规则：2次/1秒
HTTP GET /api/exchange/v2/market/tradePair/list
```

请求参数：
无

返回字段说明：

| 名称             | 类型   | 说明                 |
| ---------------- | ------ | -------------------- |
| symbol           | string | 币对名称, 如BTC/USDT |
| baseAsset        | string | 交易货币  BTC        |
| quoteAsset       | string | 计价货币 USDT        |
| pricePrecision   | string | 价格精度             |
| amountPrecision  | string | 数量精度             |
| takerFeeRate     | string | taker手续费率        |
| makerFeeRate     | string | maker手续费率        |
| minAmount        | string | 委托数量最小限制     |
| priceFluctuation | string | 价格波动限制         |
| site             | string | 所属站点             |

```
Request:
Url: http://127.0.0.1:8604/api/exchange/v2/market/tradePair/list
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: c5c7487225df0ee6ce333603f0cc2410870ba904641b068be74fec0ac36bf2ee
	ACCESS-TIMESTAMP: 2019-06-13T11:14:05.591Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-13T11:14:05.591ZGET/api/exchange/v2/market/tradePair/list


Response:
{
  "code": 200, 
  "data": [
    {
      "symbol": "ABBC/BTC", 
      "baseAsset": "ABBC", 
      "quoteAsset": "BTC", 
      "pricePrecision": "8", 
      "amountPrecision": "2", 
      "takerFeeRate": "0.0010", 
      "makerFeeRate": "0.0010", 
      "minAmount": "1.00000000", 
      "site":"MAIN",
      "priceFluctuation": "0.50"
    }, 
    {
      "symbol": "ABT/ETH", 
      "baseAsset": "ABT", 
      "quoteAsset": "ETH", 
      "pricePrecision": "6", 
      "amountPrecision": "2", 
      "takerFeeRate": "0.0010", 
      "makerFeeRate": "0.0010", 
      "minAmount": "1.00000000",
      "site":"MAIN",
      "priceFluctuation": "0.50"
    }, 
    {
      "symbol": "ABT/USDT", 
      "baseAsset": "ABT", 
      "quoteAsset": "USDT", 
      "pricePrecision": "6", 
      "amountPrecision": "2", 
      "takerFeeRate": "0.0010", 
      "makerFeeRate": "0.0010", 
      "minAmount": "1.00000000", 
      "site":"MAIN",
      "priceFluctuation": "0.50"
    }, 
    {
      "symbol": "ABYSS/ETH", 
      "baseAsset": "ABYSS", 
      "quoteAsset": "ETH", 
      "pricePrecision": "7", 
      "amountPrecision": "2", 
      "takerFeeRate": "0.0010", 
      "makerFeeRate": "0.0010", 
      "minAmount": "1.00000000", 
      "site":"MAIN",
      "priceFluctuation": "0.50"
    }
  ]
}
```


### 公共接口-获取指定交易对配置信息

```
获取指定交易对配置信息
限速规则：3次/1秒
HTTP GET /api/exchange/v2/market/tradePair/one
```

请求参数：

| 名称   | 类型   | 是否必填 | 说明                 |
| ------ | ------ | -------- | -------------------- |
| symbol | string | 是       | 币对名称，如BTC/USDT |

返回字段说明：

| 名称             | 类型   | 说明                |
| ---------------- | ------ | ------------------- |
| symbol           | string | 币对名称,如BTC/USDT |
| baseAsset        | string | 交易货币 BTC        |
| quoteAsset       | string | 计价货币 USDT       |
| pricePrecision   | string | 价格精度            |
| amountPrecision  | string | 数量精度            |
| takerFeeRate     | string | taker手续费率       |
| makerFeeRate     | string | maker手续费率       |
| minAmount        | string | 最新成交数量        |
| priceFluctuation | string | 价格波动限制        |
| site             | string | 所属站点            |

```
Request:
Url: http://127.0.0.1:8604/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 890f16e5dd3a50c2b6d19a15c5d303fca0f40470824ddae2ecbefd9cb3c49712
	ACCESS-TIMESTAMP: 2019-06-13T11:18:29.009Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-13T11:18:29.009ZGET/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT


Response:
{
  "code": 200, 
  "data": {
    "symbol": "BTC/USDT", 
    "baseAsset": "BTC", 
    "quoteAsset": "USDT", 
    "pricePrecision": "2", 
    "amountPrecision": "4", 
    "takerFeeRate": "0.0010", 
    "makerFeeRate": "0.0010", 
    "minAmount": "0.00010000", 
    "site":"MAIN",
    "priceFluctuation": "0.50"
  }
}
```

### 公共接口-获取深度

```
获取交易所现货深度列表
限速规则：6次/1秒
HTTP GET /api/exchange/v2/market/orderBook
```

请求参数：

| 名称   | 类型   | 是否必填 | 说明                         |
| ------ | ------ | -------- | ---------------------------- |
| symbol | string | 是       | 币对名称，如BTC/USDT         |
| depth  | string | 是       | 深度档位，值有5、10、50、100 |

返回字段说明：

| 名称 | 类型  | 说明                       |
| ---- | ----- | -------------------------- |
| asks | array | 卖方深度，[档位价格，数量] |
| bids | array | 买方深度，[档位价格，数量] |


```
Request:
Url: http://127.0.0.1:8604/api/exchange/v2/market/orderBook?symbol=BTC%2FUSDT&depth=5
Method: GET

Url: http://127.0.0.1:8604/api/exchange/v2/market/orderBook?symbol=BTC%2FUSDT&depth=5
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: df00bfce19d307d3ba46926be7a3f89b
	ACCESS-SIGN: fccfc92ebb53e325c388cfa14ab02adc540321841c6086a742b7e6646fe540e7
	ACCESS-TIMESTAMP: 2019-06-24T02:58:18.648Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-24T02:58:18.648ZGET/api/exchange/v2/market/orderBook?symbol=BTC%2FUSDT&depth=5


Response:
{
  "code": 200, 
  "data": {
        "asks":[
            [
                "10900.75",
                "0.0150"
            ],
            [
                "10908.61",
                "0.0031"
            ],
            [
                "10909.27",
                "0.0021"
            ],
            [
                "10909.3",
                "0.0021"
            ],
            [
                "10914.15",
                "0.0461"
            ]
        ],
        "bids":[
            [
                "10869.41",
                "0.0167"
            ],
            [
                "10869.4",
                "0.0300"
            ],
            [
                "10868.2",
                "0.0100"
            ],
            [
                "10861.83",
                "0.0019"
            ],
            [
                "10861.82",
                "0.0021"
            ]
        ]
    }
}






```


### 公共接口-获取指定的ticker信息

```
获取交易所现货指定ticker的最新成交价、买一价、卖一价和24交易量
限速规则：6次/1秒
HTTP GET /api/exchange/v2/market/ticker/one
```

请求参数：无

| 名称   | 类型   | 是否必填 | 说明                 |
| ------ | ------ | -------- | -------------------- |
| symbol | string | 是       | 币对名称，如BTC/USDT |

返回字段说明：

| 名称        | 类型   | 说明                 |
| ----------- | ------ | -------------------- |
| symbol      | string | 币对名称，如BTC/USDT |
| latestPrice | string | 最新价               |
| bestAsk     | string | 卖一价               |
| bestBid     | string | 买一价               |
| high24h     | string | 24h最高价            |
| low24h      | string | 24h最低价            |
| volume24h   | string | 24h成交量            |

```
Request:
Url: http://127.0.0.1:8604/api/exchange/v2/market/ticker/one?symbol=BTC%2FUSDT
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 92d75e564278da59c9dc1a48ec141ea2b12041f23c58fcd2910d39684f037e46
	ACCESS-TIMESTAMP: 2019-06-18T08:03:39.250Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-18T08:03:39.250ZGET/api/exchange/v2/market/ticker/one?symbol=BTC%2FUSDT


Response:
{
  "code": 200, 
  "data": {
        "symbol":"BTC/USDT",
        "latestPrice":"63.17",
        "bestBid":"63.15",
        "bestAsk":"63.17",
        "high24h":"63.17",
        "low24h":"63.17",
        "volume24h":"1906.215235"
    }
}
```


### 公共接口-查询最新成交信息

```
获取交易所现货的最新成交信息
限速规则：3次/1秒
HTTP GET /api/exchange/v2/market/trades
```

请求参数：

| 名称   | 类型   | 是否必填 | 说明                 |
| ------ | ------ | -------- | -------------------- |
| symbol | string | 是       | 币对名称，如BTC/USDT |

返回字段说明：

| 名称      | 类型   | 说明     |
| --------- | ------ | -------- |
| symbol    | string | 币对名称 |
| price     | string | 成交价格 |
| volume    | string | 成交数量 |
| direction | string | 方向     |
| tradeTime | string | 成交时间 |

```
说明：
1.只返回最新50条交易数据
2.[symbol|price|volume|direction|tradeTime]
```

```
Request:
Url: http://127.0.0.1:8604/api/exchange/v2/market/trades
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 7bee3f8dc457479c0406e5598b5ce8e7bdf348fcb815bb9aa90c7d6b4e93f276
	ACCESS-TIMESTAMP: 2019-06-13T11:05:39.797Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-13T11:05:39.797ZGET/api/exchange/v2/market/trades


Response:
{
  "code": 200, 
  "data": [
        [
            "BTC/USDT",
            "10639.97",
            "0.002",
            "buy",
            "2019-06-24T02:58:17.368Z"
        ],
        [
            "BTC/USDT",
            "10639.99",
            "0.0021",
            "buy",
            "2019-06-24T02:58:17.368Z"
        ],
        [
            "BTC/USDT",
            "10639.99",
            "0.0021",
            "buy",
            "2019-06-24T02:58:17.368Z"
        ],
        [
            "BTC/USDT",
            "10639.99",
            "0.0019",
            "buy",
            "2019-06-24T02:58:17.368Z"
        ],
        [
            "BTC/USDT",
            "10636.01",
            "0.0041",
            "buy",
            "2019-06-24T02:58:17.369Z"
        ],
        [
            "BTC/USDT",
            "10636.01",
            "0.038",
            "buy",
            "2019-06-24T02:58:17.369Z"
        ],
        [
            "BTC/USDT",
            "10636.03",
            "0.0021",
            "buy",
            "2019-06-24T02:58:17.369Z"
        ],
        [
            "BTC/USDT",
            "10636.03",
            "0.0193",
            "buy",
            "2019-06-24T02:58:17.369Z"
        ],
        [
            "BTC/USDT",
            "10636.06",
            "0.0021",
            "buy",
            "2019-06-24T02:58:17.370Z"
        ]
    ]
}

```



