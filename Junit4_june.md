## Junit4学习过程笔记

**------- 12.12更新-------------**

### 中英对照

| 英                          | 中                 |
| :-------------------------- | :----------------- |
| JUnitCommandLineParseResult | 命令解析结果       |
| notifier                    | 通知者             |
| listener                    | 监听者             |
| Parser                      | 解析               |
| filter                      | 过滤器             |
| default                     | 更倾向于默认的意思 |
|                             |                    |
|                             |                    |
|                             |                    |

### 可能存在的设计模式

* 装饰器模式
* 建造者模式
* 工厂模式
* 组合模式



### JUnitCore

#### 主要过程

1. main()：调用runMain()函数

2. runMain()函数：

   解析junit命令行（类：JUnitCommandLineParseResult）

   创建监听者listener（类：RunListener）

   调用运行方法：run(request)

   request是命令解析结果创建

3. run(Request request)

   调用run(Runner)的方法，Runner_1是request创建的

4. run(Runner runner)

   定义结果集，结果集创建监听者（类：Result,RunListener）

   notifier通知开始，runner_1运行，notifier通知结束

5. runner_1.run(notifier)

   (runner.java中)

   这只是一个抽象方法，要看具体的runner是什么

6. 回溯到寻找具体的runner_1--->runner_1 由request创建（3）

   使用的方法是request.getRunner()

7. request.getRunner()(request类中)

   这是一个抽象的方法，要看具体Request的什么子类怎么实现了这个方法

8. 回溯寻找具体的Request---->request由jUnitCommandLineParseResult，命令行解析结果创建

9. jUnitCommandLineParseResult.createRequest(defaultComputer())

   (jUnitCommandLineParseResult.java中）

   判断解析结果有没有错误，错误被保存在解析结果类的parserErrors列表中

   * 如果有错误：导出错误
   * 如果没有错误：调用request.classes创建request，并利用applyFilterSpecs（request）过滤一下

10. Request classes(Computer computer, Class<?>... classes)

    (request.java中)

    * 创建所有默认可能的runnerBuilder(:star:大概率构造者模式)
    * 创建Suite的runner，suite是runner的一种（通过computer创建）
    * 调用Request中的runner方法，创建request

11.  Request runner(Runner)

    (Request.java中)

    这是一个static方法

    直接new一个Request，因为Request是抽象类，重写实现了getrunner方法，获得runner

    **结论一：7中具体的Request不是具体的，是new了抽象类，但是把抽象方法实现了**

    **结论二：因为request实现了getRunner方法，runner_1就是方法中返回的Suite的runner**

    （对应6）

12. 重写的getRunner(Request.runner 方法中)

    return了 Request.runner 方法中导入的runner

    追根溯源，就是那个Suite的Runner（对应10）

    **结论三：创建request的runner，是在Request classes方法中，创建的Suite**

    > **综上，在main中runner.run的runner是一个Suite runner，创建这个运行的runner的request，就是抽象类Request，在new的同时实现了它的抽象方法定义**
    >
    > 

13. 返回到request的创建过程后，9

    调用方法applyFilterSpecs（request），这个过程主要是将request外加一层装饰器

    （:star:装饰器模式，有条件：）

14.  private Request applyFilterSpecs(Request request)

    （jUnitCommandLineParseResult.java中）

    * 遍历filterSpecs列表中的所有过滤说明
    * 每一条过滤说明--》构造filter过滤器，使用FilterFactories(:star:工厂模式)
    * 更新request 为 request.filterWith（filter)，实现对每条过滤说明的实现
    * 返回过滤完成的request

#### Runner

* 抽象类
* 实现接口describable
* 两个抽象方法
  * getDescription
  * run(notifier)

#### JUnitCommandLineParseResult

* 一个非常固定的类
* parserErrors：解析错误的list列表，final 型。
* classes：final型，猜测是被测试类
* filterSpecs：过滤器说明list列表，final型



#### Request

* 抽象类

* 属性：无

* 方法：

  * 抽象：

    ```java
    public abstract Runner getRunner();
    ```

  * 静态：

    ```java
    1.public static Request runner(final Runner runner);
    2.
    ```

    

