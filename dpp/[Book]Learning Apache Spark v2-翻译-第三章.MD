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

一旦数据抽取完成，下一个逻辑步骤是将数据载入至相关的框架以便于处理。在转换前将数据载入到相关的框架或工具的目的是，允许执行转换的系统与数据处理更相关且高性能。例如，如果你从一个Spark连接不上的系统中抽取数据，比如说Ingres数据库，并将其保存为文本文件，现在，在数据可用之前，你可能需要做一些转换。这里，你有两个选择：要么基于已抽取的文件做转换，要么将数据载入到类似于Spark的框架以便后续处理。后一种方法的好处是像Spark这种MPP框架比基于文件系统的数据处理性能更高。

#### 转换

一旦数据在框架中可用，你可以运用一些相关的转换。由于在Spark的核心抽象是一个RDD，而我们也看过了RDD上的一些可用转换。

Spark支持对某些系统的连接，这本质上将抽取和载入过程结合成单个活动（即一并完成了），因为可以从数据源直接将数据流入到Spark上。在一些案例中，由于我们有大量不同种类的可用源系统，而Spark并不能提供所有的连接器，意味着你不得不借助特定系统或第三方的工具抽取数据。

## 如何使用Spark

Matei Zaharia是Apache Spark项目的创建者，也是DataBricks公司的联合创始人，这家公司也是有Apache Spark的创建者们组建的。Matei在2015年秋的欧洲Spark高层会议的演讲文稿中提到，Spark使用在不同的运行环境的关键指标。我对这个数字有一些惊讶，我一直认为Spark运行在YARN的情况要比其它的要高的多，而事实并非如此，关键数据如下：

​	Spark运行在Standalone模式：48%；

​	Spark运行在YARN：40%；

​	Spark运行在MESOS：11%；

可以从数字上看的，接近90%的Apache Spark安装都是在standalone模式或在YARN上。当Spark被配置工作于YARN，可以猜想到该组织已经选择Hadoop作为他们的数据操作系统，并会计划迁移数据到Hadoop，这意味着主要的摄入数据源可能是Hive、HDFS、HBASE或者其它NoSQL系统。

当Apache Spark安装在Standalone模式，主要源的可能性增加了，但数据存放于HDFS仍然保持巨大的可能性，完全有可能客户安装了Hadoop，但希望将Spark独立出来作为“挖掘”平台。

Spark可以工作于多种源，让我们看看最常见的那些已经遇到过的源：

​	文件格式；

​	文件系统；

​	结构化数据源/数据库；

​	Key/Value存储；

#### 一般支持的文件格式

我们已经看到用基于Spark类SparkContext的textFile()方法处理文本文件是多么轻松。然而，Apache Spark支持大量的其它格式必定会让你更高兴，且随着Spark的每次发布都会增加。在Apache Spark发布版2.0，下面的文件格式也被支持：

​	Text文件（已经涵盖）；

​	JSon文件；

​	CSV文件；

​	序列化文件；

​	Object文件；

###### 文本文件

​	我们已经在第一章***架构与安装***和第二章***Spark RDD的transformation和action***看到多个用textFile()函数读取文本文件的例子。在文本文件中的每一行作为一个新纪录。我们也看过whoTextFiles()的例子，返回一个PairRDD，其key是文件的标识。这在ETL job中非常有用，有时会想基于key对数据做不同的处理，或者甚至传递至下一个处理流程。

ETL处理的一个重要部分是在平台之上运行的应用执行处理之后保存数据，一个非常趁手的方法saveAsTextFile(pathToFile)可以存储问题。值得注意的是传递给该方法的路径基本上是一个路径名称，从不同节点的输出将会保存到该特殊的路径下。

例子3.1：Scala的saveAsTextFile()：

​	// 读取所有的README.md文件

​	val dataFile = sc.textFile("file:///home/sysop/services/spark/README.md")

​	// 对行划词，将每个split的结果平铺

​	val words = dataFile.flatMap(line => line.split(" "))

​	// 保存至文本文件

​	words.saveAsTextFile("file:///tmp/scalawords")	

![saveAsTextFile_output_multifiles_Scala.png](attachments/saveAsTextFile_output_multifiles_Scala.png)

例子3.2：Python的saveAsTextFile()：

​	// 读取所有的README.md文件

​	dataFile = sc.textFile("file:///home/sysop/services/spark/README.md")

​	// 对行划词，将每个split的结果平铺

​	words = dataFile.flatMap(lambda line: line.split(" "))

​	// 保存至文本文件

​	words.saveAsTextFile("file:///tmp/pythonwords")

例子3.3：Java的saveAsTextFile()：

​	// 读取所有README.md文件

​	JavaRDD\<String\> dataFile = sc.textFile(fileName);

​	// 对行划词，每个split的结果平铺

​	JavaRDD\<String\> words = dataFile.flatMap(line -> Arrays.asList(line.split("")).iterator());

​	// 保存至文本文件

​	words.saveAsTextFile(outputFile);

###### CSV和TSV文件

CSV是一种通用的数据交易格式，被大多消费者、业务和科学应用所广泛支持，其最常用的一个场景是在应用和平台间移动数据，且在许多场景中已经被ETL应用当作事实上的标准。除此之外，大量的公开数据是CSV格式。

自Spark2.0开始，CSV现在是一个基于DataBricks的Spark-CSV模块（http://bit.ly/2cAXCyr）的原生数据源。重点需要理解的是CSV通常表示一个结构化的数据集，在文件中有由逗号分割的特定列数。在更深入些之前，我们需要理解，到现在为止，我们只使用过RDD API，但是以非结构化数据角度来查看数据，Spark框架并不知道你的RDD内容，只是当作一个可以持久化、在网络传输或者用迭代器操作的对象，这明显意味着它限制了框架执行高级优化的能力（例如压缩等等）。然而，由定义而来的应用结构化意味着，我们将限制表达能力。但是，我们通常需要对我们的数据做什么？通常的计算是读取数据、联合数据、过滤数据、统计数据以及聚合数据，这意味着，即使我们有更多结构化API，我们可以应用到多种计算上（？）。

DataFrame是一种不可变的、组织成有名列的分布式数据集合。如果你有数据库背景，可以把它类比成数据库表。如果你有Python/R背景，你将发现Spark的DataFrame使用起来极为灵活（？）；然而，框架提供了更丰富的优化，DataFrame的目标是掩盖RDD的复杂性，并让Spark为更多的人所接受。

Spark在2.0版本中统一了DataFrame和Dataset（直到1.6还是分开的），这为你提供编译时的语法和分析错误报告，该特性又比较像类型化的RDD。Dataset API是类型安全的对象，可以操作编译好的lambda函数。我们将在第5章***Spark流处理***讨论DataFrame和Dataset的更多细节，现在介绍它们的原因是我们想用最简单的方式来读取CSV文件，直接用SparkSession并返回一个DataFrame而非一个RDD。

我们将加载一个有关UK的房屋价格数据集，由land registery发布，且在链接http://bit.ly/2cb58h下载该数据集（注：貌似现在”失联“了）。

例子3.4:用Scala读取CSV：

​	val pricePaidDS = spark.read.format("csv").option("header", "false").load("file:///home/spark/sampledata/pp-monthly-update-new-version.csv")

当然，你仍可以反过来用老式工作方式，用toJavaRDD()操作将dataset转换成一个RDD。

例子3.5:用Python读取CSV：

​	pricePaidDS = spark.read.csv("file:///home/spark/sampledata/pp-monthly-update-new-version.csv", header=False)

例子3.6:用Java读取CSV：

​	SparkSession spark = SparkSession.builder().master("local").appName("SparkCSVExample").config("spark.some.config.option", "some-value").getOrCreate();

​	Dataset\<Row\> pricePaidDS = spark.read().csv(fileName);

​	JavaRDD\<Row\> pricePaidRDD = pricePaidDS.toJavaRDD();

###### CSV文件的写入

写入CSV文件与读非常相似，尽管你必须使用spark.write()存储数据，并选择csv作为输出格式。重要的是要意识到，Spark将写到多个分输出文件，可能你需要它们拼接在一起。

例子3.7:用Scala写入CSV：

​	pricePaidDS.write.format("csv").save("file:///home/spark/sampledata/price_paid_outpout")

###### Tab分割的文件

Tab分割的文件（TSV）也是一种存储结构化数据的通用方法，可以作为CSV格式的替代，这经常有难度，由于文件数据广泛使用了字面的逗号，因此需要提供转义的逗号。Spark让你可以在相同效率方式下读取TSV文件。记住，tsv与csv相似，仅仅不同之处是分隔符从逗号变为tab。

我们有一个样本文件test.tsv，这是一个tab分隔的文件，用于展示需要用于加载tsv文件的代码。

例子3.8:用Scala读取TSV：

​	val testDS = spark.read.format("csv").option("delimiter", "t").load("/home/spark/sampledata/test.tsv")

例子3.9：用Python读取TSV：

​	testDS = spark.read.csv("/home/spark/sampledata/test.tsv", sep="t")

例子 3.10:用J啊v啊读取TSV：

​	SparkSession spark = SparkSession.builder().master("local").appName("SparkTSVExample").config("spark.some.config.option", "some-value").getOrCreate();

​	Dataset\<Row\> pricePaidDS = spark.read().option("sep", "t").csv(fileName);

如你所见，你将仅仅需要指定分隔符sep并传入t参数，指定你正在读取一个TSV文件。

###### JSON文件

