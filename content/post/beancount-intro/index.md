+++
title = "beancount 简易入门指南"
author = ["chi"]
date = 2018-10-25T12:25:00+08:00
lastmod = 2019-12-04T18:54:57+08:00
tags = ["beancount", "记账"]
categories = ["实用功"]
draft = false
toc = true
+++

beancount 是一个基于文本、命令行的复式记账软件，上周看到了 [wzyboy](https://wzyboy.im/post/1063.html) 介绍这个工具以及复式记账的基础概念的安利文。花了一点时间入坑之后，自己的资产（存款）、收入、消费去向，一目了然。看到食物消费和打车话费在支出的 TreeMap 里占据的两大块，不禁让我反思起平时的好吃懒做。

<!--more-->

如果你跟我类似：

-   靠月工资流水生存，有强烈的意愿、需求理清自己的财务状态，搞清楚钱从哪里来、花到哪里去、留下了多少，并借机改善；
-   虽然平时也用过鲨鱼记账、随手记之类的 App，但是觉得每一次消费后掏出手机、打开 App、填写金额时间用途，是一个很重的中断行为；也懒得定期手工录入一批账单；
-   绝大多数消费最终都由银行账户结算，例如通过支付宝绑定信用卡快捷支付，最终只需要统计银行账单即可；
-   懂一点编程（简单的 Python 基础即可），愿意付出一点时间学习；

那么，beancount 这个工具会是一个比较合适的开始。

本文将从一个新手了解了基础概念之后，如何迈出第一步，把现存的财务状态映射到 beancount 里的角度，介绍一下个人的实践经验。


## 背景知识 {#背景知识}

后文假设读者已经读过下列文档、或者熟悉下面文档中提及的工具和概念。

-   [Beancount —— 命令行复式簿记](https://wzyboy.im/post/1063.html) wzyboy 的安利文，中文文档

如果有时间，推荐再读一下 beancount 作者写的几篇文档：

-   [Beancount - The Double-Entry Counting Method](https://docs.google.com/document/d/100tGcA4blh6KSXPRGCZpUlyxaRUwFHEvnz%5Fk9DyZFn4/edit)
-   [Beancount - Tutorial & Example](https://docs.google.com/document/d/1G-gsmwK551lSyuHboVLW3xbLhh99JfoKIbNnZSJxteE/edit)

还有更多的细节、示例可以参考这个索引[文档](https://docs.google.com/document/d/1RaondTJCS%5FIUPBHFNdT8oqFKJjVJDsfsn6JEjBG04eA/edit)。


## 目录结构 {#目录结构}

使用如下所示的目录结构：

```nil
~/Documents/accounting
├── documents
│   ├── Assets/
│   ├── Expenses/
│   ├── Income/
│   └── Liabilities/
├── documents.tmp/
├── importers
│   ├── __init__.py
├── yc.bean
└── yc.import
```

-   ~/Documents/accounting: 项目的根目录，放在任何想放的地方;
-   documents: 用于导入第三方数据后，使用 bean-file 归类存档原文件，执行 `mkdir documents/{Assets,Expenses,Inconme,Liabilities}` 创建这些目录;
-   documents.tmp: 用于临时存放待导入的第三方数据，比如银行卡账单，可以是其他路径，比如 `/tmp`, bean-extract, bean-file 这些脚本都要用到这个目录;
-   importers: 用于存放自定义的导入脚本，目前自带的导入器足够使用，可以先不管这个目录;
-   yc.import: 实际上是一个 Python 脚本，用于定义导入器的配置，后面细说;
-   yc.bean: 是实际的账簿文件，账户、交易记录都在这里，生成报表也使用这个文件,可以将账簿文件拆分成多个小文件，再使用 `include` 指令拼接，类似 C 语言或者 Python 里的 `import`;

<details>
<summary>
单文件账簿还是拆分多文件账簿？
</summary>
<p class="details">

-   刚开始建议用一个 `.bean` 文件管理所有的记录，熟悉工具的使用流程、有了明确的需求之后再拆分;
-   如果使用 emacs 的 orgmode 编辑账簿文件，建议一直使用一个 `.bean` 文件，非常好用;
</p>
</details>

刚开始使用，只需要关注主账簿文件 `yc.bean` 就行，我们来一探究竟吧。


## 开设账户 {#开设账户}

我的 `yc.bean` 文件顶层有三部分: Options, Accounts, MonthlyReconciliation，分别对应账簿文件的选项，账户，每月对账。


### Options {#options}

设置账簿的 title，定义账簿里会用到的货币。

```nil
\* Options
option "title" "My Personal Ledger"
option "operating_currency" "CNY"
option "operating_currency" "USD"
```


### Accounts {#accounts}

有五种账户类型: Assets,Liabilities,Equity,Income,Expenses。分别对应资产、负债、初始化账簿时已有的数据、收入、支出，详细含义可以看上面提及的推荐阅读文档里。

在 benacount 里会隐式创建树形账户，也就是如果开了一个账户叫做： `Assets:Bank:BoC:CardXXXX`, 那么会自动生成账户 `Assets:Bank:BoC`, `Assets:Bank`, `Assets` 。我的做法是原则上用现实世界里的最细分的账户映射 beancount 里的账户，结合账户的实际用途设置账户名。


#### 如何选择账户初始日期？ {#如何选择账户初始日期}

偷懒的话可以选择 1970-01-01。

我的做法是：Assets 类账户选择我开始使用 beancount 的日期，Liabilities、Expenses 账户用生日，Income 选择当前这份工作的日期。


#### Assets {#assets}

假设我在招商银行有两张储蓄卡，其中一张开通了朝朝盈的理财服务并且用于日常消费，另一张卡用于每月定额存款，积累资金用于凑购房首付，那么我会这样设置 Assets 账户(XXXX 是卡号末四位，下面同理)：

```nil
1970-01-01 open Assets:Bank:CMB:CardXXXX:Deposit CNY
1970-01-01 open Assets:Bank:CMB:CardXXXX:ZZY CNY
```

对于存款卡，因为只用于特定用途，不会挪作他用，还有别的账户里也有存款用于同样的用途，比如政府的住房公积金，那么我这样设置账户：

```nil
1970-01-01 open Assets:Saving:HouseFund:Bank:CMB:CardXXXX:Deposit CNY
1970-01-01 open Assets:Saving:HouseFund:Goverment CNY
```


#### Liabilities {#liabilities}

假设我在招商银行有一张银联信用卡，一张 Visa 信用卡；在交通银行有一张银联信用卡，一张 Vsia 信用卡。由于招商银行共享额度、合并账单、征信内只有一个账户；交通银行虽然也共享额度，但是拆分账单，每个账单要单独还款，并且在征信系统内一卡一账户，我这样设置账户：

```nil
1970-01-01 open Liabilities:CreditCards:CMB CNY
1970-01-01 open Liabilities:Creditcards:COMM:CardVisaXXXX CNY
1970-01-01 open Liabilities:Creditcards:COMM:CardUnionXXXX CNY
```

这样既可以既可以对单个账户断言 balance，也可以对单个银行对断言 balance。


#### Income {#income}

工资收入可以设置账户 Income:CompanyName:Salary 就行, 如果有饭补、报销之类的，可以单写 Income:CompanyName:FoodSubsidy, Income:CompanyName:Reimbursement.
这里用 event 指令，可以记录下哪天加入公司，比如 `2018-01-01 event "入职 XX"` 。


#### Expenses {#expenses}

基本原则同上，我在 Expenses 分类下设置了如下几种账户：

-   政府相关的：主要是五险一金、税之类。

<!--listend-->

```nil
1970-01-01 open Expenses:Government:Pension CNY
1970-01-01 open Expenses:Government:Unemployment CNY
1970-01-01 open Expenses:Government:MedicalCare CNY
1970-01-01 open Expenses:Government:IncomeTax CNY
```

-   日常消费，按照衣食行分了几大类，可以包含交通、食物、下馆子、日用杂物、买书、订阅（软件、VPS之类）以及宠物的支出。基本都在三级以内，再通过交易的 [tag](https://docs.google.com/document/d/1wAMVrKIA2qtRGmoVDSUBJGmYZSygUaR0uOMW1GV3YE0/edit#heading=h.2xx3dcvvf0r8) 标记消费的具体支出，比如食物相关的交易记录会打上这些 Tag：早餐、日常饮用水、饮料、零食等等，可以按需使用，最终在 fava 生成的网页里可以按照 tag 过滤查看。
-   住的消费相对固定，并且因为是在北京租房，也是一笔不小的支出，单独开设一类账户用来管理，建议使用当前住宿房屋的简称，比如：Expenses:Lofter0817:Rent, Expenses:Lofter0817:Utility。


#### Equity {#equity}

目前我只设置了一个 Equity 账户 Equity:Opening-Balances，用来平衡初始资产、负债账户时的会计恒等式。也就是，我想往一个银行卡账户里添加 1000 元，并且想保持平衡，那么需要从某个账户减 1000 元，在初始化时，这个账户就是 Equity:Opening-Balances。一个示例：

```nil
1970-01-01 open Assets:Bank:CMB:CardXXXX CNY
1991-05-21 pad Assets:Bank:CMB:C6698 Equity:Opening-Balances
2018-10-17 balance Assets:Bank:CMB:C6698 11912.77 CNY
```


### Balance {#balance}

设置了账户之后，要把对应的现实账户的状态反应出来，需要用 `balance` 指令进行断言操作，用 `pad` 指令进行辅助。比如在设置账户的当时，银行卡内有存款 1000 元，可以在 `open` 账户那行之后添加变成下面的结构，注意 beancount 默认交易都在一天的开始发生，所以 balance 断言要写在第二天，表示截止到第二天零点的情况。

```nil
1970-01-01 open Assets:Bank:CMB:Card0817
1970-01-01 pad Assets:Bank:CMB:Card0817 Equity:Opening-Balances
1970-01-02 balance Assets:Bank:CMB:Card0817 1000 CNY
```

其他账户依照此方法设置即可。


## 导入数据 {#导入数据}

除了账户和 balance 断言， `.bean` 文件里大部分内容是一笔笔交易记录，一个笔交易在 beancount 里一般长这样：

```nil
2018-10-22 * "描述"
  card: "CardXXXX"
  date: 2018-10-21
  Liabilities:CreditCards:CMB  -1921.00 CNY
  Expenses:Other
```

2018-10-22 是银行记帐日期，"\*" 号表示交易确认无误，接着是交易描述；后两行是 metadata，可以用于过滤；接下来是交易涉及的账户，有减操作的账户，就有加操作的账户，这里 Expenses:Other 账户没有写加金额，是因为加操作只涉及这一个账户，beancount 会自行补齐数据。更详细的可以参考 [Beancount Language Syntax](https://docs.google.com/document/d/1wAMVrKIA2qtRGmoVDSUBJGmYZSygUaR0uOMW1GV3YE0/edit#) 。

每笔交易都这么手写一遍就太低效率了，还好 beancount 支持从导入第三方数据，前文提到的 `importers` 目录内就可以用来存放自定义的导入脚本，不过自带 csv 导入器就可以解决目前绝大部分需求。


### 获取数据 {#获取数据}

目前国内部分银行提供 csv 各式的对账单，比如招商银行可以登录个人网银后找到对账单下载；也有银行不提供 csv、Excel 各式的对账单下载，可以尝试下面两个方法：

-   如果银行提供网页版对账单，并且账单页面内容是 html table，可以使用 Chrome 插件[ Table-Capture](https://chrome.google.com/webstore/detail/table-capture/iebpjdmgckacbodjpijphcplhebcmeop) 把页面里的 table 导出到 Google Spreadsheet，再导出为 csv;
-   银行应该都会提供 pdf 各式的对账单，可以尝试用 [Tabula](https://tabula.technology/) 这个工具，从 pdf 文件里解析账单表格并导出;

经过测试，以上两个方法能够搞定招商、交通、中信、浦发这四个银行账单。


### 准备数据 {#准备数据}

获取到 csv 各式的数据后，需要做一些准备工作：

-   去除文件里的奇怪的符号，比如交通银行的账单里会包含 `^M` 这个符号，用 `C-c C-m` 可以在终端里敲出这个字符；
-   金额改为只保留数字部分；
-   把文件编码转换为 utf-8: `iconv -f gbk -t UTF-8 file > file.utf-8` ；
-   转换文件的换行方式: `dos2unix file.utf-8` ；


### import 配置 {#import-配置}

我的 import 配置文件 `yc.imoprt` 抹去敏感信息之后示例如下下方的代码。

```python
#!/usr/bin/env python

import os
import sys

import beancount.ingest.extract
from beancount.ingest.importers import csv

beancount.ingest.extract.HEADER = ''

CONFIG = [
    # CMB Credit
    csv.Importer(
        {
            csv.Col.DATE: '记账日期',
            csv.Col.TXN_DATE: '交易日期',
            csv.Col.NARRATION1: '交易摘要',
            csv.Col.AMOUNT_DEBIT: '人民币金额',
            csv.Col.LAST4: '卡号后四位'
        },
        account='Liabilities:CreditCards:CMB',
        currency='CNY',
        regexps='\t对账标志',
        last4_map={
            "0000": "招行 0000",
        },
        # categorizer=guess.guess2
    ),
    # COMM Credit 0000
    csv.Importer(
        {
            csv.Col.DATE: '记账日期',
            csv.Col.TXN_DATE: '交易日期',
            csv.Col.NARRATION1: '交易说明',
            csv.Col.AMOUNT_DEBIT: '清算币种/金额',
            csv.Col.LAST4: '卡号末四位'
        },
        account='Liabilities:CreditCards:COMM:C0000',
        currency='CNY',
        regexps='交行0000',
        skip_lines=1,
        last4_map={
            "0000": "交行 0000",
        },
        # categorizer=guess.guess2]
    )
]
```

csv.Col.XXX 对应的是 csv 文件的 header，新加账户、账单的话对照修改就行。整体执行流程大约是，对于一个待导入文件：

1.  每个 importer 判断自己是否会处理这个文件，如果会处理，交给这个 imoprter 处理导入，并不再往下判断；csv importer 是通过 regexps 参数里指定的正则匹配整个文件内容，看能否匹配上。
2.  由于交行（其他银行也有可能）一卡一账单，账单的头部都一样，我在 csv header 下面插入一行 “交行0000”（0000是卡号末四位）来标记此文件是哪张卡的账单，应该对应到哪个账户，再配置 skip\_lines 参数，在实际导入的时候跳过这一行。
3.  last4\_map 会匹配卡号末四位，生成 `card: 交行 0000` 写到交易的 metadata 里。


### 执行导入 {#执行导入}

把准备好的账单文件放到上面提及的 documents.tmp 目录里，再执行:

```bash
bean-extract yc.import ${PWD}/documents.tmp > tmp.bean
```

我习惯先把记录先导出到临时账簿文件里，检查一下交易记录、修正一部分交易描述、添加 Expenses 账户，再导出到总账簿文件里。

添加 Expenses 账户这一步可以尝试自定义 categorizer 来实现自动化，比如交易描述里包含“饿了么”自动归到 Expenses:Food 账户里。我还没有实现这部分，可以参考这个 [Pull Request](https://bitbucket.org/blais/beancount/pull-requests/24/improve-ingestimporterscsv/diff)。

导入完成后，再执行下面的命令，把原文件归档到 documents 目录里。

```nil
bean-file yc.import ${PWD}/documents.tmp -o documents
```


## 我的工作流 {#我的工作流}

目前我的大部分支出会落到信用卡里，少量走借记卡，极少现金。信用卡出账单日也统一到一两天之内。整体工作流程大概是这样：

1.  每月最后一个账单出来后，整理好账单文件，用 bean-extract 导入账单；
2.  对 Liabilities 账户进行 balance 断言；
3.  在还款日前还款后，对 Assets 账户断言；
4.  发薪日再次对各类账户进行一次断言；
5.  每月检查个账户的错误情况，fava 生成的网页上有一个 Errors 子页面；回顾支出情况；


## 总结 {#总结}

开始说要记账、规划自己的财务状况有半年多了，断断续续用过几款 App，都没有能完全坚持下来，直到在 wzyboy 的博客上看到 beancount 工具的安利，有如开挂一样，个人的财务状况从整体到细节都能看的清楚，也是我喜欢的纯文本工具，信息不会留在第三方、方便各种编辑、导入导出、备份。

在入门上手期间，通过邮件向 [wzyboy](https://wzyboy.im/) 请教了不少疑问，都得到了细致及时的解答，表示感谢。
