## SQL-实现CreateOrUpdate

#### INSERT ... ON DUPLICATE KEY UPDATE语句

如果指定了ON DUPLICATE KEY UPDATE，并且插入行后会导致在一个**UNIQUE索引或PRIMARY KEY**中出现重复值，则执行旧行UPDATE。

```sql
insert into test values(null, 'unique', '654321') on duplicate key update col1 = 'update_unique';
```

通过这样的语句可以实现CreateOrUpdate的功能。节省了一条SQL语句。

#### REPLACE语句

　　我们在使用数据库时可能会经常遇到这种情况。如果一个表在一个字段上建立了唯一索引，当我们再向这个表中使用已经存在的键值插入一条记录，那将会抛出一个主键冲突的错误。当然，我们可能想用新记录的值来覆盖原来的记录值。如果使用传统的做法，必须先使用DELETE语句删除原先的记录，然后再使用INSERT插入新的记录。而在MySQL中为我们提供了一种新的解决方案，这就是REPLACE语句。使用REPLACE插入一条记录时，如果不重复，REPLACE就和INSERT的功能一样，如果有重复记录，REPLACE就使用新记录的值来替换原来的记录值。

　　使用REPLACE的最大好处就是可以将DELETE和INSERT合二为一，形成一个原子操作。这样就可以不必考虑在同时使用DELETE和INSERT时添加事务等复杂操作了。

　　在使用REPLACE时，表中必须有唯一索引，而且这个索引所在的字段不能允许空值，否则REPLACE就和INSERT完全一样的。　

 　  ** 在执行REPLACE后，系统返回了所影响的行数，如果返回1，说明在表中并没有重复的记录，如果返回2，说明有一条重复记录，系统自动先调用了DELETE删除这条记录，然后再记录用INSERT来插入这条记录。如果返回的值大于2，那说明有多个唯一索引，有多条记录被删除和插入。**

```sql
　　REPLACE INTO users (id,name,age) VALUES(123, '赵本山', 50);

```

  
  


