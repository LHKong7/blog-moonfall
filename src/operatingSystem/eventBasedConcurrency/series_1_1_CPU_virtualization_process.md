进程（process），为了满足能够进行上下文切换，我们需要一种抽象化的数据结构来让所有被切换的程序都有着相同的属性，而这些数据结构在操作系统中通过链表连接起来从而更便于管理。针对于前端开发，可以考虑为VDOM的抽象化数据结构。
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


### 进程例子：
