# composition design pattern in Junit

## composite 模式中的component 角色
```java
package junit.framework;

/**
 * A <em>Test</em> can be run and collect its results.
 *
 * @see TestResult
 */
public interface Test {
    /**
     * Counts the number of test cases that will be run by this test.
     */
    public abstract int countTestCases();

    /**
     * Runs a test and collects its result in a TestResult instance.
     */
    public abstract void run(TestResult result);
}
```
## composite 模式的leaf角色

```java
package junit.framework;
public abstract class TestCase extends Assert implements Test {
    /**
     * Runs the test case and collects the results in TestResult.
     */
    public void run(TestResult result) {
        result.run(this);
    }
}
```

## composite 模式的Composite participant

```java
package junit.framework;
public class TestSuite implements Test {
    private Vector<Test> fTests = new Vector<Test>(10);
    public void addTest(Test test) {
        fTests.add(test);
    }
    /**
     * Runs the tests and collects their result in a TestResult.
     */
    public void run(TestResult result) {
        for (Test each : fTests) {
            if (result.shouldStop()) {
                break;
            }
            runTest(each, result);
        }
    }
}
```
## UML图见同级目录文件composite.PNG