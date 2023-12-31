## V8编译流水线

在执行JavaScript之前，V8就准备好了代码的运行时环境，这个环境包括了：
- 堆空间
- 栈空间
- 全局执行上下文
- 全局作用域
- 内置的内建函数
- 宿主环境提供的扩展函数和对象
- 消息循环系统

准备好运行时环境之后，V8才可以执行JavaScript代码，这包括：解析源码，生成字节码，解释执行或编译执行这一系列操作。

整个流水线：
运行时环境 --> 解析 --> 生成字节码 --> 解释执行 --> 编译执行

### 运行时环境

#### V8的宿主环境

**宿主环境与V8的关系**：V8的核心在于实现了ECMAScript标准，提供了垃圾回收器，协程等基础内容。而浏览器的渲染进程则是V8的宿主环境，为V8提供基础的消息循环系统，全局变量，Web API等

**构造数据存储空间：堆空间和栈空间**： 
在 Chrome 中，只要打开一个渲染进程，渲染进程便会初始化 V8，同时初始化堆空间和栈空间。
- 栈空间：主要用来管理JavaScript函数调用，为内存中连续的一块空间，FILO的执行顺序，在函数调用过程中，涉及到上下文相关的内容都会存放在栈上，比如原生类型，引用到的对象的地址，函数执行状态，this值等，当一个函数执行结束，该函数的执行上下文便会被销毁掉。
  - 栈空间的最大的特点是空间连续，所以在栈中每个元素的地址都是固定的，因此栈空间的查找效率非常高，但是通常在内存中，很难分配到一块很大的连续空间，因此，V8 对栈空间的大小做了限制，如果函数调用层过深，那么 V8 就有可能抛出栈溢出的错误。
- 堆空间：一种树形的存储结构，用来存储对象类型的离散的数据。比如函数，数组，window对象，document对象等等。

**全局执行上下文和全局作用域**：初始完基础的存储空间后，V8会初始化全局执行上下文和全局作用域，当V8开始执行一段可执行代码时，会生成一个执行上下文。V8用执行上下文来维护执行当前代码所需要的变量声明，this指向等。执行上下文主要包含了三个部分，变量环境，词法环境和this关键字。
- 全局执行上下文会在V8的生存周期内不会被销毁，会一直保存在堆中。当执行了一段全局代码时，如果全局代码中有声明函数或定义的变量，那么函数对象和声明的变量会添加到全局执行上下文中

**构造事件循环系统**： V8需要一个主线程用来执行JavaScript和执行垃圾回收等工作。V8所执行的代码都是在宿主的主线程上执行的。因为所有的任务都是运行在主线程的，在浏览器的页面中，V8 会和页面共用主线程，共用消息队列，所以如果 V8 执行一个函数过久，会影响到浏览器页面的交互性能。





### 解释执行

函数通常有两个主要的特性：

1. 可以被调用
2. 具有作用域机制： 指函数在执行的时候可以将定义在函数内部的变量和外部环境隔离，在函数内部定义的变量我们也称为临时变量，临时变量只能在该函数中被访问，外部函数通常无权访问，当函数执行结束之后，存放在内存中的临时变量也随之被销毁。

栈是如何管理函数调用：在函数执行过程中，其内部的临时变量会按照执行顺序被压入到栈中。

**在函数A中调用函数B，如何恢复到A函数？** 只要在寄存器中保存一个永远指向当前栈顶的指针





### 编译执行 （二进制机器码如何被CPU执行的）

当有了运行时环境，V8就可以执行JavaScript代码了，首先需要将JS编译成字节码，然后再解释执行字节码，或者将需要优化的字节码编译成二进制，并直接执行二进制代码。

CPU 可以通过指定内存地址，从内存中读取数据，或者往内存中写入数据，有了内存地址，CPU 和内存就可以有序地交互。同时，从内存的角度理解地址也是非常重要的，这能帮助我们理解后续很多有深度的内容。内存中的每个存储空间都有其对应的独一无二的地址。一旦二进制代码被装载进内存，CPU 便可以从内存中取出一条指令，然后分析该指令，最后执行该指令。我们把**取出指令、分析指令、执行指令这三个过程称为一个 CPU 时钟周期**。CPU 是永不停歇的，当它执行完成一条指令之后，会立即从内存中取出下一条指令，接着分析该指令，执行该指令，CPU 一直重复执行该过程，直至所有的指令执行完成。
- CPU中的 PC寄存器：保存了将要执行的指令
- 通用寄存器：CPU用来存放数据的设备。
  - esp 寄存器： 栈顶指针的作用就是告诉你应该往哪个位置添加新元素
  - ebp 寄存器：用来保存当前函数的起始位置，我们把一个函数的起始位置也称为栈帧指针，ebp 寄存器中保存的就是当前函数的栈帧指针

通用寄存器容量小，读写速度快，内存容量大，读写速度慢。
