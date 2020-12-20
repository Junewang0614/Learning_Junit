## Junit4学习过程笔记02

### 中英对照

| 英                          | 中                 |
| :-------------------------- | :----------------- |
| JUnitCommandLineParseResult | 命令解析结果       |
| notifier                    | 通知者             |
| listener                    | 监听者             |
| Parser                      | 解析               |
| filter                      | 过滤器             |
| default                     | 更倾向于默认的意思 |
| Suite                       | 一套               |
| internal                    | 内部的             |
| deprecated                  | 弃用               |
|                             |                    |
|                             |                    |
|                             |                    |
|                             |                    |
|                             |                    |
|                             |                    |
|                             |                    |
|                             |                    |
|                             |                    |
|                             |                    |

### 可能存在的设计模式

* 建造者模式！！！
* 好像有模板模式（？）

### Runner

由main方法开始运行的测试，通过runner.run(notifier);运行测试，也分析过这个runner是由一个抽象的request通过实现其抽象方法getRunner，返回了Suite的runner。

现在对这个Runner进一步分析

#### 主要过程

1. Suite 是由builder构造器创建，computer是指挥器！！！

2. builder创建过程🈶:

   ```java
   AllDefaultPossibilitiesBuilder builder = new AllDefaultPossibilitiesBuilder();
   //构造函数没有参数
   ```

   创建所有默认可能的构造器

3. AllDefaultPossibilitiesBuilder.java

   ```java
   public AllDefaultPossibilitiesBuilder() {
           canUseSuiteMethod = true;
       }
   //设置了这个类的变量为true
   //这个变量是final修饰的基础数据类型变量
   //所以构造函数中必须确定它的值，且确定后无法更改
   //换句话说，这个构造函数除了必须做的事情以外，啥也没做
   ```

   

4. 继续看computer的指挥过程（computer.java)

   ```java
   public Runner getSuite(final RunnerBuilder builder,
               Class<?>[] classes) throws InitializationError {
           return new Suite(new RunnerBuilder() {
               @Override
               public Runner runnerForClass(Class<?> testClass) throws Throwable {
                   return getRunner(builder, testClass);
               }
           }, classes) {
               @Override
               protected String getName() {
                   /*
                    * #1320 The generated suite is not based on a real class so
                    * only a 'null' description can be generated from it. This name
                    * will be overridden here.
                    */
                   return "classes";
               }
           };
       }
   ```

   getSuit方法中直接返回一个new的Suite，调用的构造函数是Suite(RunnerBuilder)

5. 这个runnerBuilder是直接定义抽象类，重写其抽象方法，转到调用computer的getRunner函数

   ```java
   protected Runner getRunner(RunnerBuilder builder, Class<?> testClass) throws Throwable {
           return builder.runnerForClass(testClass);
       }
   //这里的builder不再是抽象的builder了
   //是当时导入的，没做什么的AllDefaultPossibilitiesBuilder
   ```

   **结论一：这个抽象的builder重写自身抽象方法runnerForClass，调用的的是导入的AllDefaultPossibilitiesBuilder的runnerForClass方法**

6. 回到Suite的构造过程，调用的构造函数有

   ```java
   public Suite(RunnerBuilder builder, Class<?>[] classes) throws InitializationError {
           this(null, builder.runners(null, classes));
       }
   //调用另一个构造方法
   protected Suite(Class<?> klass, List<Runner> runners) throws InitializationError {
           super(klass);
       	//将参数中的List返回一个不可修改的List.
       	//参数中的list就是
           this.runners = Collections.unmodifiableList(runners);
       }
   //其实是这个构造方法在工作
   //再深挖发现,返回的runners，就是将导入的一定处理后，放入runners列表中
   //处理是：safeRunnerForClass
   //这个处理是：将所有的class 构造相应的runner运行器！！！
   //此时Suite中必须指定的变量也指定了
   ```

7. 对classes追根溯源：

   * Suite的classes是computer 调用getSuite导入的

   * computer的Suite来源于Request

   * Request的classes来源于命令解析类的createRequest

     是将classes变为数组

   * 命令解析类中有一个final list变量为classes，应该是解析结果得到的

   **结论二：创造的suite的runner，其final变量中runners 就是命令解析得到的classes，为每一个class构造一个runner，形成的list**

8. 