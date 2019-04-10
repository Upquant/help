# 平台数据
优品量化目前支持上交所、深交所以及四大期货交易所的实时行情数据和历史行情数据。
还提供了股票财务、指数、行业板块、资金等金融数据。
## 市场与代码约定
### 股票
代号 |	交易所名称 | 举例
---|--- |---
CS | 上海A股（SH） |	600000.CS(浦发银行)
CS | 深圳A股（SZ） |	000001.CS(中国平安)

### 期货
代号 |	交易所名称 | 举例
---|--- |---
CF | 上海期货交易所（SHFE） |	cu1801.CF(沪铜1801)
CF | 大连商品交易所（DCE） |	jd1801.CF（鸡蛋1801）
CF | 郑州商品交易所（ZCE） |	SR1801.CF(白糖1801）
CF | 中国金融交易所（CFFEX） |	IF1801.CF（IF1801 沪深300）
CF|上海国际能源交易中心（INE）| sc1809.CF(原油1809)

## 特殊集合约定
### 市场&板块集合

市场集合	|说明 
--- |---
CS.SET | 沪深A股全部 
CF.SET | 期货全部（含主力合约） 
UPPLA.SET | 优品行业板块集合 
SW1PLA.SET  | 申万一级行业板块集合 
SW2PLA.SET | 申万二级行业板块集合  

### 期货商品集合
商品集合只包含标准合约，不包含非标准合约如主力合约次主力合约等

代号 |交易所名称 |	商品集合 |集合名称
---|--- |--- |--- 
CF |	SHFE |	cu.PRD |	沪铜
CF |	SHFE |	al.PRD  |	沪铝
CF |	SHFE |	zn.PRD 	 |沪锌
CF |	SHFE |	pb.PRD  |	沪铅
CF |	SHFE |	ru.PRD  |橡胶
CF |	SHFE |	fu.PRD  |	燃油
CF |	SHFE |	rb.PRD  |		螺纹
CF |	SHFE |	wr.PRD  |	线材
CF |	SHFE |	au.PRD  |	黄金
CF |	SHFE |	ag.PRD  |	白银
CF |	SHFE |	bu.PRD  |		沥青
CF |	SHFE |	hc.PRD  |		热卷
CF |	SHFE |	ni.PRD  |	沪镍
CF |	SHFE |	sn.PRD  |	沪锡
CF |	INE|	        sc.PRD  |	     	原油
CF |	DCE |	a.PRD  |		豆一
CF |	DCE |	b.PRD  |		豆二
CF |	DCE |	c.PRD  |	玉米
CF |	DCE |	m.PRD  |		豆粕
CF |	DCE |	y.PRD  |豆油
CF |	DCE |	p.PRD  |棕榈油
CF |	DCE |	l.PRD  |	塑料
CF |	DCE |	v.PRD  |	PVC
CF |	DCE |	j.PRD  |	焦炭
CF |	DCE |	jm.PRD  |	焦煤
CF |	DCE |	i.PRD  |铁矿石
CF |	DCE |	jd.PRD  |	鸡蛋
CF |	DCE |	fb.PRD  |	纤维板
CF |	DCE |	bb.PRD  |胶合板
CF |	DCE |	pp.PRD  |	PP
CF |	DCE |	cs.PRD  |淀粉
CF |	ZCE |	PM.PRD  |	普麦
CF |	ZCE |	WH.PRD  |	强麦
CF |	ZCE |	SR.PRD  |	白糖
CF |	ZCE |	CF.PRD  |	棉花
CF |	ZCE |	TA.PRD  |PTA
CF |	ZCE |	OI.PRD  |菜油
CF |	ZCE |	RI.PRD  |	早籼稻
CF |	ZCE |	MA.PRD  |甲醇
CF |	ZCE |	FG.PRD  |玻璃
CF |	ZCE |	RS.PRD  |菜籽
CF |	ZCE |	RM.PRD  |菜粕
CF |	ZCE |	ZC.PRD  |动力煤
CF |	ZCE |	JR.PRD  |	粳稻
CF |	ZCE |	LR.PRD  |晚籼稻
CF |	ZCE |	SF.PRD  |	硅铁
CF |	ZCE |	SM.PRD  |锰硅
CF |	ZCE |	AP.PRD  |	苹果
CF |	ZCE |	CY.PRD  |棉纱
CF |	CFFEX |	IF.PRD  |	沪深300
CF |	CFFEX |	IC.PRD  |	中证500
CF |	CFFEX |	IH.PRD  |上证50
CF |	CFFEX |	T.PRD  |	十年期国债
CF |	CFFEX |	TF.PRD  |	五年期国债

## 行情数据
优品量化目前支持上交所、深交所以及五大期货交易所的实时行情数据和历史行情数据，包含Tick行情和Bar行情。

### Tick-Tick数据

属性 | 类型 |  说明
---|---|---
symbol | str  | 标的代码。
bid | double  | 买一价。
ask | double  | 卖一价。
bidVol | double  | 买一量。（单位为实际证券股数）
askVol | double  | 卖一量。
last | double  | 最新成交价。
lastVolume | double  | 最新成交量。
lastTurnover | double  | 最新成交额。
totalVolume | double  | 交易日总成交量。
totalTurnover | double  | 交易日总成交额。
high | double  | 交易日最高价。
low | double  | 交易日最低价。
open | double  | 交易日开盘价。
close | double  | 交易日收盘价。
tradeDate | long  | 交易日-年月日，格式为YYYYMMDD
timeExch | long  | 交易所的实时时间戳-年月日时分秒 (ms)，
timeStr| str | 交易时间，格式为YYYYMMDD-hhmmss-xxx
bids| vector<qtyprice> |  五档买价，包括数量和价格，qtyprice{"quantity":qty,"price":price}。
asks| vector<qtyprice> |  五档卖价，包括数量和价格。
ceil| double  | 涨停价。
ceil| double  | 跌停价。
position| double  | 持仓量(期货)。
preClose| double  | 昨日收盘价。
preSettle| double  | 昨日结算价。

### Bar-Bar数据
属性 | 类型 |  说明
---|---|---
symbol | str  | 标的代码。
tradeDate | long  | 交易日-年月日，格式为YYYYMMDD
timeStop | long  | bar对应的截止时间戳 (ms)
timeStr| str | 交易时间，格式为YYYYMMDD-hhmmss-xxx
timeSpan| long  |K线的时间频率。60-1分钟；300-5分钟；900-15分钟；1800-30分钟；3600-60分钟；86400-1日。
high | double  | 交易日最高价。
low | double  | 交易日最低价。
open | double  | 交易日开盘价。
close | double  | 交易日收盘价。
lastVolume | double  | 最新成交量。
lastTurnover | double  | 最新成交额。
totalVolume | double  | 交易日总成交量。
totalTurnover | double  | 交易日总成交额。
preClose| double  | 昨日收盘价。

### Bar行情的频率类型
```
enum ETimeSpan
{
    MIN_1 ,  //1分钟K线
    MIN_5 ,  //5分钟K线
    MIN_15 , //15分钟K线
    MIN_30,  //30分钟K线
    MIN_60,  //60分钟K线
    DAY_1,   //日K线
};
```
### 期货主力合约

主力合约品种代号后加上Z0；次主力合约品种代号后加上Z1。

商品期货和国债主力合约定义如下：

（1）连续三天日持仓量最大，则切换该合约为主力合约

（2）品种上市第一天，主力合约为交割日最近的合约

（3）若当前主力合约到交割月份前一个月的月末持仓量仍为最大，则强制切换成持仓量次之的合约为主力合约，防止主力合约进入交割月

（4）向前切换不回退

（5）每日收盘结算后计算

股指期货主力合约定义如下：主力合约是在最后交易日前一交易日切换，当月合约和下月合约是在最后交易日后一交易日切换。如4月17是最后交易日，主力合约在16号开盘前就已切换；当月合约和下月合约是在4月18号开盘前完成合约切换。

以豆油（Y）为例，主力表示为YZ0，次主力表示为YZ1。判断合约的方法：以前三天持仓量最高且持续递增为主力合约，次之为次主力合约。次主力合约的日期必须大于主力合约。

各品种主力/次主力合约表：

代号 |	交易所名称 |	主力合约 |	次主力合约 |	合约名称
---|--- |--- |--- |---
CF |	SHFE |	cuZ0.CF |	cuZ1.CF |	沪铜
CF |	SHFE |	alZ0.CF |	alZ1.CF |	沪铝
CF |	SHFE |	znZ0.CF	 |znZ1.CF |	沪锌
CF |	SHFE |	pbZ0.CF |	pbZ1.CF |	沪铅
CF |	SHFE |	ruZ0.CF |	ruZ1.CF |	橡胶
CF |	SHFE |	fuZ0.CF |	fuZ1.CF |	燃油
CF |	SHFE |	rbZ0.CF |	rbZ1.CF |	螺纹
CF |	SHFE |	wrZ0.CF |	wrZ1.CF |	线材
CF |	SHFE |	auZ0.CF |	auZ1.CF |	黄金
CF |	SHFE |	agZ0.CF |	agZ1.CF |	白银
CF |	SHFE |	buZ0.CF |	buZ1.CF |	沥青
CF |	SHFE |	hcZ0.CF |	hcZ1.CF |	热卷
CF |	SHFE |	niZ0.CF |	niZ1.CF |	沪镍
CF |	SHFE |	snZ0.CF |	snZ1.CF |	沪锡
CF |	INE|	        scZ0.CF |	        scZ1.CF |	原油
CF |	DCE |	aZ0.CF |	aZ1.CF |	豆一
CF |	DCE |	bZ0.CF |	bZ1.CF |	豆二
CF |	DCE |	cZ0.CF |	cZ1.CF |	玉米
CF |	DCE |	mZ0.CF |	mZ1.CF |	豆粕
CF |	DCE |	yZ0.CF |	yZ1.CF |	豆油
CF |	DCE |	pZ0.CF |	pZ1.CF |	棕榈油
CF |	DCE |	lZ0.CF |	lZ1.CF |	塑料
CF |	DCE |	vZ0.CF |	vZ1.CF |	PVC
CF |	DCE |	jZ0.CF |	jZ1.CF |	焦炭
CF |	DCE |	jmZ0.CF |	jmZ1.CF |	焦煤
CF |	DCE |	iZ0.CF |	iZ1.CF |	铁矿石
CF |	DCE |	jdZ0.CF |	jdZ1.CF |	鸡蛋
CF |	DCE |	fbZ0.CF |	fbZ1.CF |	纤维板
CF |	DCE |	bbZ0.CF |	bbZ1.CF |	胶合板
CF |	DCE |	ppZ0.CF |	ppZ1.CF |	PP
CF |	DCE |	csZ0.CF |	csZ1.CF |	淀粉
CF |	ZCE |	PMZ0.CF |	PMZ1.CF |	普麦
CF |	ZCE |	WHZ0.CF |	WHZ1.CF |	强麦
CF |	ZCE |	SRZ0.CF |	SRZ1.CF |	白糖
CF |	ZCE |	CFZ0.CF |	CFZ1.CF |	棉花
CF |	ZCE |	TAZ0.CF |	TAZ1.CF |	PTA
CF |	ZCE |	OIZ0.CF |	OIZ1.CF |	菜油
CF |	ZCE |	RIZ0.CF |	RIZ1.CF |	早籼稻
CF |	ZCE |	MAZ0.CF |	MAZ1.CF |	甲醇
CF |	ZCE |	FGZ0.CF |	FGZ1.CF |	玻璃
CF |	ZCE |	RSZ0.CF |	RSZ1.CF |	菜籽
CF |	ZCE |	RMZ0.CF |	RMZ1.CF |	菜粕
CF |	ZCE |	ZCZ0.CF |	ZCZ1.CF |	动力煤
CF |	ZCE |	JRZ0.CF |	JRZ1.CF |	粳稻
CF |	ZCE |	LRZ0.CF |	LRZ1.CF |	晚籼稻
CF |	ZCE |	SFZ0.CF |	SFZ1.CF |	硅铁
CF |	ZCE |	SMZ0.CF |	SMZ1.CF |	锰硅
CF |	ZCE |	APZ0.CF |	APZ1.CF |	苹果
CF |	ZCE |	CYZ0.CF |	CYZ1.CF |	棉纱
CF |	CFFEX |	IFZ0.CF |	 |	沪深300
CF |	CFFEX |	ICZ0.CF |	 |	中证500
CF |	CFFEX |	IHZ0.CF |	 |	上证50
CF |	CFFEX |	TZ0.CF |	TZ1.CF |	十年期国债
CF |	CFFEX |	TFZ0.CF |	TFZ1.CF |	五年期国债

**注意：** 如果用主力合约下单，平台不负责帮用户做换仓操作，需要用户自己通过策略控制。

### 期货连续合约

【股指期货】

支持当月连续、下月连续、下季连续、隔季连续合约、分别用品种名称+M0,M1,M3,M6表示，如'IFM0'，'IFM1'，'IFM3'，'IFM6'表示股指期货沪深300当月、下月、下季、隔季连续合约。

举例：如果今天是17年6月29日，则IF类的当月连续为“IF1707”，因为IF1706在6月16日已经交割（每个月的第三个周五），下月连续为“IF1708”，下季连续为“IF1709”，隔季连续为“IF1712”；如果今天是17年7月3日，则IF类的当月连续为“IF1707”，因为IF1707还未到交割日（每个月的第三个周五），下月连续为“IF1708”，下季连续为“IF1709”，隔季连续为“IF1712”。

【国债期货】

支持当季连续、下季连续、隔季连续合约、分别用品种名称+M0,M3,M6表示，如'TM0'，'TM3'，'TM6'表示十年国债期货当季、下季、隔季连续合约。

## 标的基础信息

refData中保存数据的基础信息

属性 | 类型 |  说明
---|---|---
symbol | str  | 标的代码。
marketName | str  | 品种名称。CS-股票；CF-期货；IDX-指数。
exchange |str | 交易所及自定义类型后缀。SZ-深交所；SH上交所；DCE-大商所；SHFE-上期所；CZCE-郑商所；CFFEX-中金所；PLATE-板块；SET-证券集合。
currency | str  | 币种。
lotSize | double  | 一手标的的数目。
name | str  | 标的名称。
tplus | int  | T+N交易。例如tplus为1时为t+1交易。
marginRate | double | 保证金比例。
shortSellable | bool | 是否可卖空。
valuePerUnit | double | 合约乘数
priceTick| double | 标的价格浮动最小单位值
exchSymbol| str | 交易所的原始symbol 
isStandard| bool | 是否是交易所的标准合约，比如主力合约 IFZ0.CF 就不是交易所的标准合约
tradeMarket | str |该标的对应的交易市场，目前仅有 CS CF 
listDate | int | 上市日，无则为0。
lastTradeDate | int |最后交易日(退市日)，无则为0。

## 股票指数代码

目前只提供2011年1月1日至今的指数行情数据以及指数成份股数据

指数代码|	指数简称 
---|---
000001.IDX |	上证指数
399001.IDX |	深证成指
000300.IDX |	沪深300
399005.IDX |	中小板指
399006.IDX |	创业板指
000010.IDX |	上证180
000016.IDX |	上证50
000009.IDX |	上证380
000132.IDX |	上证100
000133.IDX |	上证150
399106.IDX |	深证综指
399004.IDX |	深证100
399007.IDX |	深圳300
399008.IDX |	中小300
000852.IDX |	中证1000
000903.IDX |	中证100
000904.IDX |	中证200
000905.IDX |	中证500
000906.IDX |	中证800

## 股票行业板块代码

目前主要提供优品行业板块代码、申万一级行业板块代码和申万二级行业板块代码。可以通过集合获取对应板块的所有代码，如调用UPPLA.SET则返回的是优品行业板块的所有代码。

### 优品行业代码

提供优品行业的成分股数据，只支持2014年1月1日至今的数据。

行业代码  |	行业名称 
--- | ---
880001.PLA|	农林牧渔
880002.PLA|	石油
880003.PLA|	煤炭
880004.PLA|	其他采掘
880007.PLA|	化工原料
880008.PLA|	化学制品
880009.PLA|	化学纤维
880010.PLA|	橡胶塑料
880012.PLA|	化工新材料
880013.PLA|	钢铁
880014.PLA|	有色金属
880015.PLA|	金属新材料
880016.PLA|	建材
880017.PLA|	建筑
880018.PLA|	通用机械
880019.PLA|	工程机械
880020.PLA|	仪器仪表
880021.PLA|	电气设备
880022.PLA|	金属制品
880023.PLA|	电子元器件
880024.PLA|	汽车
880025.PLA|	通信设备
880026.PLA|	电脑设备
880027.PLA|	家用电器
880028.PLA|	酿酒饮料
880029.PLA|	食品加工
880030.PLA|	纺织服装
880031.PLA|	造纸印刷
880033.PLA|	轻工制造
880035.PLA|	医药
880039.PLA|	医疗器械服务
880041.PLA|	电力
880042.PLA|	水务
880043.PLA|	燃气
880044.PLA|	环保工程
880045.PLA|	交通运输
880046.PLA|	房地产
880047.PLA|	银行
880048.PLA|	多元金融
880049.PLA|	证券
880050.PLA|	保险
880051.PLA|	零售贸易
880052.PLA|	旅游
880053.PLA|	酒店餐饮
880056.PLA|	通信运营
880057.PLA|	网络服务
880058.PLA|	软件服务
880059.PLA|	文化传媒
880060.PLA|	综合
880064.PLA|	国防军工
880065.PLA|	交运设备

### 申万一级行业代码
提供申万一级行业,包含2011版和2014版，并可获取申万一级行业指数日线级别行情数据。

行业代码|行业名称
---|---
 801010.PLA | 农林牧渔 
 801020.PLA | 采掘 
 801030.PLA | 化工 
 801040.PLA | 黑色金属 
 801050.PLA | 有色金属 
 801060.PLA | 建筑建材 
 801070.PLA | 机械设备 
 801080.PLA | 电子 
 801090.PLA | 交运设备 
 801100.PLA | 信息设备 
 801110.PLA | 家用电器 
 801120.PLA | 食品饮料 
 801130.PLA | 纺织服装 
 801140.PLA | 轻工制造 
 801150.PLA | 医药生物 
 801160.PLA | 公用事业 
 801170.PLA | 交通运输 
 801180.PLA | 房地产 
 801190.PLA | 金融服务 
 801200.PLA | 商业贸易 
 801210.PLA | 餐饮旅游 
 801220.PLA | 信息服务 
 801230.PLA | 综合 
 801710.PLA | 建筑材料 
 801720.PLA | 建筑装饰 
 801730.PLA | 电气设备 
 801740.PLA | 国防军工 
 801750.PLA | 计算机 
 801760.PLA | 传媒 
 801770.PLA | 通信 
 801780.PLA | 银行 
 801790.PLA | 非银金融 
 801880.PLA | 汽车 
 801890.PLA | 机械设备 

### 申万二级行业代码
提供申万二级行业代码，,包含2011版和2014版

|行业代码|行业名称|
|---|---|
| 801011.PLA | 林业 |
| 801012.PLA | 农产品加工 |
| 801013.PLA | 农业综合 |
| 801014.PLA | 饲料 |
| 801015.PLA | 渔业 |
| 801016.PLA | 种植业 |
| 801017.PLA | 畜禽养殖 |
| 801018.PLA | 动物保健 |
| 801021.PLA | 煤炭开采 |
| 801022.PLA | 其他采掘 |
| 801023.PLA | 石油开采 |
| 801024.PLA | 采掘服务 |
| 801031.PLA | 化工新材料 |
| 801032.PLA | 化学纤维 |
| 801033.PLA | 化学原料 |
| 801034.PLA | 化学制品 |
| 801035.PLA | 石油化工 |
| 801036.PLA | 塑料 |
| 801037.PLA | 橡胶 |
| 801041.PLA | 钢铁 |
| 801051.PLA | 金属非金属新材料 |
| 801052.PLA | 有色金属冶炼与加工 |
| 801053.PLA | 黄金 |
| 801054.PLA | 稀有金属 |
| 801055.PLA | 工业金属 |
| 801061.PLA | 建筑材料 |
| 801062.PLA | 建筑装饰 |
| 801071.PLA | 电气设备 |
| 801072.PLA | 通用机械 |
| 801073.PLA | 仪器仪表 |
| 801074.PLA | 专用设备 |
| 801075.PLA | 金属制品 |
| 801076.PLA | 运输设备 |
| 801081.PLA | 半导体 |
| 801082.PLA | 其他电子 |
| 801083.PLA | 元件 |
| 801084.PLA | 光学光电子 |
| 801085.PLA | 电子制造 |
| 801091.PLA | 非汽车交运设备 |
| 801092.PLA | 交运设备服务 |
| 801093.PLA | 汽车零部件 |
| 801094.PLA | 汽车整车 |
| 801101.PLA | 计算机设备 |
| 801102.PLA | 通信设备 |
| 801111.PLA | 白色家电 |
| 801112.PLA | 视听器材 |
| 801123.PLA | 饮料制造 |
| 801124.PLA | 食品加工制造 |
| 801131.PLA | 纺织制造 |
| 801132.PLA | 服装家纺 |
| 801141.PLA | 包装印刷 |
| 801142.PLA | 家用轻工 |
| 801143.PLA | 造纸 |
| 801144.PLA | 其他轻工制造 |
| 801151.PLA | 化学制药 |
| 801152.PLA | 生物制品 |
| 801153.PLA | 医疗器械 |
| 801154.PLA | 医药商业 |
| 801155.PLA | 中药 |
| 801156.PLA | 医疗服务 |
| 801161.PLA | 电力 |
| 801162.PLA | 环保工程及服务 |
| 801163.PLA | 燃气 |
| 801164.PLA | 水务 |
| 801171.PLA | 港口 |
| 801172.PLA | 公交 |
| 801173.PLA | 航空运输 |
| 801174.PLA | 机场 |
| 801175.PLA | 高速公路 |
| 801176.PLA | 航运 |
| 801177.PLA | 铁路运输 |
| 801178.PLA | 物流 |
| 801181.PLA | 房地产开发 |
| 801182.PLA | 园区开发 |
| 801191.PLA | 多元金融 |
| 801192.PLA | 银行 |
| 801193.PLA | 证券 |
| 801194.PLA | 保险 |
| 801201.PLA | 零售 |
| 801202.PLA | 贸易 |
| 801203.PLA | 一般零售 |
| 801204.PLA | 专业零售 |
| 801205.PLA | 商业物业经营 |
| 801211.PLA | 餐饮 |
| 801212.PLA | 景点 |
| 801213.PLA | 酒店 |
| 801214.PLA | 旅游综合 |
| 801215.PLA | 其他休闲服务 |
| 801221.PLA | 传媒 |
| 801222.PLA | 计算机应用 |
| 801223.PLA | 通信运营 |
| 801224.PLA | 网络服务 |
| 801231.PLA | 综合 |
| 801711.PLA | 水泥制造 |
| 801712.PLA | 玻璃制造 |
| 801713.PLA | 其他建材 |
| 801721.PLA | 房屋建设 |
| 801722.PLA | 装修装饰 |
| 801723.PLA | 基础建设 |
| 801724.PLA | 专业工程 |
| 801725.PLA | 园林工程 |
| 801731.PLA | 电机 |
| 801732.PLA | 电气自动化设备 |
| 801733.PLA | 电源设备 |
| 801734.PLA | 高低压设备 |
| 801741.PLA | 航天装备 |
| 801742.PLA | 航空装备 |
| 801743.PLA | 地面兵装 |
| 801744.PLA | 船舶制造 |
| 801751.PLA | 营销传播 |
| 801752.PLA | 互联网传媒 |
| 801761.PLA | 文化传媒 |
| 801881.PLA | 其他交运设备 |

## 股票交易指标-日频
### 交易状态
fields |说明 
---|---
TRADE_STA |交易状态。交易-True  停牌-False
LIST_STA |上市状态。正常上市-1，暂停上市-2，终止上市-3，恢复上市-4，退市整理-5，未上市-6
CUML_EX_FACTOR |累计复权因子
ST_STA|特殊处理状态， ST  *ST-True ， 正常-False

### 基于L2衍生因子-深市
主要统计不同资金规模的委托的成交量和成交金额（只考虑限价单）  
因子名由三部分组成：  
因子名后面的两个数字，表示该因子统计的是委托金额位于该区间内的委托，如0_5表示统计的是委托金额在`0w~5w`的委拖;如果是`inf`则表示无穷大，即没有上限    
因子名中间是`VOL`或`AMOUNT`，`VOL`表示统计的是成交量，`AMOUNT`表示统计的是成交金额  
因子名前面是`BUY`、`SELL`、`QUITS`，`BUY`表示统计的是买委托成交（即只要买委托的委托金额位于后面指定的区间则计算），`SELL`表示统计的是卖委托成交（即只要卖委托的委托金额位于后面指定的区间则计算），`QUITS`表示统计买&卖委托成交（即需要买卖委托双方的委托金额都位于后面指定的区间才计算）  


|fields|说明|
|---|---|
|BUY_AMOUNT_0_5|委托金额0w-5w的买委托的成交金额|
|BUY_AMOUNT_1000_INF|委托金额1000w-∞的买委托的成交金额|
|BUY_AMOUNT_100_200|委托金额100w-200w的买委托的成交金额|
|BUY_AMOUNT_10_20|委托金额10w-20w的买委托的成交金额|
|BUY_AMOUNT_200_500|委托金额200w-500w的买委托的成交金额|
|BUY_AMOUNT_20_50|委托金额20w-50w的买委托的成交金额|
|BUY_AMOUNT_500_1000|委托金额500w-1000w的买委托的成交金额|
|BUY_AMOUNT_50_100|委托金额50w-100w的买委托的成交金额|
|BUY_AMOUNT_5_10|委托金额5w-10w的买委托的成交金额|
|BUY_VOL_0_5|委托金额0w-5w的买委托的成交量|
|BUY_VOL_1000_INF|委托金额1000w-∞的买委托的成交量|
|BUY_VOL_100_200|委托金额100w-200w的买委托的成交量|
|BUY_VOL_10_20|委托金额10w-20w的买委托的成交量|
|BUY_VOL_200_500|委托金额200w-500w的买委托的成交量|
|BUY_VOL_20_50|委托金额20w-50w的买委托的成交量|
|BUY_VOL_500_1000|委托金额500w-1000w的买委托的成交量|
|BUY_VOL_50_100|委托金额50w-100w的买委托的成交量|
|BUY_VOL_5_10|委托金额5w-10w的买委托的成交量|
|ORDER_BUY_AMOUNT_0_5Q|0万_0.5万的买委托金额|
|ORDER_BUY_AMOUNT_1000_INF|100万_∞的买委托金额|
|ORDER_BUY_AMOUNT_100_200|100万_200万的买委托金额|
|ORDER_BUY_AMOUNT_10_20|10万_20万的买委托金额|
|ORDER_BUY_AMOUNT_1_2|1万_2万的买委托金额|
|ORDER_BUY_AMOUNT_200_500|200万_500万的买委托金额|
|ORDER_BUY_AMOUNT_20_50|20万_50万的买委托金额|
|ORDER_BUY_AMOUNT_2_5|2万_5万的买委托金额|
|ORDER_BUY_AMOUNT_500_1000|500万_1000万的买委托金额|
|ORDER_BUY_AMOUNT_50_100|50万_100万的买委托金额|
|ORDER_BUY_AMOUNT_5_10|5万_10万的买委托金额|
|ORDER_BUY_AMOUNT_5Q_1|0.5万_1万的买委托金额|
|ORDER_BUY_FRQUENCE_0_5Q|0万_0.5万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_1000_INF|100万_∞的买委托金额的频数|
|ORDER_BUY_FRQUENCE_100_200|100万_200万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_10_20|10万_20万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_1_2|1万_2万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_200_500|200万_500万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_20_50|20万_50万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_2_5|2万_5万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_500_1000|500万_1000万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_50_100|50万_100万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_5_10|5万_10万的买委托金额的频数|
|ORDER_BUY_FRQUENCE_5Q_1|0.5万_1万的买委托金额的频数|
|ORDER_BUY_VOLUME_0_5Q|0万_0.5万的买委托量|
|ORDER_BUY_VOLUME_1000_INF|100万_∞的买委托量|
|ORDER_BUY_VOLUME_100_200|100万_200万的买委托量|
|ORDER_BUY_VOLUME_10_20|10万_20万的买委托量|
|ORDER_BUY_VOLUME_1_2|1万_2万的买委托量|
|ORDER_BUY_VOLUME_200_500|200万_500万的买委托量|
|ORDER_BUY_VOLUME_20_50|20万_50万的买委托量|
|ORDER_BUY_VOLUME_2_5|2万_5万的买委托量|
|ORDER_BUY_VOLUME_500_1000|500万_1000万的买委托量|
|ORDER_BUY_VOLUME_50_100|50万_100万的买委托量|
|ORDER_BUY_VOLUME_5_10|5万_10万的买委托量|
|ORDER_BUY_VOLUME_5Q_1|0.5万_1万的买委托量|
|ORDER_SELL_AMOUNT_0_5Q|0万_0.5万的卖委托金额|
|ORDER_SELL_AMOUNT_1000_INF|100万_∞的卖委托金额|
|ORDER_SELL_AMOUNT_100_200|100万_200万的卖委托金额|
|ORDER_SELL_AMOUNT_10_20|10万_20万的卖委托金额|
|ORDER_SELL_AMOUNT_1_2|1万_2万的卖委托金额|
|ORDER_SELL_AMOUNT_200_500|200万_500万的卖委托金额|
|ORDER_SELL_AMOUNT_20_50|20万_50万的卖委托金额|
|ORDER_SELL_AMOUNT_2_5|2万_5万的卖委托金额|
|ORDER_SELL_AMOUNT_500_1000|500万_1000万的卖委托金额|
|ORDER_SELL_AMOUNT_50_100|50万_100万的卖委托金额|
|ORDER_SELL_AMOUNT_5_10|5万_10万的卖委托金额|
|ORDER_SELL_AMOUNT_5Q_1|0.5万_1万的卖委托金额|
|ORDER_SELL_FRQUENCE_0_5Q|0万_0.5万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_1000_INF|100万_∞的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_100_200|100万_200万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_10_20|10万_20万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_1_2|1万_2万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_200_500|200万_500万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_20_50|20万_50万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_2_5|2万_5万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_500_1000|500万_1000万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_50_100|50万_100万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_5_10|5万_10万的卖委托金额的频数|
|ORDER_SELL_FRQUENCE_5Q_1|0.5万_1万的卖委托金额的频数|
|ORDER_SELL_VOLUME_0_5Q|0万_0.5万的卖委托量|
|ORDER_SELL_VOLUME_1000_INF|100万_∞的卖委托量|
|ORDER_SELL_VOLUME_100_200|100万_200万的卖委托量|
|ORDER_SELL_VOLUME_10_20|10万_20万的卖委托量|
|ORDER_SELL_VOLUME_1_2|1万_2万的卖委托量|
|ORDER_SELL_VOLUME_200_500|200万_500万的卖委托量|
|ORDER_SELL_VOLUME_20_50|20万_50万的卖委托量|
|ORDER_SELL_VOLUME_2_5|2万_5万的卖委托量|
|ORDER_SELL_VOLUME_500_1000|500万_1000万的卖委托量|
|ORDER_SELL_VOLUME_50_100|50万_100万的卖委托量|
|ORDER_SELL_VOLUME_5_10|5万_10万的卖委托量|
|ORDER_SELL_VOLUME_5Q_1|0.5万_1万的卖委托量|
|QUITS_AMOUNT_0_5|委托金额0w-5w的买委托&委托金额0w~5w的卖委托成交的成交金额|
|QUITS_AMOUNT_1000_INF|委托金额1000w-∞的买委托&委托金额1000w-∞的卖委托成交的成交金额|
|QUITS_AMOUNT_100_200|委托金额100w-200w的买委托&委托金额100w-200w的卖委托成交的成交金额|
|QUITS_AMOUNT_10_20|委托金额10w-20w的买委托&委托金额10w-20w的卖委托成交的成交金额|
|QUITS_AMOUNT_200_500|委托金额200w-500w的买委托&委托金额200w-500w的卖委托成交的成交金额|
|QUITS_AMOUNT_20_50|委托金额20w-50w的买委托&委托金额20w-50w的卖委托成交的成交金额|
|QUITS_AMOUNT_500_1000|委托金额500w-1000w的买委托&委托金额500w-1000w的卖委托成交的成交金额|
|QUITS_AMOUNT_50_100|委托金额50w-100w的买委托&委托金额50w-100w的卖委托成交的成交金额|
|QUITS_AMOUNT_5_10|委托金额5w-10w的买委托&委托金额5w-10w的卖委托成交的成交金额|
|QUITS_VOL_0_5|委托金额0w-5w的买委托&委托金额0w~5w的卖委托成交的成交量|
|QUITS_VOL_1000_INF|委托金额1000w-∞的买委托&委托金额1000w-∞的卖委托成交的成交量|
|QUITS_VOL_100_200|委托金额100w-200w的买委托&委托金额100w-200w的卖委托成交的成交量|
|QUITS_VOL_10_20|委托金额10w-20w的买委托&委托金额10w-20w的卖委托成交的成交量|
|QUITS_VOL_200_500|委托金额200w-500w的买委托&委托金额200w-500w的卖委托成交的成交量|
|QUITS_VOL_20_50|委托金额20w-50w的买委托&委托金额20w-50w的卖委托成交的成交量|
|QUITS_VOL_500_1000|委托金额500w-1000w的买委托&委托金额500w-1000w的卖委托成交的成交量|
|QUITS_VOL_50_100|委托金额50w-100w的买委托&委托金额50w-100w的卖委托成交的成交量|
|QUITS_VOL_5_10|委托金额5w-10w的买委托&委托金额5w-10w的卖委托成交的成交量|
|SELL_AMOUNT_0_5|委托金额0w-5w的卖委托的成交金额|
|SELL_AMOUNT_1000_INF|委托金额1000w-∞的卖委托的成交金额|
|SELL_AMOUNT_100_200|委托金额100w-200w的卖委托的成交金额|
|SELL_AMOUNT_10_20|委托金额10w-20w的卖委托的成交金额|
|SELL_AMOUNT_200_500|委托金额200w-500w的卖委托的成交金额|
|SELL_AMOUNT_20_50|委托金额20w-50w的卖委托的成交金额|
|SELL_AMOUNT_500_1000|委托金额500w-1000w的卖委托的成交金额|
|SELL_AMOUNT_50_100|委托金额50w-100w的卖委托的成交金额|
|SELL_AMOUNT_5_10|委托金额5w-10w的卖委托的成交金额|
|SELL_VOL_0_5|委托金额0w-5w的卖委托的成交量|
|SELL_VOL_1000_INF|委托金额1000w-∞的卖委托的成交量|
|SELL_VOL_100_200|委托金额100w-200w的卖委托的成交量|
|SELL_VOL_10_20|委托金额10w-20w的卖委托的成交量|
|SELL_VOL_200_500|委托金额200w-500w的卖委托的成交量|
|SELL_VOL_20_50|委托金额20w-50w的卖委托的成交量|
|SELL_VOL_500_1000|委托金额500w-1000w的卖委托的成交量|
|SELL_VOL_50_100|委托金额50w-100w的卖委托的成交量|
|SELL_VOL_5_10|委托金额5w-10w的卖委托的成交量|
|TRADE_AMOUNT_0_5Q|0万_0.5万的成交金额|
|TRADE_AMOUNT_1000_INF|100万_∞的成交金额|
|TRADE_AMOUNT_100_200|100万_200万的成交金额|
|TRADE_AMOUNT_10_20|10万_20万的成交金额|
|TRADE_AMOUNT_1_2|1万_2万的成交金额|
|TRADE_AMOUNT_200_500|200万_500万的成交金额|
|TRADE_AMOUNT_20_50|20万_50万的成交金额|
|TRADE_AMOUNT_2_5|2万_5万的成交金额|
|TRADE_AMOUNT_500_1000|500万_1000万的成交金额|
|TRADE_AMOUNT_50_100|50万_100万的成交金额|
|TRADE_AMOUNT_5_10|5万_10万的成交金额|
|TRADE_AMOUNT_5Q_1|0.5万_1万的成交金额|
|TRADE_FRQUENCE_0_5Q|0万_0.5万的成交金额的频数|
|TRADE_FRQUENCE_1000_INF|100万_∞的成交金额的频数|
|TRADE_FRQUENCE_100_200|100万_200万的成交金额的频数|
|TRADE_FRQUENCE_10_20|10万_20万的成交金额的频数|
|TRADE_FRQUENCE_1_2|1万_2万的成交金额的频数|
|TRADE_FRQUENCE_200_500|200万_500万的成交金额的频数|
|TRADE_FRQUENCE_20_50|20万_50万的成交金额的频数|
|TRADE_FRQUENCE_2_5|2万_5万的成交金额的频数|
|TRADE_FRQUENCE_500_1000|500万_1000万的成交金额的频数|
|TRADE_FRQUENCE_50_100|50万_100万的成交金额的频数|
|TRADE_FRQUENCE_5_10|5万_10万的成交金额的频数|
|TRADE_FRQUENCE_5Q_1|0.5万_1万的成交金额的频数|
|TRADE_VOLUME_0_5Q|0万_0.5万的成交量|
|TRADE_VOLUME_1000_INF|100万_∞的成交量|
|TRADE_VOLUME_100_200|100万_200万的成交量|
|TRADE_VOLUME_10_20|10万_20万的成交量|
|TRADE_VOLUME_1_2|1万_2万的成交量|
|TRADE_VOLUME_200_500|200万_500万的成交量|
|TRADE_VOLUME_20_50|20万_50万的成交量|
|TRADE_VOLUME_2_5|2万_5万的成交量|
|TRADE_VOLUME_500_1000|500万_1000万的成交量|
|TRADE_VOLUME_50_100|50万_100万的成交量|
|TRADE_VOLUME_5_10|5万_10万的成交量|
|TRADE_VOLUME_5Q_1|0.5万_1万的成交量|


### 市值数据
主要展示策略常用因子

fields |说明 
---|---
PCF|市现率
PS|市销率
PB|市净率
PE|市盈率
VIBR_RANGE|振幅
TURNOVER_RATE|换手率
TOT_SHARE | 总股本
FLOAT_SHARE  | 流通股合计
CIRC_CAP|流通市值
MKT_CAP|总市值
CASH_BT|每股股利(税前)
CASH_AT|每股股利(税后)
BONUS_SHR|每股红股
HLD_NUM_CHG_RATIO|股东总户数增长率
STATE_REST|限售股份(国家持股)
STATE_LEG_REST|限售股份(国家法人持股)
MAN_RES|限售股份(高管持股)
DOM_REST|限售股份(其它内资持股合计)
DOM_NATU_REST|限售股份(其它内资_境内自然人持股)
DOM_LEG_REST|限售股份(其它内资_境内法人持股)
FORE_REST|限售股份(外资持股合计)
FORE_NATU_RES|限售股份(外资_境外自然人持股)
FORE_LEG_RES|限售股份(外资_境外法人持股)

## 股票财务数据-季频
### 资产负债表
fields |说明 
---|---
CASH_EQUIV	|	货币资金及现金或存放中央银行款项
SETT_PROVI	|	结算备付金(金融类)
LEND_CAPITAL	|	拆出资金(金融类)
FIN_ASSETS	|	以公允价值计量且其变动计入当期损益的金融资产
NOTES_REC	|	应收票据
DIVIDEND_REC	|	应收股利
INTEREST_REC	|	应收利息
ACCOUNTS_REC	|	应收账款
OTHER_REC	|	其他应收款
PREPAYMENT	|	预付款项
INVENTORIES	|	存货
NCA_IN_1Y	|	一年内到期的非流动资产
OTHER_CA	|	其他流动资产
SPECIAL_CA	|	流动资产特殊项目
TOT_CA	|	流动资产合计
LOAN_AND_ADVANCE	|	发放贷款和垫款(金融类)
FA_AVAIL_FOR_SALE	|	可供出售金融资产
HTM_INVEST	|	持有至到期投资
INVEST_ESTATE	|	投资性房地产
LT_EQUITY_INVEST	|	长期股权投资
LT_REC_ACCOUNT	|	长期应收款
FIXED_ASSETS	|	固定资产
CONST_MATERIALS	|	工程物资
CONST_IN_PROCESS	|	在建工程
FIXED_ASS_DISP	|	固定资产清理
BIO_ASSETS	|	生产性生物资产
OIL_GAS_ASSETS	|	油气资产
INTAN_ASSETS	|	无形资产
DEV_EXPENDITURE	|	开发支出
GOOD_WILL	|	商誉
LT_PREPAID_COST	|	长期待摊费用
DEFER_TAX_ASSETS	|	递延所得税资产
OTHER_NCA	|	其他非流动资产
SPECIAL_NCA	|	非流动资产特殊项目
TOT_NCA	|	非流动资产合计
TOT_ASSETS	|	资产总计
ST_BORR	|	短期借款
CB_BORR	|	向中央银行借款(金融类)
DEPOS	|	同业及其他金融机构存放款项(金融类)
NOTES_PAYABLE	|	应付票据
ACCOUNTS_PAYABLE	|	应付账款
ADVANCE_PECEIPTS	|	预收款项
SALARIES_PAYABLE	|	应付职工薪酬
DIVIDEND_PAYABLE	|	应付股利
TAXS_PAYABLE	|	应交税费
INTEREST_PAYABLE	|	应付利息
OTHER_PAYABLE	|	其他应付款
NCL_IN_1Y	|	一年内到期的非流动负债
OTHER_CL	|	其他流动负债
SPECIAL_CL	|	流动负债特殊项目
TOT_CL	|	流动负债合计
LT_BORR	|	长期借款
BOND_PAYABLE	|	应付债券
LT_PAYABLE	|	长期应付款
SP_PAYABLE	|	专项应付款
ESTIMATE_LIAB	|	预计负债
DEFER_TAX_LIAB	|	递延所得税负债
OTHER_NCL	|	其他非流动负债
SPECIAL_NCL	|	非流动负债特殊项目
TOT_NCL	|	非流动负债合计
SPECIAL_LIAB	|	负债特殊项目
TOT_LIAB	|	负债合计
PAIDIN_CAPITAL	|	实收资本(或股本)
CAPITAL_RESER	|	资本公积
TREASURY_SHARE	|	减：库存股
SP_RESER	|	专项储备
SURPLUS_RESER	|	盈余公积
RETAINED_PROFIT	|	未分配利润
ORDIN_RISK_RESER	|	一般风险准备
FOREIGN_DIFF	|	外币报表折算差额
TE_ATTRP	|	归属母公司权益合计
MINORITY_INTERESTS	|	少数股东权益
TOT_OWNER_EQUITY	|	所有者权益合计
TOT_LIAB_EQUITY	|	负债和所有者权益合计

### 利润表
fields |说明 
---|---
TOT_OP_REVENUE	|	营业总收入(非金融类)
REVENUE	|	营业收入
INT_NET_INCOME	|	利息净收入(金融类)
INT_INCOME	|	其中:利息收入(金融类)
INT_EXP	|	其中:利息支出(金融类)
COMMIS_NET_INCOME	|	手续费及佣金净收入(金融类)
COMMIS_INCOME	|	其中:手续费及佣金收入(金融类)
COMMIS_EXP	|	其中:手续费及佣金支出(金融类)
TOT_OP_COST	|	营业总成本(非金融类)
TOT_COST	|	营业成本(非金融类)
OP_TAX_SURCHG	|	营业税金及附加
SELL_EXP	|	销售费用(非金融类)
ADMIN_EXP	|	管理费用(非金融类)
FIN_EXP	|	财务费用(非金融类)
ASSETS_IMPAIR_LOSS	|	资产减值损失
FVALUE_CHG_INCOME	|	公允价值变动净收益
INVEST_INCOME	|	投资收益
INVEST_INCOME_ASSOCIATES	|	其中:对联营企业和合营企业的投资收益
CHARGE_INCOME	|	汇兑收益
OPERATE_PROFIT	|	营业利润
NOPERATE_INCOME	|	加：营业外收入
NOPERATE_EXP	|	减：营业外支出
NCA_DISP_LOSS	|	其中：非流动资产处置净损失
TOT_PROFIT	|	利润总额
INCOME_TAX	|	减：所得税费用
NET_PROFIT	|	净利润
NET_PROFIT_ATTRP	|	归属于母公司所有者的净利润
MINORITY_PROFIT	|	少数股东损益
BASIC_EPS	|	(一)基本每股收益
DILUTED_EPS	|	(二)稀释每股收益
OTHER_COMPR_INCOME	|	其他综合收益
OTHER_COMPR_INCOME_ATTRP	|	归属于母公司所有者的其他综合收益的税后净额
TOT_COMPR_INCOME	|	综合收益总额
TOT_COMPR_INCOME_ATTRP	|	归属于母公司所有者的综合收益总额
TOT_COMPR_INCOME_ATTRMS	|	归属于少数股东的综合收益总额

### 现金流量表
fields |说明 
---|---
CASH_RECV_FROM_GS_SERV	|	销售商品、提供劳务收到的现金(非金融类)
NET_DEPOSIT_INCREASE	|	客户存款和同业存放款项净增加额(金融类)
NINC_BORR_OTH_FI	|	向其他金融机构拆入资金净增加额(金融类)
CASH_RECV_FROM_INT_AND_COMMIS	|	收取利息、手续费及佣金的现金(金融类)
CASH_RECV_FROM_OTHER_OP	|	收到其他与经营活动有关的现金
SUBTOT_OPERATE_CASH_INFLOW	|	经营活动现金流入小计
CASH_PAID_FOR_GOODS_AND_SERV	|	购买商品、接受劳务支付的现金(非金融类)
STAFF_PAID	|	支付给职工以及为职工支付的现金
TAX_PAYMENTS	|	支付的各项税费
CASH_PAID_FOR_INT_AND_COMMIS	|	支付利息、手续费及佣金的现金(金融类)
CASH_PAID_FOR_OTHER_OP	|	支付其他与经营活动有关的现金
SUBTOT_OPERATE_CASH_OUTFLOW	|	经营活动现金流出小计
OPERATE_NET_CASH_FLOW	|	经营活动现金流量净额
CASH_RECV_FROM_INVEST	|	收回投资收到的现金
CASH_RECV_FROM_RETURN_ON_INVEST	|	取得投资收益收到的现金
NET_CASH_RECV_FROM_DEAL_ASSETS	|	处置固定资产、无形资产和其他长期资产收回的现金净额
NET_CASH_RECV_FROM_DEAL_SUBCOM	|	处置子公司及其他营业单位收到的现金净额
CASH_RECV_FROM_OTHER_INVEST	|	收到其他与投资活动有关的现金
SUBTOT_INVEST_CASH_INFLOW	|	投资活动现金流入小计
CASH_PAID_FOR_ACQU_ASSETS	|	购建固定资产、无形资产和其他长期资产支付的现金
CASH_PAID_FOR_INVEST	|	投资支付的现金
NET_CASH_RECV_FROM_SUBCOM	|	取得子公司及其他营业单位支付的现金净额
CASH_PAID_FOR_OTHER_INVEST	|	支付其他与投资活动有关的现金
SUBTOT_INVEST_CASH_OUTFLOW	|	投资活动现金流出小计
INVEST_NET_CASH_FLOW	|	投资活动现金流量净额
CASH_FROM_INVEST	|	吸收投资收到的现金
CASH_FROM_MINO_S_SUB	|	其中：子公司吸收少数股东投资收到的现金
CASH_FROM_BONDS_ISSUE	|	发行债券收到的现金
CASH_FROM_BORR	|	取得借款收到的现金
CASH_RECV_FROM_OTHER_FIN	|	收到其他与筹资活动有关的现金
SUBTOT_FIN_CASH_INFLOW	|	筹资活动现金流入小计
CASH_PAID_FOR_DEBTS	|	偿还债务支付的现金
CASH_PAID_FOR_DIST_INT_AND_COMMIS	|	分配股利、利润或偿付利息支付的现金
CASH_PAID_FOR_OTHER_FIN	|	支付其他与筹资活动有关的现金
SUBTOT_FIN_CASH_OUTFLOW	|	筹资活动现金流出小计
FIN_NET_CASH_FLOW	|	筹资活动现金流量净额
EXCHNG_RATE_CHG_EFFECT	|	汇率变动对现金及现金等价物的影响
CASH_EQUIVALENT_INCREASE	|	现金及等价物净增加额
CASH_EQUIVALENTS_AT_BEGIN	|	加：期初现金及现金等价物余额
CASH_EQUIVALENTS_AT_END	|	期末现金及现金等价物余额

## 表格型数据
获取表格型数据方式为：get_table_field(symbol, field, start_date, end_date, count, columns) 
### 期货成交量
** field: **  FUT_TRADE_RANK 

columns|说明 
---|---
rank       | 排名      
volumeDiff | 比上交易日增减/变化 
volume     | 成交量/持买单量/持卖单量
name       | 期货公司会员简称  

### 期货持卖单量
** field: **   FUT_SHORT_RANK

columns|说明 
---|---
rank       | 排名      
volumeDiff | 比上交易日增减/变化 
volume     | 成交量/持买单量/持卖单量
name       | 期货公司会员简称  

### 期货持买单量
** field:** FUT_LONG_RANK

columns|说明 
---|---
rank       | 排名      
volumeDiff | 比上交易日增减/变化 
volume     | 成交量/持买单量/持卖单量
name       | 期货公司会员简称  

### 申万一级行业指数日K行情数据
** field: **  SW_MKT 

columns|说明 
---|---
close    |   收盘价
high      |   最高价
low     |   最低价
open  |   开盘价
preClose  |   昨收价
tradeDate      |   交易日期
turnover        |   成交额
volume  |   成交量