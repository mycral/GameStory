# 2.2 – Environments and the Global Environment
As we will discuss further in §3.2 and §3.3.3, any reference to a free name (that is, a name not bound to any declaration) var is syntactically translated to _ENV.var. Moreover, every chunk is compiled in the scope of an external local variable named _ENV (see §3.3.2), so _ENV itself is never a free name in a chunk.  
我们将会在后面讨论。 任何一个自由名字的引用（也就说，这个名字没有绑定到任何声明，也就说局部没有声明这个变量）那么这个变量在语法上就会翻译到_ENV.var上面（也就是说会去_ENV上面去找这个变量）。 此外，每一个chunk被编译进入一个额外局部变量_ENV,因此_ENV本身在chunk里面不是随便定义的。  

Despite the existence of this external _ENV variable and the translation of free names, _ENV is a completely regular name. In particular, you can define new variables and parameters with that name. Each reference to a free name uses the _ENV that is visible at that point in the program, following the usual visibility rules of Lua (see §3.5).  
虽然_ENV这个变量用来包自由使用的名字，它完全是一个常规的名字。 特别是，你可以往它里面添加新的变量。 每一个使用的自由名字对于_ENV来说都是可见的。 遵从lua的可见规则。  

Any table used as the value of _ENV is called an environment.  
任何作为_ENV存在的表都称之为环境。（环境表）  
Lua keeps a distinguished environment called the global environment. This value is kept at a special index in the C registry (see §4.3). In Lua, the global variable _G is initialized with this same value. (_G is never used internally, so changing its value will affect only your own code.)  
lua还保存了另外一个环境表叫做全局环境表。这个表被注册在C注册表上面。在lua里面，全局_G 对应其值。（_G 不是内部使用的，因此改变其值，只会影响你自己的代码）（需要测试）
When Lua loads a chunk, the default value for its _ENV variable is the global environment (see load). Therefore, by default, free names in Lua code refer to entries in the global environment and, therefore, they are also called global variables. Moreover, all standard libraries are loaded in the global environment and some functions there operate on that environment. You can use load (or loadfile) to load a chunk with a different environment. (In C, you have to load the chunk and then change the value of its first upvalue; see lua_setupvalue.)  
当lua加载一个chunk的时候，_ENV就是全局环境表，默认的，lua里面的自由名字引用的都是全局环境中的变量，因此，他们被叫做全局变量。 此外，所有的标准库和部分函数都是在全局环境加载的。 你可以使用不同的环境变量 用load或者loadfile去加载一个chunk。  
在C代码中，你可以去加载一个chunk，然后修改它的upvalue中的第一个值。（这个第一个应该默认就是环境变量表）。  


# 3.4.7 – The Length Operator
# 长度运算符
The length operator is denoted by the unary prefix operator #.  
长度运算符是一个一元前缀#  
The length of a string is its number of bytes. (That is the usual meaning of string length when each character is one byte.)  
字符串的长度就是字符串所包含的字节数。（ 当一个字符一个字节的时候，也就是对应的字符串长度了）（当然字符串里面可以包含\0,作为字节流使用）  
The length operator applied on a table returns a border in that table. A border in a table t is any non-negative integer that satisfies the following condition:  
长度运算符用在table上面的时候，返回的是table的一个border(边缘）。 一个表的border的意思是一个非负整数并且满足下面几个条件：  
     (border == 0 or t[border] ~= nil) and  
     (t[border + 1] == nil or border == math.maxinteger)  
     
In words, a border is any positive integer index present in the table that is followed by an absent index, plus two limit cases: zero, when index 1 is absent; and the maximum value for an integer, when that index is present. Note that keys that are not positive integers do not interfere with borders.  
一句话，boder就是一个有内容的下表需要（ table[border]）然后紧邻一个是空值的下标序号，然后加上两个限制： 如果是0，说明序号1下面也没数据； intger的最大值的时候也有对应的数据。 注意，非正整数下标不影响border。  
  
A table with exactly one border is called a sequence. For instance, the table {10, 20, 30, 40, 50} is a sequence, as it has only one border (5). The table {10, 20, 30, nil, 50} has two borders (3 and 5), and therefore it is not a sequence. (The nil at index 4 is called a hole.) The table {nil, 20, 30, nil, nil, 60, nil} has three borders (0, 3, and 6), so it is not a sequence, too. The table {} is a sequence with border 0.  
有唯一border的table才是一个序列（数组）。 比如，table {10, 20, 30, 40, 50}是个序列，因为只有一个border(5).  table {10, 20, 30, nil, 50} 有两个 border (3 和 5),因此他不是一个序列。(序号4的位置是个nil值，称之为 hole（洞？）.)。
 table {nil, 20, 30, nil, nil, 60, nil} 有三个 border (0, 3, and 6), 所以也不是一个序列。  table {} 的border是0，是个序列。  
  
When t is a sequence, #t returns its only border, which corresponds to the intuitive notion of the length of the sequence. When t is not a sequence, #t can return any of its borders. (The exact one depends on details of the internal representation of the table, which in turn can depend on how the table was populated and the memory addresses of its non-numeric keys.)  
当t是个序列的时候，它只有一个border，也就是直觉上的序列的长度的概念。如果不是一个序列，那么#t看可能返回任何一个border值。
（具体哪个边界取决于表的内部表示的细节，这又可能取决于如何填充表以及其非数值键的内存地址。）  
The computation of the length of a table has a guaranteed worst time of O(log n), where n is the largest integer key in the table.  
计算表的长度消耗的时间是可以保证的O(log n) ，n是最大的整数 键值。 （这个地方应该是指的序列）  
A program can modify the behavior of the length operator for any value but strings through the __len metamethod (see §2.4).  
程序员可以通过修改原表中的__len 方法来修改这个获取长度的行为。（比如自定义计算长度的方法）  

*补充：O(1)：常量时间  
O(n)：线性时间  
O(log n)：对数时间  
O(n^2)：二次方时间  
O(2^n)：指数时间  
O(n!)：阶乘时间*  

# 2.5 – Garbage Collection
# 垃圾收集
*Lua performs automatic memory management. This means that you do not have to worry about allocating memory for new objects or freeing it when the objects are no longer needed.*  
lua是自动管理内存的，也就是说你不需要去担心创建对象的时候去申请内存和 对象不被需要的时候去释放对象和其对应内存。
*Lua manages memory automatically by running a garbage collector to collect all dead objects. All memory used by Lua is subject to automatic management: strings, tables, userdata,  
functions, threads, internal structures, etc.*  
lua是通过垃圾收集器去收集死亡的对象来自动管理内存。所有的对象使用的内存都是自动管理了，这些对象包括strings, tables, userdata,functions, threads, internal structures, 等等.  
*An object is considered dead as soon as the collector can be sure the object will not be accessed again in the normal execution of the program.
("Normal execution" here excludes finalizers, which can resurrect dead objects (see §2.5.3), and excludes also operations using the debug library.) 
Note that the time when the collector can be sure that an object is dead may not coincide with the programmer's expectations. 
The only guarantees are that Lua will not collect an object that may still be accessed in the normal execution of the program, and it will eventually 
collect an object that is inaccessible from Lua. (Here, inaccessible from Lua means that neither a variable nor another live object refer to the object.) 
Because Lua has no knowledge about C code, it never collects objects accessible through the registry (see §4.3), which includes the global environment (see §2.2).*  
在程序的“正常运行中”，一个对象在当其无法被访问到的时候，就会被认为是死亡了。（这里的“正常运行”意思是排除终结器，这个终结器可以救活死亡的对象而且可以避免debug库debug）（终结器里面下断点不管用）  
注意，有些时候收集器任务一个对象死亡了，但不见得符合程序员的期望。  
唯一可以保证的是在“正常运行中”lua不会回收那些可以被索引（引用）到的对象，然后lua会收集那些无法访问到的对象。（这里无法索引到意味着没有一个活着的对象索引（引用）到该对象）。  
因为lua不知道c代码，他重来不通过”注册表“ 收集对象，（注册表包含全局环境，不懂去看2.2）  

*The garbage collector (GC) in Lua can work in two modes: incremental and generational.*  
垃圾收集包括两个模式： 增量和分代

*The default GC mode with the default parameters are adequate for most uses. However, programs that waste a large proportion 
of their time allocating and freeing memory can benefit from other settings. Keep in mind that the GC behavior is non-portable 
both across platforms and across different Lua releases; therefore, optimal settings are also non-portable.*  
默认的GC模式已经适用大部分使用lua的情况。 但是，程序如果浪费大量的时间去申请和释放内存，可以通过其他设置来改善。 要时刻谨记，
垃圾收集的行为在不同的平台和不同的Lua版本之间是不可移植的；因此，最优的设置也是不可移植的。  
*You can change the GC mode and parameters by calling lua_gc in C or collectgarbage in Lua. You can also use these functions
to control the collector directly (e.g., to stop and restart it).*  
你可以通过在C中调用lua_gc或在Lua中调用collectgarbage来改变GC模式和参数。你也可以直接使用这些函数来控制垃圾收集器（比如，停止和重启它）。  
2.5.1 – Incremental Garbage Collection
*In incremental mode, each GC cycle performs a mark-and-sweep collection in small steps interleaved with the program's execution. 
In this mode, the collector uses three numbers to control its garbage-collection cycles: the garbage-collector pause, the garbage-collector step multiplier,
and the garbage-collector step size.*  
在增量模式下，每次gc循环都是一个标记清理收集的过程，分为很多小step步骤插入在程序的执行中。

在这个模式下，收集器使用3个数字去控制垃圾收集循环： 垃圾收集器暂停，垃圾收集器step倍增 和 垃圾收集器step大小。

*The garbage-collector pause controls how long the collector waits before starting a new cycle. The collector starts a new cycle when the use of memory hits n% of 
the use after the previous collection. Larger values make the collector less aggressive. Values equal to or less than 100 mean the collector will not wait to start a new cycle. 
A value of 200 means that the collector waits for the total memory in use to double before starting a new cycle. The default value is 200; the maximum value is 1000.*  
“垃圾收集暂停” 控制要等多久才开始新的垃圾收集循环。 收集器在内存使用触碰到上次收集之后的n%的时候，会开始新的循环。 越大的值会让收集器越不激进。 值等于或者小于100意味着收集器不会等待就开始一个新的收集循环。
值为200意味着收集器会等待总内存使用量翻倍后再开始新的周期。默认值是200，最大值是1000。

The garbage-collector step multiplier controls the speed of the collector relative to memory allocation, that is, how many elements it marks or sweeps for 
each kilobyte of memory allocated. Larger values make the collector more aggressive but also increase the size of each incremental step. You should not use values less than 100, 
because they make the collector too slow and can result in the collector never finishing a cycle. The default value is 100; the maximum value is 1000.  
垃圾收集器步进乘数控制收集器相对于内存分配的速度，即它每分配一千字节的内存就标记或清扫多少个元素。较大的值会使收集器更积极，但也会增加每个增量步骤的大小。  
你不应该使用小于100的值，因为它们会使收集器过慢，可能导致收集器永远无法完成一个周期。默认值是100；最大值是1000。

The garbage-collector step size controls the size of each incremental step, specifically how many bytes the interpreter allocates before performing a step. 
This parameter is logarithmic: A value of n means the interpreter will allocate 2n bytes between steps and perform equivalent work during the step.
A large value (e.g., 60) makes the collector a stop-the-world (non-incremental) collector. The default value is 13, which means steps of approximately 8 Kbytes.  
垃圾收集器步进大小控制每个增量步骤的大小，具体而言，就是解释器在执行一个步骤之前分配多少字节。这个参数是对数的：n的值意味着解释器在步骤之间会分配2^n字节，并在步骤期间执行等量的工作。
较大的值（例如，60）会使收集器成为一个停止世界（非增量）的收集器。默认值是13，这意味着步骤大约是8 K字节。
2.5.2 – Generational Garbage Collection
In generational mode, the collector does frequent minor collections, which traverses only objects recently created. If after a minor collection the use of memory is still above a limit, 
the collector does a stop-the-world major collection, which traverses all objects. The generational mode uses two parameters: the minor multiplier and the the major multiplier.  
在代际模式下，收集器会频繁进行小型收集，这只涉及到最近创建的对象。
如果在一次小型收集后，内存使用量仍然超过一个限制，收集器就会进行一次停止世界的大型收集，这涉及到所有的对象。
代际模式使用两个参数：小型乘数和大型乘数。  
The minor multiplier controls the frequency of minor collections. For a minor multiplier x, a new minor collection will be done when memory grows x% larger than the memory 
in use after the previous major collection. For instance, for a multiplier of 20, the collector will do a minor collection when the use of memory gets 20% larger than 
the use after the previous major collection. The default value is 20; the maximum value is 200.  
小型乘数控制小型收集的频率。对于小型乘数x，当内存增长到上次大型收集后的内存使用量的x%时，将进行新的小型收集。
例如，对于乘数为20，当内存使用量增长到上次大型收集后的内存使用量的20%时，收集器将进行小型收集。默认值是20；最大值是200。
The major multiplier controls the frequency of major collections. For a major multiplier x, a new major collection will be done when memory grows x% larger than 
the memory in use after the previous major collection. For instance, for a multiplier of 100, the collector will do a major collection when the use of memory gets larger than 
twice the use after the previous collection. The default value is 100; the maximum value is 1000.  
大型乘数控制大型收集的频率。对于大型乘数x，当内存增长到上次大型收集后的内存使用量的x%时，将进行新的大型收集。  
例如，对于乘数为100，当内存使用量大于上次收集后的内存使用量的两倍时，收集器将进行大型收集。默认值是100；最大值是1000。

## 2.5.3 – Garbage-Collection Metamethods
## 垃圾收集Meta方法
*You can set garbage-collector metamethods for tables and, using the C API, for full userdata (see §2.4). These metamethods, called finalizers, are called when 
the garbage collector detects that the corresponding table or userdata is dead. Finalizers allow you to coordinate Lua's garbage collection with external resource
management such as closing files, network or database connections, or freeing your own memory.*  
**你可以为一个表设置一个meta方法，或者使用C函数的API为fll userdata 设置一个meta方法。
这些meta方法，被称作终结器，是当垃圾清理器检测到对应的table或者userdata死亡的时候。
终结器可以让你通过垃圾回收器去定位需要管控的外部资源，比如关闭文件网络或者数据库连接，再或者是释放你自己管控的内存。**  
*For an object (table or userdata) to be finalized when collected, you must mark it for finalization. You mark an object for finalization when you set its metatable and the metatable has a __gc metamethod. Note that if you set a metatable without a __gc field and later create that field in the metatable, the object will not be marked for finalization.*  
**如果需要一个对象在被认定为垃圾的时候被被终结器终结，那么你必须把他标记为需要终结的。 你可以通过“设置”一个带有__gc方法的表作为该对象的metatable，来标记该对象需要终结器终结，请注意，如果你设置一个metatable没有__gc方法但是后面你又添加上去，那么这个对象不会被标记为需要终结器终结。**
*When a marked object becomes dead, it is not collected immediately by the garbage collector. Instead, Lua puts it in a list. After the collection, Lua goes through that list. For each object in the list, it checks the object's __gc metamethod: If it is present, Lua calls it with the object as its single argument.*  
**当一个被标记的对象死亡的时候，垃圾回收器不会立即回收，反而是把它放到一个列表里面去，收集完成之后，lua会遍历这个表，对于每一个表中的对象，lua都去检测是否有__gc元方法，如果有，那么会调用这个方法，并且把这个对象作为唯一的参数传入。**  
*At the end of each garbage-collection cycle, the finalizers are called in the reverse order that the objects were marked for finalization, among those collected in that cycle; that is, the first finalizer to be called is the one associated with the object marked last in the program. The execution of each finalizer may occur at any point during the execution of the regular code.*  
**在每次垃圾回收循环的末尾，终结器是按照被标记为终结器的相反顺序调用的。第一个被调用的终结器，是最晚被标记为需要终结的对象。 终结器的调用会发生在任何常规代码的执行点。**  
*Because the object being collected must still be used by the finalizer, that object (and other objects accessible only through it) must be resurrected by Lua. Usually, this resurrection is transient, and the object memory is freed in the next garbage-collection cycle. However, if the finalizer stores the object in some global place (e.g., a global variable), then the resurrection is permanent. Moreover, if the finalizer marks a finalizing object for finalization again, its finalizer will be called again in the next cycle where the object is dead. In any case, the object memory is freed only in a GC cycle where the object is dead and not marked for finalization.*  
**因为这个被收集的对象仍然被终结器使用，那么这个对象（<u>和只能通过这个对象访问到的对象</u>）必须被lua复活，通常这个复活是临时的，这个对象会在下次回收循环被回收。（调用一次setmetatable，被标记一次需要终结）。在任何情况下，被回收的对象都是已经死亡并且没有标记终结器的。**  
*When you close a state (see lua_close), Lua calls the finalizers of all objects marked for finalization, following the reverse order that they were marked. If any finalizer marks objects for collection during that phase, these marks have no effect.*  
**当你关闭一个lua虚拟机的时候，lua会调用所有被标记终结器的对象的终结器，而且是按照被标记的顺序的逆序执行。在这种时候，使用标记方法是无效的。**  
*Finalizers cannot yield nor run the garbage collector. Because they can run in unpredictable times, it is good practice to restrict each finalizer to the minimum necessary to properly release its associated resource.*  
**终结器方法不能延迟（coroutine)或者执行垃圾回收，因为他们执行的时刻是不确定的，使用终结器去释放仅仅与其关联的资源，是最好的方式。（不建议各种魔法使用）**  
*Any error while running a finalizer generates a warning; the error is not propagated.*  
**在终结器里面发生的任何错误都会生成异常警告，这些错误不会传递影响后面的数据处理。（可以任务这个__gc调用类似一个pcall或者trycatch）**  

## 2.5.4 – Weak Tables
A weak table is a table whose elements are weak references. A weak reference is ignored by the garbage collector. In other words, if the only references to an object are weak references, then the garbage collector will collect that object.  
弱引用表就是元素都是弱引用的表。 弱引用会被垃圾收集器忽略。 也就是说，如果对一个对象的引用是弱引用，那么垃圾回收器会清理这个对象。  
A weak table can have weak keys, weak values, or both. A table with weak values allows the collection of its values, but prevents the collection of its keys. A table with both weak keys and weak values allows the collection of both keys and values. In any case, if either the key or the value is collected, the whole pair is removed from the table. The weakness of a table is controlled by the __mode field of its metatable. This metavalue, if present, must be one of the following strings: "k", for a table with weak keys; "v", for a table with weak values; or "kv", for a table with both weak keys and values.  
弱引用表可以是key弱引用 或者是 value弱引用，或者二者都是。 value弱引用的表运行垃圾收集器收集他的值，但是会限制收集他的key。 二者都是弱引用的表运行key和value都被收集。 在任何时候，如果key或者value有一个被垃圾收集了，那么这个键值对就被从表里面移除了。弱引用表的弱引用是被metatable中的__mode 字段控制的。 这个元值如果出现的话，必须是以下字符串， 设置"k"那么说明key是弱引用， 设置"v"那么说明value是弱引用。 "kv"就不用解释了。  

A table with weak keys and strong values is also called an ephemeron table. In an ephemeron table, a value is considered reachable only if its key is reachable. In particular, if the only reference to a key comes through its value, the pair is removed.  
只有key是弱引用的表叫做 短暂表 ，在短暂表中，一个值是否可以达要根据key是否可以达。在特殊情况下，如果对key的引用来自他的value，那么这个键值对会被移除。 （也就是说 key不被其他对象引用，只有跟这个value作为一个键值对存在这个表中）  

Any change in the weakness of a table may take effect only at the next collect cycle. In particular, if you change the weakness to a stronger mode, Lua may still collect some items from that table before the change takes effect.  
对一个表弱引用的修改，会在下个垃圾收集循环起作用。特别是，如果设置弱引用成强引用，lua仍然可能在这个修改 起作用之前 收集掉这个表。  
Only objects that have an explicit construction are removed from weak tables. Values, such as numbers and light C functions, are not subject to garbage collection, and therefore are not removed from weak tables (unless their associated values are collected). Although strings are subject to garbage collection, they do not have an explicit construction and their equality is by value; they behave more like values than like objects. Therefore, they are not removed from weak tables.
**只有明确构建的的对象才会被从若表中移除，值，比如数字或者轻量级c函数，都不是垃圾收集的对象，因此不会冲若表中移除，除非他们关联的值被收集了（key 或者是 value有一个被回收）。 虽然字符串也是被垃圾收集的对象，但是它没有明确的构建然后他们的等价是值，他们的对象更像值。 因此不会冲弱表中移除。（不是很理解下面有字符串的详细一点的解释）**  
Resurrected objects (that is, objects being finalized and objects accessible only through objects being finalized) have a special behavior in weak tables. They are removed from weak values before running their finalizers, but are removed from weak keys only in the next collection after running their finalizers, when such objects are actually freed. This behavior allows the finalizer to access properties associated with the object through weak tables.  
**复活对象（也就是被终结的对象和通过这个对象能访问到的其他对象） 在若表中有一个特殊的行为。  
他们在执行终结器“之前”被从弱值中移除， 但是key弱引用的对象会在终结器执行之后移除，  在这些对象被实际释放的时候。  这个行为允许终结器访问一个被弱引用表引用的对象。 （这里要看仔细，也很重要）**  
If a weak table is among the resurrected objects in a collection cycle, it may not be properly cleared until the next cycle.
如果一个弱引用的表在一个收集循环中复活了，那么应该在下次收集之前不会被清理掉。  

Note:
Strings present a subtlety here: Although strings are collectible, from an implementation point of view, they are not like other collectible objects. Other objects, such as tables and functions, are created explicitly. For instance, whenever Lua evaluates {}, it creates a new table. Whenever it evaluates function () ... end, it creates a new function (a closure, actually). However, does Lua create a new string when it evaluates "a".."b"? What if there is already a string "ab" in the system? Does Lua create a new one? Can the compiler create that string before running the program? It does not matter: These are implementation details. Thus, from the programmer's point of view, strings are values, not objects. Therefore, like a number or a boolean, a string is not removed from weak tables (unless its associated value is collected).   

# collectgarbage ([opt [, arg]])
This function is a generic interface to the garbage collector. It performs different functions according to its first argument, opt:  
这个方法是垃圾收集器的通用接口。 跟据传入的参数执行不同的功能。 有以下选项：  
"collect": Performs a full garbage-collection cycle. This is the default option.  
执行一次完整的内存回收，这个是默认选项。  
"stop": Stops automatic execution of the garbage collector. The collector will run only when explicitly invoked, until a call to restart it.  
停止垃圾收集，这个停止不是立马停止，因为lua是单线程，因此这里是停止后面的垃圾回收触发，除非注定触发，知道调用restart去重启这个功能。  
"restart": Restarts automatic execution of the garbage collector.  
重启自动执行垃圾收集的功能。  
"count": Returns the total memory in use by Lua in Kbytes. The value has a fractional part, so that it multiplied by 1024 gives the exact number of bytes in use by Lua.  
返回lua使用的总内存，但挖是kbytes。 这个值有个小数部分，因此必须乘以1026才能获取以bytes为单位的精确值。  
"step": Performs a garbage-collection step. The step "size" is controlled by arg. With a zero value, the collector will perform one basic (indivisible) step.  
       For non-zero values, the collector will perform as if that amount of memory (in Kbytes) had been allocated by Lua. Returns true if the step finished a collection cycle.  

"isrunning": Returns a boolean that tells whether the collector is running (i.e., not stopped).  
获取垃圾收集器是否工作，意思是是否自动回收。  
"incremental": Change the collector mode to incremental. This option can be followed by three numbers: the garbage-collector pause, the step multiplier,  
               and the step size (see §2.5.1). A zero means to not change that value.  
切换垃圾收集器模式成增量模式，这个选项后面可以跟三个数字，分别是 暂停 倍增 和 步进大小。 0代表不改变对应的值。  
"generational": Change the collector mode to generational. This option can be followed by two numbers: the garbage-collector minor multiplier and the major multiplier (see §2.5.2).  
                 A zero means to not change that value.  
切换垃圾收集器到分代模式。 这个选项后面跟两个数字： 年轻代倍率值和主代倍率值。  0代表不改变对应的值。  

See §2.5 for more details about garbage collection and some of these options.  

This function should not be called by a finalizer.  
终结器（__gc）不能调用这个方法。  
