
GC `Garbage Collection`：垃圾回收

垃圾回收机制（GC）是通过引擎实现的，不同的引擎有不同的回收策略，v8 引擎是目前市场上比较主流的 js 引擎，对 GC 也有一定的优化

垃圾回收策略：

JavaScript 在内存管理中有一个概念`可达性`，内存中可以访问的值，我们把它存储在内存中，反之则需要被回收

垃圾回收的策略有很多种，比较常见的有两种：

- 标记清除算法
- 引用计数算法

### 💡标记清除算法

引擎在使用标记清除算法执行 GC 时，需要从`跟对象`出发，遍历内存中的所有对象去打标记，根对象在浏览器环境中包括但又不限于`全局window对象`、`文档DOM树`等

该算法执行过程大致如下：

1. 垃圾收集器在运行时给内存中的所有变量都加上标记，假定一开始所有变量都是垃圾，标记为 0
2. 然后从各个根对象开始遍历，把不是垃圾的变量标记置为 1
3. 清理掉所有标记为 0 的垃圾，销毁并回收它们所占用的内存空间
4. 最后，把所有的变量标记置为 0，等待下一轮垃圾回收

✅ 优点

实现简单，只需要二进制的 0 和 1 就可以完成标记

❎ 缺点

- 内存碎片化
- 分配速度慢

在垃圾清除之后，剩余对象的存储位置没有改变，导致空闲内存空间不是连续的，产生了`内存碎片`，在后面的内存分配中会很影响性能

内存分配三种策略：

- `First-fit` 找到大于等于 size 的内存块立即返回
- `Best-fit` 遍历整个空闲列表，返回大于等于 size 的最小分块
- `Worst-fit` 遍历整个空闲列表，找到最大的分块，切割成两部分，一部分为 size 大小，并将该部分返回

🏷️ 缺点优化

针对这种却缺点，可以使用`标记整理算法`进行解决，该算法在标记清除阶段和标记清除算法没有什么不同，但是在垃圾清理之前，会先将不需要清理的对象移到内存的一端，最后清理掉边界的内存

### 💡引用计数算法

该算法是早先的一种垃圾回收算法，简化对象是否被需要，如果该对象没有被引用（零引用），就会被垃圾回收机制回收

大致流程：

- 声明一个变量，将引用类型的值赋给该变量的时候，这个值的引用数就为 1
- 该值赋给另外一个变量时，引用数 +1
- 变量的引用值被其他值覆盖时，引用数 -1
- 垃圾回收器在执行的时候会对引用数为 0 的值所占用的内存进行回收

✅ 优点

- 可以立即回收，在变为 0 时，就会被立即回收
- 在 JS 线程中，不用每隔一段时间暂停去执行 GC
- 不用遍历堆里的所有对象

❎ 缺点

- 计数器会占很大的空间，并且还不知道引用数量的上限
- 无法解决循环引用无法回收的问题

### 💡V8 对 GC 的优化

V8 也是基于标记清除算法，但是对该算法做了一些优化。

🏷️ 分代式垃圾回收

对于不同大小、存活时间的对象来说，使用同样的频率来检查是很不友好的，对于长时间存活的对象并不需要频繁的进行清理检查，分代式垃圾回收就是针对该现象做的优化

V8 中将堆内存分为`新生代`和`老生代`两个区域，采用不同的策略管理进行垃圾回收

🏷️ 新生代垃圾回收

该回收机制是将堆内存分为两个区域：`使用区`和`空闲区`

新加入的对象会被存放在使用区，当使用区快要存满时，就会进行一次垃圾清理操作

新生代回收器在执行的时候会对使用区的活动对象进行标记，标记完成之后会将这些活对象排序并复制到空闲区，然后进入清理阶段，清空使用区中非活对象所占用的空间，最后进行区域身份的互换

对于多次复制依然存活的对象直接会被放入老生代中

还有就是一个对象在复制时，空间占用超过 25% 的会被直接移入至老生代中

🏷️ 老生代垃圾回收

对于占用空间大、存活时间长的对象会被分配到老生代里，该回收策略主要采用标记清除算法，该算法会产生内存碎片，而 V8 引擎则通过标记整理算法解决这一问题来优化内存分配

老生代垃圾回收的优化：

➡️ 并行回收

多开几个辅助线程进行 GC，缩短 GC 时间

➡️ 增量标记

将一次 GC 标记的过程，分为很多小步，穿插在 js 逻辑执行之间，交替完成

➡️ 惰性清理

对于老生代来说，假如当前内存足以让代码快速执行，那么就无需理解清理内存，也无需一次性清理完所有非活动对象，可以逐一进行

➡️ 三色标记法

由于增量标记穿插在 js 执行中，所以并不能知道下一次该从何处开始标记，三色标记法采用三种颜色：白、灰、黑

- 白色指未被标记的对象
- 灰色指自身被标记，成员（该对象引用的对象）未被标记
- 黑色指自身和成员都被标记

➡️ 写屏障

由于三色标记中过程中的 js 执行可能会修改黑色标记的引用为白色标记，导致误清除，写屏障机制则是针对此情况，一旦有黑色对象引用变为白色，会强制将引用的白色对象改为灰色