# rotation-etf50-qim

简单的量化轮动策略+
1.品种:备用两只,只轮动上证50ETF+(510050,规模500亿)和创业板50ETF+(159949,规模近百亿);
2.仓位设计:无脑全仓多空,这样的好处是交易频度低,年均50次以内(平均一周一次);
3.策略:用MA20均线*跟踪上证50指数(000016)和创业板50指数(399673),两个指数收盘价谁相对MA20的比例高(即收女盘价/MA20
1),收盘前或者第二天开盘就无脑梭对应ETF,只买高的那一只。如果都低于MA20就无脑清仓(当然更推荐买银华日利*或者逆回购*吃吃股市
余额宝)。
4.没了。

架构要求：
1、目录结构
src
├── api 外部数据爬取模块，目前支持baostock、tushare、本地存储
├── module
│   ├── backtesting 回溯测试模块
│   ├── data_acquistion 数据获取模块
│   └── trading 交易执行模块    
│       ├── buy_strategies 买入策略模块
│       ├── sell_strategies 卖出策略模块
│       ├── workflow.py 交易流程脚本
├── utils 工具模块

2、
回溯测试模块、数据获取模块、交易执行模块都用单独的脚本运行，例如：
python backtesting.py
python data_acquistion.py
python trade.py
脚本逻辑实现依赖langgraph库，用于构建交易流程的有向无环图。

交易策略分别写入到buy_strategies和sell_strategies目录下，每个策略都应该实现一个名为execute的方法，该方法接受当前的交易数据和当前的持仓信息，返回一个字典，包含策略的执行结果。

3、数据流向
回溯测试模块、数据获取模块、交易执行模块 都要生成对应的csv文件, 放在data目录下，其中：
数据获取模块生成对应的etf数据文件，放在data/etf目录下，文件名为etf_{code}.csv
交易执行模块依赖etf.csv，通过买入策略、卖出策略生成对应的交易文件，放在data/trade目录下，文件名为trade.csv
回溯测试模块依赖trade.csv、etf.csv，通过回测模块生成对应的回测结果文件，放在data/backtest目录下，文件名为backtest.csv

etf_510050.csv格式
date,symbol,MA20,close
2022-01-03,050020,12.50
2022-01-04,050020,12.65
2022-01-05,050020,12.40
...
2023-12-29,050020,15.80

date (日期):
数据类型： 字符串 (String)
格式： YYYY-MM-DD
可读性/扩展性： 同PE数据文件。
symbol (股票代码/ETF代码):
数据类型： 字符串 (String)
格式： ETF的唯一代码。
可读性/扩展性： 同PE数据文件。
close (收盘价):
数据类型： 浮点数 (Float)
格式： 标准浮点数表示。
可读性/扩展性： 清晰表示收盘价。
MA20 (20日移动平均线):
数据类型： 浮点数 (Float)
格式： 标准浮点数表示。


trade.csv格式
trade_id,date,symbol,trade_type,price,quantity,commission,notes
1,2023-01-10,050020,BUY,1.300,1000,0.0013,Initial purchase based on low PE
2,2023-03-15,050020,SELL,1.450,500,0.0007,Partial sell due to PE rebound
3,2023-05-20,050020,BUY,1.400,200,0.0003,Add to position after minor correction

字段说明：
trade_id (交易ID):
数据类型： 整型 (Integer)
格式： 每次交易的唯一标识符，可按顺序递增。
可读性/扩展性： 方便追踪单笔交易。
date (交易日期):
数据类型： 字符串 (String)
格式： YYYY-MM-DD。
可读性/扩展性： 同数据模块。
symbol (股票代码/ETF代码):
数据类型： 字符串 (String)
格式： ETF的唯一代码。
可读性/扩展性： 同数据模块。
trade_type (交易类型):
数据类型： 字符串 (String)
格式： BUY (买入) 或 SELL (卖出)。
可读性/扩展性： 清晰表明交易方向，易于理解。
price (成交价格):
数据类型： 浮点数 (Float)
格式： 实际成交价格。
可读性/扩展性： 重要的回测输入。
quantity (成交数量):
数据类型： 整型 (Integer)
格式： 成交的股数或份数。
可读性/扩展性： 重要的回测输入。
commission (交易佣金):
数据类型： 浮点数 (Float)
格式： 单笔交易产生的佣金金额（或费率，如果固定）。考虑到实际回测中通常是金额，建议直接记录金额。
可读性/扩展性： 回测时计算净收益的关键因素，考虑到手续费对收益的侵蚀。
notes (备注):
数据类型： 字符串 (String)
格式： 可选，用于记录交易触发的策略逻辑、具体原因等，方便后期分析。
可读性/扩展性： 提高了可读性，对策略开发和调试非常有帮助。



backtest.csv 格式
strategy_name,start_date,end_date,total_return,annualized_return,sharpe_ratio,max_drawdown,num_trades,winning_rate,alpha,beta
MyPEStrategy,2022-01-03,2023-12-29,0.35,0.16,1.20,0.10,25,0.65,0.05,0.85
字段说明 (包含常见的量化策略评价指标):
strategy_name (策略名称):
数据类型： 字符串 (String)
格式： 自定义的策略名称。
可读性/扩展性： 便于区分不同的策略回测结果。
start_date (回测开始日期):
数据类型： 字符串 (String)
格式： YYYY-MM-DD。
可读性/扩展性： 清晰指明回测区间。
end_date (回测结束日期):
数据类型： 字符串 (String)
格式： YYYY-MM-DD。
可读性/扩展性： 清晰指明回测区间。
total_return (总收益率):
数据类型： 浮点数 (Float)
格式： 回测期内的总收益率，例如0.35表示35%。
可读性/扩展性： 核心指标。
annualized_return (年化收益率):
数据类型： 浮点数 (Float)
格式： 年化收益率。
可读性/扩展性： 核心指标，便于不同区间策略比较。
sharpe_ratio (夏普比率):
数据类型： 浮点数 (Float)
格式： 风险调整后的收益指标。
可读性/扩展性： 核心风险调整收益指标。
max_drawdown (最大回撤):
数据类型： 浮点数 (Float)
格式： 回测期内的最大资金亏损百分比。
可读性/扩展性： 核心风险指标。
num_trades (交易次数):
数据类型： 整型 (Integer)
格式： 完成的买卖交易总次数。
可读性/扩展性： 指标密集度。
winning_rate (胜率):
数据类型： 浮点数 (Float)
格式： 盈利交易的比例。
可读性/扩展性： 策略有效性指标。
alpha (阿尔法):
数据类型： 浮点数 (Float)
格式： 超额收益指标。
可读性/扩展性： 反映策略跑赢基准的能力。
beta (贝塔):
数据类型： 浮点数 (Float)
格式： 市场风险敞口指标。
可读性/扩展性： 反映策略相对于市场波动的敏感度。
