# 常见问题
## 关于优品量化
### 什么是量化投资？
量化投资指的是通过数量化的方式及计算机程序将投资者的投资思想进行变量化，形成一套可以被计算机识别操作思路，利用历史数据加以分析和验证，并由程序来进行交易指令的执行。

### 量化投资具体内容是什么？
量化投资包含甚广，具体内容和手段有以下：
- 阿尔法策略：建仓具有超额收益的股票，设定风险敞口以相应的股指对冲，获取超额收益，避免系统风险。
- 对冲套利：ETF折溢价套利、股指期货期现套利、期货跨期套利、股票的配对交易、期货的跨期套利、跨品种套利、跨市场套利。
- 风险管理：净值波动管理、仓位管理、资产配置管理、组合权重管理。
- 算法交易：为了降低冲击成本的算法自动交易。
- 高频交易：针对盘口机会以极高的频率交易商品。
- CTA：基于综合交易平台的趋势程序化；基于盘口数据报单的程序化交易。

### 量化投资的优势是？
- 纪律性：严格执行投资策略，克服人性的贪婪、恐惧。
- 准确性：准确客观评价交易机会，克服主观情绪偏差，从而盈利。
- 系统性：多层次的量化模型、多角度的观察及海量数据的处理。
- 分散化：在控制风险的条件下，量化投资可以充当分散化投资的工具。
- 及时性：及时快速地跟踪市场变化，不断发现能够提供超额收益的新的统计模型，寻找新的交易机会。


### 什么是优品量化平台？

优品量化是优品财富旗下的一款在线量化交易平台，专注于为专业人士及量化投资爱好者提供的专业、可靠、高效的服务。优品量化平台是最专业的多元资产量化交易平台，支持股票、期货多账户多品种的交易。优品量化以海量金融数据为基础，覆盖量化交易完整的生命周期，提供策略研发、策略回测、模拟交易和实盘交易等功能，为您打造一个专属的策略平台。

### 优品量化提供哪些服务？
优品量化作为最专业的多元资产量化交易平台，为您提供了如下服务：

（1）海量的特色数据和优质的金融大数据

免费提供股票、期货等金融数据，通过专业团队对数据进行深度加工处理，保证数据的精准可靠，为策略运行提供最真实可靠的数据支持。

（2）策略组合功能

零门槛提供python及C++策略编写语言，多策略进行组合，分散风险、降低回撤、增加收益。基于历史行情数据对策略进行回测，一键输出全面的评估结果。

（3）高效的策略回测和模拟交易

接入高频实时行情，一键下单，秒级成交。全面支持股票、期货等多账户多品种的模拟交易，为多类型的策略提供验证支持。

（4）多账户管理

在多用户、多策略、仓位管理等核心逻辑的驱动下，进行策略回测、调试和优化后输出评价报告，产生交易信号，实盘跟踪，进行实时风控。

### 支持哪些交易品种？
优品量化当前支持股票和期货的交易，未来将会提供更多品种的交易服务。

### 可以做套利策略吗？
可以。优品量化平台的一个策略可同时支持多个市场的代码交易，支持套利策略。

### 优品量化适用群体？
专业的投资者和机构用户。

## 策略问题

### 有现成策略提供吗？是否收费？
平台提供期货和股票从入门到专业进阶多类量化策略示例，是免费的。

### 策略归谁所有？是否安全？
作为策略的开发者，策略是您最大的财富，您的所有策略、研究内容的知识产权都归您自己所有。
您策略的编写、运行和存储都是在你的本机上，除了您本人，其他人都无法获取您的策略信息，绝对安全。

### 优品量化有哪些风控策略？
优品量化提供如下风控策略：
-  购买力风控
-  亏损风控
-  仓位风控
-  策略风控

### 策略ID有什么用？如何产生的？
策略ID用于用户策略回测识别某个回测实例，从而运行该回测实例。每次创建回测示例就会生成一个回测ID。

### 支持哪些Python库？
策略在您的本地运行，您用到的Python库可以自行进行安装。

## 回测问题

### 可以支持tick级别回测吗？
可以。优品量化支持日频、分钟频和tick频。

### 按日线、分钟线和tick回测的区别
- 按日回测，每天9点（期货）或9点半（股票）调用一次行情函数
- 按分钟回测，每分钟调用一次行情函数
- 按tick回测，每5秒调用一次行情函数

### 不同市场的tick数据推送频率是否相同？
不同市场的tick数据推送频率不同，优品量化推送tick数据的频率如下：

| 代号 | 交易所名称      | tick推送频率 |
| ---- | --------------- | ------------ |
| CS   | 上交所（SH）    | 5秒          |
| CS   | 深交所（SZ）    | 5秒          |
| CF   | 中金所（CFFEX） |              |
| CF   | 上期所（SHFE）  |              |
| CF   | 郑商所（ZCE）   |              |
| CF   | 大商所（DCE）   |              |

### 回测股票策略时如何处理复权、拆分、分红等变化？
当股票发生分红拆分时，股价会发生变化，往往在股价走势图上出现向下的跳空缺口。在回测部分我们默认提供前复权的数据，用户也可以选择采用不复权的数据，利用我们提供的复权因子数据自行计算除权除息，若使用不复权的数据，则系统会在回测时自动处理好除权除息的问题，计算对应的价格和数量。

## 模拟交易问题

### 模拟交易有什么用处？
模拟交易是一个符合交易规则的高度契合真实环境的交易系统，可以检验策略回测结果可靠性，避免您直接从回测到实盘试错浪费钱。

### 模拟交易和回测有什么区别？
回测成交量不会超过当天的最大成交量（或当天成交量的一部分），模拟交易则是全部成交。这会导致相同日期同一策略的回测结果和模拟交易的结果不一样。另外，按天模拟的交易暂不支持限价单。

### 一个账户能能否同时运行多个模拟交易？
可以。优品量化支持多用户多账号多实例运行。

### 模拟交易中哪些参数可以调整优化？
在模拟交易和实盘交易中，策略模块设置的默认参数不需要修改代码，可以直接在界面做调整，更方便您操作。

## 平台使用问题
### 使用SDK常见错误码问题
1.用户ID不存在

报错信息截图如下：

![](http://cdn.upchina.com/uptest/201811/1543304596835_20181127154316_6e3c29a508fff34a.jpg)

修改步骤：检查配置文件config中的user是否与量化平台右上角的SDK_User一致，若不一致修改之即可。

![](http://cdn.upchina.com/uptest/201811/1543304938826_20181127154858_9a44682a5c87ce16.jpg)

2.TOKEN错误

报错信息截图如下：
![](http://cdn.upchina.com/uptest/201811/1543305265132_20181127155425_82ecd3b7c6e38a11.jpg)

修改步骤：检查配置文件config中的token是否与量化平台右上角的SDK_Token一致，若不一致修改之即可。

![](http://cdn.upchina.com/uptest/201811/1543304938826_20181127154858_9a44682a5c87ce16.jpg)

3.文件不存在

报错信息截图如下：

![](http://cdn.upchina.com/uptest/201811/1543305840208_20181127160400_bec94241e049f1e5.jpg)

修改步骤：检查是否有命名为如图test1的文件，如果不存在此文件，在config对应的位置改成需要运行的文件名即可

4.策略不存在

报错信息截图如下：

![](http://cdn.upchina.com/uptest/201811/1543305936452_20181127160536_bd84c0f81dea26e8.jpg)

修改步骤：检查配置文件config中的”name“配置的是否和实例中模块参数下的策略名一致，若不是修改之即可。

![](http://cdn.upchina.com/uptest/201811/1543306057927_20181127160737_80e52e74becabea7.jpg)

5.策略ID不存在

报错信息截图如下：

![](http://cdn.upchina.com/uptest/201811/1543306409112_20181127161329_bbc192cd1eb38927.jpg)

修改步骤：检查config配置文件中的”instance_id“是否和量化平台界面要运行的实例ID一致，若不一致点击”复制ID“按钮，然后粘贴到config配置文件对应位置即可

![](http://cdn.upchina.com/uptest/201811/1543306552419_20181127161552_b64ebcff2360af31.jpg)

### 更多错误码原因  

| 错误码eno | 英文解释                                                     | 中文解释                                                     |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 0         | succ                                                         | 成功                                                         |
| 5         | The parameter is empty                                       | 参数为空                                                     |
| 6         | Parameter is not a number                                    | 参数不是数字                                                 |
| 7         | Memory allocation failed                                     | 内存分配失败                                                 |
| 8         | datas is not ready!                                          | 云端数据准备中                                               |
| 9         | Unsupported interface                                        | 不支持的接口                                                 |
| 10        | Data compression failed                                      | 数据压缩失败                                                 |
| 11        | Data decompression failed                                    | 数据解压失败                                                 |
| 12        | File does not exist                                          | 文件不存在                                                   |
| 13        | Data read and write exception                                | 数据读写异常                                                 |
| 14        | Invalid argument                                             | 参数无效                                                     |
| 15        | Data does not exist                                          | 数据不存在                                                   |
| 16        | Data has not changed                                         | 数据未发生变化                                               |
| 50        | Not found refdata                                            | 未找到标的REFDATA                                            |
| 51        | Database configuration not found                             | 数据库配置未找到                                             |
| 52        | There is no corresponding Adaptor code                       | 没有对应的Adaptor代码                                        |
| 53        | There is no corresponding RESCONTAINER code                  | 没有对应RESCONTAINER                                         |
| 54        | RESCONTAINER already exists                                  | RESCONTAINER已经存在                                         |
| 55        | The server is not initialized                                | 服务端未初始化                                               |
| 56        | Invalid market time period                                   | 无效的市场时间段                                             |
| 57        | Network request REFDATA failed                               | 网络请求REFDATA失败                                          |
| 58        | Network request market information failed                    | 网络请求市场信息失败                                         |
| 59        | Network request Kbar failed                                  | 网络请求Kbar失败                                             |
| 60        | ExFactor data does not exist                                 | 不存在复权数据                                               |
| 61        | Network request ExFactor failed                              | 网络请求ExFactor失败                                         |
| 62        | REFDATA configuration error                                  | REFDATA配置错误                                              |
| 63        | Trading day is not a valid contract                          | 交日易非有效合约                                             |
| 80        | ALGO node does not exist                                     | ALGO节点不存在                                               |
| 81        | ALGO node not configured                                     | ALGO节点未配置                                               |
| 82        | ALGO does not match                                          | ALGO节点不匹配                                               |
| 83        | ALGO duplicate node configuration                            | ALGO节点配置重复                                             |
| 100       | userId duplicate                                             | 用户ID重复                                                   |
| 101       | userId not exists                                            | 用户ID不存在                                                 |
| 102       | accountId does not exist                                     | 用户账号不存在                                               |
| 103       | userId duplicate                                             | 用户账号重复                                                 |
| 104       | account not enough money                                     | 用户账号下资金不够                                           |
| 105       | account symbol duplicate                                     | 用户账户下存在相同股票代码                                   |
| 106       | account symbol not exists                                    | 用户账户下不存在股票代码                                     |
| 107       | password error                                               | 用户密码错误                                                 |
| 108       | symbol not support                                           | 不支持symbol                                                 |
| 109       | userid empty                                                 | 用户ID为空                                                   |
| 110       | userid invalid                                               | 非法用户ID                                                   |
| 111       | userid not match                                             | 用户ID不匹配                                                 |
| 118       | sdk version error                                            | sdk 版本错误                                                 |
| 119       | token error                                                  | TOKEN错误                                                    |
| 150       | position list has opposite                                   | 仓位记录中有反向仓位                                         |
| 200       | quote not in cache                                           | 未找到对应的行情数据                                         |
| 202       | Read K line exception                                        | 读取K线异常                                                  |
| 203       | Combined K line anomaly                                      | 合并K线异常                                                  |
| 204       | Open sqlite handle failed                                    | 打开sqlite句柄失败                                           |
| 205       | Sqlite execution failed                                      | sqlite执行失败                                               |
| 206       | Bar does not exist                                           | bar不存在                                                    |
| 207       | Save K line exception                                        | 保存K线异常                                                  |
| 208       | Invalid K line data                                          | 无效的K线数据                                                |
| 257       | Order quantity must be greater than 0                        | 订单的数量必须大于0                                          |
| 258       | Limit order price must be greater than 0                     | 限价单的价钱必须大于0                                        |
| 259       | Insufficient cash available                                  | 可用现金不足                                                 |
| 263       | Account state must be active.                                | 账户状态必须为活跃                                           |
| 264       | Position Qty should be greater than the Qty of OS_SELL Order | T+N标的，或者不可以卖空的标的, 检查标的仓位要大于等于此次订单的卖出的数量 |
| 265       | Ceil or Floor invalid                                        | 标的有涨跌停，必须校验订单价格                               |
| 266       | MarketSession is not the trading time.                       | 当前不是交易时间                                             |
| 267       | Qty of order is not the integer multiple of 'lot size' .     | 订单数量不是lot size的整数                                   |
| 268       | Refdata  does not include the order's info.                  | Refdata中不存在此标的                                        |
| 269       | Qty of order is greater than the allowed maximum Qty         | 下单数量超过最大交易所允许数量                               |
| 270       | Incompleted orders' qty is greater than the default qty (300) . | 用户最大未完成订单超过一定数量                               |
| 271       | Uers' trading frequency is greater than the default value in 1 sec / 1 min. | 同一用户对于同一股票一秒（1）或者一分钟（20）下单数量超过阈值 |
| 273       | Symbol Trade Suspension.                                     | 当前标的今日停牌中                                           |
| 300       | Strategy Not Exists.                                         | 策略不存在                                                   |
| 301       | Analyzer Not Exists.                                         | 分析器不存在                                                 |
| 302       | Strategy Template Not Exists.                                | 策略模块不存在                                               |
| 303       | Strategy Start Failed.                                       | 策略启动失败                                                 |
| 304       | Strategy Export Failed.                                      | 策略导出失败                                                 |
| 305       | No Symbol Configured.                                        | 未配置任何标的                                               |
| 306       | Lang Not Match.                                              | 语言类型不匹配                                               |
| 307       | Analyzer Not Match.                                          | 分析器类型不匹配                                             |
| 308       | Not Support External Analyzer.                               | 系统策略不支持用户分析器                                     |
| 309       | Analyzer Params Loss.                                        | 分析器参数缺失                                               |
| 310       | Analyzer Params Loss.                                        | 分析器参数缺失                                               |
| 311       | Strategy Create Fail.                                        | 策略创建失败                                                 |
| 312       | Strategy Analyzer Lang Not Match.                            | 策略和分析器语言不匹配                                       |
| 313       | Not External Analyzer.                                       | 请选择至少一个用户模块                                       |
| 314       | Symbol Have No Account.                                      | 策略标的没有配置对应的账户                                   |
| 315       | Symbol Num More Than One.                                    | 标的数目的大于一                                             |
| 316       | Symbol Not Found.                                            | 标的未找到                                                   |
| 317       | Strategy Run Exception.                                      | 策略运行异常                                                 |
| 318       | Strategy Symbol Position Not Found.                          | 标的仓位未找到                                               |
| 320       | Wrong Order Side.                                            | 错误的交易方向                                               |
| 321       | Analyzer Param Loss.                                         | 分析器参数缺失                                               |
| 322       | Analyzer Param Invalid.                                      | 分析器参数无效                                               |
| 323       | Analyzer Register Fail.                                      | 分析器注册失败                                               |
| 324       | Account Confused.                                            | 账号混淆                                                     |
| 325       | Quote Confused.                                              | 行情混淆                                                     |
| 326       | Symbol Strategy Position Not Exist.                          | 标的策略仓位不存在                                           |
| 327       | Order Confused.                                              | 订单混淆                                                     |
| 328       | Analyzer Name Repeat.                                        | 分析器名称重复                                               |
| 329       | Internal Analyzer Name Repeat.                               | 内置分析器名称重复                                           |
| 330       | Analyzer is Using.                                           | 分析器正在使用                                               |
| 331       | External Strategy Not Support Non-Remote.                    | 外置策略不支持非远程模式                                     |
| 332       | Analyzer Run Error.                                          | 分析器运行错误                                               |
| 333       | Strategy Symbol Account Invalid.                             | 获取标的账号失败，标的所属账号无效                           |
| 334       | Update Strategy Account Failed.                              | 更新策略账号失败,策略没有该账号                              |
| 335       | Update Symbol OverallPosition Failed.                        | 更新策略标的汇总仓位失败,策略不存在该标的                    |
| 336       | Strategy focus symbols exceed limited!                       | 策略关注标的超过限制                                         |
| 360       | Order Pending Too Long.                                      | 执行器订单挂起超时                                           |
| 361       | Execution State Unexpected.                                  | 执行器状态异常                                               |
| 362       | Execution Failed Too Many Times.                             | 执行器失败次数太多                                           |
| 400       | No Symbol Configured.                                        | 没有配置标的                                                 |
| 401       | Date Invalid.                                                | 回测区间无效                                                 |
| 402       | BackTest Id Not Exist.                                       | 回测ID不存在                                                 |
| 403       | Strategy Not Init.                                           | 策略未初始化                                                 |
| 404       | BackTest Remote Mode.                                        | 回测为远程模式                                               |
| 405       | Market Init Fail.                                            | 市场模块初始化失败                                           |
| 406       | DownStream Init Failed.                                      | 下单模块初始化失败                                           |
| 407       | User Auth Failed.                                            | 用户验证失败                                                 |
| 408       | Strategy Not Exist.                                          | 策略不存在                                                   |
| 409       | BackTest Not Stop.                                           | 回测未处于停止状态                                           |
| 410       | BackTest Have No Record.                                     | 没有回测结果记录                                             |
| 411       | BackTest Started.                                            | 回测已经开始                                                 |
| 412       | BackTest Not Running.                                        | 回测未运行                                                   |
| 413       | BackTest Can not Start.                                      | 回测无法开始                                                 |
| 414       | BackTest User Not Exsit.                                     | 用户不存在                                                   |
| 415       | BackTest ID Not Match.                                       | 回测ID不匹配                                                 |
| 416       | BackTest Instance Too Many.                                  | 同时运行的回测实例太多                                       |
| 417       | Symbol Have No RefData.                                      | 标的RefData未配置                                            |
| 418       | Strategy Template Has Changed, Please try create a new BackTest Instance. | 策略模板已经变更, 请重新创建回测实例                         |
| 500       | Replay quote timeout.                                        | 回放行情数据超时,可能当前回放行情用户过多或同时关注标的太多  |
| 9990      | Terminated by ctrl+c!                                        | Ctrl+c终止                                                   |
| 9999      | Connect Time Out!Please check out your configuration again!  | 连接超时，请检查配置之后重试                                 |
| 60000     | loss config param!                                           | 缺少启动参数                                                 |
| 60001     | config file not exists!                                      | 配置文件不存在                                               |
| 60002     | get strategy info failed!                                    | 获取策略信息失败                                             |
| 60003     | Failed to get backtest parameters                            | 获取回测参数失败                                             |
| 60004     | Initialization signal daata failed                           | 初始化信号数据失败                                           |
| 60005     | Missing analyzer                                             | 缺少分析器                                                   |
| 60006     | Preload bar data timeout                                     | 初始化信号数据失败                                           |
| 60007     | Start backtest failed                                        | 启动回测失败                                                 |
| 60008     | Preload data timeout                                         | 初始化信号数据失败                                           |
| 60009     | Network request timeout                                      | 网络请求超时                                                 |

### 没有触发onBar回调原因
1.先检查在onBeforeMarketOpen事件中是否setFocusSymbols相关标的，若无请设置

2.若已设置相关标的仍未响应onBar，再检查所设置标的是否停牌，看所选频率是不是非tick的

### 没有触发onHandleData回调原因
1.先检查在onBeforeMarketOpen事件中是否setFocusSymbols相关标的，若无请设置

2.若已设置相关标的仍未响应onHandleData，检查onInitialize中是否有配置setGroupMode

3.若已配置setGroupMode仍未响应，再检查所设置标的是否停牌

### getBarsHistory获取不到历史K线的原因
在onInitialize中没有设置相关标的的缓存数据，参考setRequireBars-设置行情数据缓存条数

### 期货交易是否有结算处理
有，模拟盘和实盘在每个交易日的下午5点结算

### 优品的数据是什么时候更新的？
- tick,bar实时更新
- 日线当天17:00结算更新
- 主力合约当天17:00结算后更新
- 日频因子当天17:00结算更新
- 季频因子第二天凌晨3:00更新
- level2因子数据第二天凌晨5:00更新

## 其他问题

### 什么是主力合约？什么的当月连续合约？
主力合约定义如下：

（1）以过去三天的平均持仓量排名为第一名的，则切换该合约为主力合约

（2）品种上市第一天，主力合约为交割日最近的合约

（3）向前切换不回退

（4）每日收盘结算后计算

当月连续、下月连续、下季连续、隔季连续分别用M0,M1,M3,M6表示，如'IFM0'，'IFM1'，'IFM3'，'IFM6'表示沪深300期货当月、下月、下季、隔季连续合约。

举例：如果今天是17年6月29，那么IF类的当月连续就是“IF1707”，因为IF1706在6月16号已经交割了（每个月的第三个周五），下月连续就是“IF1708”，下季连续是“IF1709”，隔季连续是“IF1712”；如果今天是17年7月3号，那么那么IF类的当月连续就是“IF1707”，因为IF1707还未到交割日（每个月的第三个周五），下月连续就是“IF1708”，下季连续是“IF1709”，隔季连续是“IF1712”。