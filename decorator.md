# decorator pattern in Junit4

statement部分的装饰器模型为简化的透明装饰模型。简化是在于没有定义ConcreteComponent角色类，透明是在于所有ConcreteDecorator类中的“新增方法”实际上是为了辅助实现ConcreteComponent类的功能，对于客户端来讲，无需知道新增方法的存在，也没有调用它的必要，达到一致使用的效果。

## ConcreteComponent角色

```java
package org.junit.runners.model;

/**
 * Represents one or more actions to be taken at runtime in the course
 * of running a JUnit test suite.
 *
 * @since 4.5
 */
public abstract class Statement {
    /**
     * Run the action, throwing a {@code Throwable} if anything goes wrong.
     */
    public abstract void evaluate() throws Throwable;
}
```

## Decorator角色

此角色在statement中缺失，以ConcreteDecorator的RunAfter为例，在RunAfter类中定义了成员变量，则无需创建一个Decorator角色，然后继承它。在一般的装饰器模式中定义Decorator角色，可以在同时用Decorator声明不同的ConcreteDerator类，一定程度简化客户端代码（个人看法，简化程度很弱且增减冗余，很没必要）。在Junit4"客户端"调用情况来看，不涉及类变量的声明的问题，而是直接new后返回，所以缺失这个角色更合理。

```java
public class RunAfters extends Statement {
    private final Statement next;

    private final Object target;

    private final List<FrameworkMethod> afters;

    public RunAfters(Statement next, List<FrameworkMethod> afters, Object target) {
        this.next = next;
        this.afters = afters;
        this.target = target;
    }
}
```

## ConcreteDecorator 角色

在此案例中有多个 ConcreteDecorator 角色，避免代码篇幅过多，仅显示部分。junit装饰器模式的完整的具体抽象类见UML图。

```java
package org.junit.internal.runners.statements;

import java.util.ArrayList;
import java.util.List;

import org.junit.runners.model.FrameworkMethod;
import org.junit.runners.model.MultipleFailureException;
import org.junit.runners.model.Statement;

public class RunAfters extends Statement {
    private final Statement next;

    private final Object target;

    private final List<FrameworkMethod> afters;

    public RunAfters(Statement next, List<FrameworkMethod> afters, Object target) {
        this.next = next;
        this.afters = afters;
        this.target = target;
    }

    @Override
    public void evaluate() throws Throwable {
        List<Throwable> errors = new ArrayList<Throwable>();
        try {
            next.evaluate();
        } catch (Throwable e) {
            errors.add(e);
        } finally {
            for (FrameworkMethod each : afters) {
                try {
                    invokeMethod(each);
                } catch (Throwable e) {
                    errors.add(e);
                }
            }
        }
        MultipleFailureException.assertEmpty(errors);
    }

    /**
     * @since 4.13
     */
    protected void invokeMethod(FrameworkMethod method) throws Throwable {
        method.invokeExplosively(target);
    }
}
```

```java
package org.junit.internal.runners.statements;

import org.junit.runners.model.Statement;

public class Fail extends Statement {
    private final Throwable error;

    public Fail(Throwable e) {
        error = e;
    }

    @Override
    public void evaluate() throws Throwable {
        throw error;
    }
}
```

```java
package org.junit.internal.runners.statements;

import org.junit.internal.AssumptionViolatedException;
import org.junit.runners.model.Statement;

public class ExpectException extends Statement {
    private final Statement next;
    private final Class<? extends Throwable> expected;

    public ExpectException(Statement next, Class<? extends Throwable> expected) {
        this.next = next;
        this.expected = expected;
    }

    @Override
    public void evaluate() throws Exception {
        boolean complete = false;
        try {
            next.evaluate();
            complete = true;
        } catch (AssumptionViolatedException e) {
            if (!expected.isAssignableFrom(e.getClass())) {
                throw e;
            }
        } catch (Throwable e) {
            if (!expected.isAssignableFrom(e.getClass())) {
                String message = "Unexpected exception, expected<"
                        + expected.getName() + "> but was<"
                        + e.getClass().getName() + ">";
                throw new Exception(message, e);
            }
        }
        if (complete) {
            throw new AssertionError("Expected exception: "
                    + expected.getName());

    }
}
```

## UML图

![pic](decorator.PNG)

## 应用装饰器模式的原因

对于不同的testClass，在具体的执行上会有区别，为了保持调用的接口一致，即都用可以使用 `statement.evaluate()`执行测试，故使用装饰器模式对不同的testClass对应的不同的statement封装出成类，并在类内部定义 `evaluate()` 方法。
