# iOS 摸鱼周报 #70 | iOS / iPadOS 16.1 公测版 Beta 3 发布，支持老款 iPad 台前调度

![](https://cdn.zhangferry.com/Images/moyu_weekly_cover.jpeg)

### 本期概要

> * 本期话题：苹果 iOS / iPadOS 16.1 公测版 Beta 3 发布，为老款 iPad 支持台前调度
> * 本周学习：排查修复 App Store 上架项目闪退问题
> * 内容推荐：iOS 开发技巧及计算机基础内容学习
> * 摸一下鱼：计算麦当劳套餐卡路里的营养计算器，可以模拟木鱼声音的软件，以及插图绘制软件

## 本期话题

### 苹果 iOS / iPadOS 16.1 公测版 Beta 3 发布，为老款 iPad 支持台前调度

[@师大小海腾](https://juejin.cn/user/782508012091645/posts)：苹果于 9 月 29 日向 iPhone 用户推送了 **iOS 16.1 公测版 Beta 3 更新（内部版本号：20B5056e）**，本次更新距离上次发布隔了 7 天。此外，iPadOS 16.1 也迎来新的公测版。

公测版的更新内容与昨日推送的开发者预览版基本一致，对于苹果 iPad 用户，苹果将台前调度功能扩展到了搭载 **A12X 或 A12Z 芯片**的 iPad Pro，不过功能上有缺失，无法支持外部显示器使用。该功能此前只能在 M1 芯片的 iPad 上使用。

![](https://cdn.zhangferry.com/Images/20220929212647.png)

在 iOS 16.1 的最新更新中，苹果在设置中加入了新的修改壁纸选项，并且移除了 Matter 配件，添加了新的开关选项，允许应用在下载后和用户首次启动之前自动开始下载其应用内内容。

![](https://cdn.zhangferry.com/Images/20220929212708.png)

![](https://cdn.zhangferry.com/Images/20220929212719.png)

另外，苹果将在 iOS 16.1 中为用户带来实时活动通知，用户将可以在锁屏、灵动岛上看到 App 的实时消息。

![](https://cdn.zhangferry.com/Images/20220929212740.png)

来源：[IT之家 - 苹果 iOS / iPadOS 16.1 公测版 Beta 3 发布，为老款 iPad 支持台前调度](https://m.ithome.com/html/643916.htm "IT之家 - 苹果 iOS / iPadOS 16.1 公测版 Beta 3 发布，为老款 iPad 支持台前调度")

## 本周学习

整理编辑：[FBY 展菲](https://github.com/fanbaoying)

### App Store 已上架项目打开瞬闪问题

#### 1. 问题背景

用户反馈 iPhone11 iOS14.7 下载安装 App 后，点击图标，App 闪一下就回到了桌面。

收到问题反馈之后，使用手上测试机测试，iPhone11 iOS15.5 和 iPhone12 iOS15.0 均没有复现问题。

一时没有找到和用户相同的版本的测试手机，找到一台 iPhone11 iOS13.6 的手机。复现了问题。

后面使用 iPhone7 iOS13.6 也复现了问题。iPhoneX iOS16.0 没有问题。

#### 2. 问题分析

问题分析使用的是 iPhone11 iOS13.6 和 iPhone7 iOS13.6 两部手机。

App 安装版本限制是 iOS13 及以上版本。

**怀疑一：** 是项目中引入的音频动态库版本太老不兼容导致。

检查之后发现虽然和最新版本差了2个小版本，并且文档中没有更新提示相关兼容性问题。并且项目打包上架，经过了 `Validate App`。排除怀疑。

**怀疑二：** 系统 Api 在 iOS15.0 以下版本不兼容 。

如果是系统 Api 不兼容，不管是直接在 App store 下载安装，还是直接编译到手机，都会有问题。实际测试，直接编译到手机没有复现问题。

**怀疑三：** 群友提出可能是因为 Xcode 版本太老导致的问题

我目前的 Xcode 版本是 13.3.1，最新版本是 13.4.1，只差了一个小版本。

**怀疑四：** 群友提出可能电脑是 M1 芯片导致

感觉关系不大。

#### 3. 问题调试

根据以上的四个疑问，逐个排查。

在调试之前，已经清除掉手机上已经存在的 App，并且卸载清除掉所有缓存。

**1. 联机调试**

手机连接电脑，直接编译到手机中。App 正常使用，没有闪退问题

**2. Crashes**

Xcode 中的 Crashes 也没有收到任何崩溃信息。

**3. TestFlight**

通过 TestFlight 的内外部测试，收集闪退的问题。

**4. 升级 Xcode**

申请使用备用电脑，进行 Xcode 升级，项目打包上架。在 Xcode 升级到 13.4.1 后打包上架的项目，闪退的问题消失。


来源：[App Store 已上架项目打开瞬闪问题 - Swift 社区](https://mp.weixin.qq.com/s/QOB5alijsV5Gg8pi4lg03g "App Store 已上架项目打开瞬闪问题 - Swift 社区")

## 内容推荐

整理编辑：[Mimosa](https://juejin.cn/user/1433418892590136)

1、[How necessary are the programming fundamentals?](https://swiftrocks.com/how-necessary-are-the-programming-fundamentals "How necessary are the programming fundamentals?") -- 来自： Bruno Rocha

[@Mimosa](https://juejin.cn/user/1433418892590136)：我们平时写写业务代码好像用不到高深的算法、数据结构等知识，但是在大厂的面试中似乎又不可避免以高难度的形态出现，那这些编程基础知识到底有什么用呢？本文的作者讨论了这一普遍的现象，并以生动的例子提出了自己的见解和类比，并解释了这种情况出现的原因，同时也对这种略显病态的面试流程提出了自己的看法，如果你也有类似的疑惑，相信这篇文章可以给你一些启发。

2、[DocC Tutorial for Swift](https://www.raywenderlich.com/34919511-docc-tutorial-for-swift-getting-started "DocC Tutorial for Swift") -- 来自： raywenderlich

[@Mimosa](https://juejin.cn/user/1433418892590136)：在 WWDC21 上，Apple 推出了 DocC，这是一个文档编译器，可以在 Xcode 文档窗口中构建和查看 Swift 包的文档。Apple 在 WWDC22 中扩展了 DocC 功能，因此它也可以记录 Swift 和 Objective-C 项目。在这个教程中，会告诉你 DocC 的工作原理、一些实操的例子以及如何导出和发布。

3、[How to create Rounded Corners Button in UIKit](https://sarunw.com/posts/uikit-rounded-corners-button/ "How to create Rounded Corners Button in UIKit") -- 来自： Sarunw

[@Mimosa](https://juejin.cn/user/1433418892590136)：相信很多人还不知道在 iOS 15 之后，我们可以使用 `UIButton.Configuration` 来设置按钮的圆角以及其他表现，这篇文章就带大家熟悉一下这个好用的配置属性。

4、[聊聊 iOS 中的像素对齐](https://juejin.cn/post/7124658703088910350 "聊聊 iOS 中的像素对齐") -- 来自： JPlay

[@Mimosa](https://juejin.cn/user/1433418892590136)：当一个 UILabel 的宽度是 500.001 和 500 时会有什么区别？本文探讨了像素不对齐出现的原因以及系统像素补齐的原则，并给出了一些避免和解决的方法。

5、[重新开始学习计算机](https://juejin.cn/post/7124660156612214814 "重新开始学习计算机") -- 来自： JPlay

[@Mimosa](https://juejin.cn/user/1433418892590136)：这是一份一位程序员面对 35 岁职业魔咒的应对之道：拥有扎实的、广泛的、不受平台局限的计算机基础知识。


## 摸一下鱼

整理编辑：[夏天](https://juejin.cn/user/3298190611456638)

1、[麦当劳-营养计算器](https://www.mcdonalds.com.cn/nutrition_calculator "麦当劳-营养计算器")：如同其提及的，这是一款计算麦当劳套餐卡路里的网页，你可以怀揣着健康饮食的心来面对你手中的汉堡和多冰可乐。

![](https://cdn.zhangferry.com/Images/nutrition_calculator.png)

2、[Wooden Fish](https://apps.apple.com/app/id1522144157 "Wooden Fish"): 木鱼-念经助手。一个模拟真实敲木鱼的App。如果你是一个虔诚的基督徒，你可以试试 [我的圣经](https://apps.apple.com/cn/app/my-holy-rosary-%E6%88%91%E7%9A%84%E5%9C%A3%E7%BB%8F-%E6%9C%89%E5%A3%B0%E8%AF%BB%E7%89%A9/id1188342937?mt=12 "我的圣经")。

![](https://cdn.zhangferry.com/Images/wooden_fish.png)

3、[Vectornator](https://www.vectornator.io "Vectornator"): 一款好用的插图绘制软件。

![](https://cdn.zhangferry.com/Images/vectornator.png)

> 官网的交互体验感觉很不错

## 关于我们

iOS 摸鱼周报，主要分享开发过程中遇到的经验教训、优质的博客、高质量的学习资料、实用的开发工具等。周报仓库在这里：https://github.com/zhangferry/iOSWeeklyLearning ，如果你有好的的内容推荐可以通过 issue 的方式进行提交。另外也可以申请成为我们的常驻编辑，一起维护这份周报。另可关注公众号：iOS成长之路，后台点击进群交流，联系我们，获取更多内容。

### 往期推荐

[iOS 摸鱼周报 #69| 准备登陆灵动岛](https://mp.weixin.qq.com/s/Miy8xsHYHHSXsl5NtxswQA)

[iOS 摸鱼周报 #68 |  iPhone14 灵动岛创意](https://mp.weixin.qq.com/s/YNukagI-VTOsIkhlYM6dEQ)

[iOS 摸鱼周报 #67 | Xcode Cloud 已支持订阅](https://mp.weixin.qq.com/s/8H7YnrVTubKvVnYJBXcF_A)

[iOS 摸鱼周报 #66 | Shazam 迎来问世 20 周年](https://mp.weixin.qq.com/s/5chb-a9u7VMdLis1FG6B6Q)

![](https://cdn.zhangferry.com/Images/WechatIMG384.jpeg)
