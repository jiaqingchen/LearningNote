## JVM

### 类字节码

- JAVA源代码由javac编译器编译为.class字节码，在jvm虚拟机上运行
- class文件

	- 本质是以8位字节为基础单位的二进制流
	- 结构类型

		- 魔数：头4个字节。为固定值：0xCAFEBABE
		- 常量池：资源仓库，包括变量和方法的属性、类型、名称等
		- 访问标志：属性和访问类型，如class或者interface，public或者private等
		- 类索引、父类索引、接口索引：确定类的继承关系
		- 字段表属性：描述接口或类声明的变量，比如类型、作用域
		- 方法表属性：描述方法，比如类型、作用域
		- 属性表属性：描述字段表或方法表一些特殊属性

### 类加载

- 类生命周期

	- 加载

		- 查找并加载类的二进制数据

	- 连接

		- 验证

			- 确保被加载的类的正确性： 文件格式验证+元数据验证+字节码验证+符号引用验证

		- 准备

			- 为类的静态变量分配内存，并将其初始化为默认值

		- 解析

			- 把类中的符号引用转化为直接引用

	- 初始化

		- 为类的静态变量赋予正确的初始值

			- 假如类没有被加载和连接，则先执行这两步
			- 假如该类的直接父类还没有被初始化，先初始化其直接父类
			- 假如类中有初始化语句，系统依次执行

	- 使用

		- 类访问方法区内的数据结构的接口，对象是heap区的数据

	- 卸载

		- java虚拟机结束生命周期

- 类加载器

	- 启动类加载器Bootstrap ClassLoader
	- 扩展类加载器Extension ClassLoader
	- 应用程序类加载器Application ClassLoader

- 类加载方式

	- 命令行启动应用时候由JVM初始化加载
	- 通过Class.forName()方法动态加载：将类的.class文件加载到jvm中之外，还会执行类中的静态初始化static块
	- 通过ClassLoader.loadClass()方法动态加载：不执行static块

- 类加载机制

	- 全盘负责
	- 父类委托
	- 缓存机制
	- 双亲委派机制

		- Application ClassLoader ——> Extension ClassLoader——>Bootstrap ClassLoader，先往parent传，加载失败再依次反序加载

### 内存结构

- 程序计数寄存器

	- 存储下一条字节码指令的地址
	- 是线程私有的，生命周期与线程一致，无OOM

- 虚拟机栈

	- 主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。
	- StackOverFlowError:线程请求分配的栈容量超过 Java 虚拟机栈允许的最大容量
	- OutOfMemoryError: 没有足够的内存扩展或者创建虚拟机栈
	- 栈的存储单位：栈帧

		- 局部变量表

			- 存储单元是slot

		- 操作数栈

			- 后进先出，保存计算过程的中间结果

		- 动态链接

			- 运行时常量池的方法引用

		- 方法返回地址

			- PC寄存器

		- 附加信息

- 本地方法栈

	- 本地方法接口

		- 一个 Java 调用非 Java 代码的接口

	- 本地方法栈

		- 管理本地方法的调用

- 堆内存

	- 逻辑内存划分

		- 年轻代

			- Eden Memory
			- 幸存区Survivor Memory S0
			- 幸存区Survivor Memory S1

		- 老年代

			- 年轻代中经过多轮GC存活下来的对象

		- 元空间

	- OOM

		- -Xmx 堆的起始内存
		- -Xms 堆的最大内存
		- 通常会将 -Xmx 和 -Xms 两个参数配置为相同的值，其目的是为了能够在垃圾回收机制清理完堆区后不再需要重新分隔计算堆的大小，从而提高性能

	- 对象在堆中的生命周期

		- 1. 优先分配到年轻代中的Eden区，并定义了一个对象年轻计数器MaxTenuringThreshold
		- 2. Eden空间不足时，执行MinorGC，存活的对象转移到幸存区，且对象年龄+1
		- 3. 当分配的对象计数器超过了PretenureSizeThreshold，转移到老年代中

	- TLAB

		- 线程本地分配缓存区

	- 栈上分配、标量替换优化技术

		- 逃逸分析

			- 分析对象的动态作用域，可以转为栈上分配

		- 同步省略
		- 标量替换

			- 为了减少临时对象在堆内分配的数量，JVM 通过逃逸分析确定该对象不会被外部访问。那就通过标量替换将该对象分解在栈上分配内存，这样该对象所占用的内存空间就可以随栈帧出栈而销毁，就减轻了垃圾回收的压力。

- 方法区

	- 方法区用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等。
	- 类型信息
	- 域信息
	- 方法信息
	- 常量池

		- String类的intern()方法

	- 垃圾回收

		- 回收内容：常量池中废弃的常量和不再使用的类型
		- 回收条件：1. 该类的所有实例已回收 2. 类加载器已被回收 3. Class对象已回收，没有地方通过反射访问这个类

### 内存模型

- 引入

	- 硬件内存

		- cpu寄存器， cpu高速缓存存储器，主存储器

	- 线程栈，堆
	- 连接硬件内存

		- 对象共享后的可见性

			- 使用Java的volatile关键字。 volatile关键字可以确保直接从主内存读取给定变量，并在更新时始终写回主内存

		- 竞态条件

			- Java synchronized块

- 基础

	- 线程通信与同步机制：共享内存

		- 通过写 - 读内存中的公共状态来隐式进行通信
		- 显式指定某个方法或某段代码需要在线程之间互斥执行

- 重排序

	- 1. 编译器优化的重排序
	- 2. 指令级并行的重排序
	- 3. 内存系统的重排序
	- as-if-serial：不管怎么重排序，程序的执行结果不会改变

- 顺序一致性

	- 一个线程中的所有操作必须按照的程序的顺序来执行
	- 所有线程都只能看到一个单一的程序执行顺序

### 调优

- 垃圾回收基础

	- 判断一个对象是否可被回收

		- 引用计数算法
		- 可达性分析算法
		- 方法区中的回收： 常量池的回收和对类的卸载
		- finalize()

	- 引用类型

		- 强引用：new Object()

			- 不会回收

		- 软引用： new SoftReference<Object>()

			- 软引用关联的对象内存不足时会回收

		- 弱引用：new WeakReference<Object>(obj)

			- 弱引用关联的对象一定会被回收

		- 虚引用：new PhantomReference<Object>()

			- 对回收无影响，仅在回收时收到系统通知

	- 垃圾回收算法

		- 1.  标记——清除

			- 标记存活的对象，清理未标记对象（会产生大量不连续的内存碎片）

		- 2.  标记——整理

			- 标记后让所有存活对象向着一端移动，然后清理掉端边界以外所有内存

		- 3. 复制

			- 将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理

		- 4. 分代收集

			- 新生代： 复制算法

				- Eden + Survivor 0 + Survivor 1

			- 老年代： 标记-清楚 or  标记-整理

	- 垃圾收集器

		- Serial收集器

			- 串行的单线程收集器

		- ParNew 收集器

			- Serial的多线程版本

		- Parallel Scavenge 收集器

			- 吞吐量优先

		- Serial Old 收集器
		- Parallel Old 收集器
		- CMS 收集器

			- 初始标记—并发标记—重新标记—并发清除

		- G1 收集器

			- 初始标记—并发标记——最终标记——筛选回收

	- 内存分配策略

		- 1.  对象优先在 Eden 分配 2. 大对象直接进入老年代 3. 长期存活的对象进入老年代 4. 动态对象年龄判定 5. 空间分配担保

	- 回收策略

		- MinorGC

			- Eden空间满时就会触发

		- FullGC

			- 1. 调用System.hc()
			- 2. 老年代空间不足
			- 3. 空间分配担保失败
			- 4. JDK 1.7 及以前的永久代空间不足
			- 5. Concurrent Mode Failure
