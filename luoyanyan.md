## 命令模式

---
## Statement


Statement是测试过程中的各种大小步骤，可以在测试过程中进行插入，当runner发出各种statement时，就表示正在完成整个测试的运行。

org.junit.internal.runners.statement包中定义了Statement的子类（具体命令），针对各种标注，像@Test,@Berfore,@After,@BeforeClass,@AfterClass和各种测试场景，Statement的子类封装一个或者多个动作。

抽象命令类：Statement

具体命令类：
1. Fail
1. InvokeMethod
1. ExceptException
1. RunAfter
1. RunBefore
1. FailonTimeout
1. RunRules

调用者：
各种Runner,默认为BlockJUnit4ClassRunner为测试执行器，发出Statement进行测试。
