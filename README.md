# Crypto Signals

Crypto Signals is a command line tool that automates your crypto currency Technical Analysis (TA).

It is based on [Crypto Signals] https://github.com/CryptoSignal/crypto-signal , so I recommend you to take a look in that proyect to known what is it.

I'm making minor changes and adding some features in this repo because the original CryptoSignal project is no longer maintained.

## Notable Changes
- It creates candle bar charts with MAs, RSI and MACD. These images can be sent as part of a Telegram notification or a Webhook call.
- It allows to include prices as part of the notification message.
- New configuration to easily add many coins. Check bellow for "all_pairs".
- New config var to use a custom "indicator_label" for each configured indicator and crossovers. Mainly useful for std_crossover.

## Installing And Running
The commands listed below are intended to be run in a terminal.

1. Clone this repo

1. Create a config.yml file and put it into "app" folder.

1. Build your own Docker image, for example `docker build -t laliux/crypto-signals:latest .`

1. For testing and debugging run `docker run --rm -ti -v  $PWD/app:/app laliux/crypto-signals:latest`

1. For production run in daemon mode `docker run --rm -di -v  $PWD/app:/app laliux/crypto-signals:latest`


### Configuring config.yml

All possible options for config.yml are almost the same for original CryptoSignal, so look [here](docs/config.md)

However there are some aditional options to use.

#### Charts

You can enable/disable charts creation.

```
settings:
    log_level: INFO
    update_interval: 600
    enable_charts: true
    market_pairs:
        - XRP/USDT
        - ETH/USDT
        - BTC/USDT
        ....
```

#### All pairs

It is very useful to process all symbols of a specific base market (BTC, ETH, USD, USDT). In this way you can avoid to write pair by pair in settings.market_pairs option.

The all_pairs option must be specified by each exchange and base market. Example:

```
exchanges:
    binance:
        required:
            enabled: true
        all_pairs:
            - USDT
    bittrex:
        required:
            enabled: false
        all_pairs:
            - ETH
            - BTC
        ....
```

Finally, if you want prices in your notification messages, you can use a new variable "prices".

```
notifiers:
    telegram:
        required:
            token: 791615820:AAGFgGSumWUrb-CyXtGxzAuYaabababababababa
            chat_id: 687950000
        optional:
            parse_mode: html
            template: "[{{analysis.config.candle_period}}] {{market}} {{values}} Prices: [{{prices}}]"
```

By the way, to have this feature you need to configure "ohlcv" informant for each candle period of your indicators.

```
informants:
    ....
    bollinger_bands:
        - enabled: false
    ohlcv:
        - enabled: true
          signal:
            - high
            - low
            - close
          candle_period: 1h
          period_count: 14
        - enabled: true
          signal:
            - high
            - low
            - close
          candle_period: 4h
          period_count: 14
```

#### Chart images on webhook

The config for a webhook notifier is the same as original CryptoSignal, no changes here, BUT the data sent in the request is completely different. 

```
notifiers:
    telegram:
        required:
            token: 791615820:AAGFgGSumWUrb-CyXtGxzAuY7UPxxxxx
            chat_id: 687957000
        optional:
            parse_mode: html
            template: "[{{indicator_label}}] {{market}} {{values}}, Prices: [{{prices}}]" 
    webhook:
        required:
            url: http://somename.or.ip.here:8888/someuri
        optional:
            username: null
            password: null    
```

Then in your webhook you will have a parameter "messages" which is a list/array of the notification messages in form of string values. Apply a json decode to access all values. In python pseudo code:

```
msg = self.get_argument('messages')
print ( json.loads(msg) )
        
[{'values': {'rsi': '84.20'}, 'exchange': 'binance', 'market': 'VET/USDT', 'base_currency': 'VET', 'quote_currency': 'USDT', 'indicator': 'rsi', 'indicator_number': 0, 'analysis': {'config': {'enabled': True, 'alert_enabled': True, 'alert_frequency': 'always', 'signal': ['rsi'], 'hot': 40, 'cold': 60, 'candle_period': '15m', 'period_count': 13}, 'status': 'cold'}, 'status': 'cold', 'last_status': 'cold', 'creation_date': '2019-01-04 15:31:40', 'indicator_label': 'rsi 15m'}]
```

If you have enable_charts: true, you will have a parameter "chart" in the same way of a HTTP file upload. Reading this file depends 100% in your backend. Example python pseudo code to save the received chart:

```
    fileinfo = self.request.files['chart'][0]
        
    print ("filename is " , fileinfo['filename'], ', content_type: ', fileinfo['content_type'])

    fname = fileinfo['filename']
    extn = os.path.splitext(fname)[1]
    cname = str(uuid.uuid4()) + extn
    fh = open(__UPLOADS__ + cname, 'wb')
    fh.write(fileinfo['body'])
```


# Liability
I am not your financial adviser, nor is this tool. Use this program as an educational tool, and nothing more. None of the contributors to this project are liable for any losses you may incur. Be wise and always do your own research.
