### 索引

索引相关不错的文章：

[腾讯技术工程官方-MySQL索引知识点](https://cloud.tencent.com/developer/article/1761267)

- 浓缩：

  - ``````````````````SQL
    CREATE TABLE T_Mch******Stat (`FStatDate` int unsigned NOT NULL DEFAULT 19700101 COMMENT '统计日期',
    `FMerchantId` bigint unsigned NOT NULL DEFAULT 0 COMMENT '商户ID',
    `FVersion` int unsigned NOT NULL DEFAULT 0 COMMENT '数据版本号',
    `FBatch` bigint unsigned NOT NULL DEFAULT 0 COMMENT '统计批次',
    `FTradeAmount` bigint NOT NULL DEFAULT 0 COMMENT '交易金额'
    PRIMARY KEY (`FStatDate`,`FMerchantId`,`FVersion`),
    INDEX i_FStatDate_FVersion (`FStatDate`,`FVersion`))
    DEFAULT CHARSET = utf8 ENGINE = InnoDB;
    ``````````````````

  - SQL- A：

    - `````````````````````sql
      SELECT SQL_CALC_FOUND_ROWS FStatDate,
          FMerchantId,
          FVersion,
          FBatch,
          FTradeAmount,
          FTradeCount
      FROM T_Mch******Stat_1020
      WHERE FStatDate = 20201020
          AND FVersion = 0
          AND FMerchantId > 0
      ORDER BY FMerchantId ASC LIMIT 0, 8000
      `````````````````````

  - SQL-B :

    - ```````````````````````sql
      SELECT SQL_CALC_FOUND_ROWS a1.FStatDate,
          a1.FMerchantId,
          a1.FVersion,
          FBatch,
          FTradeAmount,
          FTradeCount
      FROM T_Mch******Stat_1020 a1, (
          SELECT FStatDate, FMerchantId, FVersion
          FROM T_Mch******Stat_1020
          WHERE FStatDate = 20201020
              AND FVersion = 0
              AND FMerchantId > 0
              ORDER BY FMerchantId ASC LIMIT 0, 8000 ) a2
      where a1.FStatDate = a2.FStatDate
          and a1.FVersion = a2.FVersion
          and a1.FMerchantId = a2.FMerchantId;
      ```````````````````````

  - SQL B的执行效率高于SQL A，原因是：__

- 

- 相关知识点:

  - 索引下推：

     	first_name字段已设置idx，考虑以下SQL：

- ```````````````SQL
  		select * from employees where hire_date > '1990-01-14' and first_name like 'Hi%';
  ```````````````

  ​			直接取索引的 first_name 值进行过滤，这样不符合"first_name like 'Hi%'"条件的记录就不再需要回表操作。

- ​	索引下推概括来讲就是索引的列在筛选条件中时且包含模糊查询/模糊查询，会被进行筛选后再做后续操作，不需要回表再查询了。

- 

  



### MySQL总体结构

总结部分：

https://juejin.cn/post/6920076107609800711

