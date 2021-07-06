# 一. ADT设计

## ADT是什么

ADT是Abstract Data types（**抽象数据类型**)的缩写形式, 是具有类似行为的特定类别的数据结构的数学模型。例如我们设计一个表示队列的ADT, 可以对其进行push和pop操作。

```java
public interface Queue<L> {
    public boolean push(L element);
    public Optional<L> pop();
}
```

上述代码在代码层面对ADT进行了一个实现, ADT是一个模型, 不是代码结构, ADT定义一组操作来描述一个数据类型的方法、属性等外在表现, 对其内部实现不作假设。

## ADT的表示独立性

上一节提到了ADT对其内部实现不作假设，意思是ADT的使用者不应该对ADT的具体实现类进行ADT中未定义的操作。例如我们实现一个ListQueue, 它实现了Queue ADT。

```java
public class ListQueue<L> implements Queue<L> {
    public List<L> Mlist = new ArrayList<>();
    
    @Override
    public boolean push(L element){
        return Mlist.add(element);
    }

    @Override
    public Optional<L> pop(){
        try {
            L element = Mlist.get(0);
            return Optional.of(element);
        } catch (IndexOutOfBoundsException e){
            return Optional.empty();
        }
    }
}
```

在ListQueue的内部我们使用了List<L>来保存所有的元素。现在假设我们需要获取Queue中的所有值。

```java
Queue<String> queue = new ListQueue<>();
queue.push("Hello");
queue.push("World");
List<String> allElements = ((ListQueue)queue).Mlist;
for str in allElements {
    System.out.println(str);
}
```

这样是符合Java语法的, 但是这种做法破坏了ADT的**表示独立性**(**Representation Independence**)，即上层调用者不应依赖ADT的实现细节。这里的代码依赖了ListQueue内部实现的Mlist成员。

**Representation Independence**的意义在于分离程序中的数据结构的形式和对其使用的方式。在ListQueue中, Mlist成员变量代表数据结构的形式, push和pop方法则代表了使用的方式。通过将数据结构与使用方式分离可以有效提高代码的可复用性和减少潜在的bug。

- ### 提高代码可复用性

  假设我们的ADT实现发生变化, 例如我们打算使用ArrayQueue、MapQueue、LinkedQueue来对Queue进行实现，我们就可以不对ADT的使用进行修改

  ```java
  Queue<Integer> queue = new ListQueue<>();
  queue.push(Integer.valueOf(1));
  queue.push(Integer.valueOf(2));
  // ============================================ //
  Queue<Integer> queue = new ArrayQueue<>();
  queue.push(Integer.valueOf(1));
  queue.push(Integer.valueOf(2));
  // ============================================ //
  Queue<Integer> queue = new MapQueue<>();
  queue.push(Integer.valueOf(1));
  queue.push(Integer.valueOf(2));
  // ============================================ //
  Queue<Integer> queue = new LinkedQueue<>();
  queue.push(Integer.valueOf(1));
  queue.push(Integer.valueOf(2));
  // ============================================ //
  ```

  上述代码中的我们的push方法调用的代码是一样的，然而内部的实现有可能是不一样的。我们只需要关注ADT为我们提供的抽象, 即Queue中的元素遵循先进先出的原则，而不需考虑不同实现的差异，减少了开发过程中的心智负担。

  实际上对于上述代码，new 关键字还是依赖了具体的实现类，比如 new ArrayQueue<>()依赖了ArrayQueue类的存在，如果ArrayQueue不存在代码无法通过编译。实际中可以使用工厂模式绕过new 对具体类的依赖。例如使用工厂模式后的代码可能变成以下形式。

  ```java
  public static void doSomething(QueueFactory queueFactory){
      Queue<Integer> queue = queueFactory.getInstance();
      queue.push(Integer.valueOf(1));
      queue.push(Integer.valueOf(2));
      Optional<Integer> element = queue.pop();
      System.out.println(element.get()); // Should output "1";
  }
  // ============================================ //
  public interface QueueFactory<L> {
      public Queue<L> getInstance();
  }
  ```

  对于Queue的不同实现类，例如ListQueue、ArrayQueue等等可以为它们实现QueueFactory的接口，通过getInstance方法返回它们的一个实例，这样上述的代码就不再依赖具体的Queue实现类，当需求变化需要改变Queue的实现类时就不需要改这段代码了。

## 数据类型的可变性

数据类型的是否可变的维度可以分为可变(**Mutable**)数据类型的不可变(**Immutable**)数据类型。

- 可变数据类型

  可变数据类型如字面意思，其内部数据可以发生变化。例如上面我们提到的Queue, 当使用push和pop操作时必然会导致内部的数据发生变化以表示对象发生的变化。一般编程语言中的容器类型基本上属于可变数据类型。

- 不可变数据类型

  不可变数据类型其内部的数据不可以发生变化，例如Java中的String类型或者C++中的string等。不可变类型的基本特点是构造完成后内部数据不可变。

数据类型提供的方法在数据操作的维度一般可以分为四类：构造器(**Creator**)、生产器(**Producer**)、观察器(**Observer**)和变值器(**Mutator**)。

- 构造器

  构造器返回一个类型的实例，例如构造函数，返回实例的静态方法等。

- 生产器

  生产器通过原有实例生成新的实例，而不是修改原有实例的数据，例如Java String中的concat方法。

- 观察器

  观察器读取类型的数据，例如容器的大小等 。

- 变值器

  变值器修改类型的数据，例如Queue中的push和pop操作等。

这里将数据类型进行可变性的分类主要帮助我们引入下面的表示泄漏。



## 表示泄漏

数据类型的表示(**Representation**)指的是使用了哪种数据结构来表示一个类型。例如上面的ListQueue中我们在ListQueue的实现中使用了List作为队列的表示，而像ArrayQueue、MapQueue等等可能使用了其他的数据结构对Queue进行表示, 比如数组、Map等等。这意味着一个数据类型可以有多种表示。上面我们提到的表示独立性其中“表示"的含义也与此相同，指的是ADT的操作与实现ADT的数据类型是相互独立的。下面是表示泄漏对数据破坏的例子。

```java
// 表示一个钱包，里面有一定的金额
class Wallet {
    public int Mmoney;
};
// =================================== //
/** 表示一个人，其拥有钱包，并且可以通过
 *  showUp()方法炫耀自己的钱包
 *  也可以通过getMoney()方法努力打工，
 *  从老板手里拿到一点钱
 */
class Person {
    private Wallet Mwallet = new Wallet();
    
    public Wallet showUp() {
        return Mwallet;
    }
    
    public void getMoney(int money) {
        Mwallet.Mmoney = money;
    }
};
// =================================== //
/**
 * 表示一个小偷，主要通过stealMoney方法偷别人
 * 钱包里的钱
 */
class Thief {
    public Wallet Mwallet = new Wallet();
    
    public void stealMoney(Wallet wallet) {
        Mwallet.Mmoney += wallet.Mmoney;
        wallet.Money = 0;
    }
}
// =================================== //
// 某一天
public void oneDay(Person wang, Thief thief){
    // 小王刚赚了100块钱
    wang.getMoney(100);
    // 他炫耀了一下他的钱包
    Wallet wallet = wang.showUp();
    // 被小偷瞅准时机，拿到钱包，偷走了钱
    thief.stealMoney(wallet);
    // 小王一看钱包没钱了
    System.out.println(wang.Mwallet.Mmoney);
}
```

上面小王只是想炫耀一下钱包，但是钱包里的钱却被偷走了。这里因为Person类发生了表示泄漏使外部类可以修改内部数据，这是小王不希望的。有什么方法能让小王调用showUp方法炫耀钱包又不会被偷吗?

在设计Wallet类时应该考虑，Wallet为可变类型还是不可变类型。如果使用可变类型，showUp时应当返回Wallet的一个拷贝而不是直接返回内部实例。如果使用不可变类型，thief自然不能修改Wallet的内部数据，但是Person类的getMoney方法应该产生一个Wallet的新的实例来表示更多的钱。

其次，对于Wallet内部的数据Mmoney不应该直接操作，而是定义一组操作，这样能一定程度上防止表示泄漏。
