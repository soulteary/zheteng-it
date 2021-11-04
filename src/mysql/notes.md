# MySQL小技巧

## 批量移除一定相同命名前缀的数据表

不知道哪个有才的脚本，在数据库里加了一堆无意义前缀的表。

删除方式很多，然而最终都是殊途同归，比如读出所有表名称，然后输出删表语句。

这里可以选择直接用 SQL 解决问题：

```sql
SELECT CONCAT('drop table ', table_name, ';') 
FROM information_schema.tables
WHERE table_schema= 'YourDatabase'
AND table_name LIKE 'prefix_%' ;
```

## 手动转换表数据默认字符集

有一些云控制台实现的非常粗糙，创建的时候选项比较单一，选择不到我们所需要的字符集，只能靠创建后手动调整了：

```sql
ALTER DATABASE dbname CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
ALTER TABLE `tbname` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
ALTER TABLE `tbname` DEFAULT CHARACTER SET=utf8mb4, COLLATE=utf8mb4_bin;
```

另外，为了避免转换过程中失败，可以调整 `innodb_large_prefix on` 参数

这里可以借助灵活的 js 来快速解决战斗，批量生成转换语句：

```javascript
tpl = (tblName)=>`ALTER TABLE \`${tblName}\` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;`;

tables = `history
...
...
...
user_relation
usercontent_relation
users`.split('\n').filter(n=>n).map(n=>tpl(n))
console.log(tables.join('\n'))
```