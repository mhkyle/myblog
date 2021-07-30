# Vim and vi
[上一篇blog](https://minghuiyuan.github.io/myblog/contents/01-generator_consumer_model)主要记录了Golang语言版本的最简单的“生产者消费者”模型。

本节主要记录一些vi的比较实用的简单操作。

### 删除指定范围行
```
#删除344-1234行
:344,1234d
```

### 删除选中行
```
# v 进入 VISUAL 模式
v
# 上下键选中,然后键入d删除选中
d
```



#### [last: golang生产者消费者模型](https://minghuiyuan.github.io/myblog/contents/01-generator_consumer_model) < > [next: k8s的crds](https://minghuiyuan.github.io/myblog/contents/03-k8s_crds)