# Elasticsearch search API

    实现对es中存储的数据进行分析
    可以指定索引进行查询
    可以使用通配符进行查询
    查询的两种的主要的方式
        -URI search:操作简便，方便通过命令行进行测试，仅包含部分的查询语法
            GET my_index/_search?q=user:panda
        request body search:完备的Query DSL查询语法
            GET my_index/_search
            {
                "query":{
                    "terms":{
                        "user":"panda"
                    }
                }
            }

## URI search

    常用参数
        -q:制定查询的语句，语法为Query String Syntax
        -df:q中不指定字段时默认查询的字段，如果不指定，es会查询所有的字段
        -sort:排序
        -timeout:制定超时时间，默认超时
        -from,size用于分页
    demo: GET my_index/_search?q=panda?df=user&sort=age:asc&from=4&size=10&timeout=15s

### Query String Syntax

    term 与 phrase
        panda way 等效于 panda or way
        "panda way"词语查询要求先后顺序
    泛查询
        panda 等效于在所有的字段中匹配该terms
    制定字段
        name:panda
    Group分组设定，使用括号指定匹配的规则

    布尔操作符
        -AND OR NOT
            name:(tom NOT li):注意大小写
        - +，-分别对应must or must not
        注： + 号在url中会被识别为空格，需要转义 %2B
            加不加括号查询的结果是不一样的
    范围查询，支持数值和日期
        -区间写法：[]闭区间，{}开区间
        -算数符号写法
    通配符查询：
        -？代表一个字符，×代表一个或者多个字符
        -效率较低
        -若无特殊需求，不要将?*放在最前面
    正则表达式
    模糊匹配：～
        1.单词 以一个character的词 
        2.短语 “××× ×××”：以terms为单位进行差异化比较

### Request Body Search

    Query DSL语法
    -from，size
    -timeout
    -sort

#### Query DSL

    基于json定义的一套查询语言
    -字段类查询
        term,match,range等，针对某一个字段进行查询
    -复合查询
        bool等包含一个或者多个字段类查询或者复合查询语句

##### 字段类查询

    -全文匹配
        针对text类型的字段进行全文检索，会对查询进行分词处理，在匹配es terms的倒排索引，如match,match_phrase等query类型
    -单词匹配
        不会进行分词，直接进行字段倒排索引的匹配，如term,terms,range等

###### Match Query流程

        1.对查询语句进行分词
        2.根据字段进行相关性算分
        3.将算分进行汇总
        4，根据得分进行排序，返回匹配的文档
    可以通过operator控制单词之间的匹配关系
    可以通过minimum_should_match匹配最少需要多少个匹配的单词数目

##### 相关性算分

    -词频(TF)：词频越高，相关度越高
    -文档频率（DF）：单词出现的文档数
    -逆向文档频率（IDF）:单词出现的文档频率越小，相关度越高
    -Field length Norm:文档越短，相关度越高
    1.TF/IDF:
        score(q,d)=coord(q,d)*queryNorm(q)*(求和t in q)(tf(t in d)*idf(t)2*boost(t)*norm(t,d))
        可以通过explain参数来查看具体的的计算方法
        注：es的算分是按照shard进行的，，可以通过设置number_of_shards：1避免该问题
    2.BM25:
        最佳匹配，迭代25词才进行计算
        score(D,Q) = (求和 1 to n)IDF(qi) * (f(qi,D)*(k+1)) / (f(qi,D)+k1*(1-b+ (|D|/avgdl))

##### Match Phrase Query

    对字段进行索引，有顺序要求
    可以通过slop参数控制单词的键的间隔

##### Query String Query

##### Term Query

    回去倒排索引中查找对应的term，不会进行分词

##### range Query

    主要针对数值和日期
    -gt >
    -gte >=
    -lt <
    -lte <=
    支持 Date Math :now -10d/d

### 复合查询

    包含字段查询和符合查询的类型
    -constant_score query
        把查询文档中的文档得分都设定为1或者boost的值，用于进行bool查询实现自定义得分
    -bool query
        filter:只过滤符合条件的文档，不会进行相关性算分
            es针对filter会有智能的缓存，之后执行的效率很高
            不考虑相关性算分时，推荐使用
        must:必须符合must中所有的条件
        must_not:
        should:文档可以符合条件
            minimum_should_match控制匹配的最少数量
    -dis_max query
    -function_score query
    -boosting query

    filter和query的区别：
    filter不会进行相关性算分
    query会进行相关性算分并排序
    将不进行相关性算分的字段存放在filter当中，可以提升效率

### count Api

    获取符合条件的文档数量

### source Filtering

    控制返回source中的某些字段
    -url参数
    -禁用返回source
    —_source：[]
    -模糊匹配
