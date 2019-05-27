## SQL-CreateOrUpdate

如果指定了ON DUPLICATE KEY UPDATE，并且插入行后会导致在一个**UNIQUE索引或PRIMARY KEY**中出现重复值，则执行旧行UPDATE。

```sql
insert into test values(null, 'unique', '654321') on duplicate key update col1 = 'update_unique';
```

通过这样的语句可以实现CreateOrUpdate的功能。节省了一条SQL语句。

