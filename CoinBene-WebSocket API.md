# CoinBene-WebSocket API

- [CoinBene-WebSocket API](#coinbene-websocket-api)
  * [Overview](#overview)
  * [Command Format](#command-format)
    + [Subscription](#subscription)
    + [Unsubscription](#unsubscription)
  * [Connect limit](#connect-limit)
    + [Heartbeats](#heartbeats)
    + [The limits](#the-limits)
  * [Topics](#topics)
    + [Public Topics](#public-topics)
      - [OrderBook](#orderbook)
      - [Trade List](#trade-list)
      - [Ticker](#ticker)
      - [Kline](#kline)
  * [Error Codes](#error-codes)

## Overview

WebSocket protocol is a new HTML5 protocol, which provides full-duplex communication between web browsers and web servers. Connection can be established after one handshake. Web server can then push business logic data to web browsers.Advantages:

Request header is small in size (around 2 bytes) during communication Since there is no need to create and delete TCP connection repeatedly, it saves resources . 

**We strongly recommend developers to use Websocket API to access market related information and trading depth.**

The connection will break automatically when a network problem occurs

url: *wss://ws.coinbene.com/stream/ws*

## Command Format

Send format:

```json
{"op":"<value>","args": ["<value1>","<value2>"]}
```

The `op` support the following values

* `subscribe` 
* `unsubscribe` 

The `args` array,.

Successful response format:

```json
{"event": "<value>","topic":"<value>"}
{"topic":"<value>","data":"[<value1>,<value2>]"}
```

In the orderBook topic, the return formats for distinguishing between the first full amount and the subsequent incremental are:

```json
{"topic":"<value>","action":"<value>","data":["<value1>","<value2>"]}
```

Failure response format:

```json
{"event":"<value>","message":"<errorMessage>","code":"<errorCode>"}
```



### Subscription

Users may subscribe to one or more topics

The format:

```json
{"op":"subscribe", "args":["<topic>"]}
```

the value of `op` is `subscribe`

`args` the array content is topic names

Example:

```json
// send
{"op":"subscribe","args":["spot/orderBook.BTCUSDT.10","spot/tradeList.BTCUSDT"]}

// response
{"event":"subscribe","topic":"spot/orderBook.BTCUSDT.10"}
{"event":"subscribe","topic":"spot/tradeList.BTCUSDT"}
```

### Unsubscription

Users can cancel one or more topics

The format:

```json
{"op":"unsubscribe","args": ["<topic>"]}
```

Example:

```json
// send
{"op":"unsubscribe","args":["spot/orderBook.BTCUSDT.10","spot/tradeList.BTCUSDT"]}

// response
{"event":"unsubscribe","topic":"spot/orderBook.BTCUSDT.10"}
{"event":"unsubscribe","topic":"spot/tradeList.BTCUSDT"}
```



## Connect limit

### Heartbeats

After connected to CoinBene's Websocket server, the server will send heartbeat periodically (currently at 5s interval) , The literal string `'ping'`. When client receives this heartbeat message, it should response with a literal string `'pong'`.

> After the server sent two consecutive heartbeat messages without receiving at least one  "pong" response from a client, then right before server sends the next "ping" heartbeat, the server will disconnect this client

### The limits

**Connection limit**：1 times/s

**Subscription limit**：240 times/hour

## Topics

Public topic list (don't require login)

| Description             | Format                          | Example                   |
| ----------------------- | ------------------------------- | ------------------------- |
| depth information       | spot/orderBook.{symbol}.{depth} | spot/orderBook.BTCUSDT.10 |
| trade information       | spot/tradeList.{symbol}         | spot/tradeList.BTCUSDT    |
| ticker information      | spot/ticker.{symbol}            | spot/ticker.BTCUSDT       |
| 1mins kline information | spot/kline.{symbol}             | spot/kline.BTCUSDT        |



### Public Topics

#### OrderBook

Format: `spot/orderBook.{symbol}.{depth}`

*`symbol`*: contract name
*`depth`* : support 5、10、50、100

Example:

```json
// send 
{"op": "subscribe", "args": ["spot/orderBook.BTCUSDT.10"]}

//response
// full data
{
    "topic": "spot/orderBook.BTCUSDT", 
    "action": "insert",
    "data": [{
        "asks": [
            ["5621.7", "58"], 
            ["5621.8", "125"],
            ["5621.9", "100"],
            ["5622", "84"],
            ["5623.5", "90"],
            ["5624.2", "1540"],
            ["5625.1", "300"],
            ["5625.9", "350"],
            ["5629.3", "200"],
            ["5650", "1000"]
        ],
        "bids": [
            ["5621.3", "287"],
            ["5621.2", "41"],
            ["5621.1", "2"],
            ["5621", "26"],
            ["5620.8", "194"],
            ["5620", "2"],
            ["5618.8", "204"],
            ["5618.4", "30"],
            ["5617.2", "2"],
            ["5609.9", "100"]
        ],
        "version":1,
        "timestamp": 1584412740809
    }]
 }
 // incremental data
{
    "topic": "spot/orderBook.BTCUSDT", 
    "action": "update", 
    "data": [{
        "asks": [
            ["5621.7", "50"],
            ["5621.8", "0"],
            ["5621.9", "30"]
        ],
        "bids": [
            ["5621.3", "10"],
            ["5621.2", "20"],
            ["5621.1", "80"],
            ["5621", "0"],
            ["5620.8", "10"]
        ],
        "version":2,
        "timestamp": 1584412740809
    }]
 }
```

> Full data `action = insert`   incremental data `action = update`
> ["5621.3", "10"] [String,String] 5621.3 is the price, 10 is the quantity of the price.
> version   The data version is strictly incremented. The client can judge whether the data is continuous according to the version.

| Parameter | Parameter Type | Description       |
| --------- | -------------- | ----------------- |
| symbol    | string         | name              |
| asks      | string         | Selling side      |
| bids      | string         | Buying side       |
| version   | number         | Data version      |
| timestamp | string         | System time stamp |

* manage a local order book correctly 

  > 1. subscribe `orderBook.{symbol}.{depth}` get full data first.
  > 2. receive incremental data resolve as following rule.
  >     * If price level not exists insert into order book
  >     * If price level exists and new quantity not equals 0, replace old.
  >     * If the quantity is 0, remove the price level

#### Trade List

topic format: `spot/tradeList.{symbol}`

Example

```json
// send
{"op": "subscribe", "args": ["spot/tradeList.BTCUSDT"]}

// response
{
    "topic": "spot/tradeList.BTCUSDT",
    "data": [  
      [
        "8600.0000", 
        "s", 
        "100", 
        1584412740809
      ]
    ]
 }
// array [price,side,quantity,timestamp]
```

返回参数

| Parameter | Parameter Type | Description                             |
| --------- | -------------- | --------------------------------------- |
| price     | string         | Filled price                            |
| quantity  | string         | Filled quantity                         |
| side      | string         | Filled side (b/s)    s: taker sell base |
| timestamp | number         | Filled time                             |



#### Ticker

To capture the latest traded price, best-bid price, best-ask price, and 24-hour trading volume of  contracts on the platform.

topic format: `spot/ticker.{symbol}`

Example

```json
// send
{"op": "subscribe", "args": ["spot/ticker.BTCUSDT","spot/ticker.ETHUSDT"]}

// response
{
    "topic": "spot/ticker.BTCUSDT",
    "data": [
        {
          "symbol": "BTCUSDT",
          "lastPrice": "8548.0", 
          "bestAskPrice": "8601.0", 
          "bestBidPrice": "8600.0",
          "bestAskVolume": "1222", 
          "bestBidVolume": "56505",
          "high24h": "8600.0000", 
          "low24h": "242.4500", 
          "volume24h": "4994", 
          "timestamp": 1584412736365
        }
    ]
 }
```



| Parameter    | Parameter Type | Description            |
| ------------ | -------------- | ---------------------- |
| symbol       | string         | trade pair name                   |
| bestAskPrice | string         | Best ask price         |
| bestAskSize  | string         | Best ask quantity      |
| bestBidPrice | string         | Best bid price         |
| bestBidSize  | string         | Best bid quantity      |
| lastPrice    | string         | Last traded price      |
| high24h      | string         | 24h max trade price              |
| low24h       | string         | 24 hour low price      |
| volume24h    | string         | 24 hour trading volume |
| timestamp    | string         | timestamp  |

#### Kline

1mins kline information

topic format: `spot/kline.{symbol}`

Example:

```json
// send
{"op": "subscribe", "args": ["spot/kline.BTCUSDT","spot/kline.ETHUSDT"]}

// response
{
    "topic": "spot/kline.BTCUSDT",
    "data": [
        {
          "c": 7513.01,
          "h": 7513.37,
          "l": 7510.02,
          "o": 7510.24,
          "m": 7512.03,
          "v": 60.5929,
          "t": 1578278880
        }
    ]
 }
```



| Parameter | Parameter Type | Description      |
| --------- | -------------- | ---------------- |
| c         | number         | Close price      |
| h         | number         | Highest price    |
| l         | number         | Lowest price     |
| o         | number         | Open price       |
| v         | number         | Trading quantity |
| t         | number         | Create time      |

## Error Codes

| Code  | Description             |
| ----- | :---------------------- |
| 429   | API rate limit exceeded |
| 10501 | ping check timeout      |
| 10502 | unrecognized request    |
| 10503 | unsupported topic       |
| 10504 | user not logged in      |
| 10505 | invalid signature       |
| 10506 | invalid args            |
| 10507 | invalid expires         |
| 10508 | Invalid apiKey          |
| 10509 | unsupported depth       |
| 10500 | Internal system error   |
