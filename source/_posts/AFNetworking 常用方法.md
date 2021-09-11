---
layout: git
title: AFNetworking 常用方法
date: 2021-09-11 20:00:00
tags: OC
---
# AFNetworking 常用方法
## GET 方法获取并显示图片
```
UIImageView *imageNetView = [UIImageView new];
[self.view addSubview: imageNetView];
self.ImageView = imageNetView;
imageNetView.frame = CGRectMake(200, 200, 200, 200);

AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
/* 知识点1：设置AFN采用什么样的方式来解析服务器返回的数据*/

 //如果返回的是XML，那么告诉AFN，响应的时候使用XML的方式解析
// manager.responseSerializer = [AFXMLParserResponseSerializer serializer];

 //如果返回的就是二进制数据，那么采用默认二进制的方式来解析数据（HTTP）
 manager.responseSerializer = [AFHTTPResponseSerializer serializer];

 //采用JSON的方式来解析数据
 //manager.responseSerializer = [AFJSONResponseSerializer serializer];
 
  //指定接收信号为image/png
 manager.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"image/png"];
 
 [manager GET:@"https://search-operate.cdn.bcebos.com/9dfdb7a4fa9dab231f5dd9b90dc91597.png" parameters:nil headers:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
    
    NSLog(@"%@---%@",[responseObject class],responseObject);
    self.ImageView.image = [UIImage imageWithData:responseObject]; //NSData转UIimage

} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
    NSLog(@"请求失败--%@",error);
}];

```
## 通过 URL 下载文件
```
@property (nonatomic, strong) NSURLSessionDownloadTask* downloadTask;

- (void)downloadFromURL:(NSString *)downloadURL
{
    /* 创建网络下载对象 */
    AFURLSessionManager *manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    /* 下载地址 */
    NSURL *url = [NSURL URLWithString:downloadURL];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    /* 下载路径 */
    NSString *path = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents/Announcement"]; // 指定下载到沙盒下新建的Announcement文件夹中
    NSFileManager *fileManager = [NSFileManager defaultManager];
    // 创建文件夹
    BOOL isDir = NO;
    BOOL existed = [fileManager fileExistsAtPath:path isDirectory:&isDir]; // fileExistsAtPath 判断一个文件或目录是否有效，isDirectory判断是否一个目录
    if (!(isDir && existed)) {
        // 在Document目录下创建一个Announcement目录
        [fileManager createDirectoryAtPath:path withIntermediateDirectories:YES attributes:nil error:nil];
    }
    NSString *filePath = [path stringByAppendingPathComponent:url.lastPathComponent];

    /* 开始请求下载 */
    self.downloadTask = [manager downloadTaskWithRequest:request progress:^(NSProgress * _Nonnull downloadProgress) {
        NSLog(@"下载进度：%.0f％，线程：%@", downloadProgress.fractionCompleted * 100, [NSThread currentThread]);
        dispatch_async(dispatch_get_main_queue(), ^{
            //进行UI操作（刷新进度条），需要切换到主线
        });

    } destination:^NSURL * _Nonnull(NSURL * _Nonnull targetPath, NSURLResponse * _Nonnull response) {
        /* 设定下载到的位置 */
        return [NSURL fileURLWithPath:filePath];

    } completionHandler:^(NSURLResponse * _Nonnull response, NSURL * _Nullable filePath, NSError * _Nullable error) {
        [[[UIAlertView alloc] initWithTitle:@"下载完成" message:self.downloadTask.response.suggestedFilename delegate:self cancelButtonTitle:@"知道了" otherButtonTitles: nil] show]; // 弹出下载完成弹窗提示
    }];
    [self.downloadTask resume];
}

```
### POST 登录用户名密码
#### ViewController.m
```
- (void)post {
    //1.创建会话管理者
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.responseSerializer = [AFJSONResponseSerializer serializer];
    manager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"application/json",@"text/html",@"text/javascript",@"text/json",@"text/plain", nil];
    
    //2.创建参数
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    [dict setObject:@"phoneAreaCode" forKey:@"86"];
    [dict setObject:@"phoneNumber" forKey:@"13766666666"];
    [dict setObject:@"password" forKey:@"12345678"];
    //3.发送POST请求
    [manager POST:@"http://cn.test.api.gethover.com/api/auth/login/phone"
       parameters:dict headers:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"请求成功%@---%@",[responseObject class], responseObject);

    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        NSLog(@"请求失败---%@",error);
    }];
}
```
#### 解决POST secure connection 问题
报错信息:
```
Error Domain=NSURLErrorDomain Code=-1022 "The resource could not be loaded because the App Transport Security policy requires the use of a secure connection." UserInfo={NSLocalizedDescription=The resource could not be loaded because the App Transport Security policy requires the use of a secure connection...
```
解决方法:
Info.plist
Information Property List -> App Transport Security Settings ->Allow Arbitrary Loads:YES