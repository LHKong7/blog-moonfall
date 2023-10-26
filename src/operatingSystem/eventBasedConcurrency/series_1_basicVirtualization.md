## 探索基于事件的并发---操作系统中的虚拟化

### TL;DR
浏览器运行时和nodejs（服务端框架）在编程语言中都使用了JavaScript这个脚本式，单线程的语言，乍一看这种语言对比 Golang，Java，这种可以编译后运行的语言在性能上是无法对比的。但如果多观察，nodejs作为服务端框架在运行时有着自己的优势，在一些场景下的性能比Java和Golang更要优秀。而这一切并不是仅仅局限于单一语言。而是基于事件并发赋予了框架更深层次的力量。

### 前言
操作系统相关的话题写起来都比较有难度，而且出现错误很容易误人子弟。如果文章中出现定义错误或不清晰的地方，也希望阅读的同好能指出来，必定尽快修改与解决，也希望能与大家讨论交流。

### 操作系统 & 虚拟化
操作系统作为软件开发的基石，为使用者提供了很多的便利与好处，而最主要的一条就是虚拟化（Virtualization），将物理资源（CPU，内存卡，硬盘）转换为更通用，比较容易使用的虚拟化的模式。所以也可以将操作系统定义为虚拟机（virtual machine）。操作系统也定义了**系统调用（system call）** 使应用程序可以直接使用，例如，在UNIX/LINUX操作系统中常见的 `signal()`, 就是一个系统调用，而用户可以在终端中输入 `man signal` 来查看相关的定义。之所以虚拟化重要，就是因为在人体感官上，虚拟化使硬件资源（例如，CPU）看起来是共享且“同时进行”的，让软件开发更多样性，提高了软件商业化的可能性。

通过简单的对操作系统的虚拟化进行解释，希望对了解接下来的理论有着更好的帮助，在开始并发和基于事件模型的并发开始前，让我们先了解一些简单的操作系统知识：
- CPU虚拟化：操作系统通过分时手段（time-sharing）把有限的CPU资源利用到了最大化，让用户感受不到其中的操作切换与启动；分时手段： 是资源的共享方式，将一份资源共享给多个任务或者用户，分时系统允许一个用户操作多个任务或者多个用户周期，而达到其中的奥秘在广义上则需要底层与上层的相互实现：
  - 底层：上下文切换（Context Switch）
    - 定义
    - scheduling policy
  - 上层：进程（process），为了满足能够进行上下文切换，我们需要一种抽象化的数据结构来让所有被切换的程序都有着相同的属性，而这些数据结构在操作系统中通过链表连接起来从而更便于管理。针对于前端开发，可以考虑为VDOM的抽象化数据结构。
    - 定义：一个跑在操作系统的程序
    - 组成部分：
      - machine state：一个程序可以读取和更新的部分，包含了
        - 进程内存：其中也包括了指令，数据，地址（address space）
        - registers：
          - program counter PC：那条指令会在下一步之行
          - stack pointer 和相对应的frame pointer：管理栈中的函数参数，本地变量和返回地址
          - 有时也包含 IO信息。
    - 操作系统提供的进程通用API，无论是UNIX，Linux类，windows等不同的操作系统都会暴露出相关的内容：
      - 创建进程：进程创建/程序转换为进程的过程：
        - 1. 从硬盘中读取可执行文件的代码。 
        - 2. 读取代码和静态数据（初始化变量）到内存中/进程中的地址空间。 TL；DR：现代操作系统会有懒加载进程的操作，有兴趣的同学可以了解 paging 和 swapping（内存虚拟化）
        - 3. 为运行栈分配空间（stack/run-time stack）对于C程序来讲，stack for本地变量，函数参数和返回地址都会在栈中
        - 4. 为堆分配空间，堆一般是用来存储动态数据结构的，比如链表，树状结构，开始时堆的空间会比较小，随着数据的丰富，堆会不断的变大
        - 5. 初始化操作：IO初始化
        - 6. 进程/程序开始运行
      - 销毁进程
      - 等待进程
      - 管理进程
      - 当前进程状态获取：
        - 进程的状态：
          - Running: 当前进程正在CPU中被执行
          - Ready：进程可以被执行，但当前还未被执行
          - Blocked：进程正在进行操作直到一些事件发生前无法被执行
    - 分时系统的挑战：
      - performance
      - control




- 内存（memory）虚拟化


