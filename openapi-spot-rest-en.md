

# coinbene-spot-rest Spot openapi rest interface description

[中文版本](https://github.com/Coinbene/API-SPOT-v2-Documents/blob/master/openapi-spot-rest.md)

   * [coinbene-spot-rest Spot openapi rest interface description](#coinbene-spot-rest-spot-openapi-rest-interface-description)
      * [Basic Information](#basic-information)
      * [restriction of visit](#restriction-of-visit)
      * [Interface Type](#interface-type)
      * [Signature](#signature)
      * [Interface Specification](#interface-specification)
         * [Public Interface - Get all transaction configuration information](#public-interface---get-all-transaction-configuration-information)
         * [Public interface - Get the specified transaction pair configuration information](#public-interface---get-the-specified-transaction-pair-configuration-information)
         * [Public interface - Get depth](#public-interface---get-depth)
         * [Public interface - Get the specified all ticker information](#public-interface---get-the-specified-all-ticker-information)
         * [Public interface - Get the specified ticker information](#public-interface---get-the-specified-ticker-information)
         * [Public Interface - Check the latest transaction information](#public-interface---check-the-latest-transaction-information)
         * [Public Interface-Get Kline Data Information](#public-interface-get-kline-data-information)
         * [Public Interface-Get Exchange Rate](#public-interface-get-exchange-rate)
         * [Private Interface - Query all account information](#private-interface---query-all-account-information)
         * [Private Interface - Query specified account asset information](#private-interface---query-specified-account-asset-information)
         * [Private Interface - Order](#private-interface---order)
         * [Private Interface - Batch Order](#private-interface---batch-order)
         * [Private Interface - Query the current list of delegate orders](#private-interface---query-the-current-list-of-delegate-orders)
         * [Private Interface - Query History Order List](#private-interface---query-history-order-list)
         * [Private Interface - Query specified order information](#private-interface---query-specified-order-information)
         * [Private Interface - Query Order Transactions List](#private-interface---query-order-transactions-list)
         * [Private Interface - Undo the specified order](#private-interface---undo-the-specified-order)
         * [Private Interface - Bulk Revocation Order](#private-interface---bulk-revocation-order)
      * [Error Code Summary](#error-code-summary)
      

## Basic Information
- This section lists the baseurl for the REST interface: https://openapi-exchange.coinbene.com
- It is recommended to add the own server export IP after modifying the API to further enhance the API security check.
- The response of all interfaces is in JSON format
- All time and timestamp are UNIX time in milliseconds
- The HTTP 4XX error code is used to indicate the content, behavior, and format of the error.
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
- secret is generated when the user applies for the API.

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
Generate a sign string: da496b6d235c6bb76015eb660c3457958d2b26637a28937cd58d4329d588fb05

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
        Sn = sign("2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
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
baseAsset | string | Trading currency BTC  
quoteAsset | string | pricing Currency USDT  
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
baseAsset | string | Trading currency BTC  
quoteAsset | string | pricing Currency USDT 
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
depth | string | yes | Depth stall, values ​​are 5, 10, 50, 100. Default value 10

Return field description:

Name | Type | Description
---|---|---
asks | array | seller depth, [gear price, quantity]
bid | array | buyer depth, [gear price, quantity]


```
Request:
Url: http://domain/api/exchange/v2/market/orderBook?symbol=BTC%2FUSDT&depth=5
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
        ],
	"timestamp":"2019-08-09T02:34:28.284Z"
    }
}



```
 

### Public interface - Get the specified all ticker information

```

Get the latest transaction price, buy one price, sell one price and 24 transaction volume of the exchange all ticker
Speed ​​limit rule: 6 times / 1 second
HTTP GET /api/exchange/v2/market/ticker/list
```


Request parameters: none


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
chg24h | string | 24h Ups and downs
chg0h | string | 0h Ups and downs

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
        "symbol":"BTC/USDT",
        "latestPrice":"1263.17",
        "bestBid":"1263.15",
        "bestAsk":"1263.17",
        "high24h":"1263.17",
        "low24h":"1263.17",
        "volume24h":"190226.215235",
	"chg24h":"2.12%",
        "chg0h":"0.21%"
    },
    {
        "symbol":"ETH/USDT",
        "latestPrice":"63.17",
        "bestBid":"63.15",
        "bestAsk":"63.17",
        "high24h":"63.17",
        "low24h":"63.17",
        "volume24h":"1906.12",
	"chg24h":"3.12%",
        "chg0h":"1.21%"
    }
    ]
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
chg24h | string | 24h Ups and downs
chg0h | string | 0h Ups and downs

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
preHash: 2019-06-18T08:03:39.250ZGET/api/exchange/v2/market/ticker/one?symbol=BTC%2FUSDT


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
        "volume24h":"1906.215235",
	"chg24h":"2.12%",
        "chg0h":"0.21%"
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
timestamp | string | trade time

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



### Public Interface-Get Kline Data Information
```
Get Kline data information for different specified periods. The maximum amount of data for a single request is 200. If your chosen start / end time and time granularity result in exceeding the maximum amount of data for a single request, your request will only return 200 data. If you want to get enough granular data over a larger time frame, you need to make multiple requests with multiple start / end ranges.
Speed ​​limit rule: 1 time / 1 second
HTTP GET /api/exchange/v2/market/instruments/candles
```
Request parameters:

Name | Type | Required | Description
--- | --- | --- | ---
symbol | string | Yes | The name of the currency pair, such as BTC / USDT
period | string | is | time period, values ​​are as follows ["1", "3", "5", "15", "30", "60", "120", "240", "360", "720 "," D "," W "," M "], which correspond to [1min, 3min, 5min, 15min, 30min, 1hour, 2hour, 4hour, 6hour, 12hour, 1day, 1week, 1month]
start | string | No | Specify the start time of the kline line, timestamp accurate to the second
end | string | No | Specify kline line end time, timestamp accurate to seconds
```
start and end indicate that if only start is passed, then the kline data from the start time to the present is returned; if only end is passed, the kline data with the record start time to the end of the end is returned; if both start and end are not filled, the system is returned Save all kline data for the current cycle
```
```
Description of the returned array format:
[
timestamp start time
open price
high price
low lowest price
close closing price
volume
]
```

```
Request:
Url: http://domain_name/api/exchange/v2/market/instruments/candles?symbol=BCH%2FUSDT&period=1
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
 
### Public Interface-Get Exchange Rate
```
Get the exchange rate interface provided by the platform
Speed ​​limit rule: 1 time / 1 second
HTTP GET /api/exchange/v2/market/rate/list

```
Request parameters:
no


Return field description:

Name | Type | Description
--- | --- | ---
symbol | string | Asset Currency Pair
rate | string | Interest Rate on USD
timestamp | string | Request time, international time

```
Request:
Url: http: //domain/api/exchange/v2/market/rate/list
Method: GET
Headers:
Accept: application / json
ACCESS-KEY: 2c8b514c28b6404f0d0333b958379484
ACCESS-SIGN: ed9843787c351e10891cf91fc171506ff3fde81b638aba8619e8740808ea90ab
ACCESS-TIMESTAMP: 2019-12-16T09: 41: 59.433Z
Content-Type: application / json; charset = UTF-8
Cookie: locale = en_US
Body:
preHash: 2019-12-16T09:41:59.433ZGET/api/exchange/v2/market/rate/list


Response:
{
    "code": 200,
    "data": [
        {
            "symbol": "USD_USDT",
            "rate": "1.0001",
            "timestamp": "2019-12-16T09: 42: 00.248Z"
        },
        {
            "symbol": "USD_CNY",
            "rate": "7.0040",
            "timestamp": "2019-12-16T09: 42: 00.248Z"
        },
        {
            "symbol": "USD_KRW",
            "rate": "1173.7900",
            "timestamp": "2019-12-16T09: 42: 00.248Z"
        },
        {
            "symbol": "USD_JPY",
            "rate": "109.4700",
            "timestamp": "2019-12-16T09: 42: 00.248Z"
        }
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
price | string | yes | order price, Market Order Type Assignment 0
quantity | string | yes | Quantity of limit order entrusted, quantity of market order sold, Market order when buying is value 0
orderType | string | yes | 1: Limit  2: Market  8:postOnly 9:fok 10:ios
notional | string | no | Market Order Purchase Amount
clientId | string | no | user request id, transparently returned to the user


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

### Private Interface - Batch Order

```
Order by user input
Speed ​​limit rule: 3 times / 1 second
HTTP POST/api/exchange/v2/order/batchPlaceOrder
```
Request parameters: The request parameter is an array object containing the following parameters

Name | Type | Required | Description  
---|---|---|---  
symbol | string | yes | currency pair name, such as BTC/USDT  
direction | string | yes | direction, 1: buy 2: sell  
price | string | yes | order price，Market Order Type Assignment 0
quantity | string | yes | Quantity of limit order entrusted, quantity of market order sold，Market order when buying is value 0
orderType | string | yes | 1: Limit price 2: Market price
notional | string | no | Market Order Purchase Amount
clientId | string | no | user request id, transparently returned to the user


Return field description:

Name | Type | Description
---|---|---
orderId | string | generated order id
clientId | string | clientId requested by client


```
Request:
Url: http://domain/api/exchange/v2/order/batchPlaceOrder
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
baseAsset | string | Trading currency BTC  
quoteAsset | string | pricing Currency USDT 
orderDirection | string | direction
quantity | string | order quantity
fillQuantity | string | Number of transactions
amount | string | order amount
filledAmount | string |
avgPrice | string | Average price
orderPrice | string | order price
orderStatus | string | Order status, unfilled: Open Completed: Filled Cancel: Canceled Partial deal: Partially cancelled
orderTime | string | Order time
fee | string | handling fee


```
Request:
Url: http://domain/api/exchange/v2/order/openOrders?symbol=BTC%2FUSDT
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
      "orderPrice": "10", 
      "orderStatus": "Open",
      "orderTime": "2019-06-13T02:41:24.0Z",
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
      "orderPrice": "10", 
      "orderStatus": "Open",
      "orderTime": "2019-06-12T09:39:12.0Z",
      "totalFee": "0",
      "orderPrice": "8000.1"
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
baseAsset | string | Trading currency BTC  
quoteAsset | string | pricing Currency USDT 
orderDirection | string | direction
quantity | string | order quantity
amount | string | order amount
filledAmount | string |
takerFeeRate | string | taker rate
makerFeeRate | string | maker rate
avgPrice | string | Average price
orderPrice | string | order price
orderStatus | string | Order status, unfilled: Open Completed: Filled Cancel: Canceled Partial deal: Partially cancelled
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
      "avgPrice": "0", 
      "orderPrice": "10", 
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
---|---|---
orderId | string | Order Id  
baseAsset | string | Trading currency BTC  
quoteAsset | string | pricing Currency USDT 
orderDirection | string | Direction 
quantity | string | order quantity
amount | string | order amount
filledAmount | string |
takerFeeRate | string | taker rate
makerFeeRate | string | maker rate  
avgPrice | string | Average price  
orderPrice | string | order price
orderStatus | string | Order status, unfilled: Open Completed: Filled Cancel: Canceled Partial deal: Partially cancelled
orderTime | string | Order time 
totalFee | string | handling fee  

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
    "orderDirection": "buy",
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
ACCESS-KEY: 2c8b514c28b6404f0d0333b958379484
ACCESS-SIGN: f485d03ee462a717de28f465225a90b226c3d7c08e5ce9b36c42f41e635dbdaf
ACCESS-TIMESTAMP: 2019-12-20T03:21:50.580Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
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


## Error Code Summary

Error code | message
---|:---
429 | Requests are too frequent
430 | API user transactions are not supported at this time
10001 | "ACCESS_KEY" cannot be empty
10002 | "ACCESS_SIGN" cannot be empty
10003 | "ACCESS_TIMESTAMP" cannot be empty
10005 | Invalid ACCESS_TIMESTAMP
10006 | Invalid ACCESS_KEY
10007 | Invalid Content_Type, please use "application / json" format
10008 | Request timestamp expired
10009 | System Error
10010 | API authentication failed
11000 | Required parameter cannot be empty
11001 | Incorrect parameter value
11002 | Parameter value exceeds maximum limit
11003 | No data returned by third-party interface
11004 | Order price accuracy does not match
11005 | The currency pair has not yet opened leverage
11007 | Currency pair does not match asset
51800| The transaction has been traded, failure
51801| The order does not exist, the cancellation of failure
51802| TradePair Wrong
51803| Buy Price must not be more than current price {0}%
51804| Sell price must not be less than current price {0}%
51805| Order price Most decimal point {0}
51806| Order quantity Most decimal point {0}
51807| Buy at least {0}
51808| Sell at least {0}
51809| Insufficient balance or account is frozen
51810| selling not supported
51811| Sorry, you do not have the authority to trade.
51812| The buy price exceeds the limit of {1} within current {0}-hour cycle. Please adjust the price.
51813| The sell price exceeds the limit of {1} within current {0}-hour cycle. Please adjust the price.
51814| Schedule Order can only be cancelled before triggering
51815| Order Type Error
51816| Account Type Error
51817| Trade Pair Error
51818| Trade Orientation Error
51819| Order Interface Error
51820| Trigger Price Error
51821| Trigger price Most decimal point {0}
51822| Purchase price shall not be higher than trigger price {0}%
51823| Selling Price shall not be under Trigger Price{0}%
51824| Order Price Error
51825| Order Amount Error
51826| Order amount Most decimal point {0}
51827| Order Quantity Error
51828| Quantity of senior open order can not exceed {0}
51829| Trigger price shall be higher than the latest filled price
51830| Trigger price shall be lower than the latest filled price
51831| Limited Price Error
51832| Limited price Most decimal point {0}
51833| Limited price shall be higher than the latest filled price
51834| Limited price shall be lower than the latest filled price
51835| Account not found
51836| Order does not exist
51837| Order Number Error
51838| Quantity of batch ordering can not exceed {0}
51839| Account freezing failed
51840| Account checking failed
51841| Trade pair have no settings of price limit
51842| Showing Quantity of Iceberg Order shall be greater than 0
51843| Price limit checking failed
51844| Start time error
51845| End time error
51846| Start time should be earlier than end time
51847| Maximum download time period is {0} days
51848| Purchase Price shall not be under Trigger Price {0}%
51849| Selling price can not be higher than trigger price {0}%
51850| The maximum number of download tasks is {0}
51851| Start time: Only a specific time of the past 3 months is available

