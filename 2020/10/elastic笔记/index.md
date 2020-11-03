# Elastic笔记


## 全文检索

是指在全文数据中检索单个文档或文档集合的搜索技术。

全文数据：像文章、网页、邮件这种全文本数据
文档：全文本数据中的一条数据

## 倒排索引

倒排索引先将文档中包含的关键字全部提取出来，然后将关键字与文档的对应关系保存起来，最后再对关键字本身做索引排序

## elasticsearch 索引

elasticsearch 中所有数据的检索都必须要通过倒排索引来检索。所以将文档的容器直接称为索引，这里的索引就是倒排索引，更准确的说是一组倒排索引。

## 分区

为了避免重新索引导致的严重性能开销，elasticsearch 对索引分片数量做了一个严格的限制，就是索引分片数量一旦在创建索引时确定后就不能再修改了。

但无论容量规划得多科学依然不能完全避免文档实际存储量与索引容量不想等的情况。唯一可行的情况就是创建新的索引，再将原索引中的文档存储到新索引中

### 索引别名

`_alias` 和`_aliases` 接口设置别名

`_rollover` 用于根据一系列条件将别名指向一个新的索引

## 索引配置

setting
索引关闭和打开
`_close` ,`_open`

## 动态字段

dynamic:

-   true 默认，支持动态增加新字段
-   false,新字段依然会被保存到文档中，只是这个字段的定义不会被添加到索引映射的字段字段定义中。

## 动态模板

用于自定义动态添加字段时的映射规则。通过索引映射类型的 dynamic_templates 参数设置

## 索引模板

在创建索引时默认为索引添加的别名，配置，和映射关系等。分别通过`index_patterns`,`aliases`,`setting`,`mapping`设置。可通过`_template接口创建`

## `_split` 接口

在新索引中将每个主分区分裂为两个或更多分区，所以分区数量都是成倍增加而不能逐个增加。分裂虽然会创建新的索引，但是新索引中的数据只是通过文件系统硬连接到新的索引中，所以不存在复制。分片又是在本地分裂，不存在网络传输，所以效率还是比较高的。
需要预先设置 number_of_routing_shards 参数

## `_shrink`接口

用于缩减索引分片

## `_reindex`接口

将文档从一个索引重新索引到另一个索引中。不会将原索引的配置信息复制到新索引中。必须将索引的`_source`字段开启

## `_refresh`接口

主动刷新一个或多个索引，将已经添加的文档编入索引使它们在检索时可见

## `_cache`接口

用于主动清理缓存，需要在`_cache`后附加关键字`clear`。

```
clear?query=true
clear?request=true
clear?fielddata=true&fields=notes
```

-   query:节点查询缓存，负责存储节点查询的结果。一个节点只有一个缓存，同一个节点上的分片共享一个缓存。默认开启。默认使用节点内存的 10%作为上线
-   request：分片请求缓存，负责存储分片接受到的查询结果。不会缓存查询结果的 hits 字段，也就是具体的文档内容，它一般 之缓存聚集查询的相关结结果。默认开启。
-   fielddata：将 text 类型字段提取到所有词项全部加载到内存中，以提高使用该字段做排序和聚集运算的效率。由于 fielddata 是 text 对文档值机制的替代，所以天然开启且无法关闭。

## `_stats`接口

查看运行状态

## `_shard_stores`和`_segments`接口

`_shard_stores`用于查询索引分片存储情况
`_segments`用于查看底层 Lucene 的分段情况

## 索引文档

将文档分析处理后编入索引以使文档可检索。

### 获取文档

-   获取单个文档
    如`GET /test/_doc/1`
-   获取多个文档
    如
    ```
    GET _mget
    {
        "docs":[
            "_index":"students",
            "_id":"1",
            "_source":{
                "include":["name"],
                "exclude":["gender"]
            }
        ]
    }
    ```

### 删除文档

-   根据 id
-   根据查询条件

### 更新文档

-   `_update`接口：用于解决更新文档单个字段的问题。困纯属 doc 参数指明要更新的字段。还可以用 scipt 参数设置更新脚本，一般用 Painless 语言
-   `_update_by_query`接口：根据查询条件更新

### 批量操作

`_bulk`接口：接受一族请求体，请求体一般分为两组，第一个代表操作文档的类型，第二个代表操作文档所需的参数。

## 分析与检索

### \_search 接口

两种方式：基于 URI 和基于请求体

-   基于 URI：

    ```
    GET index/_search?q=message:chrome firefox
    ```

-   基于请求体：
    `GET index/_saerch { "query":{ "term":{ "DestCountry":"CN" } } }`

`mat_all`和`match_none`:

```
GET index/_saerch
{
    "query":{
        "match_all":{}
    }
}
```

### 分页与排序

`from`参数代表索引文档的起始位置，`size`代表每次检索文档的数量，默认为 10。
form 和 size 处理时会将所有的数据全部取出来，再截取范围内返回。

`scroll`参数类似数据库游标的文档遍历机制。

```
POST index/_search?scroll=2m&size=1000
{
    "query":{
        "term":{
            "message":"chrome"
        }
    }
}
```

scroll 参数只能在 URI 中使用，2m 代表我分钟，1h 代表 1 小时。这个保留时长是处理单次遍历所需要的时间。返回结果包含了一个`scroll_id`，下次根据这个 id 进行查询就可以遍历了。

```
POST _search/scroll
{
    "scroll":"2m",
    "scroll_id":"..."
}
```

scoll 也会将数据整体加载进来，不同的是 from/size 每次请求都会加载，scroll 只在初始时加载。

`search_after`定义检索应该在文档某些字段的值之后查询其他文档。

`sort`参数代表排序。

`_source`代表字段投影，只返回需要的字段，也可以设置`include`和`exclude`字段。

`stored_fields`指定哪些被存储的字段出现在结果中。当然这些字段的`store`属性要设为 true，默认不返回`_source`

`docvalue_fields`用于将文档字段以文档值机制保存的值返回。返回的结果默认会包含`_source`字段

`script_fields`可以通过脚本向检索结果中添加字段。

### 分析器与规整器

文档分析器是用于文档分析的组件，通常由字符过滤器、分词器和分词过滤器组成。它们就像连接在一起的管道。

除了分析器外，还有一种称为规整器的文档标准化工具。与分析器最大的区别在于规整器没有分词器，所以它能保证分析后的结果只有一个分词。文档规整器只能应用于字段类型为 keyword 的字段，可通过 normalizer 参数配置字段规整器。规整器的作用就是对 keyword 字段做标准化处理，比如将字段值转换为小写字母等等。

### standard 分析器

分词规则是根据 Unicode 文本分隔规范中定义的标准分隔符区分词项。
没有字符过滤器，但包含三个词项过滤器：

-   标准词项过滤器：只是占位，实际没有任何处理
-   小写字母过滤器：将词项转换为小写字母
-   停止词过滤器：将停止词删除，默认关闭

### stop 分析器

使用小写字母分词器，分词规则是使用所有非字母分隔单词。
没有字符过滤器，但包含一个停止词过滤器。

### pattern 分析器

使用模式分词器，使用 Java 正则表达式匹配文本以提取词项。
没有字符过滤器，包含两个分词过滤器：小写字母过滤器和停止词过滤器

### simple 分析器

使用小写字母分词器，没有过滤器

### keyword 分析器

不做任何处理的分析器，使用的分词器是关键字分词器。

### 中文分析器

IK

-   ik_smart:提取颗粒度最粗
-   ik_max_word:提取颗粒度最细

# 叶子查询与模糊查询

## 基于词项的查询

精确匹配查询条件，不会对查询条件做分词、规范化等预处理。所以一般不对 text 类型字段做检索，而用于类似数值、日期、枚举类型等结构化数据的精确匹配。

-   term 查询
    对字段做单词项的精确匹配
-   terms 查询
    类似 in 操作，在一组指定的词项范围内匹配字段值
-   terms_set 查询
    与 terms 类似，不同的是被匹配的字段类型是数组，接受以数组类型表示的多个词项，被匹配的字段只要包含期望词项中的几个即可。具体数量有两种设置方式，一种是通过`minimum_should_match_field`指定，另一种通过`minimum_should_match_script`

-   range 查询
    用于匹配一个字段是否在指定范围内，一般用于具有数字、日期等结构化数据类型的字段
-   exists 查询
    用于检索指定字段不为空的文档。
    在验证非空时需要明确什么样的值是空，什么值不是空。默认情况下，字段空值与 java 语言的空值相同都是 null.空值字段不会被索引，也就不可检索

### 使用模式匹配

-   prefix 查询
    检索字段值中包含指定前缀的文档
-   wildcard 查询
    允许在查询条件中使用通配符`*`和`?`，`*`代表 0 个或多个字符，`?`代表单个字符
-   regexp 查询
    正则查询

---

-   type 查询:根据映射类型做查询，由于 6.0 版本后只能定义一个`_doc`映射类型，故没用了
-   ids 查询：根据一组 id 值查询

---

-   停止词：
    出现在文档中但并不会编入索引中的词项就是停止词。一般在定制分析器时预先定义好，文档在编入索引时分析器就会将这些停止词从文档中剔除。
-   common 查询
    将词项分为重要词项和非重要词项。使用`cutoff_frequency`设置词频来判定是否是重要词项，

## 基于全文的查询

### 词项匹配

-   match 查询
    接受文本、数值和日期类型值。在检索时将查询条件做分词处理再以提取出来的词项与字段做匹配。如果提取出来的词项为多个，词项与词项之间的匹配结果按布尔或运算。
    词项匹配的运算逻辑和匹配个数通过`operator`和`minimum_should_match`来改变
-   multi_match
    与 match 类似，但可以实现对多字段的同时匹配

---

### 短语匹配

-   match_phrase 查询
    将查询条件按顺序分词，然后再查看他们在字段中的位置之差，只有差都为 1 才满足查询条件，换句话说就是这些词项要紧挨着。
-   match_phrase_prefix 查询
    在这种匹配中，最后一个词项可以设置为前缀。

---

### 查询字符串

查询字符串是具有一定逻辑的字符串，因此不会直接分词提取，而是先通过某种类型的解析器解析为逻辑操作符和更小的字符串。

-   query_string 查询
    是基于请求体查询字符串的形式，与基于 URI 时使用的请求参数 q 没有本质区别。可以用`field_name:query_term`的形式
-   simple_query_string
    是对 query_string 查询的简化，体现在解析字符串是会忽略异常。

---

### 间隔查询：

可以定义一组词项或短语组成的匹配规则，然后按顺序在文本中检查这些规则

-   intervals
    主要包括 all_of、any_of 和 match 三个参数

---

### 模式查询

-   fuzzy 查询
    模糊查询
-   suggest
    纠错和提示。
    提示器：
    -   term：会将需要提示的文本拆分成词项，然后对每一个词项做单独的提示
    -   phrase：使用整个文本内容做提示
    -   completion 提示器：同于输入提示和自动补全。需要提示词产生的字段为 completion 类型

---

# 相关性评分与组合查询

## 相关性评分

### 相关度模型

-   布尔模型
-   向量空间模型
-   概率模型
-   语言模型

### TF/IDF

-   TF：term frequency,词频。
-   IDF：invert document frequency,逆向文档频率，指词项在所有文档中出现的次数。

TF 越高，相关度越高，IDF 越高，相关度越低。

### BM25

被认为是当今最先进的相关度算法之一。

### 相关度解释

相关度算法可通过 text 和 keyword 类型字段的 similarity 参数修改，也就是说相关度算法不针对整个文档而是针对单个字段，默认是 BM25。

### 相关度权重

-   boost 参数
    默认是 1，在检索时设置
-   indices_boost 参数
    可调整多索引查询条件时每个索引的权重

## 组合查询与相关度组合

### bool 组合查询

子句类型：

-   must：影响相关度
-   filter：不影响
-   should：影响
-   must_not：不影响

当 should 字句与 must 字句或 filter 字句同时出现时，should 字句不会过滤结果。

### dis_max 组合查询

在计算相关度值时，会在子查询中取最大相关性值为最终相关度分值结果，而忽略其他子查询的相关性得分。
通过 queries 参数接受对象数组，数组元素可以是前面讲解的叶子查询。
可以用 tie_breaker 参数设置其他字段参与相关度运算的系数。

### constant_score 查询

返回结果的相关度为固定值，由 boost 参数设置。
match_all 可以当成一个 boost 为 1 的 constant_score 查询。

### boosting 查询

通过 positive 字句设置满足条件的文档，类似 boost 查询中的 must 字句。通过 negative 字句设置需要排除文档的条件，类似 boost 的 must_not 字句。
不同的是，不会将满足 negative 条件的文档从返回结果中排除，而只是会拉低他们的相关性分值。
参数 negative_boost 设置一个系数，当满足 negative 时相关度会乘这个系数，所以系数要大于 0 小于 1。

### function_score 查询

通过为查询条件定义不同的打分函数实现自定义打分，使用 functions 参数设置打分函数。
打分函数如果有多个，最后打分函数的值由 score_mode 的值来决定。

打分函数运算的相关性评分会与 query 参数中查询条件的相关度组合起来，组合的方式是通过 boost_mode 参数指定。

当只有一个打分函数值，可以直接使用打分函数名称做设置，取代 functions。
几个内置的打分函数：

-   weight：固定值
-   random_score：0-1 之间的随机数
-   script_score：通过脚本得到值，非负
-   field_value_factor：加入某一字段作为干扰因子。
-   衰减函数 ：
    -   gauss（高斯）
    -   linear（线性）
    -   exp（指数函数）

### 相关度组合

query_string 和 multi_match 查询，由于对多个字段设置查询条件，所以需要考虑组合多个相关度的问题。
type 参数指定多个字段的执行逻辑和相关度组合方法。有下面这些值可选：

-   best_field
    取最高分为整个查询的相关度。在执行时会转化为 dis_max 查询

-   phrase 和 phrase_prefix
    执行逻辑与 best_field 完全相同，至于在转化为 dis_max 时 queries 查询中的子查询会使用 phrase 或 phrase_prefix 而不是 match
-   most_field
    会将所有相关度累加，再除以相关度的个数。会转化为 bool 查询的 should 字句
-   cross_field
    会将词项拆分，在效果上不要求字段同时包含多个词项，而要求词项分散在多个字段中。

# 聚集查询

执行聚集查询时参数是`aggregation`或简写`aggs`。

## 指标聚集

### 平均值聚集

-   avg 聚集
-   weighted_avg 聚集：权重值可以从文档的某一字段中获取，也可以通过脚本运算

### 计数聚集和极值聚集

计数聚集同于统计字段值的数量，而极值聚集则是查找字段的极大值和极小值，都支持使用脚本。

-   计数聚集
    `value_count`聚集和`cardinality`聚集可以归入计数聚集，前者用于统计从字段中取值的总数，而后者则用于统计不重复数值的总数
-   极值聚集
    是在文档中提取某一字段的最大值或最小值的聚集，包括 `max` 聚集和 `min` 聚集

### 统计聚集

-   stats 聚集
    stats 聚集返回的结果中包括字段的最大值、最小值、总和、数量和平均值
-   entended_stats 聚集
    增加了几项统计数据

### 百分位聚集

根据文档字段统计字段值按百分比的分布情况，包括`percentiles`聚集和`percentile_ranks`两种。前者统计的是百分比与值的对应关系，后者正好相反，统计值与百分比的对应关系。

## 使用范围分桶

### 数值范围

-   range
    使用 ranges 参数设置多个数值范围
-   date_range
    与 range 类似，只是范围和字段的类型为日期而非数值
-   ip_range
    ip 范围

### 间隔范围

-   histogram
    以数值为间隔定义数值范围，字段值具有相同范围的文档落入同一桶中。
-   date_histogram
    以时间为间隔定义日期范围，字段值具有相同日期范围的文档落入同一桶中。
-   auto_date_histogram
    预先指定需要返回多少个桶，间隔值通过桶的数量及字段值的跨度共同决定。

### 聚集嵌套

前两小节介绍的桶型聚集，他们的结果都只是返回满足聚集条件的文档数量。通过嵌套聚集才能实现强大的功能。
嵌套聚集应该位于父聚集名称下而与聚集类型同级，并且需要 aggs 参数再次声明使用。

## 使用词项分桶

对于字符串类型的字段来说，显然不适合用范围分桶。字符串类型字段的分桶一般是通过词项实现。

### terms 聚集

根据文档中的词项做分桶，所有包含同一词项的文档将被归入同一桶中。默认还会根据词频排序。对于 text 类型的字段需要打开 fielddata 机制。会导致内存消耗大，所以 terms 聚集一般针对 keyword 类型。

与 cardinality 聚集一样，terms 聚集统计出来的词频也不能保证完全精确。

### significant_terms 聚集

它将文档和词项分为前景集和背景集。前景集对应一个文档子集，而背景集对应文档全集。significant_terms 根据 query（和 aggs 同级）指定前景集，运算 field 参数指定字段中短词项在前景集和背景集中的词频总和，并在结果的 doc_count 和 bg_count 中保存它们。

### significant_text 聚集

如果参与 significant_terms 聚集的字段类型是 text 类型，那么 需要打开 fielddata 机制。significant_text 就不需要打开 fielddata 机制。

### 样本

sample 聚集的作用是限定其内部嵌套聚集在运算时采用的样本数量，样本数量是在每个分片上的数量而不是整体数量。提取样本时会按照文档检索的相似度排序，由高到低的顺序提取。
为了降低样本减少对结果的准确性，需要将一些重复的数据从样本中剔除。diversified_sampler 聚集提供了样本多样性的能力。它提供了 field 或 script 两个参数用于去除样本中可能重复的数据。

## 单桶聚集和聚集组合

前三节都是多桶型聚集。

### 单桶聚集

在返回的结果中只会形成一个桶。

-   过滤型聚集
    -   filter
    -   filters（属于多桶聚集）
-   global 聚集
    把索引中所有文档归入一个桶中
-   missing 聚集
    将某一字段缺失的文档归入一桶

### 聚集组合

composite 聚集可以将不类型的聚集组合到一起，它会从不同的聚集中提取数据并以笛卡尔乘积的形式组合它们，而每一个组合就会形成一个新桶。
返回的结果会有一个 after_key 字段，它包含了当前聚集结果最后一个结果的 key.所以下一页聚集结果就可以通过 after 和 size 参数指定。

### 领接矩阵

类似图论中的无向图邻接矩阵。
`adjacency_matrix`
顶点指的是一组过滤条件，而这些条件两两组合就形成了邻接矩阵。所以使用邻接矩阵一定要给定一组过滤条件。

## 管道聚集

可分为基于父聚集结果和基于兄弟聚集结果两类。前者将运算结果加到父聚集结果中，后者展示在自己的聚集结果中。
聚集名称与聚集名称之间的分隔符是`“>”`，聚集名称和指标名称之间的分隔符使用`“.”`

### 基于兄弟聚集

包括 avg_bucket , max_bucket , min_bucket , sum_bucket , stats_bucket , extended_stats_bucket , percentiles_bucket 七种。

### 基于父聚集

包括 moving_avg（已弃用）, moving_fn , bucket_script , bucket_selector , bucket_sort , derivative , cumulative_sum , serial_diff 八种

-   滑动窗口
    moving_avg（已弃用）和 moving_fn 都是基于滑动窗口。
    moving_fn 内置了一个 MovingFunctions 类，包含多个运算函数。
-   单桶运算
    moving_fn 会对父聚集结果中落在窗口内的多个桶做聚集运算，而 bucket_script , bucket_selector , bucket_sort 这三个会针对父聚集结果中的每一个桶做单独的运算
-   特定数学运算
    剩下的 derivative , cumulative_sum , serial_diff 就是用于特定的数学运算。他们只能应用于 histogram 或 date_histogram 父聚集中。
    derivative 求导，cumulative_sum 累计和，serial_diff 时序差分，就是贸易指标值与指定时间段之前值之差。

## 矩阵聚集

matrix_stats,通过 fields 参数接受统计字段的名称。

# 处理特殊数据类型

## 父子关系

### join 类型

在 elasticsearch 中并没有外键的概念，文档之间的父子关系通过给索引定义 join 类型字段实现

```json
PUT employees
{
    "mapping":{
        "properties":{
            "management":{
                "type":"join",
                "relations":{
                    "manager":"member"
                }
            }
        }
    }
}
```

manager 为父而 member 为子，名称可以由用户定义。文档在父子关系中的地位，是在添加文档时通过 join 类型字段指定的。

在使用父子关系时，要求父子文档必须要映射到同一分片中，所以在添加子文档时 routing 参数是必须要设置的。

可在父子关系中使用的查询有 has_child , has_parent 和 parent_id 查询，还有 parent 和 children 两种聚集

### has_child 查询

是根据子文档检索父文档的一种方法。它先根据查询条件将满足条件的子文档检索出来，在最终的结果中会返回具有这些子文档的父文档。

### has_parent 查询

与上面正好相反，是通过父文档检索子文档

### parent_id 查询

和 has_parent 查询类似，都是根据父文档检索子文档。但 parent_id 查询只能通过父文档`_id`做检索。

### children 聚集

如果想通过父文档检索与其关联的所有子文档就可以使用 children 聚集

### parent 聚集

与 children 聚集相反，是根据子文档查找父文档。

## 嵌套类型

### nested 类型

为了解决对象类型在数组中丢失内部字段之间匹配关系的问题。
这种类型会为数组中的每一个对象创建一个单独的文档，一保存对象的字段信息并使它们可检索。这类文档并不直接可见，而是藏匿在父文档之中。

当字段被设置为 nested 类型后，必须用专门的检索方法，包括 nested 查询，还有聚集查询中的 nested 和 reverse_nested 两种聚集。

### nested 查询

nested 查询只能针对 nested 类型字段，需要通过 path 参数指定 nested 类型字段的路径，query 参数包含具体查询条件。

### nested 聚集

是一个单桶聚集，通过 path 字段指定 nested 字段的路径，包含在 path 指定路径中的隐式文档都将落入桶中。

### reverse_nested 聚集

用于在隐式文档中对父文档做聚集，所以这种聚集必须作为 nested 聚集的嵌套聚集使用。

## 处理地理信息

### 地理类型字段

-   geo_point 类型
    可以用经纬度或 GeoHash 编码来表示位置
-   geo_shape 类型
    用于存储地理形状。

### geo_shape 查询

根据 geo_shape 类型来过滤文档。

### geo_bounding_box 查询

和 geo_shape 类型，只是查询条件中指定的形状只能是矩形

### geo_distance 查询

将所有该再传存储点到指定点距离小于某一特定值的文档查询出来。

### geo_polygon 查询

和 geo_bounding_box 类型，但只是定义查询条件为多边形，不要求一定是矩形。

### geohash_grid 聚集

很句 GeoHash 运算区域，并根据文档 geo_point 类型的字段将文档归入不同区域形成的桶中。

### geo_distence 聚集

根据文档中某个坐标字段到指定地理位置的距离分组。

## 使用 SQL 语言

### `_sql`接口

`_sql`接口通过 query 参数接受 SQL 语句，URL 请求参数 format 定义返回的格式。

### 操作符与函数

引入了一个等号比较`<=>`，可以在左值为 null 时不出现异常。
LIKE 字句可以用`%`代表任意多个字符，用`_`代表单个字符。还可以用 RLIKE 使用正则表达式。
还有 MATCH 和 QUERY 实现全文检索。

