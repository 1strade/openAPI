1strade trading platform official API documentation
==================================================
[1strade][]Trading platform developer documentation([English Docs][])。
<!-- TOC -->
- [Introduction ](#Introduction )
- [Start to use](#Start to use)
- [API interface encryption verification](#api interface encryption verification)
    - [Generate API Key](#Generate api-key)
    - [Propose request](#Propose request)
    - [Signature](#Signature)
    - [Select timestamp](#Select timestamp)
    - [Request interaction](#Request interaction)
        - [Request](#Request)
    - [Standard](#Standard)
        - [Timestamp](#Timestamp)
        - [Examples](#Examples)
        - [Number](#Number)
        - [Current limit](#Current limit)
                - [REST API](#rest-api)
- [Spot business API reference](#Spot business api reference)
    - [Cryptos market API](#Cryptos market api)
        - [1. Get a list of all pairs](#1-Get a list of all pairs)
        - [2. Get trading depth list of trading pairs](#2-Get trading depth list of trading pairs)
        - [3. Get pairs Ticker](#3-Get pairs ticker)
        - [4. Get trading history of trading pairs](#4-Get trading history of trading pairs)
        - [5. Get K-Line data](#5-Get K-Line data)
        - [6. Get server time](#6-Get server time)
    - [Crypto account API](#Crypto account api)
        - [1. Get account information](#1-Get account information)
        - [2. Orders](#2-Orders)
        - [3. Cancel all orders](#3-Cancel all orders)
        - [4. Cancel orders](#4-按订单撤销委托)
        - [5. Query all orders](#5-Query all orders])
        - [6. Get bills](#6-Get bills)
<!-- /TOC -->
# Introduction 
Welcome to use[1strade][]developer documentation.
This document provides an introduction of 1strade crypto-crypto trading service API usage methods such as market inquiry, trading, and account management .
The market API is a public interface that provides market data of the crypto trading market; the trading and account API require identity authentication for functions such as order placing, order cancellation and account information query .
# Start to use   
REST，the abbreviation of Representational State Transfer，is a kind of popular  internet transfer architecture featured with clear and standard structure ,scalablity and easy operation .The advantages are as follows: 
+ In a RESTful architecture, each URL represents a kind of resource;
+ A layer of presentation of such resources between the client end and the server;
+ The client end operates on the server-side resources through four HTTP commands to realize  "presentation layer state conversion".
Developers are advised to use the REST API for market queries, crypto trading, and account management,etc.
# API interface encryption verification
## Generate API KEY
Before signing any request, you must generate  an API KEY via the 1strade website [User Center] - [API]. After generating the API KEY, you will get 3 pieces of information that you must remember:
* API Key
* Secret Key
* Passphrase
API Key and Secret Key will be generated randomly.Passphrase will be set by users.
## Propose request
All REST requests must contain the following title：
* ACCESS-KEY API Key as a string
* ACCESS-SIGN Use base64 encoding signature (refer to signature message)
* ACCESS-TIMESTAMP As the timestamp of your request
* ACCESS-PASSPHRASE Passphrase that you set when generating  the API KEY
* All requests should contain application/json type content and it should be valid JSON.
## 签名
ACCESS-SIGN的请求头是对 **timestamp + method + requestPath + "?" + queryString + body** 字符串(+表示字符串连接)使用**HMAC SHA256**方法加密，通过**BASE64**编码输出而得到的。其中，timestamp的值与ACCESS-TIMESTAMP请求头相同。
* method是请求方法(POST/GET/PUT/DELETE)，字母全部大写
* requestPath是请求接口路径
* queryString是GET请求中的查询字符串
* body是指请求主体的字符串，如果请求没有主体(通常为GET请求)，则body可省略
**例如：对于如下的请求参数进行签名**
* 获取深度信息，以LTC_BTC为例
```java
Timestamp = 1540286290170 
Method = "GET"
requestPath = "/openapi/exchange/public/LTC_BTC/orderBook"
queryString= "?size=100"
```
生成待签名的字符串
```java
Message = '1540286290170GET/openapi/exchange/public/LTC_BTC/orderBook?size=100'  
```
* 下单，以LTC_BTC为例
```java
Timestamp = 1540286476248 
Method = "POST"
requestPath = "/openapi/exchange/LTC_BTC/orders"
body = {"price":"1","side":"buy","source":"web","systemOrderType":"limit","volume":"1"}
```
生成待签名的字符串
```java
Message = '1540286476248POST/openapi/exchange/LTC_BTC/orders{"price":"1","side":"buy","source":"web","systemOrderType":"limit","volume":"1"}'  
```
然后，将待签名字符串添加私钥参数生成最终待签名字符串
```java
hmac = hmac(secretkey, Message, SHA256)
```
在使用前需要对于hmac进行base64编码
```java
Signature = base64.encode(hmac.digest())
```
## 请求交互  
REST访问的根URL：`https://www.1strade.co`
### 请求
所有请求基于Https协议，请求头信息中Content-Type需要统一设置为:'application/json’。
**请求交互说明**
1、请求参数：根据接口请求参数规定进行参数封装。
2、提交请求参数：将封装好的请求参数通过POST/GET/DELETE等方式提交至服务器。
3、服务器响应：服务器首先对用户请求数据进行参数安全校验，通过校验后根据业务逻辑将响应数据以JSON格式返回给用户。
4、数据处理：对服务器响应数据进行处理。
**成功**
HTTP状态码200表示成功响应，并可能包含内容。如果响应含有内容，则将显示在相应的返回内容里面。
**常见错误码**
* 400 Bad Request – Invalid request forma 请求格式无效
* 401 Unauthorized – Invalid API Key 无效的API Key
* 403 Forbidden – You do not have access to the requested resource 请求无权限
* 404 Not Found – 没有找到请求
* 429 Too Many Requests – 请求太频繁被系统限流
* 500 Internal Server Error – We had a problem with our server 服务器内部错误
如果失败，Response body带有错误描述信息
## 标准规范
### 时间戳
除非另外指定，API中的所有时间戳均以微秒为单位返回。
请求签名中的ACCESS-TIMESTAMP的单位是秒，允许用小数表示更精确的时间。请求的时间戳必须在API服务时间的30秒内，否则请求将被视为过期并被拒绝。如果本地服务器时间和API服务器时间之间存在较大的偏差，那么我们建议您使用通过查询API服务器时间来更新Http Header。
### 例子
`1524801032573`
### 数字
为了保持跨平台时精度的完整性，十进制数字作为字符串返回。建议您在发起请求时也将数字转换为字符串以避免截断和精度错误。 
整数（如交易编号和顺序）不加引号。
### 限流
如果请求过于频繁系统将自动限制请求，并在Http Header中返回429 too many requests状态码。
##### REST API
* 公共接口：我们通过IP限制公共接口的调用：每2秒最多6个请求。
* 私人接口：我们通过用户ID限制私人接口的调用：每2秒最多6个请求。
* 某些接口的特殊限制在具体的接口上注明
# 币币交易(Spot)API参考
## 币币行情API
### 1. 获取所有币对列表
**请求**
```http
    # Request
    GET /openapi/exchange/public/currencies
```
**响应**
```javascript
    # Response
    [{
    	"baseIncrement": 0,
    	"baseSymbol": "BTC",
    	"makerFeesRate": "0",
    	"maxPrice": 4,
    	"maxVolume": 4,
    	"minTrade": 0.00001000,
    	"online": 0,
    	"pairCode": "BTC_USDT",
    	"quoteIncrement": 0,
    	"quotePrecision": 0,
    	"quoteSymbol": "USDT",
    	"sort": 1,
    	"tickerFeesRate": "0"
    }, {
    	"baseIncrement": 0,
    	"baseSymbol": "ETH",
    	"makerFeesRate": "0",
    	"maxPrice": 4,
    	"maxVolume": 4,
    	"minTrade": 0.01000000,
    	"online": 0,
    	"pairCode": "ETH_USDT",
    	"quoteIncrement": 0,
    	"quotePrecision": 0,
    	"quoteSymbol": "USDT",
    	"sort": 2,
    	"tickerFeesRate": "0"
    },...]
```
**返回值说明**  

|返回字段 | 字段说明|
| ----------|:-------:|
| baseIncrement | 交易数量最小交易变动单位 |
| baseSymbol    | 交易货币 |
| makerFeesRate | maker 费率 |
| maxPrice  | 交易价格小数位数 |
| maxVolume | 交易数量小数位数 |
| minTrade | 最小委托量 |
| online | 是否上线 |
| pairCode | 是Base和quote之间的组合 BTC_USD |
| quoteIncrement | 最小交易单位 |
| quotePrecision | 计价货币数量单位精度 |
| quoteSymbol | 计价货币 |
| sort | 排序值 |
| tickerFeesRate | ticker 费率 |

### 2. 获取币对交易深度列表
**请求**
```http
    # Request
    GET /openapi/exchange/public/{pairCode}/orderBook
```
**响应**
```javascript
    # Response
    {
        "asks":[
            [
                "10463.3399",
                "0.0025"
            ],
            ...
        ],
        "bids":[
            [
                "7300.2456",
                "0.0022"
            ],
            ...
        ]
    }
```
**返回值说明**  

|返回字段|字段说明|
|--------| :-------: |
|asks| 卖方深度 |
|bids| 买方深度 |

**请求参数**  

| 参数名 | 参数类型  | 必填 | 描述 |
| ------------- |----|----|----|
| pairCode | String | 是 | 币对，如LTC_BTC |

### 3. 获取币对Ticker
最新成交价、买一价、卖一价、24h最高价、24h最低价、24h开盘价和24h成交量的快照信息。
**请求**
```http
    # Request
    GET /openapi/exchange/public/{pairCode}/ticker
```
**响应**
```javascript
    # Response
    {
    	"buy": "9512.70000000",
    	"change24": "19.60000000",
    	"changePercentage": "",
    	"changeRate24": "0.0020000000000000",
    	"close": "",
    	"createOn": 1564404929000,
    	"high": "9729.50000000",
    	"high24": "9729.50000000",
    	"last": "9525.20000000",
    	"low": "9171.80000000",
    	"low24": "9171.80000000",
    	"open": "9505.60000000",
    	"pairCode": "BTC_USDT",
    	"quoteVolume": "39140860.67498000",
    	"sell": "9515.60000000",
    	"volume": "4101.34040000"
    }
```
**返回值说明**
 
|返回字段|字段说明|
|--------| :-------: |
|buy| 最新买入价 |
|change24| 24小时变化值 |
|changePercentage| 变化百分比 |
|changeRate24| 24小时涨跌比例 |
|close| 24小时 close |
|createOn| 创建时间 |
|high| 最高成交价 |
|high24| 24小时最高成交价 |
|last| 最新成交价 |
|low| 最低成交价 |
|low24| 24小时最低成交价 |
|open| 24小时 open |
|pairCode| 币对信息 |
|quoteVolume| 计价币的成交量 |
|sell| 最新卖出价 |
|volume| 基准币的成交量 |
    
**请求参数**

|参数名|参数类型|必填|描述|
|------|----|:---:|:---:|
|pairCode|String|是|币对，如ETH_BTC|
    
### 4. 获取币对历史成交记录，支持分页查询
获取所请求交易对的历史成交信息，该请求支持分页。
**请求**
```http
    # Request
    GET /openapi/exchange/{pairCode}/fulfillment
```
**响应**
```javascript
    # Response
    [
        [
            	"id": 1524801032573,
				"pairCode": "BTC_USDT",
				"userId": 1001,		
				"brokerId": 10000,		
				"side": "buy",
				"entrustPrice": "1",
				"amount": "1",
				"dealAmount": "1",
				"quoteAmount": "1",
				"dealQuoteAmount": "1",
				"systemOrderType": "limit",
				"status": 0,
				"sourceInfo": "web",
				"createOn": 1524801032573,
				"updateOn": 1524801032573,
				"symbol": "BTC",
				"trunOver": "1",
				"notStrike": "0",
				"averagePrice": "1",
				"openAmount": "1"
        ],
        ...
    ]
```
**返回值说明**

|返回字段|字段说明|
|--------|----|
| id |订单id|
| pairCode |是Base和quote之间的组合 BTC_USD|
| userId |用户id|
| brokerId |券商id|
| side |方向 买、卖|
| entrustPrice |下单价格|
| amount |下单数量|
| dealAmount |成交数量|
| quoteAmount |基准币数量  只有在市价买的情况下会用到|
| dealQuoteAmount |基准币已成交数量|
| systemOrderType |10:限价 11:市价|
| status |0:未成交 1:部分成交 2:完全成交 3:撤单中 -1:已撤单|
| sourceInfo |下单来源 web,api,Ios,android|
| createOn |创建时间|
| updateOn |修改时间|
| symbol |币种|
| trunOver |成交金额  dealQuoteAmount * dealAmount|
| notStrike |尚未成交的数量|
| averagePrice |平均成交价|
| openAmount |下单数量|

**请求参数**

|参数名|参数类型|必填|描述|
|-----|:---:|----|----|
|pairCode|String|是|币对，如LTC_BTC|
|startDate|Long|否|开始时间，如1524801032573|
|endDate|Long|否|结束时间，如1524801032573|
|systemOrderType|Integer|否|10:限价 11:市价|
|price|BigDecimal|否|价格|
|amount|BigDecimal|否|数量|
|source|String|否|币对，如LTC_BTCweb,api,Ios,android|
|isHistory|Boolean|否|是否查历史数据、一周前成交的数据是历史数据、默认false|
|page|Integer|否|第几页|
|pageSize|Integer|否|每页条数|

**解释说明**
+ 交易方向 side 表示每一笔成交订单中 maker 下单方向,maker 是指将订单挂在订单深度列表上的交易用户，即被动成交方。
+ buy 代表行情下跌，因为 maker 是买单，maker 的买单被成交，所以价格下跌；相反的情况下，sell代表行情上涨，因为此时maker是卖单，卖单被成交，表示上涨。

### 5. 获取K线数据
**请求**
```http
    # Request
    GET  /openapi/exchange/public/{pairCode}/candles?interval=1min&start=start_time&end=end_time
```
**响应**
    
```javascript
    # Response
    {
        [ 1415398768, 0.32, 0.42, 0.36, 0.41, 12.3 ]
        ...
    }
```
**返回值说明（按顺序）**  
    
|返回字段|字段说明|
|-----|----|
|1415398768|K线开始时间戳|
|0.32|最低价|
|0.42|最高价|
|0.36|开盘价（第一笔交易）|
|0.41|收盘价（最后一笔交易）|
|12.3|交易量（按交易币统计）|
**请求参数**
    
|参数名|参数类型|必填|描述|
|-----|----|----|----|
|pairCode|String|是|币对如btc_usdt|
|interval|String|是|K线周期类型如1min/1hour/day/week/month|
|start|String|否|基于ISO 8601标准的开始时间|
|end|String|否|基于ISO 8601标准的结束时间|

### 6. 获取服务器时间
获取API服务器的时间的接口。
**请求**
```http
    # Request
    
    GET /openapi/exchange/public/time
```
**响应**
    
```javascript
    # Reponse
{
        "epoch": "1524801032.573"
        "iso": "2015-01-07T23:47:25.201Z",
        "timestamp": 1524801032573
    }
```
    
**返回值说明**
    
|返回字段|字段说明|
|-----|----|
|epoch|以秒为时间戳形式表达的服务器时间|
|iso|为ISO 8061标准的时间字符串表达的服务器时间|
|timestamp|以毫秒为时间戳形式表达的服务器时间|

## 币币账户API
### 1. 获取账户信息
获取币币交易账户余额列表，查询各币种的余额，冻结和可用情况。
**请求**
```
    # Request
    GET /openapi/exchange/assets
```
**响应**
```
    # Response
    [
        {
            "brokerId":10000,
            "symbol":"BTC",
            "available":"1",
            "hold":"0",
            "baseBTC":1,
            "withdrawLimit":"1",
        },
        ...
    ]
```
**返回值说明**

|返回字段|字段说明|
|----|----|
|brokerId|券商id|
|symbol|币种|
|available|余额|
|hold|冻结|
|baseBTC|折合BTC|
|withdrawLimit|提币限额|

### 2. 交易委托
提供限价和市价两种订单类型。
**请求**
```
    # Request
    POST /openapi/exchange/{pairCode}/orders
```
**响应**
```javascript
    # Response
    10000
```   
**返回值说明**
订单id

**请求参数**

|参数名| 参数类型 |必填|描述|
|:----:|:----:|:---:|----|
|pairCode|String|是|币对，如BTC_USDT|
|side|String|是|买入为buy，卖出为sell|
|systemOrderType|String|是|限价委托为limit，市价委托为market|
|volume|String|否|限价委托以及市价卖出时传递，代表交易币的数量|
|price|String|否|限价委托时传递，代表交易价格|
|quoteVolume|String|否|市价买入时传递，代表计价币的数量|

### 3. 撤销所有委托
撤销目标币对下所有未成交委托，最多撤销50条。由于是异步撤单，所以该接口没有返回值。
**请求**
```
    # Request
    DELETE /openapi/exchange/{pairCode}/cancel-all
```
**响应**
```javascript
    # Response
    { ...}
```
**请求参数**

|参数名|参数类型|必填|描述|
|----|----| ----| ----|
|pairCode|String|是|币对， 如BTC_USDT|
目前批量撤销200个挂单

### 4. 按订单撤销委托
按照订单id撤销指定订单。由于是异步撤单，所以该接口没有返回值。
**请求**
```http
    # Request
    DELETE /openapi/exchange/{pairCode}/orders/{id}
```
**响应**
```javascript
    # Response
    {...}
```
**请求参数**

|参数名|参数类型|必填|描述|
|---|----|----|----|
|code|String|是|币对，如BTC_USDT|
|orderId|String|是|需要撤销的未成交委托的id|

### 5. 查询订单，支持分页查询
按照订单状态查询订单。
    
**请求**
```http   
    # Request
    GET /openapi/exchange/orders
```
**响应**
```javascript
    # Response
    [
        [
            	"id": 1524801032573,
				"pairCode": "BTC_USDT",
				"userId": 1001,		
				"brokerId": 10000,		
				"side": "buy",
				"entrustPrice": "1",
				"amount": "1",
				"dealAmount": "1",
				"quoteAmount": "1",
				"dealQuoteAmount": "1",
				"systemOrderType": "limit",
				"status": 0,
				"sourceInfo": "web",
				"createOn": 1524801032573,
				"updateOn": 1524801032573,
				"symbol": "BTC",
				"trunOver": "1",
				"notStrike": "0",
				"averagePrice": "1",
				"openAmount": "1"
        ],
        ...
    ]
```
**返回值说明**

|返回字段|字段说明|
|--------|----|
| id |订单id|
| pairCode |是Base和quote之间的组合 BTC_USD|
| userId |用户id|
| brokerId |券商id|
| side |方向 买、卖|
| entrustPrice |下单价格|
| amount |下单数量|
| dealAmount |成交数量|
| quoteAmount |基准币数量  只有在市价买的情况下会用到|
| dealQuoteAmount |基准币已成交数量|
| systemOrderType |10:限价 11:市价|
| status |0:未成交 1:部分成交 2:完全成交 3:撤单中 -1:已撤单|
| sourceInfo |下单来源 web,api,Ios,android|
| createOn |创建时间|
| updateOn |修改时间|
| symbol |币种|
| trunOver |成交金额  dealQuoteAmount * dealAmount|
| notStrike |尚未成交的数量|
| averagePrice |平均成交价|
| openAmount |下单数量|

**请求参数**

|参数名 | 参数类型 | 必填 | 描述 |
|---|----|----|----|
|pairCode|String|否|币对，如BTC_USDT|
|startDate|Long|否|开始时间 毫秒|
|endDate|Long|否|结束时间 毫秒|
|price|BigDecimal|否|下单价格|
|amount|BigDecimal|否|下单数量|
|systemOrderType|Integer|否|10:限价 11:市价|
|source|String|否|币对，如LTC_BTCweb,api,Ios,android|
|page|Integer|否|第几页|
|pageSize|Integer|否|每页条数|

### 6. 获取账单，支持分页查询
获取币币交易账单。
**请求**
```http
    # Request
    GET /openapi/exchange/bills
```
**响应**
```javascript
    # Response
    {
    	"code": 0,
    	"data": {
    		"bills": [{
    				"amount": 1.6,
    				"assets": "51.44",
    				"brokerId": 0,
    				"createOn": 1552636850000,
    				"fee": -0.16,
    				"id": 0,
    				"makerTaker": 0,
    				"referId": 0,
    				"symbol": "",
    				"tradeNo": "",
    				"type": 7,
    				"updateOn": 0,
    				"userId": 0
    			},
    			...
    		],
    		"paginate": {
    			"page": 1,
    			"pageSize": 10,
    			"total": 0
    		}
    	},
    	"msg": "success"
    }
```
**返回值说明**

|返回字段 | 字段说明 |
|----|----|
|amount|变动数量|
|balance|变动后余额|
|createdDate|账单时间戳|
|details|详情|
|orderId|对应订单ID|
|code|订单对应币对，如BTC_USDT|
|id|账单ID|
|type|交易类型|

**请求参数**  

|参数名|参数类型|必填|描述|
|----|---|---|---|
|currencyCode|String|是|币种，如BTC|
|limit|Integer|否|请求返回数据量，默认/最大值为100|
  
[1strade]: https://www.1strade.co 
[English Docs]: https://github.com/1strade/openAPI/blob/master/README_EN.md

