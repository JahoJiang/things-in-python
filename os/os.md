### 1. 虚拟化 <div id="virtualization"/>

虚拟化是为了让操作系统更易用。



#### 1.1 虚拟化 CPU <div id="virtual-cpu"/>

虚拟化 CPU 的目的是为了更好的利用 CPU，例如，产生多道程序“同时”运行在单个 CPU 核的假象。



##### 进程 <div id="process"/>

进程就是运行中的程序，是操作系统为正在运行的程序提供的抽象，是资源分配和调度的单位。

因此，进程的机器状态（组成）：

1. 内存。进程要执行的指令存在内存中，读取和写入的数据也在内存中，因此进程可访问的内存是该进程的一部分。

2. 寄存器。许多指令会明确的读取和更新寄存器，如：程序计数器，告诉进程执行哪个指令；栈指针和相关的帧指针，管理函数参数栈、局部变量和返回地址。



**进程如何被操作系统创建** <div id="process-creation"/>

    ——————————————————
    |                |
    |       代码     |
    |     静态数据    |
    |       堆       |
    |                |
    |                |
    |                |
    |                |
    |_______栈_______|
        
        进程内存示意图

1. 将代码和所有静态数据加载到内存中，加载到进程的地址空间

2. 程序使用栈存放局部变量，函数参数和返回地址。操作系统分配这些内存，提供给进程。

3. 操作系统也为程序的堆分配一些内存，主要是用于显式请求的动态分配数据。    



**进程状态** <div id="process-status"/>

1. 创建状态(new) ：进程正在被创建，尚未到就绪状态。

2. 就绪状态(ready) ：进程已处于准备运行状态，即进程获得了除了处理器之外的一切所需资源，一旦得到处理器资源(处理器分配的时间片)即可运行。

3. 运行状态(running) ：进程正在处理器上上运行(单核CPU下任意时刻只有一个进程处于运行状态)。

4. 阻塞状态(waiting) ：又称为等待状态，进程正在等待某一事件而暂停运行如等待某资源为可用或等待 IO 操作完成。即使处理器空闲，该进程也不能运行。

5. 结束状态(terminated) ：进程正在从系统中消失。可能是进程正常结束或其他原因中断退出运行。



**数据结构 PCB ** <div id="pcb"/>

PCB，是操作系统存储进程的描述信息所用的数据结构。

PCB 的内容主要有：

1. 标示符。描述本进程的唯一标示符，用来区别其他进程
2. 状态。任务状态，退出代码，退出信号等。
3. 优先级。相对于其他进程的优先级。
4. 程序计数器。程序中即将被执行的下一条指令的地址。
5. 内存指针。包括程序代码和进程相关数据的指针，还有和其他进程共享的内存块的指针
6. 上下文数据。进程执行时处理器的寄存器中的数据
7. I／O状态信息。包括显示的I/O请求,分配给进程的I／O设备和被进程使用的文件列表。
8. 记账信息。可能包括处理器时间总和，使用的时钟数总和，时间限制，记账号等



**进程 API** <div id="process-api"/>

1. fork()

   调用 fork() 创建的子进程与父进程几乎一样，但是父进程中，fork()返回的是子进程的 PID，子进程返回的是 0，由此可以区分以编程。子进程有自己的地址空间。

2. wait()

   父进程 调用 wait() 等待子进程结束后，再继续执行

3. exec()

   fork() 适用于让子进程运行相同程序的拷贝，如果想要运行不同的程序，则可以使用  exec()，先通过 fork() 创建子进程，在子进程代码区 使用 exec() 加载其它的可执行文件。



**进程调度算法** <div id="process-algorithm"/>

1. FIFO：先进先出

   缺点：一些耗时较少的进程被排在重量级进程之后，得不到及时的执行。（护航效应）

2. SJF：最短任务优先

   缺点：只有进程同时到达才能评估并达到理想效果，但是现实不是这样，对于后来的进程，仍有可能出现（护航效应）。

3. STCF：最短完成时间优先

   向 SJF 增加抢占，每当新的进程进入，比较当前进程的剩余时间和新进程的剩余时间。

4. Round-Robin：轮转

   在一个时间片内运行一个进程，然后切换到运行队列中的下一个进程。

   缺点：假如时间片太短，上下文切换的开销带来的影响会增大。

5. MLFQ：多级反馈队列

    基本规则：存在多个运行队列，每个队列有不同的优先级。优先执行高优先级的进程，同一队列进行轮转。

    那么，如何设置优先级？

    答：进程的优先级是动态的，例如，一个工作不断放弃 CPU 去等待用户的I/O，则可能是交互型进程，会让它保持较高的优先级。假如一个进程长时间占有 CPU，则 MLFQ 有可能会降低其优先级。

    规则：


    1. 进程 A > 进程 B 的优先级，则运行 A
    2. 进程 A = 进程 B 的优先级，则轮转运行 A 和 B
    3. 新的进程进入系统，置为最高优先级
    4. 进程使用完一个时间片，降低优先级，在此之前主动放弃 CPU 则保持优先级不变
    5. 经过一段时间 S，当前所有工作都会重新进入最高优先级队列。避免饥饿的问题。如果 S 太高，长的工作会饥饿，太低则 I/O 进程得不到合适的 CPU 时间比例。
    6. 记录一个进程在某一层消耗的总时间，避免被进程愚弄（总在时间片结束前主动放弃）。对进程每一层的执行时间施行配额，用完了配额就一定降低优先级

    用例：

    1. 如果来了一个进程：如果是短的进程，则会在被移入最低优先级之前运行完毕，否则会被慢慢降入低优先级。
    2. I/O 密集型进程：这样的进程经常会在时间片结束前主动放弃CPU，不降低它的优先级而是保持




**多处理器调度下的问题** <div id="multi-processor"/>

多处理器架构：多处理器共享内存的问题主要是缓存的问题。

**缓存** ： 缓存是基于**局部性**的概念，分为时间局部性和空间局部性。时间局部性是指，当一个数据被访问后，很有可能不久后再次访问；空间局部性是指，访问地址为 x 的数据后，有可能接着访问 x 周围的数据。

1. 缓存一致性

   多 CPU 下的缓存，可能会出现**缓存一致性**问题，例如 CPU1 上的程序从内存地址 A 中读取数据，由于不再 CPU1 的缓存中，所以系统直接访问内存，得到值 D1，程序随后在缓存中将其更新为值 D2，还没来得及写入内存，这时系统中断了程序执行，将其交给 CPU2，这时只会读取到旧值。

   

   如何解决：

   答：缓存一致性协议。

   硬件层面的缓存一致性协议（Cache Coherence Protocol），最出名的就是Intel 的MESI协议，MESI协议保证了每个缓存中使用的共享变量的副本是一致的。

   MESI的核心的思想是：当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

   在MESI协议中，每个缓存可能有有4个状态，它们分别是：

   M(Modified)：这行数据有效，数据被修改了，和内存中的数据不一致，数据只存在于本Cache中。

   E(Exclusive)：这行数据有效，数据和内存中的数据一致，数据只存在于本Cache中。

   S(Shared)：这行数据有效，数据和内存中的数据一致，数据存在于很多Cache中。

   I(Invalid)：这行数据无效。


​    
2. 缓存亲和度

   一个进程在某个 CPU 上运行，会在该 CPU 的缓存中维护许多状态，下次在相同的 CPU 上运行时才会运行的更快。



3. 多队列调度

   为每个 CPU 单独维护调度队列。工作量较少的队列不定期“偷看”其他队列是否比自己工作量少多，是的话就“偷窃”一个或多个工作，实现负载均衡，这个技术叫做工作窃取。



#### 8.2 虚拟化内存 <div id="virtual-memory">

虚拟化内存，是为了程序更好更容易的使用内存，主要目标为：

1. 透明。使运行的程序不能察觉到内存被虚拟化。哪怕是 C 指针的地址也是虚拟的。

2. 效率。

3. 保护。确保进程收到保护，不受其它进程的影响。

操作系统需要提供一个易用的物理内存抽象，这个抽象叫做“地址空间”，是运行的程序所看到的系统中的内存。

一个进程的地址空间，包含程序指令（代码），位于地址空间的顶部。其次是堆（顶部）管理动态分配的、用户管理的内存，栈（底部）保存当前函数的调用信息，分配空间给局部变量，传递参数和函数返回值。


**内存使用** <div id="memory-usage"/>

1. double* ptr = (double*) malloc(10 * sizeof(double)) 传入要申请的堆空间的大小，返回指向新空间的指针。
2. free(ptr) 传入要释放的内存的指针。



**地址转换** <div id="address-translation"/>

既然进程使用的是虚拟地址，则就涉及基于硬件的从虚拟地址到物理地址的地址转换。操作系统需要在转换时，设置好硬件，以便正确转换，因此操作系统需要记录被占有的和空闲的内存位置。

这个硬件单元一般叫做 **MMU**。MMU(Memory Management Unit)主要用来管理虚拟存储器、物理存储器的控制线路，同时也负责虚拟地址映射为物理地址，以及提供硬件机制的内存访问授权、多任务多进程操作系统。

最简单的即为在MMU中引入两个寄存器：基址寄存器和界限寄存器。这样：物理地址 = 虚拟地址 + 基础地址。界限寄存器提供保护。



**分段** <div id="segment"/>

 不是只在 MMU 中引入一对基址寄存器和界限寄存器，还是给地址空间内每个逻辑段一对。一个段只是地址空间里的一个连续定长的区域，典型的地址空间3个逻辑不同的段：

 1. 代码段
 2. 栈
 3. 堆

 分段的机制使得操作系统可以将不同的段放到不同的物理内存区域，从而避免虚拟地址空间内未使用部分占用物理内存。所谓的 SegmentFault 段错误就是在分段的机器上发生了非法的内存访问。如今，在非分段的机器上，试图访问非法地址也会导致段异常或段错误。

 我们可以基于如此的虚拟地址进行转换：

 ——————————————————
｜ 段号            ｜         段内偏移量            ｜
 ——————————————————

 存在的问题：

  1. 在所有地址都保持一个增长方向时才可以，然而栈是反向增长的，因此它可能是从 32KB 增长回到 24KB。因此，**硬件还需记录段的增长方向**。
  2. 为了支持内存共享，那么每个段一个增加保护位，来标识这个程序是否能够读写该段，或执行其中的代码，抑或是只能进行读取。这样，同样的代码可以被多个进程共享，同时保证不被隔离。
 3. 分段带来的问题，进程上下午切换时，也应该进行段寄存器段的保存与恢复。


**分页** <div id="page"/>

分段技术将空间切成不同长度的分片以后，容易导致外部碎片，随着时间退役，分配内存会比较困难。

分页，不是将地址空间分割成几个不同长度的逻辑段，而是分割成固定大小的单元，每个单元称之为一页。相应地，我们把物理内存看成是定长槽块的阵列，叫做页帧。每个这样的页帧包含一个虚拟内存页。

**一个例子：**

假如，一个页大小 16 字节，操作系统需要将 64 字节的小地址空间放到 8 页的物理地址空间，只需要找到 4 个空闲页，将虚拟页映射到这4个物理页帧即可。为了为地址空间每个虚拟页保存地址转换，记录在物理内存中的实际位置，操作系统通常会为每个进程保存一个数据结构，成为**页表**。

页表是一个每进程的数据结构！

我们可以基于如此的虚拟地址进行转换：

 ——————————————————————
｜ 虚拟页号 ｜ 页内偏移量 ｜
 ——————————————————————

 在转换时，在页表中查询虚拟页号对应的实际的物理页面，就可以得到真实的物理地址。由于页表可以变得很大，因此不是存在 MMU 中，而是将每个进程的页表存在内存中。页表中的项称为页表项，有效位用于指示特定地址转换是否有效；保护位用于指示页是否可以读取、写入或执行；存在位表示该页在内存中还说物理磁盘中；脏位表示页面被加载进内存后是否被修改过。

 由于页表的存在，当页表很大时，地址转换会很慢。 -- **TLB**，快速地址转换。

 为了加快地址转换，增加了硬件 TLB。它就是对频繁发生的虚拟到物理地址转换的硬件缓存。对每次内存访问，硬件先检查 TLB，再检查页表。



 **TLB基本流程：**

 1. 从虚拟地址提取虚拟页号 VPN。
 2. 检查 TLB 中是否有该 VPN的转换映射。如果用，命中，快速完成转换
 3. 如果没有命中，查找页表寻找转换映射，更新 TLB，系统重新尝试该指令，得到处理。

 TLB 中的存储项结构：

 ————————————————————————————————————
｜ 虚拟页号VPN ｜ 物理页帧 PFN ｜ 其它位 ｜
 ————————————————————————————————————

 TLB是全关联的，硬件会并行的进行查找，因此速度很快。

 *TLB 中的有效位指示该映射是否有效，PTE 的有效位指示该页有没有被进程申请使用。*


​     

 **TLB替换策略：**

 1. LRU：最近最少使用


**多级页表** <div id="multi-level-page-table"/>

如何去掉页表中的所有无效区域，而不是将它们全部保留在内存中？使用多级页表将线性页表变成类似树结构。

多级页表的基本思想为：

1. 将页表分为页大小的单元

2. 如果整页的页表项无效，则完全不分配该页的页表

3. 使用页目录来追踪页表的页是否有效，页目录会记录页表的页的位置，以及页表的整个页是否包含有效页。

    多级页表是有成本的，TLB 未命中时需要加载两次，一次用于页目录，一次用于PTE本身。
    
    

**空闲内存管理** <div id="free-memory-management"/>

free() 的调用只需要传入指针，而不需要传入指针对应的空间大小，这就是因为操作系统对内存进行了管理。在堆上追踪空闲和已分配空间的数据结构通常被称为**空闲列表**（不一定是真的列表形式）。

如果要管理的空闲空间大小是不一致的，那将主要面临外部碎片问题。例如，总的空间 32KB 被已分配空间16KB 分割成两个 8 KB空间，那么一个分配16 KB 的请求就会失败。这样的情况常发生在用户级别的内存分配库或者操作系统使用分段的形式实现虚拟内存。

基本机制：

1. 分割与合并。需要分配内存时，分割满足要求的空闲内存区域；释放内存时，操作系统有时会其去其它空闲区域合并以满足较大的内存分配请求。
2. 追踪以分配空间的大小。大多数分配程序会在一个已分配的区域加上 Header，写入例如 Size 等管理空间所需的信息。
3. 当堆空间耗尽，再向操作系统申请更大的空间，通常是经过系统调用，例如 UNIX 系统中的 sbrk，让堆增长。一般是会找到空闲的物理页，将它映射到进程的地址空间去，并返回新的堆末尾地址。



**匹配策略**：

1. 最优匹配。每次遍历空闲列表，找到最合适的。

    优点：避免空间浪费；缺点：性能代价高

2. 最差匹配。与最优匹配相反，表现糟糕。


3. 首次匹配。返回第一个满足要求的块。

   优点：速度快；缺点：使得空闲列表开头外部碎片多

4. 下次匹配。不同于首次匹配从列表头开始查找，而是维护一个指针，记录上次匹配查找结束的位置。

   优点：和首次匹配性能相近，避免遍历查找。缺点：需要额外空间。



**伙伴系统**

将空闲空间看作2的幂次方大小的空间。每次分配，空闲空间都被一分为二，直到刚好满足要求。如果一块内存被释放，会检查伙伴释放被释放，是的话就合并并向上检查。



**上下文切换**  <div id="context-switch"/>

当操作系统决定切换一个进程，会为当前进程保存一些寄存器的值（例如，存到内核栈），并为即将执行的进程恢复一些寄存器的值，这个过程叫做上下文切换。由于 TLB 中的内容只对当前进程有效，因此在进行进程切换时：
1. 要么将 TLB 的内容清空（将有效位置为 0
2. 一些系统增加了标识符，表示此映射被哪个进程使用，就可以共享 TLB



**空间交换** <div id="space-switch"/>

为了超越物理内存，利用较慢的设备，透明的提供巨大虚拟地址空间的假象，需要进行空间交换：在硬盘上开辟一部分空间用于物理页的移入和移出。这样的空间叫做交换空间，因为我们将内存中的页交换到其中，并在需要的时候交换回去。为了达到这个目的，操作系统需要记住给定页的硬盘物理地址。

交换空间的大小，决定了系统在某一时刻能够使用的最大内存页数。因此，PTE 内需要存在位来表示页是否在物理内存中。访问不在物理内存中的页就会引发页错误 PageFault。

操作系统会使用 PTE 内的某些位来存储硬盘地址，当操作系统接收到页错误时，会在 PTE 中查找地址，将I/O请求发送到硬盘，将页读取到内存中。所以，发生页错误的时候，进程将进入阻塞状态。流程：

1. 页错误

2. 寻找可用的物理页帧

3. 如找不到，则需要抛弃当前内存中的某些页帧

4. I/O 请求读取页

5. 更新页表，操作系统重试指令，TLB 未命中

6. 更新 TLB，操作系统重试命中

    那么，何时发生页交换？大多数操作系统会设置高水位线 High Watermark,HW 和低水位线 Low Watermark,LW。当操作系统发现少于 LW 个页可用时，后台负责释放内存的线程会开始运行，直到有 HW 个可用页。这个后台线程有时被称为交换守护线程。
    
    

**页替换策略** <div id="page-replacement"/>

1. 最优策略，替换将来被用的最少的页。
2. FIFO
3. 随机
4. LRU



**脏页，抖动** <div id="dirty-page"/>

* 脏页就是读入内存并已经被修改的页，替换时置换脏页就需要将其写入硬盘，代价很大。因此，修改位标志其是否被修改。
* 假如系统将时间主要花在页的置换，这种情况称之为抖动。




### 2. 并发 <div id="concurrency"/>

对于多进程程序，多线程程序所可能出现的问题的处理。

对于单线程程序，一个进程只有一个执行点（一个程序计数器来存放要执行的指令），但多线程程序会有多个执行点，并且共享进程地址空间，能够访问相同的数据。



当进行线程之间的切换时，也需要上下文切换，因此使用 TCB（Thread Control Block）来保存每个线程的状态。线程切换与进程切换区别在于，线程切换不需要改变地址空间。



此外，对于单线程来说，只需要一个栈，而对于多线程程序，需要多个栈。



**多线程程序的核心问题为不可控的调度，线程的执行顺序是不可控的，因此要保证临界区（访问共享资源的代码片段）不能被多个线程同时执行。**



* 临界区：访问共享资源的代码片段
* 竞态条件：出现多个执行线程大致同时进入临界区，都试图更新共享数据，导致出现意外结果
* 不确定性：程序由一个或多个竞态条件组成，程序的输出因运行而异
* 为了避免这些问题，应该使用某种**互斥**原语，保证只有一个线程进入临界区



**原子指令**

原子指令是硬件保证以原子方式执行的指令，发生中断时，要么整体执行完毕，要么根本没有运行，没有中间状态。



典型的原子指令为 CAS（Compare And Swap），即比较并交换，是一种实现并发算法时常用到的技术。它的基本思路是，检测 ptr 指向的值是否和 expected 相等；如果是，更新 ptr 所指的值为新值；否则，什么也不做。调用会返回该内存地址的实际值，让调用者知道是否成功。



**Python 中 如何创建一个线程**

```python
class threading.Thread(group=None,
                       target=None,
                       name=None,
                       args=(),
                       kwargs={})
```

上面的代码中：

- `group`: 一般设置为 `None` ，这是为以后的一些特性预留的
- `target`: 当线程启动的时候要执行的函数
- `name`: 线程的名字，默认会分配一个唯一名字 `Thread-N`
- `args`: 传递给 `target` 的参数，要使用tuple类型
- `kwargs`: 同上，使用字典类型dict

线程被创建之后并不会马上运行，需要手动调用 `start()` ;  `join()` 让调用它的线程一直等待直到执行结束（即阻塞调用它的主线程， `t` 线程执行结束，主线程才会继续执行）



**线程状态**

1. 初始(NEW)：新创建了一个线程对象，但还没有调用start()方法。
2. 运行(RUNNABLE)：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。 线程对象创建后，其他线程(比如main线程）调用了该对象的 start() 方法。
   该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得 CPU时间片后 变为运行中状态（running）。
3. 阻塞(BLOCKED)：**表示线程阻塞于锁。注意和进程的区别，进程是IO阻塞。**
4. 等待(WAITING)：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
5. 超时等待(TIMED_WAITING)：该状态不同于WAITING，它可以在指定的时间后自行返回。
6. 终止(TERMINATED)：表示该线程已经执行完毕。



#### 2.1 锁 <div id="lock"/>

锁就是一个变量，在某一时刻要么是被占用的，要么是可用的。在临界区前尝试获得锁，如果没有其它线程持有锁，那么该线程就会进入临界区。锁的持有者在退出临界区后，需要释放锁，这样锁就变成了可用状态。



##### Python 中的锁

###### threading

threading 模块中的 Lock 为互斥锁，RLock 为可重入锁。

**API**

1. acquire()  返回是否获得了锁
2. release() 释放锁



**Lock**

- 如果状态是 unlocked， 可以调用 `acquire()` 将状态改为locked
- 如果状态是 locked， `acquire()` 会被block直到另一线程调用 `release()` 释放锁
- 如果状态是 unlocked， 调用 `release()` 将导致 `RuntimError` 异常
- 如果状态是 locked， 可以调用 `release()` 将状态改为unlocked



**Rlock**

RLock其实叫做“Reentrant Lock”，就是可以重复进入的锁，也叫做“递归锁”。这种锁对比Lock有三个特点：

1. 谁拿到谁释放。如果线程 A 拿到锁，线程 B 无法释放这个锁，只有 A 可以释放；
2. 同一线程可以多次拿到该锁，即可以 acquire 多次；
3. acquire 多少次就必须 release 多少次，只有最后一次 release 才能改变 RLock 的状态为unlocked



###### 示例代码

不加锁，输出结果不一定是预期结果0：

```python
from threading import Thread

balance = 0

def counter(n):
		global balance
		for i in range(999999):
				balance = balance + n
				balance = balance - n

threads = [
		Thread(target=counter, args=(8,)),
    Thread(target=counter, args=(10,))
]

[t.start() for t in threads]
[t.join() for t in threads]

print(balance)


Out [1]: -10
```



加互斥锁版本，输出结果一定是预期结果0：

```python
from threading import Lock, Thread

lock = Lock()
balance = 0

def counter(n):
		global balance
		if lock.acquire():
      	try:
            for i in range(999999):
                balance = balance + n
                balance = balance - n
        finally:
          	lock.release()

threads = [
		Thread(target=counter, args=(8,)),
    Thread(target=counter, args=(10,))
]

[t.start() for t in threads]
[t.join() for t in threads]

print(balance)


Out [2]: 0
```





**两阶段锁**

当一个线程试图进入临界区，无法获得锁时，就会“自旋”，即不断尝试获得锁，带来性能问题。

两阶段锁：

1. 第一阶段，先自旋一段时间，希望可以获取到锁
2. 第一阶段没有获得锁，进入第二阶段，调用者会睡眠，直到锁可用。



#### 2.2 Python 中的并发数据结构 <div id="data"/>

1. list，list如下操作是线程安全的

   ```python
   L.append(x)
   L1.extend(L2)
   x = L[i]
   x = L.pop()
   L1[i:j] = L2
   L.sort()
   x = y
   x.field = y
   D[x] = y
   D1.update(D2)
   D.keys()
   ```

   这些不是：

   ```python
   i = i+1
   L.append(L[-1])
   L[i] = L[j]
   D[x] = D[x] + 1
   ```

2. queue 模块中的队列都是线程安全的（不适用于同一线程的多次重入

   1. **Queue**
   2. **LifoQueue**
   3. **PriorityQueue**

   

   一些重要的 API：

   1. Queue.put(*item*, *block=True*, *timeout=None*)

      向队列中添加数据。

   2. Queue.get(*block=True*, *timeout=None*)[¶](https://docs.python.org/3.5/library/queue.html#queue.Queue.get)

      从队列中获取一项数据。

   3. Queue.join()

      阻塞，直到队列中所有的数据都被获取并处理完毕。

3.  multiprocessing.Queue

   这个队列用于多进程程序之间的通信。

4. collections.deque 的 [`append()`](https://docs.python.org/3.5/library/collections.html#collections.deque.append) 和 [`popleft()`](https://docs.python.org/3.5/library/collections.html#collections.deque.popleft) 属于原子操作，不需要加锁。



#### 2.3 信号量 <div id="semaphore"/>

信号量是一个有整数值的对象。当信号量值为正，则可以获取锁，否则线程会被挂起，直到其它线程改变了信号量的值。一个二值信号量就是一个互斥锁。



##### Python 中的信号量

###### threading



在threading模块中，信号量的操作有两个函数，即 `acquire()` 和 `release()` ，解释如下：

1. 创建信号量，传入的参数为信号量的初始值，默认为 1。
2. 每当线程想要读取关联了信号量的共享资源时，必须调用 `acquire()` ，此操作减少信号量的内部变量, 如果此变量的值非负，那么分配该资源的权限。如果是负值，那么线程被挂起，直到有其他的线程释放资源。
3. 当线程不再需要该共享资源，必须通过 `release()` 释放。这样，信号量的内部变量增加，在信号量等待队列中排在最前面的线程会拿到共享资源的权限。

###### 示例代码（生产者消费者模型

```python
import random
from threading import Thread, Semaphore

item = None
semaphore = Semaphore(0)

def consumer():
    print("consumer is waiting.")
    semaphore.acquire()
    # 这时就可以进入临界区
    print("Consumer notify : consumed item number %s " % item)
    # 释放

def producer():
    global item
    item = random.randint(0, 1000)
    print("producer notify : produced item number %s" % item)
    semaphore.release()

for _ in range(5):
    pt = Thread(target=producer)
    ct = Thread(target=consumer)
    
    pt.start()
    ct.start()
    
    pt.join()
    ct.join()

print('Program Terminated')
```





#### 2.4 条件变量 <div id="condition"/>

很多时候，线程需要检查某一条件满足以后才会继续运行。例如，父线程 调用 join() 时需要检查子线程是否执行完毕。



条件变量是一个显式队列，当某些状态不满足时，线程可以把自己加入队列，等待该条件。另外某个线程，当它改变了上述状态时，就可以唤醒一个或多个等待线程，让他们继续执行。



##### Python 中的条件变量

**API**

1. acquire(): 获取条件变量
2. wait(): 当前不满足运行条件，释放锁，并将自己加入到队列中
3. notify(): 改变条件变量，唤醒队列中的线程
4. release(): 释放锁



使用生产者-消费者问题示例。在本例中，只要缓存不满，生产者一直向缓存生产；只要缓存不空，消费者一直从缓存取出（之后销毁）。当缓冲队列不为空的时候，生产者将通知消费者；当缓冲队列不满的时候，消费者将通知生产者。

```python
import time
from threading import Thread, Condition

items = []
condition = Condition()

class Consumer(Thread):
  #  实现自己的线程，需要重写 __init__ 和 run 方法
  
    def __init__(self):
        Thread.__init__(self)
  
    def consume(self):
        global items
        global condition
      
        condition.acquire()
      
        if len(items) == 0:
            condition.wait()
            print("Consumer notify : no item to consume")
        items.pop()
        print("Consumer notify : consumed 1 item")
        print("Consumer notify : items to consume are " + str(len(items)))
      
        condition.notify()
        condition.release()
      
    def run(self):
        for i in range(20):
            time.sleep(2)
            self.consume()

class Producer(Thread):
  
    def __init__(self):
        Thread.__init__(self)
    
    def produce(self):
        global items
        global condition
        
        condition.acquire()
        if len(items) == 10:
            condition.wait()
            print("Producer notify : items producted are " + str(len(items)))
            print("Producer notify : stop the production!!")
        
        items.append(1)
        print("Producer notify : total items producted " + str(len(items)))
        
        condition.notify()
        condition.release()
    
    def run(self):
        for i in range(20):
            time.sleep(1)
            self.produce()

producer = Producer()
consumer = Consumer()

producer.start()
consumer.start()

producer.join()
consumer.join()
```



#### 2.5 I/O模型 <div id="io"/>

还有一种机制，称为基于事件的并发：等待某事发生（例如 I/O 工作完成），当它发生时，检查事件类型（例如，是成功了？还是失败了？），然后执行相应的工作。

对于一次 I/O 访问，数据先会被拷贝到操作系统内核的缓冲区中，然后才会被拷贝到应用程序的地址空间中。

以服务器 socket 为例，系统调用 recv_from 需要两个阶段：

1. 内核准备数据
2. 将数据从内核拷贝到应用程序





##### 1. 阻塞 I/O



当用户进程调用了 recv_from 系统调用，操作系统内核就进入第一阶段，这时数据还没准备好，怎么办？整个用户进程会被阻塞。

即：

调用 ----> 等待数据，进程被阻塞 ----> 数据准备好，进程被唤醒



##### 2. 非阻塞 I/O

当数据还没有准备好，用户进程不会被阻塞，而是由内核迅速的返回一个错误，从而告知用户进程数据还没有准备好，这样进程就可以再次询问，等待数据准备好再进行处理。

即：

调用 ----> 数据未准备好，收到错误 ----> 再次询问 ----> 再次询问 ....... ----> 数据准备好，进程进行数据处理工作



##### 3. I/O 多路复用

常见的就是 select, poll, epoll，有时也被称为基于事件的 I/O。这样的好处在于，可以让一个进程负责处理多个 socket。当有数据到达了，就通知用户进程。



**select**

调用 select 后，进程被阻塞，内核会监听 select 负责的所有 socket，当有数据就绪后通知用户进程有事件可以进行处理（但不知道是哪个事件），进程遍历所有事件（轮询），进行处理。

select的默认最大监听数为 1024，超出后性能会下降。



**poll**

poll 并未解决 select 需要轮询的问题，只是解决了最大监听数的问题，它使用链表进行存储。



**epoll**

epoll 解决了轮询的问题，内核会通知 epoll 需要处理的具体事件，不需要轮询。



epoll对文件描述符的操作有两种模式：LT（level trigger水平触发）和 ET（edge trigger边缘触发）。LT模式是默认模式，LT模式与ET模式的区别如下：

1. LT模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。实质上，只有还有可读数据，就会一直通知。
2. ET模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。实质上，对一次到达的数据，不管有没有被读取完，都只会通知一次。



##### 4. 异步 I/O

同步 I/O 是指，进行真实的 I/O 操作时，会将用户进程阻塞。上述的 阻塞 I/O，非阻塞 I/O还有 I/O 多路复用，都属于同步 I/O。Nginx 通过对 epoll 设置超时时间来实现 异步 I/O。非阻塞 I/O 虽然调用 recv_from 时不会被阻塞，但是当将数据从内核拷贝到程序中时，还是会被阻塞。但是 异步 I/O，从调用 I/O 操作开始，直到 I/O 完成都不会被阻塞，内核只会在 I/O完成后通知用户进程。





##### Python 中使用事件进行线程同步

事件是线程之间用于通讯的对象。有的线程等待信号，有的线程发出信号。基本上事件对象都会维护一个内部变量，可以通过 `set()` 方法设置为 `true` ，通过 `clear()` 方法设置为 `false` 。 `wait()` 方法将会阻塞线程，直到内部变量为 `true` 。



仍以生产者消费者模型为例。

```python
import time
import random
from threading import Event, Thread

class Consumer(Thread):
    def __init__(self, items, event):
        Thread.__init__(self)
        self.items = items
        self.event = event
    
    def run(self):
        for i in range(100):
            time.sleep(2)
            # 等待收到生产者通知再读取数据
            self.event.wait()
            item = self.items.pop()
            print('Consumer notify : %d popped from list by %s' % (item, self.name))

class Producer(Thread):
    def __init__(self, items, event):
        Thread.__init__(self)
        self.items = items
        self.event = event
    
    def run(self):
        for i in range(100):
            time.sleep(2)
            item = random.randint(0, 256)
            self.items.append(item)
            print('Producer notify : item %d appended to list by %s' % (item, self.name))
            
            print('Producer notify : event set by %s' % self.name)
            self.event.set()
            
            print('Produce notify : event cleared by %s '% self.name)
            self.event.clear()

items = []
event = Event()

producer = Producer(items, event)
consumer = Consumer(items, event)

producer.start()
consumer.start()

producer.join()
consumer.join()
```



#### 2.6 常见并发问题 <div id="problems"/>

##### 1. 非死锁缺陷

**违反原子性缺陷**

例如，线程 A 先检查变量 Val 不为空，然后试图访问 Val，而线程 B 试图将 Val 置为空。当线程 A检查到 Val 不为空时，发生了线程切换，被线程 B 将Val 置为空，程序就会出现问题。



**违反顺序缺陷**

例如，线程 A 直接访问变量 Val，线程 B 负责将变量 Val初始化。这个程序实际上默认线程 B 会先于线程 A 执行，然而实际的执行顺序总是不可预料的。即，两个内存访问的预期顺序被打破了。



##### 2. 死锁缺陷

**产生死锁的四个必要条件**

1. 互斥：线程对需要的资源进行互斥访问。例如一个互斥锁，线程 A 抢到了，线程 B 就无法得到。
2. 持有并等待：线程持有了资源，会等待其他需要的资源。例如线程 A 抢到了锁 A，需要 锁 B 但未获得，则等待 锁 B 被释放。
3. 非抢占：线程已经获得的资源，不能被其他线程抢占。
4. 循环等待：出现了一个环路。环路上的每个线程都持有部分资源，而这个资源又是下一个线程想要的。



**预防/解决死锁**

破坏死锁的四个必要条件之一。

###### 1. 循环等待：严格规定所有锁的获取顺序（全序），例如必须先获得锁 A 才能继续申请锁 B。这样严格的顺序避免了循环等待。

```python
from threading import Lock

lock1 = Lock()
lock2 = Lock()
lock3 = Lock()

# 所有线程都采用如下顺序获取锁
lock1.acquire()
lock2.acquire()
lock3.acquire()
```



缺点：需要提前知道全部需要的锁，这在大型系统中很难做到。考虑使用偏序（只规定部分关键锁的获取顺序），Linux 的内存映射代码使用的就是偏序锁。



###### 2. 持有并等待：通过原子的抢锁来规避持有部分资源并等待其他资源。例如，可以增添一个全局互斥锁 Prevention，只有先抢到 Prevention 的线程才能继续获取所需要的锁。

```python
from threading import Lock

prevention = Lock()
lock1 = Lock()
lock2 = Lock()

if prevention.acuire():
  lock1.acquire()
  lock2.acquire()
```



###### 3. 非抢占：当已经获取部分资源，却无法获取其他资源时，先释放当前已获得的资源。

```python
from threading import Lock

lock1 = Lock()
lock2 = Lock()

def consumer():
    if lock1.acquire():
      if lock2.acquire():
        pass
      else:
        # 获取不到 lock2 则释放已获得的lock1
        lock1.release()
        
def producer():
    if lock2.acquire():
      if lock1.acquire():
        pass
      else:
        # 获取不到 lock1 则释放已获得的lock2
        lock2.release()
```

缺点：这样有可能发生“活锁”的问题。多个线程同时释放资源又同时重新获取资源，却总是抢不到所有想要获取的资源。



###### 4. 互斥：避免互斥，则需要借助特殊的指令或者数据结构，即不需要互斥来进行值的读写。

例如，使用 CAS 指令，无需加锁，而是不断尝试并检查目标值是否符合预期，再进行操作。

```python
def CAS(target, expected, new_val):
    if target == expected:
        target = new_val
        return True
     return False

# 在链表头插入新的元素
# 不加锁时，在即将执行 head = n 时，假如其他线程此时更新了 head，那么就会出现问题。
def insert(val):
    node = Node(val)
    node.next = head
    head = n


# 加锁版本
def insert_with_lock(val):
    node = Node(val)
    if linklist_lock.acquire():
        try:
            node.next = head
            head = n
        finally:
            linklist_lock.release()

# 使用 CAS 版本
# 这段代码只有当 head 确实与当前 node.next 指向的值一致时，才会进行更新，否则就将node.next 指向被其它线程更新后的 head，这样就保证了不会被其他线程更新后的 head 干扰
def insert_with_CAS(val):
    node = Node(val)
    while not CAS(head, node.next, val):
        node.next = head
```



###### 5. 检查和恢复：对于允许死锁情况很少发生的程序，可以检查死锁发生后再去处理。



#### 2.7. 并发通信 <div id="communication"/>

**进程通信**

大概有 7 种常见的进程间的通信方式。

1. 管道/匿名管道(Pipes) ：用于具有亲缘关系的父子进程间或者兄弟进程之间的通信。只存在于内存中的文件

   * 匿名管道由于没有名字，只能用于亲缘关系的进程间通信。
   * 为了克服这个缺点，提出了有名管道。有名管道严格遵循先进先出 (FIFO)。有名管道以磁盘文件的方式存在，可以实现本机任意两个进程通信。

2. 信号(Signal) ：信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生；

3. 消息队列(Message Queuing) ：消息队列是消息的链表, 具有特定的格式, 存放在内存中并由消息队列标识符标识。管道和消息队列的通信数据都是先进先出的原则。与管道（无名管道：只存在于内存中的文件；命名管道：存在于实际的磁盘介质或者文件系统）不同的是消息队列存放在内核中，只有在内核重启(即，操作系统重启)或者显示地删除一个消息队列时，该消息队列才会被真正的删除。消息队列可以实现消息的随机查询, 消息不一定要以先进先出的次序读取, 也可以按消息的类型读取. 比FIFO更有优势。消息队列克服了信号承载信息量少，管道只能承载无格式字 节流以及缓冲区大小受限等缺。

4. 信号量(Semaphores) ：信号量是一个计数器，用于多进程对共享数据的访问，信号量的意图在于进程间同步。这种通信方式主要用于解决与同步相关的问题并避免竞争条件。

5. 共享内存(Shared memory) ：使得多个进程可以访问同一块内存空间，不同进程可以及时看到对方进程中对共享内存中数据的更新。这种方式需要依靠某种同步操作，如互斥锁和信号量等。可以说这是最有用的进程间通信方式。

6. 套接字(Sockets) : 此方法主要用于在客户端和服务器之间通过网络进行通信。套接字是支持TCP/IP的网络通信的基本操作单元，可以看做是不同主机之间的进程进行双向通信的端点，简单的说就是通信的两方的一种约定，用套接字中的相关函数来完成通信过程。



**利用队列进行线程/进程通信**

**利用队列进行线程通信**

Queue常用的方法有以下四个：

1. `put()`: 往queue中放一个item

2. `get()`: 从queue删除一个item，并返回删除的这个item

3. `task_done()`: 每次item被处理的时候需要调用这个方法

4. `join()`: 所有item都被处理之前一直阻塞

```python
import time
import random
from queue import Queue
from threading import Thread

queue = Queue()
def consumer(name):
    for _ in range(10):
        item = queue.get()
        print('Consumed: ', item, 'by ', name)
        queue.task_done()

def producer():
    for _ in range(20):
        item = random.randint(0, 256)
        queue.put(item)
        print('Put: ', item)
        time.sleep(1)

ct1, ct2, pt = Thread(target=consumer, args=('c1',)), Thread(target=consumer, args=('c2',)), Thread(target=producer)
ct1.start()
ct2.start()
pt.start()

ct1.join()
ct2.join()
pt.join()
```



**进程的创建和通信**

1. 创建、执行进程

   ```python
   from multiprocessing import Process
   
   process = Process(target=target_func, args=(arg1,))
   
   # 执行
   process.start()
   
   # 等待结束
   process.join()
   ```

2. 命名

   ```python
   import multiprocessing
   
   def target_func():
       name = multiprocessing.current_process().name
       print('Process name: ', name)
   
   process = Process(name='pname', target=target_func)
   ```

3. 后台运行

   ```python
   process.daemon = True
   ```

4. 检查存活/杀死进程

    ```python
   process.is_alive()
   process.terminate()
    ```

5. 也可以像重写线程对象那样继承并重新进程来创建自己的进程类

6. 进程之间交换数据采用 multiprocessing 下的 Pipe 或 Queue

   **Queue**

   ```python
   import time
   import random
   from multiprocessing import Process, Queue
   
   class Producer(Process):
       def __init__(self, queue):
           self.queue = queue
       
       def run(self):
           for i in range(10):
               item = random.randint(0, 256)
               self.queue.put(item)
               print('Produce: ', item)
               time.sleep(1)
   
   class Consumer(Process):
       def __init__(self, queue):
           self.queue = queue
       
       def run(self):
           while True:
               if self.queue.empty():
                   print('Queue Empty, Finished')
               else:
                   item = self.queue.get()
                   print('Consume: ', item)
                   time.sleep(1)
   
   if __name__ == '__main__':
       queue = Queue()
       producer = Producer(queue)
       consumer = Consumer(queue)
       
       producer.start()
       consumer.start()
       producer.join()
       consumer.join()
   ```

   队列还有一个 `JoinaleQueue` 子类，它有以下两个额外的方法：

   - `task_done()`: 此方法意味着之前入队的一个任务已经完成，比如， `get()` 方法从队列取回item之后调用。所以此方法只能被队列的消费者调用。
   - `join()`: 此方法将进程阻塞，直到队列中的item全部被取出并执行。

   

   **Pipe**

   API: `multiprocessing.Pipe`([*duplex*])

   返回一对由管道连接起来的连接符，`(conn1, conn2)`。如果*duplex* 是 True（默认），那么这个管道就不是双向的，则 conn1 只能接受数据，conn2 只能发送数据。

   

   ```python
   from multiprocessing import Pipe, Process
   
   def f(sender):
       sender.send('Hello')
       sender.close()
   
   if __name__ == '__main__':
       recver, sender = Pipe()
   
       creater = Process(target=f, args=(sender,))
       creater.start()
       print(recver.recv())
       creater.join()
   ```

   

7. 创建进程时将数据结构作为参数传入，即可实现共享内存

8. 进程的同步机制和 threading 模块中的基本一致





#### 2.8 一些问答



1. 共享数据是堆内存还说栈内存？堆和栈存的东西有什么区别？
2. Python 哪些队列是线程安全的？能用于进程通信吗？
3. 内存的缓存机制？LRU 是什么意思？如何实现？
4. Linux的 watch 和 ctrl+c 是怎么实现的
5. 指针和引用的区别
6. 多线程怎么实现同步
7. 介绍一下虚拟内存
8. 动态绑定的底层原理
9. 进程调度算法
10. Epoll和其他两个IO复用的区别
       - 多线程的IO复用和单线程的IO复用有什么区别，为什么要用多线程呢
       - Redis为什么高效，为什么它不用多线程呢
       - 水平触发和边缘触发的区别和使用场景
11. 讲一下操作系统的内存管理
    - 地址转换
    - CPU的缓存，为什么要设置L1 L2 L3缓存（面试官想考察程序的局部性原理）
12. 进程和线程的区别
    - 多线程怎么通信
    - 条件变量
    - 多线程之间怎么保证安全，用过哪种
    - 两个线程能在cpu中同时运行吗

