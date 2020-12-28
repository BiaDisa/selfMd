#### 错误起因

相同代码（将null值插入createTime列中），在移植到另一个环境时发生了

> create_time cannot be null

的error，无法往下运作。在代码中临时将createTime设置为

>new Date()

使之能够正常运行。



#### 冲突点

在本地/测试/线上环境没有问题，为什么到了移植环境，**相同**的代码会有不同的结果？



#### 解决

看到了这篇文章：

https://blog.csdn.net/m0_38016951/article/details/103601485

其实博主的解释是错误的，注释掉的部分里

> explicit_defaults_for_timestamp=true

这段代码才是问题的发生关键。

执行代码

> show VARIABLES like "explicit_defaults_for_timestamp"

发现本地和移植环境的结果不同：

- 本地为off
- 移植环境为on

根据 oracle官方在 [dev文档](https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html)中的解释如下：

>[`explicit_defaults_for_timestamp`](https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_explicit_defaults_for_timestamp)
>
>
>
>| Command-Line Format | `--explicit-defaults-for-timestamp[={OFF|ON}]` |
>| :------------------ | ---------------------------------------------- |
>| Deprecated          | Yes                                            |
>| System Variable     | `explicit_defaults_for_timestamp`              |
>| Scope               | Global, Session                                |
>| Dynamic             | Yes                                            |
>| Type                | Boolean                                        |
>| Default Value       | `OFF`                                          |
>
>This system variable determines whether the server enables certain nonstandard behaviors for default values and `NULL`-value handling in [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) columns. By default, [`explicit_defaults_for_timestamp`](https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_explicit_defaults_for_timestamp) is disabled, which enables the nonstandard behaviors.
>
>If [`explicit_defaults_for_timestamp`](https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_explicit_defaults_for_timestamp) is disabled, the server enables the nonstandard behaviors and handles [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) columns as follows:
>
>- [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) columns not explicitly declared with the `NULL` attribute are automatically declared with the `NOT NULL` attribute. Assigning such a column a value of `NULL` is permitted and sets the column to the current timestamp.
>
>- The first [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) column in a table, if not explicitly declared with the `NULL` attribute or an explicit `DEFAULT` or `ON UPDATE` attribute, is automatically declared with the `DEFAULT CURRENT_TIMESTAMP` and `ON UPDATE CURRENT_TIMESTAMP` attributes.
>
>- [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) columns following the first one, if not explicitly declared with the `NULL` attribute or an explicit `DEFAULT` attribute, are automatically declared as `DEFAULT '0000-00-00 00:00:00'` (the “zero” timestamp). For inserted rows that specify no explicit value for such a column, the column is assigned `'0000-00-00 00:00:00'` and no warning occurs.
>
>  Depending on whether strict SQL mode or the [`NO_ZERO_DATE`](https://dev.mysql.com/doc/refman/5.6/en/sql-mode.html#sqlmode_no_zero_date) SQL mode is enabled, a default value of `'0000-00-00 00:00:00'` may be invalid. Be aware that the [`TRADITIONAL`](https://dev.mysql.com/doc/refman/5.6/en/sql-mode.html#sqlmode_traditional) SQL mode includes strict mode and [`NO_ZERO_DATE`](https://dev.mysql.com/doc/refman/5.6/en/sql-mode.html#sqlmode_no_zero_date). See [Section 5.1.10, “Server SQL Modes”](https://dev.mysql.com/doc/refman/5.6/en/sql-mode.html).
>
>In MySQL 5.6, the nonstandard behaviors just described are deprecated; expect them to be removed in a future MySQL release.
>
>If [`explicit_defaults_for_timestamp`](https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_explicit_defaults_for_timestamp) is enabled, the server disables the nonstandard behaviors and handles [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) columns as follows:
>
>- It is not possible to assign a [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) column a value of `NULL` to set it to the current timestamp. To assign the current timestamp, set the column to [`CURRENT_TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/date-and-time-functions.html#function_current-timestamp) or a synonym such as [`NOW()`](https://dev.mysql.com/doc/refman/5.6/en/date-and-time-functions.html#function_now).
>- [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) columns not explicitly declared with the `NOT NULL` attribute are automatically declared with the `NULL` attribute and permit `NULL` values. Assigning such a column a value of `NULL` sets it to `NULL`, not the current timestamp.
>- [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) columns declared with the `NOT NULL` attribute do not permit `NULL` values. For inserts that specify `NULL` for such a column, the result is either an error for a single-row insert or if strict SQL mode is enabled, or `'0000-00-00 00:00:00'` is inserted for multiple-row inserts with strict SQL mode disabled. In no case does assigning the column a value of `NULL` set it to the current timestamp.
>- [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) columns explicitly declared with the `NOT NULL` attribute and without an explicit `DEFAULT` attribute are treated as having no default value. For inserted rows that specify no explicit value for such a column, the result depends on the SQL mode. If strict SQL mode is enabled, an error occurs. If strict SQL mode is not enabled, the column is declared with the implicit default of `'0000-00-00 00:00:00'` and a warning occurs. This is similar to how MySQL treats other temporal types such as [`DATETIME`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html).
>- No [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) column is automatically declared with the `DEFAULT CURRENT_TIMESTAMP` or `ON UPDATE CURRENT_TIMESTAMP` attributes. Those attributes must be explicitly specified.
>- The first [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) column in a table is not handled differently from [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.6/en/datetime.html) columns following the first one.
>
>



可以知道：

- 在这个参数为off的情况下，可以将null值插入到not null 的timeStamp中，并最终赋值为对应列的默认值