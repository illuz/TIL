# DDD, Java Log System

## 领域驱动设计 DDD - 初探

大佬的文章集合：https://martinfowler.com/tags/domain%20driven%20design.html
SO 上的回答：https://stackoverflow.com/questions/1222392/can-someone-explain-domain-driven-design-ddd-in-plain-english-please/1222488#1222488
中文博文介绍：http://blog.didispace.com/txh-ddd-model/
中文电子书：InfoQ 的「领域驱动设计精简版」 http://www.infoq.com/cn/minibooks/domain-driven-design-quickly-new

DDD 主要解决的是业务专家和软件专家之间的模型约定不统一的问题。领域是指业务领域，就是一套先搞清楚领域，制定领域模型，然后再面向这些模型开发的流程。

按一个大佬说的，就是：

1. 确定问题域
2. 回归业务本质，理解业务的时序和规律
3. 对业务本质和流程做抽象
4. 定义模型
5. 定义上下文

而这些模型成功与否，就在于「我们需要解决的问题如何能够毫无歧义的表达出来」。

#### Apache ISIS

https://github.com/apache/isis

一个能快速搭建 DDD 项目的工具？

感觉很冷门，基本上没有人用的样子。（或者是原因为名字的关系？）

## Java Log System

文章：https://mp.weixin.qq.com/s/XiCky-Z8-n4vqItJVHjDIg 主要介绍了 Java 日志系统的历史

Java 的日志工具还是有不少的，一般用的 log4j 或 logback，众多的日志工具列表见：http://www.java-logging.com/

Slf4j 是一个 java 日志系统的接口，和 Lockback 直接兼容，也能适配其它日志系统，如 Log4j, JDK loggin, tinylog 等等。

顺便一提，lombok 插件可以有个实用的 @Slf4j 注解，可以帮你注入 Slf4j 接口的日志。

另外，还有 log4j 和 logback 的比较：https://stackoverflow.com/questions/178215/log4j-vs-logback。





