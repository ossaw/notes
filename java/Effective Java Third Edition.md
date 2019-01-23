# Effective Java Third Edition 读书笔记

## 第1章 引言
略

## 第二章 创建和销毁对象

> 何时以及如何创建对象, 何时以及如何避免创建对象, 如何确保它们能够适时地销毁, 以及如何管理对象销毁之前必须进行的各种清理动作.

### 第1条: 用静态工厂方法代替构造器

静态工程方法较构造器相比有以下几个优势:

* **静态工厂方法有名称**, 一个类的多个构造器之间只能靠参数类型顺序来区分, 静态工厂方法则不同, 它可以有多个方法签名来替代多个构造器.
* **静态工厂方法不必每次调用时都创建一个新对象**, 使得不可变类可是使用预先构建好的实例, 或者将构建好的实例缓存起来, 进行重复利用, 静态工厂方法能够为重复的调用返回相同的对象, 这样有助于类总是能够控制在某个时刻哪些实例应该存在, 这种类被称为实例受控的类. 编写实力受控的类原因如下: 实例受控的类可以确保它是一个Singleton(第3条), 或者是不可实例化的(第4条), 它还使得不可变的值类(第17条)可以确保不存在两个相等的实例, 即当且仅当a == b时, a.equals(b)才为true, 这是享元模式, 枚举类型保证了这一点(第34条).
* **静态工厂方法可以返回原返回类型的任何子类型的对象**, 这样在选择返回对象的类时, 就有了更大的灵活性, 配合接口使用, 可增加扩展性.
* **静态工厂方法所返回对象的类可随着每次调用而发生变化, 这取决于静态工厂方法的参数值.** 只要是以声明的返回类型的子类型这都是可以的. 已返回的子类型的存在对客户端来说是透明的, 这在将来某个版本中被替换也不会有影响, 客户端永远也不知道也不关心他们从工厂方法中返回的对象的类, 他们只管它是否是已声明返回类型的子类.
* **静态方法返回的对象所属的类, 可以在编写该静态工厂方法的类时不存在.** 

静态工厂的缺点如下:
* **类如果不包含共有或者受保护的构造器, 就不能被初始化.**
* **程序员很难发现它们, 因为他们不像构造器那样在API中明确标识出来.**

在编写一个新类时, 切记第一反应是提供共有的构造器, 而不是静态工厂方法.

### 第2条: 遇到多个构造器参数时, 考虑使用构建器

静态工厂和构造器都不能很好的扩展到有**大量的可选的**参数

```java
// Telescoping constructor pattern - does not scale well! (Pages 10-11)
public class NutritionFacts {
    private final int servingSize;  // (mL)            required
    private final int servings;     // (per container) required
    private final int calories;     // (per serving)   optional
    private final int fat;          // (g/serving)     optional
    private final int sodium;       // (mg/serving)    optional
    private final int carbohydrate; // (g/serving)     optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
}
```

1. 考虑上面这个类, 采用的是重叠构造器的方式, 只有两个参数时必须的, 其余四个参数时可选的, 四个可选参数排列组合, 构造器数量会爆炸, 而且参数类型还一致, 难以阅读, 如果在增加参数会越来越复杂, 如果客户端不小心颠倒了两个参数顺序, 不会有编译错误, 但是运行时就会有问题.

```java
// JavaBeans Pattern - allows inconsistency, mandates mutability  (pages 11-12)
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize = -1; // Required; no default value
    private int servings = -1; // Required; no default value
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }

    // Setters
    public void setServingSize(int val) {
        servingSize = val;
    }

    public void setServings(int val) {
        servings = val;
    }

    public void setCalories(int val) {
        calories = val;
    }

    public void setFat(int val) {
        fat = val;
    }

    public void setSodium(int val) {
        sodium = val;
    }

    public void setCarbohydrate(int val) {
        carbohydrate = val;
    }
}
```

1. 在考虑上面的类, 它采用的是JavaBean模式, 它弥补了重叠构造器的不足, 但是它的构造过程被分布到多个调用中, 在构造过程中JavaBean可能存在不一致的状态, 也就是JavaBean模式是不安全发布(详见Java并发编程实战第16章),  JavaBean模式使类做成不可变的可能性不复存在. 较重叠构造器相比, JavaBean模式存在线程安全问题, JavaBean模式可以用在方法体内作为局部变量使用.

```java
// Builder Pattern  (Page 13)
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}
```

1. 考虑上面代码, 采用了Builder模式, 它即可以满足重叠构造器模式的安全发布的优点, 保证参数颠倒问题, 类的不可变性, 又可以满足JavaBean模式的代码易于阅读.

Builder模式也适用于类层次结构.

遗漏东西

Builder模式的不足在于, 为了创建对象, 必须先创建它的构建器, 在一些十分注重性能的情况下, 可能会有问题.

在最开始设计类时, 如果使用构造器或者工厂方法, 等到类参数过多时才添加构建器, 那么那些过时的构造器和静态工程会不协调, 因此通常一开始就使用构建器.

### 第3条: 用私有构造器或者枚举类型强化Singleton属性

使类成为Singleton会使它的客户端测试十分困难, 因为不能个Singleton替换模拟实现, 除非实现一个充当器类型的接口.

实现Singleton的两种常用方式:

```java
// Singleton with public final field  (Page 17)
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}

// Singleton with static factory (Page 17)
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public static Elvis getInstance() {
        return INSTANCE;
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}
```

1. 共有静态域的方法优势在于, API很清楚的表明了这是一个Singleton: 公有的静态域是final的, 该域总是包含相同的对象引用, 第二个优势在于它更简单.

2. 静态工厂方法的优势在于, 它提供了灵活性: 再不改变API的情况下, 我们可以改变该类是否为Singleton的想法, 第二个优势在于如果应用程序需要, 可以编写一个泛型Singleton工厂, 最后一个优势是可以通过方法引用作为提供者, 比如Elvis::instance就是一个Supplier<Elvis>. (个人理解方法引用就是语法糖, 不明白为啥是优势)

3. 这两种方式在受到反射攻击时单例会被破坏.

4. 如果单例类实现了Serializable接口, 也会受到在反序列化时单例也会遭到破坏, 在类中加入readResolve方法可以防止.

```java
// readResolve method to preserve singleton property
private Object readResolve() {
	return INSTANCE;
}
```

1. readResolve方法官方文档定义签名如下: ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;

实现Singleton的第三种方式是声明一个包含单个元素的枚举类型: 

```java
// Enum singleton - the preferred approach (Page 18)
public enum Elvis {
    INSTANCE;

    // This code would normally appear outside the class!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}
```

1. 此种方式可防止反射或者反序列化破坏单例, 已经成为实现Singleton的最佳方式.

2. 如果Singleton必须扩展一个超类, 而不是扩展Enum的时候, 不宜使用这种方式(即使可以声明枚举去实现接口).

### 第4条: 通过使用私有构造器来强化不可实例化的能力

1. 对于一些工具类而言, 它们只包含静态方法和静态域, 不希望被实例化. 企图通过把类做成抽象类来强制该类不可被实例化是行不通的, 该类可以被子类化, 子类可以实例化, 而且这样也会误导用户, 以为这种类就是为了继承而设计的(第19条).

2. 让这个类包含一个私有构造器, 可以防止不被实例化

```java
// Noninstantiable utility class (Page 19)
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }

    // Remainder omitted
}
```

1. AssertionError不是必须的, 但是它可以避免不小心在类内部调用构造器, 也可以通过注释来代替AssertionError.

2. 这种方式的缺点在于不能使该类被子类化, 因为子类没有可访问的超类构造器可用.

### 第5条: 优先考虑依赖注入来引用资源

1. 有的类可能需要依赖一个或多个底层资源, 静态工具类和Singleton类并不适合于需要引用底层资源的类.

```java
public class SpellChecker {
    private static final Lexicon dictionary = new Object();

    private SpellChecker() {} // Noninstantiable

    public static boolean isValid(String word) {}

    public static List<String> suggestions(String typo) {}
}

// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    private final Lexicon dictionary = new Object();

    private SpellChecker() {}

    public static final INSTANCE = new SpellChecker();

    public boolean isValid(String word) {}

    public List<String> suggestions(String typo) {}
}
```

2. 上面两种方式, 一个不可实例化, 一个单例, 也就表明它们无法依赖多个类型的Lexicon, 如果想要依赖多种类型的Lexicon, 必须提供set函数, 这会破坏Lexicon final属性, 导致无法并行工作.

3. 这种类需要的是能够支持类的多个实例, 当创建一个新的实例时, 就将该资源传到构造器中, 这是依赖注入的一种形式.

4. 依赖注入的对象具有不可变性, 多个客户端可以共享依赖对象, 依赖注入同样适用于构造器, 静态工厂和构建器.

5. 这种模式的另一种变体是, 将资源工厂传给构造器, 工厂是可以被重复调用来创建类型实例的一个对象, 这类工厂具体表现为工厂方法模式. 在Java8中增加的函数式接口Supplier<T>, 最适合用于表示工厂.

6. 依赖注入极大地提升了灵活性和可测试性, 但会导致大型项目的凌乱不堪, 因为它通常包含上千个依赖, 这可以采用一种依赖注入框架来解决, 如Spring.

7. 需要注意的是Spring框架注入的Bean虽然默认是单例的, 且大多数Bean中依赖多个资源, set注入方式还破坏资源的final属性, 这些看起来似乎和本条目相矛盾, 实则不然, Spring采用set注入的这种Bean, 虽然从技术上来看是可变的, 但是从业务上来看其依赖资源在发布后是不可变的(在传统的MVC模式中, 似乎没有哪个系统会人为修改Spring注入的资源), 这种Bean被称为事实不可变对象(参加那个Java并发编程实战第3.5.4章).

### 第6条: 避免创建不必要的对象

1. 最后能够重用单个对象, 而不是在每次需要的时候创建一个相同功能的对象, 重用方式即快速, 又流行. 如果对象是不可变的(当然某些适当的情况下也包括事实不可变对象), 它就始终可以被重用.

```java
String s = new String("bikini"); // DON'T DO THIS!

String s = "bikini"; // DO THIS!
```

2. 对于同时提供了静态工厂和构造器的不可变类, 通常优先使用静态工厂方法而不是构造器, 以避免创建不必要的对象.

3. 处理重用不可变对象之外, 也可以重用哪些已知不会修改的可变对象(事实不可变对象).

4. 有些对象的创建成本比较高, 如果需要重复的需要这类对象, 可将其缓存起来(例如数据库连接或者线程, 可采用池化技术缓存连接或者线程)

```java
// Performance can be greatly improved!
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

这段代码在不适合在注重性能的场景下被频繁的调用, 问题在于matches方法内部创建了一个Pattern实例, 只用了一次就进行垃圾回收了, 这个实例的创建成本很高.

```java
// Reusing expensive object for improved performance
public class RomanNumerals {
	private static final Pattern ROMAN =
			Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

	static boolean isRomanNumeral(String s) {
		return ROMAN.matcher(s).matches();
	}
}
```

将Pattern变成一个成为类初始化的一部分, 将其缓存起来, 每次调用都会重用, 必要的时候可以采用延迟初始化技术(第83条).

当然如果isRomanNumeral方法没有被频繁调用, 可以采用第一种方式, 节省内存空间, 具体可酌情考虑.

5. 另一种创建对象的方式是自动装箱, 自动装箱是的基本类型和包装类型之间的差别模糊起来, 但是并没有完全消除.

```java
// Hideously slow! Can you spot the object creation?
private static long sum() {
	Long sum = 0L;
	for (long i = 0; i <= Integer.MAX_VALUE; i++)
		sum += i;
	return sum;
}
```

这段程序就因为将sum声明为Long包装类, 在后续的计算过程中构造了$2^31$个Long实例, 将sum的声明long基本类型之后, 性能提升很高.

6. 优先使用基本类型, 而不是包装类型, 当心无意识的自动装箱操作.

7. 对象池技术适合用来缓存一些重量级对象.

8. 本条目与第50条有关保护性拷贝相对应, 本条目是说当你应该重用现有对象时, 请不要创建新对象. 第50条是说: 当你应该创建新对象时, 请不要重用现有对象. 必要时如果没能实施保护性拷贝, 将会导致潜在的BUG和安全漏洞, 而不必要的创建对象则只会影响程序的风格和性能.

### 第7条: 消除过期的引用

> 即使有垃圾回收器, 也会存在内存泄露问题, 消除过期引用有助于垃圾回收.

```java
// Can you spot the "memory leak"?  (Pages 26-27)
public class Stack {
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    private Object[] elements;
    private int size = 0;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public static void main(String[] args) {
        Stack stack = new Stack();
        for (String arg : args) {
            stack.push(arg);
        }

        while (true) {
            System.err.println(stack.pop());
        }
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    /**
     * Ensure space for at least one more element, roughly doubling the capacity each time the array
     * needs to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

1. 这段程序没有明显的功能错误, 但是却存在内存泄露问题, 如果这个栈先是增长, 然后在收缩, 那么从栈中弹出来的对象不会被垃圾回收, 因为栈内部维护了对这些对象的过期引用.

2. 如果一个引用被无意识的保存起来, 那么垃圾回收器不仅不处理这个对象, 也不会处理这个对象所引用的其它可达对象.

```java
// Corrected version of pop method (Page 27)
public Object pop() {
	if (size == 0)
		throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null; // Eliminate obsolete reference
	return result;
}
```

新的pop方法可修正内存泄露问题, 显示清空已弹出栈的过期引用.

3. 清除过期引用的另一个好处是, 如果过期引用再次被使用, 那么程序立刻回抛出NullPointException异常, 而不是悄悄的错误运行下去.

4. 只要类是自己管理内存, 程序员就应该警惕内存泄露问题, 一旦元素被释放掉, 则该元素包含的任何对象引用都应该被清空.

5. 内存泄露另一常见来源是缓存, 将对象引用放到缓存中, 很容易被忘掉, 造成内存泄露.

6. 只要缓存之外存在对某个项的键的引用, 该项就有意义, 那么可以使用WeakHashMap代表缓存, 当缓存中的项过期后, 它们就会被自动的清除, 记住只有当所要的缓存项的生命周期是由该键的外部引用而不是由值决定时, WeakHashMap才有用处.

7. 内存泄露的第三个常见来源是监听器和其它回调, 如果你实现一个API, 客户端在这个API中注册回调, 却没有显式取消回调.

8. 借助Heap剖析工具可发现内存泄露问题.

### 第8条: 避免使用终结方法和清除方法

1. 终结(finalizer)方法通常是不可预测的, 也是很危险的, 一般情况下不要使用, 使用终结方法会导致行为不稳定, 性能降低, 以及可移植性问题.

2. 在Java9中用清除方法代替了终结方法, 清除方法没有终结方法那么危险, 但仍然是不可预测, 运行缓慢, 一般情况下也是不必要的.

3. 终结方法和清除方法的缺点在于不能保证会被及时执行, 注重时间的任务不应该用终结方法或者清除方法来完成.

4. 永远不应该依赖终结方法或清除方法来更新重要的持久状态.

5. 如果忽略在终结过程中被抛出的未被捕获的异常, 该对象的终结过程也会终止.

6. 使用终结方法或者清除方法还有严重的性能损失.

7. 终结方法有一个严重的安全问题: 它们为终结方法攻击打开了类的大门, 从构造器中抛出异常, 应该足以防止对象继续存在, 有了终结方法的存在, 这一点就做不到了, 为了防止非final类受到终结方法攻击, 要编写一个空的final的finalizer方法.

8. 终结方法和清除方法有两个用途如下:
当资源的所有者忘记调用它的close方法时, 终结方法或者清除方法可以充当安全网.
清除方法的第二种理由与对象的本地对等体有关. 本地对等体属于堆外内存, JVM不会回收, 这部分的内存需要人为释放.

9. 此条目后续内容涉及Java9特性, 暂不介绍.

### 第9条: try-wiht-resources优先于try-finally

1. 在处理必须关闭的资源时, 始终要优先考虑用try-with-resources, 而不是用try-finally, 这样得到的代码将更简洁, 产生的异常也更有价值.

## 第3章 对于所有对象都通用的方法

> 任何一个类在重写Object类的非final方法时, 都有责任遵守通用约定, 如果不能做到这一点, 其它依赖这些约定的类将无法结合该类一起工作

### 第10条: 重写equals请遵守通用约定

在一下任意一种情况下不用重写equals方法:

* 类的每个实例本质上都是唯一的.(一些非值类, 例如Thread)

* 类没有必要提供逻辑相等的测试功能.(一些工具类, 单例类)

* 超类已经重写了equals, 超类的行为对于这个类也是合适的. (在一些抽象类中可能会提供这种全局统一的实现, 例如Set从AbstractSet继承equals)

* 类是私有的(通常指内部类), 或是包级私有的, 并且你可以确定业务中它的equals方法永远不会被调用.

如果类具有自己的特有的逻辑相等的概念, 且超类还没有重写equals, 这通常是指值类, 例如String.

覆盖equals时, 必须遵守它的通用约定:

* 自反性: 对于非null的引用值x, x.equals(x)必须返回true.

* 对称性: 对于非null的引用值x, y, x.equals(y)为true, 那么y.equals(x)也必须返回true.

* 传递性: 对于非null的引用值x, y, z, x.equals(y)为true, y.equals(z)为true, 那么x.equals(z)也必须返回true.

* 一致性: 对于非null的引用值x, y, 只要equals比较的对象信息没有被修改, 多次调用x.equals(y)都会一致返回true或false.

* 对于非null引用x, x.eqauls(null), 必须返回false.

实现高质量equals方法的诀窍: 

1. 使用 == 操作符判断参数是否为这个对象的引用, 是则返回true, 否则返回false.

2. 使用instanceof操作符检查参数是否为正确的类型, 不是返回false.

3. 把参数转换为正确的类型.

4. 对于该类中的每个关键域, 检查参数对象的域是否与该对象中的域想匹配.

```java
// Class with a typical equals method (Page 48)
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix = rangeCheck(prefix, 999, "prefix");
        this.lineNum = rangeCheck(lineNum, 9999, "line num");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max) {
            throw new IllegalArgumentException(arg + ": " + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof PhoneNumber)) {
            return false;
        }
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    // Remainder omitted - note that hashCode is REQUIRED (Item 11)!
}
```

1. 重写equals总要重写hashCode.

2. 不要企图让equals方法过于智能.

3. 不要将equals生命中的Object对象替换为其它类型, 替换了那就不是重写了, 可以使用@Override注解来协助检查.
