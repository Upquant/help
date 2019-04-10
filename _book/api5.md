# API使用文档

## 从一个简单的策略示例说起

与其他纯web端的量化平台相比，本平台提供的方式支持本地化代码，因此在初始化阶段，比其他平台稍微多了数据缓存的初始化流程。相对于这一点点繁琐，我们可以保证任何代码都不用提交到平台，更能保证策略的私密性，

一个典型的策略代码需要包含以下事件函数的实现，完整的例子可以直接看安装包下`example`文件夹中的`BuySell-future`。
### 引入etasdk量化策略库
``` 
from etasdk import *

if __name__ == '__main__':
    # 读取配置文件并启动策略
    bar_span = ETimeSpan.DAY_1
    config = {
        "host" : "120.55.22.60",
        "port" : 80,
        "user" : "SDK-User",   # Web端：点击量化平台右上角用户名可查看填写
        "token" : "SDK-Token", # Web端：点击量化平台右上角用户名可查看填写
        "strategy_name": "test", # 策略名称
        "console" : 0,
        "loglevel" : "ERROR", # 显示日志级别，具体可查看帮助文档
        # Web端：回测记录中本策略的回测ID,点击复制ID,粘贴此处即可
        "instance_id" : "20180530-163657-609-001BI",
        # 初始化参数
        "initialize": {
            # symbols里设置单个股票或者其他有效的标的
            # symbol_sets里设置标的集合，如此处的IF.PRD指的就是IF品种的所有标准合约，不包含IFZ0.CF和IFZ1.CF
            # 注：symbols和symbol_sets不能全部为空
            'symbols': ['IFZ0.CF'],
            'symbol_sets': ['IF.PRD', 'CF.PRD', 'IC.PRD'],
            # fields缓存的因子
            'fields': ['FUT_TRADE_RANK', 'FUT_SHORT_RANK', 'FUT_LONG_RANK', 'STK_HOLD_DET', 'STK_HOLD_STAT', 'ST_STA','LIST_STA', 'TRADE_STA', 'NET_PROFIT', 'PB', 'PE', 'PS', 'PE_TTM'],
            # 必填：设置所需k线类型，interval可以选取1分钟，5分钟，15分钟，30分钟，60分钟，日k，Tick级别，count为提前缓存的数目
            'prepared_bars': [
                {'interval': bar_span, 'count': 1}
            ],
            # 选填：设置撮合参数 interval为撮合周期，time_ranges为日内撮合时间段，将只在指定时间段以interval周期回调on_bar或on_handle_date
            'match_param': {
                'interval': bar_span,
                # 'time_ranges': [('12:00', '14:45'), ('10:10', '11:00')]
            },
            # 选填：为group_only，则只响应on_handle_data；为single_only，则只响应on_bar；为both，则两个回调同时响应
            'mode': 'group_only',
            # 选填：设置手续费
            'commission': 0.0002,

            # 选填：仅回测生效的项，不设置时以web页面上的设置为准
            'backtest': {
                # 回测开始日期
                'start_date': 20190101,
                # 回测结束日期
                'end_date': 20190104,
                # 设置滑点,取值为整数,标的价格浮动最小单位值的整数倍
                'slippage': 2,
                # 设置账户初始资金
                'cash': {
                    # 设置A股资金
                    'CS': 0,
                    # 设置期货资金
                    'CF': 11000000
                }
            },

            # 选填：仅模拟盘/实盘生效的项
            'realtime': {
                # 设置开盘前回调on_before_market_open响应事件时间，默认为web页面上设置的时间
                'pre_market_time': '19:00',
                # 设置收盘前最后一次回调on_bar或on_handle_data响应事件时间，默认为web页面上设置的时间
                'closing_time': '14:55',
                # 设置group模式下的组播，等待多少ms则强制响应，默认为5000ms
                'group_timeout': 10000,
                # 设置onTimer响应时间(ms), 为0则不响应onTimer，默认情况下不回调onTimer
                'timer_cycle': 60000
            }
        }
    }
    
    etasdk.load_config_json(config).start()
```
### 系统初始化事件
**必要**：作为系统进入状态前设置必要的参数，包括自定义参数和系统预设参数，如在`config`配置中已设置初始化参数`initialize`则不需要再调用`def on_initialize(api)`函数，如`config`中未设置，则可参考如下示例设置。

```  
def on_initialize(api):
    """
    初始化
    :param api:
    :return:
    """
    api.bar_span = ETimeSpan.DAY_1  # 自定义变量，代表撮合周期
    # api.bar_span = ETimeSpan.MIN_5  # 自定义变量，代表撮合周期
    api.context.init = {
        # symbols里设置单个股票或者其他有效的标的
        # symbol_sets里设置标的集合，如此处的IF.PRD指的就是IF品种的所有标准合约，不包含IFZ0.CF和IFZ1.CF
        # 注：symbols和instsets不能全部为空
        # 'symbols': ['IFZ0.CF'],
		'symbols': ['000001.CS'],
        'symbol_sets': ['IF.PRD'],
        # fields缓存的因子,
        'fields': ['PE', 'PS', 'FUT_TRADE_RANK','TRADE_STA'],
        # 必填：设置所需k线类型，interval可以选取1分钟，5分钟，15分钟，30分钟，60分钟，日k，Tick级别，count为提前缓存的数目
        'prepared_bars': [
            {'interval': api.bar_span, 'count': 100}
        ],
        # 选填：设置撮合参数 interval为撮合周期，time_ranges为日内撮合时间段，将只在指定时间段以interval周期回调on_bar或on_handle_date
        'match_param': {
            'interval': api.bar_span,
            # 'time_ranges': [('12:00', '14:45'), ('10:10', '11:00')]
        },
        # 选填：为group_only，则只响应on_handle_data；为single_only，则只响应on_bar；为both，则两个回调同时响应
        'mode': 'group_only',
        # 选填：设置手续费
        'commission': 0.0002,

        # 选填：仅回测生效的项，不设置时以web页面上的设置为准
        'backtest': {
            # 回测开始日期
            'start_date': 20181124,
            # 回测结束日期
            'end_date': 20190101,
            # 设置滑点,取值为整数,标的价格浮动最小单位值的整数倍
            'slippage': 2,
            # 设置账户初始资金
            'cash': {
                # 设置A股资金
                'CS': 5000000,
                # 设置期货资金
                'CF': 11000000
            }
        },

        # 选填：仅模拟盘/实盘生效的项
        'realtime': {
            # 设置开盘前回调on_before_market_open响应事件时间，默认为web页面上设置的时间
            # 'pre_market_time': '20:00',
            # 设置收盘前最后一次回调on_bar或on_handle_data响应事件时间，默认为web页面上设置的时间
            # 'closing_time': '14:55',
            # 设置group模式下的组播，等待多少ms则强制响应，默认为5000ms
            'group_timeout': 10000,
            # 设置onTimer响应时间(ms), 为0则不响应onTimer，默认情况下不回调onTimer
            'timer_cycle': 60000
        }
    }
```

### 盘前事件

**必要**：每个交易日回测/模拟/实盘前要做的操作，选标的和进行盘前下单操作，**必须实现！**

```   
def on_before_market_open(api, time):
    """
    必要：每个交易日回测/模拟/实盘前需要响应的事件
    :param api:
    :param trade_now:
    :return:
    """
    api.futures_code = api.get_continuous_symbol("IFZ0.CF")  # 获得当前交易日IF主力合约代码

    api.context.pool.focus = [api.futures_code, 'IFZ0.CF']
```

### 盘中事件

根据初始化参数来选择响应`on_bar`和`on_handle_data`，此处初始化使用的是`group_only`模式，则只响应`on_handle_data`
       
``` 
# on_bar每一个Focus标的对应的k线到时间都会响应
def on_bar(api, bar):
    ...
```

```     
# on_handle_data在关注的所有标的都可以响应时或者超时时响应
def on_handle_data(api, time):      
    trade_now = api.context.time.trade_now
    symbol_positions = api.context.strategy.positions  # 获取持仓标的信息
    bars = api.context.bar[api.futures_code]  # 获得合约当前bar数据
    factor = bars['preClose'] / bars['open']  # 当前开盘价与昨日收盘价之比
    # print(factor)

    if symbol_positions:  # 如果有持仓
        if api.futures_code == symbol_positions[0].symbol:  # 当前持仓为主力合约
            # 止盈5% 止损3%
            if symbol_positions[0].posHigh / symbol_positions[0].posPrice > 1.05 \
                or symbol_positions[0].posLow / symbol_positions[0].posPrice < 0.97:
                # 平仓
                print(trade_now.date, "target position 0!")  # 录入日志
                #卖出 ，symbol = 持仓标的，数量变为0
                api.target_position(symbol=symbol_positions[0].symbol, qty=0,
                                    side=EPositionSide.LONG)  
        # 当前持仓非主力合约 则换仓为主力合约
        else:
            print(trade_now.date, symbol_positions[0].symbol, "换合约", api.futures_code)
            api.target_position(symbol=symbol_positions[0].symbol, \
                                qty=0, side=EPositionSide.LONG)
            api.target_position(symbol=api.futures_code, \
                                qty=2, side=EPositionSide.LONG)
    else:
        # factor 算出有3%涨幅
        if factor[0] < 0.97:
            print(trade_now.date, "target position 2")  # 非必要：输出日志
            # 买入选中主力合约对应标的开多，直至持仓数达2手
            api.target_position(symbol=api.futures_code, 
                                qty=2, side=EPositionSide.LONG)  
```

### 系统终止事件-回测
回测状态下，系统最后会将回测结果在这里输出

``` 
#运行终止调用
def on_terminate(api, exit_info):
    # 存入回测结果 
    api.save_object(exit_info)

```

### 小结
至此，一个简单的期货买卖策略就介绍完毕了，接下来将要了解本平台提供的各种接口和参数，以便写出自己的策略代码。

## 深度理解api-必看
### 主要问题
为了高效的使用API, 需要对平台的机制有一定的了解. 优品量化的平台最核心的特点是SDK模式, 本机安装SDK后, 策略基于SDK实现, 和云端量化引擎通讯来完成策略的运行, 同时策略运行过程中, 数据也需要自动的从云端拉取到本地. 那么为了保证策略运行的高效性,  就需要考虑如下几个问题:
- 这么大量的数据, 如何保证拉取数据的效率?
- 策略运行过程中, 播放K线时如果保证效率?

为了解决这两个问题, API提供了相关机制来控制和优化效率:

### 设置数据
在[initialize](api5.html#引入etasdk量化策略库)中设置需要的数据:
- 个股, 标的池(市场, 板块,期货品种等)
- K线类型和历史根数
- A股各类因子
```
#设置股票池、相关标的因子数据和K线。
'symbols': ["IFZ0.CF","000001.CS"],
'symbol_sets': ["000016.IDX","cu.PRD"],
'fields': ["CIRC_CAP", "TURNOVER_RATE", "PE"],
'prepared_bars': [{'interval': bar_span, 'count': 1}],
#设置股票池、相关标的因子数据和K线。
'symbol_sets': ["000016.IDX","cu.PRD"],
'fields': ["CIRC_CAP", "TURNOVER_RATE", "PE"],
'prepared_bars': [{'interval': bar_span, 'count': 1}],
#设置股票池、相关标的因子数据和K线。
'symbols': ["IFZ0.CF","000001.CS"],
'fields': ["CIRC_CAP", "TURNOVER_RATE", "PE"],
'prepared_bars': [{'interval': bar_span, 'count': 1}],
#设置股票池和K线。
'symbols': ["IFZ0.CF","000001.CS"],
'symbol_sets': ["000016.IDX","cu.PRD"],
'prepared_bars': [{'interval': bar_span, 'count': 1}],
#设置股票池和相关标的因子数据
'symbol_sets': ["000016.IDX","cu.PRD"],
'symbols':  ["IFZ0.CF","000001.CS"],
'fields': ["CIRC_CAP", "TURNOVER_RATE", "PE"]
```
[initialize](api5.html#引入etasdk量化策略库)在策略启动后只会调用一次, 在策略运行过程中, 只会拉取相关的数据. 因此在实现策略过程中, 尽量只设置自己需要的数据, 避免拉取太多数据, 这有助于提高策略运行的性能.

### 减少K线播放量
策略运行过程中, 云端量化引擎会播放相应的K线来驱动策略运行.SDK中的`on_before_market_open`每个交易日开盘前会响应一次, 策略可以在这个函数中执行选股逻辑, 然后调用`set_focus_symbols`来关注选出来的标的.

注意:  只有`set_focus_symbols`的股票以及持仓的标的才会收到当天的行情推送.  

比如:在[initialize](api5.html#引入etasdk量化策略库)中,设置了A股全市场(CS.SET), 但是每天选股过程中只会选取几只股票, 并对这几只股票做买卖操作, 因此只需要`set_focus_symbols`这几只股票, 从而大幅度提升了效率.

### K线的三种播放机制
- `on_handle_data`模式: `on_before_market_open`中focus了多少标的, 都只会回调`on_handle_data`一次.
- `on_bar`模式: `on_before_market_open`中`set_focus_symbols`的标的+持仓标的, 每个标的都会收到`on_bar`的回调, 比如`set_focus_symbols`标的+持仓标的有10只, 播放是日K行情, 那么当天`on_bar`会有10次回调;
- `on_handle_data/on_bar`混合的`both`模式: 两者都会回调

[initialize](api5.html#引入etasdk量化策略库)中通过调用`mode`来控制播放模式, 默认不设置`mode`则是`on_bar`模式

```
#设置只响应on_handle_data，超时时间为5000ms
'mode': 'group_only',
...
'group_timeout': 5000,
#设置既响应on_handle_data，也响应on_bar，超时时间为10000ms
'mode': 'both',
...
'group_timeout': 10000,
```
`group_timeout`用于在模拟盘/实盘情况下, 如果超过`group_timeout`, 相应的K线还没有到齐, 则不再等待, 直接回调`on_handle_data`.

## 内置数据结构

### RefData-股票基础信息

| 属性          | 类型   | 说明                                                         |
| ------------- | ------ | ------------------------------------------------------------ |
| symbol        | str    | 标的代码。                                                   |
| marketName    | str    | 品种名称。CS-股票；CF-期货；IDX-指数。                       |
| exchange      | str    | 交易所及自定义类型后缀。SZ-深交所；SH上交所；DCE-大商所；SHFE-上期所；CZCE-郑商所；CFFEX-中金所；PLATE-板块；SET-证券集合。 |
| currency      | str    | 币种。                                                       |
| lotSize       | double | 一手标的的数目。                                             |
| name          | str    | 标的名称。                                                   |
| tplus         | int    | T+N交易。例如tplus为1时为t+1交易。                           |
| marginRate    | double | 保证金比例。                                                 |
| shortSellable | bool   | 是否可卖空。                                                 |
| valuePerUnit  | double | 合约乘数                                                     |
| priceTick     | double | 标的价格浮动最小单位值                                       |
| exchSymbol    | str    | 交易所的原始symbol                                           |
| isStandard    | bool   | 是否是交易所的标准合约，比如主力合约 IFZ0.CF 就不是交易所的标准合约 |
| tradeMarket   | str    | 该标的对应的交易市场，目前仅有 CS CF                         |
| listDate      | int    | 上市日，无则为0。                                            |
| lastTradeDate | int    | 最后交易日(退市日)，无则为0。                                |


### Tick-Tick行情信息

| 属性          | 类型             | 说明                                                         |
| ------------- | ---------------- | ------------------------------------------------------------ |
| symbol        | str              | 标的代码。                                                   |
| bid           | double           | 买一价。                                                     |
| ask           | double           | 卖一价。                                                     |
| bidVol        | double           | 买一量。（单位为实际证券股数）                               |
| askVol        | double           | 卖一量。                                                     |
| last          | double           | 最新成交价。                                                 |
| lastVolume    | double           | 最新成交量。                                                 |
| lastTurnover  | double           | 最新成交额。                                                 |
| totalVolume   | double           | 交易日总成交量。                                             |
| totalTurnover | double           | 交易日总成交额。                                             |
| high          | double           | 交易日最高价。                                               |
| low           | double           | 交易日最低价。                                               |
| open          | double           | 交易日开盘价。                                               |
| close         | double           | 交易日收盘价。                                               |
| tradeDate     | long             | 交易日-年月日，格式为YYYYMMDD                                |
| timeExch      | long             | 交易所的实时时间戳-年月日时分秒 (ms)，                       |
| timeStr       | str              | 交易时间，格式为YYYYMMDD-hhmmss-xxx                          |
| bids          | vector<qtyprice> | 五档买价，包括数量和价格，qtyprice{"quantity":qty,"price":price}。 |
| asks          | vector<qtyprice> | 五档卖价，包括数量和价格。                                   |
| ceil          | double           | 涨停价。                                                     |
| floor         | double           | 跌停价。                                                     |
| position      | double           | 持仓量(期货)。                                               |
| preClose      | double           | 昨日收盘价。                                                 |
| preSettle     | double           | 昨日结算价。                                                 |



### Bar-Bar数据信息

| 属性          | 类型   | 说明                                |
| ------------- | ------ | ----------------------------------- |
| symbol        | str    | 标的代码。                          |
| tradeDate     | long   | 交易日-年月日，格式为YYYYMMDD       |
| timeStop      | long   | bar对应的截止时间戳 (ms)            |
| timeStr       | str    | 交易时间，格式为YYYYMMDD-hhmmss-xxx |
| timeSpan      | long   | K线的时间频率。                     |
| high          | double | 交易日最高价。                      |
| low           | double | 交易日最低价。                      |
| open          | double | 交易日开盘价。                      |
| close         | double | 交易日收盘价。                      |
| volume        | double | KBar成交量。                        |
| turnover      | double | KBar成交额。                        |
| totalVolume   | double | 交易日总成交量。                    |
| totalTurnover | double | 交易日总成交额。                    |
| preClose      | double | 昨日收盘价。                        |
| settle        | double | 结算价。                            |
| preSettle     | double | 昨结。                              |
| position      | double | 持仓量。                            |
| isSuspended   | bool   | 是否停牌。                          |


### SymbolPosition-策略的标的持仓

某一个策略中某只标的的仓位信息。

| 属性               | 类型         | 说明                                             |
| ------------------ | ------------ | ------------------------------------------------ |
| symbol             | str          | 标的代码。                                       |
| positionSide       | PositionSide | 持仓方向。                                       |
| posQty             | double       | 持仓数量，不包括挂单的仓位。                     |
| posDailyQty        | double       | 当日开仓的持仓数量，不包括挂单的仓位。           |
| posPrice           | double       | 持仓均价。                                       |
| posHigh            | double       | 持仓以来最高价。                                 |
| posLow             | double       | 持仓以来最低价。                                 |
| posTime            | long         | 开仓时间（时间戳）。                             |
| posUrPnL           | double       | 仓位总浮动盈亏。                                 |
| availableQty       | double       | 今日可平仓数量。                                 |
| posDailyOverallPnL | double       | 当日盈亏=当日浮动盈亏+当日平仓盈亏，结算时清零。 |
| posTradeDate       | int          | 开仓交易日，格式为YYYYMMDD。                     |
| posMargin          | double       | 仓位占用保证金。                                 |
| posMarketValue     | double       | 仓位市值。                                       |


 ### StrategyPnL-策略的盈亏信息

策略的整体盈亏信息。

| 属性            | 类型   | 说明                                                   |
| --------------- | ------ | ------------------------------------------------------ |
| id              | double | 策略ID。                                               |
| urPnL           | double | 策略的总浮动盈亏。                                     |
| overallPnL      | double | 策略的整体总盈亏，包含平仓盈亏。                       |
| dailyPnL        | double | 策略的当日盈亏=当日浮动盈亏+当日平仓盈亏，结算时清零。 |
| totalCommission | double | 策略总手续费                                           |



 ### OverallPosition-账户的标的持仓

由于多个策略可以共享账户资金，因此可能会存在多个策略中都持有某个标的的情况，账户中会将多个策略中标的信息进行汇总。

| 属性              | 类型   | 说明               |
| ----------------- | ------ | ------------------ |
| accountId         | str    | 账户ID。           |
| symbol            | str    | 标的代码。         |
| buyPrice          | double | 加权平均买价。     |
| sellPrice         | double | 加权平均卖价。     |
| buyQty            | double | 买数量。           |
| sellQty           | double | 卖数量。           |
| longQty           | double | 多仓持仓数量。     |
| shortQty          | double | 空仓持仓数量。     |
| longPrice         | double | 多仓持仓均价。     |
| shortPrice        | double | 空仓持仓均价。     |
| longLastUrPnL     | double | 多仓浮动盈亏。     |
| shortLastUrPnL    | double | 空仓浮动盈亏。     |
| lastUrPnL         | double | 总持仓总浮动盈亏。 |
| longAvailableQty  | double | 多仓可平仓数量。   |
| shortAvailableQty | double | 空仓可平仓数量。   |
| longMargin        | double | 多仓占用保证金。   |
| shortMargin       | double | 空仓占用保证金。   |
| longMarketValue   | double | 多仓持仓市值。     |
| shortMarketValue  | double | 空仓持仓市值。     |


 ### Account-账户信息

| 属性            | 类型   | 说明                                             |
| --------------- | ------ | ------------------------------------------------ |
| id              | str    | 账户ID。                                         |
| market          | str    | 市场。                                           |
| currency        | str    | 账户币种。                                       |
| dailyPnL        | double | 当日盈亏=当日浮动盈亏+当日平仓盈亏，结算时清零。 |
| urLastPnL       | double | 浮动盈亏。                                       |
| allTimePnL      | double | 平仓盈亏。                                       |
| cashDeposited   | double | 累积出入金。                                     |
| cashAvailable   | double | 可用现金。                                       |
| unitValue       | double | 账户基金净值。                                   |
| bonus           | double | 现金分红。                                       |
| margin          | double | 持仓占用资金。                                   |
| marketValue     | double | 持仓市值。                                       |
| totAssets       | double | 总资产。                                         |
| totalCommission | double | 总手续费。                                       |


 ### Order-订单委托信息

| 属性            | 类型            | 说明                         |
| --------------- | --------------- | ---------------------------- |
| symbol          | string          | 标的代码。                   |
| side            | EOrderSide      | 买卖方向。                   |
| positionSide    | EPositionSide   | 订单持仓方向。               |
| positionEffect  | EPositionEffect | 订单开平仓类型。             |
| qty             | double          | 委托量。                     |
| price           | double          | 委托价。                     |
| userId          | string          | 用户id。                     |
| accountId       | string          | 账户id。                     |
| created         | long            | 委托创建日期。               |
| id              | string          | 订单id。                     |
| tradeDate       | int             | 交易日。                     |
| strategyId      | string          | 所属策略id。                 |
| portfolioId     | string          | 组合id。                     |
| commission      | commission      | 手续费。                     |
| tradeAccount    | string          | 交易账号。                   |
| externalOrderId | string          | SDK生成的ID-SDK。            |
| orderType       | EOrderType      | 订单委托类型，市价或限价单。 |
| ordStatus       | EOrdStatus      | 订单状态。                   |
| cumQty          | double          | 已成量。                     |
| avgPx           | double          | 已成均价。                   |
| tif             | TimeInForce     | 订单时间状态。               |
| modified        | long            | 委托最后修改日期。           |
| execType        | EExecType       | 订单执行状态。               |


 ### EtaTime-时间信息

| 属性      | 类型    | 说明                               |
| --------- | ------- | ---------------------------------- |
| year      | int     | 年                                 |
| month     | int     | 月                                 |
| day       | int     | 日                                 |
| hour      | int     | 时                                 |
| minute    | int     | 分                                 |
| second    | int     | 秒                                 |
| week      | int     | 周                                 |
| weekday   | int     | 星期 0-6 代表星期一-星期日         |
| date      | int     | YYYYmmdd                           |
| now       | EtaTime | 当前时间，回测则为回测当前时间     |
| trade_now | EtaTime | 当前交易日，回测则为回测当前交易日 |

内置向前取交易日函数：
``` 
get_prev_trade_dates(market='CS', count=None, start=None, period=('D', 1))
```
示例：
``` 
nowdate = EtaTime.trade_now

print(nowdate.get_prev_trade_dates().date) # 20180103

print(nowdate.get_prev_trade_dates(count=2)) # list[EtaTime]

print(nowdate.get_prev_trade_dates(start=20171225)) # list[EtaTime]
```



## 内置枚举类型
内置枚举类型的取用方式为enumClass.enum，例如：tick级别的频率类型，ETimeSpan.TICK

### ETimeSpan-频率类型

```
enum ETimeSpan
{
    TICK,    //tick类型
    MIN_1,   //1分钟K线
    MIN_5,   //5分钟K线
    MIN_15,  //15分钟K线
    MIN_30,  //30分钟K线
    MIN_60,  //60分钟K线
    DAY_1,   //日K线
};
```


### EPriceMode-复权模式

```
enum EPriceMode
{
    REAL,    //不复权
    FORMER,  //前复权
};
```


### EPositionSide-持仓方向

```
enum EPositionSide
{
    LONG = 1,   //多仓
    SHORT = 2,  //空仓
};
```


### EOrderSide-买卖方向

```
enum EOrderSide
{
    BUY = 1, //买入
    SELL = 2,//卖出
};
```


### EOrderType-订单类型

```
enum EOrderType 
{
    MARKET = 1,     //市价单
    LIMIT  = 2,     //限价单
};
```


### EOrdStatus-订单状态

```
enum EOrdStatus 
{
    NONE = 0,
    NEW = 1,				   //新单
    PARTIALLY_FILLED=2,		//部分成交
    FILLED= 3,			 	//全部成交
    DONE_FOR_DAY=4,			//今日完结
    CANCELED=5,				//撤销
    REPLACED=6,				//修改
    PENDING_CANCEL=7,	  	//正在撤销
    STOPPED=8,				 //终止
    REJECTED=9,				//拒绝
    SUSPENDED=10,			  //挂起
    PENDING_NEW=11,			//正在创建
    CALCULATED=12,		 	//正在计算
    EXPIRED=13,				//过期
    ACCEPTED_FOR_BIDDING=14,   //正在接受交易
    PENDING_REPLACE=15,		//正在修改
};
```



### EExecType-订单执行状态

```
enum EExecType 
{                                                                  
    NONE = 0,
    NEW = 1,				   //新单
    PARTIALLY_FILLED=2,		//部分成交
    FILLED= 3,			 	//全部成交
    DONE_FOR_DAY=4,			//今日完结
    CANCELED=5,				//撤销
    REPLACED=6,				//修改
    PENDING_CANCEL=7,	  	//正在撤销
    STOPPED=8,				 //终止
    REJECTED=9,				//拒绝
    SUSPENDED=10,			  //挂起
    PENDING_NEW=11,			//正在创建
    CALCULATED=12,		 	//正在计算
    EXPIRED=13,				//过期
    ACCEPTED_FOR_BIDDING=14,   //正在接受交易
    PENDING_REPLACE=15,		//正在修改
};
```


### ETimeInForce-订单委托属性

```
enum ETimeInForce
{
    NONE = 0,  //未指定，如果不指定;则限价单默认为DAY,市价单默认为 IMMEDIATE_OR_CANCEL
    DAY = 1,                  //订单在交易日内有效，会自动取消
    GOOD_TILL_CANCEL = 2,     //暂时不支持—订单一直有效，直到投资者主动撤销订单（一般也有默认有效期30~90天）
    AT_THE_OPENING = 3,       //暂时不支持-订单在开市的时候被执行，如果开盘内不能执行，则取消；通常是开盘前集合竞价或者开盘前的瞬间
    IMMEDIATE_OR_CANCEL = 4,  //订单必须立即执行，不能被立即执行的部分将被取消
    FILL_OR_KILL = 5,         //订单必须被完全执行或者完全不执行
    GOOD_TILL_CROSSING = 6,   //暂时不支持-
    GOOD_TILL_DATE = 7,       //暂时不支持-订单一直有效，直到指定日期
    AT_THE_CLOSE = 8,         //暂时不支持-订单在交易最后执行（或者尽可能接近收盘价），通常是收拾撮合的时间段
};
```


### EtaPriceList-报价列表

```
enum EtaPriceList
{
    EPriceList.NONE = 0,
    EPriceList.BID1 = 1, //买一价
    EPriceList.BID2 = 2, //买二价
    EPriceList.BID3 = 3, //买三价
    EPriceList.BID4 = 4, //买四价
    EPriceList.BID5 = 5, //买五价
    EPriceList.BID6 = 6, //买六价
    EPriceList.BID7 = 7, //买七价
    EPriceList.BID8 = 8, //买八价
    EPriceList.BID9 = 9, //买九价
    EPriceList.BID10 = 10, //买十价

    EPriceList.ASK1 = 101, //卖一价
    EPriceList.ASK2 = 102, //卖二价
    EPriceList.ASK3 = 103, //卖三价
    EPriceList.ASK4 = 104, //卖四价
    EPriceList.ASK5 = 105, //卖五价
    EPriceList.ASK6 = 106, //卖六价
    EPriceList.ASK7 = 107, //卖七价
    EPriceList.ASK8 = 108, //卖八价
    EPriceList.ASK9 = 109, //卖九价
    EPriceList.ASK10 = 110, //卖十价
};
```

## 需要实现的事件

### on_initialize(api)-初始化

**函数示例：**

``` 
def on_initialize(api):
    api.context.init = {
        ...
    }
```

**说明：** 在策略运行前会调用一次，用来进行初始化各项运行参数,如config配置中已有[initialize](api5.md#从一个简单的策略示例说起)配置，则此步骤可省略。


### on_before_market_open(api, trade_date)-盘前运行

**函数示例：**

``` 
def on_before_market_open(api, trade_date):
    print(trade_date) # 20180103
    api.context.focus = ['IFZ0.CF'] # 设置关注的股票
```

**说明：** trade_date表示当前日期，开盘前调用一次，选股的逻辑可以在这个函数中实现，你可以根据策略的复杂度在创建实例时设置其运行时间。另外，在on_before_market_open中只有在撮合周期为日K时才能进行挂单。


### on_bar(api, bar)-Bar响应回调
**函数示例：**

```
def on_bar(api, bar):
    ...
```

**说明：** 创建实例时设置撮合周期，[Bar](api5.html#bar-bar数据信息)产生时会调用。为了提高效率，只有在关注列表（即api.context.focus中存在的标的列表）中和有持仓的标的才会响应行情回调。当api.context.init.mode为'single_only' 或者 'both'时响应


### on_handle_data(api, timestamp)-数据到齐后触发

**函数示例：** 

``` 
def on_handle_data(api, timestamp):
    print (api.context.time.now['%Y-%m-%d %H:%M:%S'])
    print (api.context.bar.focus)
```

**说明：** 针对某一周期的bar数据或者tick数据，搜集齐后再统一响应回调，timestamp为当前周期的时间戳。当api.context.init.mode为'group_only' 或者 'both'时响应


### on_terminate(api, result)-运行终止

**函数示例：** 
```
def on_terminate(api, result):
    api.save_object('xxx', api.props(result))
```
**说明：** 运行终止时会调用一次, result返回值为回测结果。


### on_timer(api)-定时器

**函数示例：** 

```
def on_timer(api):
    ...
```

**说明：** 每隔api.context.init.time_cycle毫秒会响应一次。



### on_order_update(api, order)-订单状态更新回调

**函数示例：**

```
def on_order_update(api, order):
    ...
```

**说明：**  订单状态更新时会响应一次，[order](api5.html#bar-bar数据信息)为订单详情。


## 内置上下文参数-api.context

### 取当前策略上下文的方式

`api.context`

### 取指定策略名称的上下文的方式

`api.context['test']`

### 初始化参数

`api.context.init`

**说明：**

该参数仅在on_initialize事件能够被修改，其他事件时均只读，无法修改。如在其他函数中修改，会抛出异常。


一次性初始化示例：

``` 
api.context.init = {
    # symbols里设置单个股票或者其他有效的标的
    # symbol_sets里设置标的集合，如此处的IF.PRD指的就是IF品种的所有标准合约，不包含IFZ0.CF和IFZ1.CF
    # 注：symbols和symbol_sets不能全部为空
    'symbols': ['IFZ0.CF'],
    'symbol_sets': ['IF.PRD'],
    # 必填：设置所需k线类型，interval可以选取1分钟，5分钟，15分钟，30分钟，60分钟，日k，Tick级别，count为提前缓存的数目
    'prepared_bars': [
        {'interval': api.bar_span, 'count': 100},
        {'interval': ETimeSpan.MIN_5, 'count': 100}
    ],
    # 选填：设置撮合参数 interval为撮合周期，time_ranges为日内撮合时间段，将只在指定时间段以interval周期回调on_bar或on_handle_date
    'match_param': {
        'interval': api.bar_span,
        # 'time_ranges': [('12:00', '14:45'), ('10:10', '11:00')]
    },
    # 选填：为group_only，则只响应onHandleData；为single_only，则只响应on_bar；为both，则两个回调同时响应
    'mode': 'group_only',
    # 选填：设置手续费
    'commission': 0.0002,

    # 选填：仅回测生效的项，不设置时以web页面上的设置为准
    'backtest': {
        # 回测开始日期
        'start_date': 20180101,
        # 回测结束日期
        'end_date': 20180123,
        # 设置滑点,取值为整数,标的价格浮动最小单位值的整数倍
        'slippage': 2,
        # 设置账户初始资金
        'cash': {
            # 设置A股资金
            'CS': 0,
            # 设置期货资金
            'CF': 11000000
        }
    },

    # 选填：仅模拟盘/实盘生效的项
    'realtime': {
        # 设置开盘前回调on_before_market_open响应事件时间，默认为web页面上设置的时间
        'pre_market_time': '19:00',
        # 设置收盘前最后一次回调on_bar或onHandleData响应事件时间，默认为web页面上设置的时间
        'closing_time': '14:55',
        # 设置group模式下的组播，等待多少ms则强制响应，默认为5000ms
        'group_timeout': 10000,
        # 设置onTimer响应时间(ms), 为0则不响应onTimer，默认情况下不回调onTimer
        'timer_cycle': 60000
    }
}
```



单参数初始化示例：

**必要**

**设置缓存股票池**

``` 
# 注：symbols和symbol_sets不能全部为空
# symbols里设置单个股票或者其他有效的标的
api.context.init.symbols = ['IFZ0.CF']
# symbol_sets里设置标的集合，如此处的IF.PRD指的就是IF品种的所有标准合约，不包含IFZ0.CF和IFZ1.CF
api.context.init.symbol_sets = ['IF.PRD']
```

**设置缓存K线或tick**

``` 
# 必填：设置所需k线类型，interval可以选取1分钟，5分钟，15分钟，30分钟，60分钟，日k，Tick级别，count为提前缓存的数目
api.context.init.prepared_bars = [
        {'interval': api.bar_span, 'count': 100},
        {'interval': ETimeSpan.MIN_5, 'count': 100}
    ]
```

**web可配选填项**

**设置撮合参数**

``` 
# 选填：设置撮合参数 interval为撮合周期，time_ranges为日内撮合时间段，将只在指定时间段以interval周期回调on_bar或on_handle_date
api.context.init.match_param.interval = api.bar_span
api.context.init.match_param.time_ranges = [('12:00', '14:45'), ('10:10', '11:00')]
```

**设置K线或tick响应方式**

```
# 选填：为group_only，则只响应onHandleData；为single_only，则只响应on_bar；为both，则两个回调同时响应
api.context.init.mode = 'group_only'
```

**设置手续费**

```
# 选填：设置手续费
api.context.init.commission = 0.0002
```

**设置回测参数**
```
# 选填：仅回测生效的项，不设置时以web页面上的设置为准
# 回测开始日期
api.context.init.backtest.start_date = 20180101
# 回测结束日期
api.context.init.backtest.end_date = 20180123
# 设置滑点,取值为整数,标的价格浮动最小单位值的整数倍
api.context.init.backtest.slippage = 2
# 设置账户初始资金
# 设置A股资金
api.context.init.backtest.cash['CS'] = 0
# 设置期货资金
api.context.init.backtest.cash['CF'] = 1000000
```

**设置实盘/模拟盘参数**
```
# 选填：仅模拟盘/实盘生效的项
# 设置开盘前回调on_before_market_open响应事件时间，默认为web页面上设置的时间
api.context.init.realtime.pre_market_time = '19:00'
# 设置收盘前最后一次回调on_bar或onHandleData响应事件时间，默认为web页面上设置的时间
api.context.init.realtime.closing_time = '14:55'
# 设置group模式下的组播，等待多少ms则强制响应，默认为5000ms
api.context.init.realtime.group_timeout = '10000'
# 设置onTimer响应时间(ms), 为0则不响应onTimer，默认情况下不回调onTimer
api.context.init.realtime.timer_cycle = '60000'
```



### 时间处理参数

`EtaTime 类型`

**说明：**
可以处理etasdk产生的所有时间参数，如bar.tradeDate(20180101)，bar.timeStop(1548744180000)，time等转化为api时间格式

#### 1. 当前时间

**说明：**
回测则为回测当前时间

```
api.context.time.trade_now # 获取当前的交易日，注：期货夜盘归为下一交易日
api.context.time.now # 获取当前时间
```

#### 2. 取年月日时分秒等

```
api.context.time.now.day # 日 12
api.context.time.now.hour # 时 15
api.context.time.now.minute # 分 0
api.context.time.now.second # 秒 0
api.context.time.now.week # 第几周 2
api.context.time.now.weekday # 星期几 0-6代表星期一-星期天
api.context.time.now.date # 日期 20180101
api.context.time.now['%Y%m%d'] # 按照指定的格式输出 '20180112' 
```

#### 3. 转化普通时间

**说明：**
将普通的时间标识转化为EtaTime格式可以使用该格式的取年月日时分秒等的方法

```
# 将20180101转为EtaTime格式
time = api.context.time[20180101]
# 将unix时间戳转为EtaTime格式
time = api.context.time[1548744180000]
# 将time.time格式转为EtaTime格式
time = api.context.time[dtime]
```

### 股票池参数

`api.context.pool`

**说明：**
该参数可以设置或者获取股票池类数据

```
api.context.pool.all # 获取全部缓存的股票池，此参数只读
api.context.pool.focus # 获取关注的股票池，此参数仅可在on_before_market_open被设置
api.context.pool.position # 获取全部持仓中的股票池，此参数只读
```

### k线类参数

`api.context.bar`

**说明：**
该参数仅在on_handle_data里生效，其他事件调用无效，返回值为DataFrame格式

```
api.context.bar.focus # 获取关注的标的，当前收齐的bar
api.context.bar.position # 获取持仓的标的，当前收齐的bar
api.context.bar['IFZ0.CF'] # 获取指定的标的当前的bar，注：只能针对有关注的标的代码

可采用类似 api.context.bar.focus['close']取得数据
```

### 账户信息

`api.context.account`

**说明：**数据返回格式为[Account](api5.html#account-账户信息)

```
api.context.account['CF'] # 获取期货账户信息
api.context.account['CS'] # 获取A股账户信息
```

### 账户持仓信息

`api.context.positions`

**说明：**数据返回格式为[OverallPosition](api5.html#overallposition-账户的标的持仓)

```
api.context.positions # 获取所有持仓标的的列表
api.context.positions['IFZ0.CF'] # 获取指定持仓标的的列表
```

### 策略信息

`api.context.strategy`

**说明：**数据返回格式为[StrategyPnL](api5.html#strategypnl-策略的盈亏信息)

```
api.context.strategy
api.context['test'].strategy
```

**策略持仓**

数据返回格式为[SymbolPosition](api5.html#symbolposition-策略的标的持仓)
``` 
# 策略持仓信息，数据返回格式为 SymbolPosition的数组
api.context.strategy.positions
api.context.strategy.positions.short 所有空方向持仓
api.context.strategy.positions.long 所有多方向持仓
api.context.strategy.positions['IFZ0.CF'] 所有IFZ0.CF的持仓
# 数据返回格式为 SymbolPosition
api.context.strategy.positions['IFZ0.CF'].short IFZ0.CF多仓
api.context.strategy.positions['IFZ0.CF'].long IFZ0.CF空仓
```

## 只盘前支持的函数

### set_focus_symbols(symbols)-设置关注的标的池
与内置参数api.context.pool.focus功能一致。只能在on_before_market_open函数中调用，且必须调用。只有在此函数中进行设置的标的和当前持仓的标的才会在on_bar、on_handle_data或者on_tick函数中进行行情响应。若不设置关注的且不在持仓的标的不会在此三个回调事件中响应，并且获取不到该标的当天的K线。
<font color="#660000">**注：set_focus_symbols 如果多次使用，则只会记录最后一条的设置；前面的设置将会被覆盖**</font>

**函数原型：**

`set_focus_symbols(symbols)`

**函数示例：**

```
def on_before_market_open(api, trade_date):
    ...
    api.set_focus_symbols('000001.CS')
    ...
```

**参数：**

| 参数    | 类型        | 是否必填 | 说明           |
| ------- | ----------- | -------- | -------------- |
| symbols | str or list | 是       | 标的或标的列表 |



## 交易函数
### target_position(symbol, qty, price, side, tif, remark, price_list)-目标仓位数量下单

**函数原型：**

`target_position(symbol, qty, price=None, side=None, tif =ETimeInForce.NONE, remark=None, price_list=EPriceList.None)`

**参数：**

| 参数       | 类型          | 是否必填 | 说明                                                         |
| ---------- | ------------- | -------- | ------------------------------------------------------------ |
| symbol     | str           | 是       | 策略中的标的代码                                             |
| qty        | int           | 是       | 交易量                                                       |
| price      | double        | 否       | 价格，默认不传参（表示以当时现价下单），若传参则表示以该委托价下限价单 |
| side       | EPositionSide | 否       | 多或者空，不填时默认为多仓，股票不能为short。详见 [EPositionSide](api5.html#epositionside-持仓方向) 。 |
| tif        | ETimeInForce  | 否       | 下单属性说明，不填和填NONE时效果相同，详见[ETimeInForce-订单委托属性](api5.html#etimeinforce-订单委托属性) |
| remark     | str           | 否       | 下单原因说明，填写后可以在web端展示                          |
| price_list | EtaPriceList  | 否       | 报价列表，用于下限价单。若与price同时传参，则以price为准，详见[EtaPriceList-报价列表](api5.html#etapricelist-报价列表) |

**返回值：** orderID 作为对此条下单操作的标识。

**举例：**

股票下单

``` 
# 以市价单买入100股000001
api.target_position (symbol="000001.CS", qty=100) 
# 以11元限价单买入100股000001
api.target_position (symbol="000001.CS", qty=100, price=11)  
# 以市价单卖出000001股票至持仓为0，remark里的描述会在web交易信息里体现
api.target_position (symbol="000001.CS", qty=0, remark="卖出平仓") 
# 以限价单卖一价买入000001股票至持仓为100股
api.target_position (symbol="000001.CS", qty=100, price_list=EPriceList.ASK1) 
```

期货下单

``` 
# IF1809市价单开多1手
api.target_position (symbol='IF1809.CF',qty=1, side=EPositionSide.LONG)  
# IF1809市价单开空1手
api.target_position (symbol='IF1809.CF',qty=1, side=EPositionSide.SHORT)  
```

注意：以报价列表方式挂限价单不能在on_before_market_open中挂单


### cancel_order(symbol, side)-撤单

**函数原型**

`cancel_order(symbol， side=EPositionSide.LONG, remark='')`

**参数：**

| 参数   | 类型          | 是否必填 | 说明                                                         |
| ------ | ------------- | -------- | ------------------------------------------------------------ |
| symbol | str           | 是       | 策略中的标的代码                                             |
| side   | EPositionSide | 否       | 多或者空，不填时默认为多仓，股票不能为short。详见 [EPositionSide](api5.html#epositionside-持仓方向) 。 |
| remark | str           | 否       | 撤单原因说明，填写后可以在web端展示                          |

**举例：**

```
#撤销股票000001.CS的挂单
api.cancel_order(symbol="000001.CS", remark="撤销000001.CS的挂单")
#撤销期货rb1901.CF的空头挂单
api.cancel_order(symbol="rb1901.CF", side=EPositionSide.SHORT)
```


## 查询函数

### 获取策略/账户信息
#### get_symbol_position(symbol, side) 获取策略级别单个标的仓位

**函数原型：**

`get_symbol_position(symbol, side=None)`

**参数：**

| 参数   | 类型          | 是否必填 | 说明                                                         |
| ------ | ------------- | -------- | ------------------------------------------------------------ |
| symbol | str           | 是       | 标的代码                                                     |
| side   | EPositionSide | 否       | 多或者空，不填时默认为多仓，股票不能为short。详见[EPositionSide](api5.html#epositionside-持仓方向) 。 |

**返回值：**[SymbolPosition](api5.html#symbolposition-策略的标的持仓)对象。

**举例：**

``` 
#获取股票000001.CS仓位信息
symbolposition = api.get_symbol_position (symbol="000001.CS")  
#获取期货IF1809.CF多仓的仓位信息
symbolposition = api.get_symbol_position (symbol="IF1809.CF", side=EPositionSide.LONG)  
#获取期货IF1809.CF空仓的仓位信息
symbolposition = api.get_symbol_position (symbol="IF1809.CF"，side=EPositionSide.SHORT) 
```


#### get_symbol_positions() 获取所有持仓标的仓位信息


**函数原型：**

`get_symbol_positions()`

**参数：**无

**返回值：**list结构，元素为[SymbolPosition](api5.html#symbolposition-策略的标的持仓)对象。

**举例：**

```
#获取所有标的的仓位信息
symbolpositions = api.get_symbol_positions()  
```


#### get_account(symbol, market) 获取标的或者市场所属账户信息


获取标的所属账号信息，由于账号获取是异步的，所以下完单立即获取账户数据会有延迟。
**函数原型：**

`get_account(symbol=None, market=None)`

**参数：**

| 参数   | 类型 | 是否必填 | 说明                                                         |
| ------ | ---- | -------- | ------------------------------------------------------------ |
| symbol | str  | 否       | 标的代码，可缺省，但不能同时和市场一起缺省，若都不缺省以market为准 |
| market | str  | 否       | 市场，主要有股票MARKET_CHINASTOCK和期货MARKET_CHINAFUTURE    |

**返回值：**标的所属证券的市场账号信息，[Account](api5.html#account-账户信息)结构。

**举例：**

``` 
# 用标的获取A股账号的信息
account = api.get_account(symbol='000001.CS')  
# 用市场获取A股账号的信息
account = api.get_account(market=MARKET_CHINASTOCK)  
# 同时填入标的和市场时，返回通过市场获取的账号信息，如下返回期货账号信息
account = api.get_account(symbol='000001.CS', market=MARKET_CHINAFUTURE)  
```


#### get_overall_position(symbol) 获取标的汇总仓位

由于多个策略可以共享账户资金，因此可能会存在多个策略中都持有某个标的的情况，账户中会将多个策略中标的信息进行汇总。本函数用来获取某个标的在账户中汇总后的仓位信息。

**函数原型：**

`get_overall_position(symbol)`

**参数：**

| 参数   | 类型 | 是否必填 | 说明     |
| ------ | ---- | -------- | -------- |
| symbol | str  | 是       | 标的代码 |

**返回值：**该标的在其所属账户中的总仓位信息，[OverallPosition](api5.html#overallposition-账户的标的持仓)对象结构。

**举例：**

```
#获取账户下000001.CS的汇总仓位信息
overallposition = api.get_overall_position(symbol="000001.CS") # 获取000001.CS仓位信息
availableqty = overallposition.longAvailableQty # 获取多仓可平仓数量 
```


#### get_strategy_PnL-获取策略盈亏信息



**函数原型：**

```
get_strategy_PnL()
```
**参数：**无

**返回值：**策略盈亏信息，[StrategyPnL](api5.html#strategypnl-策略的盈亏信息)对象结构

**举例：**
```
strategyPnL= api.get_strategy_PnL() #获取策略盈亏信息
overallPnL   = strategyPnL.overallPnL   #获取策略总盈亏字段数据
```

### 获取交易日期
#### get_trade_day_interval(symbol, start_date, end_date) 获取交易日数

获取某时间区间交易日数，起止时间只能是回测区间内的

**函数原型：**
`get_trade_day_interval(symbol, start_date, end_date)`

**参数：**

| 参数       | 类型 | 是否必填 | 说明                   |
| ---------- | ---- | -------- | ---------------------- |
| symbol     | str  | 是       | 标的代码               |
| start_date | int  | 是       | 开始时间，格式为YYMMDD |
| end_date   | int  | 是       | 结束时间，格式为YYMMDD |

**返回值：**返回值为交易日数

**举例：**

获取一段时间内的交易日数，包含所传起止日期
```
data = api.get_trade_day_interval(symbol="000002.CS", start_date=20151217, end_date=20160704)
```
返回结果：
```
3  # 过滤停牌日期
```


#### get_prev_trade_date(trade_date, market) 获取上一个交易日期

获取上一个交易日期

**函数原型：**
`get_prev_trade_date(trade_date，market=None)`

**参数：**

| 参数      | 类型 | 是否必填 | 说明                                  |
| --------- | ---- | -------- | ------------------------------------- |
| tradeDate | int  | 是       | 交易日，格式为YYMMDD                  |
| market    | str  | 否       | 市场代码，不填默认返回A股市场的交易日 |
**返回值：**返回上一交易日的日期

**举例：**

```
api.get_prev_trade_date(trade_date=20150101) # 返回值为20141231
api.get_prev_trade_date(trade_date=20150101, market="CS") # 返回值为20141231
```


#### get_next_trade_date(trade_date, market) 获取下一个交易日期

获取下一个交易日期

**函数原型：**
`get_next_trade_date(trade_date，market=none)`

**参数：**

| 参数       | 类型 | 是否必填 | 说明                                  |
| ---------- | ---- | -------- | ------------------------------------- |
| trade_date | int  | 是       | 开始时间，格式为YYMMDD                |
| market     | str  | 否       | 市场代码，不填默认返回A股市场的交易日 |
**返回值：**返回下一交易日的日期

**举例：**

```
api.get_next_trade_date(trade_date=20150101) # 返回值为20150105
api.get_next_trade_date(trade_date=20150101, market='CS') # 返回值为20150105
```


#### get_trade_dates(start_date, end_date, market, period) 获取一段时间内交易日期

指定起止日期，获取某个市场一段时间的交易日期

**函数原型：**
`get_trade_dates(start_date, end_date, market=None，period=('D', 1))`

**参数：**

| 参数       | 类型  | 是否必填 | 说明                                                         |
| ---------- | ----- | -------- | ------------------------------------------------------------ |
| start_date | int   | 是       | 开始时间（含当天），格式为YYMMDD                             |
| end_date   | int   | 是       | 结束时间（含当天），格式为YYMMDD                             |
| market     | str   | 否       | 交易市场,不填默认返回A股市场的交易日                         |
| period     | tuple | 否       | 变频参数，D-每天一值、W-每周一值、M-每月一值、Q-每季度一值、S-每半年一值、Y-每年一值;1-第一个交易日，-1-最后一个交易日;不填默认返回全部即(D,1) |

**返回值：**交易日list，日期格式YYYMMDD，类型为int

**举例：**
获取一段时间内的交易日期，包含所传起止日期
```
api.get_trade_dates(start_date=20171001, end_date =20171011)
```
返回结果：
```
[20171009, 20171010, 20171011]
```


#### get_prev_trade_dates(trade_date, count, market, period) 获取某个市场截止指定日期的多个交易日期

指定截止日期，返回某个市场指定天数的交易日期


**函数原型：**
`get_prev_trade_dates(trade_date, count, market=None，period=('D', 1))`

**参数：**

| 参数      | 类型        | 是否必填 | 说明                                                         |
| --------- | ----------- | -------- | ------------------------------------------------------------ |
| tradeDate | date        | 是       | 截止日期（含当天），格式为YYMMDD                             |
| count     | int         | 是       | 返回天数（含当天），count需大于0                             |
| mktIndex  | str or list | 否       | 交易市场,不填默认返回A股市场的交易日                         |
| period    | tuple       | 否       | 变频参数，D-每天一值、W-每周一值、M-每月一值、Q-每季度一值、S-每半年一值、Y-每年一值;1-第一个交易日，-1-最后一个交易日;不填默认返回全部即(D,1) |

**返回值：**交易日list，日期格式YYYMMDD，类型为int

**举例：**

```
api.get_prev_trade_dates(20181008,7) # 返回A股市场截止20181008日7个交易日日期
```
返回结果
```
 [20180920, 20180921, 20180925, 20180926, 20180927, 20180928, 20181008]
```


#### date_now() 获取当前交易日期

获取当前的交易日期。回测情况下是回测历史当天，交易情况下是当前交易日。

**函数原型：**
`date_now()`

**参数：** 无

**返回值：**返回历史当天的交易日期，期货夜盘默认归为后一交易日, api.context.time类型


#### time_now() 获取当前交易时间


获取当前的交易时间。回测情况下是回测历史当天，交易情况下是当前交易日。

**函数原型：**

`time_now()`

**参数：** 无

**返回值：**返回历史当前的时间, api.context.time类型

### 获取标的信息
#### get_ref_data(symbol) 获取标的基本信息

**函数原型：**
`get_ref_data(symbol)`

**参数：**

| 参数   | 类型 | 是否必填 | 说明     |
| ------ | ---- | -------- | -------- |
| symbol | str  | 是       | 标的代码 |

**返回值：**证券基本信息，[RefData](api5.html#refdata-股票基础信息)。

**举例：**

```
# 获取000001.CS证券基本信息
ref_data = api.get_ref_data(symbol="000001.CS") 
```


#### get_constituent_symbols(symbol_set, trade_date) 获取成分股数据

**函数原型：**

`get_constituent_symbols(symbol_set, trade_date=None)`

获取股票指数或者行业板块的成分股数据。获取期货某品种当前可交易的标的数据。详见指数代码说明。

**参数：**

| 参数       | 类型        | 是否必填 | 说明                                                         |
| ---------- | ----------- | -------- | ------------------------------------------------------------ |
| symbol_set | str or list | 是       | 指数代码、板块代码、指数代码列表或者行业板块代码列表，为列表时表示获取的股票同时属于这些指数或者行业。 |
| trade_date | int         | 否       | 交易日，形式为YYYYMMDD，支持获取历史成分股信息，不填默认返回当天的数据 |

**返回：** 返回股票代码的list

**举例：**

获取上证指数成分股
``` 
symbols = api.get_constituent_symbols(symbol_set="000001.IDX", trade_date=20170403) 
```
返回结果：
```
['600023.CS', '600908.CS', '600909.CS', '600917.CS', '600919.CS', '600926.CS', '600936.CS', '600939.CS', '600958.CS', '600959.CS', '600977.CS', '600996.CS', '600998.CS', '600999.CS', '601000.CS', '601002.CS', '601003.CS', '601005.CS', '601007.CS', '601008.CS', '601009.CS', '601010.CS', '601011.CS', '601012.CS', '601015.CS', '601016.CS', '601018.CS', '601020.CS', '601021.CS', '601028.CS', '601038.CS', '601058.CS', '601069.CS', '601088.CS', '601098.CS', '601099.CS', '601100.CS', '601101.CS', '601106.CS', ...]
```
获取RU的可以交易标的列表
``` 
ru_now = api.get_constituent_symbols(symbol_set="ru.PRD", trade_date=20170403)  
```
返回结果：
```
['ru1704.CF', 'ru1705.CF', 'ru1706.CF', 'ru1707.CF', 'ru1708.CF', 'ru1709.CF', 'ru1710.CF', 'ru1711.CF', 'ru1801.CF', 'ru1803.CF']
```
获取优品行业板块集合的列表
```
up_now = api.get_constituent_symbols(symbol_set="UPPLA.SET", trade_date=20170403)  

```
返回结果：
```
['880001.PLA', '880002.PLA', '880003.PLA', '880004.PLA', '880007.PLA', '880008.PLA', '880009.PLA', '880010.PLA', '880012.PLA', '880013.PLA', '880014.PLA', '880015.PLA', '880016.PLA', '880017.PLA', '880018.PLA', '880019.PLA', '880020.PLA', '880021.PLA', '880022.PLA', '880023.PLA', '880024.PLA', '880025.PLA', '880026.PLA', '880027.PLA', '880028.PLA', '880029.PLA', '880030.PLA', '880031.PLA', '880033.PLA', '880035.PLA', '880039.PLA', '880041.PLA', '880042.PLA', '880043.PLA', '880044.PLA', '880045.PLA', '880046.PLA', '880047.PLA', '880048.PLA', '880049.PLA', '880050.PLA', '880051.PLA', '880052.PLA', '880053.PLA', '880056.PLA', '880057.PLA', '880058.PLA', '880059.PLA', '880060.PLA', '880064.PLA', '880065.PLA']
```


#### get_continuous_symbol(symbol, trade_date) 获取连续合约对应的标的


**函数原型：**

`get_continuous_symbol(symbol, trade_date=None)`

传入连续合约代码，获取对应的实际的标的代码。期货专用。

**参数：**

| 参数       | 类型 | 是否必填 | 说明                                                         |
| ---------- | ---- | -------- | ------------------------------------------------------------ |
| symbol     | str  | 是       | 连续合约的代码，包括主力、次主力合约，当月、次月、下季、隔季连续合约 |
| trade_date | int  | 否       | 指定某个交易日，不填则默认为回测当天。形式为YYYYMMDD         |

**返回：**连续合约对应的期货代码。

**举例：**

获取主力合约在某交易日对应的标准合约

`api.get_continuous_symbol(symbol="jmZ0.CF", trade_date=20180820)`

返回结果：

`jm1901.CF`




#### get_appointed_symbols(mode, trade_date) 获取指定合约集合


获取指定合约集合，如主力合约集合，次主力合约集合。

**函数原型：**
`get_appointed_symbols(mode, trade_date=None)`

**参数：**

| 参数       | 类型 | 是否必填 | 说明                                            |
| ---------- | ---- | -------- | ----------------------------------------------- |
| mode       | str  | 是       | 指定合约的类型，有Z0、Z1，Y1-Y12                |
| trade_date | int  | 否       | 交易日 ，形式为YYYYMMDD，不填默认返回当天的数据 |

**返回：** 返回指定合约集合

**举例：**

获取20180710当天的所有主力合约
```
api.get_appointed_symbols(mode='Z0', trade_date=20180710)
```
返回结果
```
['APZ0.CF', 'CFZ0.CF', 'CYZ0.CF', 'FGZ0.CF', 'ICZ0.CF', 'IFZ0.CF', 'IHZ0.CF', 'JRZ0.CF', 'LRZ0.CF', 'MAZ0.CF', 'OIZ0.CF', 'PMZ0.CF', 'RIZ0.CF', 'RMZ0.CF', 'RSZ0.CF', 'SFZ0.CF', 'SMZ0.CF', 'SRZ0.CF', 'TAZ0.CF', 'TFZ0.CF', 'TZ0.CF', 'WHZ0.CF', 'ZCZ0.CF', 'aZ0.CF', 'agZ0.CF', 'alZ0.CF', 'auZ0.CF', 'bZ0.CF', 'bbZ0.CF', 'buZ0.CF', 'cZ0.CF', 'csZ0.CF', 'cuZ0.CF', 'fbZ0.CF', 'fuZ0.CF', 'hcZ0.CF', 'iZ0.CF', 'jZ0.CF', 'jdZ0.CF', 'jmZ0.CF', 'lZ0.CF', 'mZ0.CF', 'niZ0.CF', 'pZ0.CF', 'pbZ0.CF', 'ppZ0.CF', 'rbZ0.CF', 'ruZ0.CF', 'scZ0.CF', 'snZ0.CF', 'vZ0.CF', 'wrZ0.CF', 'yZ0.CF', 'znZ0.CF']
```
获取当天（20181127）所在月份为1的所有合约，若某标的同时存在1901和2001则返回较近的合约，即1901
```
api.get_appointed_symbols(mode="Y1")
```
返回结果
```
['OI1901.CF', 'ZC1901.CF', 'cu1901.CF', 'y1901.CF', 'WH1901.CF', 'bb1901.CF', 'FG1901.CF', 'rb1901.CF', 'sc1901.CF', 'sn1901.CF', 'RI1901.CF', 'm1901.CF', 'CY1901.CF', 'SR1901.CF', 'al1901.CF', 'a1901.CF', 'c1901.CF', 'JR1901.CF', 'fb1901.CF', 'i1901.CF', 'l1901.CF', 'RM1901.CF', 'IH1901.CF', 'ag1901.CF', 'IC1901.CF', 'IF1901.CF', 'au1901.CF', 'ru1901.CF', 'zn1901.CF', 'pp1901.CF', 'p1901.CF', 'j1901.CF', 'jd1901.CF', 'TA1901.CF', 'MA1901.CF', 'ni1901.CF', 'bu1901.CF', 'pb1901.CF', 'hc1901.CF', 'wr1901.CF', 'jm1901.CF', 'cs1901.CF', 'v1901.CF', 'b1901.CF', 'SF1901.CF', 'LR1901.CF', 'SM1901.CF', 'PM1901.CF', 'CF1901.CF', 'AP1901.CF', 'fu1901.CF']
```
#### is_suspend(symbol, trade_date) 是否停牌

判断标的当日是否停止交易


**函数原型：**
`is_suspend(symbol, trade_date=None)`

**参数：**

| 参数       | 类型 | 是否必填 | 说明                                                         |
| ---------- | ---- | -------- | ------------------------------------------------------------ |
| symbol     | str  | 是       | 策略中的证券代码                                             |
| trade_date | int  | 否       | 交易日，形式为YYYYMMDD，可以查询历史上的状态，不填默认返回当天的数据 |

**返回值：**返回True表示停牌，返回False表示未停牌

**举例：**
```
api.is_suspend(symbol="000001.CS",trade_date=20180104) # 获取000001.CS在20180104是否停牌
```


#### is_ST(symbol, trade_date) 是否ST股

判断标的当日是否ST股

**函数原型：**
`is_ST(symbol, trade_date=None)`

**参数：**

| 参数       | 类型 | 是否必填 | 说明                                                         |
| ---------- | ---- | -------- | ------------------------------------------------------------ |
| symbol     | str  | 是       | 策略中的证券代码                                             |
| trade_date | int  | 否       | 交易日，形式为YYYYMMDD，可以查询历史上的状态，不填默认返回当天的数据 |

**返回值：**返回True表示ST股，返回False表示非ST股

**举例：**
```
api.is_ST(symbol ="000001.CS",trade_date =20180104)#获取000001.CS在20180104是否ST
```


#### is_listed(symbol, trade_date) 是否上市

判断标的当日是否上市


**函数原型：**
`is_listed(symbol, trade_date=None)`

**参数：**

| 参数       | 类型 | 是否必填 | 说明                                                         |
| ---------- | ---- | -------- | ------------------------------------------------------------ |
| symbol     | str  | 是       | 策略中的证券代码                                             |
| trade_date | int  | 否       | 交易日，形式为YYYYMMDD，可以查询历史上的状态，不填默认返回当天的数据 |

**返回值：**返回True表示上市，返回False表示未上市或者已退市

**举例：**
```
api.is_listed(symbol="000001.CS",trade_date=20180104) # 获取000001.CS在20180104是否上市
```

### 获取标的数据
#### get_bars_history(symbol, time_span, count, price_mode, skip_suspended, fields) 获取历史K线数据

获取K线数据。模拟盘和实盘在on_handle_data 和 on_bar中获取当天的日K不是全日K线，是开盘时间到所设置的收盘时间的日K。

**注意：** 调用此api之前需在初始化[initialize](api5.html#从一个简单的策略示例说起)中设置所需的symbols和prepared_bars，否则可能取不到字段值。

**函数原型：**
`get_bars_history(symbol, time_span, count=None，price_mode=EPriceMode.REAL, skip_suspended=True, fields=None)`

**参数：**

| 参数           | 类型        | 是否必填 | 说明                                                         |
| -------------- | ----------- | -------- | ------------------------------------------------------------ |
| symbol         | str         | 是       | 标的代码                                                     |
| time_span      | ETimeSpan   | 是       | bar的频率，详见 [ETimeSpan](api5.html#etimespan-频率类型)。    |
| count          | int         | 是       | 获取历史数据条数，超过缓存类型的返回空                       |
| price_mode     | EPriceMode  | 否       | 复权模式，不填默认返回不复权，详见[EPriceMode](api5.html#epricemode-复权模式) 。 |
| skip_suspended | bool        | 否       | 为了保证取不同标的数据时时间对齐，可以设置是否跳过停牌， 不填写默认为跳过。若不跳过，填充方式为高开低收均用前收盘价填充，成交量=成交额=0。 |
| fields         | str or list | 否       | 返回的字段，不填默认返回全部。                               |

**返回值：**行情数据 ,默认返回dataFrame格式。

**举例：**

获取不过滤停牌的前复权5分钟K线2支
```
# 然后在对应事件回调中获取历史K线
bars = api.get_bars_history(symbol ="000001.CS", time_span=ETimeSpan.MIN_5, count=2, price_mode=EPriceMode.FORMER, skip_suspended=0)  
```

返回结果：

``` 
[{ "symbol": "000001.CS", "timeSpan": 300, "timeStop": 1543215300000, "timeStr": "20181126-145500-000", "tradeDate": 20181126, "close": 10.32, "high": 10.33, "isSuspended": False, "low": 10.31, "open": 10.32, "position": 0, "preClose": 10.32, "preSettle": 0, "settle": 0, "totalTurnover": 5.24417e+08, "totalVolume": 5.05396e+07, "turnover": 1.37453e+07, "volume": 1.33236e+06 }, { "symbol": "000001.CS", "timeSpan": 300, "timeStop": 1543215600000, "timeStr": "20181126-150000-000", "tradeDate": 20181126, "close": 10.34, "high": 10.34, "isSuspended": False, "low": 10.32, "open": 10.32, "position": 0, "preClose": 10.32, "preSettle": 0, "settle": 0, "totalTurnover": 5.35963e+08, "totalVolume": 5.16569e+07, "turnover": 1.15462e+07, "volume": 1.1173e+06 }]
```

某个字段值：
```
bars[0].totalTurnover  # 交易日总成交额
```
返回结果：
```
524417138.0
```

获取不过滤停牌的不复权5分钟K线2支，格式为DataFrame
``` 
bars = api.get_bars_history(symbol='000001.CS', time_span=ETimeSpan.MIN_5, count=2) 
```
返回结果：
``` 
               timeStr       timeStop     symbol  tradeDate  close   high    low   open  preClose  settle  preSettle     volume    turnover  totalVolume  totalTurnover  position  isSuspended
0  20181126-145500-000  1543215300000  000001.CS   20181126  10.32  10.33  10.31  10.32     10.32     0.0        0.0  1332361.0  13745256.0   50539573.0    524417138.0       0.0        False
1  20181126-150000-000  1543215600000  000001.CS   20181126  10.34  10.34  10.32  10.32     10.32     0.0        0.0  1117298.0  11546172.0   51656871.0    535963310.0       0.0        False
```

获取不过滤停牌的不复权5分钟K线2支，格式为DataFrame，返回列为tradeDate和close，该返回值必然会 
```
bars = api.get_bars_history(symbol='000001.CS',timeSpan=ETimeSpan.MIN_5, count=2, fields=['tradeDate', 'close'])  
```
返回结果：
```
   tradeDate  close
0   20181126  10.32
1   20181126  10.34
```

#### get_value_field(symbols, fields, start_date, end_date, count) 获取数值类因子


获取数值类因子数据。数据信息详见[股票交易指标-日频](data7.html#股票交易指标-日频)。

调用此api之前需在初始化[initialize](api5.html#从一个简单的策略示例说起)中设置缓存的symbol和field，否则可能取不到字段值

**函数原型**
`get_value_field(symbols, fields=None, start_date=None, end_date=None, count=None)`

**参数：**

| 参数       | 类型        | 是否必填 | 说明                                                       |
| ---------- | ----------- | -------- | ---------------------------------------------------------- |
| symbols    | str or list | 是       | 股票或者股票列表                                           |
| fields     | str or list | 是       | 选取的股票属性，可以选择多个                               |
| start_date | int         | 否       | 指定某个交易日，不填则默认其他方式返回                     |
| end_date   | int         | 否       | 指定某个交易日，不填则默认为回测上一交易日。形式为YYYYMMDD |
| count      | int         | 否       | 指定从end_date往前count个交易日的时间段                    |

**返回：**返回DataFrame格式

**举例：**

获取多只标的在某交易日的因子数据
``` 
data = api.get_value_field(symbols=["000002.CS", "600000.CS"], fields=["PE", "TRADE_STA"], start_date=20181122)
```
返回结果：
``` 
       PE  TRADE_STA     symbol  tradeDate
0  5.6400       True  000002.CS   20181122
1  5.5501       True  600000.CS   20181122
```

获取某标的在一段时间内的因子数据
``` 
data = api.get_value_field(symbols="000001.CS", fields=["PE","REVENUE","TRADE_STA", "TOT_SHARE", "LIST_STA"], start_date=20181116, end_date=20181120)
```
返回结果：
``` 
   LIST_STA      PE       REVENUE     TOT_SHARE  TRADE_STA     symbol  tradeDate
0         1  7.4102  8.666400e+10  1.717041e+10       True  000001.CS   20181116
1         1  7.6065  8.666400e+10  1.717041e+10       True  000001.CS   20181119
2         1  7.4102  8.666400e+10  1.717041e+10       True  000001.CS   20181120
```
获取字段值
```
data.ix[0:1,["REVENUE","PE"]] #获取前两行的"REVENUE","PE"字段值
```
返回结果：
``` 
        REVENUE      PE
0  8.666400e+10  7.4102
1  8.666400e+10  7.6065
```

获取基于某交易日往前推的两条数据，格式为dataFrame
``` 
datas = api.get_value_field(symbols="000001.CS", fields=["TOT_SHARE", "BASIC_EPS", "PE"], start_date=20181122, count=2)
```
返回结果：
``` 
   BASIC_EPS      PE     TOT_SHARE     symbol  tradeDate
0       1.14  7.4383  1.717041e+10  000001.CS   20181121
1       1.14  7.3962  1.717041e+10  000001.CS   20181122
```




#### get_finance_fields(symbols, fields, trade_date, report) 获取财务数据(单交易日)


获取股票的财务数据。数据信息详见[股票财务数据（季频）](data7.html#股票财务数据(季频))。

调用此api之前需在初始化[initialize](api5.html#从一个简单的策略示例说起)中设置相关股票池和所需因子，否则可能取不到字段值

**函数原型**
`get_finance_field(symbols, fields=None, trade_date=None, report=None)`

**参数：**

| 参数       | 类型        | 是否必填 | 说明                                                         |
| ---------- | ----------- | -------- | ------------------------------------------------------------ |
| symbols    | str         | 是       | 股票代码或列表                                               |
| fields     | str or list | 是       | 选取的股票属性，可以选择多个                                 |
| trade_date | int         | 否       | 交易日，形式为YYYYMMDD，为了避免未来函数只能获取到所传交易日之前的数据，不填默认返回上一交易日 |
| report     | str         | 否       | 指定报告期，例如'2016Q1' - 16年一季报，'2016Q2' - 16年半年报，'2016Q3' - 16年三季报，'2016Q4'-年报，report缺省时返回标的最新的报告期数据。 |

**返回：**返回DataFrame格式

**举例：**

获取000001.CS的2016年年报数据，当前交易日为20181126
``` 
data = api.get_finance_field(symbols="000001.CS", fields=["REVENUE", "NET_PROFIT", "NET_PROFIT_ATTRP"], trade_date=20170505, report="2016Q4")
```
返回结果：
``` 
     NET_PROFIT  NET_PROFIT_ATTRP       REVENUE  reportdate     symbol  tradeDate
0  2.259900e+10      2.259900e+10  1.077150e+11    20161231  000001.CS   20170505
```
#### get_table_field(symbol, field, start_date, end_date, count, columns) 获取表格类因子数据

表格类因子数据，数据信息详见[表格型数据](data7.html#表格型数据)


由于表格类数据的数据量相对较大，因此只提供单标的单因子的数据取用接口

**函数原型：**
`get_table_field(symbol, field, start_date=None, end_date=None, count=None, columns=[])`

**参数：**

| 参数       | 类型 | 是否必填 | 说明                                                       |
| ---------- | ---- | -------- | ---------------------------------------------------------- |
| symbol     | str  | 是       | 股票代码                                                   |
| field      | str  | 是       | 因子名 ，详见[平台数据](data7.html#平台数据)                 |
| start_date | int  | 否       | 指定某个交易日，不填则默认为回测上一交易日。形式为YYYYMMDD |
| end_date   | int  | 否       | 指定起始交易日                                             |
| count      | int  | 是       | 获取基于所传交易日往前推的N条数据（包含所传交易日当天）    |
| columns    | list | 否       | 返回的字段；为空时,默认返回所有列,否则返回指定的列         |

**返回：**返回DataFrame格式

**举例：**

获取a1901.CF上一交易日成交量排名列表

``` 
api.get_table_field(symbol='a1901.CF', field='FUT_TRADE_RANK', start_date=20181120, columns=['rank', 'name', 'volume', 'volumeDiff'])
```
返回结果：
```
        name  rank  tradeDate   volume  volumeDiff
0   东证期货     1   20181120  14924.0      6320.0
1   中信期货     2   20181120  14329.0      5721.0
2   海通期货     3   20181120   7742.0      1050.0
3   国泰君安     4   20181120   7029.0      1654.0
4   华泰期货     5   20181120   6147.0      2323.0
5   华安期货     6   20181120   5279.0      -899.0
...
14  中信建投    15   20181120   3096.0      1776.0
15  银河期货    16   20181120   3059.0       543.0
16  广发期货    17   20181120   2804.0       438.0
17  南华期货    18   20181120   2392.0      -231.0
18  渤海期货    19   20181120   2318.0      1766.0
19  上海大陆    20   20181120   2245.0      1813.0
```
获取申万一级行业板块的K线数据
``` 
api.get_table_field('801780.PLA', 'SW_MKT', 20181122)
```
返回结果：
``` 
     close     high      low     open  preClose  totalTurnover   totalVolume  tradeDate
0  3302.34  3320.06  3291.49  3320.06   3314.79    752355335.0  5.570303e+09   20181122
```

获取a1901.CF多个交易日的成交量排名列表

``` 
api.get_table_field(symbol='a1901.CF', field='FUT_TRADE_RANK', start_date=20181122, end_date=20181124, columns=['rank', 'name', 'volume', 'volumeDiff']) 
```
返回结果为：
```
         name  rank  tradeDate   volume  volumeDiff
0   东证期货     1   20181122   8705.0     -4422.0
1   中信期货     2   20181122   7001.0     -6352.0
2   国泰君安     3   20181122   5417.0     -1273.0
3   海通期货     4   20181122   4833.0     -3542.0
4   方正中期     5   20181122   4043.0      -619.0
5   华泰期货     6   20181122   3838.0     -2074.0
6   国信期货     7   20181122   3259.0      -308.0
7   徽商期货     8   20181122   3174.0       173.0
8   渤海期货     9   20181122   3116.0      1557.0
9   兴证期货    10   20181122   2990.0     -2049.0
...
31  国投安信    12   20181123   4174.0      2147.0
32  中信建投    13   20181123   3563.0       738.0
33  大有期货    14   20181123   3199.0      1939.0
34  渤海期货    15   20181123   3125.0         9.0
35  永安期货    16   20181123   2979.0       453.0
36  银河期货    17   20181123   2870.0      1230.0
37  宏源期货    18   20181123   2575.0      1276.0
38  东航期货    19   20181123   2250.0       663.0
39  广发期货    20   20181123   2040.0       729.0

```
获取申万一级行业板块多个交易日的K线数据

```
api.get_table_field('801010.PLA', "SW_MKT", start_date=20181122, end_date=20181123)
```
返回结果为：
```
	close     high      low     open  preClose  totalTurnover   totalVolume  tradeDate
0  2272.95  2293.86  2264.67  2289.74   2286.45   8.149344e+08  5.235480e+09   20181122
1  2230.37  2276.36  2222.91  2276.36   2272.95   1.033341e+09  6.586772e+09   20181123
```

获取a1901.CF多个交易日成交量排名列表

```
api.get_table_field(symbol='a1901.CF', field='FUT_TRADE_RANK', start_date=20181122, count=2, columns=['rank', 'name', 'volume', 'volumeDiff'])

```
返回结果为：
```
    name  rank  tradeDate   volume  volumeDiff
0   中信期货     1   20181121  13353.0      -976.0
1   东证期货     2   20181121  13127.0     -1797.0
2   海通期货     3   20181121   8375.0       633.0
3   国泰君安     4   20181121   6690.0      -339.0
4   华泰期货     5   20181121   5912.0      -235.0
5   兴证期货     6   20181121   5039.0       138.0
6   方正中期     7   20181121   4662.0       659.0
7   国信期货     8   20181121   3567.0      -131.0
8   国投安信     9   20181121   3531.0      -988.0
9   华安期货    10   20181121   3369.0     -1910.0
10  光大期货    11   20181121   3269.0      -502.0
11  徽商期货    12   20181121   3001.0     -1614.0
...
30  华安期货    11   20181122   2890.0      -479.0
31  中信建投    12   20181122   2825.0       512.0
32  永安期货    13   20181122   2526.0       339.0
33  光大期货    14   20181122   2249.0     -1020.0
34  国投安信    15   20181122   2027.0     -1504.0
35  南华期货    16   20181122   2024.0       183.0
36  银河期货    17   20181122   1640.0     -1070.0
37  东航期货    18   20181122   1587.0      -132.0
38  浙商期货    19   20181122   1570.0       273.0
39  安粮期货    20   20181122   1386.0      -171.0
```

获取申万一级行业板块多个交易日的K线数据

```
api.get_table_field(symbol='801010.PLA', field="SW_MKT", start_date=20181122, count=3)
```
返回结果如下：
``` 
     close     high      low     open  preClose  totalTurnover   totalVolume  tradeDate
0  2263.10  2312.21  2261.69  2307.56   2320.63   1.143088e+09  7.398032e+09   20181120
1  2286.45  2288.83  2240.66  2251.17   2263.10   1.015610e+09  6.188521e+09   20181121
2  2272.95  2293.86  2264.67  2289.74   2286.45   8.149344e+08  5.235480e+09   20181122
```

### 更多

#### get_dynamic_value(field) 获取动态运行参数

在web页面上创建策略时可以预先设置动态参数，在策略中可以通过此接口获取到该策略的动态参数信息。页面上对参数的改动可以立即反映在策略中。

**函数原型：**
`get_dynamic_value(field)`

**参数：**

| 参数  | 类型 | 是否必填 | 说明     |
| ----- | ---- | -------- | -------- |
| field | str  | 是       | 参数名称 |

**返回值：** 参数值。根据预设的参数类型返回值类型。

**举例：**
```
high=api.get_dynamic_value('high') # 在页面配置了名称为high的参数，获取当前策略high参数
```


#### get_define_data-获取命令行参数
**函数原型：**

`get_define_data(name, default_value, dtype)`

**参数：**

| 参数          | 类型  | 是否必填 | 说明                        |
| ------------- | ----- | -------- | --------------------------- |
| name          | str   | 是       | 命令行传入的名称            |
| default_value | dtype | 否       | 默认值为空，类型和dtype一致 |
| dtype         | type  | 是       | 类型，有str、int等          |

**举例：**

``` 
在命令行输入如下命令 python main.py --name=upchina --value=20，在策略里可以通过api.get_define_data()获取，形式如下：
api.get_define_data(name='name', default_value='myname', dtype=str)
api.get_define_data(name='value', default_value=0, dtype=int)

上一个结果会获取到'upchina'
下一个结果获取到20
```

**示例：**

为了能动态的调整策略参数，可以通过命令行传入参数值，举例如下：

新建策略文件example.py，通过N条历史K线的收盘价的平均值和当前收盘价作比较进行下单，运行结束后调用on_terminate事件，将结果存储于result.txt文件中，代码如下：
``` 
from etasdk import *
import time

def on_initialize(api):
    print("on_initialize")
    #获取命令行参数
    api.bar_len = api.get_define_data('bar_len', 10, int)
    ...

def on_before_market_open(api, data):
    print("time----",data)
    api.context.pool.focus = [...]

def on_bar(api, bar):
    ...

def on_terminate(api, exit_info):
    #存储结果
    api.save_object('result-{}-{}.obj'.format(api.bar_len, int(time.time())), api.props(exit_info))
```
创建一个用于批处理的文件batch.py，代码如下：
``` 
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import subprocess

# 调参, 来优化策略的参数
n_bar_lenth = [i for i in range(10, 20, 2)]

for l in n_bar_lenth:
    args = ' --bar_len=' + str(l)
    print args
    status = subprocess.call("python strategy.py" + args, shell=True)

#运行会循环bar_len, 动态调整参数, 调用策略, 并在策略中保存最后的结果

```
通过运行batch.py文件，会循环调用main.py，K线条数分别是10、12、14、16、18，循环运行五次结果会存储在result-x-xxxxxx.obj文件中，通过对比几次运行结果的数据，来选择最优的参数，以上举例仅供参考。



#### log.*-日志输出

**函数原型：**


```
log.debug(msg) #DEBUG级别日志
log.info(msg) #INFO级别日志
log.error(msg) #ERROR级别日志
```

**参数：**

| 参数 | 类型 | 说明     |
| ---- | ---- | -------- |
| msg  | str  | 日志内容 |
**返回值：**日志打印的内容


**举例：**

日志打印的级别与config配置中的"loglevel"级别一一对应，即loglevel为info就调用log.info，loglevel为error就调用log.error输出日志。

```
log.info("current tradedate {}, last tradedate {}".format(curr,prev)) # 打印当前交易日和上一交易日
log.info("current tradedate = {} ".format(curr)) # 打印当前交易日
```
策略打印日志注意事项：
```
1、尽量不要输出中文，不同的python环境，输出中文后，有可能出现莫名其妙的问题
2、选股、下单等依据数据的关键数据点，要用 log.info 输出日志，这样模拟盘会把日志输出到algo.python_main.log 这个文件里，回测的时候不会输出，因为回测日志级别默认是error，如果回测也想打印日志可以用log.error或者将config中的日志级别改成info（后者会影响回测速度）； 
3、更重要的数据，可以调用 api.send_custom_msg 把日志信息输出到web平台上，但是这个接口不要调用太多，有性能问题
```


#### send_custom_message(msg) 设置用户自定义运行消息



设置用户自定义运行消息，会显示在界面上的日志中

**函数原型：**

`sendCustomMsg(msg)`

**参数：**

| 参数 | 类型 | 说明 |
| ---- | ---- | ---- |
| msg  | str  | 内容 |

**返回值：**无

