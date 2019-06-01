# Mapping 设置

    类似与数据库中的表结构定义，主要作用如下：
    1.定义index下的字段名称
    2.定义字段类型
    3.定义倒排索引相关的配置，比如是否索引和记录position

## 自定义mapping

    mapping中的字段类型一旦设定后，禁止修改
        -lucene实现倒排索引的语法不支持修改
    如何做呢？重新建立新的索引，在reindex(重新导入到新的索引中)

    允许新增字段
    通过dynamic参数来控制字段的新增(全局或者局部)
        -true:允许自动新增字段
        -false:不允许新增字段，但文档可以正常的写入，但对字段无法进行查询等操作
        -strict:文档写入报错

    copy to:
        -将该字段的值复制到目标字段
        -不会出现在——source中，只用来进行一些搜索

    index（字段）：
        控制当前字段是否索引，默认为true,建立索引，false不记录，即不可搜索

    index_options:用于控制倒排索引记录的内容
        -docs: 只记录doc id
        -freq:doc id + term frequencies(词频)
        -positions: + 记录出现的位置
        -offsets:记录开始和结束的位置
        text： 默认为position,其他的为doc id
    null_value:
        用于遇到null值后控制字段输入的默认值

## 数据类型

### 核心数据类型

    字符串：text,keyword
    数值类型：long,integer,byte,short,double,float。。。
    日期类型：date
    布尔类型：boolean
    二进制类型:binary
    范围类型：integer_range,float_range,long_range,double_range,date_range

### 复杂类型

    数组类型:array
    对象类型：object
    嵌套类型： nested object 会被es进行单独的处理
    地理位置相关的数据类型：
        -geo-point
        -geo-shape

### 专用类型

    ip类型
    实现自动补全 completion
    记录分词数
    记录字符串hash值 murmur3
    percolator
    join

### 多字段特性

    允许对同一个字段采用不同的配置，比如分词，常见例子如对人名进行拼音搜索
    可以在人名中新增子字段为拼音即可

### dynamic mapping

    es 可以自动的识别文档的字段类型
    依赖于json文档的字段类型实现自动识别字段类型
        json        es
        null        忽略
        boolean     boolean
        浮点        float
        整数        long
        object      object
        array       又第一个非null值决定
        string      匹配为日期，则设为date（默认开启），数字 float or long(默认关闭)，text ,附带keyword子字段

    日期自动识别可以自行配置日期格式
        -默认["strict_date_option_time":"yyyy/MM/dd HH:mm:ss Z|| yyyy/MM/dd Z"]
        -dynamic_date_formats可以自定义日期类型
        -date_ detection:可以日期自动识别机制
    numeric_detection可以开启字符串中的数字自动识别

### dynamic Template

    es可以通过动态的识别数据类型，字段名称等来动态的设定字段类别，可实现如下效果
        -所有字符串都设定为keyword
        -以message开头的字段类型设置为text类型
        -以long_开头的设置为long类型
        -所有自动匹配为double类型的都设定为float类型
    match_mapping_type:匹配es自动识别字段类型（boolean,long,string）
    match,unmatch:匹配字段名称
    path_match,path_unmatchL匹配路劲

### dynamic Index

    用于创建索引时自动运用预先设定的配置，简化创建缩影的步骤
        -可以设定索引的配置和mapping
        -可以有多个模板，根据order设置,order大的覆盖小的

## 如何自定义mapping

    1.写入一条文档到es的临时索引中
    2.获取mapping
    3.修改mapping
