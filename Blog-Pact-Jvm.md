

如今，契约测试已经逐渐成为测试圈中一个炙手可热的话题，特别是在微服务大行其道的行业背景下，越来越多的团队开始关注服务之间的契约及其契约测试。

从2016年开始我就在Thoughtworks和QA Community里推广契约测试，收到了不错的成效，期间有不少同学和我讨论过如何上手契约测试，发现网上介绍契约测试的讲义、博客不乏其数（当然，质量也参差不齐），可手把手教你写契约测试的文章却几乎没有，原因恐怕就是契约测试的特性吧。契约测试是架设在消费者和服务者之间的一种比较特殊的测试活动，如果你只是想自己学习而又没有合适的项目环境，那你得自己先准备适当的消费者和服务者程序源代码，然后再开始写契约测试，而不是像写个Selenium测试那样，两三行代码就可以随随便便地调戏度娘。～(￣▽￣～)

所以，我花了些时间磨叽出了这片文章……

本文不会涉及契约测试的基本概念，因为相应的文章网上太多了，请大家自己去捞吧。本文也不会讨论契约测试的使用场景以及对其的正确理解，这方面的话题我会在今后另文介绍（好吧，我承认我很懒ㄟ(▔,▔)ㄏ）。

---

OK，以下开始正文！

契约测试的精髓在于消费者驱动，实践消费者驱动契约测试的工具主要有Pact，Pacto 和 Spring Cloud Contract，其中Pact是目前最为推荐的，我下面的例子都将使用Pact来练习。[Pact](https://docs.pact.io/)最早是用Ruby实现的，目前已经扩展支撑Java，.NET，Javascript，Go，Swift，Python和PHP，其中[Java（JVM）](https://github.com/DiUS/pact-jvm)是我们目前项目中使用最频繁的，所以我的例子亦都是基于PACT JVM来实现（观众A:我们都用Pyhton，你丫给我说Java(╯°□°）╯︵┻━┻）

## 示例源码
大家可以从Github上获取本文示例的[源码](https://github.com/Mikuu/Pact-JVM-Example)，也可以从PACT JVM官网上面找到对应的PACT-JVM-Example[链接](https://github.com/DiUS/pact-jvm#links)

## 示例应用
示例应用非常简单，一个服务提供者Provider，两个服务消费者Miku和Nanoha（啥？你不知道Miku和Nanoha是什么？......问度娘吧......(～o￣3￣)～......）。
### Provider
Provider是一个简单的API，返回一些个人信息。
启动Provider：
```commandline
./gradlew :example-provider:bootRun
```
然后访问 http://localhost:8080/information?name=Miku
![](images/pact-jvm/provider.miku.png)

或者访问 http://localhost:8080/information?name=Nanoha
![](images/pact-jvm/provider.nanoha.png)

### 消费者 Miku & Nanoha
Miku和Nanoha调用Provider的API拿到各自的数据，然后显示在前端页面上。
启动Miku：
```commandline
./gradlew :example-consumer-miku:bootRun
```
然后用浏览器访问 http://localhost:8081/miku
![](images/pact-jvm/consumer.miku.png)

启动Nanoha：
```commandline
./gradlew :example-consumer-nanoha:bootRun
```
然后用浏览器访问 http://localhost:8082/nanoha
![](images/pact-jvm/consumer.nanoha.png)

Miku和Nanoha做的事情基本一样，差别就是Nanoha会去多拿`.nationality`这个字段，而`.salary`这个字段Miku和Nanoha都没有用到。


## Provider与Miku间的契约测试
## 相关的Gradle配置
## Provider与Nanoha间的契约测试
## 验证我们的测试





