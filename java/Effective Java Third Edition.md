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
