# ~~API文档（旧版）~~
## 一个简单的策略
以下是一个简单的买入标的池中的标的并持有的策略，用来说明一个基本的策略完成所需的模块，更多的策略详见[策略示例](starategy6.html#策略示例)。 
```
from etasdk import *   
#初始化
def onInitialize(api): 
    api.setSymbolPool(instsets=["000016.IDX"])
    pass

#盘前运行，必须实现，用来设置当日关注的标的
def onBeforeMarketOpen(api, tradeDate):
    symbolPool = api.getSymbolPool()  # 获取初始化中设置的标的池
    print("symbolPool---", len(symbolPool))
    symbols = [] 
    for symbol in symbolPool:
        symbols.append(symbol)
        print(symbol)
    api.setFocusSymbols(symbols)   # 从标的池中筛选标的设置为关注，被关注和在持仓中的标的当天能够收到行情推送

#Bar调用
def onBar(api, bar):      
    print ("onbar: %d"%bar.tradeDate)
    api.targetPosition(bar.symbol, 100)  #对该标的进行下单

#运行终止调用
def onTerminate(api, exitInfo): 
    print ("onTerminate")
    print (exitInfo)


```

## 深度理解API（必看）
### 主要问题
为了高效的使用API, 需要对平台的机制有一定的了解. 优品量化的平台最核心的特点是SDK模式, 本机安装SDK后, 策略基于SDK实现, 和云端量化引擎通讯来完成策略的运行, 同时策略运行过程中, 数据也需要自动的从云端拉取到本地. 那么为了保证策略运行的高效性,  就需要考虑如下几个问题:
- 这么大量的数据, 如何保证拉取数据的效率?
- 策略运行过程中, 播放K线时如果保证效率?

为了解决这两个问题, API提供了相关机制来控制和优化效率:

### 设置数据
在onInitialize中调用setRequireData来设置需要的数据:
- 个股, 标的池(市场, 板块,期货品种等)
- K线类型和历史根数
- A股各类因子
```
#设置股票池、相关标的因子数据和K线。
api.setRequireData(instsets=["000016.IDX","cu.PRD"],symbols=["IFZ0.CF","000001.CS"],fields=["CIRC_CAP", "TURNOVER_RATE", "PE"],bars=[(ETimeSpan.DAY_1,2),(ETimeSpan.MIN_1,2)]) 
#设置股票池、相关标的因子数据和K线。
api.setRequireData(instsets=["000016.IDX","cu.PRD"],fields=["CIRC_CAP", "TURNOVER_RATE", "PE"],bars=[(ETimeSpan.DAY_1,2),(ETimeSpan.MIN_1,2)])
#设置股票池、相关标的因子数据和K线。
api.setRequireData(symbols=["IFZ0.CF","000001.CS"],fields=["CIRC_CAP", "TURNOVER_RATE", "PE"],bars=[(ETimeSpan.DAY_1,2),(ETimeSpan.MIN_1,2)])
#设置股票池和K线。
api.setRequireData(instsets=["000016.IDX","cu.PRD"],symbols=["IFZ0.CF","000001.CS"],bars=[(ETimeSpan.DAY_1,2),(ETimeSpan.MIN_1,2)])
#设置股票池和相关标的因子数据
api.setRequireData(instsets=["000016.IDX","cu.PRD"],symbols=["IFZ0.CF","000001.CS"],fields=["CIRC_CAP", "TURNOVER_RATE", "PE"])
```
onInitialize在策略启动后只会调用一次, 在策略运行过程中, 只会拉取相关的数据. 因此在实现策略过程中, 尽量只设置自己需要的数据, 避免拉取太多数据, 这有助于提高策略运行的性能.

### 减少K线播放量
策略运行过程中, 云端量化引擎会播放相应的K线来驱动策略运行.SDK中的onBeforeMarketOpen每个交易日开盘前会响应一次, 策略可以在这个函数中执行选股逻辑, 然后调用setFocusSymbols来关注选出来的标的.

注意:  只有setFocusSymbols的股票以及持仓的标的才会收到当天的行情推送.  

比如:在onInitialize中, api.setRequireData设置了A股全市场(CS.SET), 但是每天选股过程中只会选取几只股票, 并对这几只股票做买卖操作, 因此只需要setFocusSymbols这几只股票, 从而大幅度提升了效率.

### K线的三种播放机制
- onHandleData模式: onBeforeMarketOpen中focus了多少标的, 都只会回调onHandleData一次.
- onBar模式: onBeforeMarketOpen中setFocusSymbols的标的+持仓标的, 每个标的都会收到onBar的回调, 比如 setFocusSymbols标的+持仓标的有10只, 播放是日K行情, 那么当天onBar会有10次回调;
- onHandleData/onBar混合模式: 两者都会回调

onInitialize中通过调用api.setGroupMode来控制播放模式, 默认不调用api.setGroupMode则是onBar模式
```
#设置只响应onHandleData，超时时间为5000ms
api.setGroupMode(timeOutMs=5000, onlyGroup=True)
#设置既响应onHandleData，也响应onBar，超时时间为10000ms
api.setGroupMode(timeOutMs=10000, onlyGroup=False)
```
timeOutMs用于在模拟盘/实盘情况下, 如果超过timeOutMs, 相应的K线还没有到齐, 则不再等待, 直接回调onHandleData.

##  数据结构
### 	RefData-股票基础信息

| 属性          | 类型   | 说明                                                         |
| ------------- | ------ | ------------------------------------------------------------ |
| symbol        | str    | 标的代码                                                     |
| marketName    | str    | 品种名称。CS-股票；CF-期货；IDX-指数                         |
| exchange      | str    | 交易所及自定义类型后缀。SZ-深交所；SH上交所；DCE-大商所；SHFE-上期所；CZCE-郑商所；CFFEX-中金所；PLATE-板块；SET-证券集合 |
| currency      | str    | 币种                                                         |
| lotSize       | double | 一手标的的数目                                               |
| name          | str    | 标的名称                                                     |
| tplus         | int    | T+N交易。例如tplus为1时为t+1交易                             |
| marginRate    | double | 保证金比例                                                   |
| shortSellable | bool   | 是否可卖空                                                   |
| valuePerUnit  | double | 合约乘数                                                     |
| priceTick     | double | 标的价格浮动最小单位值                                       |
| exchSymbol    | str    | 交易所的原始symbol                                           |
| isStandard    | bool   | 是否是交易所的标准合约，比如主力合约 IFZ0.CF 就不是交易所的标准合约 |
| tradeMarket   | str    | 该标的对应的交易市场，目前仅有 CS CF                         |
| listDate      | int    | 上市日，无则为0                                              |
| lastTradeDate | int    | 最后交易日(退市日)，无则为0                                  |

### Tick-Tick行情信息

| 属性          | 类型             | 说明                                                         |
| ------------- | ---------------- | ------------------------------------------------------------ |
| symbol        | str              | 标的代码                                                     |
| bid           | double           | 买一价                                                       |
| ask           | double           | 卖一价                                                       |
| bidVol        | double           | 买一量（单位为实际证券股数）                                 |
| askVol        | double           | 卖一量                                                       |
| last          | double           | 最新成交价                                                   |
| lastVolume    | double           | 最新成交量                                                   |
| lastTurnover  | double           | 最新成交额                                                   |
| totalVolume   | double           | 交易日总成交量                                               |
| totalTurnover | double           | 交易日总成交额                                               |
| high          | double           | 交易日最高价                                                 |
| low           | double           | 交易日最低价                                                 |
| open          | double           | 交易日开盘价                                                 |
| close         | double           | 交易日收盘价                                                 |
| tradeDate     | long             | 交易日-年月日，格式为YYYYMMDD                                |
| timeExch      | long             | 交易所的实时时间戳-年月日时分秒 (ms)                         |
| timeStr       | str              | 交易时间，格式为YYYYMMDD-hhmmss-xxx                          |
| bids          | vector<qtyprice> | 五档买价，包括数量和价格，qtyprice{"quantity":qty,"price":price} |
| asks          | vector<qtyprice> | 五档卖价，包括数量和价格                                     |
| ceil          | double           | 涨停价                                                       |
| floor         | double           | 跌停价                                                       |
| position      | double           | 持仓量(期货)                                                 |
| preClose      | double           | 昨日收盘价                                                   |
| preSettle     | double           | 昨日结算价                                                   |

### Bar-Bar数据信息
| 属性          | 类型   | 说明                                |
| ------------- | ------ | ----------------------------------- |
| symbol        | str    | 标的代码                            |
| tradeDate     | long   | 交易日-年月日，格式为YYYYMMDD       |
| timeStop      | long   | bar对应的截止时间戳 (ms)            |
| timeStr       | str    | 交易时间，格式为YYYYMMDD-hhmmss-xxx |
| timeSpan      | long   | K线的时间频率                       |
| high          | double | 交易日最高价                        |
| low           | double | 交易日最低价                        |
| open          | double | 交易日开盘价                        |
| close         | double | 交易日收盘价                        |
| volume        | double | KBar成交额                          |
| turnover      | double | KBar成交量                          |
| totalVolume   | double | 交易日总成交量                      |
| totalTurnover | double | 交易日总成交额                      |
| preClose      | double | 昨日收盘价                          |
| settle        | double | 结算价                              |
| preSettle     | double | 昨日结算价                          |
| position      | double | 持仓量                              |
| isSuspended   | bool   | 是否停牌                            |

### SymbolPosition-策略持仓信息
某一个策略中某只标的的仓位信息。

| 属性               | 类型         | 说明                                           |
| ------------------ | ------------ | ---------------------------------------------- |
| symbol             | str          | 标的代码                                       |
| positionSide       | PositionSide | 持仓方向                                       |
| posQty             | double       | 持仓数量，不包括挂单的仓位                     |
| posDailyQty        | double       | 当日开仓的持仓数量，不包括挂单的仓位           |
| posPrice           | double       | 持仓均价                                       |
| posHigh            | double       | 持仓以来最高价                                 |
| posLow             | double       | 持仓以来最低价                                 |
| posTime            | long         | 开仓时间（时间戳）                             |
| posUrPnL           | double       | 仓位总浮动盈亏                                 |
| availableQty       | double       | 今日可平仓数量                                 |
| posDailyOverallPnL | double       | 当日盈亏=当日浮动盈亏+当日平仓盈亏，结算时清零 |
| posTradeDate       | int          | 开仓交易日，格式为YYYYMMDD                     |
| posMargin          | double       | 仓位占用保证金                                 |
| posMarketValue     | double       | 仓位市值                                       |

### StrategyPnL-策略盈亏信息
策略的整体盈亏信息。

| 属性            | 类型   | 说明                                                 |
| --------------- | ------ | ---------------------------------------------------- |
| id              | double | 策略ID                                               |
| urPnL           | double | 策略的总浮动盈亏                                     |
| overallPnL      | double | 策略的整体总盈亏，包含平仓盈亏                       |
| dailyPnL        | double | 策略的当日盈亏=当日浮动盈亏+当日平仓盈亏，结算时清零 |
| totalCommission | double | 策略总手续费                                         |

### OverallPosition-账户标的持仓信息
由于多个策略可以共享账户资金，因此可能会存在多个策略中都持有某个标的的情况，账户中会将多个策略中标的信息进行汇总。

| 属性              | 类型   | 说明             |
| ----------------- | ------ | ---------------- |
| accountId         | str    | 账户ID           |
| symbol            | str    | 标的代码         |
| buyPrice          | double | 加权平均买价     |
| sellPrice         | double | 加权平均卖价     |
| buyQty            | double | 买数量           |
| sellQty           | double | 卖数量           |
| longQty           | double | 多仓持仓数量     |
| shortQty          | double | 空仓持仓数量     |
| longPrice         | double | 多仓持仓均价     |
| shortPrice        | double | 空仓持仓均价     |
| longLastUrPnL     | double | 多仓浮动盈亏     |
| shortLastUrPnL    | double | 空仓浮动盈亏     |
| lastUrPnL         | double | 总持仓总浮动盈亏 |
| longAvailableQty  | double | 多仓可平仓数量   |
| shortAvailableQty | double | 空仓可平仓数量   |
| longMargin        | double | 多仓占用保证金   |
| shortMargin       | double | 空仓占用保证金   |
| longMarketValue   | double | 多仓持仓市值     |
| shortMarketValue  | double | 空仓持仓市值     |

### Account-账户信息

| 属性            | 类型   | 说明                                                         |
| --------------- | ------ | ------------------------------------------------------------ |
| id              | str    | 账户ID                                                       |
| market          | str    | 市场                                                         |
| currency        | str    | 账户币种                                                     |
| dailyPnL        | double | 当日平仓盈亏，结算时清零                                     |
| urLastPnL       | double | 浮动盈亏                                                     |
| allTimePnL      | double | 平仓盈亏                                                     |
| cashDeposited   | double | 累积出入金                                                   |
| cashAvailable   | double | 可用现金 = 现金 + 浮盈亏 - 市值 - 手续费（A股现金分红也在可用资金里 ） |
| unitValue       | double | 账户基金净值                                                 |
| bonus           | double | 现金分红                                                     |
| margin          | double | 持仓占用资金                                                 |
| marketValue     | double | 持仓市值                                                     |
| totAssets       | double | 总资产                                                       |
| totalCommission | double | 总手续费                                                     |

### Order-订单委托信息

| 属性            | 类型            | 说明                       |
| --------------- | --------------- | -------------------------- |
| symbol          | string          | 标的代码                   |
| side            | EOrderSide      | 买卖方向                   |
| positionSide    | EPositionSide   | 订单持仓方向               |
| positionEffect  | EPositionEffect | 订单开平仓类型             |
| qty             | double          | 委托量                     |
| price           | double          | 委托价                     |
| userId          | string          | 用户id                     |
| accountId       | string          | 账户id                     |
| created         | long            | 委托创建日期               |
| id              | string          | 订单id                     |
| tradeDate       | int             | 交易日                     |
| strategyId      | string          | 所属策略id                 |
| portfolioId     | string          | 组合id                     |
| commission      | commission      | 手续费                     |
| tradeAccount    | string          | 交易账号                   |
| externalOrderId | string          | SDK生成的ID-SDK            |
| orderType       | EOrderType      | 订单委托类型，市价或限价单 |
| ordStatus       | EOrdStatus      | 订单状态                   |
| cumQty          | double          | 已成交数量                 |
| avgPx           | double          | 已成交均价                 |
| tif             | TimeInForce     | 订单时间状态               |
| modified        | long            | 委托最后修改日期           |
| execType        | EExecType       | 订单执行状态               |

## 枚举类型
### ETimeSpan-频率类型
```
enum ETimeSpan
{
    TICK ,    //tick类型
    MIN_1 ,  //1分钟K线
    MIN_5 ,  //5分钟K线
    MIN_15 , //15分钟K线
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
### EPositionEffect  -订单开平仓类型
```
   enum PositionEffect
    {
        OPEN = 1,   //开盘价
        CLOSE = 2,   //收盘价   
        CLOSE_TODAY = 3,   //当日收盘价
        CLOSE_YESTERDAY = 4,   //昨日收盘价
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

### EOrderType -订单类型
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

### onInitialize-初始化
** 函数原型：** 
```
onInitialize(api)
```
**说明：** 在策略运行前会调用一次，用来进行初始化。行情数据缓存需要在这个事件中实现。

### onBeforeMarketOpen-盘前运行
** 函数原型：** 
```
onBeforeMarketOpen(api,tradeDate)
```
**说明：** tradeDate表示当前日期，开盘前调用一次，选股的逻辑可以在这个函数中实现，可以根据策略的复杂度在创建实例时设置盘前策略运行时间。

### onTick-tick产生
** 函数原型：** 
```
onTick(api,tick)
```
**说明：** [tick](data7.md#tick-tick数据)产生时会调用，目前仅期货市场支持tick。

### onFirstTick-第一根tick触发
** 函数原型：** 
```
onFirstTick(api,tick)
```
**说明：** 第一根[tick](apiold9.html#tick-tick行情信息)产生时会调用，目前仅期货市场支持tick。可用于撮合周期为非tick级别来获取当日涨跌停价，以此做止盈止损，回测不支持此回调。

### onBar-Bar产生
** 函数原型：** 
```
onBar(api,bar)
```
**说明：** 创建实例时设置撮合周期，[bar](apiold9.html#bar-bar数据信息)产生时会调用。为了提高效率，只有在关注列表（即[setFocusSymbols](apiold9.html#setfocussymbols-设置关注的标的池)设置的标的列表）中和有持仓的标的才会响应行情回调。

### onHandleData-数据到齐后触发
** 函数原型：** 
```
def onHandleData(api,timeExch)
```
**说明：** 针对某一周期的bar数据，搜集齐后再统一响应回调，timeExch为当前周期的时间戳。对于回测，数据可以都到齐；对于模拟盘和实盘，不一定能够都到齐，需要设置超时时间。默认只回调onBar，如需回调onHandleData则需在onInitialize中配置setGroupMode函数，详见[setGroupMode](apiold9.html#setgroupmode-设置到齐回调模式)。

**示例：**
```
from etasdk import *
from etasdk.utils.time_util import TimeUtil

#初始化
def onInitialize(api):
    api.setRequireBars(ETimeSpan.MIN_1, 10);#准备1分钟K线
    api.setGroupMode(5000,True)#设置收齐数据超时为5s，True表示只回调onHandleData， false表示既回调onHandleData也回调onBar

symbolPool=[]

#盘前运行，必须实现，用来设置当日关注的标的
def onBeforeMarketOpen(api,tradeDate):
    global symbolPool
    print "onBeforeMarketOpen----",tradeDate
    symbolPool=api.getSymbolPool()
    api.setFocusSymbols(symbolPool)

#数据收齐回调
def onHandleData(api,timeExch):
    print ("onHandleData---time_stamp   %s" % TimeUtil.msToTimeString(api.timeNow()))
    for symbol in symbolPool:
        print (api.getBarsHistory(symbol, ETimeSpan.MIN_1, 1, df=True))

```

### onTerminate-运行终止
** 函数原型：** 
```
 onTerminate (api, exitInfo)
```
**说明：** 运行终止时会调用一次,exitInfo返回值为终止消息。

### onTimer-定时器
** 函数原型：** 
```
onTimer (api) 
```
**说明：** 每隔5000毫秒会响应一次。

### onOrderUpdate-订单状态更新回调
** 函数原型：** 
```
onOrderUpdate(api,order) 
```
**说明：**  订单状态更新时会响应一次，order为订单详情。

## 设置函数 

### setRequireData-设置策略所需缓存数据（包含K线、股票池和日频、季频、财务数据字段）
在onInitialize函数中调用，设置缓存数据（包含K线、股票池和日频、季频、财务数据字段），是函数setRequireFields、setRequireBars和setSymbolPool的整合。

**函数原型：**
```
setRequireData(instsets=None, symbols=None, fields=None, bars=None)
```

**参数：**

| 参数     | 类型                        | 是否必填 | 说明                                                         |
| -------- | --------------------------- | -------- | ------------------------------------------------------------ |
| instsets | list                        | 否       | 设置标的集合，如000300.IDX、CS.SET等                         |
| symbols  | list                        | 否       | 设置标的列表。                                               |
| fields   | str or list                 | 否       | 设置数据属性，可以是多个字段                                 |
| bars     | list[tuple(ETimeSpan, int)] | 否       | 设置缓存K线条数，可以设置多个频率。频率类型，详见  [ETimeSpan](apiold9.md#etimespan-频率类型) |

**举例：**
```
#设置股票池、相关标的因子数据和K线。
api.setRequireData(instsets=["000016.IDX","cu.PRD"],symbols=["IFZ0.CF","000001.CS"],fields=["CIRC_CAP", "TURNOVER_RATE", "PE"],bars=[(ETimeSpan.DAY_1,2),(ETimeSpan.MIN_1,2)]) 
#设置股票池、相关标的因子数据和K线。
api.setRequireData(instsets=["000016.IDX","cu.PRD"],fields=["CIRC_CAP", "TURNOVER_RATE", "PE"],bars=[(ETimeSpan.DAY_1,2),(ETimeSpan.MIN_1,2)])
#设置股票池、相关标的因子数据和K线。
api.setRequireData(symbols=["IFZ0.CF","000001.CS"],fields=["CIRC_CAP", "TURNOVER_RATE", "PE"],bars=[(ETimeSpan.DAY_1,2),(ETimeSpan.MIN_1,2)])
#设置股票池和K线。
api.setRequireData(instsets=["000016.IDX","cu.PRD"],symbols=["IFZ0.CF","000001.CS"],bars=[(ETimeSpan.DAY_1,2),(ETimeSpan.MIN_1,2)])
#设置股票池和相关标的因子数据
api.setRequireData(instsets=["000016.IDX","cu.PRD"],symbols=["IFZ0.CF","000001.CS"],fields=["CIRC_CAP", "TURNOVER_RATE", "PE"])

```
### setFocusSymbols-设置关注的标的池
只能在[onBeforeMarketOpen](apiold9.html#onbeforemarketopen-盘前运行)函数中调用，必须实现。只有在此函数中进行设置的标的和当前持仓的标的才会在[onBar](apiold9.html#onbar-bar产生)、[onHandleData](apiold9.html#onhandledata-数据到齐后触发)或者[onTick](apiold9.html#ontick-tick产生)函数中进行行情响应。若没有在setFocusSymbols中set的标的并且不在持仓的不会在[onBar](41.html#apiold9.html#onbar-bar产生)、[onHandleData](apiold9.html#onhandledata-数据到齐后触发)和[onTick](apiold9.html#ontick-tick产生)中响应，并且调用[getBarsHistory](apiold9.html#getbarshistory-获取历史k线数据)也拿不到当天的K线。
<font color="#660000">**注：setFocusSymbols 如果多次使用，则只会记录最后一条的设置；前面的设置将会被覆盖**</font><br />

**函数原型：**

```
setFocusSymbols(symbols)
```

**参数：**

| 参数    | 类型        | 是否必填 | 说明           |
| ------- | ----------- | -------- | -------------- |
| symbols | str or list | 是       | 标的或标的列表 |


### setSymbolPool-设置股票池
在onInitialize函数中调用，只有在此函数中进行设置相关股票，才能获取到对该股票设置缓存的相关K线和其他因子数据以及财务数据。股票池可以设置单个标的，也可以设置[指数成分股](data7.html#股票指数代码)、[行业板块](data7.html#股票行业板块代码)、[特殊集合](data7.html#特殊集合约定)。

建议不再使用, 直接使用setRequireData代替。

**函数原型：**
```
setSymbolPool(instsets = None,symbols = None)
```

**参数：**

| 参数     | 类型 | 说明                                         |
| -------- | ---- | -------------------------------------------- |
| instsets | list | 设置标的集合，如000300.IDX、CS.SET等，可缺省 |
| symbol   | list | 设置标的列表，可缺省                         |

**举例：**
```
api.setSymbolPool(instsets=["CF.SET"])  #设置股票池为期货全部，包括主力合约和次主力合约。
api.setSymbolPool(instsets=["000300.IDX，000016.IDX"])  #设置股票池沪深300和上证50的并集。
api.setSymbolPool(instsets=["cu.PRD"],symbols=["cuZ0.CF","cuZ1.CF"])  #设置股票池为期货是cu的全部标准合约（不含cuZ0.CF和cuZ1.CF）和cuZ0.CF和cuZ1.CF。
api.setSymbolPool(symbols=["IFZ0.CF"])  #设置股票池为IF的主力合约
api.setSymbolPool(instsets=["IFZ0.CF"])  #设置股票池为IF的主力合约对应的标准合约
api.setSymbolPool(symbols=["000001.CS","IF1809.CF"])  #设置股票池为000001.CS和IF1809.CF。

```

### setRequireBars-设置行情数据缓存条数
在onInitialize函数中调用，设置Bar行情数据的数据缓存条数，即[getBarsHistory](apiold9.html#getbarshistory-获取历史k线数据)函数中可以调用的历史条数，在初始化事件中进行调用。例如将日K线设置成5，则可以获取到前5个交易日的数据，随着回测时间移动之前的数据会被清掉。若需要取用多个频率的数据则需要分别设置，不设置该函数不会缓存。

建议不再使用, 直接使用setRequireData代替。

**函数原型：**
```
setRequireBars(timspan,count)
```

**参数：**

| 参数    | 类型      | 说明                                                         |
| ------- | --------- | ------------------------------------------------------------ |
| timspan | ETimeSpan | Bar的频率类型，详见 [ETimeSpan](apiold9.html#etimespan-频率类型) 。 |
| count   | int       | 设置行情数据的数据需要缓存的条数。                           |

**举例：**
```
def onInitialize(api)
    api.setRequireBars(ETimeSpan.DAY_1,20)  #将日K的数据需要的历史数据设置成20根。
```

### setRequireFields-设置缓存日频、季频、财务等非行情类数据
在onInitialize函数中调用，设置缓存日频、季频、财务数据字段，即[getFieldsOneDay](apiold9.html#getfieldsoneday-获取多只股票单个交易日的数据)、[getFieldsRangeDays](apiold9.html#getfieldsrangedays-获取单只股票多个交易日的数据)、[getFieldsCountDays](apiold9.html#getfieldscountdays-获取单只股票基于某个交易日的前n条数据)和[getStockFinFields](apiold9.html#getstockfinfields-获取财务数据) 函数中可以调用的字段。例如设置"TOT_SHARE", "SP_RESER"，则调用前面的函数中就可以取到这两个字段的值，否则取不到值。不设置该函数就不会缓存。

建议不再使用, 直接使用setRequireData代替。

**函数原型：**
```
setRequireFields(fields)
```

**参数：**

| 参数   | 类型        | 说明                           |
| ------ | ----------- | ------------------------------ |
| fields | str or list | 设置数据属性，可以是多个字段。 |

**举例：**
```
def onInitialize(api)
     api.setRequireFields(fields=["TOT_SHARE", "SP_RESER", "PE"]) #设置需要获取的字段。
```

### setGroupMode-设置到齐回调模式
在onInitialize函数中调用，若不调用本函数，则只响应[onBar](apiold9.html#onbar-bar产生)。

**函数原型：**

```
setGroupMode(timeOutMs,onlyGroup)
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                                                         |
| --------- | ---- | -------- | ------------------------------------------------------------ |
| timeOutMs | int  | 是       | 超时时间，单位为毫秒，最好设置不少于5000ms                   |
| onlyGroup | bool | 是       | True-只回调[onHandleData](apiold9.html#onhandledata-数据到齐后触发)  ，False- 既回调[onHandleData](apiold9.html#onhandledata-数据到齐后触发)也回调[onBar](apiold9.html#onbar-bar产生) |

**举例：**

```
#设置只响应onHandleData，超时时间为5000ms
api.setGroupMode(timeOutMs=5000, onlyGroup=True)
#设置既响应onHandleData，也响应onBar，超时时间为10000ms
api.setGroupMode(timeOutMs=10000, onlyGroup=False)
```
### setTimerCycle-设置定时回调频率
在onInitialize函数中调用，设置[onTimer](apiold9.html#ontimer-定时器)定时回调函数的响应周期。在回测环境下无效。
**函数原型：**

```
setTimerCycle(cycle)
```
**参数：**

| 参数  | 类型 | 是否必填 | 说明                       |
| ----- | ---- | -------- | -------------------------- |
| cycle | int  | 是       | 响应时间间隔，单位为毫秒。 |

**举例：**

```
#设置定时回调频率为5000ms
api.setTimerCycle(cycle=5000)
```

## 交易函数
### targetPosition-下单
目标仓位数量下单

**函数原型：**

```
targetPosition (symbol,qty,price=None,positionSide=None,tif =ETimeInForce.NONE,remark=None,priceList=EPriceList.None)
```
**参数：**

| 参数         | 类型          | 是否必填 | 说明                                                         |
| ------------ | ------------- | -------- | ------------------------------------------------------------ |
| symbol       | str           | 是       | 策略中的标的代码                                             |
| qty          | int           | 是       | 交易量                                                       |
| price        | double        | 否       | 价格，默认不传参（表示以市价下单），若传参则表示以该委托价下限价单 |
| positionSide | EPositionSide | 否       | 多或者空，不填时默认为多仓，股票不能为short。详见 [EPositionSide](apiold9.html#epositionside-持仓方向) |
| tif          | ETimeInForce  | 否       | 下单属性说明，不填和填None时效果相同，详见[ETimeInForce-订单委托属性](apiold9.html#etimeinforce-订单委托属性) |
| remark       | str           | 否       | 下单原因说明，填写后可以在web端展示                          |
| priceList    | EtaPriceList  | 否       | 报价列表，用于下限价单。若与price同时传参，则以price为准，详见[EtaPriceList-报价列表](apiold9.html#etapricelist-报价列表) |

**返回值：** orderID。orderID作为对此条下单操作的标识。

**举例：**

股票下单

```
#以市价单买入100股000001
api.targetPosition (symbol="000001.CS",qty=100) 
#以11元限价单买入100股000001
api.targetPosition (symbol="000001.CS",qty=100,price=11)  
#以市价单卖出000001股票至持仓为0
api.targetPosition (symbol="000001.CS",qty=0,remark="卖出平仓") 
#以限价单卖一价买入000001股票至持仓为100股
api.targetPosition (symbol="000001.CS",qty=100,priceList=EPriceList.ASK1) 
```
期货下单
```
#IF1809市价单开多1手
api.targetPosition (symbol='IF1809.CF',qty=1,positionSide=EPositionSide.LONG)  
#IF1809市价单开空1手
api.targetPosition (symbol='IF1809.CF',qty=1,positionSide=EPositionSide.SHORT)  
```


### cancelOrder-撤单
取消订单

**函数原型**
```
cancelOrder(symbol,  positionSide=EPositionSide.LONG, remark="")
```
**参数：**

| 参数         | 类型          | 是否必填 | 说明                                                         |
| ------------ | ------------- | -------- | ------------------------------------------------------------ |
| symbol       | str           | 是       | 策略中的标的代码                                             |
| positionSide | EPositionSide | 否       | 多或者空，不填时默认为多仓，股票不能为short。详见 [EPositionSide](apiold9.html#epositionside-持仓方向) 。 |
| remark       | str           | 否       | 撤单原因说明，填写后可以在web端展示                          |

**举例：**

```
#撤销股票000001.CS的挂单
api.cancelOrder(symbol="000001.CS",remark="撤销000001.CS的挂单")
#撤销期货rb1901.CF的空头挂单
api.cancelOrder(symbol="rb1901.CF", positionSide=EPositionSide.SHORT)
```

## 标的查询函数
### getSymbolPosition-获取单个标的仓位
获取标的仓位信息

**函数原型：**

```
 getSymbolPosition (symbol,positionSide=None)
```
**参数：**

| 参数          | 类型         | 是否必填 | 说明                                                         |
| ------------- | ------------ | -------- | ------------------------------------------------------------ |
| symbol        | str          | 是       | 标的代码                                                     |
| EPositionSide | positionSide | 否       | 多或者空，不填时默认为多仓，股票不能为short。详见[EPositionSide](apiold9.html#epositionside-持仓方向) 。 |

**返回值：**[SymbolPosition](apiold9.html#symbolposition-策略持仓信息)对象。

**举例：**
```
#获取股票000001.CS仓位信息
symbolposition = api.getSymbolPosition (symbol="000001.CS")  
#获取期货IF1809.CF多仓的仓位信息
symbolposition = api.getSymbolPosition (symbol="IF1809.CF"，positionSide=EPositionSide.LONG)  
#获取期货IF1809.CF空仓的仓位信息
symbolposition = api.getSymbolPosition (symbol="IF1809.CF"，positionSide=EPositionSide.SHORT) 
```
### getSymbolPositions-获取多个/所有持仓标的仓位信息
获取所有标的仓位信息

**函数原型：**

```
 getSymbolPositions ()
```
**参数：**无

**返回值：**List结构，元素为[SymbolPosition](apiold9.html#symbolposition-策略持仓信息)对象。

**举例：**

```
#获取所有标的的仓位信息
symbolpositions = api.getSymbolPositions ()  
```

## 账户查询函数
### getAccount-获取标的所属账号信息
获取标的所属账号信息，由于账号获取是异步的，所以下完单立即获取账户数据会有延迟。

**函数原型：**

```
getAccount (symbol=None, market=None)
```
**参数：**

| 参数   | 类型 | 是否必填 | 说明                                                         |
| ------ | ---- | -------- | ------------------------------------------------------------ |
| symbol | str  | 否       | 标的代码，可缺省，但不能同时和市场一起缺省，若都不缺省以market为准 |
| market | str  | 否       | 市场，主要有股票MARKET_CHINASTOCK和期货MARKET_CHINAFUTURE    |

**返回值：**标的所属证券的市场账号信息，[Account](apiold9.html#account-账户信息)结构。

**举例：**
```
#用标的获取A股账号的信息
account = api.getAccount (symbol="000001.CS")  
#用市场获取A股账号的信息
account = api.getAccount ( market=MARKET_CHINASTOCK)  
#同时填入标的和市场时，返回通过市场获取的账号信息，如下返回期货账号信息
account = api.getAccount (symbol="000001.CS"， market=MARKET_CHINAFUTURE)  

```

### getOverallPosition-获取标的汇总仓位
由于多个策略可以共享账户资金，因此可能会存在多个策略中都持有某个标的的情况，账户中会将多个策略中标的信息进行汇总。本函数用来获取某个标的在账户中汇总后的仓位信息。

**函数原型：**

```
getOverallPosition (symbol)
```
**参数：**

| 参数   | 类型 | 是否必填 | 说明     |
| ------ | ---- | -------- | -------- |
| symbol | str  | 是       | 标的代码 |

**返回值：**该标的在其所属账户中的总仓位信息，[OverallPosition](apiold9.html#overallposition-账户标的持仓信息)对象结构。

**举例：**

```
#获取账户下000001.CS的汇总仓位信息
overallposition = api.getOverallPosition (symbol="000001.CS")  #获取000001.CS仓位信息
availableQty = overallposition. longAvailableQty        #获取多仓可平仓数量 

```
## 策略查询函数

### getStrategyPnL-获取策略盈亏信息
获取策略盈亏信息。

**函数原型：**

```
getStrategyPnL()
```
**参数值：**无

**返回值：**策略盈亏信息，[StrategyPnL](apiold9.html#strategypnl-策略盈亏信息)对象结构

**举例：**
```
getStrategyPnL= api.getStrategyPnL() #获取策略盈亏信息
overallPnL   = getStrategyPnL.overallPnL   #获取策略总盈亏字段数据
```

## 策略数据查询函数

策略相关数据的获取在api中进行调用。
### getValueAs*-获取动态运行参数
在web页面上创建策略时可以预先设置参数，再通过api进行调用，获取策略中某策略的参数信息。页面上对参数的改动可以立即反映在策略中。

**函数原型：**
```
getValueAsDouble(field)
getValueAsString(field)
getValueAsLong(field)

```
**参数：**

| 参数  | 类型 | 是否必填 | 说明     |
| ----- | ---- | -------- | -------- |
| field | str  | 是       | 参数名称 |

**返回值：** 参数值。getValueAsDouble返回float类型，getValueAsString返回string类型，getValueAsLong返回int类型。

**举例：**
```
high=api.getValueAsDouble(field="high") #在页面配置了名称为high的参数，获取当前策略high参数
```
### getSymbolPool-获取股票池代码
与setRequireData 和 setSymbolPool对应，获取之前设置的股票池代码

**函数原型：**
```
getSymbolPool()
```
**参数：**无

**返回值：** 获取之前设置的股票池代码

**举例：**
```
pool=api.getSymbolPool() #获取股票池代码
```

### getFocusAndPositionSymbols-获取持仓和当日关注的标的代码
只有当日关注和有持仓的标的才会响应行情回调函数，可以通过本函数来获取持仓和关注的标的代码。

**函数原型：**
```
getFocusAndPositionSymbols()
```
**参数：**无

**返回：** 返回股票代码的list

### getPositionSymbols-获取持仓标的代码
仅获取当前有持仓的标的代码。

**函数原型：**
```
getPositionSymbols()
```
**参数：**无

**返回：** 返回持仓标的代码的List

### getAppointedSymbols-获取指定合约集合
获取指定合约集合，如主力合约集合，次主力合约集合。

**函数原型：**
```
getAppointedSymbols(eType, tradeDate=None)
```
**参数：**

| 参数      | 类型  | 是否必填 | 说明                                            |
| --------- | ----- | -------- | ----------------------------------------------- |
| eType     | eType | 是       | 指定合约的类型，有Z0、Z1，Y1-Y12                |
| tradeDate | int   | 否       | 交易日 ，形式为YYYYMMDD，不填默认返回当天的数据 |

**返回：** 返回指定合约集合

**举例：**

获取20180710当天的所有主力合约
```
api.getAppointedSymbols(eType="Z0", tradeDate=20180710)
```
返回结果
```
['APZ0.CF', 'CFZ0.CF', 'CYZ0.CF', 'FGZ0.CF', 'ICZ0.CF', 'IFZ0.CF', 'IHZ0.CF', 'JRZ0.CF', 'LRZ0.CF', 'MAZ0.CF', 'OIZ0.CF', 'PMZ0.CF', 'RIZ0.CF', 'RMZ0.CF', 'RSZ0.CF', 'SFZ0.CF', 'SMZ0.CF', 'SRZ0.CF', 'TAZ0.CF', 'TFZ0.CF', 'TZ0.CF', 'WHZ0.CF', 'ZCZ0.CF', 'aZ0.CF', 'agZ0.CF', 'alZ0.CF', 'auZ0.CF', 'bZ0.CF', 'bbZ0.CF', 'buZ0.CF', 'cZ0.CF', 'csZ0.CF', 'cuZ0.CF', 'fbZ0.CF', 'fuZ0.CF', 'hcZ0.CF', 'iZ0.CF', 'jZ0.CF', 'jdZ0.CF', 'jmZ0.CF', 'lZ0.CF', 'mZ0.CF', 'niZ0.CF', 'pZ0.CF', 'pbZ0.CF', 'ppZ0.CF', 'rbZ0.CF', 'ruZ0.CF', 'scZ0.CF', 'snZ0.CF', 'vZ0.CF', 'wrZ0.CF', 'yZ0.CF', 'znZ0.CF']
```
获取当天（20181127）月份为1的所有合约，若某标的同时存在1901和2001则返回较近的合约，即1901
```
api.getAppointedSymbols(eType="Y1")
```
返回结果
```
['OI1901.CF', 'ZC1901.CF', 'cu1901.CF', 'y1901.CF', 'WH1901.CF', 'bb1901.CF', 'FG1901.CF', 'rb1901.CF', 'sc1901.CF', 'sn1901.CF', 'RI1901.CF', 'm1901.CF', 'CY1901.CF', 'SR1901.CF', 'al1901.CF', 'a1901.CF', 'c1901.CF', 'JR1901.CF', 'fb1901.CF', 'i1901.CF', 'l1901.CF', 'RM1901.CF', 'IH1901.CF', 'ag1901.CF', 'IC1901.CF', 'IF1901.CF', 'au1901.CF', 'ru1901.CF', 'zn1901.CF', 'pp1901.CF', 'p1901.CF', 'j1901.CF', 'jd1901.CF', 'TA1901.CF', 'MA1901.CF', 'ni1901.CF', 'bu1901.CF', 'pb1901.CF', 'hc1901.CF', 'wr1901.CF', 'jm1901.CF', 'cs1901.CF', 'v1901.CF', 'b1901.CF', 'SF1901.CF', 'LR1901.CF', 'SM1901.CF', 'PM1901.CF', 'CF1901.CF', 'AP1901.CF', 'fu1901.CF']
```
### getCurrTradeDate-获取当前交易日期
获取当前的交易日期。回测情况下是回测历史当天，交易情况下是当前交易日。

**函数原型：**
```
getCurrTradeDate()
```
**参数：**无

**返回值：**返回历史当天的日期，int类型

### getBarsHistory-获取历史K线数据
获取K线数据。模拟盘和实盘在[onHandleData](apiold9.html#onhandledata-数据到齐后触发)和[onBar](apiold9.html#onbar-bar产生)中获取当天的日K不是全日K线，是开盘时间到所设置的收盘时间的日K。

**函数原型：**

```
getBarsHistory (symbol, timeSpan, count = None，priceMode = EPriceMode.REAL, skipSuspended = 1 ,fields=None, df=True)
```
**参数：**

| 参数          | 类型        | 是否必填 | 说明                                                         |
| ------------- | ----------- | -------- | ------------------------------------------------------------ |
| symbol        | str         | 是       | 标的代码                                                     |
| timeSpan      | ETimeSpan   | 是       | bar的频率，详见 [ETimeSpan](apiold9.html#etimespan-频率类型)。 |
| count         | int         | 是       | 获取历史数据条数，超过缓存类型的返回空                       |
| priceMode     | EPriceMode  | 否       | 复权模式，不填默认返回不复权，详见[EPriceMode](apiold9.html#epricemode-复权模式) 。 |
| skipSuspended | int         | 否       | 为了保证取不同标的数据时时间对齐，可以设置是否跳过停牌， 0-不跳过，1-跳过，不填写默认为跳过。若不跳过，填充方式为高开低收均用前收盘价填充，成交量=成交额=0。 |
| fields        | str or list | 否       | 返回的字段，不填默认返回全部。                               |
| df            | bool        | 否       | 返回值是否采用dataFrame格式，不填默认为dataFrame格式         |

**返回值：**行情数据 ,df为false时返回[Bar](apiold9.html#bar-bar数据信息)结构List。df为true时返回dataFrame格式。

**举例：**

获取不过滤停牌的前复权5分钟K线2支
```
#首先在def onInitialize(api)中设置缓存数据：
api.setRequireData(instsets=[],symbols=["000001.CS"],fields=[], bars=[(ETimeSpan.MIN_5, 5)])
#然后在对应事件回调中获取历史K线
bars = api.getBarsHistory (symbol ="000001.CS",timeSpan=ETimeSpan.MIN_5,count=2,priceMode=EPriceMode.FORMER,skipSuspended=0,df=False)  
```
返回结果：
```
[{ "symbol": "000001.CS", "timeSpan": 300, "timeStop": 1543215300000, "timeStr": "20181126-145500-000", "tradeDate": 20181126, "close": 10.32, "high": 10.33, "isSuspended": False, "low": 10.31, "open": 10.32, "position": 0, "preClose": 10.32, "preSettle": 0, "settle": 0, "totalTurnover": 5.24417e+08, "totalVolume": 5.05396e+07, "turnover": 1.37453e+07, "volume": 1.33236e+06 }, { "symbol": "000001.CS", "timeSpan": 300, "timeStop": 1543215600000, "timeStr": "20181126-150000-000", "tradeDate": 20181126, "close": 10.34, "high": 10.34, "isSuspended": False, "low": 10.32, "open": 10.32, "position": 0, "preClose": 10.32, "preSettle": 0, "settle": 0, "totalTurnover": 5.35963e+08, "totalVolume": 5.16569e+07, "turnover": 1.15462e+07, "volume": 1.1173e+06 }]
```
打印某个字段值：
```
print bars[0].totalTurnover  #打印交易日总成交额
```
返回结果：
```
524417138.0
```
获取不过滤停牌的不复权5分钟K线2组，格式为dataFrame
```
bars = api.getBarsHistory (symbol ="000001.CS",timeSpan=ETimeSpan.MIN_5,count=2) 
```
返回结果：
```
               timeStr       timeStop     symbol  tradeDate  close   high    low   open  preClose  settle  preSettle     volume    turnover  totalVolume  totalTurnover  position  isSuspended
0  20181126-145500-000  1543215300000  000001.CS   20181126  10.32  10.33  10.31  10.32     10.32     0.0        0.0  1332361.0  13745256.0   50539573.0    524417138.0       0.0        False
1  20181126-150000-000  1543215600000  000001.CS   20181126  10.34  10.34  10.32  10.32     10.32     0.0        0.0  1117298.0  11546172.0   51656871.0    535963310.0       0.0        False
```
获取不过滤停牌的不复权5分钟K线2支，格式为dataFrame，返回列为tradeDate，和close 
```
bars = api.getBarsHistory (symbol ="000001.CS",timeSpan=ETimeSpan.MIN_5,count=2,fields=['tradeDate', 'close'])  
```
返回结果：
```
               timeStr  tradeDate     symbol  close
0  20181126-145500-000   20181126  000001.CS  10.32
1  20181126-150000-000   20181126  000001.CS  10.34
```

## 基础数据查询函数
若想独立使用以下数据，请前往<a href="https://quant.upchina.com/navdata.html" target="_blank">数据</a>栏查看相关使用详情

### getRefData-获取标的基本信息
获取标的基本信息。

**函数原型：**
```
getRefData(symbol)
```
**参数：**

| 参数   | 类型 | 是否必填 | 说明     |
| ------ | ---- | -------- | -------- |
| symbol | str  | 是       | 标的代码 |

**返回值：**证券基本信息，[RefData](apiold9.html#refdata-股票基础信息)。

**举例：**
```
#获取000001.CS证券基本信息
refData = api.getRefData (symbol="000001.CS") 

```
返回结果:
```
{ "symbol": "000001.CS", "marketName": "CS", "exchange": "SZ", "currency": "CNY", "lotSize": 100, "name": "平安银行", "tplus": 1, "marginRate": 1, "shortSellable": False, 
"valuePerUnit": 1, "priceTick": 0.01, "exchSymbol": 000001, "isStandard": True, "tradeMarket": CS", "listDate": 19910403, "lastTradeDate": 0 }

```
### getConstituentSymbols-获取成分股数据

**函数原型：**
```
getConstituentSymbols(indexs,tradeDate)
```
获取股票指数或者行业板块的成分股数据。获取期货某品种当前可交易的合约数据。详见指数代码说明。

**参数：**

| 参数      | 类型        | 是否必填 | 说明                                                         |
| --------- | ----------- | -------- | ------------------------------------------------------------ |
| indexs    | str or list | 是       | 指数代码、板块代码、指数代码列表或者行业板块代码列表，为列表时表示获取的股票同时属于这些指数或者行业。 |
| tradeDate | int         | 否       | 交易日，形式为YYYYMMDD，支持获取历史成分股信息，不填默认返回当天的数据 |

**返回：** 返回股票代码的List

**举例：**

获取上证指数成分股
```
constituentSymbols = api.getConstituentSymbols(indexs="000001.IDX",tradeDate=20170403) 
```
返回结果：
```
['600023.CS', '600908.CS', '600909.CS', '600917.CS', '600919.CS', '600926.CS', '600936.CS', '600939.CS', '600958.CS', '600959.CS', '600977.CS', '600996.CS', '600998.CS', '600999.CS', '601000.CS', '601002.CS', '601003.CS', '601005.CS', '601007.CS', '601008.CS', '601009.CS', '601010.CS', '601011.CS', '601012.CS', '601015.CS', '601016.CS', '601018.CS', '601020.CS', '601021.CS', '601028.CS', '601038.CS', '601058.CS', '601069.CS', '601088.CS', '601098.CS', '601099.CS', '601100.CS', '601101.CS', '601106.CS', '601107.CS', '601113.CS', '601116.CS', '601117.CS', '601118.CS', '601126.CS', '601127.CS', '601128.CS', '601137.CS', '601139.CS', '601155.CS', '601158.CS', '601163.CS', '601166.CS', '601168.CS', '601169.CS', '601177.CS', '601179.CS', '601186.CS', '601188.CS', '601198.CS', '601199.CS', '601208.CS', '601211.CS', '601212.CS', '601216.CS', '601218.CS', '601222.CS', '601225.CS', '601226.CS', '601229.CS', '601231.CS', '601233.CS', '601238.CS', '601258.CS', '601288.CS', '601311.CS', '601318.CS', '601328.CS', '601336.CS', '601339.CS', '601360.CS', '601368.CS', '601369.CS', '601375.CS', '601377.CS', '601388.CS', '601390.CS', '601500.CS', '601515.CS', '601518.CS', '601519.CS', '601555.CS', '601558.CS', '601566.CS', '601567.CS', '601579.CS', '601595.CS', '601599.CS', '601600.CS', '601601.CS', '601608.CS', '601611.CS', '601616.CS', '601618.CS', '601628.CS', '601633.CS', '601636.CS', '601668.CS', '601669.CS', '601677.CS', '601678.CS', '601688.CS', '601689.CS', '601700.CS', '601717.CS', '601718.CS', '601727.CS', '601766.CS', '601777.CS', '601788.CS', '601789.CS', '601798.CS', '601799.CS', '601800.CS', '601801.CS', '601808.CS', '601811.CS', '601818.CS', '601857.CS', '601858.CS', '601866.CS', '601877.CS', '601880.CS', '601881.CS', '601882.CS', '601886.CS', '601888.CS', '601890.CS', '601898.CS', '601899.CS', '601900.CS', '601901.CS', '601908.CS', '601918.CS', '601919.CS', '601928.CS', '601929.CS', '601933.CS', '601939.CS', '601958.CS', '601965.CS', '601966.CS', '601968.CS', '601969.CS', '601985.CS', '601989.CS', '601992.CS', '601996.CS', '601997.CS', '601998.CS', '601999.CS', '603000.CS', '603001.CS', '603002.CS', '603003.CS', '603005.CS', '603006.CS', '603007.CS', '603008.CS', '603009.CS', '603010.CS', '603011.CS', '603012.CS', '603015.CS', '603016.CS', '603017.CS', '603018.CS', '603019.CS', '603020.CS', '603021.CS', '603022.CS', '603023.CS', '603025.CS', '603026.CS', '603027.CS', '603028.CS', '603029.CS', '603030.CS', '603031.CS', '603032.CS', '603033.CS', '603035.CS', '603036.CS', '603037.CS', '603038.CS', '603039.CS', '603040.CS', '603058.CS', '603060.CS', '603066.CS', '603067.CS', '603069.CS', '603077.CS', '603085.CS', '603088.CS', '603089.CS', '603090.CS', '603098.CS', '603099.CS', '603100.CS', '603101.CS', '603108.CS', '603111.CS', '603116.CS', '603117.CS', '603118.CS', '603123.CS', '603126.CS', '603128.CS', '603131.CS', '603138.CS', '603158.CS', '603159.CS', '603160.CS', '603165.CS', '603166.CS', '603167.CS', '603168.CS', '603169.CS', '603177.CS', '603179.CS', '603186.CS', '603188.CS', '603189.CS', '603198.CS', '603199.CS', '603203.CS', '603208.CS', '603218.CS', '603222.CS', '603223.CS', '603227.CS', '603228.CS', '603238.CS', '603239.CS', '603258.CS', '603266.CS', '603268.CS', '603288.CS', '603298.CS', '603299.CS', '603300.CS', '603306.CS', '603308.CS', '603309.CS', '603311.CS', '603313.CS', '603315.CS', '603318.CS', '603319.CS', '603322.CS', '603323.CS', '603328.CS', '603330.CS', '603333.CS', '603336.CS', '603337.CS', '603338.CS', '603339.CS', '603345.CS', '603355.CS', '603358.CS', '603360.CS', '603366.CS', '603368.CS', '603369.CS', '603377.CS', '603389.CS', '603393.CS', '603398.CS', '603399.CS', '603416.CS', '603421.CS', '603429.CS', '603444.CS', '603456.CS', '603508.CS', '603515.CS', '603517.CS', '603518.CS', '603519.CS', '603520.CS', '603528.CS', '603555.CS', '603556.CS', '603558.CS', '603559.CS', '603566.CS', '603567.CS', '603568.CS', '603569.CS', '603577.CS', '603578.CS', '603579.CS', '603585.CS', '603588.CS', '603589.CS', '603598.CS', '603599.CS', '603600.CS', '603601.CS', '603603.CS', '603606.CS', '603608.CS', '603609.CS', '603611.CS', '603615.CS', '603616.CS', '603618.CS', '603626.CS', '603628.CS', '603630.CS', '603633.CS', '603636.CS', '603637.CS', '603638.CS', '603639.CS', '603658.CS', '603660.CS', '603663.CS', '603665.CS', '603667.CS', '603668.CS', '603669.CS', '603677.CS', '603678.CS', '603686.CS', '603688.CS', '603689.CS', '603690.CS', '603696.CS', '603698.CS', '603699.CS', '603701.CS', '603703.CS', '603708.CS', '603716.CS', '603718.CS', '603726.CS', '603727.CS', '603729.CS', '603737.CS', '603738.CS', '603766.CS', '603777.CS', '603778.CS', '603779.CS', '603788.CS', '603789.CS', '603798.CS', '603799.CS', '603800.CS', '603806.CS', '603808.CS', '603811.CS', '603816.CS', '603817.CS', '603818.CS', '603819.CS', '603822.CS', '603823.CS', '603828.CS', '603838.CS', '603839.CS', '603843.CS', '603858.CS', '603859.CS', '603861.CS', '603866.CS', '603868.CS', '603869.CS', '603877.CS', '603878.CS', '603881.CS', '603883.CS', '603885.CS', '603886.CS', '603887.CS', '603888.CS', '603889.CS', '603898.CS', '603899.CS', '603900.CS', '603901.CS', '603903.CS', '603908.CS', '603909.CS', '603918.CS', '603919.CS', '603928.CS', '603929.CS', '603936.CS', '603939.CS', '603955.CS', '603958.CS', '603959.CS', '603960.CS', '603966.CS', '603968.CS', '603969.CS', '603977.CS', '603979.CS', '603986.CS', '603987.CS', '603988.CS', '603989.CS', '603990.CS', '603991.CS', '603993.CS', '603996.CS', '603997.CS', '603998.CS', '603999.CS']
```
获取RU的可以交易标的列表
```
ruList=api.getConstituentSymbols(indexs="ru.PRD",tradeDate=20170403)  
```
返回结果：
```
['ru1704.CF', 'ru1705.CF', 'ru1706.CF', 'ru1707.CF', 'ru1708.CF', 'ru1709.CF', 'ru1710.CF', 'ru1711.CF', 'ru1801.CF', 'ru1803.CF']
```
获取优品行业板块集合的列表
```
upList = api.getConstituentSymbols(indexs="UPPLA.SET",tradeDate=20170403)  

```
返回结果：
```
['880001.PLA', '880002.PLA', '880003.PLA', '880004.PLA', '880007.PLA', '880008.PLA', '880009.PLA', '880010.PLA', '880012.PLA', '880013.PLA', '880014.PLA', '880015.PLA', '880016.PLA', '880017.PLA', '880018.PLA', '880019.PLA', '880020.PLA', '880021.PLA', '880022.PLA', '880023.PLA', '880024.PLA', '880025.PLA', '880026.PLA', '880027.PLA', '880028.PLA', '880029.PLA', '880030.PLA', '880031.PLA', '880033.PLA', '880035.PLA', '880039.PLA', '880041.PLA', '880042.PLA', '880043.PLA', '880044.PLA', '880045.PLA', '880046.PLA', '880047.PLA', '880048.PLA', '880049.PLA', '880050.PLA', '880051.PLA', '880052.PLA', '880053.PLA', '880056.PLA', '880057.PLA', '880058.PLA', '880059.PLA', '880060.PLA', '880064.PLA', '880065.PLA']
```

### getContinuousSymbol-获取指定交易日连续/主力等合约对应的合约代码
**函数原型：**
```
getContinuousSymbol (maincfs,tradeDate)
```
传入连续/主力合约名称，获取对应的实际的标准合约代码。期货专用。

**参数：**

| 参数      | 类型 | 是否必填 | 说明                                                         |
| --------- | ---- | -------- | ------------------------------------------------------------ |
| maincfs   | str  | 是       | 连续合约的代码，包括主力、次主力合约，当月、次月、下季、隔季连续合约 |
| tradeDate | int  | 否       | 指定某个交易日，不填则默认为回测当天。形式为YYYYMMDD         |

**返回：**对应的期货合约代码。

**举例：**

获取主力合约在某交易日对应的标准合约
```
api.getContinuousSymbol (maincfs="jmZ0.CF", tradeDate=20180820)
```
返回结果：
```
jm1901.CF
```
获取当月合约在某交易日对应的标准合约
```
api.getContinuousSymbol (maincfs="IFM0.CF", tradeDate=20180820)
```
返回结果：
```
IF1809.CF
```


### getFieldsOneDay-获取多只股票单个交易日的数据
获取股票的股本、市值等信息，主要是日频和部分季频因子数据。数据信息详见[股票交易指标-日频](data7.html#股票交易指标-日频)和[股票财务数据-季频](data7.html#股票财务数据-季频)。

调用此api之前需在初始化onInitialize中setRequireData中设置所需股票和对应属性字段，不然取不到数据

**函数原型**

```
getFieldsOneDay(symbols, fields=None,tradeDate,df = True)
```
**参数：**

| 参数      | 类型        | 是否必填 | 说明                                                       |
| --------- | ----------- | -------- | ---------------------------------------------------------- |
| symbols   | str or list | 是       | 股票或者股票列表                                           |
| fields    | str or list | 是       | 选取的股票属性，可以选择多个                               |
| tradeDate | int         | 否       | 指定某个交易日，不填则默认为回测上一交易日。形式为YYYYMMDD |
| df        | bool        | 否       | 返回是否采用dataFrame格式，默认为True                      |

**返回：**df为True时返回dataFrame格式，df为False时返回dict类型。

**举例：**

获取多只标的在某交易日的金融数据
```
#首先在def onInitialize(api)中设置缓存数据
api.setRequireData(instsets=[],symbols=["000002.CS", "600000.CS"],
                               fields=["PE", "TRADE_STA"],
                               bars=[(ETimeSpan.DAY_1,20)]) #设置股票池，缓存要获取的因子,设置要取的因子数据为前20个交易日以内
#在对应的事件回调中调用api获取数据。对于日频因子，获取的某一交易日一定要在准备的日K条数之内，否则获取不到值。在onBeforeMarketOpen中调用如下：
data = api.getFieldsOneDay(["000002.CS", "600000.CS"], fields =["PE", "TRADE_STA"], tradeDate =20181122,df =True )
```
返回结果：
```
       PE  TRADE_STA     symbol  tradeDate
0  5.6400       True  000002.CS   20181122
1  5.5501       True  600000.CS   20181122
```

### getFieldsRangeDays-获取单只股票多个交易日的数据
获取一只股票多个交易日的股本、市值等信息。数据信息详见[股票交易指标-日频](data7.html#股票交易指标-日频)和[股票财务数据-季频](data7.html#股票财务数据-季频)。


调用此api之前需在初始化onInitialize中setRequireData中设置所需股票和对应属性字段，不然取不到数据。

**函数原型**
```
getFieldsRangeDays(symbol,fields,startDate ,endDate,df=True)
```
**参数：**

| 参数      | 类型        | 是否必填 | 说明                                                   |
| --------- | ----------- | -------- | ------------------------------------------------------ |
| symbol    | str         | 是       | 股票代码                                               |
| fields    | str or list | 是       | 选取的股票属性，可以选择多个                           |
| startDate | int         | 是       | 指定取数据的开始时间,YYYYMMDD                          |
| endDate   | int         | 否       | 指定取数据的结束时间，不填默认返回上一交易日，YYYYMMDD |
| df        | bool        | 否       | 返回是否采用dataFrame格式，默认为True                  |

**返回：** pandas DataFrame

**举例：**

获取某标的在一段时间内的因子数据
```
#首先在def onInitialize(api)中设置缓存数据
api.setRequireData(instsets=[],symbols=["000001.CS"] ,
                                fields=["PE","REVENUE","TRADE_STA", "TOT_SHARE", "LIST_STA"],
                                bars=[(ETimeSpan.DAY_1,20)])
#在对应的事件回调中调用api获取数据。对于日频因子，获取的交易日开始日期一定要在准备的日K条数之内，否则会获取到空值。在onBeforeMarketOpen中调用如下：
data = api.getFieldsRangeDays(symbol="000001.CS",fields = ["PE","REVENUE","TRADE_STA", "TOT_SHARE", "LIST_STA"],startDate=20181116, endDate=20181120,df=True)
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
TOT_SHARE=date.ix[0:1,["REVENUE","PE"]] #获取前两行的"REVENUE","PE"字段值
```
返回结果：
```
        REVENUE      PE
0  8.666400e+10  7.4102
1  8.666400e+10  7.6065
```

### getFieldsCountDays-获取单只股票基于某个交易日的前n条数据
获取单只股票基于某个交易日的前n条股本、市值等数据。数据信息详见[股票交易指标-日频](data7.html#股票交易指标-日频)和[股票财务数据-季频](data7.html#股票财务数据-季频)。

调用此api之前需在初始化onInitialize中setRequireData中设置所需股票和对应属性字段，不然取不到数据。


**函数原型**
```
getFieldsCountDays(symbol, fields=None, tradeDate=None, count=1, df=True)
```
**参数：**

| 参数      | 类型        | 是否必填 | 说明                                                    |
| --------- | ----------- | -------- | ------------------------------------------------------- |
| symbol    | str         | 是       | 股票代码                                                |
| fields    | str or list | 是       | 选取的股票属性，可以选择多个                            |
| count     | int         | 是       | 获取基于所传交易日往前推的N条数据（包含所传交易日当天） |
| tradeDate | int         | 否       | 交易日，不填按默认返回上一交易日，形式为YYYYMMDD        |
| df        | bool        | 否       | 返回是否采用dataFrame格式，默认为True                   |

**返回：** pandas DataFrame

**举例：**

获取基于某交易日往前推的两条数据，格式为dataFrame
```
#首先在def onInitialize(api)中设置缓存数据
api.setRequireData(instsets=[],symbols=["000001.CS"],
                               fields=["TOT_SHARE","BASIC_EPS", "PE"],
                               bars=[(ETimeSpan.DAY_1,20)])
#然后在对应的事件回调中调用api获取数据。对于日频因子，获取的数据一定要在准备的日K条数之内，否则会获取到空值。在onBeforeMarketOpen中调用如下：
datas = api.getFieldsCountDays(symbol="000001.CS", fields=["TOT_SHARE", "BASIC_EPS", "PE"], tradeDate=20181122, count=2)
```
返回结果：
```
   BASIC_EPS      PE     TOT_SHARE     symbol  tradeDate
0       1.14  7.4383  1.717041e+10  000001.CS   20181121
1       1.14  7.3962  1.717041e+10  000001.CS   20181122
```
获取上一交易日往前推的两条数据，当前交易日为20181126，格式为dataFrame。在onBeforeMarketOpen中调用如下：
```
datas = api.getFieldsCountDays(symbol="000001.CS", fields=["TOT_SHARE", "BASIC_EPS", "PE"],  count=2)
```
返回结果：
```
BASIC_EPS      PE     TOT_SHARE     symbol  tradeDate
0       1.14  7.3962  1.717041e+10  000001.CS   20181122
1       1.14  7.2350  1.717041e+10  000001.CS   20181123
```

### getStockFinFields-获取财务数据
获取股票的财务数据。数据信息详见[股票财务数据-季频](data7.html#股票财务数据-季频)。

调用此api之前需在初始化onInitialize中setSymbolPool设置相关股票池，然后在setRequireFields中设置所需字段，不然取不到字段值

**函数原型**
```
getStockFinFields(symbols,fields,tradeDate=None,report = None,df = True)
```
**参数：**

| 参数      | 类型        | 是否必填 | 说明                                                         |
| --------- | ----------- | -------- | ------------------------------------------------------------ |
| symbols   | str         | 是       | 股票代码或列表                                               |
| fields    | str or list | 是       | 选取的股票属性，可以选择多个                                 |
| tradeDate | int         | 否       | 交易日，形式为YYYYMMDD，为了避免未来函数只能获取到所传交易日之前的数据，不填默认返回上一交易日 |
| report    | str         | 否       | 指定报告期，例如'2016Q1' - 16年一季报，'2016Q2' - 16年半年报，'2016Q3' - 16年三季报，'2016Q4'-年报，report缺省时返回标的最新的报告期数据。 |
| df        | boo         | 否       | 返回是否采用dataFrame格式，默认为True                        |

**举例：**

获取000001.CS的2016年年报数据，当前交易日为20181126
```
#首先在def onInitialize(api)中设置缓存数据
api.setSymbolPool(instsets=[],symbols=["000001.CS"])#设置股票池
api.setRequireFields(fields=["REVENUE", "NET_PROFIT", "NET_PROFIT_ATTRP"]）#缓存要获取的因子
#然后在对应的事件回调中调用api获取数据。在onBeforeMarketOpen中调用如下：
data = api.getStockFinFields(symbols="000001.CS",fields = ["REVENUE", "NET_PROFIT", "NET_PROFIT_ATTRP"], tradeDate = 20170505,report="2016Q4")
```
返回结果：
```
     NET_PROFIT  NET_PROFIT_ATTRP       REVENUE  reportdate     symbol  tradeDate
0  2.259900e+10      2.259900e+10  1.077150e+11    20161231  000001.CS   20170505
```

### getTableFieldOneDay-获取单标的某一交易日的表格型数据
获取单只标单个交易日的表格型数据，数据信息详见[表格型数据](data7.html#表格型数据)

调用此api之前需在初始化onInitialize里setRequireData中设置所需股票和对应属性字段，不然取不到数据。

**函数原型：**
```
getTableFieldOneDay( symbol, field, tradeDate=None, columns=None, df = True)　
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                                                       |
| --------- | ---- | -------- | ---------------------------------------------------------- |
| symbol    | str  | 是       | 股票代码                                                   |
| field     | str  | 是       | 表格名 ，详见[表格型数据](data7.html#表格型数据)           |
| tradeDate | int  | 否       | 指定某个交易日，不填则默认为回测上一交易日。形式为YYYYMMDD |
| columns   | list | 否       | 返回的字段；为空时,默认返回所有列,否则返回指定的列         |
| df        | bool | 否       | 返回是否采用dataFrame格式，默认为True                      |

**举例：**

获取a1901.CF上一交易日成交量排名列表

```
#首先在def onInitialize(api)中设置缓存数据
api.setRequireData(instsets=[],symbols=["a1901.CF"],
                             fields=[ 'FUT_TRADE_RANK'],
                             bars=[(ETimeSpan.DAY_1,20)])  
#然后在对应的事件回调中调用api获取数据。对于日频因子，获取的数据一定要在准备的日K条数之内，否则会获取到空值。在onBeforeMarketOpen中调用如下：
api.getTableFieldOneDay(symbol  ="a1901.CF",field= 'FUT_TRADE_RANK', tradeDate =20181120,columns=['rank', 'name', 'volume', 'volumeDiff'], df=True)
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
6   兴证期货     7   20181120   4901.0      -151.0
7   徽商期货     8   20181120   4615.0      1769.0
8   国投安信     9   20181120   4519.0      -343.0
9   方正中期    10   20181120   4003.0       207.0
10  光大期货    11   20181120   3771.0      1155.0
11  国信期货    12   20181120   3698.0     -2126.0
12  永安期货    13   20181120   3457.0       379.0
13  创元期货    14   20181120   3200.0      1742.0
14  中信建投    15   20181120   3096.0      1776.0
15  银河期货    16   20181120   3059.0       543.0
16  广发期货    17   20181120   2804.0       438.0
17  南华期货    18   20181120   2392.0      -231.0
18  渤海期货    19   20181120   2318.0      1766.0
19  上海大陆    20   20181120   2245.0      1813.0
```
获取申万一级行业板块的K线数据
```
#先在def onInitialize(api):中设置股票池、缓存要获取的因子和K线条数：
api.setRequireData(instsets=[],symbols=['801780.PLA'],
                              fields=[ "SW_MKT"],
                             bars=[(ETimeSpan.DAY_1,20)])  
#然后在对应的事件回调中调用api获取数据。对于日频因子，获取的数据一定要在准备的日K条数之内，否则会获取到空值。在onBeforeMarketOpen中调用如下：
api.getTableFieldOneDay('801780.PLA', 'SW_MKT', 20181122, df=True)
```
返回结果：
```
     close     high      low     open  preClose  totalTurnover   totalVolume  tradeDate
0  3302.34  3320.06  3291.49  3320.06   3314.79    752355335.0  5.570303e+09   20181122
   
```


### getTableFieldRangeDays-获取单标的多个交易日的表格型数据
获取单标的多个交易日的表格型数据，获取的ttype与getTableFieldOneDay相同。

调用此api之前需在初始化onInitialize里setRequireData中设置所需股票和对应属性字段，不然取不到数据。


**函数原型：**
```
　getTableFieldRangeDays( symbol, field, startDate, endDate=None, columns=None, df = True)　
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                                                   |
| --------- | ---- | -------- | ------------------------------------------------------ |
| symbol    | str  | 是       | 股票代码                                               |
| field     | str  | 是       | 表格名 ，详见[表格型数据](data7.html#表格型数据)       |
| startDate | int  | 是       | 指定取数据的开始时间,YYYYMMDD                          |
| endDate   | int  | 否       | 指定取数据的结束时间，不填默认返回上一交易日，YYYYMMDD |
| columns   | list | 否       | 返回的字段；为空时,默认返回所有列,否则返回指定的列     |
| df        | bool | 否       | 返回是否采用dataFrame格式，默认为True                  |

**举例：**

获取a1901.CF多个交易日的成交量排名列表

```
#首先在def onInitialize(api)中设置缓存数据
api.setRequireData(instsets=[],symbols=["a1901.CF"],
                             fields=[ 'FUT_TRADE_RANK'],
                             bars=[(ETimeSpan.DAY_1, 20)])
#然后在对应的事件回调中调用api获取数据。对于日频因子，获取的数据一定要在准备的日K条数之内，否则会获取到空值。在onBeforeMarketOpen中调用如下：
api.getTableFieldRangeDays(symbol  ="a1901.CF",field= 'FUT_TRADE_RANK',startDate=20181122,endDate =20181124,columns=['rank', 'name', 'volume', 'volumeDiff'], df=True) 
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
10  华安期货    11   20181122   2890.0      -479.0
11  中信建投    12   20181122   2825.0       512.0
12  永安期货    13   20181122   2526.0       339.0
13  光大期货    14   20181122   2249.0     -1020.0
14  国投安信    15   20181122   2027.0     -1504.0
15  南华期货    16   20181122   2024.0       183.0
16  银河期货    17   20181122   1640.0     -1070.0
17  东航期货    18   20181122   1587.0      -132.0
18  浙商期货    19   20181122   1570.0       273.0
19  安粮期货    20   20181122   1386.0      -171.0
20  东证期货     1   20181123  16670.0      7965.0
21  中信期货     2   20181123  12174.0      5173.0
22  国泰君安     3   20181123  11408.0      5991.0
23  海通期货     4   20181123   8317.0      3484.0
24  华泰期货     5   20181123   7781.0      3943.0
25  方正中期     6   20181123   6921.0      2878.0
26  华安期货     7   20181123   6349.0      3459.0
27  徽商期货     8   20181123   5378.0      2204.0
28  国信期货     9   20181123   5031.0      1772.0
29  兴证期货    10   20181123   4877.0      1887.0
30  光大期货    11   20181123   4499.0      2250.0
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
#首先在def onInitialize(api):中设置股票池、缓存要获取的因子和K线条数：
api.setRequireData(instsets=[],symbols=['801010.PLA'],
                                fields=[ "SW_MKT"],
                               bars=[(ETimeSpan.DAY_1,20)])
#然后在对应的事件回调中调用api获取数据。对于日频因子，获取的数据一定要在准备的日K条数之内，否则会获取到空值。在onBeforeMarketOpen中调用如下：
api.getTableFieldRangeDays( '801010.PLA', "SW_MKT", 20181122,20181123, columns=None, df = True)
```
返回结果为：
```
	close     high      low     open  preClose  totalTurnover   totalVolume  tradeDate
0  2272.95  2293.86  2264.67  2289.74   2286.45   8.149344e+08  5.235480e+09   20181122
1  2230.37  2276.36  2222.91  2276.36   2272.95   1.033341e+09  6.586772e+09   20181123
```


### getTableFieldCountDays-获取单标的多条表格型数据
获取单标的多条表格型数据，获取的type与getTableFieldOneDay相同。

调用此api之前需在初始化onInitialize中setRequireData设置股票和所需[表格型数据](data7.html#表格型数据) ，不然取不到字段值

**函数原型：**
```
　　getTableFieldCountDays( symbol, field, tradeDate=None, count=None, columns=None, df = True)　
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                                                    |
| --------- | ---- | -------- | ------------------------------------------------------- |
| symbol    | str  | 是       | 股票代码                                                |
| field     | str  | 是       | 表格名 ，详见[表格型数据](data7.html#表格型数据)        |
| tradeDate | int  | 否       | 指定取数据的开始时间,不填默认返回上一交易日，YYYYMMDD   |
| count     | int  | 是       | 获取基于所传交易日往前推的N条数据（包含所传交易日当天） |
| columns   | list | 否       | 返回的字段；为空时,默认返回所有列,否则返回指定的列      |
| df        | bool | 否       | 返回是否采用dataFrame格式，默认为True                   |

**举例：**

获取a1901.CF多个交易日成交量排名列表

```
#首先在def onInitialize(api)中设置缓存数据
    api.setRequireData(instsets=[], symbols=["a1901.CF"],
                       fields=['FUT_TRADE_RANK'],  
                       bars=[(ETimeSpan.DAY_1, 20)])
#然后在对应的事件回调中调用api获取数据。对于日频因子，获取的数据一定要在准备的日K条数之内，否则会获取到空值。在onBeforeMarketOpen中调用如下：
#获取a1901基于当前交易日（20181122）往前推共2个交易日的成交排名列表
api.getTableFieldCountDays(symbol="a1901.CF", field='FUT_TRADE_RANK',tradeDate=20181122,
                        count=2, columns=['rank', 'name', 'volume', 'volumeDiff'], df=True)

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
12  银河期货    13   20181121   2710.0      -349.0
13  广发期货    14   20181121   2680.0      -124.0
14  中信建投    15   20181121   2313.0      -783.0
15  永安期货    16   20181121   2187.0     -1270.0
16  南华期货    17   20181121   1841.0      -551.0
17  东航期货    18   20181121   1719.0      -355.0
18  弘业期货    19   20181121   1676.0       643.0
19  大有期货    20   20181121   1673.0      -296.0
20  东证期货     1   20181122   8705.0     -4422.0
21  中信期货     2   20181122   7001.0     -6352.0
22  国泰君安     3   20181122   5417.0     -1273.0
23  海通期货     4   20181122   4833.0     -3542.0
24  方正中期     5   20181122   4043.0      -619.0
25  华泰期货     6   20181122   3838.0     -2074.0
26  国信期货     7   20181122   3259.0      -308.0
27  徽商期货     8   20181122   3174.0       173.0
28  渤海期货     9   20181122   3116.0      1557.0
29  兴证期货    10   20181122   2990.0     -2049.0
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
#首先在def onInitialize(api):中设置股票池、缓存要获取的因子和K线条数：
api.setRequireData(instsets=[],symbols=['801010.PLA'],
                                fields=[ "SW_MKT"],
                               bars=[(ETimeSpan.DAY_1,20)])
#然后在对应的事件回调中调用api获取数据。对于日频因子，获取的数据一定要在准备的日K条数之内，否则会获取到空值。在onBeforeMarketOpen中调用如下：
api.getTableFieldCountDays(symbol='801010.PLA',field="SW_MKT",tradeDate= 20181122,count=3, columns=None, df = True)
```
返回结果如下：
```
     close     high      low     open  preClose  totalTurnover   totalVolume  tradeDate
0  2263.10  2312.21  2261.69  2307.56   2320.63   1.143088e+09  7.398032e+09   20181120
1  2286.45  2288.83  2240.66  2251.17   2263.10   1.015610e+09  6.188521e+09   20181121
2  2272.95  2293.86  2264.67  2289.74   2286.45   8.149344e+08  5.235480e+09   20181122
```
### getTradeDayInterval-获取交易日数
获取某时间区间交易日数，起止时间只能是回测区间内的

**函数原型：**
```
getTradeDayInterval（symbol, startDate,endDate）
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                   |
| --------- | ---- | -------- | ---------------------- |
| symbol    | str  | 是       | 标的代码               |
| startDate | int  | 是       | 开始时间，格式为YYMMDD |
| endDate   | int  | 是       | 结束时间，格式为YYMMDD |

**返回值：**返回值为交易日数

**举例：**

获取一段时间内的交易日数，包含所传起止日期
```
data = api.getTradeDayInterval(symbol="000002.CS",startDate= 20151217, endDate=20160704)
```
返回结果：
```
3  
(注：该结果过滤了万科A在这段时间内的停牌）
```

### getPrevTradeDate-获取上一个交易日期
获取上一个交易日期

**函数原型：**
```
getPrevTradeDate(tradeDate，mktIndex=none)
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                                  |
| --------- | ---- | -------- | ------------------------------------- |
| tradeDate | int  | 是       | 交易日，格式为YYMMDD                  |
| mktIndex  | str  | 否       | 市场代码，不填默认返回A股市场的交易日 |
**返回值：**返回上一交易日的日期

**举例：**

```
api.getPrevTradeDate(tradeDate=20150101) #返回值为20141231
api.getPrevTradeDate(tradeDate=20150101,mktIndex="CS") #返回值为20141231
```
### getNextTradeDate-获取下一个交易日期
获取下一个交易日期

**函数原型：**
```
getNextTradeDate(tradeDate，mktIndex=none)
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                                  |
| --------- | ---- | -------- | ------------------------------------- |
| tradeDate | int  | 是       | 开始时间，格式为YYMMDD                |
| mktIndex  | str  | 否       | 市场代码，不填默认返回A股市场的交易日 |
**返回值：**返回下一交易日的日期

**举例：**

```
api.getNextTradeDate(tradeDate =20150101) #返回值为20150105
api.getNextTradeDate(tradeDate =20150101,mktIndex="CS") #返回值为20150105
```
### getTradeDates-获取一段时间内交易日期
指定起止日期，获取某个市场一段时间的交易日期

**函数原型：**
```
getTradeDates（startDate,endDate,mktIndex = None）
```
**参数：**

| 参数      | 类型        | 是否必填 | 说明                                 |
| --------- | ----------- | -------- | ------------------------------------ |
| startDate | date        | 是       | 开始时间（含当天），格式为YYMMDD     |
| endDate   | date        | 是       | 结束时间（含当天），格式为YYMMDD     |
| mktIndex  | str or list | 否       | 交易市场,不填默认返回A股市场的交易日 |

**返回值：**交易日list，日期格式YYYMMDD，类型为int

**举例：**
获取一段时间内的交易日期，包含所传起止日期
```
api.getTradeDates(startDate =20171001, endDate =20171011)
```
返回结果：
```
[20171009, 20171010, 20171011]
```
### getPrevTradeDates-获取某个市场截止指定日期的多个交易日期
指定截止日期，返回某个市场指定天数的交易日期

**函数原型：**
```
getPrevTradeDates(tradeDate, count, mktIndex=None)
```

**参数：**

| 参数      | 类型        | 是否必填 | 说明                                 |
| --------- | ----------- | -------- | ------------------------------------ |
| tradeDate | date        | 是       | 截止日期（含当天），格式为YYMMDD     |
| count     | int         | 是       | 返回天数（含当天），count需大于0     |
| mktIndex  | str or list | 否       | 交易市场,不填默认返回A股市场的交易日 |

**返回值：**交易日list，日期格式YYYMMDD，类型为int


**举例：**

```
api.getPrevTradeDates(20181008,7)#返回A股市场截止20181008日7个交易日日期
```
返回结果
```
 [20180920, 20180921, 20180925, 20180926, 20180927, 20180928, 20181008]
```



### isSuspend-是否停牌
判断标的当日是否停牌

**函数原型：**
```
isSuspend(symbol,tradeDate)
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                                                         |
| --------- | ---- | -------- | ------------------------------------------------------------ |
| symbol    | str  | 是       | 策略中的证券代码                                             |
| tradeDate | int  | 否       | 交易日，形式为YYYYMMDD，可以查询历史上的状态，不填默认返回当天的数据 |

**返回值：**返回True表示停牌，返回False表示未停牌

**举例：**
```
 isSuspend = api.isSuspend(symbol ="000001.CS",tradeDate =20180104)#获取000001.CS在20180104是否停牌
```
### isST-是否ST股
判断标的当日是否ST股

**函数原型：**
```
isST(symbol,tradeDate）
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                                                         |
| --------- | ---- | -------- | ------------------------------------------------------------ |
| symbol    | str  | 是       | 策略中的证券代码                                             |
| tradeDate | int  | 否       | 交易日，形式为YYYYMMDD，可以查询历史上的状态，不填默认返回当天的数据 |

**返回值：**返回True表示ST股，返回False表示非ST股

**举例：**
```
 isST = api.isST(symbol ="000001.CS",tradeDate =20180104)#获取000001.CS在20180104是否ST
```
### isListed-是否上市
判断标的当日是否上市

**函数原型：**
```
isListed(symbol,tradeDate）
```
**参数：**

| 参数      | 类型 | 是否必填 | 说明                                                         |
| --------- | ---- | -------- | ------------------------------------------------------------ |
| symbol    | str  | 是       | 策略中的证券代码                                             |
| tradeDate | int  | 否       | 交易日，形式为YYYYMMDD，可以查询历史上的状态，不填默认返回当天的数据 |

**返回值：**返回True表示上市，返回False表示未上市或者已退市

**举例：**
```
 isListed = api.isListed(symbol ="000001.CS",tradeDate =20180104)#获取000001.CS在20180104是否上市
```

## 其他函数

### getDefineData-获取命令行参数
**函数原型：**

```
getDefineData(name, default_value, etype)
```

**参数：**

| 参数          | 类型  | 是否必填 | 说明                        |
| ------------- | ----- | -------- | --------------------------- |
| name          | str   | 是       | 命令行传入的名称            |
| default_value | etype | 否       | 默认值为空，类型和etype一致 |
| etype         | type  | 是       | 类型，有str、int等          |

**举例：**

```
在命令行输入如下命令 python main.py --name=upchina --value=20，在策略里可以通过api.getDefineData()获取，形式如下：
api.getDefineData(name='name', default_value='myname', etype=str)
api.getDefineData(name='value', default_value=0, etype=int)

上一个结果会获取到'upchina'
下一个结果获取到20
```

**示例：**

为了能动态的调整策略参数，可以通过命令行传入参数值，举例如下：

新建策略文件example.py，通过N条历史K线的收盘价的平均值和当前收盘价作比较进行下单，运行结束后调用onTerminate事件，将结果存储于result.txt文件中，代码如下：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#回测周期为20180702-20180719
from etasdk import *
import time
from datetime import datetime

def onInitialize(api):
    print("onInitialize")
    #获取命令行参数
    api.bar_len = api.getDefineData('bar_len', 10, int)
    api.setRequireData(
        symbols=['000001.CS'],
        fields=["TOT_SHARE", "SP_RESER", "PE"],
        bars=[
            (ETimeSpan.DAY_1, api.bar_len)
        ]
    )

def onBeforeMarketOpen(api, data):
    print("time----",data)
    symbolPool = api.getSymbolPool()
    api.setFocusSymbols(symbolPool)

def onBar(api, bar):
    print("onbar: %d"%bar.tradeDate)
    hisBar=api.getBarsHistory(bar.symbol, ETimeSpan.DAY_1, api.bar_len, skipSuspended=1, df=True,priceMode = EPriceMode.FORMER)
    close=hisBar["close"].mean()
    current_price = hisBar["close"][api.bar_len-1]
    print close,current_price,bar.close
    cash = api.getAccount(bar.symbol).cashAvailable
    qty = int(0.8 * cash / current_price / 100) * 100
    if current_price > close:
        api.targetPosition(bar.symbol, qty)

def onTerminate(api, exit_info):
    LOG.INFO("onTerminate")
    print(exit_info)

    #存储结果
    line = str(api.bar_len)  + ':'
    line += str(exit_info.sharpeRatio) + ',' + str(exit_info.maximumDrawdown) + ',' + str(exit_info.annualizedReturn)
    line += '\n'
    file = r'./result.txt'
    with open(file, 'a+') as f:
        f.write(line)    

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
    status = subprocess.call("python main.py" + args, shell=True)

#运行会循环bar_len, 动态调整参数, 调用策略, 并在策略中保存最后的结果

```
通过运行batch.py文件，会循环调用main.py，K线条数分别是10、12、14、16、18，循环运行五次结果会存储在result.txt文件中，例子结果如下：
```
10:-7.80182813879,0.63050232,-20.2195962497
12:-7.80182813879,0.63050232,-20.2195962497
14:0.0,0.0,0.0
16:0.0,0.0,0.0
18:0.0,0.0,0.0
```
通过对比几次运行结果的数据，来选择最优的参数，以上举例仅供参考。


### LOG.*-日志输出
**函数原型：**

```
LOG.DEBUG(logstr) #DEBUG级别日志
LOG.INFO(logstr) #INFO级别日志
LOG.ERROR(logstr) #ERROR级别日志
```

**参数：**

| 参数   | 类型 | 说明     |
| ------ | ---- | -------- |
| logstr | str  | 日志内容 |
**返回值：**日志打印的内容


**举例：**

日志打印的级别与config配置中的"loglevel"级别一一对应，即loglevel为info就调用LOG.INFO，loglevel为error就调用LOG.ERROR输出日志。

```
prev= api.getPrevTradeDate(tradeDate)                                #获取上一交易日
curr=api.getCurrTradeDate()                                          #获取当前交易日
LOG.INFO("current tradedate %s,last tradedate %s" ,curr,prev)        #打印当前交易日和上一交易日
LOG.INFO("current tradedate = %s ", curr)                            #打印当前交易日
```
策略打印日志注意事项：
```
1、尽量不要print中文，不同的python环境，print中文后，有可能出现莫名其妙的问题
2、选股、下单等依据数据的关键数据点，要用 LOG.INFO 输出日志，这样模拟盘会把日志输出到algo.python_main.log 这个文件里，回测的时候不会输出，因为回测日志级别默认是error，如果回测也想打印日志可以用LOG.ERROR或者将config中的日志级别改成info（后者会影响回测速度）； 
3、更重要的数据，可以调用 api.sendCustomMsg 把日志信息输出到 web平台上，但是这个接口不要调用太多，有性能问题
```

### sendCustomMsg-设置用户自定义运行消息
设置用户自定义运行消息，会显示在界面上的日志中

**函数原型：**

```
sendCustomMsg(msgstr)
```
**参数：**

| 参数   | 类型 | 说明 |
| ------ | ---- | ---- |
| msgstr | str  | 内容 |

**返回值：**Web端日志显示需要显示的信息

**举例：**

```
api.sendCustomMsg(str("focus symbol:"+";".join(code_)))      #Web端日志中显示关注股票
```
**返回值：**

```
2019-01-25 08:30:03 795 - INFO - focus symbol:002075.CS;300003.CS;000963.CS;002798.CS
```

### isTradingNow-标的当前时间是否为交易时间
对于股票交易时间为每个交易日9:30-11:30，13:00-15:00；但是对于期货、外汇等不同标的交易时间并不一致。

**函数原型：**

```
isTradingNow (symbol)
```
**参数：**

| 参数   | 类型 | 是否必填 | 说明     |
| ------ | ---- | -------- | -------- |
| symbol | str  | 是       | 标的代码 |

**返回值：**当前是否交易，bool类型。

**举例：**

```
istradingnow = api.isTradingNow (symbol ="000001.CS")  #获取000001.CS是否交易
```

### timeNow-获取时间

**函数原型：**

```
timeNow ()
```
**参数：**无

**返回值：**当前行情时间戳,int类型,单位毫秒。

**举例：**ms=api.timeNow ()