# 第三章 Spark ETL

***set_daemon@126.com 2017-09-22 图书：Learning Apache Spark 2，可在github https://github.com/PacktPublishing/Learning-Apache-Spark-2找到源码。 （如有侵权，请与我联系）***

现在我们已查看了Spark的架构，并且对RDD做过一番详细的讨论，在第二章***Spark RDD的transformation和action***的结尾部分，我们重点关注了PairRDD和它的一些transformation。

本章重点是用Apache Spark实现ETL，包括以下的一些主题，希望可以为你在Apache Spark上做更多事情提供帮助：

​	理解ETL过程；

​	通常支持的文件格式；

​	通常支持的文件系统；

​	对接NoSQL数据库；

让我们开始吧！

## 什么是ETL

ETL代表抽取（Extraction）、转换（transformation）以及加载（loading），该术语存在了近20年，作为工业标准，代表数据移动和转换处理以构建数据管道传递给BI和分析系统。ETL处理广泛用于数据迁移和主数据管理创始（master data management initiatives？）。由于本书的重点是Spark，我们将稍微讲讲ETL这个话题，但不会过多过细。

#### 抽取

抽取是ETL处理的第一个部分，表示从源系统的数据抽取。这通常是ETL处理的最重要部分之一，为后续更多处理设立了阶段（？），在抽取过程中主要考虑以下事情：

​	源系统类型（RDBMS、NoSQL、FlatFiles、Twitter/Facebook流）

​	文件格式（CSV、JSON、XML、Parquet、Sequence、Object文件）

​	抽取频率（天、小时、每秒）

​	抽取的大小

#### 加载



