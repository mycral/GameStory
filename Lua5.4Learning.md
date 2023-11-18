

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

The garbage-collector pause controls how long the collector waits before starting a new cycle. The collector starts a new cycle when the use of memory hits n% of 
the use after the previous collection. Larger values make the collector less aggressive. Values equal to or less than 100 mean the collector will not wait to start a new cycle. 
A value of 200 means that the collector waits for the total memory in use to double before starting a new cycle. The default value is 200; the maximum value is 1000.

The garbage-collector step multiplier controls the speed of the collector relative to memory allocation, that is, how many elements it marks or sweeps for 
each kilobyte of memory allocated. Larger values make the collector more aggressive but also increase the size of each incremental step. You should not use values less than 100, 
because they make the collector too slow and can result in the collector never finishing a cycle. The default value is 100; the maximum value is 1000.

The garbage-collector step size controls the size of each incremental step, specifically how many bytes the interpreter allocates before performing a step. 
This parameter is logarithmic: A value of n means the interpreter will allocate 2n bytes between steps and perform equivalent work during the step.
A large value (e.g., 60) makes the collector a stop-the-world (non-incremental) collector. The default value is 13, which means steps of approximately 8 Kbytes.

2.5.2 – Generational Garbage Collection
In generational mode, the collector does frequent minor collections, which traverses only objects recently created. If after a minor collection the use of memory is still above a limit, 
the collector does a stop-the-world major collection, which traverses all objects. The generational mode uses two parameters: the minor multiplier and the the major multiplier.

The minor multiplier controls the frequency of minor collections. For a minor multiplier x, a new minor collection will be done when memory grows x% larger than the memory 
in use after the previous major collection. For instance, for a multiplier of 20, the collector will do a minor collection when the use of memory gets 20% larger than 
the use after the previous major collection. The default value is 20; the maximum value is 200.

The major multiplier controls the frequency of major collections. For a major multiplier x, a new major collection will be done when memory grows x% larger than 
the memory in use after the previous major collection. For instance, for a multiplier of 100, the collector will do a major collection when the use of memory gets larger than 
twice the use after the previous collection. The default value is 100; the maximum value is 1000.


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
