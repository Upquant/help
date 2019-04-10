# 策略示例
## 入门策略
### buySellSymbol-股票买卖策略

``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from etasdk import *

'''
股票策略：买卖策略
本策略是一个简单的买卖策略，主要是体现如何下单的。当没有持仓时就进行买入操作，当有持仓时，就进行卖出平仓操作
回测标的：平安银行（000001.CS）
回测周期：20180702-20181101
撮合周期：日K
更多详情请看帮助文档：http://quant.upchina.com/doc/first.html
'''
if __name__ == '__main__':
    # 读取配置文件并启动策略
    config = {
        "host": "120.55.22.60",
        "port": 80,
        "user": "SDK-User",         # Web端：点击量化平台右上角用户名可查看填写
        "token": "SDK-Token",          # Web端：点击量化平台右上角用户名可查看填写
        "strategy_name": "buySellSymbol",   # Web端：策略名称
        "console": 0,
        "loglevel": "ERROR",         # 显示日志级别，具体可查看帮助文档
        "instance_id": "20190222-113322-609-001BI",    # Web端：回测记录中本策略的回测ID,点击复制ID,粘贴此处即可
        # 初始化参数
        "initialize": {
            # symbols里设置单个股票或者其他有效的标的
            # symbol_sets里设置标的集合，如此处的IF.PRD指的就是IF品种的所有标准合约，不包含IFZ0.CF和IFZ1.CF
            # 注：symbols和symbol_sets不能全部为空
            'symbols': ['000001.CS'],
            # fields缓存的因子
            # 'fields': ['FUT_TRADE_RANK', 'FUT_SHORT_RANK', 'FUT_LONG_RANK', 'STK_HOLD_DET', 'STK_HOLD_STAT', 'ST_STA','LIST_STA', 'TRADE_STA', 'NET_PROFIT', 'PB', 'PE', 'PS', 'PE_TTM'],
            # 必填：设置所需k线类型，interval可以选取1分钟，5分钟，15分钟，30分钟，60分钟，日k，Tick级别，count为提前缓存的数目
            'prepared_bars': [{'interval': ETimeSpan.DAY_1, 'count': 60}],
            # 选填：设置撮合参数 interval为撮合周期，time_ranges为日内撮合时间段，将只在指定时间段以interval周期回调on_bar或on_handle_date
            'match_param': {
                'interval': ETimeSpan.DAY_1,
                # 'time_ranges': [('12:00', '14:45'), ('10:10', '11:00')]
            },

            'mode': 'both',                       # 选填：为group_only，则只响应on_handle_data；为single_only，则只响应on_bar；为both，则两个回调同时响应
            'commission': 0.0002,                       # 选填：设置手续费

            # 选填：仅回测生效的项，不设置时以web页面上的设置为准
            'backtest': {
                'start_date': 20180702,                 # 回测开始日期
                'end_date': 20181101,                   # 回测结束日期
                'slippage': 1,                          # 设置滑点,取值为整数,标的价格浮动最小单位值的整数倍
                # 设置账户初始资金
                'cash': {
                    'CS': 1000000,                            # 设置股票账户资金
                    'CF': 0                      # 设置期货账户资金
                }
            },

            # 选填：仅模拟盘/实盘生效的项
            'realtime': {
                'pre_market_time': '8:50',            # 设置开盘前回调on_before_market_open响应事件时间，默认为web页面上设置的时间
                'closing_time': '14:55',               # 设置收盘前最后一次回调on_bar或on_handle_data响应事件时间，默认为web页面上设置的时间
                'group_timeout': 10000,                # 设置group模式下的组播，等待多少ms则强制响应，默认为5000ms
                'timer_cycle': 60000                   # 设置onTimer响应时间(ms), 为0则不响应onTimer，默认情况下不回调onTimer
            }
        }
    }

    etasdk.load_config_json(config).start()



# 必要：每个交易日回测/模拟/实盘前要做的操作
def on_before_market_open(api, date_now):
    # 必要：获取当个交易日需关注的股票
    # 设置今天关注的股票，如果股票池中只有一只股票，则全部关注即可；股票池中股票过多时，需要先选股在设置部分关注以提高运行效率
    api.context.pool.focus = api.context.pool.all


# 必要：每天回测/交易 策略 两种方式回调  on_bar/on_handle_data
# 用日K做回测，所以每天只会进入一次，收盘时响应此函数
# 策略逻辑：第一天买入，第二天卖出；判断逻辑，如果当前有仓位，则平仓，如果当前没有仓位，则买入
def on_bar(api, bar):
    symbol_positions = api.get_symbol_positions()  # 获取持仓标的信息
    if symbol_positions:  # 如果持仓数大于0
        # 平仓
        log.info("target position zero!")
        api.target_position(symbol=bar.symbol, qty=0)  # 卖出 ，symbol = 选中标的，数量变为0
    else:
        # 市价下单1000股
        log.info("target position 1000!")
        api.target_position(symbol=bar.symbol, qty=50000)  # 买入 symbol = 选中标的，直至持仓数达500股


# 定时器，模拟盘响应；回测时可忽略
def on_timer(api):
    pass


# 策略终止时响应；回测运行终止时会调用一次, exit_info返回值为终止消息
def on_terminate(api, exit_info):
    log.info("***************onTerminate*********")  # 输出日志



```
### buySellFutures-期货买卖策略

``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-

'''
期货策略：买卖策略
本策略是一个简单的期货买卖策略，主要是体现期货简单买卖
更多详情请看帮助文档：http://quant.upchina.com/doc/first.html
'''
from etasdk import *

if __name__ == '__main__':
    config = {
        "host": "120.55.22.60",
        "port": 80,
        "user": "SDK-User",         # Web端：点击量化平台右上角用户名可查看填写
        "token": "SDK-Token",          # Web端：点击量化平台右上角用户名可查看填写
        "strategy_name": "buySellFutures",   # Web端：策略名称
        "console": 0,
        "loglevel": "ERROR",         # 显示日志级别，具体可查看帮助文档
        "instance_id": "20180605-165138-917-002BI",    # Web端：回测记录中本策略的回测ID,点击复制ID,粘贴此处即可
        # 初始化参数
        "initialize": {
            # symbols里设置单个股票或者其他有效的标的
            # symbol_sets里设置标的集合，如此处的IF.PRD指的就是IF品种的所有标准合约，不包含IFZ0.CF和IFZ1.CF
            # 注：symbols和symbol_sets不能全部为空
            'symbols': ['IFZ0.CF'],
            'symbol_sets': ['IF.PRD'],
            # fields缓存的因子
            # 'fields': ['FUT_TRADE_RANK', 'FUT_SHORT_RANK', 'FUT_LONG_RANK', 'STK_HOLD_DET', 'STK_HOLD_STAT', 'ST_STA','LIST_STA', 'TRADE_STA', 'NET_PROFIT', 'PB', 'PE', 'PS', 'PE_TTM'],
            # 必填：设置所需k线类型，interval可以选取1分钟，5分钟，15分钟，30分钟，60分钟，日k，Tick级别，count为提前缓存的数目
            'prepared_bars': [{'interval': ETimeSpan.MIN_5, 'count': 100}],
            # 选填：设置撮合参数 interval为撮合周期，time_ranges为日内撮合时间段，将只在指定时间段以interval周期回调on_bar或on_handle_date
            'match_param': {
                'interval': ETimeSpan.MIN_5,
                # 'time_ranges': [('12:00', '14:45'), ('10:10', '11:00')]
            },

            'mode': 'both',                       # 选填：为group_only，则只响应on_handle_data；为single_only，则只响应on_bar；为both，则两个回调同时响应
            'commission': 0.0002,                       # 选填：设置手续费

            # 选填：仅回测生效的项，不设置时以web页面上的设置为准
            'backtest': {
                'start_date': 20180629,                 # 回测开始日期
                'end_date': 20181228,                   # 回测结束日期
                'slippage': 1,                          # 设置滑点,取值为整数,标的价格浮动最小单位值的整数倍
                # 设置账户初始资金
                'cash': {
                    'CS': 0,                            # 设置股票账户资金
                    'CF': 1000000                      # 设置期货账户资金
                }
            },

            # 选填：仅模拟盘/实盘生效的项
            'realtime': {
                'pre_market_time': '19:00',            # 设置开盘前回调on_before_market_open响应事件时间，默认为web页面上设置的时间
                'closing_time': '14:55',               # 设置收盘前最后一次回调on_bar或on_handle_data响应事件时间，默认为web页面上设置的时间
                'group_timeout': 10000,                # 设置group模式下的组播，等待多少ms则强制响应，默认为5000ms
                'timer_cycle': 60000                   # 设置onTimer响应时间(ms), 为0则不响应onTimer，默认情况下不回调onTimer
            }
        }
    }

    etasdk.load_config_json(config).start()


def on_before_market_open(api, date_now):
    print(date_now.date)
    """
    必要：每个交易日回测/模拟/实盘前需要响应的事件
    :param api:
    :param trade_now:
    :return:
    """
    api.futures_code = api.get_continuous_symbol("IFZ0.CF")  # 获得当前交易日IF主力合约代码
    print(api.futures_code)
    api.context.pool.focus = [api.futures_code, 'IFZ0.CF']

def on_handle_data(api, time_now):
    """
    仅当mode为group_only和both时响应该事件
    on_handle_data在Focus标的收齐时或者超时时响应
    必要：每天回测/交易 策略 可选择两种方式  on_bar/on_handle_data
    策略逻辑：如果价格低于前一日收盘价的0.2%则开多，止盈0.5%止损0.3%
    :param api:
    :param time_now: EtaTime
    :return:
    """

    print('------------------%s-------------------' % time_now["%Y%m%d %H:%M"])

    symbol_positions = api.get_symbol_positions()  # 获取持仓标的信息
    bars = api.get_bars_history(symbol=api.futures_code, time_span=ETimeSpan.MIN_5, count=1,
                              price_mode=EPriceMode.FORMER, skip_suspended=0)   # 获得合约当前bar数据
    factor = bars['preClose'] / bars['open']  # 当前开盘价与昨日收盘价之比
    # print(factor, bars['preClose'], bars['open'])
    # print(symbol_positions)
    if symbol_positions:  # 如果有持仓
        if api.futures_code == symbol_positions[0].symbol:  # 当前持仓为主力合约
            # 止盈0.3%止损0.5%
            if symbol_positions[0].posHigh / symbol_positions[0].posPrice > 1.03 \
                or symbol_positions[0].posLow / symbol_positions[0].posPrice < 0.95:
                # 平仓
                print(api.context.time.trade_now.date, "target position 0!")  # 录入日志
                api.target_position(symbol=symbol_positions[0].symbol, qty=0,
                                    side=EPositionSide.SHORT)  # 卖出 ，symbol = 持仓标的，数量变为0
        # 当前持仓非主力合约 则换仓为主力合约
        else:
            print(api.context.time.trade_now.date, symbol_positions[0].symbol, "换合约", api.futures_code, )
            api.target_position(symbol=symbol_positions[0].symbol, qty=0, side=EPositionSide.SHORT)
            api.target_position(symbol=api.futures_code, qty=2, side=EPositionSide.SHORT)
    else:
        if factor[0] < 0.97:
            print(api.context.time.trade_now.date, "target position 2")  # 非必要： 输出日志
            api.target_position(symbol=api.futures_code, qty=2, side=EPositionSide.SHORT)  # 买入 symbol = 选中标的开空，直至持仓数达2手


def on_timer(api):
    """
    定时器事件，模拟盘响应；回测时可忽略
    :param api:
    """


def on_order_update(api, order):
    """
    订单更新事件
    :param api:
    :param order:
    """

def on_terminate(api, exit_info):
    """
    策略终止时响应；运行终止时会调用一次
    :param api:
    :param exit_info:终止消息
    """

    log.info("***************onTerminate*********")
    # 输出日志
    api.save_object('exit_info', api.props(exit_info))



```

### MA-双均线策略（股票）
本示例演示双均线策略，股票池为沪深300，实现逻辑为：当5日均线上穿60日均线，买入；当5日均线下穿60日均线，卖出

``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from etasdk import *
import time
from datetime import datetime
from etasdk.utils.time_util import TimeUtil
import numpy as np

'''
#本示例演示双均线策略，股票池为上证50
#策略逻辑：
#1.当5日均线上穿60日均线，买入
#2.当5日均线下穿60日均线，卖出
回测标的：上证50
回测周期：20180702-20181101
撮合周期：日K

'''

long_period = 60
short_period = 5
coefficient=0.06

#初始化
def onInitialize(api):
    #设置计算MACD需要的历史K线，为日K，最多历史上30根
    api.setRequireBars(ETimeSpan.DAY_1,long_period);
    #设置安组回调模式，超时时间为 10S
    api.setGroupMode(10000);
    #设计模拟盘Timer函数的定时回调
    api.setTimerCycle(60000)
    #设置股票池为沪深300
    api.setSymbolPool(instsets=["000016.IDX"])

#盘前运行，必须实现，用来设置当日关注的标的
def onBeforeMarketOpen(api,tradeDate):
    #取出股票池
    symbolPool=api.getSymbolPool()

    # 备选买入清单
    buy_list = []
    # 卖出清单
    sell_list = []

    #遍历股票池，分别获取5和60 的K线
    for i, symbol in enumerate(symbolPool):
        bars = api.getBarsHistory(symbol, ETimeSpan.DAY_1, long_period,priceMode=EPriceMode.FORMER, df=True,fields=['close']);
        if len(bars) == long_period:
            bar_close_long = bars[0:long_period]
            bar_close_short = bars[long_period-short_period:long_period]
            long_mean =  np.mean( np.array(bar_close_long['close']) )# 计算60日均线
            short_mean = np.mean( np.array(bar_close_short['close']) )# 计算5日均线

            #print data , symbol,short_mean,long_mean ,"|",(short_mean - long_mean) ,coefficient* long_mean
            #如果
            buy_flag = True if (short_mean - long_mean) > coefficient* long_mean else False
            if buy_flag:
                buy_list.append(symbol)
            else:
                sell_list.append(symbol)
    # 关注当前的buy_list即可，因为如果有持仓会自动关注
    api.setFocusSymbols(buy_list);
    #获取当前持仓的标的，如果需要卖出，则平仓
    position_symbols = api.getPositionSymbols();

    # 输出当前交易日
    LOG.INFO  ("[%s] buy list = [%s]" ,tradeDate,buy_list)
    LOG.INFO  ("[%s] position_symbols list = [%s]" , tradeDate,position_symbols)

    #如果买入列表的标的没有仓位则买入
    for i, buy_symbol in enumerate(buy_list):
        if buy_symbol not in position_symbols:
            LOG.INFO("[%s] target position long 1000![%s]", tradeDate,buy_symbol);
            api.targetPosition(symbol=buy_symbol, qty=1000)

    #卖出有卖出信号的标的
    for i, pos_symbol in enumerate(position_symbols):
        if pos_symbol in sell_list:
            LOG.INFO("[%s] target position short 0![%s]", tradeDate,pos_symbol);
            api.targetPosition(symbol=pos_symbol, qty=0);

#Bar回调
def onHandleData(api,bar):
    pass;

#定时器，模拟盘响应
def onTimer(api):
    pass;

#策略终止时响应
def onTerminate(api,exitInfo):
    LOG.INFO ("***************onTerminate*********")

```
## 进阶股票策略

### buyDemo-带参数的买入策略（股票）  
本策略是一个简单的下单策略，主要是体现如何调用策略参数 ，方便策略参数较多时无需修改策略代码直接进行合理的参数优化管理。
需要在web平台中策略中的参数进行设置，具体步骤为：策略管理>-参数设置>-添加基础参数，如下图所示：
![](//cdn.upchina.com/uptest/201901/1547119539211_20190110192539_1d8758084721f16a.jpg)

``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#回测周期为20180702-20180719
#撮合周期：日K
from etasdk import *
import time
from datetime import datetime

def onInitialize(api):
    print("onInitialize")
    #获取命令行参数
    api.bar_len = 10
    api.setRequireData(
        symbols=['000001.CS'],
        bars=[ (ETimeSpan.DAY_1, api.bar_len)]
    )
    #获取策略参数
    #需在web平台策略管理>-参数设置>-添加基础参数  
    # 参数名称：capital ； 参数类型 SFT_LONG；  默认值（自行选择）10000000；是否必填（自行选择）非必填 
    api.capital = api.getValueAsDouble(field="capital")

def onBeforeMarketOpen(api, data):
    time = api.timeNow()
    print("time----",data)
    symbolPool = api.getSymbolPool()
    api.setFocusSymbols(symbolPool)

def onBar(api, bar):
    print("onbar: %d"%bar.tradeDate)
    hisBar=api.getBarsHistory(bar.symbol, ETimeSpan.DAY_1, api.bar_len, skipSuspended=1, df=True,priceMode = EPriceMode.FORMER)
    close=hisBar["close"].mean()
    current_price = hisBar["close"][api.bar_len-1]
    cash=api.capital
    #用获取的资金计算可下单数量，取100的整数倍
    qty = int(0.8 * cash / current_price / 100) * 100
    if current_price >close:
        api.targetPosition(bar.symbol, qty)

def onTerminate(api, exit_info):
    LOG.INFO("onTerminate")
    print(exit_info)

```
### multiFactor-多因子选股（股票）
本策略为股票多因子选股，通过Fama-French三因子模型对每只股票进行回归，得到其alpha值, 买入alpha为负的股票
``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from etasdk import *
import numpy as np
import pandas as pd
import math
'''
股票策略：多因子选股
本策略根据Fama-French三因子模型对每只股票进行回归，得到其alpha值, 买入alpha为负的股票。
策略思路：
计算市场收益率,并对个股的账面市值比和市值进行分类,
根据分类得到的组合计算市值加权收益率、SMB和HML. 
对各个股票进行回归(无风险收益率等于0)得到alpha值.
选取alpha值小于0并为最小的10只股票进入标的池
等权买入在标的池的股票并卖出不在标的池的股票
回测数据:000300.IDX的成份股
回测区间：2018-07-01 到2018-10-01
回测周期：日K
'''

#初始化
def onInitialize(api):
    # 数据滑窗
    api.dataWindow = 20
    # 设置开仓的最大资金量
    api.ratio = 0.8
    # 账面市值比的大/中/小分类
    api.BM_BIG = 3.0
    api.BM_MID = 2.0
    api.BM_SMA = 1.0
    # 市值大/小分类
    api.MV_BIG = 2.0
    api.MV_SMA = 1.0
    # 持仓股票
    api.symbols_pool = []
    # 设置股票池，因子以及K线类型
    api.setRequireData(instsets=["000300.IDX"], # 000300成分股
                       fields=['MKT_CAP', "PB"],
                       bars=[(ETimeSpan.DAY_1, api.dataWindow + 10)]
                       )
    #设置按组回调
    api.setGroupMode(timeOutMs=10000,onlyGroup = False);

#盘前运行，用于选股逻辑；必须实现，用来设置当日关注的标的
def onBeforeMarketOpen(api,tradeDate):
    print("on date", tradeDate)
    # 获取股票池
    symbolPool = api.getSymbolPool()

    fundamentalDf = pd.DataFrame(columns=["symbol", 'MKT_CAP', "PB"])
    # 获取基本面数据
    for symbol in symbolPool:
        # 检查股票是否停牌，不交易停牌股票
        if api.isSuspend(symbol, tradeDate):
            continue
        # 获取基本面数据
        fundamentals = api.getFieldsCountDays(symbol=symbol, fields=['MKT_CAP', "PB"], count=api.dataWindow + 1)
        fundamentalDf.loc[len(fundamentalDf)] = {"symbol": symbol, "MKT_CAP": fundamentals.MKT_CAP.values[0],
                                                 "PB": fundamentals.PB.values[0]}

    # 计算账面市值比, PB倒数
    fundamentalDf["PB"] = (fundamentalDf['PB'] ** -1)

    # 计算市值的50%的分位点, 用于分类
    sizeGate = fundamentalDf['MKT_CAP'].quantile(0.50)

    # 计算账面市值比的30%和70%分位点,用于分类
    bm_gate = [fundamentalDf['PB'].quantile(0.30), fundamentalDf['PB'].quantile(0.70)]
    fundamentalDf.index = fundamentalDf.symbol
    x_return = []
    # 计算return
    tickerCloseValues = {}
    for symbol in fundamentalDf.symbol.values:
        price = api.getBarsHistory(symbol, timeSpan=ETimeSpan.DAY_1, count=api.dataWindow + 1, df=True, \
                                   priceMode=EPriceMode.FORMER, skipSuspended=0)
        if len(price) < api.dataWindow:
            continue
        price.fillna(method="ffill")

        stockReturn = price.close.values[-1] / price.close.values[0] - 1
        pb = fundamentalDf['PB'][symbol]
        market_value = fundamentalDf['MKT_CAP'][symbol]
        tickerCloseValues[symbol] = price.close.values[-1]
        # 获取[股票代码. 股票收益率, 账面市值比的分类, 市值的分类, 流通市值]
        if pb < bm_gate[0]:
            if market_value < sizeGate:
                label = [symbol, stockReturn, api.BM_SMA, api.MV_SMA, market_value]
            else:
                label = [symbol, stockReturn, api.BM_SMA, api.MV_BIG, market_value]
        elif pb < bm_gate[1]:
            if market_value < sizeGate:
                label = [symbol, stockReturn, api.BM_MID, api.MV_SMA, market_value]
            else:
                label = [symbol, stockReturn, api.BM_MID, api.MV_BIG, market_value]
        elif market_value < sizeGate:
            label = [symbol, stockReturn, api.BM_BIG, api.MV_SMA, market_value]
        else:
            label = [symbol, stockReturn, api.BM_BIG, api.MV_BIG, market_value]

        if len(x_return) == 0:
            x_return = label
        else:
            x_return = np.vstack([x_return, label])

    stocks = pd.DataFrame(data=x_return, columns=['symbol', 'return', 'BM', 'MKT_CAP', 'mv'])
    stocks.index = stocks.symbol
    columns = ['return', 'BM', 'MKT_CAP', 'mv']
    for column in columns:
        stocks[column] = stocks[column].astype(np.float64)

    # 计算SMB.HML和市场收益率
    # 获取小市值组合的市值加权组合收益率
    smb_s = (market_weighted_return(stocks, api.MV_SMA, api.BM_SMA) +
             market_weighted_return(stocks, api.MV_SMA, api.BM_MID) +
             market_weighted_return(stocks, api.MV_SMA, api.BM_BIG)) / 3

    # 获取大市值组合的市值加权组合收益率
    smb_b = (market_weighted_return(stocks, api.MV_BIG, api.BM_SMA) +
             market_weighted_return(stocks, api.MV_BIG, api.BM_MID) +
             market_weighted_return(stocks, api.MV_BIG, api.BM_BIG)) / 3

    smb = smb_s - smb_b
    # 获取大账面市值比组合的市值加权组合收益率
    hml_b = (market_weighted_return(stocks, api.MV_SMA, 3) +
             market_weighted_return(stocks, api.MV_BIG, api.BM_BIG)) / 2
    # 获取小账面市值比组合的市值加权组合收益率
    hml_s = (market_weighted_return(stocks, api.MV_SMA, api.BM_SMA) +
             market_weighted_return(stocks, api.MV_BIG, api.BM_SMA)) / 2

    hml = hml_b - hml_s
    close = api.getBarsHistory(symbol, timeSpan=ETimeSpan.DAY_1, count=api.dataWindow + 1, df=True, \
                                   priceMode=EPriceMode.FORMER, skipSuspended=1)["close"].values
    market_return = close[-1] / close[0] - 1
    coff_pool = []

    # 对每只股票进行回归获取其alpha值
    for stock in stocks.index:
        if math.isnan(float(market_return)) or math.isnan(float(smb)) or math.isnan(float(hml)) or math.isnan(float(stocks['return'][stock])):
            coff_pool.append(np.NaN)
            continue
        x_value = np.array([[market_return], [smb], [hml], [1.0]])
        y_value = np.array([stocks['return'][stock]])
        # OLS估计系数
        coff = np.linalg.lstsq(x_value.T, y_value)[0][3]
        coff_pool.append(coff)

    # 获取alpha最小并且小于0的10只的股票进行操作(若少于10只则全部买入)
    stocks['alpha'] = coff_pool
    stocks = stocks[stocks.alpha < 0].sort_values(by='alpha').head(10)
    api.symbols_pool = stocks.index.tolist()

    # 设置当天需要交易的股票，如果历史上有过持仓了，则系统会默认自动关注
    api.setFocusSymbols(api.symbols_pool )

    # 根据持仓信息，处理持仓股票。平掉不在信号池中的股票
    symbolpositions = api.getSymbolPositions()
    print("symbol before close", len(symbolpositions))
    for position in symbolpositions:
        if api.isSuspend(position.symbol, api.getCurrTradeDate()):
            continue
        if position.symbol not in api.symbols_pool:
            api.targetPosition(symbol=position.symbol, qty=0, positionSide=EPositionSide.LONG,
                               remark="not in signal pool")

def onHandleData(api, timeExch):
    # 获取账户信息，根据剩余可用现金和持仓信息，计算新开仓量
    ratio_by_index = 0.9  # %90%仓位，可根据大盘走势进行调整
    account = api.getAccount("000001.CS")
    capital = account.totAssets * ratio_by_index - account.marketValue
    # 获取止损和平仓后的持仓
    holdingPos = []
    symbolpositions = api.getSymbolPositions()
    for position in symbolpositions:
        holdingPos.append(position.symbol)
    newSymbols = list(set(api.symbols_pool) - set(holdingPos))
    # 检查是否有新标的需要买入
    if len(newSymbols) == 0:
        return
    # 计算个股capital
    eachTickerCapital = capital / len(newSymbols)
    # 根据个股capital下单
    for symbol in newSymbols:
        # 获取标的当前价格
        price = api.getBarsHistory(symbol, timeSpan=ETimeSpan.DAY_1, count=1, df=True,\
                                   priceMode=EPriceMode.FORMER, skipSuspended=0)
        if len(price) <= 0:
            continue
        # 获取当前标的持仓信息
        symbolPosition = api.getSymbolPosition(symbol)
        targetQty = int(eachTickerCapital / price.close.values[-1])
        deltaQty = targetQty - symbolPosition.posQty
        #下单手数取整
        finalQty = symbolPosition.posQty + int(deltaQty / 100) * 100
        finalQty = max(finalQty, 0)

        api.targetPosition(symbol=symbol, qty=finalQty, positionSide=EPositionSide.LONG, remark="open new")
        print("open  new long postion:", symbol, finalQty)
        LOG.INFO("open  new long postion:" + str(symbol) + " " + str(finalQty))


#策略终止时响应
def onTerminate(api,exitInfo):
    LOG.INFO ("***************onTerminate*********")
    print ("***************onTerminate*********")

# 计算市值加权的收益率,MV为市值的分类,BM为账目市值比的分类
def market_weighted_return(stocksValues, MV_class, BM_class):
    select = stocksValues[(stocksValues.MKT_CAP == MV_class) & (stocksValues.BM == BM_class)]
    marketValue = select['mv'].values
    mvTotal = np.sum(marketValue)
    mvWeighted = [mv / mvTotal for mv in marketValue]
    stock_return = select['return'].values
    # 返回市值加权的收益率的和
    returnTotal = []
    for i in range(len(mvWeighted)):
        returnTotal.append(mvWeighted[i] * stock_return[i])
    return_total = np.sum(returnTotal)
    return return_total
```
### stockTiming-股票择时（股票）
``` Ruby
# coding=utf-8
from __future__ import print_function, absolute_import, unicode_literals, division
import numpy as np
from etasdk import *
from talib import EMA

'''
本策略为股票个股择时，等资金分配仓位。
回测参数设置：
1、回测时间：2014-01-06到2018-11-08；
2、撮合周期：日K；
3、费率：万3
4、滑点：1个tick
5、股票初始资金：100万
6、股票池：上证50
'''

def capital_lots(close, capital,ratio_by_index):
    ## 计算下单股数
    # input:价格、资金和仓位比例（0-100%）
    volume = {}
    capitalSingle = ratio_by_index * capital / len(close)  # 均分后根据指数计算仓位比例
    for symbol in close.keys():
        volume[symbol] =  max(100*int(capitalSingle / (close[symbol] * 100.)),100)  # 不足1手，补齐1手
    return volume

def onInitialize(api):
    ## 设置标的
    api.index = str("000016.IDX") # 以上证50股票池为例
    api.setSymbolPool(instsets=[api.index], symbols=[api.index])
    api.setGroupMode(50000, True)

    ##  全局参数
    api.dataLen = 200  # 每次下载的数据长度
    api.setRequireBars(ETimeSpan.DAY_1, api.dataLen)
    account = api.getAccount(symbol=api.index, market=MARKET_CHINASTOCK)
    api.capital = account.cashAvailable  # 账户初始资金（100万）

    ## 时间窗口参数
    api.windows_SMA        = 5   # 短期移动平均窗口
    api.windows_LMA        = 20  # 长期移动平均窗口
    api.windows_LLMA       = 120  # 更长周期移动平均窗口

    ## 临界值类
    api.stop_method = 1

    ## 行情数据类
    api.tradeday = [] # 交易日
    api.symbolPool = []
    api.price = None # 交易信号

    print("initialize successly.")
    LOG.INFO("initialize successly.")

def onBeforeMarketOpen(api, trade_date):
    print("-------------------")
    print("tradingday:", trade_date)
    LOG.INFO("tradingday:%s", trade_date)

    api.tradeday.append(trade_date)
    api.symbolPool = api.getSymbolPool()

    ## 止损平仓
    stop_loss(api)

    ## 1、择时与选股
    api.price = timeAlgorithm(api) # 修改择时算法，返回选股的价格（字典形式）
    print("trade symbol:", api.price.keys())

    # 设置当天需要交易的股票，如果历史上有过持仓了，则系统会默认自动关注
    api.setFocusSymbols(api.price.keys())

    # 根据持仓信息，处理持仓股票。平掉不在信号池中的股票
    symbolpositions = api.getSymbolPositions()
    print("symbol before close", len(symbolpositions))
    for position in symbolpositions:
        if api.isSuspend(position.symbol, api.getCurrTradeDate()):
            continue
        if position.symbol not in api.price.keys():
            api.targetPosition(symbol=position.symbol, qty=0, positionSide=EPositionSide.LONG,
                               remark="not in signal pool")

def onHandleData(api, timeExch):

    # 当前信号不为空
    if not api.price or len(api.price) == 0:
        return

    # 获取账户信息，根据剩余可用现金和持仓信息，计算新开仓量
    ratio_by_index = 0.9  # %90%仓位，可根据大盘走势进行调整
    account = api.getAccount(symbol=api.index, market=MARKET_CHINASTOCK)
    capital = account.totAssets * ratio_by_index - account.marketValue
    # 获取止损和平仓后的持仓
    holdingPos = []
    symbolpositions = api.getSymbolPositions()
    for position in symbolpositions:
        holdingPos.append(position.symbol)
    newSymbols = list(set(api.price.keys()) - set(holdingPos))
    # 检查是否有新标的需要买入
    if len(newSymbols) == 0:
        return
    # 计算个股capital
    eachTickerCapital = capital / len(newSymbols)
    # 根据个股capital下单
    for symbol in newSymbols:
        # 获取标的当前价格
        price = api.getBarsHistory(symbol, timeSpan=ETimeSpan.DAY_1, count=1, df=True,\
                                   priceMode=EPriceMode.FORMER, skipSuspended=0)
        if len(price) <= 0:
            continue
        # 获取当前标的持仓信息
        symbolPosition = api.getSymbolPosition(symbol)
        targetQty = int(eachTickerCapital / price.close.values[-1])
        deltaQty = targetQty - symbolPosition.posQty
        #下单手数取整
        finalQty = symbolPosition.posQty + int(deltaQty / 100) * 100
        finalQty = max(finalQty, 0)

        api.targetPosition(symbol=symbol, qty=finalQty, positionSide=EPositionSide.LONG, remark="open new")
        print("open  new long postion:", symbol, finalQty)
        LOG.INFO("open  new long postion:" + str(symbol) + " " + str(finalQty))


def onTerminate(api, exit_info):
    pass

def timeAlgorithm(api):
    price = {}
    for symbol in api.symbolPool:
        ## 下载行情数据
        HQData = api.getBarsHistory(symbol, timeSpan=ETimeSpan.DAY_1, count=api.dataLen, df=True, \
                               priceMode=EPriceMode.FORMER, skipSuspended=0)
        # 新股排除
        if len(HQData) <= 120:
            continue

        # 检查标的是否停牌
        if api.isSuspend(symbol, api.getCurrTradeDate()):
            continue

        # 根据模型计算开仓信号
        if calculateSignal(api, HQData) == 1:
            price[symbol] = HQData.close.values[-1]
    return price
def calculateSignal(api,HQData):
    if np.isnan(HQData.close.values[-1]):
        return 0

    close = HQData.close.values
    closeSMA = EMA(close, api.windows_SMA)[-1]
    closeLMA = EMA(close, api.windows_LMA)[-1]

    signal01 = 1 if (closeSMA > closeLMA) else 0

    volume = HQData.volume.values
    volumeMA = EMA(volume[:-1],20)
    closeSMADouble08 = EMA(EMA(close, 8), 8)
    dailyRet = np.log(closeSMADouble08[-1]) - np.log(closeSMADouble08[-2])
    signal = 1 if (volume[-1] > 2.*volumeMA[-1]) and (dailyRet>0) else signal01
    return signal

def stop_loss(api):
    # 移动止损止盈
    positionQty, positionHigh, positionClose = queryPosition(api)
    for ind in positionQty.keys():
        # 检查标的是否停牌
        if api.isSuspend(ind, api.getCurrTradeDate()):
            continue
        hist_data = api.getBarsHistory(ind, timeSpan=ETimeSpan.DAY_1, count=90, df=True,
                                       priceMode=EPriceMode.FORMER)
        if len(hist_data) >= 60:
            close = hist_data.close.values
            if api.stop_method == 1:
                # 止损
                drawdown = np.log(positionHigh[ind] / positionClose[ind])
                if drawdown < 0.10:
                    dropline = positionHigh[ind] * 0.90
                else:
                    dropline = positionHigh[ind] * 0.94
                if (close[-1] <= dropline):
                    api.targetPosition(symbol=ind, qty=0, positionSide=EPositionSide.LONG,remark="stop loss")
                    print(api.tradeday[-1], ",stop loss:", ind)
                # 止盈
                if (((close[-1] / positionClose[ind]) - 1) >= 0.15):
                    api.targetPosition(symbol=ind, qty=0, positionSide=EPositionSide.SHORT)
                    print(api.tradeday[-1], ",stop return", ind)

def queryPosition(api):
    # #  查询持仓
    positionQty = {}
    positionHigh = {}
    positionClose = {}
    symbolpositions = api.getSymbolPositions()
    if not symbolpositions:
        return positionQty,positionHigh,positionClose
    else:
        for ind in symbolpositions:
            positionQty[ind.symbol] = ind.posQty
            positionHigh[ind.symbol] = ind.posHigh
            positionClose[ind.symbol] = ind.posPrice
        return positionQty,positionHigh,positionClose

```
### industryRotation-行业轮动（股票）
``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-


from etasdk import *
import numpy as np
from operator import itemgetter
from collections import OrderedDict

'''
股票策略：行业轮动
本策略每天计算"880008.PLA", "880023.PLA", "880030.PLA", "880035.PLA", "880046.PLA", "880064.PLA"
(化学制品.电子.纺织服装.医药.房地产.国防军工)这几个行业指数过去
20个交易日的收益率,选取收益率最高的指数的成份股中流通市值最大的5只股票作为下期的持仓股票
对不在股票池的股票平仓并等权配置股票池的标的
回测区间：2018-07-01 到2018-10-01
撮合周期：日K
'''
#初始化
def onInitialize(api):
    # 选取行业板块
    # 这里我们选择 化学制品（880008.PLA）， 电子（"880023.PLA"），纺织服装（"880030.PLA"）， 医药（"880035.PLA")，房地产（"880046.PLA"），国防军工（"880064.PLA"）
    api.industries = ["880008.PLA", "880023.PLA", "880030.PLA", "880035.PLA", "880046.PLA", "880064.PLA"]
    api.returnWindow = 20
    api.capitalRatio = 0.8
    api.numberOfChosenTickers = 5
    api.chosenTickers = []

    # 设置股票池，因子以及K线类型
    api.setRequireData(instsets=api.industries,
                       symbols=api.industries, # 单独订阅行情数据
                       fields=['MKT_CAP'],
                       bars=[(ETimeSpan.DAY_1, api.returnWindow + 10)]
                       )
    #设置按组回调
    api.setGroupMode(timeOutMs=10000,onlyGroup = False);

#盘前运行，用于选股逻辑；必须实现，用来设置当日关注的标的
def onBeforeMarketOpen(api,tradeDate):
    print("on date", tradeDate)
    # 计算各个行业指数的return
    return_index = []

    for industry in api.industries:
        bars = api.getBarsHistory(industry, timeSpan=ETimeSpan.DAY_1, count=api.returnWindow + 1, df=True, priceMode=EPriceMode.FORMER, skipSuspended=0)
        bars.fillna(method="ffill")
        if len(bars) < api.returnWindow:
            continue
        return_index.append(bars.close.values[-1] / bars.close.values[0] - 1)

    # 找到return最高的行业
    chosenIndustry = api.industries[np.argmax(return_index)]
    print("chosenIndustry", chosenIndustry)

    # 获取行业成分股
    industrySymbols = api.getConstituentSymbols(chosenIndustry)

    # 获取股票的市值
    symbolMarketCaps = {}
    for symbol in industrySymbols:
        # 检查股票是否停牌，不交易停牌股票
        if api.isSuspend(symbol, tradeDate):
            continue
        # 获取单个标的的市值
        marketCapital = api.getFieldsCountDays(symbol=symbol, fields=["MKT_CAP"], count=1)
        symbolMarketCaps[symbol] = marketCapital.MKT_CAP.values[-1]

    # 按照市值对股票排序
    orderedDict = OrderedDict(sorted(symbolMarketCaps.items(), key=itemgetter(1), reverse=True))
    api.chosenTickers = [item[0] for item in orderedDict.items()][:api.numberOfChosenTickers]
    print("chosenTickers", api.chosenTickers)

    # 设置当天需要交易的股票，如果历史上有过持仓了，则系统会默认自动关注
    api.setFocusSymbols(api.chosenTickers)

    # 根据持仓信息，处理持仓股票。平掉不在信号池中的股票
    symbolpositions = api.getSymbolPositions()
    for position in symbolpositions:
        if api.isSuspend(position.symbol, api.getCurrTradeDate()):
            continue
        if position.symbol not in api.chosenTickers:
            api.targetPosition(symbol=position.symbol, qty=0, positionSide=EPositionSide.LONG,
                               remark="not in signal pool")

def onHandleData(api, timeExch):
    # 获取账户信息，根据剩余可用现金和持仓信息，计算新开仓量
    ratio_by_index = 0.9  # %90%仓位，可根据大盘走势进行调整
    account = api.getAccount(api.chosenTickers[0])
    capital = account.totAssets * ratio_by_index - account.marketValue
    # 获取止损和平仓后的持仓
    holdingPos = []
    symbolpositions = api.getSymbolPositions()
    for position in symbolpositions:
        holdingPos.append(position.symbol)
    newSymbols = list(set(api.chosenTickers) - set(holdingPos))
    # 检查是否有新标的需要买入
    if len(newSymbols) == 0:
        return
    # 计算个股capital
    eachTickerCapital = capital / len(newSymbols)
    # 根据个股capital下单
    for symbol in newSymbols:
        # 获取标的当前价格
        price = api.getBarsHistory(symbol, timeSpan=ETimeSpan.DAY_1, count=1, df=True, \
                                   priceMode=EPriceMode.FORMER, skipSuspended=0)
        if len(price) <= 0:
            continue
        # 获取当前标的持仓信息
        symbolPosition = api.getSymbolPosition(symbol)
        targetQty = int(eachTickerCapital / price.close.values[-1])
        deltaQty = targetQty - symbolPosition.posQty
        # 下单手数取整
        finalQty = symbolPosition.posQty + int(deltaQty / 100) * 100
        finalQty = max(finalQty, 0)

        api.targetPosition(symbol=symbol, qty=finalQty, positionSide=EPositionSide.LONG, remark="open new")
        print("open  new long postion:", symbol, finalQty)
        LOG.INFO("open  new long postion:" + str(symbol) + " " + str(finalQty))


#策略终止时响应
def onTerminate(api,exitInfo):
    LOG.INFO ("***************onTerminate*********")
    print ("***************onTerminate*********")
```

### index_hedge_alpha-指数对冲（股票+期货）
``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from __future__ import print_function, absolute_import, unicode_literals,division
from etasdk import *

''' 
指数对冲策略
策略基本思想：本策略为股票市场中性策略，即股票多头+股指期货空头，二者在换仓日保持市值相等。
策略交易频率：每月换仓一次
交易或订阅标的：沪深300指数的成分股和沪深300指数期货，从沪深300指数成分股中按照动量和估值选出30只较强的股票。
回测时间：2016-12-01 到2017-12-29
撮合频率：日K
初始资金：A股200万，期货200万
'''
def onInitialize(api):
    #设置股票池
    api.setSymbolPool(instsets=["IF.PRD","000300.IDX"])
    # 设置数据下载的超时时间
    api.setGroupMode(500, True)
    # 设置行情数据缓存条数
    api.data_len = 21
    api.setRequireBars(ETimeSpan.DAY_1,api.data_len)
    # 设置缓存日频因子数据
    api.setRequireFields(fields=["PE"])

    api.stock_num = 30
    api.trade_day_now = 0
    api.trade_day_last = 0
    api.lots_stock = {}
    api.futures_code = []
    api.lots_futures = 0

def onBeforeMarketOpen(api,trade_date):
    print ("date：",trade_date)
    # 订阅的标的代码
    symbol_subscribe = api.getSymbolPool()
    data_code   = [ind01 for ind01 in symbol_subscribe if "CS" in ind01]
    api.futures_code =  api.getContinuousSymbol("IFZ0.CF", trade_date)

    data_code.append(str(api.futures_code))
    # 关注相关股票和股指期货
    # api.setFocusSymbols(data_code)
    # 当前股指期货主力合约代码
    api.futures_code =  api.getContinuousSymbol("IFZ0.CF", trade_date)

    # 获取当前交易日
    api.trade_day_now  = api.getCurrTradeDate()
    # 获取上一交易日
    api.trade_day_last = api.getPrevTradeDate(trade_date)

    # 如果当前是每月的第一个交易日即选股调仓并执行对冲
    # if str(api.trade_day_now)[4:6] == str(api.trade_day_last)[4:6]: return
    # 获取股票的PE指标数据
    findata = api.getFieldsOneDay(data_code[:-1], ["PE"],api.trade_day_last, df=True)
    findata.index = findata["symbol"]
    # 获取历史行情数据
    close = {}
    for ind03 in data_code:
        bars = api.getBarsHistory(ind03, ETimeSpan.DAY_1, count=api.data_len, priceMode=EPriceMode.FORMER, df=True)
        if (len(bars) >= api.data_len) and (bars.ix[api.data_len-1,"isSuspended"] == 0):
            close[ind03] = bars.ix[api.data_len - 1, "close"]
            if ("CS" in ind03):
                findata.ix[ind03,"momentum01"] = (bars.ix[api.data_len-1,"close"]/bars.ix[0,"close"])-1
    # 先按照估值PE和动量排序选出前30只股票
    findata[["ranks_pe","rank_mom"]] = findata.rank()[["PE","momentum01"]]
    findata["score"]  = findata["rank_mom"]-findata["ranks_pe"]
    findata_sorted = findata.sort_values("score",ascending=False)[:api.stock_num]
    symbol_stock = list(findata_sorted.index.values)
    symbol_stock.append(api.futures_code)
    # 关注相关股票和股指期货
    api.setFocusSymbols(symbol_stock)

    symbolpositions = api.getSymbolPositions()

    # 每天判断是否需要换仓

    for ind in symbolpositions:
        if ind.symbol not in symbol_stock:
            if ind.positionSide == EPositionSide.LONG:  # 股票不在关注的标的中，卖出
                api.targetPosition(symbol=ind.symbol, qty=0, positionSide=EPositionSide.LONG)
            elif ind.positionSide == EPositionSide.SHORT:  # 期货在换月时刻，近月合约平仓
                api.targetPosition(symbol=ind.symbol, qty=0, positionSide=EPositionSide.SHORT)
                api.targetPosition(symbol=api.futures_code, qty=ind.posQty, positionSide=EPositionSide.SHORT)
                print("Shifting positions for months，close market positions：", ind.symbol)

    # 平均分配每只股票资金，计算每只股票仓位
    api.lots_stock = {}
    account_stock   = api.getAccount(data_code[0])
    capital = account_stock.cashAvailable + account_stock.marketValue
    for ind05 in close.keys():
        if ("CS" in ind05) and (ind05 in symbol_stock):
            api.lots_stock[ind05] = int(capital*0.70/close[ind05]/api.stock_num/100.0)*100
        elif ("CF" in ind05) :
            api.lots_futures = int(capital*0.70/close[ind05]/300.0)

    # 1、获取当前仓位换仓
    for ind04 in symbolpositions:
        symbol_last = ind04.symbol
        # 不在标的的股票将卖出
        if ("CS" in symbol_last) and (symbol_last not in symbol_stock):
            api.targetPosition(symbol=symbol_last, qty=0)
            print("not in symbolPool，close market positions：", symbol_last)

def onHandleData(api, timeExch):
    # 如果当前是每月的第一个交易日即选股调仓并执行对冲
    if str(api.trade_day_now)[4:6] == str(api.trade_day_last)[4:6]:
        return
        # 2、买入标的池股票
    for ind06 in api.lots_stock.keys():
        # 如果股票有分红送股的情况下，增加仓位也需要时 100 的整数倍
        currentShares = api.getSymbolPosition(ind06).posQty
        addShares = ((api.lots_stock[ind06] - currentShares + 50) // 100) * 100
        totalShares = currentShares + addShares
        api.targetPosition(symbol=ind06, qty=totalShares)
        print("open market positions：", ind06, totalShares)
    # 开空股指期货对冲
    api.targetPosition(symbol=api.futures_code, qty=api.lots_futures, positionSide=EPositionSide.SHORT)
    print("open market short positions：", api.futures_code)

def onTerminate(api,exitInfo):
    print("onTerminate")
```

## 精通股票策略
### machineLearning-机器学习（股票）
``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from __future__ import print_function, absolute_import
from etasdk import *
import numpy as np
from datetime import datetime
import sys
try:
    from sklearn import svm
except:
    print('please install scikit-learn and numpy with mkl')
    sys.exit(-1)

'''
股票策略：机器学习
本策略选取了七个特征变量组成了滑动窗口长度为20天的训练集, 训练了一个二分类的支持向量机模型来预测股票的上涨和下跌.
若每个星期一没有仓位则计算标的股票近20个交易日的特征变量进行预测,并在预测结果为上涨的时候购买标的.
若已经持有仓位则在盈利大于10%的时候止盈,在星期五损失大于2%的时候止损.
特征为:1.收盘价/均值2.现量/均量3.最高价/均价4.最低价/均价5.现量6.区间收益率7.区间标准差
训练数据为:600036.CS招商银行,时间从20170507到20171107
回测时间为:20171108-20181101
撮合周期：日K
'''

#初始化
def onInitialize(api):
    # 数据滑窗
    # 回测时间20171108-20181101
    api.train_start_date = 20170507
    api.train_end_date = 20171107
    api.trainFinished = False
    api.symbol = "600036.CS" # 订阅招商银行股票
    api.ratio = 0.8
    api.dataWindow = 20
    api.clf = None
    # 设置股票池，因子以及K线类型
    api.setRequireData(symbols=[api.symbol],
                       bars=[(ETimeSpan.DAY_1, 1000)])
    #设置按组回调
    api.setGroupMode(timeOutMs=10000, onlyGroup = False);

# train
def trainHistoryData(api):
    # 获取目标股票的daily历史行情
    recent_data = api.getBarsHistory(api.symbol, timeSpan=ETimeSpan.DAY_1, count=1000, df=True, \
                                   priceMode=EPriceMode.FORMER, skipSuspended=0)
    recent_data.fillna(method="ffill")
    # 获取目标股票的训练数据集
    recent_data = recent_data[(recent_data["tradeDate"] >= api.train_start_date) & (recent_data["tradeDate"] <= api.train_end_date)]
    days_value = recent_data['tradeDate'].values
    days_close = recent_data['close'].values
    days = []
    # 获取行情日期列表
    print('prepare training data for SVM', api.train_start_date, " - ", api.train_end_date)
    for i in range(len(days_value)):
        days.append(days_value[i])

    x_all = []
    y_all = []
    for index in range(api.dataWindow, (len(days) - 5)):
        # 计算一个月共20个交易日相关数据
        start_day = days[index - api.dataWindow]
        end_day = days[index]
        data = recent_data[(recent_data["tradeDate"] >= start_day) &(recent_data["tradeDate"] <= end_day)]
        close = data['close'].values
        max_x = data['high'].values
        min_n = data['low'].values
        amount = data['totalVolume'].values
        volume = []
        for i in range(len(close)):
            volume_temp = amount[i] / close[i]
            volume.append(volume_temp)

        close_mean = close[-1] / np.mean(close)  # 收盘价/均值
        volume_mean = volume[-1] / np.mean(volume)  # 现量/均量
        max_mean = max_x[-1] / np.mean(max_x)  # 最高价/均价
        min_mean = min_n[-1] / np.mean(min_n)  # 最低价/均价
        vol = volume[-1]  # 现量
        return_now = close[-1] / close[0]  # 区间收益率
        std = np.std(np.array(close), axis=0)  # 区间标准差

        # 将计算出的指标添加到训练集X
        # features用于存放因子
        features = [close_mean, volume_mean, max_mean, min_mean, vol, return_now, std]
        x_all.append(features)

    # 准备算法需要用到的数据
    for i in range(len(days_close) - api.dataWindow - 5):
        if days_close[i + api.dataWindow + 5] > days_close[i + api.dataWindow]:
            label = 1
        else:
            label = 0
        y_all.append(label)

    x_train = x_all[: -1]
    y_train = y_all[: -1]
    # 训练SVM
    api.clf = svm.SVC(C=1.0, kernel=str('rbf'), degree=3, gamma=str('auto'), coef0=0.0, shrinking=True, probability=False,
                          tol=0.001, cache_size=400, verbose=False, max_iter=-1,
                          decision_function_shape=str('ovr'), random_state=None)
    print(x_train, y_train)
    api.clf.fit(x_train, y_train)
    #print('finished training!')

#盘前运行，用于选股逻辑；必须实现，用来设置当日关注的标的
def onBeforeMarketOpen(api,tradeDate):
    print("on date", tradeDate)
    # 设置当天需要交易的股票，如果历史上有过持仓了，则系统会默认自动关注
    api.setFocusSymbols(api.symbol)
    # firstly time to run, need to train the system
    if not api.trainFinished:
        trainHistoryData(api)
        api.trainFinished = True

    # 当前工作日
    weekday = datetime.strptime(str(tradeDate), '%Y%m%d').isoweekday()
    # 获取持仓
    symbolposition = api.getSymbolPosition(api.symbol)

    # 获取预测用的历史数据
    data = api.getBarsHistory(api.symbol, timeSpan=ETimeSpan.DAY_1, count=api.dataWindow, df=True, \
                              priceMode=EPriceMode.FORMER, skipSuspended=0)
    data.fillna(method="ffill")
    # 如果是新的星期一且没有仓位则开始预测
    if not symbolposition or symbolposition.posQty == 0 and weekday == 1:
        close = data['close'].values
        train_max_x = data['high'].values
        train_min_n = data['low'].values
        train_amount = data['totalVolume'].values
        volume = []
        for i in range(len(close)):
            volume_temp = train_amount[i] / close[i]
            volume.append(volume_temp)

        close_mean = close[-1] / np.mean(close)
        volume_mean = volume[-1] / np.mean(volume)
        max_mean = train_max_x[-1] / np.mean(train_max_x)
        min_mean = train_min_n[-1] / np.mean(train_min_n)
        vol = volume[-1]
        return_now = close[-1] / close[0]
        std = np.std(np.array(close), axis=0)

        # 得到本次输入模型的因子
        features = [close_mean, volume_mean, max_mean, min_mean, vol, return_now, std]
        features = np.array(features).reshape(1, -1)
        if api.clf:
            prediction = api.clf.predict(features)[0]
        else:
            print("[ERR]: train model is None")

        # 获取账户信息
        account = api.getAccount(api.symbol)
        totAssets = account.totAssets
        eachTickerCapital = totAssets * api.ratio
        # 若预测值为上涨则开仓
        if prediction == 1:
            # 获取昨收盘价
            price = close[-1]
            # 开仓
            qty = int(eachTickerCapital / price / 100) * 100
            api.targetPosition(symbol=api.symbol, qty=qty)
            print(api.symbol, 'open positions for qty of', qty)

    # 当涨幅大于10%,平掉所有仓位止盈
    elif symbolposition and data.close.values[-1] / symbolposition.posPrice >= 1.10:
        if symbolposition.posQty != 0:
            api.targetPosition(symbol=api.symbol, qty=0)
            print('stop win, close positions for', api.symbol)

    # 当时间为周五并且跌幅大于2%时,平掉所有仓位止损
    elif symbolposition and data.close.values[-1] / symbolposition.posPrice < 1.02 and weekday == 5:
        if symbolposition.posQty != 0:
            api.targetPosition(symbol=api.symbol, qty=0)
            print('stop loss, close positions for', api.symbol)

#策略终止时响应
def onTerminate(api,exitInfo):
    LOG.INFO ("***************onTerminate*********")
    print ("***************onTerminate*********")

```
### intradayStockTrade-日内回转交易（股票）
``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from __future__ import print_function, absolute_import, unicode_literals,division
from etasdk import *
import numpy as np
from pandas import DataFrame
import sys
try:
    import talib
except:
    print('please install TA-Lib')
    sys.exit(-1)

'''
本策略为股票日内回转交易。
1、首先买入600000.CS股票10000股底仓；
2、根据分钟数据计算MACD(12,26,9)：
在MACD>0的时候买入100股;在MACD<0的时候卖出100股
3、每日操作的股票数不超过原有仓位,并于收盘前把仓位调整至开盘前的仓位
撮合周期为:600000.CS分钟数据
回测时间为:2018-09-03 到2018-10-01
其他参数：按系统默认设置
'''

def onInitialize(api):
    ## 全局参数设置
    print("SDK version:",GlobalConfig.getVersion())
    # 设置标的股票
    api.symbol = '600000.CS'
    # 订阅标的
    api.setSymbolPool(instsets=[], symbols=[api.symbol])
    api.setGroupMode(1000, False)
    # 设置行情数据缓存条数
    api.init_data_len = 120  # 数据长度
    api.setRequireBars(ETimeSpan.MIN_1, api.init_data_len)

    # 底仓
    api.total = 10000
    # 用于判定第一个仓位是否成功开仓
    api.first = 0
    # 日内回转每次交易100股
    api.trade_n = 100
    # 交易日
    api.tradeDate = [0,0]
    # 每日仓位
    api.turnaround = [0, 0]
    # 用于判断是否触发了回转逻辑的计时
    api.ending = 0
    # 当日可平仓数量
    api.availableQty = 0

def onBeforeMarketOpen(api,trade_date):
    print("tradingday: ",trade_date)
    # 每日开盘前订阅行情
    api.setFocusSymbols(api.symbol)
    print("Focus symbols: %s" %(api.symbol))
    # 查询当日可平仓数量
    api.availableQty = api.getSymbolPosition(symbol=api.symbol, positionSide=EPositionSide.LONG)
def onBar(api, bar):
    if api.first == 0:
        # 购买10000股'600000.CS'股票
        api.targetPosition(symbol=bar.symbol, qty=api.total, positionSide=EPositionSide.LONG, remark="open new position")
        print("open in new long postion by market order:", bar.symbol)
        api.first = 1.
        api.tradeDate[-1] = bar.tradeDate
        # 每天的仓位操作
        api.turnaround = [0, 0]
        return

    # 更新最新的日期
    api.tradeDate[0] = bar.tradeDate
    # 若为新的一天,获取可用于回转的昨仓
    if api.tradeDate[0] != api.tradeDate[-1]:
        api.ending = 0
        api.turnaround = [0, 0]
    if api.ending == 1:
        return

    # 若有可用的昨仓则操作
    if api.total > 0 and api.availableQty.availableQty > 0 :
        # 获取时间序列数据
        recent_data = api.getBarsHistory(api.symbol, timeSpan=ETimeSpan.MIN_1, count=api.init_data_len, df=True,priceMode=EPriceMode.FORMER,skipSuspended=0)
        # 计算MACD线
        macd = talib.MACD(recent_data['close'].values)[0][-1]
        # 根据MACD>0则开仓,小于0则平仓
        if macd > 0:
            # 多空单向操作都不能超过昨仓位,否则最后无法调回原仓位
            if api.turnaround[0] + api.trade_n < api.total:
                # 计算累计仓位
                api.turnaround[0] += api.trade_n
                volume = api.total+api.turnaround[0]-api.turnaround[1]
                api.targetPosition(symbol=bar.symbol, qty=volume, positionSide=EPositionSide.LONG,
                                   remark="open new position")
                print("open in new long postion by market order:%s : %i" %(bar.symbol,api.trade_n))
        elif macd < 0:
            if api.turnaround[1] + api.trade_n < api.total:
                api.turnaround[1] += api.trade_n
                volume = api.total + api.turnaround[0] - api.turnaround[1]
                api.targetPosition(symbol=bar.symbol, qty=volume, positionSide=EPositionSide.LONG,
                                   remark="close position")
                print("close long postion by market order:%s : %i" %(bar.symbol, api.trade_n))
        # 临近收盘时若仓位数不等于昨仓则回复所有仓位
        if int(bar.timeStr[9:13]) >= 1455:
            print("recover position before close.")
            symbolposition = api.getSymbolPosition (symbol=api.symbol,positionSide=EPositionSide.LONG)
            if symbolposition.posQty != api.total:
                api.targetPosition(symbol=bar.symbol, qty=api.total, positionSide=EPositionSide.LONG,
                                   remark="target position")
                print("To target position:%d" %(api.total))
                api.ending = 1
        # 更新过去的日期数据
        api.tradeDate[-1] = api.tradeDate[0]


```

## 高阶经典期货策略

### net_trade-网格交易（期货）
本策略为经典的期货日内策略（网格交易），设置不同的临界线，并分配不同的资金仓位

``` Ruby
# coding=utf-8
from __future__ import print_function, absolute_import, unicode_literals,division
from etasdk import *
import numpy as np
import pandas as pd
'''
策略基本思想：本策略为经典的期货日内策略（网格交易），设置不同的临界线，并分配不同的资金仓位。
策略交易频率：1分钟
交易或订阅标的：rb1801
回测时间：2017-12-01 到2017-12-29
'''
def onInitialize(api):
    #设置股票池
    api.setSymbolPool(symbols=["rb1801.CF"])
    # 设置行情数据缓存条数
    api.data_len = 100
    api.setRequireBars(ETimeSpan.MIN_1,api.data_len)
    api.trade_product = str("rb1801.CF")
    # 查询初始资金
    account = api.getAccount(api.trade_product)
    api.capital = account.cashAvailable
    # 交易参数设定
    api.k1 = [-40.0,-3.0,-2.0,2.0,3.0,40.0]
    api.weight = [0.5,0.3, 0.0, 0.3, 0.5]
    api.trade_peroid = 60 # 每小时更新一次网格临界线
    api.minute_num = 0
    api.level  = 3.0
    api.volume = []
    api.band   = []
def onBeforeMarketOpen(api, trade_date):
    print(trade_date)
    api.tradeday = trade_date
    ##  订阅行情
    api.setFocusSymbols( api.trade_product)
def onBar(api,data):
    if (api.minute_num % api.trade_peroid) == 0:
        # 获取过去300个数据计算临界线
        data01 = api.getBarsHistory(data.symbol, timeSpan=ETimeSpan.MIN_1, count=api.data_len, df=False)
        close = []
        for ind01 in data01:
            close.append(ind01.close)
        api.band = np.mean(close)+np.array(api.k1)*np.std(close)
    # 计算网格状态
    grid = pd.cut([data.close],api.band,labels=[0,1,2,3,4])[0]
    print(grid)
    # 更新计算交易量
    api.volume = []
    for weight in api.weight:
        api.volume.append(lots(api, data, weight))
    api.minute_num += 1
    # 查询持仓
    position_long = api.getSymbolPosition(symbol=data.symbol, positionSide=EPositionSide.LONG)
    position_short = api.getSymbolPosition(symbol=data.symbol, positionSide=EPositionSide.SHORT)
    if not position_short.posQty and not position_long.posQty and grid != 2:
        if grid >= 3:
            api.targetPosition(symbol=data.symbol, qty=api.volume[grid], positionSide=EPositionSide.LONG)
            print("open market long positions 1：", data.symbol)
        if grid <= 1:
            api.targetPosition(symbol=data.symbol, qty=api.volume[grid], positionSide=EPositionSide.SHORT)
            print("open market short positions 1：", data.symbol)
    elif position_long.posQty:
        if grid >= 3:
            api.targetPosition(symbol=data.symbol, qty=api.volume[grid], positionSide=EPositionSide.LONG)
            print("open market long positions 2：", data.symbol)
        elif grid == 2:
            api.targetPosition(symbol=data.symbol, qty=0, positionSide=EPositionSide.LONG)
            print("close market long positions：", data.symbol)
        elif grid <= 1:
            api.targetPosition(symbol=data.symbol, qty=0, positionSide=EPositionSide.LONG)
            api.targetPosition(symbol=data.symbol, qty=api.volume[grid], positionSide=EPositionSide.SHORT)
            print("close market long and open market short positions：", data.symbol)
    elif position_short.posQty:
        if grid <= 1:
            api.targetPosition(symbol=data.symbol, qty=api.volume[grid], positionSide=EPositionSide.SHORT)
            print("open market short positions 2：", data.symbol)
        elif grid == 2:
            api.targetPosition(symbol=data.symbol, qty=0, positionSide=EPositionSide.SHORT)
            print("close market short positions：", data.symbol)
        elif grid >= 3:
            api.targetPosition(symbol=data.symbol, qty=0, positionSide=EPositionSide.SHORT)
            api.targetPosition(symbol=data.symbol, qty=api.volume[grid], positionSide=EPositionSide.LONG)
            print("close market short and open market long positions：", data.symbol)


def lots(api, data,weight):
    # 获取账户资金计算下单手数
    refdata = api.getRefData(data.symbol)
    multiper = refdata.valuePerUnit
    return (int(api.capital * api.level*weight / data.close / multiper))
```
### turtleTradingRule-海龟交易法（期货）
``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from __future__ import print_function, absolute_import, unicode_literals

import sys
import numpy as np

try:
    import talib
except:
    print('please Install TA-Lib')
    sys.exit(-1)
from etasdk import *

'''
期货策略：海龟交易法
本策略通过计算FG801.CF和rb1801.CF的ATR.唐奇安通道和MA线,
-- 当价格上穿唐奇安通道且短MA在长MA上方时开多仓;
-- 当价格下穿唐奇安通道且短MA在长MA下方时开空仓(8手)
若有多仓则在价格跌破唐奇安平仓通道下轨的时候全平仓位,否则根据跌破
持仓均价 - x(x=0.5,1,1.5,2)倍ATR把仓位平至6/4/2/0手
若有空仓则在价格涨破唐奇安平仓通道上轨的时候全平仓位,否则根据涨破
持仓均价 + x(x=0.5,1,1.5,2)倍ATR把仓位平至6/4/2/0手
撮合周期为:1分钟
回测时间为:2017-09-15 到2017-10-01
'''

def onInitialize(api):
    print("analyzer initialize")
    # 设置响应模式
    api.setGroupMode(5000, False)
    # 设置关注的合约标的
    api.tickersCode = ["FG1801.CF", "rb1801.CF"]

    # api.parameter 分别为唐奇安开仓通道.唐奇安平仓通道.短ma.长ma.ATR的参数
    api.parameter = [55, 20, 10, 60, 20]
    api.tar = api.parameter[4]

    api.data_len = 600

    api.setSymbolPool(symbols=api.tickersCode)
    api.setRequireBars(ETimeSpan.MIN_1, api.data_len)

    # 初始资金和数据测试
    account = api.getAccount(symbol=api.tickersCode[0], market=MARKET_CHINAFUTURE)
    api.capital = account.cashAvailable  # 账户初始资金
    api.canTrade = True


def onBeforeMarketOpen(api, tradeDate):
    api.trading_day = tradeDate
    # 订阅行情
    api.setFocusSymbols(api.tickersCode)

    # 获取账户盈亏信息，如果亏损超过初始资金一半，不再开仓
    getStrategyPnL = api.getStrategyPnL()
    cumpnl_raw = api.capital + getStrategyPnL.overallPnL

    if cumpnl_raw < 0.5 * api.capital:
        api.canTrade = False
        print("Losing more than half of the initial funds, closing all positions!")
        symbol_positions = api.getSymbolPositions()
        for symbol_obj in symbol_positions:
            if symbol_obj.positionSide == EPositionSide.SHORT:
                api.targetPosition(symbol=symbol_obj.symbol, qty=0, positionSide=EPositionSide.SHORT)
            elif symbol_obj.positionSide == EPositionSide.LONG:
                api.targetPosition(symbol=symbol_obj.symbol, qty=0, positionSide=EPositionSide.LONG)


def onBar(api, bar):
    if api.canTrade:
        bar_data = api.getBarsHistory(symbol=bar.symbol, timeSpan=ETimeSpan.MIN_1, count=api.data_len,
                                      priceMode=EPriceMode.FORMER, fields=None, df=True)
        close = bar_data["close"].values[-1]
        # 计算ATR
        atr = talib.ATR(bar_data["high"].values, bar_data["low"].values, bar_data["close"].values,
                        timeperiod=api.tar)[-1]

        # 计算唐奇安开仓和平仓通道
        api.don_open = api.parameter[0] + 1
        upper_band = talib.MAX(bar_data['close'].values[:-1], timeperiod=api.don_open)[-1]
        api.don_close = api.parameter[1] + 1
        lower_band = talib.MIN(bar_data['close'].values[:-1], timeperiod=api.don_close)[-1]

        # 若没有仓位则开仓
        position_long = api.getSymbolPosition(symbol=bar.symbol, positionSide=EPositionSide.LONG)
        position_short = api.getSymbolPosition(symbol=bar.symbol, positionSide=EPositionSide.SHORT)

        if not position_long.posQty and not position_short.posQty:
            # 计算长短ma线.DIF
            ma_short = talib.MA(bar_data['close'].values, timeperiod=(api.parameter[2] + 1))[-1]
            ma_long = talib.MA(bar_data['close'].values, timeperiod=(api.parameter[3] + 1))[-1]

            diff = ma_short - ma_long

            # 获取当前价格
            # 上穿唐奇安通道且短ma在长ma上方则开多仓
            if bar.close > upper_band and (diff > 0):
                api.targetPosition(symbol=bar.symbol, qty=8, positionSide=EPositionSide.LONG)
                print(bar.symbol, 'Open 8 Long-shares at market position')
            # 下穿唐奇安通道且短ma在长ma下方则开空仓
            if bar.low < lower_band and (diff < 0):
                api.targetPosition(symbol=bar.symbol, qty=8, positionSide=EPositionSide.SHORT)
                print(bar.symbol, 'Open 8 Short-shares at market position')

        elif position_long.posQty:
            # 价格跌破唐奇安平仓通道全平仓位止损
            if close < lower_band:
                api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.LONG)
                print(bar.symbol, 'Close all Long-positions')
            else:
                # 获取持仓均价
                vwap = position_long.posPrice
                # 根据持仓以来的最高价计算不同的止损价格带
                band = vwap - np.array([2, 1.5, 1, 0.5]) * atr
                # 计算最新应持仓位
                if bar.close <= band[0]:
                    api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.LONG)
                    print(bar.symbol, 'Long target positions to 0 shares')
                elif band[0] < bar.close <= band[1]:
                    api.targetPosition(symbol=bar.symbol, qty=2, positionSide=EPositionSide.LONG)
                    print(bar.symbol, 'Long target positions to 2 shares')
                elif band[1] < bar.close <= band[2]:
                    api.targetPosition(symbol=bar.symbol, qty=4, positionSide=EPositionSide.LONG)
                    print(bar.symbol, 'Long target positions to 4 shares')
                elif band[2] < bar.close <= band[3]:
                    api.targetPosition(symbol=bar.symbol, qty=6, positionSide=EPositionSide.LONG)
                    print(bar.symbol, 'Long target positions to 6 shares')

        elif position_short.posQty:
            # 价格涨破唐奇安平仓通道或价格涨破持仓均价加两倍ATR平空仓
            if close > upper_band:
                api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.SHORT)
                print(bar.symbol, 'Close all Short-positions')
            else:
                # 获取持仓均价
                vwap = position_long.posPrice
                # 根据持仓以来的最高价计算不同的止损价格带
                band = vwap + np.array([2, 1.5, 1, 0.5]) * atr
                # 计算最新应持仓位
                if bar.close >= band[0]:
                    api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.SHORT)
                    print(bar.symbol, 'Short target positions to 0 shares')
                elif band[0] > bar.close >= band[1]:
                    api.targetPosition(symbol=bar.symbol, qty=2, positionSide=EPositionSide.SHORT)
                    print(bar.symbol, 'Short target positions to 2 shares')
                elif band[1] > bar.close >= band[2]:
                    api.targetPosition(symbol=bar.symbol, qty=4, positionSide=EPositionSide.SHORT)
                    print(bar.symbol, 'Short target positions to 4 shares')
                elif band[2] > bar.close >= band[3]:
                    api.targetPosition(symbol=bar.symbol, qty=6, positionSide=EPositionSide.SHORT)
                    print(bar.symbol, 'Short target positions to 6 shares')
    else:
        print("Accumulated losses exceed half of the initial funds, no longer open positions!")
        return

def onTerminate(api, exit_info):
    print("Finished test")
    pass

```

### interCommoditySpread-跨市场套利（期货）
``` Ruby
# coding=utf-8
from __future__ import print_function, absolute_import, unicode_literals
import datetime
from etasdk import *
import numpy as np

'''
期货策略：跨市场套利
本策略首先滚动计算过去30个1min收盘价的均值,然后用均值加减2个标准差得到布林线.
若无仓位,在最新价差上穿上轨时做空价差;下穿下轨时做多价差
若有仓位则在最新价差回归至上下轨水平内时平仓
回测数据为:rb1801和hc1801的1min数据
回测时间为:2017-09-01 到2017-09-29
撮合周期：1分钟
初始资金（期货）：10万
'''


def onInitialize(api):
    # 设置响应模式
    api.setGroupMode(5000, False)
    # 设置bar长度
    api.data_len = 30
    # 进行套利的品种
    api.tickersCode = ['rb1801.CF', 'hc1801.CF']        # "rb1801.CF"---螺纹1801合约
    # 设置行情数据缓存
    api.setRequireData(instsets=['rb.PRD', 'hc.PRD'], symbols=api.tickersCode, fields=[], bars=[(ETimeSpan.MIN_1, 300)])


def onBeforeMarketOpen(api,tradeDate):
    # print(tradeDate)
    api.trading_day = tradeDate
    # 订阅行情
    api.setFocusSymbols(api.tickersCode)

    symbol_positions = api.getSymbolPositions()
    print("symbol_positions:", symbol_positions)

    position_rb_long = api.getSymbolPosition(symbol=api.tickersCode[0], positionSide=EPositionSide.LONG)
    position_rb_short = api.getSymbolPosition(symbol=api.tickersCode[0], positionSide=EPositionSide.SHORT)



def onHandleData(api, timeExch):
    api.trade_date = datetime.datetime.fromtimestamp(timeExch / 1000)

    # 获取两个品种的时间序列
    data_rb = api.getBarsHistory(symbol=api.tickersCode[0], timeSpan=ETimeSpan.MIN_1, count=api.data_len+1,
                                 priceMode=EPriceMode.FORMER, fields=None, df=True)
    close_rb = data_rb["close"].values

    data_hc = api.getBarsHistory(symbol=api.tickersCode[1], timeSpan=ETimeSpan.MIN_1, count=api.data_len+1,
                                 priceMode=EPriceMode.FORMER, fields=None, df=True)
    close_hc = data_hc["close"].values

    # 计算价差
    spread = close_rb[:-1] - close_hc[:-1]
    # 计算布林带的上下轨
    up = np.mean(spread) + 2 * np.std(spread)
    down = np.mean(spread) - 2 * np.std(spread)
    # 计算最新价差
    spread_now = close_rb[-1] - close_hc[-1]

    # 无交易时若价差上(下)穿布林带上(下)轨则做空(多)价差
    position_rb_long = api.getSymbolPosition(symbol=api.tickersCode[0], positionSide=EPositionSide.LONG)
    position_rb_short = api.getSymbolPosition(symbol=api.tickersCode[0], positionSide=EPositionSide.SHORT)

    print("I'm here!")
    print(position_rb_long)

    if not position_rb_long and not position_rb_short:
        if spread_now > up:
            api.targetPosition(symbol=api.tickersCode[0], qty=1, positionSide=EPositionSide.SHORT)
            LOG.INFO("Open short at market price:%s", api.tickersCode[0])
            api.targetPosition(symbol=api.tickersCode[1], qty=1, positionSide=EPositionSide.LONG)
            LOG.INFO("Open long at market price:%s", api.tickersCode[1])

        if spread_now < down:
            api.targetPosition(symbol=api.tickersCode[0], qty=1, positionSide=EPositionSide.LONG)
            LOG.INFO("Open long at market price:%s", api.tickersCode[0])
            api.targetPosition(symbol=api.tickersCode[1], qty=1, positionSide=EPositionSide.SHORT)
            LOG.INFO("Open short at market price:%s", api.tickersCode[1])

    # 价差回归时平仓
    elif position_rb_short:
        if spread_now <= up:
            api.targetPosition(symbol=api.tickersCode[0], qty=0, positionSide=EPositionSide.SHORT)
            api.targetPosition(symbol=api.tickersCode[1], qty=0, positionSide=EPositionSide.LONG)
            LOG.INFO("Clear all positions")

        # 跌破下轨反向开仓
        if spread_now < down:
            api.targetPosition(symbol=api.tickersCode[0], qty=1, positionSide=EPositionSide.LONG)
            LOG.INFO("Open long at market price:%s", api.tickersCode[0])
            api.targetPosition(symbol=api.tickersCode[1], qty=1, positionSide=EPositionSide.SHORT)
            LOG.INFO("Open short at market price:%s", api.tickersCode[1])

    elif position_rb_long:
        if spread_now >= down:
            api.targetPosition(symbol=api.tickersCode[0], qty=0, positionSide=EPositionSide.SHORT)
            api.targetPosition(symbol=api.tickersCode[1], qty=0, positionSide=EPositionSide.LONG)
            LOG.INFO("Clear all positions")
        # 突破上轨开多
        if spread_now > up:
            api.targetPosition(symbol=api.tickersCode[0], qty=1, positionSide=EPositionSide.SHORT)
            LOG.INFO("Open short at market price:%s", api.tickersCode[0])
            api.targetPosition(symbol=api.tickersCode[1], qty=1, positionSide=EPositionSide.LONG)
            LOG.INFO("Open long at market price:%s", api.tickersCode[1])


def onTerminate(api,exitInfo):
    print("Finished test")
```

### calendarArbitrage-跨期套利（期货）
``` Ruby
# coding=utf-8
from __future__ import print_function, absolute_import, unicode_literals
import sys
import datetime
import numpy as np
from etasdk import *
try:
    import statsmodels.tsa.stattools as ts
except:
    print('please install statsmodels')
    sys.exit(-1)

'''
期货策略：跨期套利
本策略根据EG两步法(1.序列同阶单整2.OLS残差平稳)判断序列具有协整关系后(若无协整关系则全平仓位不进行操作)
通过计算两个价格序列回归残差的均值和标准差并用均值加减0.9倍标准差得到上下轨
在价差突破上轨的时候做空价差;在价差突破下轨的时候做多价差
若有仓位,在残差回归至上下轨内的时候平仓
回测数据为:rb1801和rb1805的1min数据
回测时间为:2017-09-25 到2017-10-01
撮合周期：1分钟
'''


def onInitialize(api):
    # 设置响应模式
    api.setGroupMode(5000, False)
    # 设置bar长度
    api.data_len = 800
    # 进行套利的品种
    api.tickersCode = ['rb1801.CF', 'rb1805.CF']
    # 设置行情数据缓存
    api.setRequireData(instsets=['rb.PRD'], symbols=api.tickersCode, fields=[], bars=[(ETimeSpan.MIN_1, 900)])


def onBeforeMarketOpen(api, tradeDate):
    # print(tradeDate)
    api.trading_day = tradeDate
    # 订阅行情
    api.setFocusSymbols(api.tickersCode)


def onHandleData(api, timeExch):
    api.trade_date = datetime.datetime.fromtimestamp(timeExch / 1000)

    # 获取两个品种的时间序列
    data_01 = api.getBarsHistory(symbol=api.tickersCode[0], timeSpan=ETimeSpan.MIN_1, count=api.data_len + 1,
                                 priceMode=EPriceMode.FORMER, fields=None, df=True)
    close_01 = data_01["close"].values

    data_02 = api.getBarsHistory(symbol=api.tickersCode[1], timeSpan=ETimeSpan.MIN_1, count=api.data_len + 1,
                                 priceMode=EPriceMode.FORMER, fields=None, df=True)
    close_02 = data_02["close"].values

    # 展示两个价格序列的协整检验的结果
    beta, c, resid, result = cointegration_test(close_01, close_02)

    # 如果返回协整检验不通过的结果则全平仓位等待
    if not result:
        symbol_positions = api.getSymbolPositions()
        if symbol_positions:
            for symbol_obj in symbol_positions:
                if symbol_obj.positionSide == EPositionSide.SHORT:
                    api.targetPosition(symbol=symbol_obj.symbol, qty=0, positionSide=EPositionSide.SHORT)
                    LOG.INFO("close old contract short position:%s", symbol_obj.symbol)
                elif symbol_obj.positionSide == EPositionSide.LONG:
                    api.targetPosition(symbol=symbol_obj.symbol, qty=0, positionSide=EPositionSide.LONG)
                    LOG.INFO("close old contract long position:%s", symbol_obj.symbol)
        return

    # 计算残差的标准差上下轨
    mean = np.mean(resid)
    up = mean + 1.5 * np.std(resid)
    down = mean - 1.5 * np.std(resid)

    # 计算新残差
    resid_new = close_01[-1] - beta * close_02[-1] - c

    # 获取rb1801的多空仓位
    position_01_long = api.getSymbolPosition(symbol=api.tickersCode[0], positionSide=EPositionSide.LONG)
    position_01_short = api.getSymbolPosition(symbol=api.tickersCode[0], positionSide=EPositionSide.SHORT)

    if not position_01_long and not position_01_short:
        if resid_new > up:
            api.targetPosition(symbol=api.tickersCode[0], qty=1, positionSide=EPositionSide.SHORT)
            LOG.INFO("Open short at market price:%s", api.tickersCode[0])
            api.targetPosition(symbol=api.tickersCode[1], qty=1, positionSide=EPositionSide.LONG)
            LOG.INFO("Open long at market price:%s", api.tickersCode[1])

        if resid_new < down:
            api.targetPosition(symbol=api.tickersCode[0], qty=1, positionSide=EPositionSide.LONG)
            LOG.INFO("Open long at market price:%s", api.tickersCode[0])
            api.targetPosition(symbol=api.tickersCode[1], qty=1, positionSide=EPositionSide.SHORT)
            LOG.INFO("Open short at market price:%s", api.tickersCode[1])

    # 价差回归时平仓
    elif position_01_short:
        if resid_new <= up:
            api.targetPosition(symbol=api.tickersCode[0], qty=0, positionSide=EPositionSide.SHORT)
            api.targetPosition(symbol=api.tickersCode[1], qty=0, positionSide=EPositionSide.LONG)
            LOG.INFO("Clear all positions")

        # 跌破下轨反向开仓
        if resid_new < down:
            api.targetPosition(symbol=api.tickersCode[0], qty=1, positionSide=EPositionSide.LONG)
            LOG.INFO("Open long at market price:%s", api.tickersCode[0])
            api.targetPosition(symbol=api.tickersCode[1], qty=1, positionSide=EPositionSide.SHORT)
            LOG.INFO("Open short at market price:%s", api.tickersCode[1])

    elif position_01_long:
        if resid_new >= down:
            api.targetPosition(symbol=api.tickersCode[0], qty=0, positionSide=EPositionSide.SHORT)
            api.targetPosition(symbol=api.tickersCode[1], qty=0, positionSide=EPositionSide.LONG)
            LOG.INFO("Clear all positions")
        # 突破上轨开多
        if resid_new > up:
            api.targetPosition(symbol=api.tickersCode[0], qty=1, positionSide=EPositionSide.SHORT)
            LOG.INFO("Open short at market price:%s", api.tickersCode[0])
            api.targetPosition(symbol=api.tickersCode[1], qty=1, positionSide=EPositionSide.LONG)
            LOG.INFO("Open long at market price:%s", api.tickersCode[1])


def onTerminate(api,exitInfo):
    print("Finished test")


# 协整检验的函数
def cointegration_test(series01, series02):
    urt_rb1801 = ts.adfuller(np.array(series01), 1)[1]
    urt_rb1805 = ts.adfuller(np.array(series02), 1)[1]
    # 同时平稳或不平稳则差分再次检验
    if (urt_rb1801 > 0.1 and urt_rb1805 > 0.1) or (urt_rb1801 < 0.1 and urt_rb1805 < 0.1):
        urt_diff_rb1801 = ts.adfuller(np.diff(np.array(series01)), 1)[1]
        urt_diff_rb1805 = ts.adfuller(np.diff(np.array(series02)), 1)[1]
        # 同时差分平稳进行OLS回归的残差平稳检验
        if urt_diff_rb1801 < 0.1 and urt_diff_rb1805 < 0.1:
            matrix = np.vstack([series02, np.ones(len(series02))]).T
            beta, c = np.linalg.lstsq(matrix, series01)[0]
            resid = series01 - beta * series02 - c
            if ts.adfuller(np.array(resid), 1)[1] > 0.1:
                result = 0.0
            else:
                result = 1.0
            return beta, c, resid, result

        else:
            result = 0.0
            return 0.0, 0.0, 0.0, result

    else:
        result = 0.0
        return 0.0, 0.0, 0.0, result

```

### dualTrust-日内交易（期货）
``` Ruby
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from __future__ import print_function, absolute_import, unicode_literals, division
import numpy as np
import datetime
from etasdk import *
import talib
import heapq
import csv
import pandas as pd
from collections import deque, OrderedDict
import sys

reload(sys)
sys.setdefaultencoding("utf-8")

"""
期货日内交易
Dual-Trust策略概述：
==================
在Dual Thrust 交易系统中，对于震荡区间的定义非常关键，这也是该交易系统的核心。Dual Thrust在Range
的设置上，引入前N 日的四个价位，Range = Max(HH-LC,HC-LL)来描述震荡区间的大小。其中HH 是N 日High
的最高价，LC 是N 日Close 的最低价，HC 是N 日Close 的最高价，LL 是N 日Low 的最低价。这种方法使得一
定时期内的Range 相对稳定，可以适用于日间的趋势跟踪。Dual Thrust 对于多头和空头的触发条件，考虑了非
对称的幅度，做多和做空参考的Range可以选择不同的周期数，也可以通过参数K1 和K2 来确定。

第一步：计算相关参数，得到上轨Buy_line 和下轨Sell_line：
1、N 日High 的最高价HH, N 日Close 的最低价LC;
2、N 日Close 的最高价HC，N 日Low 的最低价LL;
3、Range = Max(HH-LC,HC-LL)；
4、BuyLine = Open + K1*Range；
5、SellLine = Open + K2*Range；

第二步：交易逻辑
1、当价格向上突破上轨时，如果当时持有空仓，则先平仓，再开多仓；如果没有仓位，则直接开多仓；
2、当价格向下突破下轨时，如果当时持有多仓，则先平仓，再开空仓；如果没有仓位，则直接开空仓；

第三步：止损
1、初始止损
2、入场后跟踪止损
3、出现反向信号止损开反向仓位
4、在收盘前1分钟平仓（14:58）
5、每日最多交易次数2次，多空各一次


回测数据为:ruZ0.CF主力合约的的1min数据
回测时间为:2018-10-8 到2018-10-18

"""


def onInitialize(api):
    # 设置响应模式
    api.setGroupMode(5000, False)
    api.focused_contracts = "ruZ0.CF"
    api.threshold = 0.015
    api.percent = 0.035
    api.k = 0.65
    api.focused_prd = str(api.focused_contracts.replace("Z0.CF", ".PRD"))
    api.setRequireData(instsets=api.focused_prd, symbols=api.focused_contracts, fields=[],
                       bars=[(ETimeSpan.DAY_1, 10), (ETimeSpan.MIN_1, 130)])

    # 初始资金和数据测试
    account = api.getAccount(symbol=api.focused_contracts, market=MARKET_CHINAFUTURE)
    api.capital = account.cashAvailable  # 账户初始资金


def onBeforeMarketOpen(api, tradeDate):
    # print(tradeDate)
    api.trading_day = tradeDate
    api.bars_since_today = 0
    api.lots = 1
    api.can_trade = False
    api.long_can_trade = True
    api.short_can_trade = True
    api.code = api.getContinuousSymbol(api.focused_contracts, tradeDate)
    api.history_range, normalize_tr, last_close = get_history_range(api, api.code)

    # 订阅合约
    if normalize_tr > api.threshold:
        api.setFocusSymbols(api.code)
        api.can_trade = True
        multiply = get_multiply(api, api.focused_contracts)
        api.lots = int(1000000 / (last_close * multiply))
    # print("api.can_trade:", api.can_trade)


def onBar(api, bar):
    if api.can_trade:
        time_now = datetime.datetime.fromtimestamp(api.timeNow() / 1000)
        api.bars_since_today += 1
        bars = api.getBarsHistory(symbol=bar.symbol, timeSpan=ETimeSpan.MIN_1, count=api.bars_since_today,
                                  priceMode=EPriceMode.REAL, fields=None, df=True)
        # print("time_now:", time_now)
        today_open = bars['open'].values[0]
        buy_line, sell_line = get_buy_and_sell_lines(today_open=today_open, history_range=api.history_range,
                                                     k1=api.k, k2=api.k)

        position_side = get_position_side(api, bar.symbol)

        if position_side <= 0 and bar.high > buy_line and api.long_can_trade:
            api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.SHORT,
                               remark="Reverse!")
            api.targetPosition(symbol=bar.symbol, qty=api.lots, positionSide=EPositionSide.LONG,
                               remark="Entry LONG!")
            api.long_can_trade = False
        if position_side >= 0 and bar.low < sell_line and api.short_can_trade:
            api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.LONG,
                               remark="Reverse!")
            api.targetPosition(symbol=bar.symbol, qty=api.lots, positionSide=EPositionSide.SHORT,
                               remark="Entry Short!")
            api.short_can_trade = False

        if position_side > 0:
            symbol_position = api.getSymbolPosition(symbol=bar.symbol, positionSide=EPositionSide.LONG)
            if bar.low <= symbol_position.posHigh * (1 - api.percent):
                api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.LONG,
                                   remark="Long Liquid!")
        if position_side < 0:
            symbol_position = api.getSymbolPosition(symbol=bar.symbol, positionSide=EPositionSide.SHORT)
            if bar.high >= symbol_position.posLow * (1 + api.percent):
                api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.SHORT,
                                   remark="Short Liquid!")

        if time_now.hour == 14 and time_now.minute == 59:
            api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.SHORT,
                               remark="Closing the market, Exit Short!")
            api.targetPosition(symbol=bar.symbol, qty=0, positionSide=EPositionSide.LONG,
                               remark="Closing the market, Exit LONG!")


def onTerminate(api, exit_info):
    print("Finished test")


def get_history_range(api, symbol, look_back=5):
    """
    每次开盘之前进行判断：
    获取过去N个交易日的最高价HH，最低价LL，最高收盘价HC，最低收盘价
    """
    str_fields = ['tradeDate', 'open', 'high', 'low', 'close']
    str_index = 'tradeDate'
    bars = api.getBarsHistory(symbol=symbol, timeSpan=ETimeSpan.DAY_1, skipSuspended=1, count=look_back,
                              df=True, priceMode=EPriceMode.REAL, fields=str_fields).set_index(str_index)
    if len(bars) < look_back:
        print("%s lack of data:", symbol)
        history_range = 0
        normalize_tr = 0
        last_close = float("inf")
    else:
        highs = bars['high'].values
        lows = bars['low'].values
        closes = bars['close'].values
        highest_high = np.max(highs)
        lowest_low = np.min(lows)
        highest_close = np.max(closes)
        lowest_close = np.min(closes)
        history_range = np.max([highest_high - lowest_close, highest_close - lowest_low])
        true_range = np.max([highs[-1] - lows[-1], abs(highs[-1] - closes[-2]), abs(closes[-2] - lows[-1])])
        last_close = closes[-1]
        normalize_tr = true_range / last_close

    return history_range, normalize_tr, last_close


def get_buy_and_sell_lines(today_open, history_range, k1=0.45, k2=0.55):
    """计算买入开仓价上轨和卖出开仓价下轨
    BuyLine = today_open + K1*Range
    SellLine = today_open + K2*Range
    """
    buy_line = today_open + k1 * history_range
    sell_line = today_open - k2 * history_range
    return buy_line, sell_line


def get_position_side(api, symbol):
    """
    通过API获取给定合约的持仓方向及数量
    1持仓为多单，-1持仓为空单，等于0表明当前没有持仓
    """
    long_position = int(api.getSymbolPosition(symbol, EPositionSide.LONG).posQty)
    short_position = int(api.getSymbolPosition(symbol, EPositionSide.SHORT).posQty)
    position_side = 0
    if long_position:
        position_side = 1
    if short_position:
        position_side = -1
    return position_side


def get_multiply(api, symbol):
    """
    获取给定合约的合约价值和最小变动价格
    """
    ref_data = api.getRefData(str(symbol))
    multiplier = ref_data.valuePerUnit
    return multiplier

```

