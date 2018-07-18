---
layout: blog
istop: true
jishu: true
category: Mysql
title: 提升 SQL 的查询效率？来看MySQL SQL 优化之覆盖索引
tags:
- 阿里云
- 双十一
- 云服务器
- SQL
- 查询效率
- 优惠券
- 阿里云优惠
date:   2017-11-01 9:25:54
---

# **[本站福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=9ytvzpwr)**

> 利用索引提升 SQL 的查询效率是我们经常使用的一个技巧，但是有些时候 MySQL 给出的执行计划却完全出乎我们的意料，我们预想 MySQL 会通过索引扫描完成查询，但是 MySQL 给出的执行计划却是通过全表扫描完成查询的，其中的某些场景我们可以利用覆盖索引进行优化。

前些天，有个同事跟我说：“我写了个 SQL，SQL 很简单，但是查询速度很慢，并且针对查询条件创建了索引，然而索引却不起作用，你帮我看看有没有办法优化？”。

我对他提供的 case 进行了优化，并将优化过程整理了下来。

我们先来看看优化前的表结构、数据量、SQL、执行计划、执行时间等。

**1. 表结构：**

```
CREATE TABLE `t_order` (  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,  `order_code` char(12) NOT NULL,  `order_amount` decimal(12,2) NOT NULL,  PRIMARY KEY (`id`),  UNIQUE KEY `uni_order_code` (`order_code`) USING BTREE) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

隐藏了部分不相关字段后，可以看到表足够简单， 并且在 order_code 上创建了唯一性索引 uni_order_code。

**2. 数据量：316977**

这个数据量还是比较小的，不过如果 SQL 足够差，一样会查询很慢。

**3. SQL：**

```
select order_code, order_amount from t_order order by order_code limit 1000;
```

哇，SQL 足够简单，不过有时候越简单也越难优化。

**4. 执行计划：**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

全表扫描、文件排序，注定查询慢！

那为什么 MySQL 没有利用索引（uni_order_code）扫描完成查询呢？因为 MySQL 认为这个场景利用索引扫描并非最优的结果。我们先来看下执行时间，然后再来分析为什么没有利用索引扫描。

**5. 执行时间：260ms**

![](http://mmbiz.qpic.cn/mmbiz_png/DmibiaFiaAI4B36XmwteZwQUaB5ACyKK34Oxt7CkBSS4vHo6F2bSh8KzkD2iaktjWrR5XZiaJS2OEL3WoicfKR6EBvHw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

的确，执行时间太长了，如果表数据量继续增长下去，性能会越来越差。

 

我们来分析下 MySQL 为什么使用全表扫描、文件排序，而没有使用索引扫描、利用索引顺序：

**\*1. 全表扫描、文件排序：***

虽然是全表扫描，但是扫描是顺序的（不管机械硬盘还是 SSD 顺序读写性能都是高的），并且数据量不是特别大，所以这部分消耗的时间应该不是特别大，主要的消耗应该是在排序上。

****

**2. 利用索引扫描、利用索引顺序：**

uni_order_code 是二级索引，索引上保存了（order_code,id），每扫描一条索引需要根据索引上的 id 定位（随机 IO）到数据行上读取 order_amount，需要 1000 次随机 IO 才能完成查询，而机械硬盘随机 IO 的效率是极低的（机械硬盘每秒寻址几百次）。

根据我们自己的分析选择全表扫描相对更优。如果把 limit 1000 改成 limit 10，则执行计划会完全不一样。

 

既然我们已经知道是因为随机 IO 导致无法利用索引，那么有没有办法消除随机 IO 呢？

有，覆盖索引。

 

我们来看看利用覆盖索引优化后的索引、执行计划、执行时间。

**1. 创建索引：**

```
ALTER TABLE `t_order`ADD INDEX `idx_ordercode_orderamount` USING BTREE (`order_code` ASC, `order_amount` ASC);
```

创建了复合索引 idx_ordercode_orderamount(order_code,order_amount)，将 select 的列 order_amount 也放到索引中。

**2. 执行计划：**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

执行计划显示查询会利用覆盖索引，并且只扫描了 1000 行数据，查询的性能应该是非常好的。

**3. 执行时间：13ms**

![](http://mmbiz.qpic.cn/mmbiz_png/DmibiaFiaAI4B36XmwteZwQUaB5ACyKK34OuknrshibBXjT6UAt38cwzV1oTAKX4AicRNIxMrZLh1a9PBP8OoLm28jQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

从执行时间来看，SQL 的执行时间提升到原来的 1/20，已经达到我们的预期。

## 总结：

覆盖索引是 select 的数据列只用从索引中就能够取得，不必读取数据行，换句话说查询列要被所建的索引覆盖。索引的字段不只包含查询列，还包含查询条件、排序等。

要写出性能很好的 SQL 不仅需要学习 SQL，还要能看懂数据库执行计划，了解数据库执行过程、索引的数据结构等。

> 来源：Mr 船长
>
> my.oschina.net/loujinhe/blog/1528233


# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=9ytvzpwr)**

我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=9ytvzpwr>

