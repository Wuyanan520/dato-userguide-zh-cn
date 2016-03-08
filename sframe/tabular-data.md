# 处理表格数据

通常情况下，你第一次接触一个数据集的时候，它的格式类似一张表。在为更复杂的数据分析而清洗数据时，
表是一种很简洁明了的格式。[SFrame](https://dato.com/products/create/docs/generated/graphlab.SFrame.html)
是GraphLab Create中的表格数据结构。SFrame的目的是扩展到远超内存容量的数据集。

我们将在下面的章节介绍SFrame的基本内容：

* [加载和保存](sframe-intro.md) 主要讲述从已有的CSV格式数据创建SFrame及如何持久化SFrame（将SFrame保存到文件）。

* SFrame支持大量通用的数据处理操作，我们将在[数据处理](data-manipulation.md)一章回顾大量通用数据处理操作。

* [Apache Spark RDDs](../data_formats_and_sources/spark_integration.md) 详细描述与Apache Spark RDD的相互转换。

* [SQL数据库](../data_formats_and_sources/sql_integration.md)一章解释如何通过Python DBAPI2或ODBC与关系型数据库交互。
