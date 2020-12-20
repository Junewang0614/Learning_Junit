### 模板方法模式

* 比较明显的有：

  抽象类：parentRunner,里面定义了很多Runner进行的行为，ParentRunner类定义了Statement的构造和执行流程。模板方法：

  ```java
      protected Statement classBlock(final RunNotifier notifier) {
              Statement statement = childrenInvoker(notifier);
              if (!areAllChildrenIgnored()) {
                 statement = withBeforeClasses(statement);
                  statement = withAfterClasses(statement);
                  statement = withClassRules(statement);
                 statement = withInterruptIsolation(statement);
              }
             return statement;
         }
  ```

  这是运行器流程

  对应操作：

  ```java
  protected Statement withBeforeClasses(Statement statement) {
  238            List<FrameworkMethod> befores = testClass
  239                    .getAnnotatedMethods(BeforeClass.class);
  240            return befores.isEmpty() ? statement :
  241                    new RunBefores(statement, befores, null);
  242        }
  
  251        protected Statement withAfterClasses(Statement statement) {
  252            List<FrameworkMethod> afters = testClass
  253                    .getAnnotatedMethods(AfterClass.class);
  254            return afters.isEmpty() ? statement :
  255                    new RunAfters(statement, afters, null);
  256        }
  
  
  267        private Statement withClassRules(Statement statement) {
  268            List<TestRule> classRules = classRules();
  269            return classRules.isEmpty() ? statement :
  270                    new RunRules(statement, classRules, getDescription());
  271        }
  272    
      
  
  301        protected final Statement withInterruptIsolation(final Statement statement) {
  302            return new Statement() {
  303                @Override
  304                public void evaluate() throws Throwable {
  305                    try {
  306                        statement.evaluate();
  307                    } finally {
  308                        Thread.interrupted(); // clearing thread interrupted status for isolation
  309                    }
  310                }
  311            };
  312        }
  ```

  

### 建造者模式

* ![](images\builder.png)

其中RunnerBuilder是**抽象建造者类**，除了AllDefaultPossibilitiesBuilder类以外，其他的都是**具体构造者**，通过Builder返回相应的Runner运行器。Runner则是product类，产品类。指挥类就是**Computer**类

```java
protected Runner getRunner(RunnerBuilder builder, Class<?> testClass) throws Throwable {
        return builder.runnerForClass(testClassss);
}
```

builder.runnerForClass这是所有builder方法中用来构造Runner运行器的，computer通过getRunner方法指挥Builder构造了相应的Runner，因此computer是指挥类。



