---
title: iOS点语法与下划线
date: 2022-08-25 17:54:39
tags: [iOS]
---
关于iOS中属性引用，self.xx与_xx，还有self->xx，三者的区别。
<!--more-->
在iOS开发过程中，我们用@proprety声明一个属性后，在代码中我们可以用self.xx与_xx来获取到这个属性。有的还用self->xx去获取。下面就来说说三者的区别。

先说结论：区别就是是否有调用get/set方法。
1. self.xx是点语法，本质上是调用set方法跟get方法，点(.)在(=)左侧，调用set方法，点在(=)右侧调用get方法；
2. _xx,只是使用成员变量的值，不会调用get/set方法；
3. 还有的是直接用xx,去调用，其本质跟_xx一样，只是编程规范问题，后来苹果舍弃了直接用xx调用的写法；
4. self->_xx，是C++的写法，跟_xx一样，不会调用get/set方法。
### 背景
@property 实际上是合并了@synthesize 方法，即生成set跟get方法，生成带下划线的变量'_xx'，
@property 默认是@synthesize，会生成_xx,但你也可以手动写成@dynamic，就不会生成set跟get方法，需要重写set跟get方法，还不会生成'_xx'。

@synthesize与@dynamic的区别

问题
1.当你重写了set跟get方法，系统就不会自动生成_xx变量；
这时候就需要用
```
//为属性property生成一个变量别名
@synthesize property = _property;
```
2.