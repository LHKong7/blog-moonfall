# Java Crash Course

Classes 是 Java程序中的基础组成部分，在一个类中由三个组成部分：
- constructor: 构造函数，用来创建一个新的对象
- method：方法/函数
- field：每个对象中的实例：
  - *public*:
  - *private*:

`this` : 正在调用函数的对象

在 Java 中，针对与对象和原始类型的函数传参都是 `call by value` 

对象三大特征：
- 状态（state）：当前对象状态
- 行为（behavior）：函数来定义用来表示状态切换
- 个性（Identity）：每个class都是一个独立的个体

如何定义好一个 class：
- simple rule of thumb：通过名词（nouns）来定义一个类，但不要太细化（举例来说，厨房用品，并不需要定义Toaster，洗碗机等等类，商品即可表达全部意思）和太通用，要定义好能解决问题的类。

通过这个类的动词来解决问题，多问自己：“How can an object of this class
possibly carry out this responsibility?” 

类一般会有三种关系：
- Dependency (“uses”) ： A class depends on another class if it manipulates objects of the other
class in any way
    我们在实际开发中应该减少依赖，比如Java中的 `System.out.println` 针对于嵌入式系统可能并不存在 `System` 这个API，因此我们减少这类使用或者定义一个更通用的设计。
- Aggregation(聚合) (“has”) ： A class aggregates another if
its objects contain objects of
the other class.
    
- Inheritance (“is”) ：A class inherits from another if
it incorporates the behavior of
the other class.




