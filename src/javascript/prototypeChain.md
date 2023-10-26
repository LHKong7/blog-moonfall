## JS中的原型链与继承 Prototype Chain & Inheritance


### 什么是原型 __proto__
`__proto__` 是JavaScript中的隐藏属性，这个属性是该对象的原型（prototype），`__proto__` 指向了内存地址中的另外一个对象（该对象的原型对象），这样该对象就可以直接访问其原型对象的属性和方法。

### 原型链：
当访问一个对象的属性没有找到时，V8会沿着这个对象的 `__proto__` 进行查找， 这条被查找的路径叫做原型链。

总结：JavaScript中的继承非常简洁，每个对象都有一个原型属性，该属性指向了原型对象，查找属性的时候，JavaScript虚拟机会沿着原型一层一层向上查找，直到找到正确的属性。

### JavaScript中的继承，利用 `__proto__` 实现
当有对象 A，B时，简单的继承则直接通过将对象的 `__proto__` 设置为被继承对象即可：
```js
var a = {
    foo: "foo",
    bar: "bar"
}
var b = {
    baz: "baz"
}
console.log(b.baz); // baz
console.log(b.foo); // undefined
b.__proto__ = a; // 直接通过改变__proto__属性展示继承
console.log(b.foo); // foo

```
但这种方法并不应该被使用，这里的例子只是展示继承的简单以及更便于理解。主要基于两点：
1. ` __proto__` 是隐藏属性，不应该直接修改
2. 会造成性能问题，隐藏类的优化措施优化过了对象，当修改 proto的属性指向，相当于要重建整个隐藏类，必然会影响性能


### new操作符
当我们需要创建一个新的JavaScript对象时，通常会使用 `new` 关键字 + 构造函数来创建，在执行 `new` 操作符时，V8做了这几件事情：
1. 创建一个空的 JavaScript对象 `{}`
2. 添加 `__proto__` 属性，并将该属性指向构造函数的原型对象 （`newObj.__proto__ = calledConstructor.prototype`）
3. 将新对象作为 `this` 的上下文
4. 如果该函数没有返回对象，返回 `this`
```js
// 模拟代码
function DogFactory(type,color){ this.type = type this.color = color}
var dog = new DogFactory('Dog','Black')
// new操作符
var dog = {}  
dog.__proto__ = DogFactory.prototype
DogFactory.call(dog,'Dog','Black')
```

### Prototype 属性
在V8对函数的定义中，除了 `name` (函数名) 和 `code` (代码块) 的属性外，还有一个 `prototype`的隐藏属性，每个函数对象中都有 `prototype`的属性，当将这个函数作为构造函数来创建一个新的对象时，新创建对象的原型对象就指向了该函数的 `prototype`属性。所以当使用 `new` 操作符来创建对象时，被创建的对象中的 `__proto__`都指向了构造函数中的 `prototype`属性。

### 继承：
在JavaScript中实现继承的方法比较简单，在对象中添加了一个称为原型的属性，把继承的对象通过原型链连接起来，就实现了继承；我们把这种继承方式称为基于原型链继承。




