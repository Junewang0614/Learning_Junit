## 观察者模式

![模式模型（与课本有一点出入）](https://www.runoob.com/wp-content/uploads/2014/08/observer_pattern_uml_diagram.jpg)
模式模型（与课本有一点出入）
描述对象一对多的关系
Runner在执行TestCase过程中的各个阶段都会通知RunNotifier，RunNotifier是负责listener的管理者角色;  
即**RunNotifier**是观察者模式中对应的 Subject  
**RunListener** 对应的 Observer

```
RunNotifier:
- listeners: ArrayList<RunListener>
-----------------
+ addListener
- removeListener
- notify()的实现：见下文
```

观察者模式中定义了对象的一种一对多的关系，通常而言会在Subject定义notify()方法用于通知各个观察者对象并调用对应的更新方法。
而RunNotifier通过定义内部类RunNotifier抽出了公共的逻辑部分：遍历listener队列，调用notifyListener方法，从而实现这一功能
```
private abstract class SafeNotifier {
    private final List<RunListener> currentListeners;

    SafeNotifier() {
        this(listeners);
    }

    SafeNotifier(List<RunListener> currentListeners) {
        this.currentListeners = currentListeners;
    }

    void run() { 
        int capacity = currentListeners.size();
        List<RunListener> safeListeners = new ArrayList<RunListener>(capacity);
        List<Failure> failures = new ArrayList<Failure>(capacity);
        for (RunListener listener : currentListeners) { //
            try {
                notifyListener(listener); //
                safeListeners.add(listener);
            } catch (Exception e) {
                failures.add(new Failure(Description.TEST_MECHANISM, e));
            }
        }
        fireTestFailures(safeListeners, failures);
    }

    abstract protected void notifyListener(RunListener each) throws Exception;
}
```
[UML草稿图](...)

ConcreteObserver:
用户可以扩展Observer类，进行编写自己的测试监听器

**Example:**
用户定义了一个 Cowbell 的类，当junit测试失败时Cow会开始吵闹。
```
public class RingingListener extends RunListener {
    public void testFailure(Failure failure) { //具体的update行为
        Cowbell.ring();
   }
```
**RunListener**: Observer    
**RingingListener**: ConcreteObserver
