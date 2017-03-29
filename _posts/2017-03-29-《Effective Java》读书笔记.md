---
layout: post
title: 《Effective Java》读书笔记
categories: [java]
---



## 前言

这阵子刚看完了《Effective Java》收获非常多，例如用Builder模式来简化多参数Bean的创建枚举那章让我学习到了枚举非常优雅的使用、泛型的正确使用、equals/hashcode的正确覆盖方法……

记下来这篇笔记，用于记忆每一条准则。当然还有很多点我目前还没参透的，等以后有时间第二遍学习再逐步完善自己的知识体系并且来完善这篇笔记。

至于并发那章因为很多并发的知识在之前《Java并发编程实战》这本书中学习到了，并且都记在了这本书的笔记中，因此在这里就不再赘述。Serializable特性目前项目中使用比较少，因此也是草草看过，这里也就不不懂装懂了。

## 1.考虑用静态工厂方法代替构造器
当一个类需要多个带有相同签名的构造器时，就用静态工厂方法代替多个构造器，并且慎重的选择名称以便突出他们之前的区别。

静态工厂相对构造器的三大优势：
1. 静态工厂方法都有名称
2. 不必在每次调用都创建一个新的对象（对象的复用）
3. 可以返回类型的任意子类型对象 （多态性）

静态工厂惯用名称：
1. valueOf
2. of (valueOf 更简洁用法)
3. getInstance
4. newInstance
5. getType
6. newType

## 2.遇到多个构造器参数时考虑使用构建起
构造器模式。详见构造器博客。

## 3.用私有构造器或者枚举类型强化Singleton属性
单例模式的使用。
最基本就是用私有构造器来保证实例不会被构造。
除此之外，枚举是一种很优雅的单例模式实现方法。

单例模式三种实现方式：
1. 公有域
2. 公有静态方法 （推荐）
3. 包含单个元素的枚举类型 （强烈推荐）


## 4.通过私有构造器强化不可实例化的能力
同上。主要还阐明了对于一些不希望实例化的类，例如静态工具类也应该将构造方法申明为静态的强化其不可实例化的能力。

## 5.避免创建不必要的对象
常见的如String池
``` java
String a = "hello";
String b = "hello";
String c = new String("hello");
a == b //true
a == c //false
```

ab共用连接池对象，c为不必要创建出来的对象。

使用静态工厂也可以避免产生不必要的对象：

``` java
Boolean a = new Boolean("true"); //产生对象
Boolean a = Boolean.valueOf("true"); //不产生
```

当心自动装箱以及拆箱：
``` java
Long sum = 0L;
for (long i = 0; i < 10000; i++){
    sum += i;  //自动装箱 形成一个Long对象
}
```
上面的例子就不必要的产生了10000个Long对象。因为long会自动装箱成Long。改进方法为让sum为long类型。

**慎用对象池，维护自己的对象池通常会把代码库弄得很乱，同时增加内存占用，并且还会损害性能。现代的JVM实现具有高度优化的垃圾回收机制，其性能很快就会超过轻量级对象池的性能**


## 6.消除过期的引用
清空对象引用的最好办法应该是让包含该引用的变量结束其生命周期。如果正确使用了局部变量那么这种消除操作会自然而然的发生。
除此之外，如果是依赖于程序员自己管理的内存时，应该警惕内存泄露的问题。

## 7.避免使用终结方法
**don't use finalize() **

## 8.覆盖equals时请遵守通用约定
当覆盖了equals方法时应保证该方法依然遵循下列的性质。

equals方法的一些性质(不包含null)：
* 自反性：x, x.equals(x) 返回true
* 对称性：x,y。x.equals(y) == y.equals(x)
* 传递性：x,y,z x.equlas(y) == true 且 y.equals(z) 则 x.equlas(z)
* 一致性：只要equals的比较操作在对象中的引用的信息没有被修改，那么多次调用得到结果是一致的
* 对于任何非null的引用 x x.equals(null) 返回 false

实现高质量equasl方法的诀窍：
1. 使用==操作符检查 “参数是否为这个对象的引用” 如果是直接返回true。 （性能优化）
2. 使用instanceof操作符检查 “参数是否为正确的类型”。
3. 把参数转换成正确的类型 (因为上一条的检查所以可以安全的转换)
4. 对于该类中的每个“关键”域，检查参数中的域是否与该对象中的对应域相匹配。
5. 覆盖equals方法时总要覆盖hashCode方法
6. 不要将equals申明中的Object对象替换为其他已知的类型。（会使方法变成重载，记得使用@override)

**对于既不是float也不是double类型的基本类型域可以直接使用==操作符。对于float域可以使用Float.compare方法，而对于double域，可以调用Double.compare。对于float和double域进行特殊处理是有必要的，因为存在着Float.NaN、-0.0f以及类似的double常量。并且存在着精度的偏差问题。**

## 9.覆盖equals时总要覆盖hashCode
如果两个对象根据equals（Object）方法比较是相等的，那么调用这两个对象中任意一个对象的hashCode方法都必须产生相同的整数结果。

## 10.始终要覆盖toString
覆盖toString可以让类打印出更加有用的信息。
**个人觉得始终说的有点太绝对，如果你坚信自己这个类不会被打印出来，那么还是没有必要去覆盖的。**

## 11.谨慎地覆盖clone
不要使用clone，请自己实现深拷贝。

## 12.考虑实现Compareable接口
CompareTo的通用约定与equasls方法的相似：
1. 确保 a.compareTo(b) == - b.compareTo(a) 当且仅当a.compareTo(b) 抛出异常时 b.compareTo(a) 抛出异常。
2. 可传递： a.compareTo(b) > 0 && b.compareTo(z) > 0  => a.compareTo(z) > 0
3. 强烈建议 a.compareTo(b) == 0 => x.equlas(b)

compareTo的比较要确保结果不会越界：
return Integer.compare(a,b) 而不是 return a - b // 可能会溢出

## 13.使类和成员的可访问性最小化
如果类或者接口能被做到包级私有的，它就应该被做成包级私有的。通过把类或者接口做成包级私有的，它实际上成了这个包实现的一部分，而不是该包导出API的一部分，在以后发行版本中，可以对它进行修改、替换、或者删除，而无需担心会影响到现有的客户端。

另外记住，实例域不能是公有的。如果域是非final的或者是指向一个可改变对象final引用。那么一旦使这个域为公有的，就放弃了对存储在这个域中值进行限制的能力，这意味着你也放弃了强制这个域不可变的能力。

## 14.在公有类中使用访问方法而非公有域
getter、setter意义：
如果类可以在它所在的包外部进行方法，就提供访问方法，以保留将来改变该类内部表示法的灵活性。如果公有类暴露了它的数据域，要想在在将来改变其内部方法是不可能的，因为公有类客户端代码已经遍布各处了。

## 15.使可变性最小
为了使类成为不可变的，要遵循下面五条规则：
1. 不要提供任何修改对象状态的方法
2. 保证类不会被扩展
3. 使所有的域都是final的
4. 使所有的域都是私有的
5. 确保对于任何可变组件的互斥访问。如果类有指向可变对象的域要确保该对象不被客户端引用

**不可变对象本质是线程安全的，他们不需要同步。可以被自由的共享。且可以配合静态工厂方法实现缓存。**

不可变对象的真正唯一缺点是，对于每个不同的值都要创建一个单独对象。创建这种对象可能代价很高。

## 16.复合优先于继承
跨包继承存在危险，当父类改变了，会间接破坏了子类。
只有当子类真正是超类的子类型时，才适合使用继承。即必须有 "is-a" 关系。
否则考虑使用复合（包装类），包装类不仅比子类更加健壮，而且功能也更加强大（更易于扩展）。

## 17.要么为继承而设计，并提供文档说明，要么就禁止继承

## 18.接口优于抽象类
实现接口需要重复定义某些方法，这个问题在Java 8 中已经得到解决。但若非用到Java 8的新特性可以考虑引入骨架实现类，减少重复代码。 详情见骨架类笔记。

## 19.接口只用于定义类型
常量接口模式是对接口的不良使用。

## 20.类层次优先于标签类
标签类：用一个域来记录该类的性质
应该使用清晰的类层次而不是硬编码实现。

## 21.用函数对象表示策略
例如Comparator接口。
通过一个类实现了Comparator接口来后传递给Arrays.sort(a,new comp()) 实现一组对象的排序。这就是策略模式，不改变原对象的代码而是生成新策略来改变它的行为。

相反Comparable接口则需要需要排序的类来实现它的compareTo方法。

策略类可以使用匿名类，但是请注意使用匿名类时每次调用都会创建一个新的实例。如果反复使用的话可以将其存放在一个私有的静态final域里。
``` java
class Host{
	// 无需导出具体策略 , 还可以实现其他接口（Serializable)
	private static class StrLenCmp implements Comparator<String> , Serializable {
		public int compare(String s1,String s2){
			return s1.length() - s2.length();
		}
	}

	// 导出公有静态域
	public static final Comparator<String> STRING_LENGTH_COMPARATOR = new StrLenCmp();
}
```

## 22.优先考虑静态成员类
如果申明成员类不要求访问外围实例，就要始终把static修饰符放在它的声明中，使它成为静态成员类而不是非静态成员类。如果省略了static修饰符，则每个实例都包含一个额外的指向外围对象的引用。保留这份引用需要消耗时间和空间，并且会导致外围实例在符合垃圾回收时却仍然得以保留。

## 23.请不要在新代码中使用原生态类型
使用泛型而不是原生类型。反省有子类型化的规则，List<String>是List的子类型，而不是List<Object>的子类型。因此如果使用了List这样的原生态类型，就会失掉类型安全性。

但是这条规则也有两个小小的例外,这两者都源于“泛型信息可以在运行时被擦除”这一事实。在类文字中必须使用原生态类型。规范不允许使用参数化类型（虽然允许数组类型和基本类型）。即List.class、String[].class和int.class都是合法的，但是List<String.class> 和 List<?>.class 都是不合法的。

Set<?> 无限制通配符不允许插入任何元素（除了null）

## 24.消除非受检警告
SuppressWarning("unchecked") 消除未受检警告，但是必须自己确保确实这个警告是不必要的。
**永远不要在整个类上使用@SuppressWarnings，尽可能小范围的使用**

你可以在整个方法上使用但也尽量不要而是用在一个局部变量上：
``` java
public <T> T[] toArray(T[] a) {
	if (a.length < size) {
		@suppressWarnings("unchecked")
		T[] result = (T[]) Arrays.copyOf(elements,size,a.getClass());
		return result;
	}
}
```

## 25.列表优先于数组
利用数组你会在运行时发现所犯的错误，而利用列表，则可以在编译时发现错误。因为数组是具体化的所以会在运行时才知道并检查他们的元素类型的约束。泛型则是通过擦除来实现的，因此泛型在编译时强化他们的类型信息，并在运行时丢弃（擦除）他们的元素类型信息。
``` java
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // 运行时抛出异常

List<Object> o1  = new ArrayList<Long>(); // 编译时就抛出
o1.add("I don't fit in"); 
```

像E、List<E>和List<String>这样的类型被称作不可具体化的类型。直观的说，不可具体化的类型是指其运行时表示法包含的信息比其编译时包含的信息更少。唯一可具体化的参数化类型是无限制的通配符类型，例如List<?> 和 Map<?,?>。虽然不常用，但是创建无限制通配符类型的数组是合法的。

## 26.优先考虑泛型
记住，你不能创建不可具体化的数据。
``` java
E[] e = new E[10]; // error
```
因此应该优先考虑使用泛型，即使使代码没有那么简洁，但是更加安全也更加容易。

## 27.优先考虑泛型方法
理由和上一条类似，更加安全、可靠。

## 28.利用有限制通配符来提升API灵活性
PECS = producer-extends , consumer-super 原则
如果参数化类型表示一个T生产证，就使用<? extends T> 如果表示的是一个消费者使用<? super T>

一般来说，如果类型参数只在方法申明中出现一次，就可以用通配符来取代它。如果是无限制的类型参数，就用无限制的通配符来取代它；如果是有限制的类型参数，就用有限制的通配符取代它。

## 29.优先考虑类型安全的异构容器
异构容器是指能够容纳不同类型对象的容器。像我们通常用的List、Map等容器，它们的原生态类型本身就是异构容器，一旦给它们设置了泛型参数，例如List<String>、Map<Integer, String>，它们就不再是异构容器。但是，原生态类型是不安全的，你无法知道从容器取出的类型到底是什么，很容易导致错误。因此，如何构建类型安全的异构容器就成了一个重要的话题。

使用Map实现类型安全的异构容器
局限性
使用Map实现类型安全的异构容器

我们将要实现一个Favorites类，用来对每个类型保存一个最喜欢的实例。它的API如下：
``` java
public class Favorites {
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type);
}
```
下面是一个测试程序，说明了如何使用Favorites类保存、获取并打印最喜爱的String、Integer和Class实例。

``` java
public static void main(String[] args) {
  Favorites f = new Favorites();
  f.putFavorite(String.class, "Java");
  f.putFavorite(Integer.class, 0xcafebabe);
  f.putFavorite(Class.class, Favorites.class);
  String favoriteString = f.getFavorite(String.class);
  int favoriteInteger = f.getFavorite(Integer.class);
  Class<?> favoriteClass = f.getFavorite(Class.class);
  System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
}
```
打印结果是
Java cafebabe Favorites
Favorite实例是类型安全的，当你向它请求String的时候，它绝不会返回一个Integer给你。同时它也是异构的，它的键可以是任意类型。

Favorites的实现也很简单：

``` java
public class Favorites {
  private Map<Class<?>, Object> favorites = new HashMap<Class<?>, Object>();

  public <T> void putFavorite(Class<T> type, T instance) {
    if (type == null)
      throw new NullPointerException("Type is null");
    favorites.put(type, instance);
  }

  public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
  }
}
```
内部用一个Map<Class<?>, Object>来保存所有的爱好，使用Class<?>作为键记录每个爱好的类型，而用Object作为值不再区分它们的类型。当取出时，根据请求的类型从Map中查找相应的值，由于值是Object类型的，需要使用type.cast强制转换为type指定的类型。只要客户端按照API的要求使用，这里的强制转换一定不会出错。

局限性

这种实现方法有两种局限性。

首先，恶意的客户端可以破坏Favorites实例的类型安全。如果客户端传入原生态的Class对象和不一致的值对象，则会在getFavorite的cast时抛出ClassCastException异常。不过好在我们可以对这一情况加以约束。只需要在put时使用一个动态的转换就可以了：

``` java
public <T> void putFavorite(Class<T> type, T instance) {
  favorites.put(type, type.cast(instance));
}
```
一旦客户端传入值类型不一致，就立即抛出异常。

第二种局限性是它不能用于泛型化类型，例如，你无法把List<String>作为Favorites的键，因为List<String>.class是个语法错误。这一局限性还没有很好的解决方法。

## 30.用enum代替int常量
Java枚举类型背后的基本思想非常简单：他们就是用公有的静态final域为每个枚举常量导出实例的类。因为没有可以访问的构造器，枚举类型是真正的final。
枚举类还有一个好处，你可以增加或者重排列枚举类型中的常量，而无需重新编译它的客户端代码，因为导出常量的域在枚举类型和他的客户端之间提供了一个隔离层：常量值并没有被编译到客户端代码中，而是在int枚举模式中。

**枚举的toString方法返回每个枚举的声明名字。valueOf方法通过名字来获取枚举类,valueOf方法依赖于默认的toString方法因此如果覆盖了toString将不再有用。**

**特定于方法的方法实现:**
可以在枚举中申明一个抽象方法，这样所有的枚举常量都得实现它：
``` java
public enum type{
	JSON{
		String convert(String raw) { 
			//...covert in json 
		}
	}, XML {
		String covert(String raw) {
			//...convert in xml
		}
	}
	abstract String convert(String raw);
}
```

**嵌套的枚举：策略枚举模式**
假如对于一个枚举每次都得选择一个方法，就想上述的特定于方法的方法实现，但是有很多个枚举他们用的方法是一样的，如果用上述的代码你得重复写很多一样的代码。这个时候可以考虑将方法嵌套在一个内部类中，对于外部枚举每定义一个就选择一个策略。

``` java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY), 
    TUESDAY(PayType.WEEKDAY), 
    WEDNESDAY(PayType.WEEKDAY), 
    THURADAY(PayType.WEEKDAY), 
    FRIDAY(PayType.WEEKDAY), 
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);

    private final PayType payType;
    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    double pay(double hoursWorked, double payRate) {
        return payType.pay(hoursWorked, payRate);
    }

    //私有嵌套的枚举类
    private enum PayType {
        WEEKDAY {
            double pay(double hoursWorked, double payRate) {
                return hoursWorked - HOURS_PER_SHIFT > 0 
                    ?(hoursWorked*payRate*1.5 - 0.5*HOURS_PER_SHIFT*payRate) 
                    : hoursWorked*payRate;
            }
        },
        WEEKEND {
            double pay(double hoursWorked, double payRate) {
                return hoursWorked * payRate * 1.5;
            }
        };
        private static final int HOURS_PER_SHIFT = 8;

        abstract double pay(double hoursWorked, double payRate);
    }

    public static void main(String[] args) {
        System.out.println(PayrollDay.MONDAY.pay(10,10));
        System.out.println(PayrollDay.SUNDAY.pay(10,10));
    }
}
```

## 31.用实例域代替序数
所有枚举类都有一个ordinal方法，它返回枚举常量在类型中的数字位置。但是**永远不要根据枚举的序数导出与它关联的值，你应该将它保存在一个实例域中。**
``` java
enum Enumble{
	SOLE(1),DUET(2);

	private final int number;
	public int Enumble(int size){
		this.number = size;
	}
}
```

## 32.用EnumSet代替位域
如果方法需要传递多个枚举值，使用EnumSet会比用位域来控制更好。
``` java
public class Main {
    public enum Style {BOLD, ITALTC, UNDERLTINE}

    public void applyStyles(Set<Style> styles) {
        System.out.println(styles);
    }

    public static void main(String[] args) {
        Main main = new Main();
        main.applyStyles(EnumSet.of(Style.BOLD, Style.ITALTC));
    }
}
```

## 33.使用EnumMap代替序数索引
千万不要使用ordinal来索取枚举类型。可以使用EnumMap来代替索引。
``` java
public class Main {
    public enum Style {BOLD, ITALTC, UNDERLTINE}

    public EnumMap<Style, String> push() {
        EnumMap<Style, String> enumMap = new EnumMap<Style, String>(Style.class);
        enumMap.put(Style.BOLD,"good");
        enumMap.put(Style.ITALTC,"normal");
        return enumMap;
    }

    public static void main(String[] args) {
        Main main = new Main();
        EnumMap<Style,String> enumMap = main.push();
        System.out.println(enumMap.get(Style.BOLD));
    }
}
```

### 34.用接口模拟可伸缩的枚举
虽然枚举类型不是可以扩展的，但是接口是。因此可以利用接口来模拟可以伸缩的枚举。
``` java
interface Operation {
    double apply(double x, double y);
}


enum BaseOperation implements Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    };

    private String symbol;

    BaseOperation(String symbol) {
        this.symbol = symbol;
    }
}
```
也可以将方法申明为抽象的实现这种伸缩性。（30条）

### 35.注解优先于命名模式
使用合适的注解，会比你对一个方法任何简明的命名更加让人了解它的用途。

### 36.坚持使用Override注解
Override通常能让你避免重载了方法却以为是覆盖了方法。

### 37.用标记接口定义类型
标记接口是没有包含方法申明的接口，而只是指明一个类实现了具有某种属性的接口。例如，考虑Serializable接口，通过实现这个接口，类表明它的实例可以被写到ObjectOutputStream。
**注解以及标记接口区分使用：**
* 注解：该标记应用到任何元素而不是只给类和接口。因为只有类和接口可以用来或者扩展接口。
* 标记接口：只应用给类和接口。

### 38.检查参数的有效性
每当编写方法或者构造器的时候，应该考虑它的参数有哪些限制。应该把这些限制写到文档中，并且在这个方法体的开头处，通过显示的检查来实施这些限制。

### 39.必要时进行保护性拷贝
保护性拷贝是在检查参数的有效性之前进行的，并且有效性检查是针对拷贝之后的对象，而不是针对原始对象。这样做可以避免在检查参数以及拷贝对象参数时间段间原始对象的改变。
**使用保护性拷贝主要是预防原始对象的改变而对当前对象造成非预期的异常。所以只要可能就应该尽可能的使用不可变对象。**

### 40.谨慎设计方法签名
* 谨慎地选择方法的名称。
* 不要过于追求提供便利的方法。每个方法都应该尽其所能。只有当一个些组合的方法经常被调用时才为他们创建“快捷方式”。
* 避免过长的参数。目标是四个或者更少。
* 对于参数类型，要优先使用接口而不是类。

### 41.慎用重载
重载方法，要调用哪一个重载方法是在编译时决定的。因此重载方法的选择是静态的。
``` java
public void test(){
	Collection<?> c = new HashSet<>();
	classify(c);
}

public void classify(Collection<?> c){
	//打印出这个
	System.out.println("collection");
}

public void classify(Set<?> s){
	System.out.println("set");
}
```
所以上面代码只有Collection这个方法会被选中因为在编译时，c对象就是Collection类型。
**需要当心覆盖机制没有成功覆盖而变成了重载，因此要坚持使用@Override**

### 42.慎用可变参数
在重视性能的情况下，使用可变参数要小心。可变参数方法的每次调用都会导致进行一次数组分配和初始化。有一个改进办法，为这个方法提供5个重载方法，分别为0个、1个、2个、3个、3个以上参数版本。
``` java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1,int a2) {}
public void foo(int a1,int a2,int a3) {}
public void foo(int a1,int a2,int a3, int... rest) {}
```
EnumSet类对它的静态工厂使用这种方法，最大限度的减少创建枚举集合的成本。当时这么做是必要的，因为枚举集合为位域提供在性能方面有竞争力的替代方法。

### 43.返回零长度的数组或者集合,而不是null
``` java
public List<Chesse> getCheeseList() {
	if (cheeseInStock.isEmpty()){
		// 永远返回同一个对象
		return Collections.emptyList();
	}else{
		return new ArrayList<Cheese>(cheeseInStock);
	}
}
```
返回类型为数组或者集合而不是null，可以避免客户端调用抛出的空指针异常。使用Collections.emptyList等方法可以返回同一个空对象节省对象创建的资源。

### 44.为所有导出的API元素编写文档注释
如果是编写API务必遵守

### 45.将局部变量的作用域最小化
要使局部变量的作用域最小化最有力的方式是**在第一次使用它的地方申明，不要提前！** 几乎每个局部变量的申明都应该包含一个初始化表达式。如果你还没有足够的信息来对这个变量进行有意义的初始化，就应该推迟这个申明。

**for循环 以及 while **
如果在循环的终止之后就不再需要使用循环变量的内容应该更优先于使用for。

### 46.for-each循环优先于传统的for循环
**利用for-each循环不会有性能的损失，甚至用数组也一样。**

三种情况不能用for-each：
1. 过滤
   如果需要遍历集合，并删除特定的元素，就需要使用显示的迭代器，以便可以调用它的remove方法。
2. 转换
   如果需要遍历列表或者数组，并取代它部分或者全部的元素值，就需要列表迭代器或者数组索引，以便设定元素的值。
3. 平行迭代
   如果需要并行的遍历多个集合，就需要显示的控制迭代器或者索引变量，以便所有迭代器或者索引变量都可以同步前移。

### 47.了解和使用类库
类库或者第三方库（guava等）拥有的方法应该了解并使用它们而不要亲自造轮子。

### 48.如果需要精确的答案，请避免使用float和double
float以及double不适合用于非常精确的计算。解决办法是使用BigDecimal、int或者long。后面两者不能进行浮点数计算。
此外使用BigDecimal有两个缺点：
1. 使用不方便
2. 效率低
   但是BigDecimal**可以很方便的控制保留小数。**因此，如果你需要精确答案并且是浮点数类型的请使用BigDecimal。

### 49.基本类型优先于装箱基本类型
基本类型和装箱基本类型的三个主要区别：
1. 基本类型只有值，装箱类型具有与他们值不同的唯一性——可以具有不同对象但是值相同
2. 装箱基本类型不仅有功能完备值还有非功能值：null
3. 基本类型更加节省空间和时间

综上所述如果不是业务需求，更优先考虑基本类型。
**使用过程中当心自动装箱、自动拆箱带来的不必要对象创建浪费。**

### 50.如果其他类型更适合，则尽量避免使用字符串
* 字符串不适合代替其他的值类型。应该根据语义使用相对应的值类型
* 字符串不适合代替枚举类型
* 字符串不适合代替聚集类型，通常可以编写一个私有静态类来表示，而不是使用字符串
* 字符串也不适合代替能力表字符串作为键可能会被共享引发不安全


### 51.当心字符串连接的性能
``` java
// 会被JVM自动转化为StringBuilder
String a = "hello" + name + " world";
// 不会自动转化
String a = "hello";
a += name;
a += " world";
```

有多行拼接操作记得使用StringBuilder 或者 StringBuffer

### 52.通过接口引用对象
使用接口引用对象，可以利用多态性，提升代码的可维护性。
例如使用List而不是ArrayList 将来发现该链表经常被修改想要改成LinkedList只需修改定义地方就可以了。

### 53.接口优先于反射机制
反射机制是Java一种很强大的机制，但是使用这种能力也是要付出代价的：
1. 丧失了编译时类型检查的好处。反射是在运行中进行的
2. 执行反射访问所需的代码非常笨拙和冗长。虽然可以使用第三方库，但是还是略显繁琐
3. 性能损失。反射方法调用比普通方法慢很多。

因此，如果你编写的程序是必须要与编译时未知的类一起工作，如有可能就应该仅仅使用反射机制来实例化对象，而访问对象时则使用编译时已知的某个（父类）接口或者超类。
如果是与编译时已知的类一起工作的则不必要使用反射机制。

### 54.谨慎地使用本地方法
JNI允许JAVA应用程序可以调用本地方法。所谓调用本地方法是用本地程序语言（主要是C或者C++）来编写的特殊方法。本地方法在本地语言中可以执行任意的计算任务，并且返回到Java程序设计语言。
历史上使用本地方法主要是希望提升性能。但是**使用本地方法来提升性能的做法不值得提倡。**现在的VM已经非常快了无需再去借助其他语音。并且本地方法还有一些严重的缺点：
1. 本地语言不是安全的
2. 不可移植
3. 使用本地方法的应用程序难以调试
4. 进入和退出本地方法时需要固定的开销，浪费资源
5. 需要“胶合代码”的本地方法编写起来单调乏味，并且难以阅读

### 55.谨慎地进行优化
不要因为性能而牺牲合理的结构。需要努力编写好的程序而不是快的程序。如果好的程序不够快，它的结构将使它可以得到优化。好的程序体现了信息隐蔽的原则：只要有可能，它们就会把设计决策集中在单个模块中，因此可以改变单个决策，而不会影响到系统的其他部分。

### 56.遵守普遍接受的命名规范
Java规范 以及 驼峰命名

### 57.只针对异常的情况才使用异常
不要为了快速、业务去滥用异常，只有在真正异常的情况下才使用他们。

### 58.对可恢复的情况使用受检异常，对编程错误使用运行时异常
Java提供了三种可抛出结构：
1. 受检的异常 —— 可恢复时使用
2. 运行时异常 —— 编程错误抛出
3. 错误


### 59.避免不必要地使用受检的异常
``` java
try{
    obj.action(args);
}catch(TheCheckException e){
    //Handle exceptional condition
    ...
}

//重构为
if (obj.actionPermitted(args){
    obj.action(args);
}else{
    //Handle exception condition
    ...
}
```

可以考虑使受检的异常重构。这样的重构并不总是恰当的。但是在恰当的时候可以使API用起来更加的舒服。

### 60.优先使用标准异常
最常见的可重用异常：

| 异常                              |          使用场合          |
| :------------------------------ | :--------------------: |
| IllealArgumentException         |      非null到数值不正确       |
| IllegalStateException           |   对于方法的调用而言，对象状态不适合    |
| NullPointerException            | 在禁止使用null的情况下参数值为null  |
| IndexOutOfBoundsException       |        下表参数值越界         |
| ConcurrentModificationException | 在禁止并发修改的情况下，检查到对象的并发修改 |
| UnsupportedOperationException   |      对象不支持用户请求的方法      |

### 61.抛出与抽象相对应的异常
想想这样一种情况：方法B抛出了一个受检的异常 ，那么方法A在内部调用方法B时，面对方法B抛出的受检异常，可以选择继续抛出向上传播这个异常，
也可以捕获这个异常进行处理。究竟是向上传播抛出，还是捕获处理呢？
有一个指导原则是：抛出与抽象相对应的异常。
例如如果方法B抛出了NoSuchElementException这个受检异常，然而在方法A中调用方法B时，根据方法A中的逻辑，当遇到NoSuchElementException
异常时，抛出一个IndexsOutOfBoundsException异常更为合适。那么就不应该选择向上传播抛出NoSuchElementException，而是应该选择捕获NoSuchElementException，然后抛出IndexsOutOfBoundsException。
更高层的实现应该捕获底层的异常，同时抛出可以按照高层抽象进行解释的异常。这种做法称为异常转译（exception translation）。
一种特殊的异常转译形式称为异常链（exception chaining)。
尽管异常转译让异常更加明确。但是如有可能，处理来自底层的异常的最好的做法是，在调用低层方法之前确保它们会成功执行，从而避免它们抛出异常。
有时候，可以在给低层方法传递参数之前，检查更高层方法的参数的有效性，从而避免低层方法抛出异常。
如果无法避免低层异常，次选方案是，让更高层的方法来悄悄地绕开这些异常（方法C调用方法A，那么方法C就是更高层的方法）。那么在高层方法中
调用低层方法时，面对低层方法抛出的受检异常，高层异常可以捕获异常，转化为非受检异常，或者利用某种适当的记录机制（日志）将异常记录下来。
这样更高层的方法C在调用高层方法A是，不用再受来自低层方法的异常烦扰，而异常在高层方法中也得到了处理。

### 62.每个方法抛出的异常都要有文档
使用@thorws标签记录下一个方法可能抛出的每个未受检异常，但是不要使用throws的关键字将未受检异常也包含在方法的申明中。

### 63.在细节消息中包含能捕获失败失败的信息
为了确保在异常的细节消息中包含足够的能捕获失败的信息，一种办法是在异常的构造器而不是字符串细节信息。然后有了这些信息，只要把他们放到消息描述中，就可以自动产生细节消息了。
``` java
public  IndexOutBoundException(int lowerBound,int upperBound,int index){
	//Generate a detail message that captures the failure
	super("Lower bound: " + lowerBound +
		  ",Upper bound: " + upperBound +
		  ", Index:");
	this.lowerBound = lowerBound;
	this.upperBound = upperBound;
	this.index = index;
}
```

### 64.努力使失败保持原子性
失败的方法调用应该使对象保持在被调用之前的状态。具有这种属性的方法被称为具有**失败原子性**。
保持失败原子性的方法：
1. 最简单的方法设计不可变对象,如果对象不必变,那么失败原则性是必然.如果一个操作失败,它可能会组织创建新的对象,但永远不会使已有的对象保持在不一致的状态中,因为每个对象被创建后就处于一致的状态,以后再也不会改变.(实在没看懂)
2. 对于可变对象上执行操作的方法,获得失败原子性最常见的方法是,在执行操作之前检查参数有效性.使得对象状态被改变之前,先抛出适当异常.类似获取失败原子性的方法,调整计算过程顺序,使得任何可能导致失败的计算部分都在对象被改变之前发生.如果对参数检查只有执行部分计算后才进行,实际上就是上面方法的拓展而已.例如,考虑TreeMap情景,它的元素按照某种特定顺序排列,添加元素时,该元素的类型必须可以利用TreeMap的排序规则和其他元素比较.如果企图添加不正确的元素,在tree以任何方式被修改之前,自然会导致ClassCastException异常.
3. 不太常用的操作方法,编写一段恢复代码,由它来拦截操作过程中发生的失败,以及使对象回滚到操作开始之前的状态.这种方法主要用于永久性数据结构(存在磁盘中).
4. 在对象的一份临时拷贝上执行操作,当操作完成在用临时性拷贝的结果代替对象的内容.例如.Collections.sort在执行排序之前,需要把输入列表转到一个数组中,以便降低排序内循环中访问元素的开销.虽然是出于性能考虑,但是及时排序失败,能保证输入列表保持原样.

### 65.不要忽略异常
``` java
//don't do that!
try{
	...
}catch(SomeException e){
	//empty
}
```
> 以下为并发笔记，可以参考之前写的并发笔记。因此写的比较简陋。

### 66.同步访问共享的可变数据
共享数据记得使用同步操作。或者使用线程安全的类。

### 67.避免过度同步
缩小同步的范围、减小同步的粒度都可以很大程度提升性能。

### 68.executor优先于线程
**线程池的选择**
如果编写的是小程序或者轻载的服务器使用Executors.newCachedThreadPool通常是个不错的选择，因为它不需要配置，并且一般情况下能够正确的完成工作。
但是在大负载的产品服务器中，最好使用Executors.newFixedThreadPool，因为它为你提供了一个包含固定线程数目的线程池，或者为了最大限度的控制它，就直接使用ThreadPoolExecutor类。

**定时调度**
虽然timer使用起来更加容易，但是被调度的小城吃executor更加灵活。timer只用一个线程来执行任务，这在面对长期运行的任务时，会影响到定时的准确性。如果timer唯一的线程抛出未被捕获的异常，timer就会停止运行。被调度的线程池支持多个线程，并且优雅的从抛出未受检异常任务中恢复。

### 69.并发工具优先于wait和notify
使用并发工具，优先于wait和notify。
常用工具：
* 阻塞队列
* CountDownLatch

### 70.线程安全性的文档化
导出API，应该同时编写好线程安全的文档

### 71.慎用延迟初始化
延迟初始化，最好的建议是“除非绝对必要，否则不要这么做。”

### 72.不要依赖于线程调度器
要编写健壮的、相应良好的、可移植的多线程应用程序，最好的办法是确保可运行线程的平均数量不明显多余处理器的数量。
线程不应该一直处于忙-等的状态，即反复地检查一个共享对象，以等待某些事情发生。你可能会想用方法Thread.yield来让这个线程让出CPU。但是不要企图这可以很好的“修正”这个问题。你可能好不容易的让程序能够工作，但是这样的程序仍然是不可移植的。同一个yield调用在一个JVM实现上能提高性能，而在另一个JVM上实现可能会更差。Thread.yield没有可测试的语义，更好的解决办法是重新构造应用程序，以减少可并发的线程数量。
**所以慎用Thread.yield。**

### 73.避免使用线程组
**几乎可以说永远也不要使用线程组。**


> Serializable 的使用目前比较少，因此这部分笔记待补充。

### 74.谨慎地实现Serializable接口
1. 实现Serializable接口而付出的最大代价是，一旦一个类被发布，就大大降低了“改变这个类的实现”的灵活性。因为序列化与反序列化版本必须要保持一致。
2. 实现Serializable的第二个代价是，它增加了出现Bug和安全漏洞的可能性。
3. 实现Serializable的第三个代价是，随着类发行新的版本，相关的测试负担也增加了。

### 75.考虑使用自定义的序列化形式

### 76.保护性地编写readObject方法

### 77.对于实例控制，枚举类型优先于readResolve

### 78.考虑使用序列化代理代替序列化实例