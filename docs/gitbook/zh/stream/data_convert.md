# 如何将Kafka中JSON/CSV转化为表

在大多数系统里，Kafka的value都是json格式的，当然也有其他格式。这里我们会重点介绍如何使用MLSQL自动将json/csv展开成
表，方便后续操作。


我们先看如下一段代码：

```sql
-- the stream name, should be uniq.
set streamName="streamExample";

-- mock some data.
set data='''
{"key":"yes","value":"a,b,c","topic":"test","partition":0,"offset":0,"timestamp":"2008-01-24 18:01:01.001","timestampType":0}
{"key":"yes","value":"d,f,e","topic":"test","partition":0,"offset":1,"timestamp":"2008-01-24 18:01:01.002","timestampType":0}
{"key":"yes","value":"k,d,j","topic":"test","partition":0,"offset":2,"timestamp":"2008-01-24 18:01:01.003","timestampType":0}
{"key":"yes","value":"m,d,z","topic":"test","partition":0,"offset":3,"timestamp":"2008-01-24 18:01:01.003","timestampType":0}
{"key":"yes","value":"o,d,d","topic":"test","partition":0,"offset":4,"timestamp":"2008-01-24 18:01:01.003","timestampType":0}
{"key":"yes","value":"m,m,m","topic":"test","partition":0,"offset":5,"timestamp":"2008-01-24 18:01:01.003","timestampType":0}
''';

-- load data as table
load jsonStr.`data` as datasource;

-- convert table as stream source
load mockStream.`datasource` options 
stepSizeRange="0-3"
and valueFormat="csv"
and valueSchema="st(field(column1,string),field(column2,string),field(column3,string))"
as newkafkatable1;
```

在最后一个语句里，我们在加载数据流的时候，额外增加了 valueFormat和valueSchema.

valueFormat告诉系统，Kafka value的存储格式是什么，在示例中是csv, valueSchema则告诉系统，里面的数据schema是什么。

为了方便的描述schema, MLSQL提供了一套简单的语法：

```
st(field(column1,string),field(column2,string),field(column3,string))
```

st 表示StructType, field表示StructField。 分别表示表和字段，里面则是表明和数据类型。schema支持如下的数据结构：

1. st
1. field
1. string
1. float
1. double
1. integer
1. short
1. date
1. binary
1. map
1. array

比如，你也可以这么描述一个json的数据：{"column1":{"key":"value"}}

```
st(field(column1,map(string,string)))
```

当然也可以描述的更复杂一些，支持st的嵌套。

```
st(field(column1,map(string,array(st(field(columnx,string))))))
```





