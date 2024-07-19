slf4j全称是`Simple Logging Facade for Java`。facade是一种设计模式。

slf4j 是一个抽象程度更高的日志组件，本身并不提供实际的日志功能。实际的日志功能是通过log4j等日志组件实现，而使用者只需要关心 slf4j 给出的API。

# 结合Log4j的使用示例 - 示例1

使用 IDEA 创建 gradle项目，我用的gradle版本是4.1。

在 build.gradle 的 dependencies 中增加依赖：

```groovy
// https://mvnrepository.com/artifact/org.slf4j/slf4j-api
compile group: 'org.slf4j', name: 'slf4j-api', version: '1.7.25'

// https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12
compile group: 'org.slf4j', name: 'slf4j-log4j12', version: '1.7.25'

// https://mvnrepository.com/artifact/log4j/log4j
compile group: 'log4j', name: 'log4j', version: '1.2.17'
```

- `slf4j-api`提供了slf4j的抽象接口，我们作为使用者，只需要关心它提供的API就行。
- `slf4j-log4j12`是slf4j与log4j的桥接组件。
- `log4j`是我们常见的log4j日志组件。

在`src/main/java`目录下创建类`Example01.java`，内容如下：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Example01 {

    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(Example01.class);
        logger.info("Hello {}", "World");
    }
}
```

`logger.info("Hello {}", "World");`的作用是打出info级别的日志内容`Hello World`。是的，我们不必写成`logger.info("Hello %s", "World");` 或者`logger.info("Hello " + "World");`，一切变得不用想太多了。

运行 Example01，输出：

```plain
log4j:WARN No appenders could be found for logger (Example01).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

是的，没有输出我们想要的内容，反而报了警告。这是因为我们没有配置log4j。

在`src/main/resources`目录中创建文件`log4j.properties`，内容如下：

```plain
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

再次运行 Example01，将会输出：

```plain
 INFO [main] - Hello World
```

# 结合Log4j的使用示例 - 示例2

log4j有一个MDC(Mapped Diagnostic Context)、NDC(Nested Diagnostic Context)机制，这个机制用于打印一些线程粒度上下文信息，但又不需要每次`logger.info`时把上下文信息填进去。

slf4j提供了对MDC的封装。

有什么使用场景呢？比如我们写了一个web服务器，客户端的每一次请求一般都是放在一个线程中处理的。如果我们想在日志中加入客户端的IP信息，就可以用到MDC。

首先修改`log4j.properties`中的`log4j.appender.stdout.layout.ConversionPattern`，在值中加入`%X{IP}`：

```plain
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] %X{IP} - %m%n
```

创建新的Java类Example02：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

public class Example02 {

    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(Example01.class);
        MDC.put("IP", "123.123.123.123");
        logger.info("Hello {}", "World");
    }
}

```

运行结果：

```plain
 INFO [main] 123.123.123.123 - Hello World
```

Example01 也能正常运行，只不过没有IP信息而已。

# 结合Log4j的使用示例 - 示例3

slf4j自带了一个很好用的formatter，`{}`会被替换成对应的变量值。那么，如何转义`{}`呢 ？

创建新的Java类Example03：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

public class Example03 {

    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(Example03.class);

        MDC.put("IP", "123.123.123.123");
        logger.info("Hello {}", "World");
        logger.info("Hello {} {}", "World");
        logger.info("Hello {} \\{}", "World");
        logger.info("Hello {} \\{} {}", "World");
        logger.info("Hello {} \\{} {}", "World", "HI");
    }
}
```

运行后输出：

```plain
 INFO [main] 123.123.123.123 - Hello World
 INFO [main] 123.123.123.123 - Hello World {}
 INFO [main] 123.123.123.123 - Hello World \{}
 INFO [main] 123.123.123.123 - Hello World \{} {}
 INFO [main] 123.123.123.123 - Hello World {} HI
```

我们把info的第一个参数称为format字符串，剩下的参数叫做参数列表。 可以总结下：

1. 如果format字符串中的`{}`，在参数列表找不到对应的参数。那么`{}`会被打印出来。
2. 如果format字符串中`\\{}`在两个`{}`之间，如果这两个`{}`能在参数列表中找到对应的参数，则`\\{}`会被输出为`{}`。

# 结合slf4j-simple的使用示例

`slf4j-simple`是slf4j系列自带的一个日志实现，可以不用配置，默认会把日志打印到终端。

使用 IDEA 新创建一个 gradle 项目。 `build.gradle`中添加依赖：

```groovy
// https://mvnrepository.com/artifact/org.slf4j/slf4j-api
compile group: 'org.slf4j', name: 'slf4j-api', version: '1.7.25'
// https://mvnrepository.com/artifact/org.slf4j/slf4j-simple
compile group: 'org.slf4j', name: 'slf4j-simple', version: '1.7.25'
```

编写类 Example01，内容如下：

```plain
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Example01 {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(Example01.class);
        logger.info("Hello {}", "World");
    }
}
```

运行后输出：

```plain
[main] INFO Example01 - Hello World
```

如果要配置`slf4j-simple`，可以参考下这篇文章： [How to configure slf4j-simple](https://stackoverflow.com/questions/14544991/how-to-configure-slf4j-simple)。

# 剖析：slf4j如何找到log4j？

我们一点点翻代码。

首先：

```plain
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Example01 {

    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(Example01.class);
        logger.info("Hello {}", "World");
    }
}

```

看`LoggerFactory.getLogger`的主要实现：

```plain
package org.slf4j;
public final class LoggerFactory {
    // ... 省略
    
    public static Logger getLogger(Class<?> clazz) {
        Logger logger = getLogger(clazz.getName());
		// ... 省略
        return logger;
    }
    
    public static Logger getLogger(String name) {
        ILoggerFactory iLoggerFactory = getILoggerFactory();
        return iLoggerFactory.getLogger(name);
    }
    
   // ... 省略
}
```

getLoggerFactory 是如何实现的？

```plain
public final class LoggerFactory {
    // ... 省略
    public static ILoggerFactory getILoggerFactory() {
        // ... 省略部分代码
        performInitialization();
        return StaticLoggerBinder.getSingleton().getLoggerFactory();
    }
    // ... 省略
}
```

performInitialization()的主要特性是调用 bind() 方法。

我们直接看bind()的实现：

```plain
public final class LoggerFactory {
    // ... 省略
    private final static void bind() {
        // 依然省略了很多代码
        try {
            Set<URL> staticLoggerBinderPathSet = null;
            // skip check under android, see also
            // http://jira.qos.ch/browse/SLF4J-328
            if (!isAndroid()) {
                staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
                reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
            }
            // the next line does the binding
            StaticLoggerBinder.getSingleton();
        } catch (Exception e) {
            failedBinding(e);
            throw new IllegalStateException("Unexpected initialization failure", e);
        }
    }
    // ... 省略
}
```

`findPossibleStaticLoggerBinderPathSet()`是去找当前classpath下所有的`org/slf4j/impl/StaticLoggerBinder.class`。如果有多个，则通过`reportMultipleBindingAmbiguity`发出警告。

> 是的，classpath下（包括自己的代码）有完全路径重名的类是可以接受的，如果我们代码里要获取这种类的实例，或者已经import进来了，实际使用的类是找到的第一个符合条件的类。所以有时会遇到坑，比如一个项目的jar包的低版本和高本版都在classpath中，我们希望用高版本中特性，但是程序表现一直不符合我们的预期。这时，可能是因为程序实际用的是低版本特性。这种情况下，需要分析依赖关系，禁止掉重复的依赖。[防痴呆设计](http://javatar.iteye.com/blog/804187)提供了一个简单但很有意思的思路，来防止自己编写的代码库出现这种问题。 当然，一个jar中不会出现完全路径重名的类。

然后，最重要的代码来了：

```plain
StaticLoggerBinder.getSingleton();
```

我们之前介绍过：

- `slf4j-api`提供了slf4j的抽象接口，我们作为使用者，只需要关心它提供的API就行。
- `slf4j-log4j12`是slf4j与log4j的桥接组件。
- `log4j`是我们常见的log4j日志组件。

我们刚才看到的代码都是`slf4j-api`中的，但是`StaticLoggerBinder`这个类是 `slf4j-log4j12`中的。也就是说，每一个slf4j和实际日志组件的桥接组件中都有一个`org.slf4j.impl.StaticLoggerBinder` 类。

> 补充： `slf4j-api`的源码中是有`org.slf4j.impl.StaticLoggerBinder`类的，不过没被打包到jar中。它的pom.xml中的打包配置，有一个删除操作：`<delete dir="target/classes/org/slf4j/impl"/>`。

**下面，我们进入`slf4j-log4j12`探索。**

`StaticLoggerBinder.getSingleton().getLoggerFactory()`返回的是什么呢？

是`Log4jLoggerFactory`。

我们直接去看`Log4jLoggerFactory`的`getLogger`方法：

```plain
    public Logger getLogger(String name) {
        // `loggerMap`是缓存logger实例。
        Logger slf4jLogger = loggerMap.get(name);
        if (slf4jLogger != null) {
            return slf4jLogger;
        } else {
            org.apache.log4j.Logger log4jLogger;
            if (name.equalsIgnoreCase(Logger.ROOT_LOGGER_NAME))
                log4jLogger = LogManager.getRootLogger();
            else
                log4jLogger = LogManager.getLogger(name);

            Logger newInstance = new Log4jLoggerAdapter(log4jLogger);
            Logger oldInstance = loggerMap.putIfAbsent(name, newInstance);
            return oldInstance == null ? newInstance : oldInstance;
        }
    }
```

是的，getLogger并没有直接返回log4j的logger，因为不符合log4j的logger没有继承自`org.slf4j.Logger`，所以用`Log4jLoggerAdapter`进行了封装。

> `Adapter`是一种设计模式。

# 剖析：slf4j的 formatter 原理

我们看下`logger.info("Hello {}", "World");`中的format机制是如何实现的，即`{}`是如何被替换的。

`slf4j-log4j12`中`Log4jLoggerAdapter`类info方法内容如下：

```java
public void info(String format, Object... argArray) {
    if (logger.isInfoEnabled()) {
        FormattingTuple ft = MessageFormatter.arrayFormat(format, argArray);
        logger.log(FQCN, Level.INFO, ft.getMessage(), ft.getThrowable());
    }
}
```

`MessageFormatter`来自`slf4j-api`，arrayFormat 方法有多个同名实现，最终调用的是：

```java
final public static FormattingTuple arrayFormat(final String messagePattern, final Object[] argArray, Throwable throwable) {

    if (messagePattern == null) {
        return new FormattingTuple(null, argArray, throwable);
    }

    if (argArray == null) {
        return new FormattingTuple(messagePattern);
    }

    int i = 0;
    int j;
    // use string builder for better multicore performance
    StringBuilder sbuf = new StringBuilder(messagePattern.length() + 50);

    int L;
    for (L = 0; L < argArray.length; L++) {

        j = messagePattern.indexOf(DELIM_STR, i); // DELIM_STR 的值是 "{}"

        if (j == -1) {
            // no more variables
            if (i == 0) { // this is a simple string
                return new FormattingTuple(messagePattern, argArray, throwable);
            } else { // add the tail string which contains no variables and return
                // the result.
                sbuf.append(messagePattern, i, messagePattern.length());
                return new FormattingTuple(sbuf.toString(), argArray, throwable);
            }
        } else {
            if (isEscapedDelimeter(messagePattern, j)) {
                if (!isDoubleEscaped(messagePattern, j)) {
                    L--; // DELIM_START was escaped, thus should not be incremented
                    sbuf.append(messagePattern, i, j - 1);
                    sbuf.append(DELIM_START);  // DELIM_START 的值是 "{"
                    i = j + 1;
                } else {
                    // The escape character preceding the delimiter start is
                    // itself escaped: "abc x:\\{}"
                    // we have to consume one backward slash
                    sbuf.append(messagePattern, i, j - 1);
                    deeplyAppendParameter(sbuf, argArray[L], new HashMap<Object[], Object>());
                    i = j + 2;
                }
            } else {
                // normal case
                sbuf.append(messagePattern, i, j);
                deeplyAppendParameter(sbuf, argArray[L], new HashMap<Object[], Object>());
                i = j + 2;
            }
        }
    }
    // append the characters following the last {} pair.
    sbuf.append(messagePattern, i, messagePattern.length());
    return new FormattingTuple(sbuf.toString(), argArray, throwable);
}
```

原理就是解析字符串，进行替换。 对照下面几个用例，理解下就行了：

```java
logger.info("Hello {}", "World");
logger.info("Hello {} {}", "World");
logger.info("Hello {} \\{}", "World");
logger.info("Hello {} \\{} {}", "World");
logger.info("Hello {} \\{} {}", "World", "HI");
```
