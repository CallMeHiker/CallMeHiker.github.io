---
title: Markdown简单语法规则
date: 2018-03-02 17:38:46
tags: [Markdown]
---
## 认识Markdown
轻量级文本编辑器，优点多多不一一赘述。在Mac 上，推荐应用 [Mou](http://25.io/mou/) ，不过，现在这个只支持OS X 10.7 to 10.11，像我的Sierra，还需等待一段时间。
(更新：现在我用的是[Typora](https://typora.io/)，额~感觉不大想推荐，因为它写了之后只显示最后效果，没有了markdown的语法，修改起来，会麻烦一点。不想折腾还是直接在简书上Markdown吧，哈哈哈)

PS:写了之后，发现简书上其实有了更全面的，给个[传送门](https://www.jianshu.com/p/191d1e21f7ed)。
<!--more-->

## Markdown的简单语法及范例
### 1. 一级标题
一个`#`

### 2. 二级标题
 两个`##`
 没错，你已经想到了，每个多一级标题，就多一个"#"，最多6级标题，每个"#"后面，最好多敲一个空格。
 
 ### 3. 无序列表
 文字前直接加 "`*`"
 例子：
 * 我是无序列表
 * 我也是无序列表
 

 ### 4. 有序列表
 文字前直接加 "1." "2." "3."
 例子：
 1. 我是有序列表
 2. 我也是有序列表
 

 ### 5. 分割线
 三个 `***`
 例子：
***
### 6. 引入
 文字前直接加 ">"(大于号)
 例子：
 > 我是引入的内容
 ### 7. 插入图片
其实就是超链接前加个`!`
例子：
`![我养你啊](http://upload-images.jianshu.io/upload_images/1648750-db628ae15041d78d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)` ==>
![我养你啊](http://upload-images.jianshu.io/upload_images/1648750-ebaf61e474f854fd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 8. 插入链接
` [需要链接的文字](链接地址)`
 例子：`[Hiker的博客](https://callmehiker.github.io/)`  ==>  [Hiker的博客](https://callmehiker.github.io/)
 
 ### 9. 粗体字
`**需要加粗的文字**（将文字用4个*包裹起来）`
 例子：**我是粗体字**
 
 ### 10. 斜体字
` *需要斜体的文字*（将文字用2个"**"包裹起来）`
例子： *我是斜体字*
 
 ### 11. 代码框(程序员最爱)
 `我是代码`（将代码用两个```包裹起来，符号在键盘左上角，需要在英文输入模式下敲打）
```
//远程推送注册成功
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    // Required - 注册 DeviceToken
    [self JpushServiceRegisterDeviceToken:deviceToken];
}
```
 ### 12. 红色字体
> 用两个 ``包起来

例子 `我是红色字体`
