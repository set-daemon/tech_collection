# DSP整体架构
***set_daemon@126.com 2018-04-16***

## 架构图
![架构图](attachments/DSP_entire_architecture.png)

### Delivery Unit
投放单元，通过Nginx负载接入竞价流量，竞价请求为长连接。加入zookeeper集群管理。
1. 处理流程
  流量适配、用户识别、地域识别、流量过滤及限流、人群查询、广告筛选、频次控制、点击率预估、价格预估、预算控制、广告Rank，竞价拼包、响应，记录日志。
2. 外部请求
* 用户识别 -- 比如PC/WAP流量，通过cookiemap数据查询对应的内部ID，这时会发起网络请求；而移动端用户，通过常见的AndroidId、Mac、IDFA和IMEI，本地处理即可。
* 人群查询 -- 使用用户ID通过网络查询人群信息，如demography、兴趣爱好，是否为可疑的作弊用户。
* 广告筛选 -- 理应通过一个分布式索引器做广告筛选。
* 频次控制 -- 查询用户在多个广告（非广告组和计划）上的当日投放次数，如果超过限制，则过滤掉该广告。
* 价格预估 -- 可能要查询计算出来的推荐广告位可能的竞价成功价格
* 预算控制 -- 请求预算控制器判别当前的广告是否可投
  以上请求均会产生IO等待，即可以在这些点上做异步处理。

3. 框架/工作单元
  动态配置管理线程、请求接入线程、请求处理线程、广告缓存线程、预算控制器监听线程构成。

4. zookeeper中的集群数据结构
```
|---deliveries
     |--- 01 (数据：{ip:"", port:"", id:""})
```

### Budget Controller
预算控制器，为投放单元提供实时预算控制。

1. 处理流程
* 优先处理控制消息，如广告主账户余额变化、计划预算变化、计划状态变化等，影响投放的信息。
* 其次是预算判别请求，由投放单元发来的请求，携带信息如[计划ID,出价]，返回[计划ID,是否可投]。
* 最后是实时推送/消耗消息，在内存中累积缓存起来，为同步数据做准备。

以上是由消息处理线程顺序处理，防止加锁。

2. 外部请求
  无。

3. 框架/工作单元
  三种消息接入线程分别监听和接收端口数据，通过三个队列发送给顺序处理线程，并由一个缓存线程将数据刷入磁盘，防止因意外丢失数据。

4. 控制逻辑
* 判别是否该计划出价是否可行的依据是，计划是否正常、全局投放开关是否开启、“伪push量”是否超过预算和广告主账户余额，如果未超过，将该出价累计到“伪push量上”，并反馈结果给对应的投放单元，如果原来有锁，去锁，否则，“锁定”该计划状态，并记录锁定时间。
* 收到控制信息中的同步真实push数据，如果锁定时长超过5秒，则同步真实push数据到“伪push量”，并比较真实推送数据是否已经达到限额，如果是，加“二重”锁定。
* 收到控制信息中的同步真实曝光数据，如果加的是“二重锁定”，且时长超过2分钟，则同步真实曝光数据到真实push数据以及“伪push量上”，如果此时都达到限额，则标识为“绝对锁定”，否则解锁。
* 收到刷数据至磁盘的消息，则将全量数据拷贝推送给刷磁盘线程。

5. 输入/输出
* 控制消息
  tcp承载，二进制数据，格式为：请求类型(1byte)、请求时间(4bytes)、请求ID(4bytes)、请求源类型(1byte)、请求源标识(4bytes)、数据长度(2bytes）、请求数据（不定，看具体请求）、校验值(4bytes，暂不使用)。

| 类型值  | 类型说明     | 请求数据                                     | 说明                     |
| ---- | -------- | ---------------------------------------- | ---------------------- |
| 1    | 广告主状态更新  | 广告主ID(4bytes)、状态(1byte)                  | 无响应                    |
| 2    | 广告主余额更新  | 广告主ID(4bytes)、余额(4byte，值*10000，整数)       | 无响应                    |
| 3    | 计划状态更新   | 计划ID(4bytes)、状态(1byte)                   | 无响应                    |
| 4    | 计划预算更新   | 计划ID(4bytes)、总预算(4byte，值*10000，整数)、天预算(4byte，值*10000，整数) | 无响应                    |
| 5    | 同步数据请求   | 空                                        | 即将锁定的计划的真实数据刷到伪数据上；无响应 |
| 6    | 数据刷入磁盘   | 空                                        | 将变化的数据更新到磁盘；无响应        |
| 7    | 系统可投全局开关 | 状态(1byte,0或1)                            | 系统是否可投放                |

* 预算判别请求
   tcp承载，二进制数据，格式为：请求类型(1byte)、请求时间(4bytes)、请求ID(4bytes)、请求源类型(1byte)、请求源标识(4bytes)、个数(1byte)、[计划ID(4bytes)、出价价格(4byte，值*10000)]
   响应数据格式：响应类型(1byte)、响应时间(4bytes)、响应ID(4bytes)、响应源类型(1byte)、响应源标识(4bytes)、个数(1byte)、[计划ID(4bytes)、允许出价(1byte，0或者1)]

* 推送/消耗消息
  tcp承载，二进制数据，格式为：请求类型(1byte)、请求时间(4bytes)、请求ID(4bytes)、请求源类型(1byte)、请求源标识(4bytes)、预算控制器id(1byte)、个数(1byte)、[广告主ID(4bytes)、计划ID(4bytes)、push价格(4bytes，*10000)、曝光价格(4bytes，*10000)]


以上数据接口暂时自行解析，以后可采用thrift或protobuf来实现。

6. 部署
  可多个实例部署，投放单元查询请求可均衡负载到这些控制器，并将推送与控制器ID关联，发生真实推送前，将推送信息发送给对应控制器，且在实时流统计时，也将来自该控制器的推送和曝光数据发送到对应的控制器上。

下发器根据控制器个数，平均分配预算给控制器。

7. 问题
* 可能会出现有的控制器提前消耗完预算，而有的无法消耗完，在投放单元使用Round Robin均衡方法可尽可能消除影响。
* 如果升级，则需要先全局停广告后，等待5分钟左右；
* 如果某个控制器出现异常，则向该控制器传递数据的模块需要感知，并且将数据缓存等下次启动；或者将数据放入消息队列中等待启动时消费。

### Budget Distributor
预算分配器，根据业务系统的数据变化，分配预算给预算控制器。

1. 处理流程
  监听预算控制器的变化（zookeeper），如果发生变化，则读取缓存中的广告主和广告计划信息，如账户余额、预算，根据当前的预算控制器进行平均分配。
  监听业务数据变化事件（kafka），变更相应计划/广告主在预算控制器的分配。

2. 框架/服务单元
  监视预算控制器的线程，以及监听业务事件线程。

3. 输入/输出
  3.1 输入
* 监听zookeeper上的`/budgetors`路径。
  -- 即当前预算控制器的活跃列表
* 监听读取kafka的topic`businessEvent`
  -- 广告主余额变更：{cmd:sponsor_balance_update, sponsor_id:1333, newBalance:30.0}
  -- 广告主状态变更：{cmd:sponsor_status_update, sponsor_id:1333, newStatus: 1}
  -- 计划预算变更：{cmd:plan_budget_update, plan_id:4444, newDayBudget: 33344, newBudget:44444}
  -- 计划状态变更：{cmd:plan_status_update, plan_id:4444, newStatus: 0}
* 读取缓存（redis）上的计划、广告主信息
  -- 今日投放的广告主集合，sponsors:{day} 
  -- 今日投放的广告计划集合，plans:{day}
  -- 计划状态：plan:{plan_id}，包括属性有，status、total_budget、day_budget
  -- 广告主状态：sponsor:{sponsor_id}，包括的属性有：status、balance

4. 部署
  单节点部署，性能上可以保证。后台运行，可加入zookeeper管理，方便运维监控。

### tempSync
临时同步器，实现小batch数据同步到BudgetController，降低接收端的压力。

1. 处理逻辑
监听消耗消息事件（kafka topic impEvent），读取后做小时间段的batch（大约5秒钟）数据聚合（以计划、控制器ID为key），根据推送决策源（即由哪个控制器决定的当次推送）发送至对应的预算控制器。如果选择的控制器不工作，则将数据在本地缓存，待检测到该预算控制器工作后，回补数据。

2. 输入/输出

3. 问题
* 由于预算控制器不返回响应，存在数据未被正常处理的风险。
* 该同步器的数据源为消息队列（kafka），所以如果出现单点故障，只要快速（30分钟内）解决，对投放影响不会太大。

### Monitor Unit
监控单元，监控数据项有：bid win, ad impression, ad click。使用zookeeper做集群管理。
1. 处理流程
  根据URL中的特征，识别请求平台类型，获取平台处理器，处理请求，记录日志，发送日志。

2. 外部请求
* 发送日志 -- 发送至诸如kafka的消息队列。

3. 框架/服务单元
一个请求接收线程，多个服务处理线程（一生产者，多消费者）。

4. 部署
集群部署，前置nginx做负载，由于nginx的可靠性，其负载均衡的服务器健康检查可避免单点故障。当检测到出现异常，部署的检查器迅速将其拉起。


### Cookemap Unit
cookiemap单元，与其它平台/媒体的PC/WAP端做cookie交换共识。
1. 处理逻辑
由于需要做多个平台的cookie打通，所以只要有机会，都需要主动向其它未做过映射的平台都请求一遍。
在自有媒体或客户媒体上先建立好cookie或cookie映射，然后收到曝光/点击/竞价成功请求后，主动在别的媒体上与其它平台做cookiemap（逐个302跳转），这样，未经自有媒体、客户媒体同意的情况下，用户数据不会暴露给其它平台。

2. 输入/输出

3. 框架/服务单元
接入请求线程，多个请求处理线程。

4. 部署
集群部署，前置nginx做负载均衡；后端存储使用redis集群，将key做sharding。

### Log Aggregator
日志聚合器，将线上的投放数据拉取、聚合并上传至Hadoop存储。

1. 处理逻辑
从投放单元、监控单元服务器将上一个小时的数据拉取到本地，将数据过滤成两份，上上个小时以及上个小时的：
```
	将上上个小时的旧数据与新过滤出来的数据，合并后，再独立上传（hadoop上的旧数据删除）；
	将上小时的数据上传至hadoop
```
处理完毕后，向消息队列的topic dataProcChn发送类似消息:{msg:data_upload, dname:/data/20180417/ad_show_17.log}、{msg:data_upload_update, dname:/data/20180417/ad_show_16.log}。

2. 问题
* 为什么不将数据做5分钟切割
碎片文件过多，业务量小的时候，5分钟切割不合适，而且没有考虑实时结算，所以暂时采用每小时切割上传。

* 为什么数据存磁盘，而不是通过消息队列实时上报
对于关键性的业务数据，走网络仍旧存在风险，而且数据量目前不是太大，磁盘压力不大。

* 多次删除数据对hadoop的影响会不会很大
删除应该会导致碎片增多，一天相当于删除23次，肯定会有影响。替代的方案可以做往hadoop块上append数据，但因为出现过问题，所以使用删除再上传的方案。

### Issuer
业务数据下发器，连接投放系统和业务系统。
1. 处理逻辑
从消息队列中得到业务系统数据变更事件，根据事件的具体对象做对应的处理：
* 广告主锁定/解锁
	-- 向Budget Distributor发送信息
	-- 修改广告主下面所有广告的状态（广告缓冲池）
* 广告主账户余额变更
	-- 向Budget Distributor发送信息
    -- 修改广告主控制信息（更新余额）
* 计划锁定/解锁
	-- 修改计划下面所有广告的状态（广告缓冲池）
	-- 向Budget Distributor发送信息
* 计划变更预算
	-- 向Budget Distributor发送信息
	-- 修改计划下面所有广告的预算数据
    -- 修改计划控制信息（更新预算）
* 广告组锁定/解锁
	-- 修改广告组下面所有广告的状态（广告缓冲池）
* 广告组策略变更
	-- 修改广告组下面所有广告的策略信息（广告缓冲池）
* 广告锁定/解锁
	-- 修改对应广告的状态（广告缓冲池）
* 广告修改
	
* 广告审核通过
	
2. 部署
业务修改量不大，性能不会成为问题，单点足以满足。如果修改的业务量大时，可以考虑按广告主做key，部署多个节点服务。外加zookeeper管理，方便运维监控。

3. 问题
* 单点故障的影响
	如果出现单点故障，需要迅速拉起，否则不能及时反馈业务需求。


### Feedbacker
反馈器，将线上投放情况反馈至业务系统。
接收到“反馈”消息（大约1分钟产生一次，该消息产生的控制端会收到处理结束的响应及时间，如果时间超过1分钟，立即再发一次“反馈”消息请求）

	读取当天可投放广告主集合，比较这些广告主的当天消耗与账户余额，如果剩余金额<=0，则修改业务数据库中该广告主的状态为“余额不足”，否则修改为“正常”；

	读取当天可投放计划集合，比较这些计划当天的消耗达到日预算，修改业务数据库中该计划的投放状态为“日预算满”；如果总消耗达到总预算，修改业务数据库中该计划的投放状态为“总预算满”。否则，因为预算发生变化后，要将原来投放状态为“满”的状态修改为“正常”。

### Daily Balancer
日结算器。
1. 执行逻辑
* 等待“日结算启动”消息（由crontab产生，一般23:59:30开始）
* 设置“全局广告暂停”（缓存redis控制信息，并向各预算控制器发送该消息，以尽快全局停投）
* 等待“订单报表统计成功”消息，收到消息后，对广告主按天统计（mongo），将统计得到的数据执行扣款、生成账单事务；
* 查询业务数据库中状态为“正常”（非锁定）、账户余额大于0的广告主，及其下状态为“正常投放”或“日预算投满”的计划，将广告主列表写入到缓存redis集合sponsors:{day}，将计划列表写入到集合plans:{day}；
* 对查询到当日可投的广告主和计划账户余额更新到控制缓存中
* 重新设置“全局广告开启”（缓存redis控制信息，并向各预算控制器发送该消息，以尽快全局开投）；

2. 问题

### Realtime Calculation
实时计算要实现实时报表、实时数据统计的功能，为用户提供及时的投放反馈，为投放提供控制参考数据。

1. 实现的任务
1.1 订单小时投放报表
	维度数据包括：广告主、计划、广告组、广告、日期、小时

	指标包括：推送量、竞价成功量、曝光量、点击量、媒体成本

	存储：mongodb

	用处：实时报表

1.2 投放消耗统计
	维度数据：预算控制器ID、计划、广告主、日期、小时

	指标包括：媒体成本

	存储：消息队列kafka

	用处：预算控制

1.3 用户频次统计
	维度数据：用户、广告、日期

	指标包括：曝光次数、点击次数、上次曝光时间、上次点击时间

	存储：缓存redis

	用处：用户频次控制

1.4 广告主天消耗统计
	维度数据：广告主、日期

	指标包括：曝光次数、点击次数、媒体成本

	存储：缓存redis

	用处：反馈至业务系统

1.5 广告主消耗统计
	维度数据：广告主

	指标包括：曝光次数、点击次数、媒体成本

	存储：缓存redis

	用处：反馈至业务系统

1.6 计划天消耗统计
	维度数据：计划、日期

	指标包括：曝光次数、点击次数、媒体成本

	存储：缓存redis

	用处：反馈至业务系统

1.7 计划消耗统计
	维度数据：计划

	指标包括：曝光次数、点击次数、媒体成本

	存储：缓存redis

	用处：反馈至业务系统

1.8 用户媒体天投放统计
	维度数据：用户、媒体类别、媒体标识、日期

	指标包括：曝光次数、点击次数、上次曝光时间、上次点击时间

	存储：缓存redis

	用处：用户媒体频次控制

1.9 广告位投放天统计
	维度数据：平台、广告位、日期、小时

	指标包括： 请求量、推送量、推送成本、竞价成功量、曝光量、点击量、竞价成功成本

	存储：mongodb

	用处：实时广告位投放展现

2. 问题
2.1 实时统计报表与离线报表统计冲突
	比如，重启统计任务，可能会统计上一个小时的数据，从而覆盖离线统计的数据，解决方法：1)判断当前时间，不处理上个小时的数据（需要同步好机器时间） 2)表中增加字段is_offline，表示如果有离线写入，则实时不写入，但由于mongodb无事务的概念，该方法仍有出问题的可能性

2.2 写mongodb/redis的压力导致处理变慢、增加累积消息
从消息队列kafka批处理，将数据预处理成统计结果，再次写入到kafka，由另外一个数据入库任务写入相应的库（数据缓存超时或次数达到限值时）。

2.3 如果要升级任务时，怎么办？
停投广告5分钟后，迅速重新启动。


### Offline Calculation
由于有的任务有数据依赖，不能并行处理的，需要制定一个执行计划，比如数据上传成功之前，执行预处理任务是无效的。计划管理可以通过消息来定义是否需要启动执行：
1. 数据聚合器在上传数据成功后，发出“数据上传成功”或“数据更新成功”的消息M1；
2. 数据预合并收到M1消息，执行任务，完成后，发出“数据合并成功”消息M2；
3.
3.1 订单报表任务收到M2消息，启动任务，删除原报表数据，插入新数据，发出“订单报表统计成功”消息M3_1；
3.2 媒体报表任务收到M2消息，启动任务，插入数据，发出“媒体报表统计成功”消息M3_2；

4. 在收到M3_1、M3_2后，启动点击率预估模型的数据预处理任务

5. 如果日结算任务收到“启动日结算”消息，执行日结算；

### Business System
业务系统，提供业务操作服务。
