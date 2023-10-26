## 对象和函数

### 对象

在JavaScript中对象的属性有三个值：
对象是由属性（`property`）和值（`value`）组成的复杂数据结构。

- 原始类型（primitive）： 本身无法被改变，比如 JavaScript 中的字符串就是原始类型，如果你修改了 JavaScript 中字符串的值，那么 V8 会返回给你一个新的字符串，原始字符串并没有被改变，我们称这些类型的值为“原始值”。
- 对象类型（Object）：对象的属性值也可以是另外一个对象
- 函数（Function）：如果对象中的属性值是函数，那么我们把这个属性称为方法

### 函数
在 JavaScript 中，函数是一种特殊的对象，它和对象一样可以拥有属性和值，但是函数和普通对象不同的是，函数可以被调用。
在V8内部，函数对象会有其他隐藏的属性：
- name：函数名字
- code：表示函数代码
- prototype


如果某个编程语言的函数，可以和这个语言的数据类型做一样的事情，我们就把这个语言中的函数称为一等公民。支持函数是一等公民的语言可以使得代码逻辑更加清晰，代码更加简洁。相同地，也会带来一些副作用，当函数中使用的变量在函数外部时，该函数会沿着作用域链去外部的作用域中查找该变量:
```js
function foo(){
    var number = 1
    function bar(){
        number++
        console.log(number)
    }
    return bar
}
var mybar = foo()
mybar()
```
我们在 foo 函数中定义了一个新的 bar 函数，并且 bar 函数引用了 foo 函数中的变量 number，当调用 foo 函数的时候，它会返回 bar 函数。那么所谓的“函数是一等公民”就体现在，如果要返回函数 bar 给外部，那么即便 foo 函数执行结束了，其内部定义的 number 变量也不能被销毁，因为 bar 函数依然引用了该变量。我们也把这种将外部变量和和函数绑定起来的技术称为闭包


### V8中的对象

#### 对象中的快慢属性：
```js
function Foo() { 
    this[100] = 'test-100'; 
    this[1] = 'test-1'; 
    this["B"] = 'bar-B'; 
    this[50] = 'test-50';
    this[9] = 'test-9';
    this[8] = 'test-8';
    this[3] = 'test-3';
    this[5] = 'test-5';
    this["A"] = 'bar-A';
    this["C"] = 'bar-C'
}
var bar = new Foo()
for (key in bar) { console.log(`index:${key} value:${bar[key]}`) }

/**
    index:1 value:test-1
    index:3 value:test-3
    index:5 value:test-5
    index:8 value:test-8
    index:9 value:test-9
    index:50 value:test-50
    index:100 value:test-100
    index:B value:bar-B
    index:A value:bar-A
    index:C value:bar-C
 * /
```
在打印中可以发现，打印出来的属性顺序并不是我们设置的顺序，设置为数字的属性被最先打印出来了，并且按照数字大小的顺序打印的，设置的字符串属性是按照之前的设置顺序打印的。
出现上面的情况是因为在 ECMAScript中规范定义了数字属性按照索引值大小升序排列，字符串属性根据创建时的顺序升序排列。在V8中命名属性有三种不同的存储方式：
- 常规属性（properties）：字符串属性为常规属性
- 排序属性（element）：对象中的数字属性
在V8内部，为了能够有效的提升存储和访问这两种属性的功能，分别使用了两个线性结构来分别保存排序属性和常规属性。我们在执行**遍历对象或获取索引操作时，V8会先从`elements`属性中按照顺序读取所有的元素，然后再从 `properties`属性中读取所有的元素，这样就完成了一次缩阴操作**
将不同的属性保存在不同的属性对象中，在查找时，会多一步操作来获取具体属性对象中（propertioes/element）的属性，为了减少这个副作用，V8将部分常规属性（properties）直接存储到对象本身，这一部分也就是对象内属性（in-object properties）。这样再次进行查找时，可以直接访问对象内的属性而不是先查找常规属性再查找该属性。不过对象内的属性的数量是固定的（10个）。如果添加的属性超出了对象分配的空间，则它们将被保存在常规属性存储中。虽然属性存储多了一层间接层，但可以自由地扩容。这也引出了快属性和慢属性的概念：
- 快属性：保存在线性数据结构中的属性，因为线性数据结构中只需要通过索引即可以访问到属性，访问线性结构的速度快，但是如果从线性结构中添加或者删除大量的属性时，则执行效率会非常低
- 慢属性：当一个对象属性过多时，采用慢属性策略。慢属性的对象内部会有独立的非线性数据结构 (词典) 作为属性存储容器。所有的属性元信息不再是线性存储的，而是直接保存在属性字典
- 对象内属性（in-object）：保存在对象本身，提供最快的访问速度。
***为什么要分三种存储方式***：
所有的属性在底层都会表示为二进制，如果程序逻辑只涉及二进制的位运算（包含与、或、非），速度是最快的。慢属性在查找时需要先经过哈希算法计算，这是一个复杂运算，时间上若干倍于简单位运算。另外，哈希表是个二维空间，所以通过哈希算法计算出其中一维的坐标后，在另一维上仍需要线性查找。所以，当属性非常少的时候为什么不用慢属性应该就不难理解了吧。
V8哈希算法：
```c++
// V8 中字符串的哈希值生成器
uint32_t StringHasher::GetHashCore(uint32_t running_hash) {
  running_hash += (running_hash << 3);
  running_hash ^= (running_hash >> 11);
  running_hash += (running_hash << 15);
  int32_t hash = static_cast<int32_t>(running_hash & String::kHashBitMask);
  int32_t mask = (hash - 1) >> 31;
  return running_hash | (kZeroHash & mask);
}
```

#### 函数表达式的处理
函数表达式：
```js
foo = function (){
    console.log('foo')
}
```
与函数声明的区别：
- 函数表达式是在表达式语句中使用 function 的，最典型的表达式是“a=b”这种形式，因为函数也是一个对象，我们把“a = function (){}”这种方式称为函数表达式；
- 在函数表达式中，可以省略函数名称，从而创建匿名函数（anonymous functions）；
- 一个函数表达式可以被用作一个即时调用的函数表达式——IIFE（Immediately Invoked Function Expression）。

IIFE(立即调用的函数表达式): 因为函数立即表达式也是一个表达式，所以 V8 在编译阶段，并不会为该表达式创建函数对象。这样的一个好处就是不会污染环境，函数和函数内部的变量都不会被其他部分的代码访问到。在 ES6 之前，JavaScript 中没有私有作用域的概念，如果在多人开发的项目中，你模块中的变量可能覆盖掉别人的变量，所以使用函数立即表达式就可以将我们内部变量封装起来，避免了相互之间的变量污染。

