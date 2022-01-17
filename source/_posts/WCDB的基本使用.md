---
layout: git
title: WCDB的基本使用
date: 2022-01-09 20:00:00
tags: OC
---
# WCDB的基本使用
WCDB是微信移动端团队开源的移动端数据库组件
https://github.com/Tencent/wcdb

`pod 'WCDB'`


```
@interface ViewController ()
@property (nonatomic, strong)Message *BLMsg;
@end

- (void)viewDidLoad {
    self.BLMsg = [[Message alloc]init];
    NSObject *obj = [[NSObject alloc] init];
    //
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
    //
        @synchronized (obj) {
            NSLog(@"线程同步的操作1 开始");
            sleep(3);
            NSLog(@"线程同步的操作1 结束");
        }
    });
    //
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        sleep(1);
        @synchronized (obj) {
            NSLog(@"线程同步的操作2");
        }
    });
}
```


## BLMessage.h

```

#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface BLMessage : NSObject
@property(nonatomic, copy) NSString *name;
@property(nonatomic, assign) NSInteger localID;
@property(nonatomic, assign) float totalScore;
@property(nonatomic, strong) NSDate *createDate;
- (void)quickTest;
@end

NS_ASSUME_NONNULL_END
```


## BLMessage.m

```
#import "BLMessage+WCTTableCoding.h"
#import "BLMessage.h"
#import <WCDB/WCDB.h>
@interface BLMessage ()
@property (nonatomic, strong) NSString *baseDirectoty;
@property (nonatomic, strong) WCTDatabase *database;

@end
#define BL_TABLE_MESSAGE_NAME @"TableMsgName"
@implementation BLMessage

WCDB_IMPLEMENTATION(BLMessage)
//WCDB_SYNTHESIZE，用于在类文件中定义绑定到数据库表的字段
WCDB_SYNTHESIZE(BLMessage, name)
WCDB_SYNTHESIZE(BLMessage, localID)


//默认使用属性名作为数据库表的字段名。对于属性名与字段名不同的情况，可以使用WCDB_SYNTHESIZE_COLUMN(className, propertyName, columnName)进行映射。
WCDB_SYNTHESIZE_COLUMN(BLMessage, totalScore, "db_totalScore")

WCDB_SYNTHESIZE_DEFAULT(BLMessage, createDate, WCTDefaultTypeCurrentDate) //设置一个默认值

//主键

WCDB_PRIMARY_ASC_AUTO_INCREMENT(BLMessage, localID)
//用于定义非空约束
WCDB_NOT_NULL(BLMessage, name)
- (void)quickTest {
    //获取沙盒根目录
}
    
// 创建数据库和表
- (BOOL)creatDatabaseAndTable {
    //数据库路径
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    self.baseDirectoty = [paths objectAtIndex:0];
    NSString *path = [self.baseDirectoty stringByAppendingPathComponent:@"SampleDB"];
    //NSLog(@"path--> %@",path);
    //创建数据库 路径一样,  该接口使用的是IF NOT EXISTS的SQL，因此可以用重复调用
    WCTDatabase *database = [[WCTDatabase alloc] initWithPath:path];
    _database = database;
    if ([database canOpen]) {
        NSLog(@"创建数据库成功");
    }else{
        NSLog(@"创建数据库失败");
        return NO;
    }

    
    //创建表  注：该接口使用的是IF NOT EXISTS的SQL，因此可以用重复调用。不需要在每次调用前判断表或索引是否已经存在。
    BOOL result = [database createTableAndIndexesOfName:BL_TABLE_MESSAGE_NAME withClass:BLMessage.class];

    if (!result) {
        NSLog(@"创建表失败");
        return NO;
    }
    return YES;
}


// 插入单个数据
- (BOOL)insertData:(BLMessage *)message {
    BOOL result = [_database insertObject:message into:BL_TABLE_MESSAGE_NAME];
    
    //关闭数据库,_database如果能自己释放的话,会自动关闭,就不用手动调用关闭了
    [_database close];

    if (!result) {
        NSLog(@"插入失败");
        return NO;
    }else{
        NSLog(@"插入成功");
        return YES;
    }
}
- (BOOL)insertDatas:(BLMessage *)message {
    //插入多个数据:
    BOOL result = [self.database insertObject:message into:BL_TABLE_MESSAGE_NAME];

    //增删改查用下面方法,可以链式调用

/*
    WCTInsert
     WCTDelete
     WCTUpdate
     WCTSelect
 */
     WCTInsert *insert = [_database prepareInsertObjectsOfClass:BLMessage.class
                                                              into:BL_TABLE_MESSAGE_NAME];
//    BOOL result = [insert executeWithObjects:objects];/
    if (!result) {
        NSLog(@"插入失败");
        return NO;
    }else{
        NSLog(@"插入成功");
        return YES;
    }
}
// 查询数据  用localID排序
- (void)selectOrder {
    NSArray<BLMessage *> *objects2 = [_database getObjectsOfClass:BLMessage.class fromTable:BL_TABLE_MESSAGE_NAME orderBy:BLMessage.localID.order()];
    [objects2 enumerateObjectsUsingBlock:^(BLMessage *obj, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"用localID排序 --> %@ ",obj);
    }];
}

//查询数据  指定范围
- (void)selectCertainRange {
    NSArray<BLMessage *> *objects3 = [_database getObjectsOfClass:BLMessage.class fromTable:BL_TABLE_MESSAGE_NAME where:BLMessage.localID.between(0,1) || BLMessage.name.like(@"lil%")];
    [objects3 enumerateObjectsUsingBlock:^(BLMessage *obj, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"objects3 --> %@ ",obj);
    }];
}

//定向 将查询的totalScore值赋给新创建的对象
- (void)selectAndAssignment {
    BLMessage *message5 = [_database getOneObjectOnResults:BLMessage.totalScore.max().as(BLMessage.totalScore) fromTable:BL_TABLE_MESSAGE_NAME];
    message5.localID = 5;
    NSLog(@"message5 --> %@ ",message5);
}

//链式调用
- (void)selectChain {
    //所有的对象
    WCTSelect *select = [_database prepareSelectObjectsOfClass:BLMessage.class fromTable:BL_TABLE_MESSAGE_NAME ];

    //链式查询
    NSArray<BLMessage *> *objects6 = [[select where:BLMessage.totalScore < 90] limit:2].allObjects;
    [objects6 enumerateObjectsUsingBlock:^(BLMessage *obj, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"objects6 --> %@ ",obj);
    }];
}


//##### 更新
- (void)updateData {
    WCTUpdate *update = [_database prepareUpdateTable:BL_TABLE_MESSAGE_NAME
                                        onProperties:BLMessage.name];
    BLMessage *object = [[BLMessage alloc] init];
    object.name = @"xiaoming22";
    BOOL result = [update executeWithObject:object];
    if (!result) {
        NSLog(@"Update by object Error %@", update.error);
    }else{
        NSLog(@"更新成功");
    }
}


//删除表
- (void)deleteData {
    WCTDelete *deletion = [_database prepareDeleteFromTable:BL_TABLE_MESSAGE_NAME];
    BOOL result = [deletion execute];
    if (!result) {
        NSLog(@"Delete Error %@", deletion.error);
    }else{
        NSLog(@"删除成功");
    }

    [_database close];
    //删除name是xiaoming的人
//    BOOL result = [_database deleteObjectsFromTable:BL_TABLE_MESSAGE_NAME where:BLMessage.name == @"xiaoming"];
//    [_database deleteObjectsFromTable:BL_TABLE_MESSAGE_NAME where:BLMessage.localID.between(0,1) || BLMessage.name.like(@"lil%")];
    
}

@end

```