

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

> 示例中的1个Provider和2个Consumer都在一个codebase里面，这只是为了方便管理示例代码，而实际的项目中，绝大多数的Provider和Consumer都是在不同的codebase里面管理的，请注意哟！

## Provider与Miku间的契约测试
好了，大概了解示例应用之后，我们就可以开始写契约测试了（当然，如果你还想再撩撩示例的源码，也是可以的啦，不过相信我，里面没多少油水的）

我们先从Provider和Miku`之间`的契约测试开始。
> 请注意"之间"这个关键词，当我们谈论契约测试时，一定要明确它是建立在某一对Provider和Consumer之间的测试活动。没有Provider，Consumer做不了契约测试；没有Consumer，Provider不需要做契约测试。

###编写消费者Miku端的测试案例
目前，PACT JVM在消费者端的契约测试主要有三种写法：
* 基本的Junit
* Junit Rule
* Junit DSL

它们都能完成消费者端契约文件的生成，只是写法有所不同，带来的代码简洁度和部分功能有些许差异。
> 所有的契约测试代码都已经写好了，你可以在`src/test/java/ariman/pact/consumer`下面找到。

#### 基本的Junit
“talk is cheap, show you the code”

`PactBaseConsumerTest.java`

```java
public class PactBaseConsumerTest extends ConsumerPactTestMk2 {

    @Override
    @Pact(provider="ExampleProvider", consumer="BaseConsumer")
    public RequestResponsePact createPact(PactDslWithProvider builder) {
        Map<String, String> headers = new HashMap<String, String>();
        headers.put("Content-Type", "application/json;charset=UTF-8");

        return builder
                .given("")
                .uponReceiving("Pact JVM example Pact interaction")
                .path("/information")
                .query("fullName=Miku")
                .method("GET")
                .willRespondWith()
                .headers(headers)
                .status(200)
                .body("{\n" +
                        "    \"salary\": 45000,\n" +
                        "    \"fullName\": \"Hatsune Miku\",\n" +
                        "    \"nationality\": \"Japan\",\n" +
                        "    \"contact\": {\n" +
                        "        \"Email\": \"hatsune.miku@ariman.com\",\n" +
                        "        \"Phone Number\": \"9090950\"\n" +
                        "    }\n" +
                        "}")

                .toPact();
    }

    @Override
    protected String providerName() {
        return "ExampleProvider";
    }

    @Override
    protected String consumerName() {
        return "BaseConsumer";
    }

    @Override
    protected void runTest(MockServer mockServer) throws IOException {
        ProviderHandler providerHandler = new ProviderHandler();
        providerHandler.setBackendURL(mockServer.getUrl());
        Information information = providerHandler.getInformation();
        assertEquals(information.getName(), "Hatsune Miku");
    }
}
```
这里的关键是`createPact`和`runTest`这两个方法：
* `createPact`直接定义了契约交互的全部内容，比如Request的路径和参数，以及返回的Response的具体内容；
* `runTest`是执行测试的方法，其中`ProviderHandler`是Miku应用代码中的类，我们直接使用它来发送真正的Request，发给谁呢？发给`mockServer`，Pact会启动一个mockServer, 基于Java原生的HttpServer封装，用来代替真正的Provider应答`createPact`中定义好的响应内容，继而模拟了整个契约的内容；
* runTest中的断言可以用来保证我们编写的契约内容是符合Miku期望的，你可以把它理解为一种类似Consumer端的集成测试；

#### Junit Rule

`PactJunitRuleTest.java`

```java
public class PactJunitRuleTest {

    @Rule
    public PactProviderRuleMk2 mockProvider = new PactProviderRuleMk2("ExampleProvider",this);

    @Pact(consumer="JunitRuleConsumer")
    public RequestResponsePact createPact(PactDslWithProvider builder) {
        Map<String, String> headers = new HashMap<String, String>();
        headers.put("Content-Type", "application/json;charset=UTF-8");

        return builder
                .given("")
                .uponReceiving("Pact JVM example Pact interaction")
                .path("/information")
                .query("fullName=Miku")
                .method("GET")
                .willRespondWith()
                .headers(headers)
                .status(200)
                .body("{\n" +
                        "    \"salary\": 45000,\n" +
                        "    \"fullName\": \"Hatsune Miku\",\n" +
                        "    \"nationality\": \"Japan\",\n" +
                        "    \"contact\": {\n" +
                        "        \"Email\": \"hatsune.miku@ariman.com\",\n" +
                        "        \"Phone Number\": \"9090950\"\n" +
                        "    }\n" +
                        "}")
                .toPact();
    }

    @Test
    @PactVerification
    public void runTest() {
        ProviderHandler providerHandler = new ProviderHandler();
        providerHandler.setBackendURL(mockProvider.getUrl());
        Information information = providerHandler.getInformation();
        assertEquals(information.getName(), "Hatsune Miku");
    }
}
```
相较于基本的Junit写法，`PactProviderRuleMk2`能够让代码更加的简洁，它还可以自定义Mock Provider的address和port。如果像上面的代码一样省略address和port，则会默认使用127.0.0.1和随机端口。Junit Rule还提供了方法让你可以同时对多个Provider进行测试，以及让Mock Provider使用HTTPS进行交互。

>基于体力有限，本示例没有包含MultiProviders和HTTPS的例子，有需要的同学可以在PACT JVM官网上查询相关的用法......别，别打呀，俺承认，俺就是懒...还打...#$%^&*!@#$%^&...喂：110吗？俺要报警......

#### Junit DSL
`PactJunitDSLTest`

```java
public class PactJunitDSLTest {

    private void checkResult(PactVerificationResult result) {
        if (result instanceof PactVerificationResult.Error) {
            throw new RuntimeException(((PactVerificationResult.Error)result).getError());
        }
        assertEquals(PactVerificationResult.Ok.INSTANCE, result);
    }

    @Test
    public void testPact1() {
        Map<String, String> headers = new HashMap<String, String>();
        headers.put("Content-Type", "application/json;charset=UTF-8");

        RequestResponsePact pact = ConsumerPactBuilder
            .consumer("JunitDSLConsumer1")
            .hasPactWith("ExampleProvider")
            .given("")
            .uponReceiving("Query fullName is Miku")
                .path("/information")
                .query("fullName=Miku")
                .method("GET")
            .willRespondWith()
                .headers(headers)
                .status(200)
                .body("{\n" +
                        "    \"salary\": 45000,\n" +
                        "    \"fullName\": \"Hatsune Miku\",\n" +
                        "    \"nationality\": \"Japan\",\n" +
                        "    \"contact\": {\n" +
                        "        \"Email\": \"hatsune.miku@ariman.com\",\n" +
                        "        \"Phone Number\": \"9090950\"\n" +
                        "    }\n" +
                        "}")
            .toPact();

        MockProviderConfig config = MockProviderConfig.createDefault();
        PactVerificationResult result = runConsumerTest(pact, config, mockServer -> {
            ProviderHandler providerHandler = new ProviderHandler();
            providerHandler.setBackendURL(mockServer.getUrl(), "Miku");
            Information information = providerHandler.getInformation();
            assertEquals(information.getName(), "Hatsune Miku");
        });

        checkResult(result);
    }

    @Test
    public void testPact2() {
        Map<String, String> headers = new HashMap<String, String>();
        headers.put("Content-Type", "application/json;charset=UTF-8");

        RequestResponsePact pact = ConsumerPactBuilder
            .consumer("JunitDSLConsumer2")
            .hasPactWith("ExampleProvider")
            .given("")
            .uponReceiving("Query fullName is Nanoha")
                .path("/information")
                .query("fullName=Nanoha")
                .method("GET")
            .willRespondWith()
                .headers(headers)
                .status(200)
                .body("{\n" +
                        "    \"salary\": 80000,\n" +
                        "    \"fullName\": \"Takamachi Nanoha\",\n" +
                        "    \"nationality\": \"Japan\",\n" +
                        "    \"contact\": {\n" +
                        "        \"Email\": \"takamachi.nanoha@ariman.com\",\n" +
                        "        \"Phone Number\": \"9090940\"\n" +
                        "    }\n" +
                        "}")
                .toPact();

        MockProviderConfig config = MockProviderConfig.createDefault();
        PactVerificationResult result = runConsumerTest(pact, config, mockServer -> {
            ProviderHandler providerHandler = new ProviderHandler();
            providerHandler.setBackendURL(mockServer.getUrl(), "Nanoha");

            Information information = providerHandler.getInformation();
            assertEquals(information.getName(), "Takamachi Nanoha");
        });

        checkResult(result);
    }
}
```

基本的Junit和Junit Rule的写法只能在一个测试文件里面写一个Test Case，而使用Junit DSL则可以像上面的例子一样写多个Test Case。同样，你也可以通过`MockProviderConfig.createDefault()`配置Mock Server的address和port。上面的例子使用了默认配置。

`PactJunitDSLJsonBodyTest`

```java
public class PactJunitDSLJsonBodyTest {
    PactSpecVersion pactSpecVersion;

    private void checkResult(PactVerificationResult result) {
        if (result instanceof PactVerificationResult.Error) {
            throw new RuntimeException(((PactVerificationResult.Error)result).getError());
        }
        assertEquals(PactVerificationResult.Ok.INSTANCE, result);
    }

    @Test
    public void testWithPactDSLJsonBody() {
        Map<String, String> headers = new HashMap<String, String>();
        headers.put("Content-Type", "application/json;charset=UTF-8");

        DslPart body = new PactDslJsonBody()
                .numberType("salary", 45000)
                .stringType("fullName", "Hatsune Miku")
                .stringType("nationality", "Japan")
                .object("contact")
                .stringValue("Email", "hatsune.miku@ariman.com")
                .stringValue("Phone Number", "9090950")
                .closeObject();

        RequestResponsePact pact = ConsumerPactBuilder
            .consumer("JunitDSLJsonBodyConsumer")
            .hasPactWith("ExampleProvider")
            .given("")
            .uponReceiving("Query fullName is Miku")
                .path("/information")
                .query("fullName=Miku")
                .method("GET")
            .willRespondWith()
                .headers(headers)
                .status(200)
                .body(body)
            .toPact();

        MockProviderConfig config = MockProviderConfig.createDefault(this.pactSpecVersion.V3);
        PactVerificationResult result = runConsumerTest(pact, config, mockServer -> {
            ProviderHandler providerHandler = new ProviderHandler();
            providerHandler.setBackendURL(mockServer.getUrl());
            Information information = providerHandler.getInformation();
            assertEquals(information.getName(), "Hatsune Miku");
        });

        checkResult(result);
    }
    @Test
    public void testWithLambdaDSLJsonBody() {
        Map<String, String> headers = new HashMap<String, String>();
        headers.put("Content-Type", "application/json;charset=UTF-8");

        DslPart body = newJsonBody((root) -> {
            root.numberValue("salary", 45000);
            root.stringValue("fullName", "Hatsune Miku");
            root.stringValue("nationality", "Japan");
            root.object("contact", (contactObject) -> {
                contactObject.stringMatcher("Email", ".*@ariman.com", "hatsune.miku@ariman.com");
                contactObject.stringType("Phone Number", "9090950");
            });
        }).build();

        RequestResponsePact pact = ConsumerPactBuilder
            .consumer("JunitDSLLambdaJsonBodyConsumer")
            .hasPactWith("ExampleProvider")
            .given("")
            .uponReceiving("Query fullName is Miku")
                .path("/information")
                .query("fullName=Miku")
                .method("GET")
            .willRespondWith()
                .headers(headers)
                .status(200)
                .body(body)
            .toPact();

        MockProviderConfig config = MockProviderConfig.createDefault(this.pactSpecVersion.V3);
        PactVerificationResult result = runConsumerTest(pact, config, mockServer -> {
            ProviderHandler providerHandler = new ProviderHandler();
            providerHandler.setBackendURL(mockServer.getUrl());
            Information information = providerHandler.getInformation();
            assertEquals(information.getName(), "Hatsune Miku");
        });

        checkResult(result);
    }

}
```

当然，Junit DSL的强大之处绝不仅仅是让你多写几个Test Case， 通过使用PactDslJsonBody和Lambda DSL你可以更好的编写你的契约测试文件：
* 对契约中Response Body的内容，使用JsonBody代替简单的字符串，可以让你的代码易读性更好；
* JsonBody提供了强大的Check By Type和Check By Value的功能，让你可以控制对Provider的Response测试精度。比如，对于契约中的某个字段，你是要确保Provider的返回必须是具体某个数值（check by Value），还是只要数据类型相同就可以（check by type），比如都是String或者Int。你甚至可以直接使用正则表达式来做更加灵活的验证；
* 目前支持的匹配验证方法：

| method | description |
|--------|-------------|
| string, stringValue | Match a string value (using string equality) |
| number, numberValue | Match a number value (using Number.equals)\* |
| booleanValue | Match a boolean value (using equality) |
| stringType | Will match all Strings |
| numberType | Will match all numbers\* |
| integerType | Will match all numbers that are integers (both ints and longs)\* |
| decimalType | Will match all real numbers (floating point and decimal)\* |
| booleanType | Will match all boolean values (true and false) |
| stringMatcher | Will match strings using the provided regular expression |
| timestamp | Will match string containing timestamps. If a timestamp format is not given, will match an ISO timestamp format |
| date | Will match string containing dates. If a date format is not given, will match an ISO date format |
| time | Will match string containing times. If a time format is not given, will match an ISO time format |
| ipAddress | Will match string containing IP4 formatted address. |
| id | Will match all numbers by type |
| hexValue | Will match all hexadecimal encoded strings |
| uuid | Will match strings containing UUIDs |
| includesStr | Will match strings containing the provided string |
| equalsTo | Will match using equals |
| matchUrl | Defines a matcher for URLs, given the base URL path and a sequence of path fragments. The path fragments could be strings or regular expression matchers |

* 对于Array和Map这样的数据结构，DSL也有相应匹配验证方法，我这里就不罗列了，请参考[官网的介绍](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-consumer-junit#user-content-ensuring-all-items-in-a-list-match-an-example-220)；




###执行Miku端的测试
Test Case准备好后，我们就可以执行测试了。因为我们实际上是用的Junit的框架，所以和执行一般的单元测试是一样的：
```commandline
./gradlew :example-consumer-miku:clean test
```
成功执行后，你就可以在`Pacts\Miku`下面找到所有测试生成的契约文件。


###发布契约文件到Pact Broker
###Provider端的测试

## 相关的Gradle配置
## Provider与Nanoha间的契约测试
## 验证我们的测试





