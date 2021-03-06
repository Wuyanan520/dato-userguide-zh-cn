# 数据处理

数据一导入就足够清洁到可以用我们的工具包在上面执行有意义操作这种情况并不常见。
这就是文件...数据就是杂乱的。SFrames可以让你以可伸缩的方式完成数据清洗任务，
即使是在比计算机内存还大的数据集上。


#### 列选择和处理

你可能已经注意到在song metadata中存在某些song的year值为0的问题。假设我们想要将这些值
改成缺失值，以便他们不会使该列上的统计量偏斜（如`mean`或`min`）。如果我们在解析数据之前
已经知道或者我们愿意再次解析文件，我们可以将`read_csv`的参数`na_values`设置为0。或者，
我们还可以在SFrame的一列或多列上应用一个函数。下面使用Python的lambda函数将零值替换为缺失值。

```python
songs['year'] = songs['year'].apply(lambda x: None if x == 0 else x)
songs.head(5)
```
```
+--------------------+----------------------+--------------------------------+
|      song_id       |        title         |            release             |
+--------------------+----------------------+--------------------------------+
| SOQMMHC12AB0180CB8 |     Silent Night     |     Monster Ballads X-Mas      |
| SOVFVAK12A8C1350D9 |     Tanssi vaan      |       Karkuteill\xc3\xa4       |
| SOGTUKN12AB017F4F1 |  No One Could Ever   |             Butter             |
| SOBNYVR12A8C13558C | Si Vos Quer\xc3\xa9s |            De Culo             |
| SOHSBXH12A8C13B0DF |   Tangle Of Aspens   | Rene Ablaze Presents Winte ... |
+--------------------+----------------------+--------------------------------+
+------------------+------+
|   artist_name    | year |
+------------------+------+
| Faster Pussy cat | 2003 |
| Karkkiautomaatti | 1995 |
|  Hudson Mohawke  | 2006 |
|   Yerba Brava    | 2003 |
|    Der Mystic    | None |
+------------------+------+
[5 rows x 5 columns]
```

注意我们需要将结果再赋值给我们的SFrame，因为SFrame列的内容（其实是一个叫做[SArray](https://dato.com/products/create/docs/generated/graphlab.SArray.html)
的数据结构）是不可变的。SFrames可自由增减列是因为其列本质上是SArrays的引用。

这里我们使用lambda函数是因为这是实例化小函数最简单的方法。`apply`方法也可以
以命名函数（即以`def`开头的普通Python函数）为参数。只要函数有一个参数并且返
回一个值，就可以将其应用到一列上去。

我们也可以将函数应用到多列上去。假如我们需要添加一列来记录'love'在'title'列和
'artist'列出现次数之和：

```python
songs['love_count'] = songs[['title', 'artist_name']].apply(
    lambda row: sum(x.lower().split(' ').count('love') for x in row.values()))
songs.topk('love_count', k=5)
```

```
+--------------------+--------------------------------+
|      song_id       |             title              |
+--------------------+--------------------------------+
| SOMYDCX12A8AE4836B | The Love Story (Part 1) In ... |
| SONXAVM12AB017AA1D |          Document 15           |
| SOAWJOC12A8C1367A7 |       Black Black Window       |
| SOXAVWF12A8AE4922C |           One Piece            |
| SOAJRDR12A8C1383B1 |         Love Love Love         |
+--------------------+--------------------------------+
+--------------------------------+--------------------------------+------+------------+
|            release             |          artist_name           | year | love_count |
+--------------------------------+--------------------------------+------+------------+
|      Skid Row / 34 Hours       |            Skid Row            | None |     4      |
| Feels_ Feathers_ Bog and Bees  | Low Low Low La La La Love  ... | None |     3      |
|          Ends Of June          | Low Low Low La La La Love  ... | 2007 |     3      |
| Low Low Low La La La Love  ... | Low Low Low La La La Love  ... | 2007 |     3      |
|           Radio Hitz           |       Spider Murphy Gang       | None |     3      |
+--------------------------------+--------------------------------+------+------------+
[5 rows x 6 columns]
```

这里还需要注意的是：首先我们在方括号中使用列表选择了一个列子集。这通常都是有用的，
因为在这种情况下我们可以通过减少需要扫描值的数量来提高性能。当在SFrame上而不是SArray上
调用apply方法时，lambda函数的输入时一个字典，键是列名，值是行对应各列的值。

另外一个有用切常用的操作是根据类型来选择列。例如，下面代码抽取所有包含字符串的列。

```python
song[str]
```

```
Data:
+--------------------+-------------------------------+
|      song_id       |             title             |
+--------------------+-------------------------------+
| SOQMMHC12AB0180CB8 |          Silent Night         |
| SOVFVAK12A8C1350D9 |          Tanssi vaan          |
| SOGTUKN12AB017F4F1 |       No One Could Ever       |
| SOBNYVR12A8C13558C |         Si Vos Querés         |
| SOHSBXH12A8C13B0DF |        Tangle Of Aspens       |
| SOZVAPQ12A8C13B63C | Symphony No. 1 G minor "Si... |
| SOQVRHI12A6D4FB2D7 |        We Have Got Love       |
| SOEYRFT12AB018936C |       2 Da Beat Ch'yall       |
| SOPMIYT12A6D4F851E |            Goodbye            |
| SOJCFMH12A8C13B0C2 |   Mama_ mama can't you see ?  |
+--------------------+-------------------------------+
+-------------------------------+-------------------------------+
|            release            |          artist_name          |
+-------------------------------+-------------------------------+
|     Monster Ballads X-Mas     |        Faster Pussy cat       |
|          Karkuteillä          |        Karkkiautomaatti       |
|             Butter            |         Hudson Mohawke        |
|            De Culo            |          Yerba Brava          |
| Rene Ablaze Presents Winte... |           Der Mystic          |
| Berwald: Symphonies Nos. 1... |        David Montgomery       |
|   Strictly The Best Vol. 34   |       Sasha / Turbulence      |
|            Da Bomb            |           Kris Kross          |
|           Danny Boy           |          Joseph Locke         |
| March to cadence with the ... | The Sun Harbor's Chorus-Do... |
+-------------------------------+-------------------------------+
```

Now, you may not feel comfortable transforming a column without inspecting more
than the first 10 rows of it, as we did with the `year` column. To quickly get
a summary of the column, we can do:

就像我们在`year`列上的变换操作一样，现在你可能会对不检查前10行之外的数据就对一列执行变换操作感觉不放心。
我们可以通过如下方式快速获取一列的概要信息：

```python
songs['year'].sketch_summary()
```

```
+--------------------+---------------+----------+
|        item        |     value     | is exact |
+--------------------+---------------+----------+
|       Length       |    1000000    |   Yes    |
|        Min         |      0.0      |   Yes    |
|        Max         |     2011.0    |   Yes    |
|        Mean        |  1030.325652  |   Yes    |
|        Sum         |  1030325652.0 |   Yes    |
|      Variance      | 997490.582407 |   Yes    |
| Standard Deviation | 998.744503067 |   Yes    |
|  # Missing Values  |       0       |   Yes    |
|  # unique values   |       90      |    No    |
+--------------------+---------------+----------+

Most frequent items:
+-------+--------+-------+-------+-------+-------+-------+-------+-------+-------+
| value |   0    |  2007 |  2006 |  2005 |  2008 |  2009 |  2004 |  2003 |  2002 |
+-------+--------+-------+-------+-------+-------+-------+-------+-------+-------+
| count | 484424 | 39414 | 37546 | 34960 | 34770 | 31051 | 29618 | 27389 | 23472 |
+-------+--------+-------+-------+-------+-------+-------+-------+-------+-------+
+-------+
|  2001 |
+-------+
| 21604 |
+-------+

Quantiles:
+-----+-----+-----+-----+--------+--------+--------+--------+--------+
|  0% |  1% |  5% | 25% |  50%   |  75%   |  95%   |  99%   |  100%  |
+-----+-----+-----+-----+--------+--------+--------+--------+--------+
| 0.0 | 0.0 | 0.0 | 0.0 | 1970.0 | 2002.0 | 2008.0 | 2009.0 | 2011.0 |
+-----+-----+-----+-----+--------+--------+--------+--------+--------+

```

看来除了0之外没有其他不合法的年份值了。概要信息将值分成了"exact"（准确）和"approximate"（近似）两类。
近似值当然也有可能作为准确值返回，但是对于概要信息，我们使用近似值来确保对于大型数据集的探索是可扩展的（？）。
我们使用的方法仅对指定列的数据进行一遍扫描，并且每一个操作都有严格定义的范围指明结果会在多大程度上是有误的。
具体参见[API Reference](https://dato.com/products/create/docs/generated/graphlab.Sketch.html)。

使用概要信息中最常出现的项及分位数信息，你几乎可以想象出年份的分布情况，其中最大一部分是20世纪前十年（？）。
幸运的是我们可以不仅仅是在头脑中数据分布，GraphLab Canvas提供了SFrame及其他数据结构之上的可视化，
在[可视化](visualization.md)一节将深入介绍GraphLab Canvas。下面的例子将这些近似分位数以直方图的形式可视化：

```
songs['year'].show()
```

[<img alt="Histogram of Release Years" src="images/sframe_user_guide_1_histogram.png" style="max-width: 70%; margin-left: 15%;" />](images/sframe_user_guide_1_histogram.png)


我们已经了一些探索，转换和特征生成，让我们继续花些时间过滤我们后面用不到的值吧。例如，
我们可能只需要有确切日期的歌曲，基本的过滤操作如下：

```python
dated_songs = songs[songs['year'] != None]
dated_songs
```

```
+--------------------+--------------------------------+
|      song_id       |             title              |
+--------------------+--------------------------------+
| SOQMMHC12AB0180CB8 |          Silent Night          |
| SOVFVAK12A8C1350D9 |          Tanssi vaan           |
| SOGTUKN12AB017F4F1 |       No One Could Ever        |
| SOBNYVR12A8C13558C |      Si Vos Quer\xc3\xa9s      |
| SOEYRFT12AB018936C |       2 Da Beat Ch'yall        |
| SOYGNWH12AB018191E |         L'antarctique          |
| SOLJTLX12AB01890ED |       El hijo del pueblo       |
| SOMPVQB12A8C1379BB |             Pilots             |
| SOSDCFG12AB0184647 |              006               |
| SOBARPM12A8C133DFF | (Looking For) The Heart Of ... |
+--------------------+--------------------------------+
+--------------------------------+------------------+------+------------+
|            release             |   artist_name    | year | love_count |
+--------------------------------+------------------+------+------------+
|     Monster Ballads X-Mas      | Faster Pussy cat | 2003 |     0      |
|       Karkuteill\xc3\xa4       | Karkkiautomaatti | 1995 |     0      |
|             Butter             |  Hudson Mohawke  | 2006 |     0      |
|            De Culo             |   Yerba Brava    | 2003 |     0      |
|            Da Bomb             |    Kris Kross    | 1993 |     0      |
|   Des cobras des tarentules    | 3 Gars Su'l Sofa | 2007 |     0      |
| 32 Grandes \xc3\x89xitos  CD 2 |  Jorge Negrete   | 1997 |     0      |
|           The Loyal            |    Tiger Lou     | 2005 |     0      |
|       Lena 20 \xc3\x85r        | Lena Philipsson  | 1998 |     0      |
|           Cover Girl           |   Shawn Colvin   | 1994 |     0      |
+--------------------------------+------------------+------+------------+
[? rows x 6 columns]
Note: Only the head of the SFrame is printed. This SFrame is lazily evaluated.
You can use len(sf) to force materialization.
```

输出信息很好的解释了过滤操作过程中发生了什么。对不需立即使用的结果SFrames会暂不执行操作，
这种情况下，如果你看了前几行的结果发现过滤操作不需要再执行了，GraphLab就不会浪费计算时间，
但是验证缺失值的确被移除了很重要。我们确实移除了484424行，所以我们强制新的SFrame具体化（即全部执行？）。

```python
len(dated_songs)
```

```
515576
```

为什么这个过滤语法会起效？我们实际上是在方括号中放置了一个SArray，应用于SArray上的
比较操作符返回一个等长的bool型SArray。如下所示：

```python
songs['year'] != None
```

```
dtype: int
Rows: 1000000
[1, 1, 1, 1, 0, 0, 0, 1, 0, 0, 1, 1, 0, 1, 0, 1, 1, 1, 1, 0, 0, 0, 1, 1, 0, 0, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1, 1, 0, 1, 0, 0, 0, 0, 1, 1, 1, 0, 0, 1, 0, 0, 1, 1, 0, 1, 1, 1, 0, 0, 0, 1, 0, 0, 1, 0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 1, 1, 0, 1, 0, 1, 0, 0, 0, 1, 1, 1, 1, 0, 1, 1, 1, ... ]
```

我们可能会需要使用两个以上比较操作符，这时我们需要将每一个比较语句放置在一个小括号里，
并使用按位布尔逻辑操作符连接，因为Python不允许逻辑操作符重载。也许我们会使用这个数据
搭建一个推荐系统，我们可能会只使用播放次数合理的数据：

```python
reasonable_usage = usage_data[(usage_data['listen_count'] >= 10) & (usage_data['listen_count'] <= 500)]
len(reasonable_usage)
```

```
114026
```

你也可以写一个lambda函数传递给`filter`函数做参数来过滤，
具体请参考[API Reference](https://dato.com/products/create/docs/generated/graphlab.SArray.filter.html#graphlab.SArray.filter).

#### 连接和聚合

另一种过滤数据集的重要方法是去除重复值。一种比较棒的查看重复值得方法是使用GraphLab
Canvas将SFrame可视化。

```
songs.show()
```

[<img alt="Summary of Song Metadata" src="images/sframe_user_guide_2_summary.png" style="max-width: 100%; margin-left: 0%;" />](images/sframe_user_guide_1_histogram.png)

看来`song_id`并不是完全唯一的（上图num_unique(est)）。如果我们要通过合并
`songs`和`usage_data`在`usage_data`中加入歌名信息，`song_id`的不唯一会使合并操作出错。
这一数据集中包含重复歌曲是因为他们可能在几个不同的专辑中发行（电影配乐和电台单曲等）。
如果我们不关心歌曲是包含在哪个专辑中，我们可以这样来过滤掉重复值：

```python
other_cols = songs.column_names()
other_cols.remove('song_id')
agg_list = [gl.aggregate.SELECT_ONE(i) for i in other_cols]
unique_songs = songs.groupby('song_id', dict(zip(other_cols, agg_list)))
```

我们来进一步解释上面的代码块。它主要集中在带`SELECT_ONE`聚合器的`groupby`操作上。
它会从每一组随机选择一行。你必须显式地指明哪些列需要使用该聚合器，所以我们使用
列表推倒来获取除用于分组（groupby）的列之外的其他所有列，并在这些列上分别使用
`SELECT_ONE`聚合器。在这种用法下，`SELECT_ONE`在每一列上使用随机选择的同一行
（聚合操作中出现多列上执行`SELECT_ONE`这种情况时，每一列的值都选自同一个随机行，
而不会出现每一列随机选择的问题）。如果选择重复记录中的哪一行不重要的话，使用`SELECT_ONE`
是很不错的，反之，可以使用如`MIN`或`MAX`等其他聚合器。

假如我们要查看播放次数最高的歌曲呢？我们可以正确地对歌曲分组并聚合计算收听次数，然后
将结果与歌曲的元信息连接（join）来查看歌曲的标题。

```python
tmp = usage_data.groupby(['song_id'], {'total_listens': gl.aggregate.SUM('listen_count'),
                                       'num_unique_users': gl.aggregate.COUNT('user_id')})
tmp.join(songs, ['song_id']).topk('total_listens')
```

```
+--------------------+------------------+---------------+
|      song_id       | num_unique_users | total_listens |
+--------------------+------------------+---------------+
| SOBONKR12A58A7A7E0 |       6412       |     54136     |
| SOAUWYT12A81C206F1 |       7032       |     49253     |
| SOSXLTC12AF72A7F54 |       6145       |     41418     |
| SOEGIYH12A6D4FC0E3 |       5385       |     31153     |
| SOFRQTD12A81C233C0 |       8277       |     31036     |
| SOAXGDH12A8C13F8A1 |       6949       |     26663     |
| SONYKOW12AB01849C9 |       5841       |     22100     |
| SOPUCYA12A8C13A694 |       3526       |     21019     |
| SOUFTBI12AB0183F65 |       2887       |     19645     |
| SOVDSJC12A58A7A271 |       2866       |     18309     |
+--------------------+------------------+---------------+
+--------------------------------+--------------------------------+
|             title              |            release             |
+--------------------------------+--------------------------------+
|         You're The One         |       If There Was A Way       |
|              Undo              |        Vespertine Live         |
|            Revelry             |       Only By The Night        |
| Horn Concerto No. 4 in E f ... | Mozart - Eine kleine Nacht ... |
|         Sehr kosmisch          |       Musik von Harmonia       |
| Dog Days Are Over (Radio Edit) | Now That's What I Call Mus ... |
|            Secrets             |           Waking Up            |
|             Canada             |        The End Is Here         |
|            Invalid             |         Fermi Paradox          |
|        Ain't Misbehavin        |           Summertime           |
+--------------------------------+--------------------------------+
+--------------------------------+------+------------+
|          artist_name           | year | love_count |
+--------------------------------+------+------------+
|         Dwight Yoakam          | 1990 |     0      |
|          Bj\xc3\xb6rk          | 2001 |     0      |
|         Kings Of Leon          | 2008 |     0      |
| Barry Tuckwell/Academy of  ... | None |     0      |
|            Harmonia            | None |     0      |
|     Florence + The Machine     | None |     0      |
|          OneRepublic           | 2009 |     0      |
|        Five Iron Frenzy        | None |     0      |
|            Tub Ring            | 2002 |     0      |
|           Sam Cooke            | None |     0      |
+--------------------------------+------+------------+
[10 rows x 8 columns]
```

`usage_data`表已经是适合作为推荐算法输入的格式了，因为它有用户和歌曲的id以及一些度量用户对歌曲喜好程度的量（用户收听次数）了。
这里有个问题是当用户收听一首歌非常多次这会让推荐产生偏差。在某些时候，当用户收听一首歌足够多次后就说明他们的确非常喜欢这首歌了。
或许我们可以将`listen_count`转换成打分值，操作如下：

```python
s = usage_data['listen_count'].sketch_summary()
import numpy
buckets = numpy.linspace(s.quantile(.005), s.quantile(.995), 5)
def bucketize(x):
    cur_bucket = 0
    for i in range(0,5):
        cur_bucket += 1
        if x <= buckets[i]:
            break
    return cur_bucket
usage_data['rating'] = usage_data['listen_count'].apply(bucketize)
usage_data
```

```
Columns:
        user_id str
        song_id str
        listen_count    int
        rating  int

Rows: 2000000

Data:
+--------------------------------+--------------------+--------------+--------+
|            user_id             |      song_id       | listen_count | rating |
+--------------------------------+--------------------+--------------+--------+
| b80344d063b5ccb3212f76538f ... | SOAKIMP12A8C130995 |      1       |   1    |
| b80344d063b5ccb3212f76538f ... | SOBBMDR12A8C13253B |      2       |   2    |
| b80344d063b5ccb3212f76538f ... | SOBXHDL12A81C204C0 |      1       |   1    |
| b80344d063b5ccb3212f76538f ... | SOBYHAJ12A6701BF1D |      1       |   1    |
| b80344d063b5ccb3212f76538f ... | SODACBL12A8C13C273 |      1       |   1    |
| b80344d063b5ccb3212f76538f ... | SODDNQT12A6D4F5F7E |      5       |   2    |
| b80344d063b5ccb3212f76538f ... | SODXRTY12AB0180F3B |      1       |   1    |
| b80344d063b5ccb3212f76538f ... | SOFGUAY12AB017B0A8 |      1       |   1    |
| b80344d063b5ccb3212f76538f ... | SOFRQTD12A81C233C0 |      1       |   1    |
| b80344d063b5ccb3212f76538f ... | SOHQWYZ12A6D4FA701 |      1       |   1    |
|              ...               |        ...         |     ...      |  ...   |
+--------------------------------+--------------------+--------------+--------+
[2000000 rows x 4 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```


#### 处理复杂数据类型

SArray是强类型的，某些操作只能在特定数据类型上起效。在本指南中有两种数据类型值得特别关注：`list`和`dict`。
这两种数据类型的值可以是任何SArray支持的数据类型，包括它们自己本身。所以，你可以创建一个某列值为
字典类型，并且这个字典的值可以是字典的列表的列表及字符串和整数...

我们使用的数据集目前还不包含任何这种类型的数据，所以我们可以展示如何将一列或多列转换成
这些可迭代类型。比如，假设我们需要一个专辑清单，其中包含每个专辑中所有歌曲的列表，
我们可以像这样来实现：

```python
albums = songs.groupby(['release','artist_name'], {'tracks': gl.aggregate.CONCAT('title'),
                                                   'years': gl.aggregate.CONCAT('year')})
albums
```



```
+--------------------------------+--------------------------------+
|          artist_name           |            release             |
+--------------------------------+--------------------------------+
|          Veruca Salt           |     Eight Arms To Hold You     |
|  Les Compagnons De La Chanson  | Heritage - Le Chant De Mal ... |
| Nelly / Fat Joe / Young Tr ... |             Sweat              |
|           The Grouch           |       My Baddest B*tches       |
|         Ozzy Osbourne          | Diary of a madman / Bark a ... |
|        Peter Hunnigale         |      Reggae Hits Vol. 32       |
|         Burning Spear          |      Studio One Classics       |
|              Bond              |        New Classix 2008        |
| Lee Coombs feat. Katherine ... |            Control             |
|         Stevie Wonder          |    Songs In The Key Of Life    |
+--------------------------------+--------------------------------+
+--------------------------------+--------------------------------+
|             tracks             |             years              |
+--------------------------------+--------------------------------+
| [\'One Last Time\', \'With ... | array('d', [1997.0, 1997.0 ... |
| ['I Commedianti', 'Il Est  ... |              None              |
|       ['Grand Hang Out']       |      array('d', [2004.0])      |
| ['Silly Putty (Zion I Feat ... |      array('d', [1999.0])      |
| ["You Can\'t Kill Rock And ... | array('d', [1981.0, 1983.0 ... |
|        ['Weeks Go By']         |              None              |
|        ['Rocking Time']        |      array('d', [1973.0])      |
|   ['Allegretto', 'Kashmir']    |              None              |
| ['Control (10Rapid Remix)' ... |              None              |
| ['Ngiculela-Es Una Histori ... |      array('d', [1976.0])      |
|              ...               |              ...               |
+--------------------------------+--------------------------------+
[230558 rows x 4 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```

[CONCAT](https://dato.com/products/create/docs/graphlab.data_structures.html#module-graphlab.aggregate)聚合器
将每个分组上指定列的所有值连接成一个列表。增加年份列用于调试目的，因为不知道数据集中同一发行专辑中的每一首歌是不是同一年的，
仅仅通过第一行数据我们就知道不是的：

```python
albums[0]
```

```
{'artist_name': 'Veruca Salt',
 'release': 'Eight Arms To Hold You',
 'tracks': ['With David Bowie',
  'One Last Time',
  'The Morning Sad',
  'Awesome',
  'Stoneface',
  "Don't Make Me Prove It",
  'Straight',
  'Earthcrosser',
  'Benjamin',
  'Volcano Girls',
  'Shutterbug',
  'Sound Of The Bell',
  'Loneliness Is Worse',
  'Venus Man Trap'],
  'years': array('d', [1997.0, 1997.0, 1997.0, 1997.0, 1997.0, 1997.0, 1994.0, 1997.0, 1997.0, 1997.0, 1997.0, 1997.0, 1997.0, 1997.0])}
```

鉴于上面这样的复杂性（同一专辑中的每首歌发行年份会有不同），也为了演示的目的，
我们将'tracks'列使用字典类型，其中键位年份，值为音乐列表。

```python
albums = songs.groupby(['release','artist_name','year'], {'tracks':gl.aggregate.CONCAT('title')})
albums = albums.unstack(column=['year','tracks'], new_column_name='track_dict')
albums
```



```
+--------------------------------+--------------------------------+
|          artist_name           |            release             |
+--------------------------------+--------------------------------+
|          Veruca Salt           |     Eight Arms To Hold You     |
|  Les Compagnons De La Chanson  | Heritage - Le Chant De Mal ... |
| Nelly / Fat Joe / Young Tr ... |             Sweat              |
|           The Grouch           |       My Baddest B*tches       |
|         Ozzy Osbourne          | Diary of a madman / Bark a ... |
|        Peter Hunnigale         |      Reggae Hits Vol. 32       |
|         Burning Spear          |      Studio One Classics       |
|              Bond              |        New Classix 2008        |
| Lee Coombs feat. Katherine ... |            Control             |
|         Stevie Wonder          |    Songs In The Key Of Life    |
+--------------------------------+--------------------------------+
+--------------------------------+
|           track_dict           |
+--------------------------------+
| {1994: [\'Straight\'], 199 ... |
|              None              |
|   {2004: ['Grand Hang Out']}   |
|     {1999: ['Simple Man']}     |
| {1986: [\'Secret Loser\'], ... |
|              None              |
|    {1973: ['Rocking Time']}    |
|              None              |
|              None              |
| {1976: ['Have A Talk With  ... |
|              ...               |
+--------------------------------+
[230558 rows x 3 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```

`unstack`方法实质上是一个在其他所有列上调用带`CONCAT`聚合器`groupby`方法的方便函数。
这一操作可以通过`stack`方法还原，详细信息这里不在赘述。
（注：[graphlab.SFrame.unstack](https://dato.com/products/create/docs/generated/graphlab.SFrame.unstack.html)
函数原型为`SFrame.unstack(column, new_column_name=None)`，按照column指定的列之外的其他所有列聚合，
并将column列指定的列连接成一列，column参数为str型时（即指定一个列名），如果column指定的列为数值型，
连接成的新列为数组，如果column为非数值型，连接成的新列为列表；column参数为list类型（即指定两个列名）时，
连接成的新列为字典，键位column参数指定的第一个列的值，值为第二个列的值。）

这样组织我们的数据以后，我们就可以获取一些关于有多少专辑里面的歌曲年份不同的统计信息，可以看出专辑的这种情况相当流行：

```python
albums['num_years'] = albums['track_dict'].item_length()
albums['num_years'].show()
```

[<img alt="Categorical View of Number of Years per Album" src="images/sframe_user_guide_3_categorical.png" style="max-width: 70%; margin-left: 15%;" />](images/sframe_user_guide_3_categorical.png)

我们也可以通过一些字典操作恢复track列表，假如我们需要同时用`list`和`dict`形式表示track。

```python
albums['track_list'] = albums['track_dict'].dict_values()
albums
```

（注：albums['track_dict']是SArray类型，dict_values()是SArray对象的方法，
可以参考[SArray](https://dato.com/products/create/docs/generated/graphlab.SArray.html)）

```
+--------------------------------+--------------------------------+
|          artist_name           |            release             |
+--------------------------------+--------------------------------+
|          Veruca Salt           |     Eight Arms To Hold You     |
|  Les Compagnons De La Chanson  | Heritage - Le Chant De Mal ... |
| Nelly / Fat Joe / Young Tr ... |             Sweat              |
|           The Grouch           |       My Baddest B*tches       |
|         Ozzy Osbourne          | Diary of a madman / Bark a ... |
|        Peter Hunnigale         |      Reggae Hits Vol. 32       |
|         Burning Spear          |      Studio One Classics       |
|              Bond              |        New Classix 2008        |
| Lee Coombs feat. Katherine ... |            Control             |
|         Stevie Wonder          |    Songs In The Key Of Life    |
+--------------------------------+--------------------------------+
+--------------------------------+-----------+--------------------------------+
|           track_dict           | num_years |           track_list           |
+--------------------------------+-----------+--------------------------------+
| {1994: [\'Straight\'], 199 ... |     2     | [[\'Straight\'], [\'With D ... |
|              None              |    None   |              None              |
|   {2004: ['Grand Hang Out']}   |     1     |      [['Grand Hang Out']]      |
|     {1999: ['Simple Man']}     |     1     |        [['Simple Man']]        |
| {1986: [\'Secret Loser\'], ... |     3     | [["You Can\'t Kill Rock An ... |
|              None              |    None   |              None              |
|    {1973: ['Rocking Time']}    |     1     |       [['Rocking Time']]       |
|              None              |    None   |              None              |
|              None              |    None   |              None              |
| {1976: ['Have A Talk With  ... |     1     |   [['Have A Talk With God']]   |
|              ...               |    ...    |              ...               |
+--------------------------------+-----------+--------------------------------+
[230558 rows x 5 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```

上面我们仅仅恢复了旧的列，但是我们仅仅获得的一个列表的列表而不是一个列表，可以通过`apply`方法解决这个问题：

```python
import itertools
albums['track_list'] = albums['track_list'].apply(lambda x: list(itertools.chain(*x)))
albums
```



```
+--------------------------------+--------------------------------+
|          artist_name           |            release             |
+--------------------------------+--------------------------------+
|          Veruca Salt           |     Eight Arms To Hold You     |
|  Les Compagnons De La Chanson  | Heritage - Le Chant De Mal ... |
| Nelly / Fat Joe / Young Tr ... |             Sweat              |
|           The Grouch           |       My Baddest B*tches       |
|         Ozzy Osbourne          | Diary of a madman / Bark a ... |
|        Peter Hunnigale         |      Reggae Hits Vol. 32       |
|         Burning Spear          |      Studio One Classics       |
|              Bond              |        New Classix 2008        |
| Lee Coombs feat. Katherine ... |            Control             |
|         Stevie Wonder          |    Songs In The Key Of Life    |
+--------------------------------+--------------------------------+
+--------------------------------+-----------+--------------------------------+
|           track_dict           | num_years |           track_list           |
+--------------------------------+-----------+--------------------------------+
| {1994: [\'Straight\'], 199 ... |     2     | [\'Straight\', \'With Davi ... |
|              None              |    None   |              None              |
|   {2004: ['Grand Hang Out']}   |     1     |       ['Grand Hang Out']       |
|     {1999: ['Simple Man']}     |     1     |         ['Simple Man']         |
| {1986: [\'Secret Loser\'], ... |     3     | ["You Can\'t Kill Rock And ... |
|              None              |    None   |              None              |
|    {1973: ['Rocking Time']}    |     1     |        ['Rocking Time']        |
|              None              |    None   |              None              |
|              None              |    None   |              None              |
| {1976: ['Have A Talk With  ... |     1     |    ['Have A Talk With God']    |
|              ...               |    ...    |              ...               |
+--------------------------------+-----------+--------------------------------+
[230558 rows x 5 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```

现在我们也可以从列表或字典中过滤元素了。下面移除1994年和1999年的所有歌曲：

```python
albums['track_dict'] = albums['track_dict'].dict_trim_by_keys([1994, 1999])
albums
```

（注：参见[graphlab.SArray.dict_trim_by_keys](https://dato.com/products/create/docs/generated/graphlab.SArray.dict_trim_by_keys.html)
函数原型`SArray.dict_trim_by_keys(keys, exclude=True)`）

```
+--------------------------------+--------------------------------+
|          artist_name           |            release             |
+--------------------------------+--------------------------------+
|          Veruca Salt           |     Eight Arms To Hold You     |
|  Les Compagnons De La Chanson  | Heritage - Le Chant De Mal ... |
| Nelly / Fat Joe / Young Tr ... |             Sweat              |
|           The Grouch           |       My Baddest B*tches       |
|         Ozzy Osbourne          | Diary of a madman / Bark a ... |
|        Peter Hunnigale         |      Reggae Hits Vol. 32       |
|         Burning Spear          |      Studio One Classics       |
|              Bond              |        New Classix 2008        |
| Lee Coombs feat. Katherine ... |            Control             |
|         Stevie Wonder          |    Songs In The Key Of Life    |
+--------------------------------+--------------------------------+
+--------------------------------+-----------+--------------------------------+
|           track_dict           | num_years |           track_list           |
+--------------------------------+-----------+--------------------------------+
| {1997: [\'With David Bowie ... |     2     | [\'Straight\', \'With Davi ... |
|              None              |    None   |              None              |
|   {2004: ['Grand Hang Out']}   |     1     |       ['Grand Hang Out']       |
|               {}               |     1     |         ['Simple Man']         |
| {1986: [\'Secret Loser\'], ... |     3     | ["You Can\'t Kill Rock And ... |
|              None              |    None   |              None              |
|    {1973: ['Rocking Time']}    |     1     |        ['Rocking Time']        |
|              None              |    None   |              None              |
|              None              |    None   |              None              |
| {1976: ['Have A Talk With  ... |     1     |    ['Have A Talk With God']    |
|              ...               |    ...    |              ...               |
+--------------------------------+-----------+--------------------------------+
[230558 rows x 5 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```

因为字典类型方便用于元素查找，我们可以用字典中的元素进行过滤。下面查询专辑中是否有歌曲是1965年出现的：

```python
albums[albums['track_dict'].dict_has_any_keys(1965)]
```

```
+----------------------------+--------------------------------+
|        artist_name         |            release             |
+----------------------------+--------------------------------+
| Jr. Walker & The All Stars | The Motown Story: The Sixties  |
|        The Seekers         | The Best Sixties Album In  ... |
| Jr. Walker & The All Stars | Sparks Present Motown Made ... |
|          The Who           |           Who's Next           |
|        Bobby Vinton        |    The Best Of Bobby Vinton    |
|     Righteous Brothers     |         The Collection         |
|       Little Milton        |          Chess Blues           |
|          The Who           |          Singles Box           |
|       Dionne Warwick       |           Here I Am            |
|        Albert Ayler        | Nuits De La Fondation Maeg ... |
+----------------------------+--------------------------------+
+--------------------------------+-----------+--------------------------------+
|           track_dict           | num_years |           track_list           |
+--------------------------------+-----------+--------------------------------+
|      {1965: ['Shotgun']}       |     1     |          ['Shotgun']           |
| {1965: ['The Carnival Is O ... |     1     |    ['The Carnival Is Over']    |
|    {1965: ["Cleo\'s Back"]}    |     1     |        ["Cleo\'s Back"]        |
| {1995: ["I Don\'t Even Kno ... |     4     | [\'My Generation\', \'Behi ... |
| {1991: [\'Please Love Me F ... |     6     | [\'Roses Are Red (My Love) ... |
| {2006: ['Let The Good Time ... |     4     | ['Justine', 'Stagger Lee', ... |
| {1965: ["We\'re Gonna Make ... |     1     |    ["We\'re Gonna Make It"]    |
|  {1965: ['Shout And Shimmy']}  |     1     |      ['Shout And Shimmy']      |
| {1965: ['How Can I Hurt Yo ... |     1     | ['How Can I Hurt You? (LP  ... |
| {1969: ['Music Is The Heal ... |     4     | ['Spirits', 'Spirits Rejoi ... |
+--------------------------------+-----------+--------------------------------+
[? rows x 5 columns]
Note: Only the head of the SFrame is printed. This SFrame is lazily evaluated.
You can use len(sf) to force materialization.
```

要转换成`list`或`dict`类型并不一定需要对其他值执行聚合操作。比如由于某种原因我们想将数据集
转换成一个只有一列的表，该列原来所有数据打包成一个列表：

```python
big_list = albums.pack_columns(albums.column_names())
big_list
```

```
+--------------------------------+
|               X1               |
+--------------------------------+
| [\'Veruca Salt\', \'Eight  ... |
| ['Les Compagnons De La Cha ... |
| ['Nelly / Fat Joe / Young  ... |
| ['The Grouch', 'My Baddest ... |
| [\'Ozzy Osbourne\', \'Diar ... |
| ['Peter Hunnigale', 'Regga ... |
| ['Burning Spear', 'Studio  ... |
| ['Bond', 'New Classix 2008 ... |
| ['Lee Coombs feat. Katheri ... |
| ['Stevie Wonder', 'Songs I ... |
|              ...               |
+--------------------------------+
[230558 rows x 1 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```

`upack`方法实现了相反的功能。这些例子虽然有些不自然，但是在你处理如文本之类的非结构化数据的时候会非常有用，
在[文本分析](../text/analysis.md)一章你将会看到。

要了解更多SFrame内容，请查看[API Reference for SFrames](https://dato.com/products/create/docs/generated/graphlab.SFrame.html)和本章最后的[hands-on exercises](exercises.md)。