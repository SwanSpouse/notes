### **kibana简介**

Kibana是一个开源的分析与可视化平台，设计出来用于和Elasticsearch一起使用的。你可以**用kibana搜索、查看、交互存放在Elasticsearch索引里的数据，使用各种不同的图表、表格、地图等kibana能够很轻易地展示高级数据分析与可视化**。

#### Kibana查询语法：

单项term查询:

* 搜Dahlen，Malone

字段field查询:

* field:value例：city:Keyport， age:26

通配符查询：

* ? 匹配单个字符例：H?bbs
* \* 匹配0到多个字符例： H\*
* 注意： ? \* 不能用作第一个字符，例如： ?text \*text

模糊搜索：

* `quikc~ brwn~ foks~` `~`:在一个单词后面加上`~`启用模糊搜索，可以搜到一些拼写错误的单词`first~` 这种也能匹配到 frist

* 还可以设置编辑距离（整数），指定需要多少相似度`cromm~1`会匹配到 from 和 chrome 默认2，越大越接近搜索的原始值，设置为1基本能搜到80%拼写错误的单词

近似搜索：

* 在短语后面加上`~`，可以搜到被隔开或顺序不同的单词

* `"where select"~5` 表示 select 和 where 中间可以隔着5个单词，可以搜到 _select password from users where id=1_

范围查询：

* age:\[20 TO 30\]age:{20 TO 30}

* 注：\[ \] 表示端点数值包含在范围内，{ } 表示端点数值不包含在范围内

逻辑操作:

* AND OR例子：firstname:H\* AND age:20 firstname:H\* OR age:20

* \+ : 搜索结果中必须包含此项
* \- : 不能含有此项
* 例： +firstname:H\* -age:20 city:H\*    firstname字段结果中必须存在H开头的，不能有年龄是20的，city字段H开头的可有可无。

优先级

* `quick^2 fox`使用`^`使一个词语比另一个搜索优先级更高，默认为1，可以为0~1之间的浮点数，来降低优先级

分组：

* \(firstname:H\* OR age:20\) AND state:KS 先查询名字H开头年龄或者是20的结果，然后再与国家是KS的结合

字段分组：

* firstname:\(+H\* -He\*\)搜索firstname字段里H开头的结果，并且排除firstname里He开头的结果

转义特殊字符

* && \|\| ! \(\) {} \[\] ^" ~ \* ? : \
* 注意：以上字符当作值搜索的时候需要用  转义

#### 参考

* [https://blog.csdn.net/hu948162999/article/details/51258257](https://blog.csdn.net/hu948162999/article/details/51258257)
* [https://blog.csdn.net/zhengchaooo/article/details/79500130](https://blog.csdn.net/zhengchaooo/article/details/79500130) 



