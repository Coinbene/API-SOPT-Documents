
#Description of coinbene exchange rest interface

[TOC]


##Basic information

-Rest interface baseurl: https://openapi-exchange.coinbene.com
-It is recommended to modify and add your own server exit IP after creating the API to further enhance the API security verification
-All interfaces respond in JSON format
-All timestamps and timestamps are UNIX time in milliseconds
-Http 4xx error code is used to indicate the wrong request content, behavior and format.
-The HTTP 5xx error code is used to indicate a problem on the service side of coinbene.
-The specific error code and its explanation are summarized in the error code.

-The interface of the get method. The parameters must be sent in the query string.
-The interface of the post method, and the parameters are sent in the request body (content type application / JSON).
-The order of parameters is not required.

##Access restrictions

-When the access interface exceeds the frequency limit, it will return 429 status, and the request is too frequent.
-When 405 status is returned, the request is too frequent and the IP is blocked
-If a valid API key is passed in, the user ID will be used for speed restriction; if not, the user's public IP will be used for speed restriction.

##Interface type

-There are mainly two types of interfaces, public interface and private interface.
-The public interface can be called without authentication.
-Each private request must be signed in the canonical form of authentication. Private interfaces need to be validated using your API key.

##Signature method

All interface request headers must contain the following contents:

-API key of access-key string type
-Access-sign uses hex to generate strings
-The time stamp of the access-timestamp initiation request
-All requests should contain application / JSON type content and be valid JSON.

Rules for generating access-sign values:

-According to timestamp + method + requestpath + body string (+ indicates string connection) and secret, HMAC sha256 method is used to encrypt, and finally the byte array of encrypted string is converted into string to return.
-The value of timestamp is the same as the access-timestamp request header. It must be the decimal seconds of UNIX timestamp in UTC time zone or the time format of iso8601 standard, accurate to milliseconds.
-Method is the request method with all uppercase letters: get / post
-The requestpath is the path of the request interface, for example, / API / V3 / instrument / depth
-Body is the string of the request body. The get request has no body information and can be omitted; the post request has a body information JSON string, such as {"instrument"_ id":"BTC","order_ id":"xxxx"}
-Secret is generated when the user applies for the API
-Please do not disclose the secret to other people or transfer it to the server at any time

Interface request example:

-There are two cases of get protocol interface

```
1. Without parameters:
preHash String：2021-01-07T07:30:07.709ZGET/api/v3/spot/instruments/trade_ pair_ list
2. With parameters:
preHash String：2021-01-05T03:19:44.727ZGET/api/v3/spot/instruments/trade_ pair_ one?instrument_ id=BTC
```




-Post protocol interface:

```
preHash String：2021-01-07T09:22:36.443ZPOST/api/v3/spot/order{"instrument_ id":"BTC/USDT","price":"37994.13","quantity":"1","direction":"2"}
```

-Signature algorithm verification:


```
Source string: 2021-01-07t09:22:36.443zpost/api/v3/spot/order {"instrument"_ id":"BTC/USDT","price":"37994.13","quantity":"1","direction":"2"}
secret：5a3b727e4cd241af80561d0d415b6edc
Generate sign string: 38b6ca282291cc6ff1a57de6141783d9d4ed3a182850c71afd3f18ed889541ebf

Sample code (Java version)
**
*Generate signature
*
*@ param method request method: post or get
* @param requestUrl url
*The request content of @ param requestbody is not null
*@ param secret key
*/
private String signForContractOpenApi(String method, String requestUrl, String requestBody, String secret) {
final DateTimeFormatter utcFormatter = DateTimeFormatter.ofPattern ("yyyy-MM-dd'T'HH:mm: ss.SSS 'Z'");
String timestamp = utcFormatter.format ( ZonedDateTime.now ( ZoneOffset.UTC ));
//NEVER use ZonedDateTime.now ( ZoneOffset.UTC ).toString(),
//which may produce timestamp like 'yyyy-MM-dd'T'HH:mm: ss.SSSSSS 'Z', but it depends.
String shaResource = timeStamp + method + requestUrl + (requestBody == null ? "" : requestBody);
System.out.println (shaResource);
String signStr = sha256_ HMAC(shaResource, secret);
return signStr;
}

/**
* sha256_ HMAC encryption
*
*@ param resource signature source string
*@ param secret key
*@ return encrypted string
*/
private static String sha256_ HMAC(String resource, String secret) {
String hash = "";
try {
Mac sha256_ HMAC = Mac.getInstance ("HmacSHA256");
SecretKeySpec secret_ key = new SecretKeySpec( secret.getBytes ("UTF-8"), "HmacSHA256");
sha256_ HMAC.init (secret_ key);
byte[] bytes = sha256_ HMAC.doFinal ( resource.getBytes ("UTF-8"));
hash = byteArrayToHexString(bytes);
} catch (Exception e) {
System.out.println ("Error HmacSHA256 ===========" + e.getMessage());
}
return hash;
}

/**
*Convert encrypted byte array to string
*
*@ param bytes byte array
*@ return string
*/
private static String byteArrayToHexString(byte[] bytes) {
StringBuffer buffer = new StringBuffer();
String stmp;
for (int index = 0; bytes != null && index < bytes.length ; index++) {
stmp = Integer.toHexString (bytes[index] & 0XFF);
if ( stmp.length () == 1) {
//One digit zero
buffer.append ('0');
}
buffer.append (stmp);
}
return buffer.toString ().toLowerCase();
}

Sample code (Python version)

import hashlib
import hmac
import unittest

def sign(message, secret):
"""
gen sign
:param message: message wait sign
:param secret: secret key
:return:
"""
secret = secret.encode ('utf-8')
message = message.encode ('utf-8')
sign = hmac.new (secret, message, digestmod= hashlib.sha256 ).hexdigest()
return sign

class TestUtil( unittest.TestCase ):
def test_ sign(self):
sn = sign("2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
self.assertEqual (sn, "a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2")

Sample code (kotlin version)
private fun ByteArray.toHex () = this.joinToString (separator = "") { it.toInt ().and(0xff).toString(16).padStart(2, '0') }
private fun String.sha256 (secretKey: String): ByteArray = HmacUtils.hmacSha256 (secretKey, this)
private val utcFormatter = DateTimeFormatter.ofPattern ("yyyy-MM-dd'T'HH:mm: ss.SSS 'Z'")
**
*Generate signature
*
*@ param method request method: post or get
* @param requestUrl url
*The request content of @ param requestbody is not null
*@ param secret key
*/
fun signForContractOpenApi(method: String, requestUrl: String, requestBody: String, secret:String ) {
//NEVER use ZonedDateTime.now ( ZoneOffset.UTC ).toString(),
//which may produce timestamp like 'yyyy-MM-dd'T'HH:mm: ss.SSSSSS 'Z', but it depends.
val timestamp = utcFormatter.format ( ZonedDateTime.now ( ZoneOffset.UTC ))
retrun "$timestamp${ method.toUpperCase ()}$requestUrl${body ?: ""}"
.sha256(apiSecret)
.toHex()
}
```


##Interface specification

###Public interface - get currency pair list

```
Obtain the information list of all currency pairs of the exchange
Speed limit rule: 5 times / 1 second
HTTP GET /api/v3/spot/instruments/trade_ pair_ list
```

Request parameters:

nothing

Return field description:

|Name | type | description ||
| ----------------- | ------ | ---------------------- | ---- |
| trade_ pair_ Name | string | currency pair name, such as 1btc / BTC ||
| base_ Asset | string | underlying assets, such as 1btc ||
| quote_ Asset | string | valuation assets, such as BTC ||
| price_ Precision | string | price precision ||
| amount_ Precision | string | quantity precision ||
| taker_ fee_ Rate | string | taker|
| maker_ fee_ Rate | string | maker|
| min_ Amount | string | minimum number of delegates ||
| price_ Fluctuation | string | price fluctuation limit ||



```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/instruments/trade_ pair_ list
Method: GET
Headers:
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T07:30:07.709ZGET/api/v3/spot/instruments/trade_ pair_ list

Response:
{
"code":200,
"data":[
{
"trade_ pair_ name":"1BTC/BTC",
"base_ asset":"1BTC",
"quote_ asset":"BTC",
"price_ precision":"2",
"amount_ precision":"2",
"taker_ fee_ rate":"0.1",
"maker_ fee_ rate":"0.1",
"min_ amount":"0.1",
"price_ fluctuation":"0.50"
},
{
"trade_ pair_ name":"ABBC/BTC",
"base_ asset":"ABBC",
"quote_ asset":"BTC",
"price_ precision":"8",
"amount_ precision":"2",
"taker_ fee_ rate":"0.001",
"maker_ fee_ rate":"0.001",
"min_ amount":"1",
"price_ fluctuation":"0.50"
},
...
]
}
```

###Common interface - single currency pair information

```
Access to exchange single currency pair information
Speed limit rule: 5 times / 1 second
HTTP GET /api/v3/spot/instruments/trade_ pair_ one?instrument_ id=BTC%2FUSDT
```

Request parameters:

|Name | type | description ||
| ------------- | ------ | ---------------------- | ---- |
| instrument_ ID | string | currency pair name, such as BTC / usdt ||



Return field description:

|Name | type | description|
| ----------------- | ------ | ---------------------- |
| trade_ pair_ Name | string | currency pair name, such as BTC / usdt|
| base_ Asset | string | underlying assets, such as BTC|
| quote_ Asset | string | valuation assets, such as usdt|
| price_ Precision | string | price precision|
| amount_ Precision | string | quantity precision|
| taker_ fee_ Rate | string | taker|
| maker_ fee_ Rate | string | maker|
| min_ Amount | string | minimum number of delegates|
| price_ Fluctuation | string | price fluctuation limit|




```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/instruments/trade_ pair_ one?instrument_ id=BTC%2FUSDT
Method: GET
Headers:
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T07:36:29.790ZGET/api/v3/spot/instruments/trade_ pair_ one?instrument_ id=BTC%2FUSDT

Response:
{
"code":200,
"data":{
"trade_ pair_ name":"BTC/USDT",
"base_ asset":"BTC",
"quote_ asset":"USDT",
"price_ precision":"2",
"amount_ precision":"4",
"taker_ fee_ rate":"0.0015",
"maker_ fee_ rate":"0.013",
"min_ amount":"0.004",
"price_ fluctuation":"0.20"
}
}
```

###Common interface - Access Depth

```
Obtain the spot depth list of the exchange
Speed limit rule: 5 times / 1 second
HTTP GET /api/v3/spot/instruments/depth?instrument_ id=BTC%2FUSDT&depth=5
```

Request parameters:

|Name | type | required | description|
| ------------- | ------ | -------- | ---------------------------- |
| instrument_ ID | string | is the name of a currency pair, such as BTC / usdt|
|Depth | string | is the depth gear with values of 5, 10, 50 and 100|

Return field description:

|Name | type | description|
| --------- | ------ | -------------------------- |
|Ask | array | seller depth|
|Bid | array | buyer depth|
| timestamp | string | |


```

Request:
Url: http://127.0.0.1 :8604/api/v3/spot/instruments/depth?instrument_ id=BTC%2FUSDT&depth=5
Method: GET
Headers:
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T07:49:31.732ZGET/api/v3/spot/instruments/depth?instrument_ id=BTC%2FUSDT&depth=5

Response:
{
"code":200,
"data":{
"asks":[
[
"36779.79",
"0.1237"
],
[
"36787.16",
"0.0474"
],
[
"36788.17",
"0.4454"
],
[
"36794.49",
"0.0229"
],
[
"36801.84",
"0.9221"
]
],
"bids":[
[
"36728.33",
"0.0217"
],
[
"36720.99",
"0.0221"
],
[
"36713.63",
"0.1577"
],
[
"36706.28",
"0.0876"
],
[
"36698.93",
"0.0163"
]
],
"timestamp":"2021-01-07T07:49:32.836Z"
}
}
```

###Public interface - get ticker list information

```
Obtain the latest transaction price, buy one price, sell one price and 24 trading volume of all tickers in the stock exchange
Speed limit rule: 5 times / 1 second
HTTP GET /api/v3/spot/instruments/ticker_ list
```

Request parameters: None


Return field description:

|Name | type | description|
| ----------------- | ------ | -------------------- |
| trade_ pair_ Name | string | currency pair name, such as BTC / usdt|
| last_ Price | string | latest price|
| lowest_ Ask | string | one price|
| highest_ Bid | string | buy a price|
| highest_ price_ 24h | string | 24h highest price|
| lowest_ price_ 24h | string | 24h lowest price|
|Volume24h | string | 24h trading volume|
|Chg24h | string | 24h rise and fall|
|Chg0h | string | 0h up and down|
|Amount24h | string | 24h trading volume|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/instruments/ticker_ list
Method: GET
Headers:
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T07:50:34.485ZGET/api/v3/spot/instruments/ticker_ list

Response:
{
"code":200,
"data":[
{
"trade_ pair_ name":"CPU/USDT",
"last_ price":"0.007600",
"highest_ bid":"0.000000",
"lowest_ ask":"0.000000",
"highest_ price_ 24h":"0.007600",
"lowest_ price_ 24h":"0.007600",
"volume24h":"0.000000",
"chg24h":"0.00%",
"chg0h":"0.00%",
"amount24h":"0.000000"
},
...
]
}
```



###Public interface - gets the specified ticker information

```
Obtain the latest transaction price, buy one price, sell one price and 24 trading volume of the designated ticker in the stock exchange
Speed limit rule: 5 times / 1 second
HTTP GET /api/v3/spot/instruments/ticker_ one?instrument_ id=BTC%2FUSDT
```

Request parameters: None

|Name | type | required | description|
| ------------- | ------ | -------- | -------------------- |
| instrument_ ID | string | is the name of a currency pair, such as BTC / usdt|

Return field description:

|Name | type | description|
| ----------------- | ------ | --------- |
| trade_ pair_ Name | string | coin pair name|
| last_ Price | string | latest price|
| lowest_ Ask | string | one price|
| highest_ Bid | string | buy a price|
| highest_ price_ 24h | string | 24h highest price|
| lowest_ price_ 24h | string | 24h lowest price|
|Volume24h | string | 24h trading volume|
|Chg24h | string | 24h rise and fall|
|Chg0h | string | 0h up and down|
|Amount24h | string | 24h trading volume|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/instruments/ticker_ one?instrument_ id=BTC%2FUSDT
Method: GET
Headers:
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T07:53:36.493ZGET/api/v3/spot/instruments/ticker_ one?instrument_ id=BTC%2FUSDT


Response:
{
"code":200,
"data":{
"trade_ pair_ name":"BTC/USDT",
"last_ price":"36849.13",
"highest_ bid":"36827.87",
"lowest_ ask":"36879.47",
"highest_ price_ 24h":"37686.16",
"lowest_ price_ 24h":"33630.27",
"volume24h":"4363928722.84",
"chg24h":"6.23%",
"chg0h":"6.58%",
"amount24h":"4363928722.84"
}
}
```

###Common interface - get K-line data

```
Obtain the spot K-line data. The maximum number of K-line data is 2000.
Speed limit rule: 5 times / 1 second
HTTP GET /api/v3/spot/instruments/candles?instrument_ id=BTC%2FUSDT&period=1&start_ time=&end_ time=
```

Request parameters:

|Name | type | required | description ||
| ------------- | ------ | -------- | -------------------------------- | ---- |
| instrument_ ID | string | is the name of | currency pair, such as btcusdt ||
| start_ Time | string | is the start time, and the iso8601 format timestamp is seconds|
| end_ Time | string | is the deadline. The iso8601 format timestamp is seconds|
|Period | string | is the particle size of | Kline. Please refer to the description for the value range|


```

The value of resolution can only be "1", "3", "5", "15", "30",
"60", "120", "240", "360", "720", "d", "W", "m"], otherwise the request will be rejected,
Corresponding to [1min, 3min, 5min, 15min, 30min,
1 hour, 2 hour, 4 hour, 6 hour, 12 hour, 1 day, 1 week, 1 month]
```


return:


```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/instruments/candles?instrument_ id=BTC%2FUSDT&period=1&start_ time=&end_ time=
Method: GET
Headers:
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T08:08:52.984ZGET/api/v3/spot/instruments/candles?instrument_ id=BTC%2FUSDT&period=1&start_ time=&end_ time=


Response:
Format Description: [timestamp, open, high, low, close, volume]
{
"code":200,
"data":[
[
"2021-01-07T08:08:00.000Z",
"36950.56",
"36996.13",
"36946.35",
"36980.25",
"42.5518"
],
[
"2021-01-07T08:07:00.000Z",
"37007.96",
"37007.96",
"36949.97",
"36950.56",
"87.8897"
],
...
]
}
```

###Public interface - query the latest transaction information

```
Obtain the latest transaction information of spot
Speed limit rule: 5 times / 1 second
HTTP GET /api/v3/spot/instruments/trade_ list?instrument_ id=BTC%2FUSDT
```

Request parameters:

|Name | type | required | description|
| ------------- | ------ | -------- | ------------------- |
| instrument_ ID | string | is the name of a currency pair, such as btcusdt|

Return field description:


```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/instruments/trade_ list?instrument_ id=BTC%2FUSDT
Method: GET
Headers:
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T08:13:34.233ZGET/api/v3/spot/instruments/trade_ list?instrument_ id=BTC%2FUSDT

Format Description: [trade]_ pair_ name,price,volume,side,timestamp]
Response:
{
"code":200,
"data":[
[
"BTC/USDT",
"36906.04",
"1.1161",
"sell",
"2021-01-07T08:13:34.000Z"
],
[
"BTC/USDT",
"36907.39",
"1.3790",
"buy",
"2021-01-07T08:13:29.000Z"
],
...
]
}
```

###Public interface - get legal currency and USD exchange rate

```
Get the exchange rate interface provided by the platform
Speed limit rule: 1 time / 1 second
HTTP GET /api/v3/spot/instruments/rate_ list
```

Request parameters:
nothing


Return field description:

|Name | type | description|
| --------- | ------ | ----------------- |
|Symbol | string | asset currency pair|
|Rate | string | to USD|
|Timestamp | string | request time, international time|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/instruments/rate_ list
Method: GET
Headers:
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T08:16:21.638ZGET/api/v3/spot/instruments/rate_ list

Response:
{
"code":200,
"data":[
{
"symbol":"USD_ USDT",
"rate":"0.9996",
"timestamp":"2021-01-05T03:57:08.859Z"
},
{
"symbol":"USD_ CNY",
"rate":"6.4570",
"timestamp":"2021-01-05T03:57:08.859Z"
},
{
"symbol":"USD_ KRW",
"rate":"1085.4800",
"timestamp":"2021-01-05T03:57:08.859Z"
},
{
"symbol":"USD_ JPY",
"rate":"103.1400",
"timestamp":"2021-01-05T03:57:08.859Z"
},
{
"symbol":"USD_ BRL",
"rate":"5.2968",
"timestamp":"2021-01-05T03:57:08.859Z"
},
{
"symbol":"USD_ ARP",
"rate":"84.5210",
"timestamp":"2021-01-05T03:57:08.859Z"
}
]
}

```



###Private interface - query all account information

```
Obtain all account information of user's spot assets
Speed limit times: 3 times / 1 second
HTTP GET /api/v3/spot/account/list
```

Request parameter none

Return result parameters

|Name | type | description|
| -------------- | ------ | -------- |
|Asset | string | asset name|
|Available | string | available balance|
| frozen_ Balance | string | freeze balance|
| total_ Balance | string | total|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/account/list
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: d7409a21176bac4a8f288165c11373e0180d9059b6a9c8ab57a451add0f78360
ACCESS-TIMESTAMP: 2021-01-07T08:42:27.987Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T08:42:27.987ZGET/api/v3/spot/account/list


Response:
{
"code":200,
"data":[
{
"asset":"MOAC",
"available":"9990.00000000",
"frozen_ balance":"8.30000000",
"total_ balance":"9998.30000000"
},
{
"asset":"LTC",
"available":"9964.99999733",
"frozen_ balance":"0",
"total_ balance":"9964.99999733"
},
...
]
}
```

###Private interface - query asset information of specified account

```
Obtain the account information of the asset specified by the spot user
Speed limit times: 6 times / 1 second
HTTP GET /api/v3/spot/account/one?asset=USDT
```

Request parameters

|Name | type | required | description|
| ----- | ------ | -------- | --------------------- |
|Asset | string | is the name / abbreviation of an asset, such as usdt|

Return result parameters

|Name | type | description|
| -------------- | ------ | --------------- |
|Asset | string | asset name / abbreviation|
|Available | string | available balance|
| frozen_ Balance | string | freeze balance|
| total_ Balance | string | total, frozen + balance|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/account/one?asset=USDT
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 2068804b1a7f4d7fd5b3830b3bd6e1135a90565b20888f9f7c6b04596ff4c64e
ACCESS-TIMESTAMP: 2021-01-07T08:43:57.794Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T08:43:57.794ZGET/api/v3/spot/account/one?asset=USDT

Response:
{
"code":200,
"data":{
"asset":"USDT",
"available":"1389259.41398473",
"frozen_ balance":"30712.71076533",
"total_ balance":"1419972.12475007"
}
}
```



###Private interface - order

```
Order according to user input
Speed limit rule: 5 times / 1 second
HTTP POST /api/v3/spot/order
```

Request parameters:

|Name | type | required | description|
| ------------- | ------ | -------- | ----------------------------------- |
| instrument_ ID | string | is the name of | currency pair, such as BTC / usdt, which is divided by '/'|
|Direction | string | is the direction, direction = 1: buy, direction = 2: sell|
|Price | string | is the order price|
|Quantity | string | is the entrusted quantity|

Return field description:

|Name | type | description|
| -------- | ------ | ------------ |
| order_ ID | string | generated order ID|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/order
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: c105a1e0302876df5fe7ba2d58ad469ac3258c9eb027acaa9101c583ae6a01bc
ACCESS-TIMESTAMP: 2021-01-07T08:51:22.580Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body: {"instrument_ id":"BTC/USDT","price":"37994.13","quantity":"1","direction":"2"}
preHash: 2021-01-07T08:51:22.580ZPOST/api/v3/spot/order{"instrument_ id":"BTC/USDT","price":"37994.13","quantity":"1","direction":"2"}


Response:
{
"code":200,
"data":{
"order_ id":"2120223552677564416"
}
}
```

###Private interface - batch order

```
Order in batches according to user input. Only price limit orders are supported
Speed limit rule: 3 times / 1 second
HTTP POST /api/v3/spot/batch_ order
```

The request parameter is an array object that contains the following parameters

|Name | type | required | description|
| ------------- | ------ | -------- | -------------------------------------------- |
| instrument_ ID | string | is the name of | currency pair, such as BTC / usdt, which is divided by '/'|
|Direction | string | yes | order direction = 1 means buy direction = 2 means sell direction|
|Price | string | is the order price|
|Quantity | string | is the entrusted quantity|

Return field description:

|Name | type | description|
| -------- | ------ | ------------ |
| order_ ID | string | generated order ID|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/batch_ order
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 78ad005f468b964ff4371ccb21ed7c37afc01b1dc335f49f4d9b704012ab63a8
ACCESS-TIMESTAMP: 2021-01-07T08:54:03.710Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body: [{"instrument_ id":"BTC/USDT","price":"37994.13","quantity":"2","direction":"2"},{"instrument_ id":"BTC/USDT","price":"38994.13","quantity":"1","direction":"1"}]
preHash: 2021-01-07T08:54:03.710ZPOST/api/v3/spot/batch_ order[{"instrument_ id":"BTC/USDT","price":"37994.13","quantity":"2","direction":"2"},{"instrument_ id":"BTC/USDT","price":"38994.13","quantity":"1","direction":"1"}]

Response:
{
"code":200,
"data":[
{
"order_ id":"2120224216451338240",
"code":"200",
"message":""
},
{
"order_ id":"2120224216547807232",
"code":"200",
"message":""
}
]
}
```

###Private interface - query the list of the current delegated orders

```
According to the user's request to query the current list of orders, orders that have not been closed or partially closed are in the current list of orders
Speed limit rule: 3 times / 1 second
HTTP GET /api/v3/spot/open_ orders?instrument_ id=BTC%2FUSDT&latestOrderId=
```

Request parameters:

|Name | type | required | description|
| ------------- | ------ | -------- | ------------------------------------------------------------ |
| instrument_ ID | string | is the name of a currency pair, such as BTC / usdt|
|Latestorderid | string | no | order ID, which is used in pagination. The default value is empty. The latest 20 pieces of data are returned and displayed in reverse order by order ID. Get the last order Id-1, take the next page of data|

```
explain:
Pagination query, each page returns 20
```

Return field description:

|Name | type | description|
| --------------- | ------ | ------------------------------------------------------------ |
| order_ ID | string | order ID|
| base_ Asset | string | transaction currency, such as BTC|
| quote_ Asset | string | valuation currency, such as usdt|
|Direction | string | single direction|
|Quantity | string | order quantity|
| filled_ Quantity | string | number of transactions|
|Amount | string | order amount|
| filled_ Amount | string | transaction amount|
| Trade_ pair_ Name | string | coin pair name|
|Price | string | order price|
|Status | string | order status, unsettled: open, complete, cancelled, partially cancelled|
| order_ Time | string | order time|
|Fee | string | handling fee|
| taker_ fee_ Rate | string | taker|
| taker_ fee_ Rate | string | maker|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/open_ orders?instrument_ id=BTC%2FUSDT&latestOrderId=
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 1956e0de1ba125dd64402809605ed1ec9cb7135bd067d3a2629a23e47144867f
ACCESS-TIMESTAMP: 2021-01-07T09:09:18.301Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T09:09:18.301ZGET/api/v3/spot/open_ orders?instrument_ id=BTC%2FUSDT&latestOrderId=


Response:
{
"code":200,
"data":[
{
"order_ id":"2120224216451338240",
"base_ asset":"BTC",
"quote_ asset":"USDT",
"direction":"sell",
"quantity":"2",
"filled_ quantity":"0",
"amount":"75988.26",
"filled_ amount":"0",
"average_ price":"",
"status":"Open",
"order_ time":"2021-01-07T08:54:05.000Z",
"update_ time":"2021-01-07T08:54:05.000Z",
"fee":"0",
"taker_ fee_ rate":"0.013",
"maker_ fee_ rate":"0.013",
"trade_ pair_ name":"BTC/USDT",
"price":"37994.13",
"order_ type":"limit"
},
{
"order_ id":"2120223552677564416",
"base_ asset":"BTC",
"quote_ asset":"USDT",
"direction":"sell",
"quantity":"1",
"filled_ quantity":"0",
"amount":"37994.13",
"filled_ amount":"0",
"average_ price":"",
"status":"Open",
"order_ time":"2021-01-07T08:51:27.000Z",
"update_ time":"2021-01-07T08:51:26.000Z",
"fee":"0",
"taker_ fee_ rate":"0.013",
"maker_ fee_ rate":"0.013",
"trade_ pair_ name":"BTC/USDT",
"price":"37994.13",
"order_ type":"limit"
},
...
]
}
```

###Private interface - query historical delegation list

```
Query the historical order list according to the user's request, and enter the historical order list only after the order is cancelled or completely closed
Speed limit rule: 3 times / 1 second
HTTP GET /api/v3/spot/closed_ orders?instrument_ id=BTC%2FUSDT&latestOrderId=
```

Request parameters:

|Name | type | required | description|
| ------------- | ------ | -------- | ------------------------------------------------------------ |
| instrument_ ID | string | is the name of a currency pair, such as BTC / usdt|
|Latestorderid | string | no | order ID, which is used in pagination. The default value is empty. The latest 20 pieces of data are returned and displayed in reverse order by order ID. Get the last order Id-1, take the next page of data|

```
explain:
Pagination query, each page returns 20
```

Return field description:

|Name | type | description|
| --------------- | ------ | ------------------------------------------------------------ |
| order_ ID | string | order ID|
| base_ Asset | string | transaction currency, such as BTC|
| quote_ Asset | string | valuation currency, such as usdt|
|Direction | string | single direction|
|Quantity | string | order quantity|
| filled_ Quantity | string | number of transactions|
|Amount | string | order amount|
| filled_ Amount | string | transaction amount|
| Trade_ pair_ Name | string | coin pair name|
|Price | string | order price|
|Status | string | order status, unsettled: open, complete, cancelled, partially cancelled|
| order_ Time | string | order time|
|Fee | string | handling fee|
| taker_ fee_ Rate | string | taker|
| taker_ fee_ Rate | string | maker|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/closed_ orders?instrument_ id=BTC%2FUSDT&latestOrderId=
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 780778cf6a75c62fb38e88860902dec50340eb95d2512f738f4a51cdce620910
ACCESS-TIMESTAMP: 2021-01-07T09:13:33.807Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T09:13:33.807ZGET/api/v3/spot/closed_ orders?instrument_ id=BTC%2FUSDT&latestOrderId=

Response:
{
"code":200,
"data":[
{
"order_ id":"2120224216547807232",
"base_ asset":"BTC",
"quote_ asset":"USDT",
"direction":"buy",
"quantity":"1",
"filled_ quantity":"1",
"amount":"38994.13",
"filled_ amount":"37284.246459",
"average_ price":"",
"status":"Filled",
"order_ time":"2021-01-07T08:54:05.000Z",
"update_ time":"2021-01-07T08:54:05.000Z",
"fee":"50.33373271965",
"taker_ fee_ rate":"0.013",
"maker_ fee_ rate":"0.013",
"trade_ pair_ name":"BTC/USDT",
"price":"38994.13",
"order_ type":"limit"
},
{
"order_ id":"2119464720976269312",
"base_ asset":"BTC",
"quote_ asset":"USDT",
"direction":"sell",
"quantity":"2",
"filled_ quantity":"2",
"amount":"55988.26",
"filled_ amount":"61713.968507",
"average_ price":"",
"status":"Filled",
"order_ time":"2021-01-05T06:36:07.000Z",
"update_ time":"2021-01-05T06:36:07.000Z",
"fee":"83.31385748445",
"taker_ fee_ rate":"0.013",
"maker_ fee_ rate":"0.013",
"trade_ pair_ name":"BTC/USDT",
"price":"27994.13",
"order_ type":"limit"
},
...
]
}
```

###Private interface - query specified order information

```
Query the specified order according to the user's request,
Speed limit rule: 6 times / 1 second
HTTP GET /api/v3/spot/order_ info?order_ id=2120224216451338240
```

Request parameters:

|Name | type | required | description|
| -------- | ------ | -------- | ------ |
| order_ ID | string | is the order ID|

Return field description:

|Name | type | description|
| --------------- | ------ | ------------------------------------------------------------ |
| order_ ID | string | order ID|
| base_ Asset | string | transaction currency, such as BTC|
| quote_ Asset | string | valuation currency, such as usdt|
|Direction | string | single direction|
|Quantity | string | order quantity|
|Amount | string | order amount|
| filled_ Amount | string | transaction amount|
| taker_ fee_ Rate | string | taker|
| maker_ fee_ Rate | string | maker|
| order_ Type | string | order type limit indicates price limit order|
|Price | string | order price|
|Status | string | order status, unsettled: open, complete, cancelled, partially cancelled|
| order_ Time | string | order time|
|Fee | string | handling fee|
| trade_ pair_ Name | string | coin pair name|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/order_ info?order_ id=2120224216451338240
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 91cd4bec3bcfe238c64a37f6c18fa7406a75b91ac3122991ce57d064deddc11f
ACCESS-TIMESTAMP: 2021-01-07T08:56:56.731Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body:
preHash: 2021-01-07T08:56:56.731ZGET/api/v3/spot/order_ info?order_ id=2120224216451338240

Response:
{
"code":200,
"data":{
"order_ id":"2120224216451338240",
"base_ asset":"BTC",
"quote_ asset":"USDT",
"direction":"sell",
"quantity":"2",
"filled_ quantity":"0",
"amount":"75988.26",
"filled_ amount":"0",
"status":"Open",
"order_ time":"2021-01-07T08:54:05.000Z",
"fee":"0",
"taker_ fee_ rate":"0.013",
"maker_ fee_ rate":"0.013",
"trade_ pair_ name":"BTC/USDT",
"price":"37994.13",
"order_ type":"limit"
}
}
```

###Private interface - revocation of specified delegation

```
Cancel the order according to the user's request,
Speed limit rule: 6 times / 1 second
HTTP POST /api/v3/spot/cancel_ order
```

Request parameters:

|Name | type | required | description|
| -------- | ------ | -------- | -------- |
| order_ ID | string | is the order ID|

Return field description:

|Name | type | description|
| -------- | ------ | ------------ |
| order_ ID | string | cancelled order ID|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/cancel_ order
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: c860762f24679b3f68869ff6ce02cfddfabe178f5ee41aade9f00107cf880831
ACCESS-TIMESTAMP: 2021-01-07T09:17:49.591Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body: {"order_ id":"2119464720976269312"}
preHash: 2021-01-07T09:17:49.591ZPOST/api/v3/spot/cancel_ order{"order_ id":"2119464720976269312"}

Response:
{
"code":200,
"data":{
"order_ id":"2120223552677564416"
}
}
```

###Private interface - batch revocation of delegation

```
Cancel orders in batch according to user's request,
Speed limit rule: 3 times / 1 second
HTTP POST /api/v3/spot/batch_ cancel_ order
```

Request parameters:

|Name | type | required | description|
| -------- | ---- | -------- | -------- |
|Orderids | list | is the order ID|

Return field description:

|Name | type | description|
| -------- | ------ | ------------ |
| order_ ID | string | cancelled order ID|

```
Request:
Url: http://127.0.0.1 :8604/api/v3/spot/batch_ cancel_ order
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: a6df76c43c41460f4dd5a3e7f4190a661d87eae70039c7571adbf8061b8de961
ACCESS-TIMESTAMP: 2021-01-07T09:19:56.090Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_ US
Body: {"orderIds":["2120230657572687872","2120230657669156864"]}
preHash: 2021-01-07T09:19:56.090ZPOST/api/v3/spot/batch_ cancel_ order{"orderIds":["2120230657572687872","2120230657669156864"]}

Response:
{
"code":200,
"data":[
{
"order_ id":"2120230657572687872",
"code":"200",
"message":""
},
{
"order_ id":"2120230657669156864",
"code":"51800",
"message":"The transaction has been traded, failure"
}
]
}
```

##Error code summary

|Error code | message|
| -------- | ------------------------------------------------------------ |
|429 | requests too frequent|
|430 | the asset does not support API user transactions for the time being|
| 10001 | "ACCESS_ "Key" cannot be empty|
| 10002 | "ACCESS_ "Sign" cannot be empty|
| 10003 | "ACCESS_ "Timestamp" cannot be empty|
|10005 | invalid access_ TIMESTAMP |
|10006 | invalid access_ KEY |
|10007 | invalid content_ Type, please use "application / JSON" format|
|10008 | request timestamp expired|
|10009 | system error|
|10010 | API verification failed|
|11000 | required parameter cannot be empty|
|11001 | parameter value error|
|11002 | parameter value exceeds maximum limit|
|11003 | no data returned from the third party interface|
|11004 | order price accuracy does not meet|
|11005 | the currency pair has not been leveraged yet|
|11007 | currency pair does not match assets|
|51800 | closed, cancellation failed|
|51801 | the order does not exist, cancellation failed|
|51802 | illegal transaction|
|51803 | the purchase price cannot be higher than the current price {0}%|
|51804 | the selling price should not be lower than the current price {0}%|
|51805 | maximum decimal place of commission price {0}|
|51806 | the largest number of decimal places of entrustment {0}|
|51807 | buy at least {0}|
|51808 | sell at least {0}|
|51809 | insufficient balance or frozen|
|51810 | suspension of sales, recovery time, subject to the announcement|
|51811 | user does not have transaction authority|
|51812 | the price of the current {0} hour cycle is {1}. The purchase price cannot be higher than the price of the limit. Please adjust the purchase price|
|51813 | the limit price of the current {0} hour cycle is {1}. The selling price cannot be lower than the limit price. Please adjust the selling price|
|51814 | plan delegation can only be cancelled when it is not triggered|
|51815 | delegate type error|
|51816 | wrong account type|
|51817 | trade to error|
|51818 | wrong trading direction|
|51819 | wrong delegation source|
|51820 | trigger price error|
|51821 | trigger price maximum decimal {0}|
|51822 | the purchase price cannot be higher than the trigger price {0}%|
|51823 | the selling price cannot be lower than {0}% of the trigger price|
|51824 | commission price error|
|51825 | total amount of entrustment error|
|51826 | the largest decimal place of total entrusted amount {0}|
|51827 | wrong entrusted quantity|
|51828 | the number of high-level entrustment orders cannot exceed {0}|
|51829 | trigger price should be higher than the latest transaction price|
|51830 | trigger price should be lower than the latest transaction price|
|51831 | price limit error|
|51832 | maximum decimal place of price limit {0}|
|51833 | price limit should be higher than the latest transaction price|
|51834 | price limit should be lower than the latest transaction price|
|51835 | no account found|
|51836 | commission does not exist|
|51837 | wrong order number|
|51838 | the batch order quantity cannot exceed {0}|
|51839 | account freeze failed|
|51840 | failed to query account|
|51841 | no trading limit set for trading pairs|
|51843 | failed to query price limit|
