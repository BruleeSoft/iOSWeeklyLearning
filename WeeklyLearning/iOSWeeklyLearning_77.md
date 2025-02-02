# iOS 摸鱼周报 #77 |圣诞将至，请注意 App 审核进度问题

![](https://cdn.zhangferry.com/Images/moyu_weekly_cover.jpeg)

### 本期概要

> * 本期话题：圣诞节将至，请为您的 App 做好准备； Developer 设计开发加速器｜如何在游戏中支持手柄、在 iPhone 锁屏上扩展你的 App 体验
> * 本周学习：class_rw_t 与 class_ro_t 的区别
> * 内容推荐：Swift 优秀文章推荐，涉及协程，类型转换以及 `inlinable` 关键字讲解
> * 摸一下鱼：CSDN上线的新工具箱应用猿如意，内置了好用的效率工具

## 本期话题

### [圣诞节将至，请为您的 App 做好准备](https://developer.apple.com/cn/news/?id=xpkhwg3l "圣诞节将至，请为您的 App 做好准备")

[@师大小海腾](https://juejin.cn/user/782508012091645/posts)：App Store 最繁忙的季节即将到来！确保及时更新您的 App 和产品页面，并在岁末假日到来之前做好准备。很高兴今年的整个假日季期间同样会开放 App 提交，我们非常期待看到您提交的 App。平均而言，90% 的提交内容会在 24 小时内得到审核。但请注意，在 12 月 23 日至 27 日，完成审核所需的时间可能略长一些。

###  Developer 设计开发加速器｜如何在游戏中支持手柄

[@师大小海腾](https://juejin.cn/user/782508012091645/posts)：准备好了提升您的游戏操作方式吗？我们会带您了解 iPhone、iPad、Mac 和 Apple TV 平台上的虚拟和实体游戏控制器的最新改进，新推出的屏幕上的虚拟游戏控制器可以将触摸输入变成游戏控制器的输入。我们将告诉您如何在您游戏中添加对最新游戏控制器的支持和添加分享功能。您还可以了解如何将 Game Controller 框架与 Core Haptics 框架一起使用，以实现触觉反馈。学习如何通过自定义按钮映射，非标准输入，以及支持运动传感器、灯光和自适应扳机键等特殊功能，使您的游戏体验更上一层楼。

活动将于 2022 年 12 月 6 日（周二）举办。名额有限。请在 2022 年 12 月 5 日前报名参加。**报名地址**：https://developer.apple.com/events/view/2L9P7JC7G4/dashboard。

###  Developer 设计开发加速器｜在 iPhone 锁屏上扩展你的 App 体验

[@师大小海腾](https://juejin.cn/user/782508012091645/posts)：发现如何利用 iPhone 锁屏上的特性，进一步扩展你的 App 体验。学习如何利用锁屏小组件、实时活动以及通知，为用户创造更丰富的体验。

活动将于 2022 年 12 月 8 日（周四）举办。名额有限。请在 2022 年 12 月 7 日前报名参加。**报名地址**：https://developer.apple.com/events/view/US9PY3MAMG/dashboard。

## 本周学习

整理编辑：[宁静致远](https://github.com/byshb)

### class_rw_t 与 class_ro_t 的区别

这两个结构体类型在苹果 opensource 的源码中定义的，于是直接打开源代码（[objc4-838](https://github.com/apple-oss-distributions/objc4/tree/objc4-838)）进行分析：

```c++
struct class_rw_t {
    ...
    const class_ro_t *ro();
    const method_array_t methods(){ ... };
    const property_array_t properties(){ ... };
    const protocol_array_t protocols(){ ... };
    ...
}
```

```c++
struct class_ro_t {
    ...
    union {
        const uint8_t * ivarLayout;
        Class nonMetaclass;
    };
    explicit_atomic<const char *> name;
    WrappedPtr<method_list_t, method_list_t::Ptrauth> baseMethods; // 方法列表
    protocol_list_t * baseProtocols; // 协议列表
    const ivar_list_t * ivars; // 成员变量列表
    const uint8_t * weakIvarLayout; 
    property_list_t *baseProperties; // 属性列表
    ...
};
```

从代码中可见 `class_ro_t` 结构体存在于 `class_rw_t` 结构体当中，下文使用 `ro` 和 `rw` 替代。

苹果 WWDC 曾经介绍过 `ro `  和 `rw` ，并引出了两个概念，clean memory 和 dirty memory。

clean memory 是指加载后不会发生改变的内存。它可以进行移除来节省更多的内存空间，需要时再从磁盘加载。

dirty memory 是指在运行时会发生改变的内存。当类开始使用时，系统会在运行时为它分配一块额外的内存空间，也就是 dirty memory，只要进程在运行，它就会一直存在，因此使用代价很高。

`ro` 放在纯净的内存空间，是只读的，对于没有使用到的 `ro`，可以进行移除，需要时再分配。

`rw` 在运行生成，可读可写，属于脏内存。

`ro` 在编译阶段创建，将类的属性，方法，协议和成员变量添加到 `ro` 中，编译后就已经确定了。

`rw` 运行的时候创建，首先会将 `ro` 中的内容**剪切**到 `rw` 中，分类中的方法会在运行时，添加到 `rw` 的 `method_array_t` 结构的 `methods` 中，由于是放到了数组的前面部分，可达到类似**覆盖**的效果。

我们分析 `rw` 的源码时，可见 methods、properties、protocols 其实是可能存在 一个叫做 `ro_or_rw_ext`变量当中，举例如下：

```c++
const method_array_t methods() const {
    auto v = get_ro_or_rwe();
    if (v.is<class_rw_ext_t *>()) {
        return v.get<class_rw_ext_t *>(&ro_or_rw_ext)->methods;
    } else {
        return method_array_t{v.get<const class_ro_t *>(&ro_or_rw_ext)->baseMethods};
    }
}
```

之所以这样设计，是由于 `rw` 属于脏内存，使用开销大，苹果在 WWDC ⾥⾯说过，只有⼤约 10% 左右的类需要动态修改。所以只有 10% 左右的类⾥⾯需要⽣成 `class_rw_ext_t` 这个结构体。把一些类的信息分离出来，这样的话，可以节约很⼤⼀部分内存。

`class_rw_ext_t` 的⽣成的条件：

1. ⽤过 Runtime 的 Api 进⾏动态修改的时候。
2. 有分类的时候，且分类和本类都为⾮懒加载类的时候。实现了 `+load` ⽅法即为⾮懒加载类。

还有就是经上述分析，成员变量是存在于 `ro` 当中的，一经编译就不能修改了，那是不是所有的类都不能运行时添加实例变量了呢？答案是运行时创建的类，可以在 `objc_allocateClassPair` 方法之后，`objc_registerClassPair` 方法之前，通过 `class_addIvar()` 添加实例变量，除此之外已经创建的类的实例变量内存布局是不能被修改的。


## 内容推荐

整理编辑：[夏天](https://juejin.cn/user/3298190611456638)

1、[当谈论协程时，我们在谈论什么](https://mp.weixin.qq.com/s/IO4ynnKEfy2Rt-Me7EIeqg)  -- 来自： 腾讯程序员

[@夏天](https://juejin.cn/user/3298190611456638): 本文详细介绍了协程的概念，作者通过文章来回答了四个问题： 

* **Q1 (Why):** 为什么需要协程？
* **Q2 (What):** 到底什么是协程？
* **Q3 (How):** 怎么实现协程 (库)？
* **Q4 (Usage):** 使用协程时需要注意什么？

不但能够帮助你理解协程，而且文章结构清晰。

2、[Be careful with Obj-C bridging in Swift](https://swiftrocks.com/be-careful-with-objc-bridging-in-swift "Be careful with Obj-C bridging in Swift") -- 来自：SwiftRocks

[@夏天](https://juejin.cn/user/3298190611456638): 当我们要将 Swift 中 `String` 转为 `NSString` 时，一般使用 `as` 语法糖来进行转换，但是作者认为这并不是一个安全的方案。 

3、[Understanding `@inlinable` in Swift](https://swiftrocks.com/understanding-inlinable-in-swift "Understanding `@inlinable` in Swift") -- 来自：SwiftRocks

[@夏天](https://juejin.cn/user/3298190611456638): 了解 `@inlinable` 是如何工作的，通过使用 `@inlinable` 可以提高我们的某些代码的性能。

4、[App Store and TestFlight review times](https://www.runway.team/appreviewtimes "App Store and TestFlight review times") --  来自：RUNWAY

[@夏天](https://juejin.cn/user/3298190611456638): 一个关于 App Store 和 TestFlight 审核时间的统计的文章。当我们上架到 App Store 和 TestFlight 时，我们大概需要多久才能审核成功。


## 摸一下鱼

整理编辑：[CoderStar](https://mp.weixin.qq.com/mp/homepage?__biz=MzU4NjQ5NDYxNg==&hid=1&sn=659c56a4ceebb37b1824979522adbb15&scene=18)

1. 最近CSDN上线的一款新工具箱应用-- [猿如意](https://devbit.csdn.net/ "猿如意")，内置一些好用的效率工具；

![猿如意](http://cdn.zhangferry.com/20221201180541.png)

类似的工具还有 [uTools](https://www.u.tools/ "uTools")

![uTools](http://cdn.zhangferry.com/20221201181157.png)

2. [来个电话](https://apps.apple.com/cn/app/%E6%9D%A5%E4%B8%AA%E7%94%B5%E8%AF%9D-%E5%81%87%E6%9D%A5%E7%94%B5%E8%AE%A9%E4%BD%A0%E8%84%B1%E8%BA%AB%E8%A7%A3%E5%9B%B4/id985946832 "来个电话") 是一款可以帮助你从无聊的无意义社交中脱身解围的应用。不仅是精确到分钟的来电时间，还一比一仿真了系统的来电界面，当你耳朵贴近手机时也会自动熄灭屏幕，让你能毫无破绽、以假乱真的离开无聊的聚会。原价 12 元，目前限免中。设置界面如下：

    ![](https://cdn.zhangferry.com/Images/weekly_77_moyu_1.png)

3、[CampusShame](https://github.com/forthespada/CampusShame "CampusShame")：Github 上总结的校招污点和非污点公司名单。校招污点行为**包括**但不限于：毁意向书、毁两方协定、毁三方协定、试用期裁员、大量裁应届生。

4、[Roadmap.sh](https://roadmap.sh/ "Roadmap.sh")：这是一个致力于帮助开发者学习和选择的社区，这里提供了很多岗位的 Roadmap 和优质学习资料。

![](https://cdn.zhangferry.com/Images/20221201233541.png)

5、[email 是如何工作的？](https://twitter.com/alexxubyte/status/1593637888834473984 "email 是如何工作的？")：这张图描述了邮件服务的工作原理。

![](https://cdn.zhangferry.com/Images/20221202000720.png)

邮件服务主要有两套协议组成，一个是 SMTP 用于存储邮件内容，一个是 IMAP/POP 用于拉取邮件内容，对于不同的邮件服务，会由接收方进行发送，这里用的是 SMTP 协议。

6、[正则表达式速查卡](https://twitter.com/linuxtoy/status/1597754607697948673 "正则表达式速查卡")：正则表达式是一种很实用的文本模式匹配语言，Linuxtoy 总结了这份速查卡。

![](https://cdn.zhangferry.com/Images/20221202002037.png)

## 关于我们

iOS 摸鱼周报，主要分享开发过程中遇到的经验教训、优质的博客、高质量的学习资料、实用的开发工具等。周报仓库在这里：https://github.com/zhangferry/iOSWeeklyLearning ，如果你有好的的内容推荐可以通过 issue 的方式进行提交。另外也可以申请成为我们的常驻编辑，一起维护这份周报。另可关注公众号：iOS成长之路，后台点击进群交流，联系我们，获取更多内容。

### 往期推荐

[iOS 摸鱼周报 #76 | 程序员提问的智慧](https://mp.weixin.qq.com/s/UmXvtKYS6Z0a30yPRyIV9g)

[iOS 摸鱼周报 #75 | 远程工作推行之难](https://mp.weixin.qq.com/s/nguqKvkuzDBR9o-Yw6y3KQ)

[iOS 摸鱼周报 #74 | 抖音 iOS 基础技术大揭秘 Vol.02 周六见](https://mp.weixin.qq.com/s/lhhV0Qlc9NtFoM6nF7gZbA)

[iOS 摸鱼周报 #73 | macOS Ventura 初体验](https://mp.weixin.qq.com/s/5chb-a9u7VMdLis1FG6B6Q)

![](https://cdn.zhangferry.com/Images/WechatIMG384.jpeg)
