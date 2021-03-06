========
快速开始
========
xalpha 可以用来对场外基金和指数进行方便的追踪和研究，
同时可以实现投资情况的汇总管理与数据分析，并且支持一些简单的基金购买策略回测。

本文仅简述最基本的模块使用的一小部分，关于更多的函数和用法尤其是可视化的部分，请参考具体示例 :ref:`demo`。
关于数据结构和内部的模块设计，请参考高级用法 :ref:`advance`。

基金和指数的信息
----------------
使用 :class:`xalpha.info.fundinfo` 来获取场外基金的基本信息和历史每日净值情况。
其中净值以 :class:`pandas.DataFrame`  的格式存储。

.. note:: 
	基金信息只支持按净值法结算的场外基金，也即大部分货币基金不被支持。

代码示例::

	>>> import xalpha as xa
	>>> zzyl = xa.fundinfo('000968') 
	>>> zzyl
	广发养老指数A
	>>> zzyl.info()
	fund name: 广发养老指数A
	fund code: 000968
	fund purchase fee: 0.12%
	fund redemption fee info: ['小于7天', '1.50%', '大于等于7天，小于1年', '0.50%', '大于等于1年，小于2年', '0.30%', '大于等于2年', '0.00%']
	>>> zzyl.price[zzyl.price['date']<='2015-02-27']
	comment	date	netvalue	totvalue
	0	0	2015-02-13	1.0000	1.0000
	1	0	2015-02-17	1.0000	1.0000
	2	0	2015-02-27	1.0123	1.0123

使用 :class:`xalpha.info.indexinfo` 来获取相应指数的每日净值情况。

.. note::
	indexinfo() 对应的指数代码为7位数，其中后六位是正常的指数代码，第一位用来标记市场，0是沪市，1是深市。

代码示例::

	>>> zzyli = xa.indexinfo('1399812')
	>>> zzyli
	养老产业
	>>> zzyli.price[zzyli.price['date']=='2018-08-01']
		comment	date	netvalue	totvalue
	1	0	2018-08-01	7.603842	7524.4807

.. note::
	对应的 indexinfo().price 表中，netvalue 栏为以初始日归一化后的净值，而 totvalue 栏则是真实的指数值。

单一标的交易处理
-----------------
使用 :class:`xalpha.trade.trade` 来处理交易情况。
为了生成交易，需要提供标的类 :class:`xalpha.info` 和交易账单 status table。
账单的具体的数据结构请参考高级用法 :ref:`advance`。

代码示例::

	>>> yyws = xa.fundinfo('001180') # 交易标的信息
	>>> statb = xa.record(path).status # path位置的交易账单csv
	>>> yyws_t = xa.trade(yyws, statb) 
	>>> yyws_t.dailyreport()
	{'currentshare': 630.39,
	 'currentvalue': 504.12,
	 'date': datetime.datetime(2018, 8, 5, 0, 0),
	 'originalvalue': 523.86,
	 'returnrate': -3.7682,
	 'unitcost': 0.831,
	 'unitvalue': 0.79969999999999997}
	>>> yyws_t.xirrrate('2018-08-01')
	-0.01764033506484772

基金投资组合的管理分析
----------------------
使用 :class:`xalpha.multiple.mul` 可以将多个基金交易类归总，或者根据 status 表格上记录的基金代码自动汇总。
如果选择 :class:`xalpha.multiple.mulfix` 归总交易情况的话，则所有交易视作封闭系统，资金进出由虚拟的货币基金调节。
代码示例：

	>>> invclose = xa.mulfix(yyws_t, totmoney = 6000)
	>>> invclose.combsummary()
		基金代码	基金名称			基金成本	基金收益率	基金现值
	0	001180	广发医药卫生联接A	523.86	-3.7682	504.12
	1	mf		货币基金			5476.15	7.3475	5878.51
	2	xxxxxx	总计				6000.01	6.3770	6382.63
	>>> invopen = xa.mul(status=xa.record(path).status)
	>>> invopen.combsummary('2018-07-01').iloc[-1]
	基金代码      xxxxxx
	基金名称          总计
	基金成本     2379.52
	基金收益率    -4.2559
	基金现值     2278.25
	Name: 5, dtype: object
	>>> invopen.xirrrate('2018-07-01')
	-0.05594572489624858

基金交易策略与回测
------------------
通过额外导入 policy 模块，使用 :class:`xalpha.policy.policy` 的子类，进行按一定策略的模拟交易的 status 表格生成，
从而可以进行相关的交易分析，起到策略回测比较的作用。对应类的 `self.status` 属性即为相应策略的 status 交易表格，
可以用于上述的交易分析使用。
代码示例：

	>>> st = xa.policy.buyandhold(yyws,'2016-01-01') # buy and hold from 2016-01-01, 且始终分红再投入
	>>> st2 = xa.policy.scheduled(yyws, totmoney = 1000, times=pd.date_range('2016-01-01','2018-06-01',freq='W-THU')) # 定投 status 的生成：从2016-01-01 到 2018-06-01 每周四进行定额定投 1000 元。

交易策略的监视和定时提醒
--------------------------
使用 :class:`xalpha.realtime.review` 可以实现策略的监测和邮件的发送，具体使用可以查看 `示例 <https://github.com/refraction-ray/xalpha/blob/master/doc/samples/notification.py>`_

功能综述
----------------
鉴于此页仅涵盖了非常小一部分功能的展示，除了参考其他部分学习外，这里整理出了该模块的基本功能。
    1. 全部场外基金（包括货币基金）的信息获取：指定一个代码，你就能了解的基金名称，历史单位净值，历史分红送转情况，基金的折扣申购费，基金的不同持仓时长的赎回费等多样的信息。
    2. 全部 A 股指数的信息获取：同样是一个代码，获取指数名称和每日净值。
    3. 所有基金指数数据支持增量更新，csv 文件和数据库 io 的无缝支持
    4. 可以对多只基金和指数同时进行量化分析，给出走势分布和相关性分析。
    5. 虚拟可调的货币基金类型：除了前述的真实货币基金类外，还可以建立虚拟的货币基金类，来模拟理财等的行为，或单纯作为量化的基准，可以实现更灵活的仓位管理。
    6. 只需最简的账单外加一个代码就可以精确模拟一只基金你的全部交易行为，并可以输出各种量化数据和可视化。
    7. 大量基于回测的量化数据和基于趋势交易的技术面指标工具箱。
    8. 只需一个最简的账单，就可实现多基金投资系统的投资精确模拟，同时提供总金额固定和总金额变动两个选项，可以显示全部基金投资的总结表和多样的持仓与交易量化，包括折线图，河流图，饼图，柱形图等。所有可视化均为可交互的 web 级可视化方案。
    9. 可以非常简便的制定各种基于日期和点数的定投策略，包括变额定投和复杂的网格策略均可以一行完成，并进行详细的回测分析与可视化展示。
    10. 可以基于净值或各种技术指标的交叉，点位设计复杂的交易策略，并回测效果进行定量分析。
    11. 可以根据自定义的策略，建立邮件按时提醒脚本，从此实现按计划买入和对市场的实时监控，尤其适合复杂网格策略的执行，不需要自己再去看盘和计算执行条件和金额。

