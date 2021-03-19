---
layout: git
title: 编译错误 Duplicate symbols for architecture x86_64
date: 2021-03-19 20:00:00
tags: OC
---
# 编译错误 Duplicate symbols for architecture x86_64

## 错误信息
```
....../*Objects-normal/x86_64/XXXX.o
ld: 415 duplicate symbols for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)

```

> 翻译：
> 错误信息是在链接 XXXX.o 时出错
> ld: 在 x86-64 架构下有 415 个重复符号

## 可能触发该问题的两种情况
### 情况一：
* 触发原因：
由于重复导入了某文件项目中重复导入了某些文件, 一般在导入三方库时可能会重复导入
* 解决办法：
只需要在文件目录中查找到重复导入的文件，删掉即可。或将整个三方库删除掉，重新拖入，或者用 cocopods install

### 情况二：
* 触发原因：
在项目的某些地方需要 `#import"XXXX.h"` 而用了 `#import"XXXX.m"`。
* 解决办法：
核对项目