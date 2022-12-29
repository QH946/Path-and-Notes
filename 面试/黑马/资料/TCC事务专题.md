# 基于微服务的TCC解决方案

# TCC 事务介绍
在08年的软件开发2.0技术大会上，支付宝程立在PPT[大规模SOA系统中的分布事务处理](https://wenku.baidu.com/view/be946bec0975f46527d3e104.html)，提出TCC概念。 在网络上搜索分布式事务相关的博客，基本都会提及这个PPT，目前很多分布式事务开源项目也都是基于TCC的思想实现。

TCC 将事务提交分为 Try - Confirm - Cancel 3个操作。
*   Try：预留业务资源/数据效验
*   Confirm：确认执行业务操作
*   Cancel：取消执行业务操作

TCC事务处理流程和 2PC 二阶段提交类似，不过 2PC通常都是在跨库的DB层面，而TCC本质就是一个应用层面的2PC。

![TCC原理图，图片来自阿里公众号文章](http://upload-images.jianshu.io/upload_images/7528671-b766168be5f08979.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

# TCC 事务应用场景
我们通过用户下单使用现金+红包支付来看一下TCC事务的具体应用。
假设用户下单操作来自3个系统下单系统、现金账户系统、红包账户系统，下单成功需要同时调用现金账户服务和红包服务完成支付。
![image.png](https://upload-images.jianshu.io/upload_images/7528671-9b1d7bfad4503fea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

假设购买商品1000元，使用账户红包200元，现金800元，确认支付。

## Try操作
tryX 下单系统创建待支付订单
tryY 冻结账户红包200元
tryZ 冻结资金账户800元
![image.png](https://upload-images.jianshu.io/upload_images/7528671-c7ed7bff1c89b6df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Confirm操作
confirmX 订单更新为支付成功
confirmY 扣减账户红包200元
confirmZ 扣减资金账户800元
![image.png](https://upload-images.jianshu.io/upload_images/7528671-0c234fe9060e4c42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Cancel操作
cancelX 订单处理异常，资金红包退回，订单支付失败
cancelY 冻结红包失败，账户余额退回，订单支付失败
cancelZ 冻结余额失败，账户红包退回，订单支付失败

我们以TCC-Transaction为例
# 源码下载
[github下载地址](https://github.com/changmingxie/tcc-transaction)
![image.png](https://upload-images.jianshu.io/upload_images/7528671-2263e00b255251fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

# 环境搭建
略

# 演示
## 场景一 
红包账户冻结成功(try)、资金账户冻结成功(try)，订单操作异常(try)
![image.png](https://upload-images.jianshu.io/upload_images/7528671-8e76c602fc05c619.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
此时订单支付失败，红包账户冻结金额、资金账户冻结金额全部退回。
![image.png](https://upload-images.jianshu.io/upload_images/7528671-bd15724df72c285c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
通过多种类似方式测试，例如：红包账户冻结异常、资金账户冻结异常，都会调cancel，退回冻结资源。try阶段的任意一方异常，都会执行全局回滚。

## 场景二
订单处理成功(confirm)，资金账户扣减成功(confirm)，但红包账户扣减失败(confirm)
模拟异常代码，在红包扣减 confirm 操作制造异常：
![image.png](https://upload-images.jianshu.io/upload_images/7528671-e099d8d6ae6a45c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此时订单为支付成功，但实际红包金额还处在冻结状态，事务管理器记录订单confirm操作未执行成功，系统会不断重试调用订单的confirm操作，直到红包扣减成功。
![image.png](https://upload-images.jianshu.io/upload_images/7528671-51a7070e463c068f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
![image.png](https://upload-images.jianshu.io/upload_images/7528671-339df7b26995bffb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

手动将红包服务异常代码去掉，重启服务，等到下一次重试，红包冻结金额被扣除成功
![image.png](https://upload-images.jianshu.io/upload_images/7528671-ecf77274d25880be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

## 场景三
第三种情况: 资金账户冻结成功(try)，红包账户冻结成功(try)，订单处理失败(confirm)
模拟异常代码，在订单 confirm 操作制造异常：
![image.png](https://upload-images.jianshu.io/upload_images/7528671-889f8f538095e5eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

此时订单支付成功，但实际资金账户冻结金额、红包冻结金额都还没有扣除成功，事务管理器记录订单confirm操作未执行成功，系统会通过不断重试订单的confirm操作，直到资金账户和红包账户扣减成功。
![image.png](https://upload-images.jianshu.io/upload_images/7528671-f9ecae8d7bcbf321.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

![image.png](https://upload-images.jianshu.io/upload_images/7528671-b31e2efb1269c11b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

手动将异常代码去掉，重启服务，等到下一次重试，红包和资金账户冻结金额被扣除。


## 场景四
第四种情况：在第一种情况下，订单cancel操作处理失败(cancel)
模拟异常代码，在第一种情况下基础，在cancel操作上在制造异常： 
![image.png](https://upload-images.jianshu.io/upload_images/7528671-d4e4d487e02f456b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
此时订单支付失败，资金账户、红包账户冻结成功，事务管理器记录订单cancel操作失败，系统会不断重试订单的cancel操作，直到资金账户和红包账户冻结金额退回账户。
![image.png](https://upload-images.jianshu.io/upload_images/7528671-1d38cd23937f3528.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

手动将异常代码去掉，重启服务，下一次重试cancel操作，资金账户和红包账户冻结金额退回。
![image.png](https://upload-images.jianshu.io/upload_images/7528671-11e1f811f6898070.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 优缺点
## TCC 优缺点
TCC优点：让应用自己定义数据库操作的粒度，使得降低锁冲突、提高吞吐量成为可能。

## TCC不足之处：
对应用的侵入性强。业务逻辑的每个分支都需要实现try、confirm、cancel三个操作，应用侵入性较强，改造成本高。
实现难度较大。需要按照网络状态、系统故障等不同的失败原因实现不同的回滚策略。为了满足一致性的要求，confirm和cancel接口必须实现幂等。

#总结
try 操作成功，进入 confirm 操作，只要 confirm 处理失败（不管是协调者挂了，还是参与者处理失败或超时），系统通过不断重试直到处理成功。 进入 cancel 操作也是一样，只要 cancel 处理失败，系统通过不断重试直到处理成功。

>问题
>对于转账的cancel事务，相对简单，只要把账务冲负即可。可一般的业务逻辑会涉及很多流程、单证等操作，尤其是历史系统，应该很难改造成tcc结构的吧，不知道你们用tcc用在什么场景下？
>我第一次是在支付宝架构师程立PPT中听说tcc结构的，不知道你的实现跟阿里系的实现是什么关系，目前应用的规模大吗？
>我理解try程序完成后，立即提交try事务，不会有锁事务竞争。可这个时候账户余额的状态也被设置成了类似不可读的状态吧，依然不可以在其他业务中查询账户余额，那么这种方案比2PC优势具体在什么地方？

>回答
>我的感觉TCC是比较适合具有较强一致性要求的场景，账务系统就是这样场景；应用TCC也有一定的开发成本，如果没有强一致性要求，可以考虑其他补偿型方案；
>目前已应用在线上环境多个应用，主要就是应用账务系统里使用；因为不是开源的，我没有看过阿里的实现，部分借鉴了他们的思路；希望有机会能看看；
>在Try结束后，账户余额会有部分资金冻结，其他业务不可以使用冻结资金；和2PC比较的话，理解下来有两点：
>2PC是基于资源层（如数据库），TCC是基于SOA服务
>2PC是是用全局事务，数据被lock的时间跨整个事务，直到全局事务结束；而TCC里每个对资源操作的是本地事务，数据被lock的时间短，可扩展性好

# 延伸阅读
 [TCC-Transaction 源码分析 —— TCC 实现](http://www.iocoder.cn/TCC-Transaction/tcc-core/)

[使用指南1.2.x](https://github.com/changmingxie/tcc-transaction/wiki/%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%971.2.x)