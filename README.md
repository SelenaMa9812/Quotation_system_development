# Quotation-system-development
## 行情中心
<img src="https://github.com/SelenaMa9812/Quotation_system_development/blob/main/images/%E8%A1%8C%E6%83%85%E4%B8%AD%E5%BF%83.jpg" width="800" height="500" />

## cryptofeed开源框架
### Feedhandler
#### 可以连接的交易所
使用add_feed添加下面的交易所(一次调用只能添加一个交易所)，feed列表即为已添加的交易所的列表。

`AscendEX`   `Bequant`   `Bitcoin.com`   `Bitfinex`    `bitFlyer`   `Bithumb`   `Bitstamp`    `Bittrex`   `Blockchain.com`

`Bybit Binance`   `Binance Delivery`    `Binance Futures`    `Binance US`    `BitMEX`    `Coinbase`    `Deribit`   

`dYdX`    `EXX`(sometimes a connection can take many many retries)   `FTX`   `FTX US`      `Gate.io`   `Gemini`  

`HitBTC`    `Huobi`   `Huobi DM`    `Huobi Swap`    `Kraken`    `Kraken Futures`    `KuCoin`(requires an API key for L2 book data)

`OKCoin`    `OKEx`    `Phemex`    `Poloniex`    `ProBit`    `Upbit`

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


### Connection Abstraction
### Connection Handler
### Exchange Interfaces
### Callbacks
### Backends (后端)
`Redis (Streams and Sorted Sets)`   `Arctic`    `ZeroMQ`    `UDP Sockets`   `TCP Sockets`

`Unix Domain Sockets`   `InfluxDB v2`    `MongoDB`    `Kafka`     `Elastic Search`

`RabbitMQ`    `PostgreSQL`    `GCP Pub/Sub`     `VictoriaMetrics`



