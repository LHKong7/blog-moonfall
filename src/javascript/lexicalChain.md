## 作用域链与语法环境 Scope and Lexical Environment

作用域链是将一个个作用域串起来，实现变量查找的路径。作用域是存放变量和函数的地方，全局环境有全局作用域，全局作用域存放了全局变量和全局函数，每个函数也有自己的作用域存放着函数中定义的变量
当在函数内部使用一个变量的时候，V8 便会去作用域中去查找

全局作用域是在 V8 启动过程中就创建了，且一直保存在内存中不会被销毁的，直至 V8 退出。 而函数作用域是在执行该函数时创建的，当函数执行结束之后，函数作用域就随之被销毁掉了。

在全局作用域中，通常包含了：
- this
- window
- document
- Web API
V8在编译过程中，会将顶层定义的变量和声明的函数都添加到全局作用域中。全局作用域创建完成后，V8便进入了执行状态。

因为 JavaScript 是基于词法作用域的，词法作用域就是指，查找作用域的顺序是按照函数定义时的位置来决定的。bar 和 foo 函数的外部代码都是全局代码，所以无论你是在 bar 函数中查找变量，还是在 foo 函数中查找变量，其查找顺序都是按照当前函数作用域–> 全局作用域这个路径来的。
