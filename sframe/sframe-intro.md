# 基本的加载和保存操作

SFrame可以从[多种格式](../data_formats_and_sources/intro.md)加载数据，我们也在努力让SFrame支持更多格式。
很常见的一种数据格式是csv（逗号分隔值）文件，我们将在后面的示例中使用这种格式。
我们将在SFrame相关的示例中使用来自[Million Song Dataset](http://labrosa.ee.columbia.edu/millionsong/)的数据集。
第一张表包含数据库中各歌曲的元信息。下面展示如何将该数据集加载进一个SFrame：

```python
songs = gl.SFrame.read_csv("http://s3.amazonaws.com/dato-datasets/millionsong/song_data.csv")
```

简单吧！最简单的情况下不需要指定任何选项参数，因为SFrame的解析器会推断列数据类型。当然，在加载csv数据的时候
有很多选项参数可以指定。当加载用户在线收听这些歌曲的使用信息数据时，一些更常用的选项参数会有用：

```python
usage_data = gl.SFrame.read_csv("http://s3.amazonaws.com/dato-datasets/millionsong/10000.txt",
                                header=False,
                                delimiter='\t',
                                column_type_hints={'X3':int})
```

需要加上`header`和`delimiter`参数是因为这个csv文件没有在第一行提供列名，并且各值之间使用tab分割而不是逗号。
`column_type_hints`参数指定列类型而不用SFrame csv parser默认推断的列类型。要了解解析csv文件时的完整参数列表，请访问
[APIReference](https://dato.com/products/create/docs/generated/graphlab.SFrame.read_csv.html#graphlab.SFrame.read_csv).

导入完成之后我们就可以查看表的前面几行数据。

```python
songs
```

```
Columns:
	song_id	str
	title	str
	release	str
	artist_name	str
	year	int

Rows: 1000000

+--------------------+--------------------------------+
|      song_id       |             title              |
+--------------------+--------------------------------+
| SOQMMHC12AB0180CB8 |          Silent Night          |
| SOVFVAK12A8C1350D9 |          Tanssi vaan           |
| SOGTUKN12AB017F4F1 |       No One Could Ever        |
| SOBNYVR12A8C13558C |      Si Vos Quer\xc3\xa9s      |
| SOHSBXH12A8C13B0DF |        Tangle Of Aspens        |
| SOZVAPQ12A8C13B63C | Symphony No. 1 G minor "Si ... |
| SOQVRHI12A6D4FB2D7 |        We Have Got Love        |
| SOEYRFT12AB018936C |       2 Da Beat Ch'yall        |
| SOPMIYT12A6D4F851E |            Goodbye             |
| SOJCFMH12A8C13B0C2 |   Mama_ mama can't you see ?   |
+--------------------+--------------------------------+
+--------------------------------+--------------------------------+------+
|            release             |          artist_name           | year |
+--------------------------------+--------------------------------+------+
|     Monster Ballads X-Mas      |        Faster Pussy cat        | 2003 |
|       Karkuteill\xc3\xa4       |        Karkkiautomaatti        | 1995 |
|             Butter             |         Hudson Mohawke         | 2006 |
|            De Culo             |          Yerba Brava           | 2003 |
| Rene Ablaze Presents Winte ... |           Der Mystic           |  0   |
| Berwald: Symphonies Nos. 1 ... |        David Montgomery        |  0   |
|   Strictly The Best Vol. 34    |       Sasha / Turbulence       |  0   |
|            Da Bomb             |           Kris Kross           | 1993 |
|           Danny Boy            |          Joseph Locke          |  0   |
| March to cadence with the  ... | The Sun Harbor's Chorus-Do ... |  0   |
|              ...               |              ...               | ...  |
+--------------------------------+--------------------------------+------+
[1000000 rows x 5 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```


```python
usage_data
```

```
Columns:
	X1	str
	X2	str
	X3	int

Rows: 2000000

+--------------------------------+--------------------+-----+
|               X1               |         X2         |  X3 |
+--------------------------------+--------------------+-----+
| b80344d063b5ccb3212f76538f ... | SOAKIMP12A8C130995 |  1  |
| b80344d063b5ccb3212f76538f ... | SOBBMDR12A8C13253B |  2  |
| b80344d063b5ccb3212f76538f ... | SOBXHDL12A81C204C0 |  1  |
| b80344d063b5ccb3212f76538f ... | SOBYHAJ12A6701BF1D |  1  |
| b80344d063b5ccb3212f76538f ... | SODACBL12A8C13C273 |  1  |
| b80344d063b5ccb3212f76538f ... | SODDNQT12A6D4F5F7E |  5  |
| b80344d063b5ccb3212f76538f ... | SODXRTY12AB0180F3B |  1  |
| b80344d063b5ccb3212f76538f ... | SOFGUAY12AB017B0A8 |  1  |
| b80344d063b5ccb3212f76538f ... | SOFRQTD12A81C233C0 |  1  |
| b80344d063b5ccb3212f76538f ... | SOHQWYZ12A6D4FA701 |  1  |
|              ...               |        ...         | ... |
+--------------------------------+--------------------+-----+
[2000000 rows x 3 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```

我们可以重命名默认列名：

```python
usage_data.rename({'X1':'user_id', 'X2':'song_id', 'X3':'listen_count'})
```


SFrames可以以csv文件形式或SFrame二进制格式保存。若以二进制格式保存，加载是即时的，
因为不需要再去解析该文件了。默认是以二进制格式保存，我们只需提供保存二进制文件的
文件夹名即可。

```python
usage_data.save('./music_usage_data')
```

随后加载将非常快速：

```python
same_usage_data = gl.load_sframe('./music_usage_data')
```

除了这些函数之外，SFrame还支持JSON导入和导出、SQL/ODBC导入及Spark RDD转换这些功能。
想要了解更多信息，请查看GraphLab Create API文档相关页面：

* [read_json](https://dato.com/products/create/docs/generated/graphlab.SFrame.read_json.html)
* [export_json](https://dato.com/products/create/docs/generated/graphlab.SFrame.export_json.html)
* [read_csv](https://dato.com/products/create/docs/generated/graphlab.SFrame.read_csv.html)
* [export_csv](https://dato.com/products/create/docs/generated/graphlab.SFrame.export_csv.html)
* [from_odbc](https://dato.com/products/create/docs/generated/graphlab.SFrame.from_sql.html)
* [from_odbc](https://dato.com/products/create/docs/generated/graphlab.SFrame.from_odbc.html)
* [from_rdd](https://dato.com/products/create/docs/generated/graphlab.SFrame.from_rdd.html)
* [to_rdd](https://dato.com/products/create/docs/generated/graphlab.SFrame.to_rdd.html)
* [to_spark_dataframe](https://dato.com/products/create/docs/generated/graphlab.SFrame.to_spark_dataframe.html)

与Spark RDD和关系数据库相关的交互信息也可以查看本指南的相关章节：
* [Spark RDDs](../data_formats_and_sources/spark_integration.md)
* [SQL数据库](../data_formats_and_sources/sql_integration.md)



#### 数据类型

An SFrame is made up of columns of a contiguous type. For instance the `songs`
SFrame is made up of 5 columns of the following types

```
	song_id	str
	title	str
	release	str
	artist_name	str
	year	int
```

在这个SFrame中只出现了字符串(`str`)和整数(`int`)类型的列，但是SFrame还支持许多数据类型：

* `int` (有符号64位整型)
* `float` (双精度浮点型)
* `str` (字符串)
* `array.array` (双精度一维数组)
* `list` (列表)
* `dict` (字典)
* `datetime.datetime` (精确到微秒的时间日期型)
* `image` (图片)
