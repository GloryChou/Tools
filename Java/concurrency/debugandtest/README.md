# 多线程程序的测试

### 多线程测试的不确定性
多线程程序的BUG往往具有不确定性，例如可见性问题、死锁、饥饿等问题都不是必然出现的。造成不确定的常见因素包括：
+ 程序的执行会交错顺序
+ 并发程度的增加可能造成锁泄漏（一个线程在持有一个显式锁的情况下最多可以重复2,147,483,647次申请该锁，超过会抛出Error）
+ JIT编译优化造成指令重排序
+ 硬件平台本身架构导致的内存重排序



### 可测试性
提高多线程程序的可测试性可以从以下几个方面入手：

##### 抽象与实现分离
尽量面向接口编程

##### 数据与数据来源分离
例如：由文件输入的数据不要直接使用File或者FileInputStream读取，而是使用InputStream来表示输入源，这样就可以脱离数据的来源进行测试

##### 依赖注入
产生外部对象关联的时候，采用Spring IOC注入对象，或者用Mockito来Mock一个对象

##### 关注点分离
核心功能点仍是按照单线程方式进行单独的单元测试，以确保核心逻辑执行正常。在此基础上，核心功能所涉及到的对象都无须创建具体实例，可以通过创建匿名内部类或匿名内部子类的方式避开核心功能测试，这样可以让我们更加关注对多线程本身的测试

##### 使工作者线程数可以配置
用以排查由并发度造成的问题



### 多线程程序的单元测试：JCStress
JCStress是OpenJDK下的一个试验性项目，它可以用来编写多线程程序的单元测试。JCStress提供了一组注解和工具类以简化测试代码的编写：

**JCStress常用注解与工具类**
注解/类 | 解    释
:- | :-:
@JCStressTest | 代表被注释的类是一个JCStress测试用例
@State | 代表被注释的类包含共享状态
@Actor | 代表被注释的方法为并发操作
@Arbiter | 相当于一种特殊的@Actor，其注释的方法会在同一个测试用例内所有@Actor注释的并发操作结束后才被执行
@Outcome | 代表被注释的测试用例的可能输出结构及其是否可接受
IntResult1、IntResult2、LongResult1、LongResult2等 | 代表测试的结构

一个简单的示例：
``` java
@JCStressTest
@State
@Description("测试Counter的线程安全性")
@Outcome(id = "[2]", expect = Expect.ACCEPTABLE, desc = "OK")
@Outcome(id = "[1]", expect = Expect.FORBIDDEN, desc = "丢失更新或者读脏数据")
public class CounterTestV2 {
    final Counter counter = new Counter();

    @Actor
    public void actor1() {
        counter.increment();
    }

    @Actor
    public void actor2() {
        counter.increment();
    }

    @Arbiter
    public void actor3(LongResult1 r) {
        r.r1 = counter.value();
    }
}
```