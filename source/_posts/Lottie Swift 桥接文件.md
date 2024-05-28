---
layout: git
title: Lottie Swift 桥接文件 （Objective-C 项目集成 lottie-swift 并模块化处理）
date: 2023-05-19 20:00:00
tags: OC, Swift, Lottie
---

# Lottie Swift 桥接文件（Objective-C 项目集成 lottie-swift 并模块化处理）

## 背景

由于 Objective-C 版本的 Lottie 已经不再更新，随着 UI 设计师使用的插件的升级，一些特效在使用 Objective-C 版本的 Lottie 展示时出现异常。为了应对这一情况，引入 Swift 桥接并迁移旧的 Lottie 方法。

## 操作步骤

### 拉取并修改源文件

1. 创建一个 Objective-C 项目。
2. 使用 CocoaPods 导入 Swift 版本的 Lottie，下载源代码。
3. 修改以下源文件，举例版本为 lottie-ios `4.0.1`

#### 修改 LottieAnimationView.swift

```swift
-final public class LottieAnimationView: LottieAnimationViewBase {
+@objcMembers open class LottieAnimationView: LottieAnimationViewBase {
```

#### 修改 LottieAnimationViewBase.swift

```swift
-public class LottieAnimationViewBase: UIView {
+open class LottieAnimationViewBase: UIView {
```

#### 修改 CompatibleAnimationView.swift

```swift
-public final class CompatibleAnimation: NSObject {
+@objcMembers public final class CompatibleAnimation: NSObject {
```

### 新增 Lottie 的自定义桥文件

由于并非所有方法都和 Objective-C 版本的 Lottie 一样，同时部分方法由于语法原因对 Objective-C 并不暴露或暴露的方式不符合预期，需要增加一个 Swift 文件，对现有的 Lottie 方法进行润色。

```swift
import Foundation

@objcMembers open class BLLOTAnimationView: LottieAnimationView {
    /**
     创建一个 `BLLOTAnimationView` 实例并加载指定名称的 Lottie 动画
     
     - Parameters:
       - name: 动画文件的名称
       - bundle: 包含动画文件的 bundle
     - Returns: 创建的 `BLLOTAnimationView` 实例
     */
    open class func animation(name: String, bundle: Bundle) -> BLLOTAnimationView {
        let lottieView = BLLOTAnimationView()
        lottieView.animation = LottieAnimation.named(name, bundle: bundle)
        return lottieView
    }
    
    /**
     设置动画的循环次数，如果设置为 -1，表示无限循环
     */
    public var loopAnimationCount: CGFloat = 0 {
        didSet {
            self.loopMode = loopAnimationCount == -1 ? .loop : .repeat(Float(loopAnimationCount))
        }
    }
    
    /**
     设置是否循环播放动画
     */
    public var loopAnimation: Bool = false {
        didSet {
            self.loopMode = loopAnimation ? .loop : .playOnce
        }
    }
    
    /**
     设置动画的播放进度
     */
    public var animationProgress: CGFloat = 0 {
        didSet {
            self.currentProgress = animationProgress
        }
    }
    
    /**
     加载指定名称的 Lottie 动画
     
     - Parameters:
       - name: 动画文件的名称
       - bundle: 包含动画文件的 bundle
     */
    public func loadAnimation(name: String, bundle: Bundle) {
        self.animation = LottieAnimation.named(name, bundle: bundle)
    }
    
    @available(iOS 13.0, *)
    public func loadAnimation(url: URL) async {
        self.animation = await LottieAnimation.loadedFrom(url: url)
    }
    
    public func play() {
        self.play(completion: nil)
    }
    
    public func play(fromProgress: CGFloat, toProgress: CGFloat, completion: ((_ animationFinished: Bool) -> Void)? = nil) {
        self.play(fromProgress: fromProgress, toProgress: toProgress, loopMode: nil, completion: completion)
    }
}
```

### 调用桥文件里的内容

1. 在使用 Lottie 库或每次更改 Swift 代码时，可以查看桥文件，以确保方法结构符合预期。
2. 此处使用模块化方式，模块名为 `BLLottie`，引入桥文件方式为 `#import <BLLottie/BLLottie-Swift.h>`。
3. 封装 Lottie 库。

#### 封装 Lottie 库

##### BLLOTAnimationView+BLLottie.h

```objc
#import <BLLottie/BLLottie-Swift.h>

NS_ASSUME_NONNULL_BEGIN

typedef void (^LOTAnimationCompletionBlock)(BOOL animationFinished);
typedef void (^LOTAnimationUrlLoadedBlock)(BOOL isSuccess);

@interface BLLOTAnimationView (BLLottie)

+ (nonnull instancetype)animationNamed:(nonnull NSString *)animationName inBundle:(nonnull NSBundle *)bundle;

- (void)animationWithUrl:(NSURL *)url completionHandler:(LOTAnimationUrlLoadedBlock)completionHandler;

- (void)setAnimationNamed:(nonnull NSString *)animationName inBundle:(nonnull NSBundle *)bundle;

- (void)playFromProgress:(CGFloat)fromStartProgress toProgress:(CGFloat)toEndProgress withCompletion:(nullable LOTAnimationCompletionBlock)completionBlock;

@end

NS_ASSUME_NONNULL_END
```

##### BLLOTAnimationView+BLLottie.m

```objc
#import "BLLOTAnimationView+BLLottie.h"

@implementation BLLOTAnimationView (BLLottie)

+ (nonnull instancetype)animationNamed:(nonnull NSString *)animationName inBundle:(nonnull NSBundle *)bundle {
    return [BLLOTAnimationView animationWithName:animationName bundle:bundle];
}

- (void)animationWithUrl:(NSURL *)url completionHandler:(LOTAnimationUrlLoadedBlock)completionHandler {
    if (@available(iOS 13.0, *)) {
        [self animationWithUrl:url completionHandler:completionHandler];
    } else {
        // Fallback on earlier versions
        if (completionHandler) {
            completionHandler(NO);
        }
    }
}

- (void)setAnimationNamed:(nonnull NSString *)animationName inBundle:(nonnull NSBundle *)bundle {
    [self setAnimationWithName:animationName bundle:bundle];
}

- (void)playFromProgress:(CGFloat)fromStartProgress toProgress:(CGFloat)toEndProgress withCompletion:(nullable LOTAnimationCompletionBlock)completionBlock {
    [self playFromProgress:fromStartProgress toProgress:toEndProgress completion:completionBlock];
}

@end
```

经过上述步骤，您可以在 Objective-C 项目中顺利集成并使用最新版本的 Lottie 动画库，同时通过自定义桥文件实现对老版本方法的兼容和优化。