里式替换原则（LSP：Liskov Substitution Principle）
1986 Barbara Liskov
If S is a subtype of T, then objects of type T may be replaced with objects of type S, without breaking the program。

1996 Robert Martin
Functions that use pointers of references to base classes must be able to use objects of derived classes without knowing it。

子类对象（object of subtype/derived class）能够替换程序（program）中父类对象（object of base/parent class）出现的任何地方，并且保证原来程序的逻辑行为（behavior）不变及正确性不被破坏。

LSP与多态之间的区别

虽然从定义描述和代码实现上来看，多态和里式替换有点类似，但它们关注的角度是不一样的。
多态是面向对象编程的一大特性，也是面向对象编程语言的一种语法。它是一种代码实现的思路。
而里式替换是一种设计原则，是用来指导继承关系中子类该如何设计的，子类的设计要保证
在替换父类的时候，不改变原有程序的逻辑以及不破坏原有程序的正确性。

“Design By Contract”，：“按照协议来设计”。子类在设计的时候，要遵守父类的行为约定
（或者叫协议）。父类定义了函数的行为约定，那子类可以改变函数的内部实现逻辑，但不能改变函数
原有的行为约定。这里的行为约定包括：函数声明要实现的功能；对输入、输出、异常的约定；
甚至包括注释中所罗列的任何特殊说明。实际上，定义中父类和子类之间的关系，
也可以替换成接口和实现类之间的关系。


1.子类违背父类声明要实现的功能
父类中提供的 sortOrdersByAmount() 订单排序函数，是按照金额从小到大来给订单排序的，
而子类重写这个 sortOrdersByAmount() 订单排序函数之后，是按照创建日期来给订单排序的。
那子类的设计就违背里式替换原则。
2. 子类违背父类对输入、输出、异常的约定
在父类中，某个函数约定：运行出错的时候返回 null；获取数据为空的时候返回空集合
（empty collection）。而子类重载函数之后，实现变了，运行出错返回异常（exception），
获取不到数据返回 null。那子类的设计就违背里式替换原则。在父类中，某个函数约定，
输入数据可以是任意整数，但子类实现的时候，只允许输入数据是正整数，负数就抛出，也就是说，
子类对输入的数据的校验比父类更加严格，那子类的设计就违背了里式替换原则。
在父类中，某个函数约定，只会抛出 ArgumentNullException 异常，
那子类的设计实现中只允许抛出 ArgumentNullException 异常，任何其他异常的抛出，
都会导致子类违背里式替换原则。
3. 子类违背父类注释中所罗列的任何特殊说明
父类中定义的 withdraw() 提现函数的注释是这么写的：“用户的提现金额不得超过账户余额……”，
而子类重写 withdraw() 函数之后，针对 VIP 账号实现了透支提现的功能，
也就是提现金额可以大于账户余额，那这个子类的设计也是不符合里式替换原则的。

判断子类的设计实现是否违背里式替换原则：拿父类的单元测试去验证子类的代码。
如果某些单元测试运行失败，就有可能说明，子类的设计实现没有完全地遵守父类的约定，
子类有可能违背了里式替换原则。