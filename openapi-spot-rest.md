
# coinbene-spot-rest 现货openapi rest接口说明

[英文版本](https://github.com/Coinbene/API-SPOT-v2-Documents/blob/master/openapi-spot-rest-en.md)

   * [coinbene-spot-rest 现货openapi rest接口说明](#coinbene-spot-rest-现货openapi-rest接口说明)
      * [基本信息](#基本信息)
      * [访问限制](#访问限制)
      * [接口类型](#接口类型)
      * [签名方式](#签名方式)
      * [接口规范](#接口规范)
         * [公共接口-获取全部交易配置信息](#公共接口-获取全部交易配置信息)
         * [公共接口-获取指定交易对配置信息](#公共接口-获取指定交易对配置信息)
         * [公共接口-获取深度](#公共接口-获取深度)
         * [公共接口-获取全部的ticker信息](#公共接口-获取全部的ticker信息)
         * [公共接口-获取指定的ticker信息](#公共接口-获取指定的ticker信息)
         * [公共接口-查询最新成交信息](#公共接口-查询最新成交信息)
         * [公共接口-获取Kline数据信息](#公共接口-获取kline数据信息)
         * [公共接口-获取币的汇率](#公共接口-获取币的汇率)
         * [私有接口-查询全部账户信息](#私有接口-查询全部账户信息)
         * [私有接口-查询指定账户资产信息](#私有接口-查询指定账户资产信息)
         * [私有接口-下单](#私有接口-下单)
         * [私有接口-批量下单](#私有接口-批量下单)
         * [私有接口-查询当前委托挂单列表](#私有接口-查询当前委托挂单列表)
         * [私有接口-查询历史委托单列表](#私有接口-查询历史委托单列表)
         * [私有接口-查询指定订单信息](#私有接口-查询指定订单信息)
         * [私有接口-查询订单成交明细列表](#私有接口-查询订单成交明细列表)
         * [私有接口-撤销指定委托单](#私有接口-撤销指定委托单)
         * [私有接口-批量撤销委托单](#私有接口-批量撤销委托单)
      * [错误代码汇总](#错误代码汇总)


## 基本信息
- 本篇列出REST接口的baseurl: http://openapi-exchange.coinbene.com 或 https://openapi-exchange.coinbene.com
- 建议创建完API后，修改添加上自己服务器出口IP，进一步增强API安全性校验
- 需要科学上网，国内用户建议机器绑定host，104.16.127.19 openapi-exchange.coinbene.com
- 所有接口的响应都是JSON格式
- 所有时间、时间戳均为UNIX时间，单位为毫秒
- HTTP 4XX 错误码用于指示错误的请求内容、行为、格式。
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
Url: http://域名/api/exchange/v2/market/tradePair/list
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
Url: http://域名/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT
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
生成sign串：da496b6d235c6bb76015eb660c3457958d2b26637a28937cd58d4329d588fb05

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

名称   | 类型  | 说明
---|---|---
symbol   | string | 币对名称, 如BTC/USDT
baseAsset   | string | 交易货币  BTC
quoteAsset   | string | 计价货币 USDT
pricePrecision   | string | 价格精度
amountPrecision   | string | 数量精度
takerFeeRate   | string | taker手续费率
makerFeeRate   | string | maker手续费率
minAmount   | string | 委托数量最小限制
priceFluctuation   | string | 价格波动限制
site   | string | 所属站点

```
Request:
Url: http://域名/api/exchange/v2/market/tradePair/list
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

名称   | 类型  |是否必填  | 说明
---|---|---|---
symbol   | string |是  | 币对名称，如BTC/USDT

返回字段说明：

名称   | 类型  | 说明
---|---|---
symbol   | string | 币对名称,如BTC/USDT
baseAsset   | string |  交易货币 BTC
quoteAsset   | string | 计价货币 USDT
pricePrecision   | string | 价格精度
amountPrecision   | string | 数量精度
takerFeeRate   | string | taker手续费率
makerFeeRate   | string | maker手续费率
minAmount   | string | 委托数量最小限制
priceFluctuation   | string | 价格波动限制
site   | string | 所属站点

```
Request:
Url: http://域名/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT
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

名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol | string | 是 | 币对名称，如BTC/USDT
depth   | string | 是 | 深度档位，值有5、10、50、100

返回字段说明：

名称   | 类型  | 说明
---|---|---
asks   | array | 卖方深度，[档位价格，数量]
bids   | array | 买方深度，[档位价格，数量]


```
Request:
Url: http://域名/api/exchange/v2/market/orderBook?symbol=BTC%2FUSDT&depth=5
Method: GET

Url: http://172.20.22.50:8604/api/exchange/v2/market/orderBook?symbol=BTC%2FUSDT&depth=5
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




Request:
Url: http://域名/api/exchange/v2/market/instruments/candles?symbol=BCH%2FUSDT&period=1
Method: GET
Headers: 
Accept: application/json
ACCESS-KEY: 2c8b514c28b6404f0d0333b958379484
ACCESS-SIGN: 11178e61aa0f571a1a70a51ea63ac38dd013200e5b069c6c6a535a40a161a9a0
ACCESS-TIMESTAMP: 2019-12-16T09:39:57.086Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2019-12-16T09:39:57.086ZGET/api/exchange/v2/market/instruments/candles?symbol=BCH%2FUSDT&period=1


Response:
{
    "code":200,
    "data":[
        [
            "2019-12-15T00:11:00.000Z",
            "100.1",
            "100.1",
            "100.1",
            "100.1",
            "0"
        ],
        [
            "2019-12-15T00:12:00.000Z",
            "100.1",
            "100.1",
            "100.1",
            "100.1",
            "0"
        ]
    ]
}

```
 
 
### 公共接口-获取全部的ticker信息

```
获取交易所现货全部ticker的最新成交价、买一价、卖一价和24交易量
限速规则：1次/1秒
HTTP GET /api/exchange/v2/market/ticker/list
```
请求参数：无


返回字段说明：

名称   | 类型  | 说明
---|---|---
symbol         | string | 币对名称，如BTC/USDT
latestPrice           | string | 最新价
bestAsk            | string | 卖一价
bestBid            | string | 买一价
high24h        | string | 24h最高价
low24h         | string | 24h最低价
volume24h      | string | 24h成交量
chg24h      | string | 24h涨跌幅
chg0h      | string | 0h涨跌幅

```
Request:
Url: http://域名/api/exchange/v2/market/ticker/list
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: c085be146e16a508b6d177a56c05b695f92579db41642ee7c80e385579a08123
	ACCESS-TIMESTAMP: 2019-12-08T08:51:50.508Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-12-08T08:51:50.508ZGET/api/exchange/v2/market/ticker/list


Response:
{
  "code": 200, 
  "data": [{
        "symbol":"BTCUSDT",
        "latestPrice":"1263.17",
        "bestBid":"1263.15",
        "bestAsk":"1263.17",
        "high24h":"1263.17",
        "low24h":"1263.17",
        "volume24h":"190226.215235",
	"chg24h":"4.12%",
        "chg0h":"1.21%"
    },
    {
        "symbol":"ETHUSDT",
        "latestPrice":"63.17",
        "bestBid":"63.15",
        "bestAsk":"63.17",
        "high24h":"63.17",
        "low24h":"63.17",
        "volume24h":"1906.12",
	"chg24h":"2.12%",
        "chg0h":"0.21%"
    }
    ]
}
```

 

### 公共接口-获取指定的ticker信息

```
获取交易所现货指定ticker的最新成交价、买一价、卖一价和24交易量
限速规则：6次/1秒
HTTP GET /api/exchange/v2/market/ticker/one
```
请求参数：无

名称   | 类型  | 是否必填 | 说明
---|---|---|---
symbol | string | 是 | 币对名称，如BTC/USDT

返回字段说明：

名称   | 类型  | 说明
---|---|---
symbol         | string | 币对名称，如BTC/USDT
latestPrice           | string | 最新价
bestAsk            | string | 卖一价
bestBid            | string | 买一价
high24h        | string | 24h最高价
low24h         | string | 24h最低价
volume24h      | string | 24h成交量
chg24h      | string | 24h涨跌幅
chg0h      | string | 0h涨跌幅

```
Request:
Url: http://域名/api/exchange/v2/market/ticker/one?symbol=BTC%2FUSDT
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
        "symbol":"BTCUSDT",
        "latestPrice":"63.17",
        "bestBid":"63.15",
        "bestAsk":"63.17",
        "high24h":"63.17",
        "low24h":"63.17",
        "volume24h":"1906.215235",
	"chg24h":"2.12%",
        "chg0h":"0.21%"
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

名称   | 类型  | 是否必填 | 说明
---|---|---|---
symbol | string | 是 | 币对名称，如BTC/USDT

返回字段说明：

名称 | 类型  | 说明
---|---|---
symbol   | string | 币对名称
price   | string | 成交价格
volume   | string | 成交数量
direction   | string | 方向
tradeTime   | string | 成交时间

```
说明：
1.只返回最新50条交易数据
2.[symbol|price|volume|direction|tradeTime]
```
```
Request:
Url: http://域名/api/exchange/v2/market/trades
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
            "10632.77",
            "0.03",
            "buy",
            "2019-06-24T02:58:17.368Z"
        ],
        [
            "BTC/USDT",
            "10643.13",
            "0.015",
            "sell",
            "2019-06-24T02:58:17.368Z"
        ],
        [
            "BTC/USDT",
            "10643.13",
            "0.015",
            "sell",
            "2019-06-24T02:58:17.368Z"
        ],
        [
            "BTC/USDT",
            "10639.97",
            "0.0055",
            "buy",
            "2019-06-24T02:58:17.368Z"
        ],
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
            "10639.99",
            "0.0314",
            "buy",
            "2019-06-24T02:58:17.369Z"
        ],
        [
            "BTC/USDT",
            "10651.81",
            "0.015",
            "sell",
            "2019-06-24T02:58:17.369Z"
        ],
        [
            "BTC/USDT",
            "10688.51",
            "0.015",
            "sell",
            "2019-06-24T02:58:17.369Z"
        ],
        [
            "BTC/USDT",
            "10688.51",
            "0.015",
            "sell",
            "2019-06-24T02:58:17.369Z"
        ],
        [
            "BTC/USDT",
            "10636.01",
            "0.0188",
            "buy",
            "2019-06-24T02:58:17.369Z"
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
        ],
        [
            "BTC/USDT",
            "10636.06",
            "0.0032",
            "buy",
            "2019-06-24T02:58:17.370Z"
        ],
        [
            "BTC/USDT",
            "10636.06",
            "0.0041",
            "buy",
            "2019-06-24T02:58:17.370Z"
        ],
        [
            "BTC/USDT",
            "10636.07",
            "0.0021",
            "buy",
            "2019-06-24T02:58:17.370Z"
        ],
        [
            "BTC/USDT",
            "10636.07",
            "0.0041",
            "buy",
            "2019-06-24T02:58:17.370Z"
        ],
        [
            "BTC/USDT",
            "10636.07",
            "0.0032",
            "buy",
            "2019-06-24T02:58:17.370Z"
        ],
        [
            "BTC/USDT",
            "10636.09",
            "0.0041",
            "buy",
            "2019-06-24T02:58:17.370Z"
        ],
        [
            "BTC/USDT",
            "10636.09",
            "0.0021",
            "buy",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10636.09",
            "0.0193",
            "buy",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10636.09",
            "0.0247",
            "buy",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10636.09",
            "0.0041",
            "buy",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10636.09",
            "0.0021",
            "buy",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10636.09",
            "0.0041",
            "buy",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10636.1",
            "0.002",
            "buy",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10636.09",
            "0.0001",
            "buy",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10660.97",
            "0.015",
            "sell",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10660.97",
            "0.015",
            "sell",
            "2019-06-24T02:58:17.371Z"
        ],
        [
            "BTC/USDT",
            "10660.97",
            "0.0186",
            "sell",
            "2019-06-24T02:58:17.372Z"
        ],
        [
            "BTC/USDT",
            "10660.97",
            "0.015",
            "sell",
            "2019-06-24T02:58:17.372Z"
        ],
        [
            "BTC/USDT",
            "10674.07",
            "0.1",
            "sell",
            "2019-06-24T02:58:17.372Z"
        ],
        [
            "BTC/USDT",
            "10660.89",
            "0.0039",
            "buy",
            "2019-06-24T02:58:17.372Z"
        ],
        [
            "BTC/USDT",
            "10660.89",
            "0.004",
            "buy",
            "2019-06-24T02:58:17.372Z"
        ],
        [
            "BTC/USDT",
            "10660.91",
            "0.004",
            "buy",
            "2019-06-24T02:58:17.372Z"
        ],
        [
            "BTC/USDT",
            "10660.91",
            "0.004",
            "buy",
            "2019-06-24T02:58:17.372Z"
        ],
        [
            "BTC/USDT",
            "10660.92",
            "0.004",
            "buy",
            "2019-06-24T02:58:17.372Z"
        ],
        [
            "BTC/USDT",
            "10660.94",
            "0.004",
            "buy",
            "2019-06-24T02:58:17.373Z"
        ],
        [
            "BTC/USDT",
            "10698.76",
            "0.003",
            "buy",
            "2019-06-24T02:58:17.373Z"
        ],
        [
            "BTC/USDT",
            "10698.76",
            "0.003",
            "buy",
            "2019-06-24T02:58:17.373Z"
        ],
        [
            "BTC/USDT",
            "10707.23",
            "0.03",
            "sell",
            "2019-06-24T02:58:17.373Z"
        ],
        [
            "BTC/USDT",
            "9231.88",
            "0.0031",
            "buy",
            "2019-06-24T02:58:17.373Z"
        ],
        [
            "BTC/USDT",
            "9231.86",
            "0.1",
            "buy",
            "2019-06-24T02:58:17.373Z"
        ],
        [
            "BTC/USDT",
            "9179.03",
            "0.01",
            "sell",
            "2019-06-24T02:58:17.373Z"
        ],
        [
            "BTC/USDT",
            "9179.02",
            "0.01",
            "sell",
            "2019-06-24T02:58:17.373Z"
        ],
        [
            "BTC/USDT",
            "9179.02",
            "0.01",
            "sell",
            "2019-06-24T02:58:17.373Z"
        ]
    ]
}

```



### 公共接口-获取Kline数据信息
```
获取指定不同周期的Kline数据信息, 单次请求的最大数据量是200。如果您选择的开始/结束时间和时间粒度导致超过单次请求的最大数据量，您的请求将只会返回200个数据。如果您希望在更大的时间范围内获取足够精细的数据，则需要使用多个开始/结束范围进行多次请求。
限速规则：1次/1秒
HTTP GET /api/exchange/v2/market/instruments/candles
```
Request parameters:

名称 | 类型 | 是否必填 | 描述
--- | --- | --- | ---
symbol | string | 是 | 交易对名 如 BTC/USDT
period | string | 是 | 时间周期, 值如下["1","3","5","15","30","60","120","240","360","720","D","W","M"]，分别对应[1min,3min,5min,15min,30min,1hour,2hour,4hour,6hour,12hour,1day,1week,1month]
start | string | 否 | 指定kline线开始时间，时间戳 精确到秒
end | string | 否 | 指定kline线开始时间，时间戳 精确到秒
```
start和end说明，如果只传start，那么返回start时间开始到现在的kline数据；如果只传end，则返回有记录开始时间到end结束的kline数据；如果start和end都不填，则返回系统保存的当前周期的所有kline数据
```
```
返回数组格式说明：
[
timestamp 开始时间
open    开盘价格
high    最高价格
low     最低价格
close   收盘价格
volume  交易量
]
```

```
Request:
Url: http://域名/api/exchange/v2/market/instruments/candles?symbol=BCH%2FUSDT&period=1
Method: GET
Headers:
Accept: application / json
ACCESS-KEY: 2c8b514c28b6404f0d0333b958379484
ACCESS-SIGN: 11178e61aa0f571a1a70a51ea63ac38dd013200e5b069c6c6a535a40a161a9a0
ACCESS-TIMESTAMP: 2019-12-16T09: 39: 57.086Z
Content-Type: application / json; charset = UTF-8
Cookie: locale = en_US
Body:
preHash: 2019-12-16T09:39:57.086ZGET/api/exchange/v2/market/instruments/candles?symbol=BCH%2FUSDT&period=1


Response:
{
    "code": 200,
    "data": [
        [
            "2019-12-15T00:13:00.000Z",
            "100.1",
            "100.1",
            "100.1",
            "100.1",
            "0"
        ],
        [
            "2019-12-15T00:12:00.000Z",
            "100.1",
            "100.1",
            "100.1",
            "100.1",
            "0"
        ]
    ]
}
```

 
### 公共接口-获取币的汇率
```
获取平台提供的汇率接口
限速规则：1次/1秒
HTTP GET /api/exchange/v2/market/rate/list
```
请求参数：
无


返回字段说明：

名称   | 类型  | 说明
---|---|---
symbol   | string | 资产币对
rate   | string | 对USD的利率
timestamp   | string | 请求时间,国际时间

```
Request:
Url: http://域名/api/exchange/v2/market/rate/list
Method: GET
Headers: 
Accept: application/json
ACCESS-KEY: 2c8b514c28b6404f0d0333b958379484
ACCESS-SIGN: ed9843787c351e10891cf91fc171506ff3fde81b638aba8619e8740808ea90ab
ACCESS-TIMESTAMP: 2019-12-16T09:41:59.433Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2019-12-16T09:41:59.433ZGET/api/exchange/v2/market/rate/list


Response:
{
    "code":200,
    "data":[
        {
            "symbol":"USD_USDT",
            "rate":"1.0001",
            "timestamp":"2019-12-16T09:42:00.248Z"
        },
        {
            "symbol":"USD_CNY",
            "rate":"7.0040",
            "timestamp":"2019-12-16T09:42:00.248Z"
        },
        {
            "symbol":"USD_KRW",
            "rate":"1173.7900",
            "timestamp":"2019-12-16T09:42:00.248Z"
        },
        {
            "symbol":"USD_JPY",
            "rate":"109.4700",
            "timestamp":"2019-12-16T09:42:00.248Z"
        }
    ]
}

```

### 私有接口-查询全部账户信息

```
获取用户现货资产的全部账户信息
限速次数：3次/1秒
HTTP GET /api/exchange/v2/account/list
```

请求参数
无

返回结果参数

名称  | 类型  | 说明
---|---|---
asset   | string | 资产名称/缩写
available   | string | 可用余额
frozenBalance   | string | 冻结余额
totalBalance   | string | 总额

```
Request:
Url: http://域名/api/exchange/v2/account/list
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: ac5de86c304a4402ce737c1d2ab9edf34446c6a5b9d5b3136b3b9c212548f322
	ACCESS-TIMESTAMP: 2019-06-12T08:11:24.218Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-12T08:11:24.218ZGET/api/exchange/v2/account/list


Response:
{
  "code": 200, 
  "data": [
    {
      "asset": "BTC", 
      "available": "466.00000000", 
      "frozenBalance": "34.00000000", 
      "totalBalance": "500.00000000"
    }, 
    {
      "asset": "ML", 
      "available": "0", 
      "frozenBalance": "0", 
      "totalBalance": "0"
    }, 
    {
      "asset": "ETN", 
      "available": "0", 
      "frozenBalance": "0", 
      "totalBalance": "0"
    }, 
    {
      "asset": "RIF", 
      "available": "0", 
      "frozenBalance": "0", 
      "totalBalance": "0"
    }, 
    {
      "asset": "XMR", 
      "available": "0", 
      "frozenBalance": "0", 
      "totalBalance": "0"
    }
  ]
}
```

### 私有接口-查询指定账户资产信息

```
获取现货用户指定资产的账户信息
限速次数：6次/1秒
HTTP GET /api/exchange/v2/account/one
```

请求参数

名称   | 类型  | 是否必填  | 说明
---|---|---|---
asset  | string | 是 | 资产名称/缩写，如BTC

返回结果参数

名称   | 类型  | 说明
---|---|---
asset   | string | 资产名称/缩写
available   | string | 可用余额
frozenBalance   | string | 冻结余额
totalBalance   | string | 总额，冻结+余额

```
Request:
Url: http://域名/api/exchange/v2/account/one?asset=BTC
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 7675fb4ad80e4821e5285290a5805f60c14d242fa4832b6b5d1c10613f9149a9
	ACCESS-TIMESTAMP: 2019-06-12T09:19:14.547Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-12T09:19:14.547ZGET/api/exchange/v2/account/one?asset=BTC


Response:
{
  "code": 200, 
  "data": {
    "asset": "BTC",
    "available": "466.00000000", 
    "frozenBalance": "34.00000000", 
    "totalBalance": "500.00000000"
  }
}
```

### 私有接口-下单

```
按用户输入进行下单操作
限速规则：6次/1秒
HTTP POST/api/exchange/v2/order/place
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol      | string | 是 | 币对名称，如BTC/USDT，用"/"分割
direction      | string | 是 | 方向，1:买 2:卖
price      | string | 是 | 下单价格，市价单类型赋值0
quantity      | string | 是 | 限价单委托数量，市价单卖出数量，市价单买入赋值0
orderType      | string | 是 | 1:限价 2:市价
notional      | string | 否 | 市价单买入金额
clientId      | string | 否 | 用户请求id，透传返回给用户


返回字段说明：

名称   | 类型  | 说明
---|---|---
orderId   | string | 生成的订单id
clientId   | string | 用户请求的clientId


```
Request:
Url: http://域名/api/exchange/v2/order/place
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


Response:
{
  "code": 200, 
  "data": {
    "orderId": "1911862608898764800", 
    "clientId": "1560332366247"
  }
}
```



### 私有接口-批量下单

```
按用户输入进行批量下单操作
限速规则：3次/1秒
HTTP POST/api/exchange/v2/order/batchPlaceOrder
```

请求参数是一个数组对象,包含下面参数


名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol      | string | 是 | 币对名称，如BTC/USDT，用"/"分割
direction      | string | 是 | 方向，1:买 2:卖
price      | string | 是 | 下单价格，市价单类型赋值0
quantity      | string | 是 | 限价单委托数量，市价单卖出数量，市价单买入时赋值0
orderType      | string | 是 | 1:限价 2:市价
notional      | string | 否 | 市价单买入金额
clientId      | string | 否 | 用户请求id，透传返回给用户




返回字段说明：

名称   | 类型  | 说明
---|---|---
orderId   | string | 生成的订单id
clientId   | string | 用户请求的clientId


```
Request:
Url: http://域名/api/exchange/v2/order/batchPlaceOrder
Method: POST
Headers: 
Accept: application/json
ACCESS-KEY: 2c8b514c28b6404f0d0333b958379484
ACCESS-SIGN: a7b3bf35b8c65edb5e9a8b8eacc0505ce0f20d93e6541e6526f16ae12dbb6c30
ACCESS-TIMESTAMP: 2019-11-23T12:58:09.300Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: [{"symbol":"BTC/USDT","price":"11111.0","quantity":"1","direction":"1","orderType":"1"},{"symbol":"BTC/USDT","price":"0.0","quantity":"0","direction":"1","orderType":"2","notional":"10"}]
preHash: 2019-11-23T12:59:40.797ZPOST/api/exchange/v2/order/batchPlaceOrder[{"symbol":"BTC/USDT","price":"11111.0","quantity":"1","direction":"1","orderType":"1"},{"symbol":"BTC/USDT","price":"0.0","quantity":"0","direction":"1","orderType":"2","notional":"10"}]


Response:
{
    "code":200,
    "data":[
        {
            "code":"200",
            "orderId": "1911862608898764800", 
            "message":""
        },
        {
            "code":"200",
            "orderId": "1911862608898764801", 
            "message":""
        }
    ]
}

```




### 私有接口-查询当前委托挂单列表

```
按用户请求进行当前委托挂单列表查询，订单未成交或部分成交,都在当前委托列表
限速规则：3次/1秒
HTTP GET/api/exchange/v2/order/openOrders
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol      | string | 否 | 币对名称，如BTC/USDT
latestOrderId      | string | 否 | 订单id，分页使用，默认值为空，返回最新20条数据，按订单id倒排显示。获取最后一个订单id-1，取下一页数据


```
说明：
分页查询，每页返回20条
```


返回字段说明：

名称   | 类型  | 说明
---|---|---
orderId   | string | 订单Id
baseAsset   | string | 交易货币，如BTC
quoteAsset   | string | 计价货币，如USDT
orderDirection   | string | 方向
quantity   | string | 订单数量
filledQuantity   | string | 已成交数量
amount   | string | 订单金额
filledAmount   | string | 已成交金额
avgPrice   | string | 平均价格
orderPrice   | string | 下单价格
orderStatus   | string | 订单状态，未成交：Open 完全成交：Filled 取消：Cancelled 部分成交：Partially cancelled
orderTime   | string | 下单时间
fee   | string | 手续费


```
Request:
Url: http://域名/api/exchange/v2/order/openOrders?symbol=BTC%2FUSDT
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: d06bb21d3f3ca13908e52bd9dd858b80e8a00d444e6fda8c2f8729bac5e20e6a
	ACCESS-TIMESTAMP: 2019-06-13T03:09:23.235Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-13T03:09:23.235ZGET/api/exchange/v2/order/openOrders?symbol=BTC%2FUSDT


Response:
{
  "code": 200, 
  "data": [
    {
      "orderId": "1912119792282841088", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "buy", 
      "quantity": "53", 
      "amount": "3426.45", 
      "filledAmount": "0", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "0", 
      "orderStatus": "Open", 
      "orderTime": "2019-06-13T02:41:24.0Z", 
      "totalFee": "0",
      "orderPrice": "8000.1"
    }, 
    {
      "orderId": "1911862608898764800", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "buy", 
      "quantity": "54", 
      "amount": "3645", 
      "filledAmount": "0", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "0", 
      "orderStatus": "Open", 
      "orderTime": "2019-06-12T09:39:27.0Z", 
      "totalFee": "0",
      "orderPrice": "8000.1"
    }, 
    {
      "orderId": "1911862547074723840", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "buy", 
      "quantity": "15", 
      "amount": "966.75", 
      "filledAmount": "0", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "0", 
      "orderStatus": "Open", 
      "orderTime": "2019-06-12T09:39:12.0Z", 
      "totalFee": "0",
      "orderPrice": "8000.1"
    }
  ]
}
```

### 私有接口-查询历史委托单列表

```
按用户请求进行历史委托单列表查询，订单撤销或订单完全成交后, 才进入历史委托单列表
限速规则：3次/1秒
HTTP GET/api/exchange/v2/order/closedOrders
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol      | string | 否 | 币对名称，如BTC/USDT
latestOrderId      | string | 否 | 订单id，分页使用，默认值为空，返回最新20条数据，按订单id倒排显示。获取最后一个订单id-1，取下一页数据

```
说明：
分页查询，每页返回20条
```


返回字段说明：

名称   | 类型  | 说明
---|---|---
orderId   | string | 订单Id
baseAsset   | string | 交易货币，如BTC
quoteAsset   | string | 计价货币，如USDT
orderDirection   | string | 方向
quantity   | string | 订单数量
amount   | string | 订单金额
filledAmount   | string | 已成交金额
takerFeeRate   | string | taker费率
makerFeeRate   | string | maker费率
avgPrice   | string | 平均价格
orderPrice   | string | 下单价格
orderStatus   | string | 订单状态，未成交：Open 完全成交：Filled 取消：Cancelled 部分成交：Partially cancelled
orderTime   | string | 下单时间
totalFee   | string | 手续费


```
Request:
Url: http://域名/api/exchange/v2/order/closedOrders
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: d0c6e8cfe818eb263e3860eb0d7261b588a8dc13ebaa5d0799bc2e154d49877b
	ACCESS-TIMESTAMP: 2019-06-13T11:39:45.024Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-13T11:39:45.024ZGET/api/exchange/v2/order/closedOrders

Response:
{
  "code": 200, 
  "data": [
    {
      "orderId": "1912131427156307968", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "buy", 
      "quantity": "63", 
      "amount": "4205.25", 
      "filledAmount": "3979.557", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "63.1675714285714285714286", 
      "orderPrice": "63.0", 
      "orderStatus": "Filled", 
      "orderTime": "2019-06-13T03:27:38.0Z", 
      "totalFee": "3.979557",
      "orderPrice": "8000.1"
    }, 
    {
      "orderId": "1911862608898764800", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "sell", 
      "quantity": "54", 
      "amount": "3645", 
      "filledAmount": "0", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "0", 
      "orderPrice": "10", 
      "orderStatus": "Cancelled", 
      "orderTime": "2019-06-12T09:39:27.0Z", 
      "totalFee": "0",
      "orderPrice": "8000.1"
    }
  ]
}
```

### 私有接口-查询指定订单信息

```
按用户请求进行指定订单查询，
限速规则：6次/1秒
HTTP GET/api/exchange/v2/order/info
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
orderId      | string | 是 | 委托单ID


返回字段说明：

名称   | 类型  | 说明
---|---|---
orderId   | string | 订单Id
baseAsset   | string | 交易货币，如BTC
quoteAsset   | string | 计价货币，如USDT
orderDirection   | string | 方向
quantity   | string | 订单数量
amount   | string | 订单金额
filledAmount   | string | 已成交金额
takerFeeRate   | string | taker费率
makerFeeRate   | string | maker费率
avgPrice   | string | 平均价格
orderPrice | string | 下单价格
orderStatus   | string | 订单状态，未成交：Open 完全成交：Filled 取消：Cancelled 部分成交：Partially cancelled
orderTime   | string | 下单时间
totalFee   | string | 手续费

```
Request:
Url: http://127.0.0.1:8604/api/exchange/v2/order/info?orderId=1911862608898764800
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: d7157f16ab942af83b27f9eb5532c9b41444aca040135c306255517407c319db
	ACCESS-TIMESTAMP: 2019-06-12T09:49:09.004Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-12T09:49:09.004ZGET/api/exchange/v2/order/info?orderId=1911862608898764800

Response:
{
  "code": 200, 
  "data": {
    "orderId": "1911862608898764800", 
    "baseAsset": "BTC", 
    "quoteAsset": "USDT", 
    "orderDirection": "sell", 
    "quantity": "54", 
    "amount": "3645", 
    "filledAmount": "0", 
    "avgPrice": "0", 
    "orderPrice": "10", 
    "takerFeeRate": "0.001", 
    "makerFeeRate": "0.001", 
    "orderStatus": "Cancelled", 
    "orderTime": "2019-06-12T09:39:27.0Z", 
    "totalFee": "0",
    "orderPrice": "8000.1"
  }
}

```


### 私有接口-查询订单成交明细列表

```
按用户请求进行订单成交明细列表查询，
限速规则：3次/1秒
HTTP GET/api/exchange/v2/order/trade/fills
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
orderId      | string | 是 | 委托单ID


返回字段说明：

名称   | 类型  | 说明
---|---|---
price   | string | 交易价格
quantity   | string | 交易数量
amount   | string | 交易金额
fee   | string | 手续费
direction   | string | 方向
tradeTime   | string | 订单交易时间，国际时间
feeByConi   | string | coni抵扣手续

```
Request:
Url: http://域名/api/exchange/v2/order/trade/fills?orderId=1912131427156307968
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 1f32e1c5d31e0258eba479023f0e768e318d6351097a3429eaa453b690364410
	ACCESS-TIMESTAMP: 2019-06-13T03:45:23.943Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-13T03:45:23.943ZGET/api/exchange/v2/order/fills?orderId=1912131427156307968

Response:
{
  "code": 200, 
  "data": [
    {
      "price": "63.16", 
      "quantity": "1.2", 
      "amount": "75.792", 
      "fee": "0.075792", 
      "direction": "buy", 
      "tradeTime": "2019-06-13T03:27:38.0Z", 
      "feeByConi": ""
    }, 
    {
      "price": "63.16", 
      "quantity": "1.2", 
      "amount": "75.792", 
      "fee": "0.075792", 
      "direction": "buy", 
      "tradeTime": "2019-06-13T03:27:38.0Z", 
      "feeByConi": ""
    }, 
    {
      "price": "63.16", 
      "quantity": "1.2", 
      "amount": "75.792", 
      "fee": "0.075792", 
      "direction": "buy", 
      "tradeTime": "2019-06-13T03:27:38.0Z", 
      "feeByConi": ""
    }, 
    {
      "price": "63.16", 
      "quantity": "1.1", 
      "amount": "69.476", 
      "fee": "0.069476", 
      "direction": "buy", 
      "tradeTime": "2019-06-13T03:27:38.0Z", 
      "feeByConi": ""
    }
  ]
}
```




### 私有接口-撤销指定委托单

```
按用户请求进行订单撤销，
限速规则：6次/1秒
HTTP POST /api/exchange/v2/order/cancel
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
orderId      | string | 是 | 委托单ID

返回字段说明：

名称   | 类型  | 说明
---|---|---
data   | string | 撤销的订单Id

```
Request:
Url: http://域名/api/exchange/v2/order/cancel
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: f3d17b20841930f24776f3e921a270b2b47157ca62153421346de50690ce8a93
	ACCESS-TIMESTAMP: 2019-06-13T03:25:56.360Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"orderId":"1911862608898764800"}
preHash: 2019-06-13T03:25:56.360ZPOST/api/exchange/v2/order/cancel{"orderId":"1911862608898764800"}


Response:
{
  "code": 200, 
  "data": "580719990266232832"
}
```

### 私有接口-批量撤销委托单

```
按用户请求进行订单批量撤销，
限速规则：3次/1秒
HTTP POST /api/exchange/v2/order/batchCancel
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
orderIds      | list<string> | 是 | 委托单ID

返回字段说明：

名称   | 类型  | 说明
---|---|---
data   | string | 撤销的订单Id

```
Request:
Url: http://域名/api/exchange/v2/order/batchCancel
Method: POST
Headers: 
Accept: application/json
ACCESS-KEY: 2c8b514c28b6404f0d0333b958379484
ACCESS-SIGN: f485d03ee462a717de28f465225a90b226c3d7c08e5ce9b36c42f41e635dbdaf
ACCESS-TIMESTAMP: 2019-12-20T03:21:50.580Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
lang: en_US
Body: {"orderIds":["1980983481458700288","1980983581337661440","1924511943331438592"]}
preHash: 2019-12-20T03:21:50.580ZPOST/api/exchange/v2/order/batchCancel{"orderIds":["1980983481458700288","1980983581337661440","1924511943331438592"]}

Response:
{
    "code":200,
    "data":[
        {
            "orderId":"1980983481458700288",
            "code":"200",
            "message":""
        },
        {
            "orderId":"1980983581337661440",
            "code":"200",
            "message":""
        },
        {
            "orderId":"1924511943331438592",
            "code":"3004",
            "message":"The order does not exist, the cancellation of failure"
        }
    ]
}

```







## 错误代码汇总

错误代码     |  message
---|:---
429 |请求太频繁
430 | 该资产暂不支持API用户交易
10001 | "ACCESS_KEY"不能为空
10002 | "ACCESS_SIGN"不能为空
10003 | "ACCESS_TIMESTAMP"不能为空
10005 | 无效的ACCESS_TIMESTAMP
10006 | 无效的ACCESS_KEY
10007 | 无效的Content_Type，请使用“application/json”格式
10008 | 请求时间戳过期
10009 | 系统错误
10010 | api 校验失败
11000 | 必填参数不能为空
11001 | 参数值错误
11002 | 参数值超过最大值限制
11003 | 第三方接口暂无数据返回
11004 | 订单价格精度不符合
11005 | 该币对暂未开通杠杆
11007 | 币对与资产不匹配
51800| 已成交,撤单失败
51801| 委托单不存在,撤单失败
51802| 交易对不合法
51803| 买入价不可高于现价{0}%
51804| 卖出价不可低于现价{0}%
51805| 委托价最多小数位{0}
51806| 委托量最多小数位{0}
51807| 最少买入{0}
51808| 最少卖出{0}
51809| 余额不足或已冻结
51810| 暂停卖出,恢复时间,以公告为准
51811| 用户没有交易权限
51812| 当前{0}小时周期的涨停价格为{1}，买入价格不能高于涨停价格，请调整买入价格
51813| 当前{0}小时周期的跌停价格为{1}，卖出价格不能低于跌停价格，请调整卖出价格
51814| 计划委托只能在未触发状态下撤单
51815| 委托类型错误
51816| 账户类型错误
51817| 交易对错误
51818| 交易方向错误
51819| 委托来源错误
51820| 触发价格错误
51821| 触发价格最多小数位{0}
51822| 买入价格不可高于触发价{0}%
51823| 卖出价格不可低于触发价{0}%
51824| 委托价格错误
51825| 委托总额错误
51826| 委托总额最多小数位{0}
51827| 委托量错误
51828| 高级委托挂单数量不能超过{0}
51829| 触发价格应该高于最新成交价
51830| 触发价格应该低于最新成交价
51831| 限价错误
51832| 限价最多小数位{0}
51833| 限价应该高于最新成交价
51834| 限价应该低于最新成交价
51835| 没有找到账户
51836| 委托不存在
51837| 委托订单号错误
51838| 批量委托数量不能超过{0}
51839| 账户冻结失败
51840| 查询账户失败
51841| 交易对未设置涨跌停
51842| 冰山委托展示数量必须大于0
51843| 查询涨跌停价格失败
51844| 开始时间错误
51845| 结束时间错误
51846| 开始时间应该早于结束时间
51847| 最大下载时间周期为{0}天
51848| 买入价格不可低于触发价{0}%
51849| 卖出价格不可高于触发价{0}%
51850| 最大下载任务数量为{0}
51851| 开始时间只能选3个月内

