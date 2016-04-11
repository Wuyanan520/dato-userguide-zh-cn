# SFrame简介

SFrame是在GraphLab Create中从其他数据源读取数据的基本数据结构。

SFrame可以从如下静态文件格式中抽取数据：

* [CSV](https://dato.com/products/create/docs/generated/graphlab.SFrame.read_csv.html#graphlab.SFrame.read_csv)
* [JSON](https://dato.com/products/create/docs/generated/graphlab.SFrame.read_csv.html#graphlab.SFrame.read_json)
* [Apache Avro](https://dato.com/products/create/docs/generated/graphlab.SArray.from_avro.html#graphlab.SArray.from_avro)

以及其他数据源:

* [Apache Spark RDDs](spark_integration.md)
* [SQL databases](sql_integration.md)
