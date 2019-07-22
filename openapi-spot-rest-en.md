

# coinbene-spot-rest Spot openapi rest interface description



## Basic Information
- This section lists the baseurl for the REST interface: http://api.coinbene.com
- It is recommended to add the own server export IP after modifying the API to further enhance the API security check.
- The response of all interfaces is in JSON format
- All time and timestamp are UNIX time in milliseconds
- The HTTP 4XX error code is used to indicate the content, behavior, and format of the error.
- HTTP 429 error code indicates warning access frequency is exceeded, IP will be blocked
- HTTP 418 indicates that access is continued after receiving 429, so it is blocked.
- The HTTP 5XX error code is used to indicate problems on the Coinbene service side.
- HTTP 504 indicates that the API server has submitted a request to the business core but failed to get a response. It is important to note that the 504 code does not represent a request failure, but is unknown. It is very likely that it has already been executed, and it is possible that the execution will fail and further confirmation is needed.
- Each interface may throw an exception. The exception response format is as follows:


```
{
  "code": 10001,
  "msg": "Invalid Paramater."
}
```

- The specific error code and its explanation are summarized in the error code.
- The interface of the GET method, the parameter must be sent in the query string.
- The interface of the POST method, the parameter is sent in the request body (content type application/json).
- No requirements are required for the order of the parameters.
## restriction of visit
- When the access interface exceeds the frequency limit, it will return a 429 status: the request is too frequent.
- Restrict the rule. If a valid API key is passed, the user id is used to limit the rate; if not, the user's public IP address is used to limit the speed. There are separate instructions on each interface.
## Interface Type
- Mainly two types of interfaces, public and private.
- The public interface can be called without authentication.
- Private interface user orders and accounts. Each private request must be signed using a canonical form of authentication. The private interface needs to be verified with your API key.
## Signature
All interface request headers must contain the following:
- ACCESS-KEY string type API key
- ACCESS-SIGN uses Hex to generate a string return. The specific code refers to the following Java version and Python version code.
- ACCESS-TIMESTAMP timestamp of the request
- All requests should contain application/json type content and be valid JSON.

ACCESS-SIGN value generation rules:
- Follow the timestamp + method + requestPath + body string (+ for string concatenation), and secret, encrypt using the HMAC SHA256 method, and finally return the byte array of the encrypted string to a string.
- The value of timestamp is the same as the ACCESS-TIMESTAMP request header. It must be the decimal time of the UTC time zone Unix timestamp or the ISO8601 standard time format, accurate to the millisecond.
- Method is the request method, all letters are capitalized: GET/POST
- requestPath is the request interface path, for example: /api/exchange/v2/market/orderBook
- body is the string of the request body. The GET request has no body information to omit; the POST request has a body information JSON string, such as {"symbol": "BTCUSDT", "order_id": "7440"}
- secket is generated when the user applies for the API.

Sample interface request:
- GET protocol interface in two cases:

```
1. Without parameters:
Signature string preHash: 2019-06-13T11:18:29.009ZGET/api/exchange/v2/market/tradePair/list
2. With parameters:
Signature string preHash: 2019-06-13T11:18:29.009ZGET/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT

```


```
Url: http://domain/api/exchange/v2/market/tradePair/list
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
Url: http://domain/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT
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


- POST protocol interface situation:
```
Signature string preHash: 2019-06-13T11:27:14.493ZPOST/api/exchange/v2/order/place{"symbol":"BTC/USDT","price":"67.3","quantity":"38" , "direction": "1", "clientId": "1560425234454"}
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
preHash: 2019-06-13T11:27:14.493ZPOST/api/exchange/v2/order/place{"symbol":"BTC/USDT","price":"67.3","quantity":"38"," Direction":"1","clientId":"1560425234454"}

```
- Signature algorithm verification:


```
Source string: 2019-05-25T03:20:30.362ZGET/api/spot/v2/account/info
Secret:9daf13ebd76c4f358fc885ca6ede5e27
Generate a sign string: a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2

Sample code (Java version):
**
   * Generate signature
   *
   * @param timeStamp timestamp
   * @param method Request method: POST or GET
   * @param requestUrl url
   * @param requestBody request content, no null passed
   * @param secret key
   */
  Private String signForContractOpenApi(String timeStamp, String method, String requestUrl, String requestBody, String secret) {
    String shaResource = timeStamp + method + requestUrl + (requestBody == null ? "" : requestBody);
    System.out.println(shaResource);
    String signStr = sha256_HMAC(shaResource, secret);
    Return signStr;
  }

  /**
   * sha256_HMAC encryption
   *
   * @param resource signature source string
   * @param secret key
   * @return Encrypted string
   */
  Private static String sha256_HMAC(String resource, String secret) {
    String hash = "";
    Try {
      Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
      SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
      sha256_HMAC.init(secret_key);
      Byte[] bytes = sha256_HMAC.doFinal(resource.getBytes("UTF-8"));
      Hash = byteArrayToHexString(bytes);
    } catch (Exception e) {
      System.out.println("Error HmacSHA256 ===========" + e.getMessage());
    }
    Return hash;
  }

  /**
   * Convert the encrypted byte array to a string
   *
   * @param bytes byte array
   * @return string
   */
  Private static String byteArrayToHexString(byte[] bytes) {
    StringBuffer buffer = new StringBuffer();
    String stmp;
    For (int index = 0; bytes != null && index < bytes.length; index++) {
      Stmp = Integer.toHexString(bytes[index] & 0XFF);
      If (stmp.length() == 1) {
        Buffer.append('0');
      }
      Buffer.append(stmp);
    }
    Return buffer.toString().toLowerCase();
  }

Sample code (Python version):

Import hashlib
Import hmac
Import unittest

Def sign(message, secret):
    """
    Gen sign
    :param message: message wait sign
    :param secret: secret key
    :return:
    """
    Secret = secret.encode('utf-8')
    Message = message.encode('utf-8')
    Sign = hmac.new(secret, message, digestmod=hashlib.sha256).hexdigest()
    Return sign

Class TestUtil(unittest.TestCase):
    Def test_sign(self):
        Sn = sign("2019-05-25T03:20:30.362ZGET/api/spot/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
        self.assertEqual(sn, "a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2")


```


## Interface Specification

### Public Interface - Get all transaction configuration information
```
Get the exchange currency pair configuration list
Speed ​​limit rule: 2 times / 1 second
HTTP GET /api/exchange/v2/market/tradePair/list
```
Request parameters:
no

Return field description:

Name | Type | Description
--- | --- | ---  
symbol | string | currency pair name, such as BTC/USDT  
baseAsset | string | pricing currency BTC  
quoteAsset | string | Trading Currency USDT  
pricePrecision | string | Price accuracy  
amountPrecision | string | quantity accuracy  
takerFeeRate | string | taker fee  
makerFeeRate | string | maker fee  
minAmount | string | Latest Lots  
priceFluctuation | string | Price fluctuation limit  
site | string | own site  

```
Request:
Url: http://domain/api/exchange/v2/market/tradePair/list
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
      "site": "MAIN",
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
      "site": "MAIN",
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
      "site": "MAIN",
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
      "site": "MAIN",
      "priceFluctuation": "0.50"
    }
  ]
}
```


### Public interface - Get the specified transaction pair configuration information
```
Get the specified transaction pair configuration information
Speed ​​limit rule: 3 times / 1 second
HTTP GET /api/exchange/v2/market/tradePair/one
```
Request parameters:  

Name | Type | Required | Description  
---|---|---|---  
symbol | string | yes | currency pair name, such as BTC/USDT

Return field description:

Name | Type | Description
---|---|---
symbol | string | currency pair name, such as BTC/USDT
baseAsset | string | pricing currency BTC
quoteAsset | string | Trading Currency USDT
pricePrecision | string | Price accuracy
amountPrecision | string | quantity accuracy
takerFeeRate | string | taker fee
makerFeeRate | string | maker fee
minAmount | string | Latest Lots
priceFluctuation | string | Price fluctuation limit
site | string | own site

```
Request:
Url: http:// domain name/api/exchange/v2/market/tradePair/one?symbol=BTC%2FUSDT
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
    "site": "MAIN",
    "priceFluctuation": "0.50"
  }
}
```

### Public interface - Get depth
```
Get the exchange spot depth list
Speed ​​limit rule: 6 times / 1 second
HTTP GET /api/exchange/v2/market/orderBook?symbol=BTC/USDT
```

Request parameters:

Name | Type | Required | Description  
---|---|---|---  
symbol | string | yes | currency pair name, such as BTC/USDT  
depth | string | No | Depth stall, values ​​are 5, 10, 50, 100. Default value 10

Return field description:

Name | Type | Description
---|---|---
asks | array | seller depth, [gear price, quantity]
bid | array | buyer depth, [gear price, quantity]


```
Request:
Url: http://domain/api/exchange/v2/market/orderBook?symbol=ETH/USDT&depth=10
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






```
 


### Public interface - Get the specified ticker information

```
Get the latest transaction price, buy one price, sell one price and 24 transaction volume of the exchange ticker
Speed ​​limit rule: 6 times / 1 second
HTTP GET /api/exchange/v2/market/ticker/one
```
Request parameters: none

Name | Type | Required | Description  
---|---|---|---  
symbol | string | yes | currency pair name, such as BTC/USDT  

Return field description:

Name | Type | Description
---|---|---
symbol | string | currency pair name, such as BTC/USDT
latestPrice | string | Latest price
bestAsk | string | Sell one price
bestBid | string | Buy one price
high24h | string | 24h highest price
low24h | string | 24h lowest price
volume24h | string | 24h volume

```
Request:
Url: http://domain/api/exchange/v2/market/ticker/one?symbol=BTC%2FUSDT
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
ACCESS-SIGN: 92d75e564278da59c9dc1a48ec141ea2b12041f23c58fcd2910d39684f037e46
ACCESS-TIMESTAMP: 2019-06-18T08:03:39.250Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=zh_CN
Body:
preHash: 2019-06-18T08:03:39.250ZGET/api/exchange/v2/market/ticker/info?symbol=BTC%2FUSDT


Response:
{
  "code": 200,
  "data": {
        "symbol": "BTC/USDT",
        "latestPrice": "63.17",
        "bestBid": "63.15",
        "bestAsk": "63.17",
        "high24h": "63.17",
        "low24h": "63.17",
        "volume24h":"1906.215235"
    }
}
```


### Public Interface - Check the latest transaction information

```
Get the latest deal information on the spot of the exchange
Speed ​​limit rule: 3 times / 1 second
HTTP GET /api/exchange/v2/market/trades
```
Request parameters:

Name | Type | Required | Description  
---|---|---|---  
symbol | string | yes | currency pair name, such as BTC/USDT

Return field description:

Name | Type | Description
---|---|---
symbol | string | currency pair name
price | string | transaction price
volume | string |
direction | string | direction
tradeTime | string | Transaction time

```
Description:
1. Only return the latest 50 transaction data
2.[symbol|price|volume|direction|tradeTime]
```
```
Request:
Url: http://domain/api/exchange/v2/market/trades
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

### Private Interface - Query all account information

```
Obtain all account information of the user's spot assets
Speed ​​limit: 3 times / 1 second
HTTP GET /api/exchange/v2/account/list
```

Request parameter
no

Return result parameter

Name | Type | Description  
---|---|---  
asset | string | asset name  
available | string | available balance  
frozenBalance | string | Freeze balance  
totalBalance | string | total  

```
Request:
Url: http://domain/api/exchange/v2/account/list
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
      "rrozenBalance": "34.00000000",
      "totalBalance": "500.00000000"
    },
    {
      "asset": "ML",
      "available": "0",
      "rrozenBalance": "0",
      "totalBalance": "0"
    },
    {
      "asset": "ETN",
      "available": "0",
      "rrozenBalance": "0",
      "totalBalance": "0"
    },
    {
      "asset": "RIF",
      "available": "0",
      "rrozenBalance": "0",
      "totalBalance": "0"
    },
    {
      "asset": "XMR",
      "available": "0",
      "rrozenBalance": "0",
      "totalBalance": "0"
    }
  ]
}
```

### Private Interface - Query specified account asset information

```
Obtain account information of the spot user specified assets
Speed ​​limit: 6 times / 1 second
HTTP GET /api/exchange/v2/account/one
```

Request parameter

Name | Type | Required | Description  
---|---|---|---  
asset | string | yes | asset name/abbreviation, such as BTC

Return result parameter

Name | Type | Description  
---|---|---  
asset | string | asset name  
available | string | available balance  
frozenBalance | string | Freeze balance
totalBalance | string | total, freeze + balance

```
Request:
Url: http://domain/api/exchange/v2/account/one?asset=BTC
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
    "rrozenBalance": "34.00000000",
    "totalBalance": "500.00000000"
  }
}
```

### Private Interface - Order

```
Order by user input
Speed ​​limit rule: 6 times / 1 second
HTTP POST/api/exchange/v2/order/place
```
Request parameters:

Name | Type | Required | Description  
---|---|---|---  
symbol | string | yes | currency pair name, such as BTC/USDT  
direction | string | yes | direction, 1: buy 2: sell  
price | string | yes | order price  
quantity | string | yes | quantity  
clientId | string | no | user request id, transparently returned to the user


```
1. Only limit order types are supported.
```

Return field description:

Name | Type | Description
---|---|---
orderId | string | generated order id
clientId | string | clientId requested by client


```
Request:
Url: http://domain/api/exchange/v2/order/place
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
ACCESS-SIGN: 237dbda384965cf78284bdb028ab05229343e635e46adca6323833ade6d8704a
ACCESS-TIMESTAMP: 2019-06-13T11:27:14.493Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=zh_CN
Body: {"symbol":"BTC/USDT","price":"67.3","quantity":"38","direction":"1","clientId":"1560425234454"}
preHash: 2019-06-13T11:27:14.493ZPOST/api/exchange/v2/order/place{"symbol":"BTC/USDT","price":"67.3","quantity":"38"," Direction":"1","clientId":"1560425234454"}


Response:
{
  "code": 200,
  "data": {
    "orderId": "1911862608898764800",
    "clientId": "1560332366247"
  }
}
```


### Private Interface - Query the current list of delegate orders

```
Order list query by user request,
Speed ​​limit rule: 3 times / 1 second
HTTP GET/api/exchange/v2/order/openOrders
```
Request parameters:

Name | Type | Required | Description  
---|---|---|---  
symbol | string | no | currency pair name, such as BTC/USDT  
latestOrderId | string | No | Order id, used by page, the default value is empty, return the latest 20 data, displayed in reverse order by order id. Get the last order id-1, take the next page of data


```
Description:
Paging query, return 20 per page
```


Return field description:

Name | Type | Description
---|---|---
orderId | string | Order Id
baseAsset | string | base currency, such as BTC
quoteAsset | string | Trading currency, such as USDT
orderDirection | string | direction
quantity | string | order quantity
fillQuantity | string | Number of transactions
amount | string | order amount
filledAmount | string |
avgPrice | string | Average price
orderStatus | string | Order status, unfilled: open Completed: filled Cancel: canceled Partial deal: partialCancelled
orderTime | string | Order time
fee | string | handling fee


```
Request:
Url: http://domain/api/exchange/v2/order/openOrders?symbol=BTC%2FUSDT&pageNum=1&pageSize=3
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
      "totalFee": "0"
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
      "totalFee": "0"
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
      "totalFee": "0"
    }
  ]
}
```

### Private Interface - Query History Order List

```
Order list query by user request,
Speed ​​limit rule: 3 times / 1 second
HTTP GET/api/exchange/v2/order/closedOrders
```
Request parameters:

Name | Type | Required | Description  
---|---|---|---  
symbol | string | no | currency pair name, such as BTC/USDT  
latestOrderId | string | No | Order id, used by page, the default value is empty, return the latest 20 data, displayed in reverse order by order id. Get the last order id-1, take the next page of data  

```
Description:
Paging query, return 20 per page
```


Return field description:

Name | Type | Description
---|---|---
orderId | string | Order Id
baseAsset | string | base currency, such as BTC
quoteAsset | string | Trading currency, such as USDT
orderDirection | string | direction
quantity | string | order quantity
amount | string | order amount
filledAmount | string |
takerFeeRate | string | taker rate
makerFeeRate | string | maker rate
avgPrice | string | Average price
orderStatus | string | Order status, unfilled: open Completed: filled Cancel: canceled Partial deal: partially cancelled
orderTime | string | Order time
totalFee | string | handling fee


```
Request:
Url: http://domain/api/exchange/v2/order/closedOrders
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
      "orderStatus": "Filled",
      "orderTime": "2019-06-13T03:27:38.0Z",
      "totalFee": "3.979557"
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
      "orderStatus": "Cancelled",
      "orderTime": "2019-06-12T09:39:27.0Z",
      "totalFee": "0"
    }
  ]
}
```


### Private Interface - Query specified order information

```
Order list query by user request,
Speed ​​limit rule: 6 times / 1 second
HTTP GET/api/exchange/v2/order/info
```
Request parameters: 

Name | Type | Required | Description  
---|---|---|---  
orderId | string | yes | order ID


Return field description:

Name | Type | Description  
---|---|---|---  
orderId | string | Order Id  
baseAsset | string | base currency, such as BTC  
quoteAsset | string | Trading currency, such as USDT.  
orderDirection | string | Direction, 1: Buy 2: Buy.  
quantity | string | order quantity.  
amount | string | order amount.  
filledAmount | string |.  
takerFeeRate | string | taker rate.  
makerFeeRate | string | maker rate.  
avgPrice | string | Average price.  
orderStatus | string | Order status, unfilled: open Completed: filled Cancel: canceled Partial deal: partially cancelled.  
orderTime | string | Order time.  
totalFee | string | handling fee.  

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
    "orderDirection": "2",
    "quantity": "54",
    "amount": "3645",
    "filledAmount": "0",
    "takerFeeRate": "0.001",
    "makerFeeRate": "0.001",
    "orderStatus": "Cancelled",
    "orderTime": "2019-06-12T09:39:27.0Z",
    "totalFee": "0"
  }
}

```


### Private Interface - Query Order Transactions List

```
Order list query by user request,
Speed ​​limit rule: 3 times / 1 second
HTTP GET/api/exchange/v2/order/trade/fills
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
orderId | string | yes | order ID


Return field description:

Name | Type | Description
---|---|---
price | string | transaction price
quantity | string | Number of transactions
amount | string | transaction amount
fee | string | handling fee
direction | string | direction
tradeTime | string | Order trading time, international time
feeByConi | string | coni deduction

```
Request:
Url: http://domain/api/exchange/v2/order/trade/fills?orderId=1912131427156307968
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




### Private Interface - Undo the specified order

```
Order list query by user request,
Speed ​​limit rule: 6 times / 1 second
HTTP POST /api/exchange/v2/order/cancel
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
orderId | string | yes | order ID

Return field description:

Name | Type | Description
---|---|---
data | string | Undo Order Id

```
Request:
Url: http://domain/api/exchange/v2/order/cancel
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

### Private Interface - Bulk Revocation Order

```
Order list query by user request,
Speed ​​limit rule: 3 times / 1 second
HTTP POST /api/exchange/v2/order/batchCancel
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
orderIds | list<string> | Yes | Order ID

Return field description:

Name | Type | Description
---|---|---
data | string | Undo Order Id

```
Request:
Url: http://domain/api/exchange/v2/order/batchCancel
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
ACCESS-SIGN: 29b7636a3732fdbbcdde4144537faef3b1da1ed1f72f90552ccbf9b44bd79f12
ACCESS-TIMESTAMP: 2019-06-13T03:28:11.204Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=zh_CN
Body: {"orderIds":["578639816552972288","578639902896914432"]}
preHash: 2019-06-13T03:28:11.204ZPOST/api/exchange/v2/order/batchCancel{"orderIds":["578639816552972288","578639902896914432"]}

Response:
{
  "code": 200,
  "data": [
    {
      "orderId": "578639816552972288",
      "message": "The order does not exist, the cancellation of failure"
    },
    {
      "orderId": "578639902896914432",
      "message": "The order does not exist, the cancellation of failure"
    }
  ]
}

```










