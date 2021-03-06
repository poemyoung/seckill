# 单元测试本地化方案以及jmokit 的使用

[toc]

## 单元测试简介及方案探讨：

### 单元测试简介：

#### 怎么定义一个**单元**？

一个maven项目中，一般会有多个分层：

+ common  公共层： 这里主要提供一些工具包，包括对配置、redis 、消息中间件等的最底层访问。
+ repository / dao 数据库访问层：一般是以接口的形式定义数据库访的方法，以及一系列的实现类去实现
+ service 层： 业务逻辑主要处理的地方。
+ handler层：主要负责权限校验、参数校验等操作
+ controller层：如果有前端直接调用项目接口，那么就会有controller层，这一层主要用于接收请求，设置返回状态以及返回请求

那么一个单元怎么定义呢？取决于单元测试的方式，一般来说会有三种定义：

+ 大型测试：请求在handler构造，通过service、dao等多个模块的处理，最终返回到handler，然后校验响应结果是否符合预期。
+ 中型测试：使用mock（模拟）的方式mock掉一些类和一些方法的行为，使得我们可以着重关注我们所要测试的某一些模块。比如，mock掉 dao 层（使得dao层返回我们想要的数据，即假设dao层行为正确），这样我们便可以只关注业务逻辑，不必考虑数据访问是否正确
+ 小型测试：只针对一个类或者一个方法进行测试，所有不在该类中的方法全部进行mock,保证某一个类或某个方法的行为正确

#### UT (Unit Test)本地化

“ UT本应就是Local属性的。UT不应该依赖外部环境。”

对于一个项目的开发，可能会经历多个不同的环境：比如 dev开发环境、fat / fws测试环境、uat对比测试、lpt压测环境等等。而每一个环境，都会对应这个环境的数据库、配置文件等等。如果我们希望单元测试在任何一个环境都不失败，那么基本只有两种方式：

+ 保持每个环境数据库、配置文件等等相同。缺点当然显而易见：难以维护，一个数据的变更需要设置到每个环境
+ 使用模拟的方式，mock掉任何依赖环境的部分进行单元测试，也就是本地化，单元测试与环境无关。

### 单元测试本地化方案探讨：

#### 单元测试粒度：

对于一个足够熟悉业务逻辑、足够了解整个项目，甚至整个项目就是他自己写的的人来说，单元测试的粒度是无关紧要的。因为可能在测试之前就知道了会接触哪些数据库，会接触什么配置以及会处理怎样的输入，得到哪些结果。

但是对于一个不熟悉的新手来说，写单元测试尽量采用小型测试，即对某个类或者某个方法进行单独测试，mock其它接触类的行为，理由如下：

+ 简单：当你只关注于某一个类的代码逻辑，只关注方法的输入参数和最后的返回值，而不用关注整个请求的流转时，所需要读的代码会大大减少。
+ 正确性：当我们模拟其它类对某个类进行测试时，我们一定能够保证这个类的正确性。而被模拟的类的正确性，应该由被模拟类的测试方法来检验。举个例子，我们在对handler进行测试时，mock掉service层，即便service层某些逻辑仍然没有依赖外部环境。因为在模拟service的行为对handler进行测试时，我们能保证handler类行为的正确性。对于service行为是否正确，应该由service的测试类进行测试并反馈。
+ 覆盖率高：很显然的一件事是，当测试的粒度越小，代码的覆盖率就会越高。对于跨越多个模块的测试，很难通过构造不同请求的方式走遍所有branch (分支，包括 if-else、switch、try-catch 等等)。然而对于一个方法，通过构造不同参数的方式走完所有分支是完全可行且简单的。

#### 细粒度单元测试实战

##### 依赖引入：

这里主要引入junit 和 jmokit,推荐官方引入方法：

[link]:http://jmockit.cn/showChannel.htm?channel=2

##### spring 项目

区别于一般项目，spring 项目中会使用许多@Autowired方法进行属性的注入。这给我们在单元测试时提供了许多方便。

我们知道 @Autowired 是通过读取配置的方式进行Bean的自动注入，所以，首先我们需要一个基础类引入配置：

```java
public class BaseTest {
    @Before
    public void setUp(){
        
    }
    @Test
    public void doTest(){
        
    }
    @After
    public void shutDown(){
        
    }
}
```



