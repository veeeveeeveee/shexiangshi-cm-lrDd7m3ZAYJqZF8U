
## 一、问题


本人使用的MySql版本是8\.0的


当MySql5\.7及以上的版本执行带有 ORDER BY 的SQL语句时可能会报错。


例如，执行以下mysql语句：




```
SELECT id, user_id, title FROM m_article WHERE user_id>=100 AND user_id <=200 GROUP BY user_id;
```


SQL报错信息如下：




```
1055 - Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.m_article.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```


 


## 二、分析原因


SQL\-92及更早版本的查询不允许使用select列表、HAVING条件或ORDER BY列表引用未在GROUP BY子句中命名的非聚合列。


简单来说：由于sql\-mode的参数配置了ONLY\_FULL\_GROUP\_BY，这时 select 的字段不在 group by 中，并且 select 的字段未使用聚合函数（SUM,MAX,MIN等）的话，那么这条SQL查询是被 MySql 认为非法。


MySql官方文档：[https://dev.mysql.com/doc/refman/8\.0/en/group\-by\-handling.html](https://github.com)


 


## 三、解决方法


### 1、不修改 sql\-mode 的参数情况下


1\.1、以本文中的例句修改，在原来的 ORDER BY 后面多加一个主键。


![](https://img2024.cnblogs.com/blog/786166/202410/786166-20241025151229381-272514634.png)


 


1\.2、以本文中的例句修改，使用ANY\_VALUE()函数，把非 GROUP BY 列中的字段和没有使用聚合函数的都加上。使用ANY\_VALUE()不检查函数结果是否为ONLY\_FULL\_GROUP\_BY SQL模式。


MySql官方文档：[https://dev.mysql.com/doc/refman/8\.0/en/miscellaneous\-functions.html\#function\_any\-value](https://github.com):[蓝猫机场加速器](https://dahelaoshi.com)


![](https://img2024.cnblogs.com/blog/786166/202410/786166-20241025152008359-1854847147.png)


![](https://img2024.cnblogs.com/blog/786166/202410/786166-20241025152734043-734624889.png)


 


### 2、修改 sql\-mode 的参数


2\.2、使用SQL语句临时修改。




```
SET sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
```


![](https://img2024.cnblogs.com/blog/786166/202410/786166-20241025154906765-275771602.png)


 


2\.3、修改 MySql 的配置文件，这种方式配置完成后都要重启MySql。


2\.3\.1、Linux 中找到 MySql 配置文件，文件名一般叫【my.cnf】，文件路径一般在：/etc/my.cnf，/etc/mysql/my.cnf。打开【my.cnf】文件后，就在  \[mysqld] 下面追加一行即可。




```
sql-mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```


![](https://img2024.cnblogs.com/blog/786166/202410/786166-20241025154319567-1634628056.png)


 


2\.3\.2、Windows 中找到【my.ini】文件，打开后在  \[mysqld] 下面追加一行即可。




```
sql-mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```


![](https://img2024.cnblogs.com/blog/786166/202410/786166-20241025154444056-683106983.png)


 


