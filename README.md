# Quotation-system-development
## 行情中心
<img src="https://github.com/SelenaMa9812/Quotation_system_development/blob/main/images/%E8%A1%8C%E6%83%85%E4%B8%AD%E5%BF%83.jpg" width="600" height="400" />

## cryptofeed开源框架
```Python
from cryptofeed.callback import TickerCallback, TradeCallback, BookCallback, FundingCallback
from cryptofeed import FeedHandler
from cryptofeed.exchanges import Coinbase
from cryptofeed.defines import TRADES, TICKER

async def ticker(feed, symbol, bid, ask, timestamp, receipt_timestamp):
    print(f'Timestamp: {timestamp} Feed: {feed} Pair: {symbol} Bid: {bid} Ask: {ask}')

async def trade(feed, symbol, order_id, timestamp, side, amount, price, receipt_timestamp):
    print(f"Timestamp: {timestamp} Feed: {feed} Pair: {symbol} ID: {order_id} Side: {side} Amount: {amount} Price: {price}")

def main():
    f = FeedHandler()
    f.add_feed(Coinbase(symbols=['BTC-USD'], channels=[TRADES, TICKER], callbacks={TICKER: TickerCallback(ticker), TRADES: TradeCallback(trade)}))

    f.run()

if __name__ == '__main__':
    main()
```

### 1.Feedhandler
#### 可以连接的交易所
.add_feed() 可以添加下面任意交易所(一次调用只能添加一个交易所)，feed列表即为已添加的交易所的列表.

>`AscendEX`   `Bequant`   `Bitcoin.com`   `Bitfinex`    `bitFlyer`   `Bithumb`   `Bitstamp`    `Bittrex`   `Blockchain.com`
>
>`Bybit Binance`   `Binance Delivery`    `Binance Futures`    `Binance US`    `BitMEX`    `Coinbase`    `Deribit`   
>
>`dYdX`    `EXX`(sometimes a connection can take many many retries)   `FTX`   `FTX US`      `Gate.io`   `Gemini`  
>
>`HitBTC`    `Huobi`   `Huobi DM`    `Huobi Swap`    `Kraken`    `Kraken Futures`    `KuCoin`(requires an API key for L2 book data)
>
>`OKCoin`    `OKEx`    `Phemex`    `Poloniex`    `ProBit`    `Upbit`

.run() 启动 Feedhandler。注意 Feedhandler 使用 Asyncio 模块，主线程已经运行了 Feedhandler 的时候运行 run() 会造成主线程阻塞(debug 方法：run() 添加参数 start_loop = False，feedhandler 不会启动，用户可以添加更多的任务/协程，然后负责稍后启动事件循环).

```Python
def main():
    config = {'log': {'filename': 'demo.log', 'level': 'INFO'}}
    # the config will be automatically passed into any exchanges set up by string. Instantiated exchange objects would need to pass the config in manually.
    f = FeedHandler(config=config)
    
    # 示例(可通过config文件自动化输入，以下示例均为手动化配置)
    f.add_feed(KuCoin(symbols=['BTC-USDT', 'ETH-USDT'], channels=[L2_BOOK, ], callbacks={L2_BOOK: book, BOOK_DELTA: delta, CANDLES: candle_callback, TICKER: ticker, TRADES: trade}))
    f.add_feed(Gateio(symbols=['BTC-USDT', 'ETH-USDT'], channels=[L2_BOOK], callbacks={CANDLES: candle_callback, L2_BOOK: book, TRADES: trade, TICKER: ticker, BOOK_DELTA: delta}))
    pairs = Binance.symbols()
    f.add_feed(Binance(symbols=pairs, channels=[TRADES], callbacks={TRADES: TradeCallback(trade)}))
    
    f.run()

if __name__ == '__main__':
    main()
```
### 2.Connection Abstraction


### 3.Connection Handler
cryptofeed 支持 HTTP, Websocket 协议.

Connection Handler 通过创建连接、处理异常并根据需要重新启动连接，进行维护和监视 connections.

### 4.Exchange Interfaces
#### channels：

L2_BOOK - Price aggregated sizes. Some exchanges provide the entire depth, some provide a subset.

L3_BOOK - Price aggregated orders. Like the L2 book, some exchanges may only provide partial depth.

TRADES - **Note this reports the taker's side**, even for exchanges that report the maker side

TICKER - Traditional ticker updates

FUNDING - Exchange specific funding data / updates

BOOK_DELTA - **只接收数据变化**，完整数据需订阅 L2 或 L3。 注意同时订阅 L2 和 L3 并掌握变化需要两次 feedhandler，每个 feedhandler 内的 BOOK_DELTA 只能对应 L2 或 L3 中的一个.

#### symbols
格式为 BASE - QUOTE，例如 BTC-USD, BTC-USDT, ETH-USD.

#### subscriptions
以字典的格式提供
```Python
{TRADES: ['BTC-USD', 'BTC-USDT', 'ETH-USD'], L2_BOOK: ['BTC-USD']}
```
### 5.Callbacks
使用框架时定义接收数据的格式(数据库或 socket). 注意不要在 callback 中做任何耗费算力的事情，会极大影响 cryptofeed 的性能。数据应该被快速处理并传递给另一个进程/应用程序等，或者应该使用backend回调将数据转发到其他地方。

两种模式：raw, backend

1. [raw](https://github.com/bmoscon/cryptofeed/blob/master/cryptofeed/callback.py) 直接对数据进行修改

`Trade`
`Ticker`
`Book`
`Book Update`
`Open Interest`
`Funding`
`Liquidation`
`Volume`

2. [backend](https://github.com/bmoscon/cryptofeed/tree/master/cryptofeed/backends) 对数据进行存储和传输. 例如：Redis, Postgres 用来存储，TCP 传输给其他函数进行处理.

`Arctic`
`ElasticSearch`
`InfluxDB`
`Kafka`
`MongoDB`
`Postgres`
`RabbitMQ`
`Redis`
`Redis Streams`
`TCP/UDP/UDS sockets`
`ZMQ`

[wrappers](https://github.com/bmoscon/cryptofeed/blob/master/cryptofeed/backends/aggregate.py) 可以与回调函数一起使用，将数据转换为 OHLCV (Open, High, Low, Close, Volume), 节流数据等

### 6.Backends (后端)
`Redis (Streams and Sorted Sets)`   `Arctic`    `ZeroMQ`    `UDP Sockets`   `TCP Sockets`

`Unix Domain Sockets`   `InfluxDB v2`    `MongoDB`    `Kafka`     `Elastic Search`

`RabbitMQ`    `PostgreSQL`    `GCP Pub/Sub`     `VictoriaMetrics`

