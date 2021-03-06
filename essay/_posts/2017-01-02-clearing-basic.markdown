---
layout: 	essay
title: 		"账户和账务"
subtitle: 	"支付清结算-1"
date: 		2017-01-02 12:00:00
author: 	"shamphone"
chapter:	"3"
status:		"unfinished"
---

> 到了年底，业务多，有2周多没有更新了。最近开始做清结算相关的业务，因而暂停更新风控的文章，先完成这一篇清结算的入门。 对会计相关内容，理解也不深入，欢迎大家交流，指正。 

搞明白了清结算，你就明白了支付业务是怎么运转的。 从技术上来说，清结算并不是最难的，风控、信用，实施起来比清结算难多了。但从业务的角度来说，清结算可以说是最难理解的支付业务过程了。 它牵扯到支付所有相关的概念。为了降低理解难度，我们从常见的支付行为入手，逐步分析清结算如何进行。

# 支付流程
先说个比较简单的支付场景，用户（姑且称他为小明）用绑定的银行卡（用宇宙第一大行工行为例）来购买某电商公司（老熊公司）的产品。小明需要先在老熊公司网站上完成银行卡绑定的操作，绑卡的过程参见之前的文档[支付系统之绑卡、签约和身份验证](/essay/2016/10/20/account-4-contract)。绑卡以后，就可以使用这个卡来购买商品。 首先是挑选商品和下单，其后是执行支付。下单之前的流程不做介绍， 我们从支付开始，来说明支付过程中的清结算问题。 为了简化，我们先从比较简单的同渠道、公司内购买的场景来开始。商品也先假定为虚拟产品，比如会员卡。 为了实现这个流程，有一些前置的操作需要完成：

-  老熊公司已经对接了工行的快捷支付接口。通过这个接口，可以实现绑卡（签约）、支付、退款、查单等操作。如何对接，参见文档[支付系统之银行卡支付](/essay/2016/10/12/account-3-bank/)。

-  老熊公司已经按照工行的要求，在工行开了对公账户。老熊公司通过工行接口的所有收款、退款等资金往来，都发生在这个账户上。

- 小明在老熊公司的应用中绑定了自己的一张卡，为了简化处理，小明绑定的也是工行的卡，先省略掉跨行结算的步骤。具体的绑卡流程参见[支付系统之绑卡、签约和身份验证](/essay/2016/10/20/account-4-contract)

用户小明在手机或者PC Web上购买了100元的虚拟产品，比如很多公司会使用的会员卡。这里我们先从虚拟物品入手，因为实体物品情况会复杂一点，供应链和物流也是一个大课题，购买实体物品就需要考虑这个问题，而虚拟产品就可以暂不考虑。然后小明在网站上执行下单、支付操作。

老熊公司的支付系统接收到小明的支付操作请求后，系统首先会校验订单是否有问题，然后调用工行快捷支付接口，从用户的工行卡上扣除100元， 用户的工行卡的扣款是实时进行的，也就是说，这个操作完成后，小明查看他的工行余额和流水，会有一笔100元的交易，并且账户余额也减少了100元。
  但是这个钱并不是直接就进入了老熊公司的（结算）账户上的。工行在第二天凌晨会对前天的交易进行清算和结算。在计算收入的同时，也从中扣除掉通道费用，得到最终应该划拨到结算账户上的金额。在这个例子中我们假定手续费按支付金额收费，比例为0.1%。 这一笔交易，支付给工行0.10元，公司收入99.9元。

# 交易流水

用户执行支付后，系统首先需要记录交易流水，流水的内容包括：

- 交易主体：即发起本次交易的出款的用户，一般是记录ID、姓名等信息。

- 交易账户：即用户购买时使用的出款的账户，这是用户在工商银行的卡，实际账户是建立在工行，但在电商系统中，为了便于结算，为这个账户建立一个代理。这个账户在系统中的ID是10001（数据本身无其他含义）

- 交易对手：即出卖虚拟产品的业务部门，一般记录部门的ID、名称等信息。

- 结算收益： 交易对手能够拿到的金额。这里是 支付金额-渠道费用，即99.9元

- 对手账户：即虚拟产品的收款账户，为了便于结算，公司一般会对每项业务设置独立的结算账户。这个账户在系统中的ID是 20001（数据本身无其他含义）。

- 交易渠道：即工商银行的快捷支付，还需要记录渠道的ID， 名称等；

- 渠道结算账号：这也是个代理账号，记录在渠道侧的交易流水。

- 渠道提交时间：请求渠道执行支付的时间；

- 渠道支付时间：渠道一般会在返回的报文中说明本次交易的执行时间。 如果没有，则使用渠道的支付接口返回时间。

- 渠道费率：渠道的手续费，这里假定工行是按支付金额收费，比例为支付金额的0.1%。

- 渠道费用：这里是支付金额*手续费率， 即0.1元。

- 发起交易日期：2016年12月12日 13:00:10，即用户提交订单后，虚拟产品业务调用支付系统接口执行支付的时间。

- 执行交易日期： 2016年12月12日 13:00:11，即支付系统接口调用时间。

- 支付截止日期：必须在此日期前完成支付。

- 订单信息：在本例子中是会员卡，一般需要记录业务方订单ID、名称、内容等信息。

- 订单金额：提交过来的原始订单的金额 100元；

- 支付金额： 用户实际支付的金额，由于没有使用优惠券、打折卡等，这里支付金额等于订单金额，都是100元。

没有使用卡券、没有和合作方分成，这两块内容暂不记录。

交易流水是在完成支付时实时生成的。这个流水信息是后续记账的依据，所以务必在流水中真实记录能收集到的所有的现场信息。 这里从：

1.交易主体，即掏钱的小明

2.交易对手，即收钱的业务方

3.交易渠道，即工行快捷

4.交易商品，即会员卡

角度来多方位全角度的描述这笔交易。大家会注意到这里有不少冗余信息。实际上对交易涉及到所有可能会被修改的信息，比如用户姓名，商品名称，商品价格，都需要在这里留一个快照，以便后续回溯和审核。 

# 会计主体

不用说，这一笔账是老熊公司的账务，不是工行的账务，也不是小明家的账务。虽然这里会有工行和小明的信息，但记账的目的是为了了解和改进老熊公司的经营状况服务的。 老熊公司不是某个大公司的分公司或者子公司，它是一家独立核算的、具有独立的资金和经营业务的单位，从会计学角度来说，他是一个独立的会计实体。

# 会计要素

从概念上来说，所有和钱有关的活动，买会员、用户充值、支付手续费等，都需要记账，这些活动，称之为会计对象。 每个公司都有不同的会计对象，有时候同一类活动，叫法还不一样。如果直接用这些活动内容来记账，那就没法比较每个公司的情况。 比如新浪说我家微博广告收入300万，网易说我家卖猪收入了300万，到底谁家更赚钱？需要有一个记账的标准，让大家分门别类的做记录。对会计对象做规范化的管理，这就引入会计要素的概念。

会计要素是对会计对象进行的基本分类，是会计核算对象的具体化。 如果说会计对象是个Object，则会计要素是定义这个Object的Class。 不同的国家对会计要素有不同的规定。 国际会计准则委员会（IASC）在《编制和呈报财务报表的结构》将会计要素其归类为资产、负债、权益、收益和费用五个要素。美国财务会计准则委员会（FASB）在《财务会计概念公告》中将会计要素归类为资产、负债、所有者权益(净资产)、业主投资、派给业主款、综合收益、营业收入、费用、利润、损失十个要素。
我国《会计准则》将会计要素归类为资产、负债、所有者权益、收入、费用和利润六个要素。 其中资产、负债和所有者权益，是反应公司的财务状况的；它满足如下恒等式：

```hbs
资产=负债+所有者权益
```

收入、费用和利润，是反应公司动态的经营成果的，满足如下恒等式：

```hbs
收入—费用=利润
```

# 会计科目

六大会计要素指明了需要记账的scope，但毕竟粒度还是太大了。为了更详细地了解公司财务情况，引入会计科目来对会计对象进行第二层次的划分。使用IT的语言来说，会计科目其实就是一个分类体系，用来分门别类地记账。 在实现上，他也是一个编号+名称，IT俗称字典表。 从定义上来说，会计科目是指一个涵义明确、概念清楚、简明扼要、通俗易懂的**标准名称**。
会计科目按照经济内容的性质不同，可以分为资产类科目、负债类科目、所有者权益类科目、损益类科目，成本类科目，有些金融企业还有资产负债共同类科目。在每一类会计科目下，还可以继续细分，详细内容可以参考2016年财政部发布的新会计准则。 会计科目和要素之间的关系：
[![会计科目和要素](http://static.cocolian.org/img/in-post/clearing-kemu.jpg)]( http://static.cocolian.org/img/in-post/clearing-kemu.jpg)
会计科目还分为总账科目和明细科目。从IT角度，可以认为总账科目是一级分类，而明细科目则是这个一级分类下的二级、三级，甚至更多级别的详细的科目。 记账时，会同时记录到总账、明细科目。
在电商的支付系统中，一般会设置如下科目：

```hbs
1002 银行存款(资产类)
100201 业务收款
100201001 支付宝收款
100201002 工行收款
100201003 建行收款
1004 服务成本（成本类）
100401 支付平台手续费用
100401001 支付宝手续费
100401002 工行手续费
100401003  建行手续费
6001 主营业务收入（损益类）
6001001 会员卡业务
6001001 游戏业务
```

# 会计账户

账户是指对会计要素的具体内容所作的科学的分类，其包括两方面的内容：账户的名称、账户的用途与结构。会计科目是设置账户的依据，也是账户的名称。 比如对银行存款这个会计科目，也会设置一个对应的银行存款账户，用来跟踪公司在银行存款的变动。 在这个案例中，将设置的账户同会计科目。

# 记账凭证
想想在以前没有电脑的时候，去买公交卡，公交公司阿姨会认真地记录你买的卡的卡号、买卡人的姓名、卡的面值等信息，运气好的时候还会给个发票。 一般来说，阿姨会将购买记录登记到一个账册上，形成记账凭证，并在这里会登记发票号码。在现在高科技时代，这个凭证还是少不了了。
先说明细账，记录内容如下：
[![detail account](http://static.cocolian.org/img/in-post/clearing-detail.jpg)]( http://static.cocolian.org/img/in-post/clearing-detail.jpg)
这里详细记录每一条交易信息，当然，通过计算机系统，可以记录更多详情，包括时间、地点等。

# 会计分录和记账
大家经常看到的记录应该是这样的：

```hbs
借： 银行存款-工行收款  100
      贷： 主营业务收入-会员卡 100
借： 服务成本-工行手续费  0.1
      贷：银行存款-工行收款 0.1
```

如上， 银行存款、服务成本、主营业务收入，属于总账科目，而工行收款、会员卡、工行手续费，属于明细科目。 这里采用的是复式记账法中借贷记账法。 对应的账户结构如下：

[![detail account](http://static.cocolian.org/img/in-post/clearing-account.jpg)]( http://static.cocolian.org/img/in-post/clearing-account.jpg)

借贷复式记账法的特点是：

1. 采用借、贷作为记账符号，建立在会计恒等式基础上，遵循**有借必有贷，借贷必相等**的原则。

2. 账户基本结构是： 左侧为借，右侧为贷。

3.一般采用如上图所示的T行账户的形式来描述。

4. 借贷所代表的增加、减少的含义并不固定，和账户的性质有关。 

借方 | 贷方
-----|--------
资产增加 | 资产减少
权益减少 | 权益增加
成本增加 | 成本减少
收入减少 | 收入增加
费用及支出增加| 费用及支出减少

# 更多问题
作为清结算的入门介绍，这里介绍的是最简单的场景，以此来解释清结算相关的概念，特别是会计学一些从IT角度不容易理解的名词。实际上，这个场景还有很多问题：

- 严格的说，会员卡的收入，还不能立即作为公司主营业务收入。会员卡是预付款项，用户开始使用会员卡，公司需要为这个使用提供服务；用户结束使用会员卡之后，这一笔开支才算是真正落入公司主营业务收入中。
- 会员卡在使用期间，公司针对会员业务的各种开销，要分摊到这一段期间的会员上。将开销分摊到每张会员卡上，计算其使用成本，最终才能够计算出收益。
- 用户会员卡购买的款项并不是立即到对公的结算账户的，一般是T+1结算，也就是第二天银行才会将清算好的资金打到公司结算账户上，这种情况应该如何记账？
- 如果支付过程中使用了代金券和优惠券，那又应该如何考虑？

此外，还有退款、充值等场景的清结算，这些问题都将在本系列的文章中详细介绍。本文仅介绍一些相关的概念， 欢迎大家继续关注后续内容。 

感谢关注。头条的同学记得帮忙点个赞，谢谢。  