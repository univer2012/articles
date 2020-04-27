---
title: 33FMDB的使用方法
date: 2017-12-05 18:01:30
tags:
---
来自：[FMDB的使用方法（附Demo)](http://www.jianshu.com/p/54e74ce87404)

最近在项目中需要在多个页面对同样的数据进行相关操作，于是便用到了FMDB数据库操作，以下便是FMDB的一些简单的使用方法。附Demo一份:[FMDBDemo](https://github.com/MisterBooo/FMDBDemo)


## 1.为了更好的的进行管理，先创建了FMDB的单例
```Objective-C
#import "DataBase.h"

#import <FMDB.h>

#import "Person.h"
#import "Car.h"
static DataBase *_DBCtl = nil;

@interface DataBase()<NSCopying,NSMutableCopying>{
    FMDatabase  *_db;

}
@end

@implementation DataBase
+(instancetype)sharedDataBase{
    if (_DBCtl == nil) {
        _DBCtl = [[DataBase alloc] init];
        [_DBCtl initDataBase];    
    }
    return _DBCtl;
}

+(instancetype)allocWithZone:(struct _NSZone *)zone{
    if (_DBCtl == nil) {
        _DBCtl = [super allocWithZone:zone];    
    }
    return _DBCtl;    
}
-(id)copy{
    return self;
}

-(id)mutableCopy{
    return self;    
}
-(id)copyWithZone:(NSZone *)zone{
    return self;
}

-(id)mutableCopyWithZone:(NSZone *)zone{
    return self;    
}
-(void)initDataBase{
    // 获得Documents目录路径

    NSString *documentsPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];

    // 文件路径

    NSString *filePath = [documentsPath stringByAppendingPathComponent:@"model.sqlite"];

    // 实例化FMDataBase对象

    _db = [FMDatabase databaseWithPath:filePath];

    [_db open];

    // 初始化数据表
    NSString *personSql = @"CREATE TABLE 'person' ('id' INTEGER PRIMARY KEY AUTOINCREMENT  NOT NULL ,'person_id' VARCHAR(255),'person_name' VARCHAR(255),'person_age' VARCHAR(255),'person_number'VARCHAR(255)) ";
    NSString *carSql = @"CREATE TABLE 'car' ('id' INTEGER PRIMARY KEY AUTOINCREMENT  NOT NULL ,'own_id' VARCHAR(255),'car_id' VARCHAR(255),'car_brand' VARCHAR(255),'car_price'VARCHAR(255)) ";

    [_db executeUpdate:personSql];
    [_db executeUpdate:carSql];
    [_db close];
}
@end
```

现在创建好了数据库，可以保存person对象与car对象的相关属性
数据库中创建了两张表person表与car表，分别管理person的数据与car的数据，通过person_id 与own_id进行关联。

## 2.提供接口
```Objective-C
#import <Foundation/Foundation.h>
@class Person;
@class Car;

@interface DataBase : NSObject
@property(nonatomic,strong) Person *person;
+ (instancetype)sharedDataBase;
#pragma mark - Person
/**
 *  添加person
 */
- (void)addPerson:(Person *)person;
/**
 *  删除person
 */
- (void)deletePerson:(Person *)person;
/**
 *  更新person
 */
- (void)updatePerson:(Person *)person;

/**
 *  获取所有数据
 */
- (NSMutableArray *)getAllPerson;

#pragma mark - Car
/**
 *  给person添加车辆
 */
- (void)addCar:(Car *)car toPerson:(Person *)person;
/**
 *  给person删除车辆
 */
- (void)deleteCar:(Car *)car fromPerson:(Person *)person;
/**
 *  获取person的所有车辆
 *
 */
- (NSMutableArray *)getAllCarsFromPerson:(Person *)person;
/**
 *  删除person的所有车辆
 *
 */
- (void)deleteAllCarsFromPerson:(Person *)person;
@end
```

## 3.接口的实现
```Objective-C
#pragma mark - 接口

- (void)addPerson:(Person *)person{
    [_db open];
    NSNumber *maxID = @(0);

    FMResultSet *res = [_db executeQuery:@"SELECT * FROM person "];
    //获取数据库中最大的ID
    while ([res next]) {
        if ([maxID integerValue] < [[res stringForColumn:@"person_id"] integerValue]) {
            maxID = @([[res stringForColumn:@"person_id"] integerValue] ) ;
        }  
    }
    maxID = @([maxID integerValue] + 1);
    [_db executeUpdate:@"INSERT INTO person(person_id,person_name,person_age,person_number)VALUES(?,?,?,?)",maxID,person.name,@(person.age),@(person.number)];
    [_db close];  
}

- (void)deletePerson:(Person *)person{
    [_db open];
    [_db executeUpdate:@"DELETE FROM person WHERE person_id = ?",person.ID];
    [_db close];
}

- (void)updatePerson:(Person *)person{
    [_db open];

    [_db executeUpdate:@"UPDATE 'person' SET person_name = ?  WHERE person_id = ? ",person.name,person.ID];
    [_db executeUpdate:@"UPDATE 'person' SET person_age = ?  WHERE person_id = ? ",@(person.age),person.ID];
    [_db executeUpdate:@"UPDATE 'person' SET person_number = ?  WHERE person_id = ? ",@(person.number + 1),person.ID];
    [_db close];
}

- (NSMutableArray *)getAllPerson{
    [_db open];
    NSMutableArray *dataArray = [[NSMutableArray alloc] init];

    FMResultSet *res = [_db executeQuery:@"SELECT * FROM person"];

    while ([res next]) {
        Person *person = [[Person alloc] init];
        person.ID = @([[res stringForColumn:@"person_id"] integerValue]);
        person.name = [res stringForColumn:@"person_name"];
        person.age = [[res stringForColumn:@"person_age"] integerValue];
        person.number = [[res stringForColumn:@"person_number"] integerValue];

        [dataArray addObject:person];
    }
    [_db close];
    return dataArray;
}
/**
 *  给person添加车辆
 */
- (void)addCar:(Car *)car toPerson:(Person *)person{
    [_db open];
    //根据person是否拥有car来添加car_id
    NSNumber *maxID = @(0);

    FMResultSet *res = [_db executeQuery:[NSString stringWithFormat:@"SELECT * FROM car where own_id = %@ ",person.ID]];

    while ([res next]) {
        if ([maxID integerValue] < [[res stringForColumn:@"car_id"] integerValue]) {
             maxID = @([[res stringForColumn:@"car_id"] integerValue]);
        }
    }
     maxID = @([maxID integerValue] + 1);
    [_db executeUpdate:@"INSERT INTO car(own_id,car_id,car_brand,car_price)VALUES(?,?,?,?)",person.ID,maxID,car.brand,@(car.price)];
    [_db close];
}
/**
 *  给person删除车辆
 */
- (void)deleteCar:(Car *)car fromPerson:(Person *)person{
    [_db open];
    [_db executeUpdate:@"DELETE FROM car WHERE own_id = ?  and car_id = ? ",person.ID,car.car_id];
    [_db close];
}
/**
 *  获取person的所有车辆
 */
- (NSMutableArray *)getAllCarsFromPerson:(Person *)person{

    [_db open];
    NSMutableArray  *carArray = [[NSMutableArray alloc] init];

    FMResultSet *res = [_db executeQuery:[NSString stringWithFormat:@"SELECT * FROM car where own_id = %@",person.ID]];
    while ([res next]) {
        Car *car = [[Car alloc] init];
        car.own_id = person.ID;
        car.car_id = @([[res stringForColumn:@"car_id"] integerValue]);
        car.brand = [res stringForColumn:@"car_brand"];
        car.price = [[res stringForColumn:@"car_price"] integerValue];

        [carArray addObject:car];
    }
    [_db close];
    return carArray;
}
- (void)deleteAllCarsFromPerson:(Person *)person{
    [_db open];
    [_db executeUpdate:@"DELETE FROM car WHERE own_id = ?",person.ID];
    [_db close];
}
```
提供了公共接口之后，在任何一个页面都能进行数据的操作

# 4 使用
## 4.1 添加数据

只要在右上角的点击事件中填写如下代码，可以很快的在数据库中添加person数据
```Objective-C
/**
 *  添加数据到数据库
 */
- (void)addData{
    NSLog(@"addData");
    int nameRandom = arc4random_uniform(1000);
    NSInteger ageRandom  = arc4random_uniform(100) + 1;
    NSString *name = [NSString stringWithFormat:@"person_%d号",nameRandom];
    NSInteger age = ageRandom;

    Person *person = [[Person alloc] init];
    person.name = name;
    person.age = age;

    [[DataBase sharedDataBase] addPerson:person];

    self.dataArray = [[DataBase sharedDataBase] getAllPerson];
    [self.tableView reloadData];
}
```

## 4.2 删除数据
删除某个person的某辆car
```Objective-C
/**
 *  设置删除按钮
 *
 */
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath{

    if (editingStyle == UITableViewCellEditingStyleDelete){

        Car *car = self.carArray[indexPath.row];

        NSLog(@"car.id--%@,own_id--%@",car.car_id,car.own_id);
        [[DataBase sharedDataBase] deleteCar:car fromPerson:self.person];
        self.carArray = [[DataBase sharedDataBase] getAllCarsFromPerson:self.person];

        [self.tableView reloadData];
    }
}
```

## 4.3 修改数据
修改person的name 与 age
```Objective-C
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{

//    /****************跳转页面 查看car***************/
//    PersonCarsViewController *pcvc = [[PersonCarsViewController alloc] init];
//    pcvc.person = self.dataArray[indexPath.row];
//    
//    [self.navigationController pushViewController:pcvc animated:YES];
//    

   /****************person的更新操作***************/

    Person *person = self.dataArray[indexPath.row];
    person.name = [NSString stringWithFormat:@"%@",person.name];

    person.age = arc4random_uniform(100) + 1;
    [[DataBase sharedDataBase] updatePerson:person];

    self.dataArray = [[DataBase sharedDataBase] getAllPerson];

    [self.tableView reloadData];
}
```

## 4.4 查看数据
```Objective-C
self.dataArray = [[DataBase sharedDataBase] getAllPerson];
    for (int i = 0 ; i < self.dataArray.count; i++) {
        Person *person = self.dataArray[i];
        NSMutableArray *carArray =  [[DataBase sharedDataBase] getAllCarsFromPerson:person];
        [self.carArray addObject:carArray];
    }
```