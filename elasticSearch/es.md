## 概念



## 索引（Index）

### 分析器（analyzer）

Analyzer 一般由三部分构成，character filters、tokenizers、token filters。掌握了 Analyzer 的原理，就可以根据应用场景配置 Analyzer。

Elasticsearch 有10种分词器（Tokenizer）、31种 token filter，3种 character filter，一大堆配置项。此外，还有还可以安装 plugin 扩展功能。这些都是搭建 analyzer 的原材料。

#### 流程

![image-20200216084547726](.\image-20200216084547726.png)

1. 使用字符过滤器CharacterFilters对文档中的不需要的字符过滤（例如html语言的<br/>等等）
2. 用Tokenizer分词器大段的文本分成词（Tokens）（例如可以空格基准对一句话进行分词）
3. 用TokenFilter在对分完词的Tokens进行过滤、处理（比如除去英文常用的量词：a，the，或者把去掉英文复数等）

<img src=".\analyzerDemo.png" alt="image-20200216084924241" style="zoom:75%;" />

#### 自定义分析器

```js
"settings": {
        "analysis": {
            "char_filter": { ... custom character filters ... },
            "tokenizer":   { ...    custom tokenizers     ... },
            "filter":      { ...   custom token filters   ... },
            "analyzer":    { ...    custom analyzers      ... }
        }
    }
```

如上格式，我们可以基于es自带或ik、pinyin、stconvert等插件分析器定义自定义analyzer。



#### 插件分析器

##### ik

https://github.com/medcl/elasticsearch-analysis-ik

支持自定义词典、停用词

- ik_max_word 细粒度，默认将“华为手机”->“华为手机”。当将“华为手机”添加到自定义词库后，分析结果：“华为”，“手机”，“华为手机”。
- ik_smart 粗粒度，默认将“华为手机”->“华为手机”。当将“华为手机”添加到自定义词库后，分析结果：“华为手机”。
- 索引时，要提高覆盖范围，通常采用ik_max_word分析器，以最细粒度分词；
- 搜索时，要提高准确度，通常采用ik_smart分析器，以粗粒度分词

##### pinyin

https://github.com/medcl/elasticsearch-analysis-pinyin

- `limit_first_letter_length` set max length of the first_letter result, default: 16
- `keep_full_pinyin` when this option enabled, eg: `刘德华`> [`liu`,`de`,`hua`], default: true





##### stconvert



#### multi_fields

目的是给一个字段设置不同的类型或分析器。如下，city是文本类型，city.raw是关键字类型。

```console
"mappings": {
    "properties": {
      "city": {
        "type": "text",
        "fields": {
          "raw": { 
            "type":  "keyword"
          }
        }
      }
    }
  }
```

#### 组合字段

_all字段

将所有字段以空格连接拼成一个大字段。默认不开启false。当不知道文档都有哪些字段时使用。

- 可以通过设置minmum_should_match放宽匹配条件
- 通过copy_to，实现自定义_all字段

_source字段

包含所有字段。不能被index。只是存储。



## 查询（Query）

### 基本查询方式

#### 精确匹配 term

精确查询，不对query进行分词。

#### 全文检索 match

模糊匹配，先对query分词为term；再用分词进行查询。默认匹配要求：只要有一个分词查到就算匹配。

#### 短语全文检索 match_phrase

介于以上两种方式之间。先对query分词，再用分词查询。默认匹配要求：所有分词必须查到并出线顺序相同。可通过slop参数放松要求。

### 多字段查询（mult_match）

在多个字段（field）中查询。评分方法：

- best_field 使用场景：匹配度最高的字段作为多字段查询的匹配度
- most_field 使用场景：匹配的字段越多越好
- cross_field 使用场景：

### 查询组合（bool query）

- should (or) 默认方式
- must（and）
-  must_not





## 实例用法

### 逻辑操作 operator

查询中包含张 且 包含三

{
    "query":{
        "match":{
            "name":{
                "query":"张三",
                "operator":"and"
            }
        }
    }
}