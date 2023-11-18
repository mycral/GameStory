## 2.5.3 – Garbage-Collection Metamethods
垃圾收集Meta方法
You can set garbage-collector metamethods for tables and, using the C API, for full userdata (see §2.4). These metamethods, called finalizers, are called when the garbage collector detects that the corresponding table or userdata is dead. Finalizers allow you to coordinate Lua's garbage collection with external resource management such as closing files, network or database connections, or freeing your own memory.
你可以为一个表设置一个meta方法，或者使用C函数的API为fll userdata 设置一个meta方法。
这些meta方法，被称作终结器，是当垃圾清理器检测到对应的table或者userdata死亡的时候。
终结器可以让你通过垃圾回收器去定位需要管控的外部资源，比如关闭文件网络或者数据库连接，再或者是释放你自己管控的内存。
For an object (table or userdata) to be finalized when collected, you must mark it for finalization. You mark an object for finalization when you set its metatable and the metatable has a __gc metamethod. Note that if you set a metatable without a __gc field and later create that field in the metatable, the object will not be marked for finalization.
如果需要一个对象在被认定为垃圾的时候被被终结器终结，那么你必须把他标记为需要终结的。 你可以通过“设置”一个带有__gc方法的表作为该对象的metatable，来标记该对象需要终结器终结，请注意，如果你设置一个metatable没有__gc方法但是后面你又添加上去，那么这个对象不会被标记为需要终结器终结。
When a marked object becomes dead, it is not collected immediately by the garbage collector. Instead, Lua puts it in a list. After the collection, Lua goes through that list. For each object in the list, it checks the object's __gc metamethod: If it is present, Lua calls it with the object as its single argument.
当一个被标记的对象死亡的时候，垃圾回收器不会立即回收，反而是把它放到一个列表里面去，收集完成之后，lua会遍历这个表，对于每一个表中的对象，lua都去检测是否有__gc元方法，如果有，那么会调用这个方法，并且把这个对象作为唯一的参数传入。
At the end of each garbage-collection cycle, the finalizers are called in the reverse order that the objects were marked for finalization, among those collected in that cycle; that is, the first finalizer to be called is the one associated with the object marked last in the program. The execution of each finalizer may occur at any point during the execution of the regular code.
在每次垃圾回收循环的末尾，终结器是按照被标记为终结器的相反顺序调用的。第一个被调用的终结器，是最晚被标记为需要终结的对象。 终结器的调用会发生在任何常规代码的执行点。
Because the object being collected must still be used by the finalizer, that object (and other objects accessible only through it) must be resurrected by Lua. Usually, this resurrection is transient, and the object memory is freed in the next garbage-collection cycle. However, if the finalizer stores the object in some global place (e.g., a global variable), then the resurrection is permanent. Moreover, if the finalizer marks a finalizing object for finalization again, its finalizer will be called again in the next cycle where the object is dead. In any case, the object memory is freed only in a GC cycle where the object is dead and not marked for finalization.
因为这个被收集的对象仍然被终结器使用，那么这个对象（其他对象也是如此？）必须被lua复活，通常这个复活是临时的，这个对象会在下次回收循环被回收。（调用一次setmetatable，被标记一次需要终结）。在任何情况下，被回收的对象都是已经死亡并且没有标记终结器的。
When you close a state (see lua_close), Lua calls the finalizers of all objects marked for finalization, following the reverse order that they were marked. If any finalizer marks objects for collection during that phase, these marks have no effect.
当你关闭一个lua虚拟机的时候，lua会调用所有被标记终结器的对象的终结器，而且是按照被标记的顺序的逆序执行。在这种时候，使用标记方法是无效的。
Finalizers cannot yield nor run the garbage collector. Because they can run in unpredictable times, it is good practice to restrict each finalizer to the minimum necessary to properly release its associated resource.
终结器方法不能延迟（coroutine)或者执行垃圾回收，因为他们执行的时刻是不确定的，使用终结器去释放仅仅与其关联的资源，是最好的方式。（不建议各种魔法使用）
Any error while running a finalizer generates a warning; the error is not propagated.
在终结器里面发生的任何错误都会生成异常警告，这些错误不会传递影响后面的数据处理。（可以任务这个__gc调用类似一个pcall或者trycatch）
