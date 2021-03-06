# DSP结算
***set_daemon@126.com 2018-04-15***

## 种类
结算分为实时结算和定时结算，如果可以做到实时结算是最好的，但由于存在可能的异常数据，而线上无法实时剔除，那么就得采用定时结算，DSP系统采用的是定时结算。

## 定时结算
定时结算可以定时5分钟，或者一个小时，或者按天，注意到跨时的问题将会影响结算结果，比如前一个五分钟的曝光数据会出现在下一个五分钟，5点的推送会在6点的曝光日志中出现，如果按整5分钟或者一个小时的方式处理，有一部分数据无法正常计算，除非在计算这个时间的数据时，将上个时段出现的数据补回去，问题来了，如果真这么结算，会导致用户看到不一致的数据，之前看到5点的消耗是100元，过一个小时后变成了120元，所以如果按照补数据的方式，应该时间间隔较短比较好，这样，延迟5分钟后看到的数据差异是可以接受的。

另外一种方式是不补数据，那么在跨时间段之前暂停广告（暂停时间长度应保证能在下个时间段到来之前回收完，且不超过反作弊/过滤的时间长，比如15秒），假设按照15秒的时间算，23个时间间隔(按小时)，差不多每天有5分钟的不可投放时间，如果按5分钟时间段，则产生72分钟（1小时12分钟）的停投时长，而如果按天算，最少15秒，但不超过5分钟的不可投放时间，用户在第二天看前一天的数据是稳定的，应该是可以接受。

### 按天结算
完成结算的步骤如下：
1. 在23:59:00的时候对线上所有广告暂停投放（系统投放总开关），这样就解决了跨天问题，而且对前一天的投放影响不大（如果投不完的，也不差这一分钟，在预算控制前应该去掉这一分钟）。
2. 等待上一个小时的日志都已上传到Hadoop后（日志数据每小时上传），将前一天的同一条push/曝光/点击/竞价成功日志进行抽取，合并到一条记录中，格式如下（部分与结算相关的数据）：
```
push_timestamp|win_timestamp|imp_timestamp|click_timestamp|win_flag|imp_flag|click_flag|bid_price|win_price|ad_id|group_id|plan_id|sponsor_id
```
push_timestamp表示广告推送时间，会作为报表的时间依据。win_flag/imp_flag/click_flag取值为0或1,表示是否发生对应的事件。

3. 按广告主和计划进行时间统计
以SQL表示：`SELECT DAY(push_timestamp) AS day, sponsor_id, SUM(win_flag) AS win, SUM(imp_flag) AS imp, SUM(click_flag) AS click, SUM(win_price)/1000 AS cost FROM blog GROUP BY(sponsor_id, DAY(push_timestamp))`
得到每个广告主当天的投放情况（计划同理）。

注意：该统计优先执行，其余更丰富的统计分析任务后置。

4. 扣款
事务型操作，对广告主账户扣除相应金额，生成一条账单数据。

5. 更新缓存数据
同步广告主余额、计划总消耗、广告主前一天消耗、计划前一天消耗，用于预算控制使用。

6. 开启系统总开关
开启整个系统的投放开关。


## 关键问题

### 扣款时同时充值
该问题出现在于，如何对mysql进行原子操作？事务操作。

### 处理时间对业务的影响
有可能数据量较大，计算时间变长，甚至因此计算失败，将会极大影响第二天的投放以及正常结算。所以在上线前，要测算好不同数据压力下的处理性能，并监控当天的数据变化，如果有超过限额的风险，提前组织人力和调配资源。

### 充值或修改预算
当广告主充值或修改计划预算时，计划投放状态会发生相应的变化，如可投变为不可投，或者不可投变可投，但这个时候正在结算处理周期内，发生投放冲突，所以，对整个投放系统加一个全局“投放控制锁”，且只能结算时变更该状态，这样，其它同时执行的控制逻辑可以与结算保持独立。

### 结算整个过程被中断怎么处理
每一步执行时加记录，并生成一个记录表放在缓存中，如果中间失败，下次启动时，在失败点重新执行。

## 优化方案
