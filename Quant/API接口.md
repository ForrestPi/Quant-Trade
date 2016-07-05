# API接口

考虑到很多用户初次使用时对于各API接口的功能和获取方式存在较多的疑问，这里进行专门介绍。

国内程序化交易技术的爆发式发展几乎就是起源于上期技术公司基于CTP柜台推出了交易API，使得用户可以随意开发自己的交易软件直接连接到交易柜台上进行交易，同时CTP API的设计模式也成为了许多其他柜台上交易API的设计标准，本人已知的类CTP交易API包括：

    上期CTP

    飞马

    华宝证券LTS

    飞创Xspeed

    金仕达

    恒生UFT


------

### CTP

模块：vn.ctp

经纪商：期货公司、兴业证券

产品：期货、期货期权

特点：国内最早的针对程序化交易涉及的API接口

申请：[SimNow](http://www.simnow.com.cn/)网站申请

------

### 飞创

模块：vn.xspeed

经纪商：期货公司

产品：期货、期货期权

特点：交易大商所产品有速度优势

申请：通过[大连飞创](http://www.dfitc.com.cn/portal/cate?cid=1364967839100#1)上的QQ群申请

------

### 飞马

模块：vn.femmas

经纪商：期货公司

产品：中金所的期货、期货期权

特点：交易中金所产品有速度优势

申请：通过[飞马](http://www.cffexit.com.cn/static/3000201.html)上的QQ群申请

------

### LTS

模块：vn.lts

经纪商：华宝证券

产品：证券（股票、债券）、股票期权

特点：国内技术水平最高的柜台之一，机构用户值得拥有

申请：联系华宝证券的客服申请

------

### 金仕达期权

模块：vn.ksotp

经纪商：期货公司

产品：期货、期货期权、股票期权

特点：可以同时交易期货期权和股票期权，便于套利

申请：联系南华期货的客服申请

------

### 金仕达黄金

模块：vn.ksgold

经纪商：贵金属经纪公司、银行

产品：金交所的贵金属现货

特点：部分银行可以提供的程序化接口，结合黄金ETF做套利有优势（非速度优势）

申请：在浦发银行或中国银行开户黄金t+d交易后找客户经理申请

------

### 飞鼠

模块：vn.sgit

经纪商：贵金属经纪公司

产品：期货、金交所的贵金属现货

特点：针对贵金属的期货和金交所现货套利交易有速度优势

申请：联系[招金投资](http://www.au999.cn/)或[恒邦冶炼](http://www.hengbang9999.com/)申请

------

### OANDA

模块：vn.oanda

经纪商：OANDA（美国外汇经纪商）

产品：外汇现货、CFD（全球股指、主要商品）

特点：开户交易的要求非常低（10美金都行），适合新手低成本入门

申请：在[OANDA官网](http://www.vnpy.org/pages/www.oanda.com)开设模拟账户后即可使用，建议选择欧美国家（如英国），选择中国需要发送邮件申请开通API接入

------

### IB

模块：无（vn.trader中使用[IbPy](https://github.com/blampe/IbPy)接入）

经纪商：Interactive Brokers（盈透证券）

产品：全球除了中国以外市场的几乎所有产品（股票、债券、期货、期权、外汇），部分中国股票可以通过沪股通形式参与

特点：一套接口交易全球市场

申请：在[IB官网](https://www.ibkr.com.cn/cn/index.php?f=3416)下载演示客户端后即可使用



## API工作流程

简单介绍一下MdApi和TraderApi的一般工作流程，这里不会包含太多细节，仅仅是让读者有一个概念。

### MdApi

1. 创建MdSpi对象
2. 调用MdApi类以Create开头的静态方法，创建MdApi对象
3. 调用MdApi对象的RegisterSpi方法注册MdSpi对象的指针
4. 调用MdApi对象的RegisterFront方法注册行情柜台的前置机地址
5. 调用MdApi对象的Init方法初始化到前置机的连接，连接成功后会通过MdSpi对象的OnFrontConnected回调函数通知用户
6. 等待连接成功的通知后，可以调用MdApi的ReqUserLogin方法登陆，登陆成功后会通过MdSpi对象的OnRspUserLogin通知用户
7. 登陆成功后就可以开始订阅合约了，使用MdApi对象的SubscribeMarketData方法，传入参数为想要订阅的合约的代码
8. 订阅成功后，当合约有新的行情时，会通过MdApi的OnRtnDepthMarketData回调函数通知用户
9. 用户的某次请求发生错误时，会通过OnRspError通知用户。
10. MdApi同样提供了退订合约、登出的功能，一般退出程序时就直接杀进程（不太安全）

### TraderApi

1. TraderApi和MdApi类似，以下仅仅介绍不同点
2. 注册TraderSpi对象的指针后，需要调用TraderApi对象的SubscribePrivateTopic和SubscribePublicTopic方法去选择公开和私有数据流的重传方法（这一步MdApi没有）
3. 对于期货柜台而言（CTP、恒生UFT期货等），在每日第一次登陆成功后需要先查询前一日的结算单，等待结算单查询结果返回后，确认结算单，才可以进行后面的操作；而证券柜台LTS无此要求
4. 上一步完成后，用户可以调用ReqQryInstrument的方法查询柜台上所有可以交易的合约信息（包括代码、中文名、涨跌停、最小价位变动、合约乘数等大量细节），一般是在这里获得合约信息列表后，再去MdApi中订阅合约；经常有人问为什么在MdApi中找不到查询可供订阅的合约代码的函数，这里尤其要注意，必须通过TraderApi来获取
5. 当用户的报单、成交状态发生变化时，TraderApi会自动通过OnRtnOrder、OnRtnTrade通知用户，无需额外订阅