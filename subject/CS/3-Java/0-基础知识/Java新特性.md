

# Java8

## Lambda表达式

+ Lambda 是一个**匿名函数**，我们可以把 Lambda 表达式理解为是**一段可以传递的代码**（将代码像数据一样进行传递）。使用它可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使Java的语言表达能力得到了提升

- Lambda 表达式：在Java 8 语言中引入的一种新的语法元素和操作符。这个操作符为 “->” ， 该操作符被称为 Lambda 操作符 或箭头操作符。它将 Lambda 分为两个部分：

- 左侧：指定了 Lambda 表达式需要的参数列表 （其实就是接口中的抽象方法的形参列表）

- 右侧：指定了 Lambda 体，是抽象方法的实现逻辑，（其实就是重写的抽象方法的方法体）

```java
//语法格式四：Lambda 若只需要一个参数时，参数的小括号可以省略
@Test
public void test4(){
    Consumer<String> con1 = (s) -> {
        System.out.println(s);
    };
    con1.accept("一个是听得人当真了，一个是说的人当真了");
    System.out.println("*******************");

    Consumer<String> con2 = s -> {
        System.out.println(s);
    };
    con2.accept("一个是听得人当真了，一个是说的人当真了");
}


//语法格式五：Lambda 需要两个或以上的参数，多条执行语句，并且可以有返回值
@Test
public void test5(){
    Comparator<Integer> com1 = new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
                System.out.println(o1 + o2);
            System.out.println(o1);
        }
    };
    System.out.println(com1.compare(12,21));
    System.out.println("*****************************");

    Comparator<Integer> com2 = (o1,o2) -> {
           System.out.println(o1 + o2);
        return o1.compareTo(o2);
    };
    System.out.println(com1.compare(111,222));
}
```

**类型推断**

+ 在Lambda 表达式中的参数类型都是由编译器推断得出的。Lambda 表达式中无需指定类型，程序依然可以编译，这是因为 javac 根据程序的上下文，在后台推断出了参数的类型。Lambda 表达式的类型依赖于上下文环境，是由编译器推断出来的。这就是所谓的“类型推断

```java

     // 没有类型推断，因为给o1,o2指定了Enginner 类型
    Comparator<Enginner> comparator = (Enginner o1, Enginner o2) -> o1.getJob().compareTo(o2.getJob());
    

     //  有类型推断，因为没有给o1,o2指定了Enginner 类型
     Comparator<Enginner> comparator2 = ( o1,  o2) -> o1.getJob().compareTo(o2.getJob());
```

## Optional类

### why

**解决空指针处理繁琐问题**

**解决不必要的null值检查问题**

Java 8中引入了一个叫做`java.util.Optional`的新类可以避免`null`引起的诸多问题

我们看看一个`null`引用能导致哪些危害。首先创建一个类`Computer`，结构如下图所示

![输入图片说明](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407252006449.webp)

当我们调用如下代码会怎样？

```java
String version = computer.getSoundcard().getUSB().getVersion();
```

上述代码看似是没有问题的，但是很多计算机（比如，树莓派）其实是没有声卡的，那么调用`getSoundcard()`方法可定会抛出空指针异常了。

一个常规的但是不好的的方法是返回一个null引用来表示计算机没有声卡，但是这就意味着会对一个空引调用`getUSB()`方法，显然会在程序运行过程中抛出控制异常，从而导致程序停止运行

那么该怎么避免在程序运行时会出现空指针异常呢？你需要保持警惕，并且不断检查可能出现空指针的情况，就像下面这样

```java
String version = "UNKNOWN";
if(computer != null)
    {
        Soundcard soundcard = computer.getSoundcard();
        if(soundcard != null){
             USB usb = soundcard.getUSB();
             if(usb != null){
                 version = usb.getVersion();
                }
            }
    }
```

然而，你可以看到上述代码有太多的null检查，整个代码结构变得非常丑陋。但是我们又不得不通过这样的判断来确保系统运行时不会出现空指针。如果在我们的业务代码中出现大量的这种空引用判断简直让人恼火，也导致我们代码的可读性会很差。如果你忘记检查要给值是否为空，null引用也是存在很大的潜在问题

### how

Java 8引入了一个新类叫做`java.util.Optional<T>`,这个类的设计的灵感来源于Haskell语言和Scala语言。这个类可以包含了一个任意值，像下面图和代码表示的那样。你可以把Optional看做是一个有可能包含了值的值，如果Optional不包含值那么它就是空的，下图那样

![输入图片说明](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407252010160.webp)

```java
public class Computer {
  private Optional<Soundcard> soundcard;
  public Optional<Soundcard> getSoundcard() { ... }
  ...
}

public class Soundcard {
  private Optional<USB> usb;
  public Optional<USB> getUSB() { ... }

}

public class USB{
  public String getVersion(){ ... }
}
```

上述代码展现了一台计算机有可能包含一个声卡（声卡是有可能存在也有可能不存在）。声卡也是有可能包含一个USB端口的。这是一种改善方法，该模型可以更加清晰的反映一个被给定的值是可以不存在的

但是该怎么处理`Optional<Soundcard>`这个对象呢？毕竟，你想要获取的是USB的端口号。很简单,`Optional`类包含了一些方法来处理值是否存在的状况。和null引用相比`Optional`类迫使你在你要做值是否存在相关处理，从而避免了空指针异常

需要说明的是Optional类并不是要取代null引用。相反地，是为了让设计的API更容易被理解，当你看到一个函数的签名时，你就可以判断要传递给这个函数的值是不是有可能不存在。这就促使你要打开Optional类来处理确实值的状况了

### 创建Optional对象

+ 可以使用静态方法 `empty()` 创建一个空的 Optional 对象

```java
Optional<Soundcard> sc = Optional.empty();
```

+ 可以使用静态方法 `of()` 创建一个非空的 Optional 对象

```java
SoundCard soundcard = new Soundcard();
Optional<Soundcard> sc = Optional.of(soundcard);
```

如果声卡null，空指针异常会立刻被抛出（这比在获取声卡属性时才抛出要好）

+ 可以使用静态方法 `ofNullable()` 创建一个即可空又可非空的 Optional 对象

```java
Optional<Soundcard> sc = Optional.ofNullable(soundcard);
```

`ofNullable()` 方法内部有一个三元表达式，如果参数为 null，则返回私有常量 EMPTY；否则使用 new 关键字创建了一个新的 Optional 对象——不会再抛出 NPE 异常

### 判断值是否存在

可以通过方法 `isPresent()` 判断一个 Optional 对象是否存在，如果存在，该方法返回 true，否则返回 false——取代了 `obj != null` 的判断

```java
Optional<String> opt = Optional.of("沉默王二");
System.out.println(opt.isPresent()); // 输出：true

Optional<String> optOrNull = Optional.ofNullable(null);
System.out.println(optOrNull.isPresent()); // 输出：false
```

Java 11 后还可以通过方法 `isEmpty()` 判断与 `isPresent()` 相反的结果

```java
Optional<String> opt = Optional.of("沉默王二");
System.out.println(opt.isEmpty()); // 输出：false

Optional<String> optOrNull = Optional.ofNullable(null);
System.out.println(optOrNull.isEmpty()); // 输出：true
```

**非空表达式**

Optional 类有一个非常现代化的方法——`ifPresent()`，允许我们使用函数式编程的方式执行一些代码，因此，我把它称为非空表达式。如果没有该方法的话，我们通常需要先通过 `isPresent()` 方法对 Optional 对象进行判空后再执行相应的代码

```java
Optional<String> optOrNull = Optional.ofNullable(null);
if (optOrNull.isPresent()) {
    System.out.println(optOrNull.get().length());
}
```

有了 `ifPresent()` 之后，情况就完全不同了，可以直接将 Lambda 表达式传递给该方法，代码更加简洁，更加直观

```java
Optional<String> opt = Optional.of("沉默王二");
opt.ifPresent(str -> System.out.println(str.length()));
```

Java 9 后还可以通过方法 `ifPresentOrElse(action, emptyAction)` 执行两种结果，非空时执行 action，空时执行 emptyAction

```java
Optional<String> opt = Optional.of("沉默王二");
opt.ifPresentOrElse(str -> System.out.println(str.length()), () -> System.out.println("为空"));
```

### 设置（获取）默认值

有时候，我们在创建（获取） Optional 对象的时候，需要一个默认值，`orElse()` 和 `orElseGet()` 方法就派上用场了。

`orElse()` 方法用于返回包裹在 Optional 对象中的值，如果该值不为 null，则返回；否则返回默认值。该方法的参数类型和值的类型一致

```java
String nullName = null;
String name = Optional.ofNullable(nullName).orElse("沉默王二");
System.out.println(name); // 输出：沉默王二
```

`orElseGet()` 方法与 `orElse()` 方法类似，但参数类型不同。如果 Optional 对象中的值为 null，则执行参数中的函数

```java
String nullName = null;
String name = Optional.ofNullable(nullName).orElseGet(()->"沉默王二");
System.out.println(name); // 输出：沉默王二
```

从输出结果以及代码的形式上来看，这两个方法极其相似，这不免引起我们的怀疑，Java 类库的设计者有必要这样做吗？

假设现在有这样一个获取默认值的方法，很传统的方式

```java
public static String getDefaultValue() {
    System.out.println("getDefaultValue");
    return "沉默王二";
}
```

然后，通过 `orElse()` 方法和 `orElseGet()` 方法分别调用 `getDefaultValue()` 方法返回默认值

```java
public static void main(String[] args) {
    String name = null;
    System.out.println("orElse");
    String name2 = Optional.ofNullable(name).orElse(getDefaultValue());

    System.out.println("orElseGet");
    String name3 = Optional.ofNullable(name).orElseGet(OrElseOptionalDemo::getDefaultValue);
}
```

注：`类名 :: 方法名`是 Java 8 引入的语法，方法名后面是没有 `()` 的，表明该方法并不一定会被调用。

输出结果如下所示

```java
orElse
getDefaultValue

orElseGet
getDefaultValue
```

输出结果是相似的，没什么太大的不同，这是在 Optional 对象的值为 null 的情况下。假如 Optional 对象的值不为 null 呢？

```java
public static void main(String[] args) {
    String name = "沉默王三";
    System.out.println("orElse");
    String name2 = Optional.ofNullable(name).orElse(getDefaultValue());

    System.out.println("orElseGet");
    String name3 = Optional.ofNullable(name).orElseGet(OrElseOptionalDemo::getDefaultValue);
}
```

输出结果如下所示

```java
orElse
getDefaultValue
orElseGet
```

`orElseGet()` 没有去调用 `getDefaultValue()`。哪个方法的性能更佳，你明白了吧？

### 获取值

直观从语义上来看，`get()` 方法才是最正宗的获取 Optional 对象值的方法，但很遗憾，该方法是有缺陷的，因为假如 Optional 对象的值为 null，该方法会抛出 NoSuchElementException 异常。这完全与我们使用 Optional 类的初衷相悖

```java
public class GetOptionalDemo {
    public static void main(String[] args) {
        String name = null;
        Optional<String> optOrNull = Optional.ofNullable(name);
        System.out.println(optOrNull.get());
    }
}
```

### 过滤值

小王通过 Optional 类对之前的代码进行了升级，完成后又兴高采烈地跑去找老马要任务了。老马觉得这小伙子不错，头脑灵活，又干活积极，很值得培养，就又交给了小王一个新的任务：用户注册时对密码的长度进行检查。

小王拿到任务后，乐开了花，因为他刚要学习 Optional 类的 `filter()` 方法，这就派上了用场

```java
public class FilterOptionalDemo {
    public static void main(String[] args) {
        String password = "12345";
        Optional<String> opt = Optional.ofNullable(password);
        System.out.println(opt.filter(pwd -> pwd.length() > 6).isPresent());
    }
}
```

`filter()` 方法的参数类型为 Predicate（Java 8 新增的一个函数式接口），也就是说可以将一个 Lambda 表达式传递给该方法作为条件，如果表达式的结果为 false，则返回一个 EMPTY 的 Optional 对象，否则返回过滤后的 Optional 对象。

在上例中，由于 password 的长度为 5 ，所以程序输出的结果为 false。假设密码的长度要求在 6 到 10 位之间，那么还可以再追加一个条件。来看小王增加难度后的代码

```java
Predicate<String> len6 = pwd -> pwd.length() > 6;
Predicate<String> len10 = pwd -> pwd.length() < 10;

password = "1234567";
opt = Optional.ofNullable(password);
boolean result = opt.filter(len6.and(len10)).isPresent();
System.out.println(result);
```

这次程序输出的结果为 true，因为密码变成了 7 位，在 6 到 10 位之间。想象一下，假如小王使用 if-else 来完成这个任务，代码该有多冗长

### 转换值

**案例一：转换长度**

小王检查完了密码的长度，仍然觉得不够尽兴，觉得要对密码的强度也进行检查，比如说密码不能是“password”，这样的密码太弱了。于是他又开始研究起了 `map()` 方法，该方法可以按照一定的规则将原有 Optional 对象转换为一个新的 Optional 对象，原有的 Optional 对象不会更改。

先来看小王写的一个简单的例子

```java
public class OptionalMapDemo {
    public static void main(String[] args) {
        String name = "沉默王二";
        Optional<String> nameOptional = Optional.of(name);
        Optional<Integer> intOpt = nameOptional
                .map(String::length);
        
        System.out.println( intOpt.orElse(0));
    }
}
```

在上面这个例子中，`map()` 方法的参数 `String::length`，意味着要 将原有的字符串类型的 Optional 按照字符串长度重新生成一个新的 Optional 对象，类型为 Integer

**案例二：转为小写并过滤**

搞清楚了 `map()` 方法的基本用法后，小王决定把 `map()` 方法与 `filter()` 方法结合起来用，前者用于将密码转化为小写，后者用于判断长度以及是否是“password”

```java
public class OptionalMapFilterDemo {
    public static void main(String[] args) {
        String password = "password";
        Optional<String>  opt = Optional.ofNullable(password);

        Predicate<String> len6 = pwd -> pwd.length() > 6;
        Predicate<String> len10 = pwd -> pwd.length() < 10;
        Predicate<String> eq = pwd -> pwd.equals("password");

        boolean result = opt.map(String::toLowerCase).filter(len6.and(len10 ).and(eq)).isPresent();
        System.out.println(result);
    }
}
```

**案例三：获取值并过滤**

我们可以使用map方法重写**检测null，然后再提取对象**类型的对象

```java
Optional<USB> usb = maybeSoundcard.map(Soundcard::getUSB);
```

```java
maybeSoundcard.map(Soundcard::getUSB)
      .filter(usb -> "3.0".equals(usb.getVersion())
      .ifPresent(() -> System.out.println("ok"));
```

### 案例

**案例：解决多重判断空值问题**

如何使用安全的方式实现下面代码呢？

```java
String version = computer.getSoundcard().getUSB().getVersion();
```

注意上面的代码都是从一个对象中提取另一个对象，使用map函数可以实现。在前面中我们设置了Computer中包含的是一个Optional对象，Soundcard包含的是一个Optional对象，因此我们可以这么重构代码

```java
String version = computer.map(Computer::getSoundcard)
                  .map(Soundcard::getUSB)
                  .map(USB::getVersion)
                  .orElse("UNKNOWN");
```

不幸的是，上面的代码会编译错误，那么为什么呢？computer变量是Optional类型的，所以它调用map函数是没有问题的。但是`getSoundcard()`方法返回的是一个`Optional<Soundcard>`的对象，返回的是`Optional<Optional<Soundcard>>`类型的对象，进行了第二次map函数的调用，结果调用`getUSB()`函数就变成非法的了。下面的图描述了这种场景

![输入图片说明](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407252139025.webp)

map函数的源码实现是这样的

```java
 public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

可以看出map函数还会再调用一次`Optional.ofNullable()`,从而导致返回`Optional<Optional<Soundcard>>`

Optional提供了flatMap这个函数，它的设计意图是当对Optional对象的值进行转化（就像map操作）然后一个两级Optional压缩成一个。下面的图展示了Optional对象通过调用map和flatMap进行类型转化的不同

![输入图片说明](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407252140262.webp)

因此我们可以这样写

```java
String version = computer.flatMap(Computer::getSoundcard)
                   .flatMap(Soundcard::getUSB)
                   .map(USB::getVersion)
                   .orElse("UNKNOWN");
```

第一个flatMap保证了返回的是Optional而不是Optional<Optional>，第二个flatMap实现了同样的功能从而返回的是 Optional。注意第三次调用了`map()`,因为`getVersion()`返回的是一个String对象而不是一个Optional对象

我们终于把刚开始使用的嵌套null检查的丑陋代码改写了可读性高的代码，也避免了空指针异常的出现的代码