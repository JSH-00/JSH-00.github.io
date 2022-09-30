---
layout: git
title: CoreMotion 陀螺仪
date: 2022-07-02 20:00:00
tags: OC
---

# CoreMotion 陀螺仪

```
#import <CoreMotion/CoreMotion.h>
@property (nonatomic, strong)CMMotionManager * manager;

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    self.view.backgroundColor = UIColor.blackColor;
    CMMotionManager *manager = [[CMMotionManager alloc] init];
    self.manager = manager;

    UIView *view2 = [UIView new];
    view2.tag = 2;
    view2.frame = CGRectMake(100, 200, 200, 200);
    view2.backgroundColor = [UIColor systemPinkColor];
    view2.layer.borderWidth = 2.0;
    view2.layer.borderColor = [UIColor blueColor].CGColor;
    [self.view addSubview:view2];
    
    UIView *view3 = [UIView new];
    view3.tag = 2;
    view3.frame = CGRectMake(100, 500, 80, 80);
    view3.backgroundColor = [UIColor yellowColor];
    [self.view addSubview:view3];
    
    [manager startDeviceMotionUpdatesUsingReferenceFrame:CMAttitudeReferenceFrameXArbitraryZVertical toQueue:[NSOperationQueue new] withHandler:^(CMDeviceMotion * _Nullable motion, NSError * _Nullable error) {
        CGFloat angle = motion.attitude.yaw;
        CGFloat rollAngle = motion.attitude.roll;
        CGFloat pitchAngle = motion.attitude.pitch;
        CGFloat gx = motion.gravity.x;
        CGFloat gy = motion.gravity.y;

        if (fabs(gx - 1) < 0.3 && fabs(gy) < 0.3) {
                NSLog(@"横屏1");
        } else if (fabs(gx + 1) < 0.3 && fabs(gy) < 0.3) {
                NSLog(@"横屏2");
        }
        
        if ((fabs(gx) < 0.3 && fabs(gy + 1) < 0.3)|| (fabs(gx) < 0.3 && fabs(gy - 1) < 0.3)) {
                NSLog(@"竖屏的两种");
        }
                
        dispatch_async(dispatch_get_main_queue(), ^{
            CATransform3D c1 = CATransform3DMakeRotation(angle, 0, 0, 1);
            CATransform3D c2 = CATransform3DMakeRotation(rollAngle, 0, 1, 0);
            CATransform3D c3 = CATransform3DMakeRotation(pitchAngle, 1, 0, 0);
            view2.layer.transform = CATransform3DConcat(c3, CATransform3DConcat(c1, c2));
            view3.layer.transform = CATransform3DConcat(c1, c2);
        });
    }];
}
```