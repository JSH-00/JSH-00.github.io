---
layout: git
title: CAAnimation 帧动画
date: 2022-06-25 20:00:00
tags: OC
---
# CAAnimation 帧动画

1. CABasicAnimation
通过设定起始点，终点，时间，动画会沿着你这设定点进行移动。可以看做特殊的CAKeyFrameAnimation
2. CAKeyframeAnimation
Keyframe顾名思义就是关键点的frame，你可以通过设定CALayer的始点、中间关键点、终点的frame，时间，动画会沿你设定的轨迹进行移动
3. CAAnimationGroup
Group也就是组合的意思，就是把对这个Layer的所有动画都组合起来。PS：一个layer设定了很多动画，他们都会同时执行，如何按顺序执行我到时候再讲。
4. CATransition
这个就是苹果帮开发者封装好的一些动画。

```
#define SWAP_DOWN_TIME 0.4

- (void)viewDidLoad {
    [self.view addSubview:self.videoView];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [self animationManager];
}

- (void)animationManager {
    CABasicAnimation *borderWidthAnimation = [CABasicAnimation animationWithKeyPath:@"borderWidth"];
    borderWidthAnimation.toValue = [NSNumber numberWithFloat:0.0f];
    borderWidthAnimation.duration = 0.25f;
    borderWidthAnimation.fillMode = kCAFillModeForwards;
    borderWidthAnimation.removedOnCompletion = NO;
    borderWidthAnimation.beginTime = 3.0f;

    CABasicAnimation *boundsWidthAnimation = [CABasicAnimation animationWithKeyPath:@"bounds.size.width"];
    boundsWidthAnimation.toValue = [NSNumber numberWithFloat:self.view.frame.size.width];
    boundsWidthAnimation.duration = 3.0f;
    boundsWidthAnimation.fillMode = kCAFillModeForwards;
    boundsWidthAnimation.removedOnCompletion = NO;
    boundsWidthAnimation.beginTime = 0.0f;

    CABasicAnimation *boundsHeightAnimation = [CABasicAnimation animationWithKeyPath:@"bounds.size.height"];
    boundsHeightAnimation.toValue = [NSNumber numberWithFloat:self.view.frame.size.height];
    boundsHeightAnimation.duration = 3.0f;
    boundsHeightAnimation.fillMode = kCAFillModeForwards;
    boundsHeightAnimation.removedOnCompletion = NO;
    boundsHeightAnimation.beginTime = 0.0f;

    CABasicAnimation *positionXAnimation = [CABasicAnimation animationWithKeyPath:@"position.x"];
    positionXAnimation.toValue = [NSNumber numberWithFloat:400];
    positionXAnimation.duration = 3.0f;
    positionXAnimation.fillMode = kCAFillModeForwards;
    positionXAnimation.removedOnCompletion = NO;
    positionXAnimation.beginTime = 0.0f;

    CABasicAnimation *positionYAnimation = [CABasicAnimation animationWithKeyPath:@"position.y"];
    positionYAnimation.toValue = [NSNumber numberWithFloat:500];
    positionYAnimation.duration = 3.0f;
    positionYAnimation.fillMode = kCAFillModeForwards;
    positionYAnimation.removedOnCompletion = NO;
    positionYAnimation.beginTime = 0.0f;

    CAAnimationGroup *groupAnimation = [CAAnimationGroup animation];
    groupAnimation.animations = @[boundsWidthAnimation, boundsHeightAnimation,  borderWidthAnimation, positionXAnimation, positionYAnimation];
    groupAnimation.fillMode = kCAFillModeForwards;
    groupAnimation.removedOnCompletion = NO;
    groupAnimation.duration = 8.0f;
    [self.videoView.layer addAnimation:groupAnimation forKey:@"liveAnimation"];
    
    
    /// 贝塞尔曲线
    CAKeyframeAnimation *animation2 = [CAKeyframeAnimation animationWithKeyPath:@"position"];
    UIBezierPath* path2 = [UIBezierPath bezierPath];
    [path2 moveToPoint:CGPointMake(finishPoint.x, SCREEN_WIDTH/2)];
    [path2 addLineToPoint:finishPoint];
    path2.lineCapStyle  = kCGLineCapRound;
    path2.lineJoinStyle = kCGLineJoinRound;
    [path2 stroke];
    animation2.path = path2.CGPath;
    animation2.duration =SWAP_DOWN_TIME;
    animation2.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
    animation2.fillMode = @"forwards";
    animation2.beginTime = moveTime;
        
/// 多个关键帧动画
    CAKeyframeAnimation *scaleAnimation = [CAKeyframeAnimation animationWithKeyPath:@"transform.scale"];
    scaleAnimation.values = @[@(1.0),@(0.5),@(0.2),@(0.1),@(0.0)];
    scaleAnimation.keyTimes = @[@0.f, @(SWAP_DOWN_TIME/2), @(SWAP_DOWN_TIME),@(SWAP_DOWN_TIME*2),@(1)];
    scaleAnimation.duration = SWAP_DOWN_TIME;
    scaleAnimation.fillMode = @"forwards";
    scaleAnimation.beginTime = moveTime;
}


-  (UIImageView *)videoView {
    if (_videoView == nil) {
        _videoView = [[UIImageView alloc]initWithFrame:CGRectMake(self.view.frame.size.width/2 - 150, self.view.frame.size.height/2 - 150, 300, 300)];
        _videoView.image = [UIImage imageNamed:@"10000"];
        _videoView.contentMode = UIViewContentModeScaleAspectFill;
        _videoView.layer.cornerRadius = 10;
        _videoView.layer.masksToBounds = YES;
        _videoView.layer.borderColor = [UIColor whiteColor].CGColor;
        _videoView.layer.borderWidth = 20;
    }
    return _videoView;
}

// 结束/开始回调
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag {

}

- (void)animationDidStart:(CAAnimation *)anim {

}
```