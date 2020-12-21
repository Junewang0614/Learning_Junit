## 工厂方法模式

Runner 负责运行测试用例  
RunnerBuilder 负责创建Runner   
用例启动，Client在创建Request后会调用RunnerBuilder（工厂方法的抽象类）来创建Runner，默认的实现是AllDefaultPosibilitiesBuilder，根据不同的测试类定义（@RunWith的信息）返回Runner的具体实现。

**RunnerBuilder:抽象工厂**
```
public abstract class RunnerBuilder {
    public abstract Runner runnerForClass(Class<?> testClass) throws Throwable;
}
```
**AllDefaultPossibilitiesBuilder：具体工厂**
```
public class AllDefaultPossibilitiesBuilder extends RunnerBuilder {

    @Override
    public Runner runnerForClass(Class<?> testClass) throws Throwable {
        List<RunnerBuilder> builders = Arrays.asList(
                ignoredBuilder(),
                annotatedBuilder(),
                suiteMethodBuilder(),
                junit3Builder(),
                junit4Builder());

        for (RunnerBuilder each : builders) {
            Runner runner = each.safeRunnerForClass(testClass);
            if (runner != null) {
                return runner;
            }
        }
        return null;
    }
}
```

抽象产品：Runner
具体产品：默认实现 调用JUnit4Builder类返回一个Junit4对象 //待确认
```
public class JUnit4Builder extends RunnerBuilder {
    @Override
    public Runner runnerForClass(Class<?> testClass) throws Throwable {
        return new JUnit4(testClass);
    }
}
```