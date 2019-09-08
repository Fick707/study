# Java新特性
## java 8
### lambda表达式
Lambda表达式可以代替匿名类，它将功能（函数）当成方法的参数，或者将代码当成数据；实现高效且代码紧凑；  
java.util.function包中定义了好多只包含一个方法的简单函数接口；  
Lambda表达式配合stream方便高效紧凑；  
Lambda 表达式聚合操作(Aggregate Operations)功能强大；  
Lambda可以被序列化，只要其目标类型和参数可以被序列化，只是不建议序列化。因为正如内部类序列化，可能会潜在一些问题，写代码要注意很多；  

```
Arrays.sort(rosterAsArray, Person::compareByAge);
```
### 方法引用
方法引用支持以下方式：  

| 类型 | 示例 |
| -------- | -------- |
| Reference to a static method   | ContainingClass::staticMethodName   |
| Reference to an instance method of a particular object   | containingObject::instanceMethodName   |
| Reference to an instance method of an arbitrary object of a particular type   | ContainingType::methodName   |
| Reference to a constructor   | ClassName::new   |

### Default Methods
默认方法。
试想，有一个旧接口，需要添加一个方法。那可麻烦了，因为除了修改接口本身，它所有的实现类都要去修改。
哈哈，jdk8以后，默认方法解决了这个问题。给旧接口添加新方法时，指定默认实现，它的旧实现类不用动就能兼容，是不是很开心。

```
public interface TimeClient {
    void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
                               int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
        
    static ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                    "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }
            
    default ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }
}
```

### Static Methods
接口里也可以有静态方法了。如上

### 类型注解
之前的注解只能用于定义的地方，而1.8开始，则可能在类型引用的任何地方。
例如 @NotNull String name;

### 重复注解
现在支持了相同的注解在同一个地方出现多次。

### stream和聚合功能
归约是将集合中的所有元素经过指定运算，折叠成一个元素输出，如：求最值、平均数等，这些操作都是将一个集合的元素折叠成一个元素输出。


    1.Intermdiate：一个Stream后面可以调用任意多个Intermdiate操作方法，其目的主要是打开流，作出某种程度的数据映射/过滤，然后返回一个新的流交给下一步操作，但是这一系列的操作是lazy的，并不是每个操作一一调用。
    2.terminal：顾名思义，terminal操作就是将Stream关闭，一个Stream最多只能有一个terminal操作。
    
    Intermediate：
    map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered
    
    Terminal：
    forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator
    
    Short-circuiting：
    anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit


### Pipeline and Stream
一个pipeline包含：源、操作、终结操作
A pipeline contains the following components:

```
    A source: This could be a collection, an array, a generator function, or an I/O channel. 

    Zero or more intermediate operations. An intermediate operation, such as filter, produces a new stream.

    A terminal operation. A terminal operation, such as forEach, produces a non-stream result, such as a primitive value (like a double value), a collection, or in the case of forEach, no value at all. In this example, the parameter of the forEach operation is the lambda expression e -> System.out.println(e.getName()), which invokes the method getName on the object e. (The Java runtime and compiler infer that the type of the object e is Person.)
```

### Compact Profiles
jdk8提供了compact1, compact2, 和 compact3三个版本的紧凑版本，运行在不同类型的终端上。

### try-with-resources

### Optional解决空指针
```
return Optional.ofNullable(u)
                    .map(user->user.name)
                    .orElse("Unknown");
```

## java 9
[what's new in jdk9](https://www.cnblogs.com/lufeiludaima/p/pz20190216.html)

### jshell

### 多个版本jar兼容

### 接口的私有方法

### 改进try-with-resources

### 钻石操作符
钻石操作符可以使用匿名实现类，可以在匿名实现类中重写方法等操作。

### 限制使用单独下划线标识符
### 集合接口的of方法创建只读集合；

## java 10

### 局部变量类型推断

## java 11