---
layout: git
title: iOS 模糊的几种方法（待补充）
date: 2022-08-25 20:00:00
tags: OC
---

# iOS 模糊的几种方法（待补充）

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    [self.view addSubview:self.blurredImageLeft];
    self.blurredImageLeft.image = [self blurryImage:[UIImage imageNamed:@"10000"] withBlurLevel:10];
    
    [self.view addSubview:self.blurredImageRight];
    [self addBlurreView];
}


- (UIImage *)blurryImage:(UIImage *)image
           withBlurLevel:(CGFloat)blur {
    //CIImage
    CIImage *ciImage = [[CIImage alloc]initWithImage:image];

    //CIFilter
    CIFilter *blurFilter = [CIFilter filterWithName:@"CIGaussianBlur"];

    //将图片输入到滤镜中
    [blurFilter setValue:ciImage forKey:kCIInputImageKey];

    //设置的模糊程度
        [blurFilter setValue:@(blur) forKey:@"inputRadius"];
    
    //将处理好的图片输出
    CIImage *outCiImage = [blurFilter valueForKey:kCIOutputImageKey];

    NSLog(@"%@",[blurFilter attributes]);

    //CIContext
    CIContext *context = [CIContext contextWithOptions:nil];

    //获取CGImage句柄
    CGImageRef outCGImage = [context createCGImage:outCiImage fromRect:
                           [[CIImage imageWithCGImage:image.CGImage] extent]];

    //最终获取到图片
    UIImage *blurImage = [UIImage imageWithCGImage:outCGImage];

    //释放CGImage句柄
    CGImageRelease(outCGImage);

    /*.............. */

    //初始化UIImageView
    return blurImage;
}

- (void)addBlurreView {
    UIBlurEffect *effect = [UIBlurEffect effectWithStyle:UIBlurEffectStyleLight];
    UIVisualEffectView *effectView = [[UIVisualEffectView alloc] initWithEffect:effect];
    effectView.frame = self.blurredImageRight.bounds;
    [self.blurredImageRight addSubview:effectView];
    
    UIVibrancyEffect *vibrancyView = [UIVibrancyEffect effectForBlurEffect:effect];
    UIVisualEffectView *visualEffectView = [[UIVisualEffectView alloc] initWithEffect:vibrancyView];
    visualEffectView.translatesAutoresizingMaskIntoConstraints = NO;
    [effectView.contentView addSubview:visualEffectView];
}

- (UIImageView *)blurredImageLeft {
    if (_blurredImageLeft == nil) {
        _blurredImageLeft = [[UIImageView alloc]initWithFrame:CGRectMake(0, 0, self.view.frame.size.width/2, self.view.frame.size.height)];
        _blurredImageLeft.contentMode = UIViewContentModeScaleAspectFill;
        _blurredImageLeft.clipsToBounds = YES;
    }
    return _blurredImageLeft;
}

- (UIImageView *)blurredImageRight {
    if (!_blurredImageRight) {
        _blurredImageRight = [[UIImageView alloc] initWithFrame:CGRectMake(self.view.frame.size.width/2,0,self.view.frame.size.width/2, self.view.frame.size.height)];
        _blurredImageRight.image = [UIImage imageNamed:@"10000"];
        _blurredImageRight.contentMode = UIViewContentModeScaleAspectFill;
        _blurredImageRight.clipsToBounds = YES;

    }
    return _blurredImageRight;
}

```