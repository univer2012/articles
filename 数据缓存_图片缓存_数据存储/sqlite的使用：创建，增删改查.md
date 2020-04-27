1. [iOS开发数据库篇—SQLite的应用](https://www.cnblogs.com/wendingding/p/3870289.html)
2. [iOS SQLite详解](https://www.cnblogs.com/guohai-stronger/p/9218175.html)
3. [iOS开发数据库篇—FMDB简单介绍 ](https://www.cnblogs.com/wendingding/p/3871848.html)
4. [iOS CoreData (一) 增删改查](https://www.jianshu.com/p/332cba029b95)
5. 

# 2、FMDB的操作

## 四、数据库操作

### 1、执行更新操作

#### 1、CREATE (建表操作)
```
NSString *createTableSqlString = @"CREATE TABLE IF NOT EXISTS t_student (id integer PRIMARY KEY AUTOINCREMENT, name text NOT NULL, age integer NOT NULL)";
[db executeUpdate:createTableSqlString];
```

#### 2、INSERT（写入数据）
```
//不确定的参数用？来占位
NSString *sql = @"insert into t_student (name, age) values (?, ?)";
[db executeUpdate:sql, @"zhangsan", [NSNumber numberWithInt:18]];
```

#### 3、示例DELETE（删除数据）
```
NSString *sql = @"delete from t_student where id = ?";
[db executeUpdate:sql, [NSNumber numberWithInt:1]];
```

#### 4、UPDATE（更改数据）
1、`- (BOOL)executeUpdate:(NSString*)sql, ...`
```
NSString *sql = @"update t_student set name = "heiheihei"  where id = ?";
[db executeUpdate:sql, [NSNumber numberWithInt:1]];
```
2、`- (BOOL)executeUpdateWithFormat:(NSString*)format, ...`
```
//不确定的参数用%@，%d等来占位
NSString *sql = @"insert into t_student (name,age) values (%@,%i)";
[db executeUpdateWithFormat:sql, @"zhangsan", 18];
```

3、`- (BOOL)executeUpdate:(NSString*)sql withParameterDictionary:(NSDictionary *)arguments`
```
NSDictionary *studentDict = [NSDictionary dictionaryWithObjectsAndKeys:@"lisi", @"name", @"18", @"age", nil];
[db executeUpdate:@"insert into t_student (name, age) values (:name, :age)" withParameterDictionary:studentDict];  
```


### 2、执行查询操作

查询方法

1、 `- (FMResultSet *)executeQuery:(NSString*)sql, ...`

2、 `- (FMResultSet *)executeQueryWithFormat:(NSString*)format, ...`

实例：
```
// 4.查询
NSString *sql = @"select id, name, age FROM t_student";
FMResultSet *rs = [db executeQuery:sql];
while ([rs next]) {
    int id = [rs intForColumnIndex:0];
    NSString *name = [rs stringForColumnIndex:1];
    int age = [rs intForColumnIndex:2];
    NSDictionary *studentDict = [NSDictionary dictionaryWithObjectsAndKeys:[NSNumber numberWithInt:id], @"id", name, @"name", [NSNumber numberWithInt:age], @"age", nil];
    [studentArray addObject:studentDict];
}
```

### 3、多语句和批处理
通过 `-executeStatements:withResultBlock:` 方法在一个字符串中执行多语句。
```
NSString *sql = @"CREATE TABLE IF NOT EXISTS bulktest1 (id integer PRIMARY KEY AUTOINCREMENT, x text);"
                "CREATE TABLE IF NOT EXISTS bulktest2 (id integer PRIMARY KEY AUTOINCREMENT, y text);"
                "CREATE table IF NOT EXISTS bulktest3 (id integer primary key autoincrement, z text);"
                "insert into bulktest1 (x) values ('XXX');"
                "insert into bulktest2 (y) values ('YYY');"
                "insert into bulktest3 (z) values ('ZZZ');"
        ;
        result = [db executeStatements:sql];
        
sql = @"select count(*) as count from bulktest1;"
        "select count(*) as count from bulktest2;"
        "select count(*) as count from bulktest3;";
        
        result = [db executeStatements:sql withResultBlock:^int(NSDictionary *resultsDictionary) {
            NSLog(@"dictionary=%@", resultsDictionary);
            return 0;
        }];
```


## 五、队列和线程安全

不要实例化一个 FMDatabase 单例来跨线程使用。

相反，使用 `FMDatabaseQueue`，下面就是它的使用方法：

1. 创建队列
```
FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:aPath];
```

2. 示例：
```
[queue inDatabase:^(FMDatabase *db) {
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:1]];
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:2]];
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:3]];

    FMResultSet *rs = [db executeQuery:@"select * from foo"];
    while ([rs next]) {
        ...
    }
}];
```

3. 把操作放在事务中也很简单，示例：
```
[queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:1]];
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:2]];
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:3]];

    if (whoopsSomethingWrongHappened) {
        *rollback = YES;
        return;
    }
    // ...
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:4]];
}];
```