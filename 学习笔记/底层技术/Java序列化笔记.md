# Java序列化笔记

[TOC]





## 相关概念

### What

序列化：将对象状态信息转化成可以存储或传输的形式的过程（Java中就是将对象转化成字节序列的过程）

反序列化：从存储文件中恢复对象的过程（Java中就是通过字节序列转化成对象的过程）

### Why

Java对象都存储在JVM的堆和栈内存中，各线程之间可进行对象传输，但无法在进程间进行传输(包含网络传输)；

所以需要把对象进行序列化成字节序列(可传输和存储)；而反序列化就是把传输过来的字节序列再恢复成序列化之前的对象(包含对象状态和属性)；

一般来说，可用于： 

1. 进程之间传输对象（如RPC、RMI通信）
2. 网络通信时进行传输对象
3. 持久化对象时需要将对象序列化

## JDK 序列化

JDK序列化时JDK自带的序列化方式，使用其他也比较方便，只需要序列化的类实现了Serializable接口即可，Serializable接口没有定义任何方法和属性，所以只是起到了标识的作用，表示这个类是可以被序列化的。

注： 如果没有实现Serializable接口而进行序列化操作就会抛出NotSerializableException异常(其父类没实现Serializable接口时，不会抛异常，但父类的属性都不会序列化)。

- 可序列化的字段： 基本类型，对象属性(实现Serializablie接口)、父类的属性变量（父类自己实现Serializablie接口）
- 不可序列化的字段：静态变量、父类的属性变量、关键字transient修饰的变量、没有实现Serializable接口的对象属性;

注： 如果检测到反序列化后的类的serialVersionUID和对象二进制流的serialVersionUID不同，则会抛出
异常。



### 例子

略

### Serializable接口实现原理

Serializable接口是一个空接口，没有定义任何的方法和属性，所以Serialiazable接口的作用就是起到一个标识的作用，源码如下

```java
public interface Serializable {
}
```

Serializable接口既然是标识的作用，那么就需要在实际序列化操作的时候进行识别，而实际的序列化操作是通过ObjectOutputStream 和 ObjectInputStream实现的;

#### ObjectOutputStream源码

我们使用ObjectOuputStream进行序列化时，一般使用下面的代码：

```java
/**序列化对象*/
     private static void serializer(User user)throws Exception{
         ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream(new File("/Users/xxw/testlog/user.txt")));
         outputStream.writeObject(user);
         outputStream.close();
     }
```

##### ObjectOuputStream构造方法：

```java
public ObjectOutputStream(OutputStream out) throws IOException {
    //检查继承权限
    verifySubclass();
    //构造一个BlockDataOutputStream用于向out写入序列化数据
    bout = new BlockDataOutputStream(out);
    //构造一个大小为10，负载因子为3的HandleTable和ReplaceTable
    handles = new HandleTable(10, (float) 3.00);
    subs = new ReplaceTable(10, (float) 3.00);
    //恒为false，除非子类调用protected构造方法
    enableOverride = false;
    writeStreamHeader();
    //将缓存模式打开，写入数据时先写入缓冲区
    bout.setBlockDataMode(true);
    if (extendedDebugInfo) {
        debugInfoStack = new DebugTraceInfoStack();
    } else {
        debugInfoStack = null;
    }
}
```

BlockDataOutputStream将构造参数传入的OutputStream实例包装起来，当向ObjectOutputStream写入序列化数据时，由BlockDataOutputStream来实现实际的写入操作；

verifySubclass方法:

```java
private void verifySubclass() {
    Class<?> cl = getClass();
    //如果构造的不是ObjectOutputStream的子类则直接返回
    if (cl == ObjectOutputStream.class)
        return;
    
    //获取安全管理器检查当前应用是否已创建安全管理器
    SecurityManager sm = System.getSecurityManager();
    if (sm == null)
        return;
    
    //移除Caches中已经失去引用的Class对象
    processQueue(Caches.subclassAuditsQueue, Caches.subclassAudits);
    //将ObjectOutputStream的子类存入Caches
    WeakClassKey key = new WeakClassKey(cl, Caches.subclassAuditsQueue);
    Boolean result = Caches.subclassAudits.get(key);
    if (result == null) {
        result = Boolean.valueOf(auditSubclass(cl));
        Caches.subclassAudits.putIfAbsent(key, result);
    }
    if (result.booleanValue())
        return;
    
    //如果没有权限则抛出SecurityException异常
    sm.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
}
```

该方法主要是判断是否为ObjectOutputStream的子类，并且检查是否拥有SUBCLASS_IMPLEMENTATION_PERMISSION权限，否则抛出SecurityException异常；

另外ObjectOutputStream通过一个Cache静态内部类中的ConcurrentHashMap来缓存ObjectOutputStream子类信的息。Class类通过内部类WeakClassKey（继承WeakReference，将一个弱引用指向一个Class对象）存储；

> ###### 小知识插曲
>
> 我们研究一下这个Caches里是怎么回收的？
>
> ```java
>     //移除Caches中已经失去引用的Class对象
>     processQueue(Caches.subclassAuditsQueue, Caches.subclassAudits);
>     //将ObjectOutputStream的子类存入Caches
>     WeakClassKey key = new WeakClassKey(cl, Caches.subclassAuditsQueue);
>     Boolean result = Caches.subclassAudits.get(key);
>     if (result == null) {
>         result = Boolean.valueOf(auditSubclass(cl));
>         Caches.subclassAudits.putIfAbsent(key, result);
>     }
>     if (result.booleanValue())
>         return;
> ```
>
> 其中auditSubclass是判断ObjectOutputStream的子类没有重写writeUnshared()和putFields()的安全敏感方法；
>
> 我们先看Caches定义：
>
> ```java
>     private static class Caches {
>         /** cache of subclass security audit results */
>         static final ConcurrentMap<WeakClassKey,Boolean> subclassAudits =
>             new ConcurrentHashMap<>();
> 
>         /** queue for WeakReferences to audited subclasses */
>         static final ReferenceQueue<Class<?>> subclassAuditsQueue =
>             new ReferenceQueue<>();
>     }
> ```
>
> 移除Caches中已经失去引用的class对象的方法：
>
> ```java
>     /**
>      * Removes from the specified map any keys that have been enqueued
>      * on the specified reference queue.
>      */
>     static void processQueue(ReferenceQueue<Class<?>> queue,
>                              ConcurrentMap<? extends
>                              WeakReference<Class<?>>, ?> map)
>     {
>         Reference<? extends Class<?>> ref;
>         while((ref = queue.poll()) != null) {
>             map.remove(ref);
>         }
>     }
> ```
>
> 其实就是把Caches.subclassAudits里的Caches.subclassAuditsQueue里所有class对象对象；但除了上面的代码，我们并没有看到有往subclassAuditsQueue添加对象；
>
> 其实原理是：
>
> 1. WeakClassKey是继承于WeakReference，在构造对象时，把Caches.subclassAuditsQueue作为参数传入WeakReference的父构造方法： 
>
>    ```java
>    new WeakClassKey(cl, Caches.subclassAuditsQueue);
>    ```
>
> 2. 而WeakReference的父类Reference的方法，当垃圾收集器把该对象时进行入队时会调用：
>
>    ```java
>        /**
>         * Adds this reference object to the queue with which it is registered,
>         * if any.
>         *
>         * <p> This method is invoked only by Java code; when the garbage collector
>         * enqueues references it does so directly, without invoking this method.
>         *
>         * @return   <code>true</code> if this reference object was successfully
>         *           enqueued; <code>false</code> if it was already enqueued or if
>         *           it was not registered with a queue when it was created
>         */
>        public boolean enqueue() {
>            return this.queue.enqueue(this);
>        }
>    ```
>
>
> 不过没搞弄，为什么要回收这个Class？

构造方法还创建BlockDataOutputStream用于向传入的OutputStream写入对象信息，并构建长度为10，负载因子为3的HandleTable和ReplaceTable。随后，将魔数(0xACED)和版本标识符(0x0005)写入文件头，用来检测是不是一个序列化对象。

```java
final static short STREAM_MAGIC = (short)0xaced;
final static short STREAM_VERSION = 5;
protected void writeStreamHeader() throws IOException {
    bout.writeShort(STREAM_MAGIC);
    bout.writeShort(STREAM_VERSION);
}
```

##### writeObject 写入对象方法：

```java
    public final void writeObject(Object obj) throws IOException {
        // 子类继承时，才为true
        if (enableOverride) {
            writeObjectOverride(obj);
            return;
        }
        try {
            writeObject0(obj, false);
        } catch (IOException ex) {
            if (depth == 0) {
                writeFatalException(ex);
            }
            throw ex;
        }
    }

```

writeObject方法首先会检查是否是ObjectOutputStream的子类，如果是则调用writeObjectOverride方法，这个方法默认实现为空，需要子类根据实际业务需求定制序列化方法。
随后调用writeObject0方法;

>  ###### 小知识插曲
>
>  HandleTable是一个轻量级的哈希表；能根据初始化长度和因子进行自动扩展其存放对象的内存表；主要用于存放一些对象(assign方法)并返回存放的数组索引，及检查某对象是否存放过(lookup方法)并返回存放的数组索引。
>
>  ```java
>          //同个哈希值（前缀）的第一个在Ojbs里的索引
>          private int[] spine;
>          // 对象objs数组索引相应对象的下一个同个哈希值（前缀）的对象在objs里的索引
>          private int[] next;
>          //存储放入的对象
>          private Object[] objs;
>  ```
>
>  我们在这里看一下是怎么扩容：
>
>  在插入对象，会进行检查当前是否足够，否则直接插入
>
>  ```java
>  
>          /**
>           * Assigns next available handle to given object, and returns handle
>           * value.  Handles are assigned in ascending order starting at 0.
>           */
>          int assign(Object obj) {
>              if (size >= next.length) {
>                  growEntries();
>              }
>              if (size >= threshold) {
>                  growSpine();
>              }
>              insert(obj, size);
>              return size++;
>          }
>  ```
>
>  分2种情况扩容：
>
>  1. 插入对象的数量大于等于存放对象的Objs数组，但又没达到阈值（负载因子 * 哈希表索引数组长度）时：
>
>     ```java
>     /**
>      * Increases hash table capacity by lengthening entry arrays.
>      */
>      private void growEntries() {
>          //在原来长度增加一倍+1
>          int newLength = (next.length << 1) + 1;
>                 
>          //定义新长度的数据， 并移动next,objs数组的对象到新数组里
>          int[] newNext = new int[newLength];
>          System.arraycopy(next, 0, newNext, 0, size);
>          next = newNext;
>     
>          Object[] newObjs = new Object[newLength];
>          System.arraycopy(objs, 0, newObjs, 0, size);
>                 objs = newObjs;
>     }
>     ```
>
>     上面这一步，并没有扩大哈希索引数据（spine[]）
>
>  2. 插入对象的数量大于等于阈值时：
>
>     ```java
>         /**
>          * Expands the hash "spine" -- equivalent to increasing the number of
>          * buckets in a conventional hash table.
>          */
>         private void growSpine() {
>             //在原来长度增加一倍+1
>             spine = new int[(spine.length << 1) + 1];
>             //用新长度重新算阈值
>             threshold = (int) (spine.length * loadFactor);
>             Arrays.fill(spine, -1);
>             //重新把之前的存放的对象重新播入一次(需要重新每个对象的哈希索引)；
>             for (int i = 0; i < size; i++) {
>                 insert(objs[i], i);
>             }
>         }
>     ```
>
>     从这2种情况来看， 扩容哈希表索引数组相对代价更高，因为需要重新计算之前的对象哈希索引；
>
>     但适度扩容哈希表，能让重复索引的数更少，让查询的速度更快；
>
>  ReplaceTable数组在HandleTable基础上，存放多一个替换对象：
>
>  ```java
>  
>      /**
>       * Enters mapping from object to replacement object.
>       */
>      void assign(Object obj, Object rep) {
>          int index = htab.assign(obj);
>          while (index >= reps.length) {
>              grow();
>          }
>          reps[index] = rep;
>      }
>  ```
>
>  查找对象时，会优先返回替换对象，没有时，返回查找对象;
>
>  ```java
>      /**
>       * Looks up and returns replacement for given object.  If no
>       * replacement is found, returns the lookup object itself.
>       */
>      Object lookup(Object obj) {
>          int index = htab.lookup(obj);
>          return (index >= 0) ? reps[index] : obj;
>      }
>  ```
>
>  

writeObject0方法:

```java
    /**
     * Underlying writeObject/writeUnshared implementation.
     */
    private void writeObject0(Object obj, boolean unshared)
        throws IOException
    {
        //关闭缓冲模式，直接向目标OutputStream写入数据
        boolean oldMode = bout.setBlockDataMode(false);
        depth++;
        try {
            .....
            
            //序列化对象对应的Class对象的详细信息
            Object orig = obj;
            Class<?> cl = obj.getClass();
            ObjectStreamClass desc;
            for (;;) {
                // REMIND: skip this check for strings/arrays?
                Class<?> repCl;
                // //获取序列化对象对应的Class对象详细信息;
                desc = ObjectStreamClass.lookup(cl, true);
                if (!desc.hasWriteReplaceMethod() ||
                    (obj = desc.invokeWriteReplace(obj)) == null ||
                    (repCl = obj.getClass()) == cl)
                {
                    break;
                }
                cl = repCl;
            }

            .....

            // remaining cases
            //序列化不同类型的对象,分别序列化String、数组、枚举类型
            //另外Integer、Long等都属于实现了Serializable接口的数据类型
            if (obj instanceof String) {
                writeString((String) obj, unshared);
            } else if (cl.isArray()) {
                writeArray(obj, desc, unshared);
            } else if (obj instanceof Enum) {//枚举类型
                writeEnum((Enum<?>) obj, desc, unshared);
            } else if (obj instanceof Serializable) {//一般对象类型的写入(当然需要实现序列化接口)
                writeOrdinaryObject(obj, desc, unshared);
            } else {
                //如果没有实现序列化接口会抛出异常
                if (extendedDebugInfo) {
                    throw new NotSerializableException(
                        cl.getName() + "\n" + debugInfoStack.toString());
                } else {
                    throw new NotSerializableException(cl.getName());
                }
            }
        } finally {
            depth--;
            bout.setBlockDataMode(oldMode);
        }
    }
```

writeString方法的逻辑就是将字符串按字节的方式进行序列化，底层就是通过数组复制的方式获取到char[]，然后写入到缓存的序列化的byte[]数组中：

```java

    /**
     * Writes given string to stream, using standard or long UTF format
     * depending on string length.
     */
    private void writeString(String str, boolean unshared) throws IOException {
        //存放
        handles.assign(unshared ? null : str);
        //返回给定字符串的UTF编码的长度（字节）
        long utflen = bout.getUTFLength(str);
        if (utflen <= 0xFFFF) {
            bout.writeByte(TC_STRING);
            bout.writeUTF(str, utflen);
        } else {
            bout.writeByte(TC_LONGSTRING);
            bout.writeLongUTF(str, utflen);
        }
    }
```

对于自定义可序列化的对象： writeOrdinaryObject方法：

```java

    /**
     * Writes representation of a "ordinary" (i.e., not a String, Class,
     * ObjectStreamClass, array, or enum constant) serializable object to the
     * stream.
     */
    private void writeOrdinaryObject(Object obj,
                                     ObjectStreamClass desc,
                                     boolean unshared)
        throws IOException
    {
        if (extendedDebugInfo) {
            debugInfoStack.push(
                (depth == 1 ? "root " : "") + "object (class \"" +
                obj.getClass().getName() + "\", " + obj.toString() + ")");
        }
        try {
            //检查该需要序列化对象的class, 是否允许序列化，否就直接抛异常；
            desc.checkSerialize();
		   //写入标记为Object类型
            bout.writeByte(TC_OBJECT);
            //写入Class对象的描述信息
            writeClassDesc(desc, false);
            handles.assign(unshared ? null : obj);
            if (desc.isExternalizable() && !desc.isProxy()) {
                //写入实现了Externalizable接口的对象
                writeExternalData((Externalizable) obj);
            } else {
                //写入实现了Serializable
                writeSerialData(obj, desc);
            }
        } finally {
            if (extendedDebugInfo) {
                debugInfoStack.pop();
            }
        }
    }
```

最终按实现了Externalizable接口或Serializable接口分别执行writeExternalData和writeSerialData方法，writeSerialData方法如下：

```java

    /**
     * Writes instance data for each serializable class of given object, from
     * superclass to subclass.
     */
    private void writeSerialData(Object obj, ObjectStreamClass desc)
        throws IOException
    {
        //获取类的描述信息对象(包含其父类), 更高的超类的描述信息对象排到前面
        ObjectStreamClass.ClassDataSlot[] slots = desc.getClassDataLayout();
        //判断该类是否自定义类writeObject方法,如果重写了该方法则按重写的逻辑处理
        for (int i = 0; i < slots.length; i++) {
            ObjectStreamClass slotDesc = slots[i].desc;
            if (slotDesc.hasWriteObjectMethod()) {
                PutFieldImpl oldPut = curPut;
                curPut = null;
                SerialCallbackContext oldContext = curContext;

                if (extendedDebugInfo) {
                    debugInfoStack.push(
                        "custom writeObject data (class \"" +
                        slotDesc.getName() + "\")");
                }
                try {
                    curContext = new SerialCallbackContext(obj, slotDesc);
                    bout.setBlockDataMode(true);
                    //通过反射的方式执行自定义的writeObejct方法
                    slotDesc.invokeWriteObject(obj, this);
                    bout.setBlockDataMode(false);
                    bout.writeByte(TC_ENDBLOCKDATA);
                } finally {
                    curContext.setUsed();
                    curContext = oldContext;
                    if (extendedDebugInfo) {
                        debugInfoStack.pop();
                    }
                }

                curPut = oldPut;
            } else {
                //如果没有自定义writeObject方法则按默认的方法写入属性数据
                defaultWriteFields(obj, slotDesc);
            }
        }
    }
```

先是根据类的描述信息判断是否自定义了序列化方法writeObejct方法，如果自定义了就通过反射执行invokeWriteObejct方法，如果没有自定义则执行defaultWriteFields方法，defaultWriteFields方法逻辑如下:

```java

    /**
     * Fetches and writes values of serializable fields of given object to
     * stream.  The given class descriptor specifies which field values to
     * write, and in which order they should be written.
     */
    private void defaultWriteFields(Object obj, ObjectStreamClass desc)
        throws IOException
    {
        //校验对象的类信息是否和类描述信息一致
        Class<?> cl = desc.forClass();
        if (cl != null && obj != null && !cl.isInstance(obj)) {
            throw new ClassCastException();
        }

        desc.checkDefaultSerialize();

        //获取原始数据的长度
        int primDataSize = desc.getPrimDataSize();
        if (primVals == null || primVals.length < primDataSize) {
            primVals = new byte[primDataSize];
        }
        desc.getPrimFieldValues(obj, primVals);
        bout.write(primVals, 0, primDataSize, false);

        //获取所有属性
        ObjectStreamField[] fields = desc.getFields(false);
        //获取对象类型属性
        Object[] objVals = new Object[desc.getNumObjFields()];
        int numPrimFields = fields.length - objVals.length;
        desc.getObjFieldValues(obj, objVals);
        //遍历对象类型属性数组
        for (int i = 0; i < objVals.length; i++) {
            if (extendedDebugInfo) {
                debugInfoStack.push(
                    "field (class \"" + desc.getName() + "\", name: \"" +
                    fields[numPrimFields + i].getName() + "\", type: \"" +
                    fields[numPrimFields + i].getType() + "\")");
            }
            try {
                //递归写入对象类型的属性
                writeObject0(objVals[i],
                             fields[numPrimFields + i].isUnshared());
            } finally {
                if (extendedDebugInfo) {
                    debugInfoStack.pop();
                }
            }
        }
    }
```

序列化的整体逻辑就是遍历对象的所有属性，递归执行序列化方法，直到序列化的对象是String、Array或者是Eunm类，则按String、Array、Enum的序列化方式写入字节流中。

#### ObjectInputStream源码

反序列化对象，我们一般使用下面的代码：

```java
     /**反序列化对象*/
     private static User derializer()throws Exception{
         ObjectInputStream inputStream = new ObjectInputStream(new FileInputStream(new File("/Users/xxw/testlog/user.txt")));
         User user = (User) inputStream.readObject();
         inputStream.close();
         return user;
     }
```

下面我们看看readObject的源码：

```java
   //从ObjectInputStream读取对象。读取对象的类、类的签名、类及其所有超类型的非瞬态和非静态字段的值。
   public final Object readObject()
        throws IOException, ClassNotFoundException
    {
        if (enableOverride) {
            return readObjectOverride();
        }

        // if nested read, passHandle contains handle of enclosing object
        int outerHandle = passHandle;
        try {
            //执行反序列化方法
            Object obj = readObject0(false);
            handles.markDependency(outerHandle, passHandle);
            ClassNotFoundException ex = handles.lookupException(passHandle);
            if (ex != null) {
                throw ex;
            }
            if (depth == 0) {
                vlist.doCallbacks();
            }
            return obj;
        } finally {
            passHandle = outerHandle;
            if (closed && depth == 0) {
                clear();
            }
        }
    }
```

```java

    /**
     * Underlying readObject implementation.
     */
    private Object readObject0(boolean unshared) throws IOException {
        //
        boolean oldMode = bin.getBlockDataMode();
        if (oldMode) {
            int remain = bin.currentBlockRemaining();
            if (remain > 0) {
                throw new OptionalDataException(remain);
            } else if (defaultDataEnd) {
                /*
                 * Fix for 4360508: stream is currently at the end of a field
                 * value block written via default serialization; since there
                 * is no terminating TC_ENDBLOCKDATA tag, simulate
                 * end-of-custom-data behavior explicitly.
                 */
                throw new OptionalDataException(true);
            }
            bin.setBlockDataMode(false);
        }

        byte tc;
        //读取int数据，该数据表示属性的类型(序列化时候写入的时候写入属性之前都会写入int值表示属性的类型)
        while ((tc = bin.peekByte()) == TC_RESET) {
            bin.readByte();
            handleReset();
        }

        depth++;
        try {
            //判断读取的int数据表示当前读取的是什么类型的数据结构不同的类型数据分别执行不同的解析方法
            switch (tc) {
                //null对象
                case TC_NULL:
                    return readNull();
			  
                case TC_REFERENCE:
                    return readHandle(unshared);
				// Class对象
                case TC_CLASS:
                    return readClass(unshared);

                case TC_CLASSDESC:
                case TC_PROXYCLASSDESC:
                    return readClassDesc(unshared);

                case TC_STRING:
                case TC_LONGSTRING:
                    return checkResolve(readString(unshared));

                case TC_ARRAY:
                    return checkResolve(readArray(unshared));

                case TC_ENUM:
                    return checkResolve(readEnum(unshared));

                case TC_OBJECT:
                    return checkResolve(readOrdinaryObject(unshared));

                case TC_EXCEPTION:
                    IOException ex = readFatalException();
                    throw new WriteAbortedException("writing aborted", ex);

                case TC_BLOCKDATA:
                case TC_BLOCKDATALONG:
                    if (oldMode) {
                        bin.setBlockDataMode(true);
                        bin.peek();             // force header read
                        throw new OptionalDataException(
                            bin.currentBlockRemaining());
                    } else {
                        throw new StreamCorruptedException(
                            "unexpected block data");
                    }

                case TC_ENDBLOCKDATA:
                    if (oldMode) {
                        throw new OptionalDataException(true);
                    } else {
                        throw new StreamCorruptedException(
                            "unexpected end of block data");
                    }

                default:
                    throw new StreamCorruptedException(
                        String.format("invalid type code: %02X", tc));
            }
        } finally {
            depth--;
            bin.setBlockDataMode(oldMode);
        }
    }

```

```java

    /**
     * If resolveObject has been enabled and given object does not have an
     * exception associated with it, calls resolveObject to determine
     * replacement for object, and updates handle table accordingly.  Returns
     * replacement object, or echoes provided object if no replacement
     * occurred.  Expects that passHandle is set to given object's handle prior
     * to calling this method.
     */
    private Object checkResolve(Object obj) throws IOException {
        if (!enableResolve || handles.lookupException(passHandle) != null) {
            return obj;
        }
        Object rep = resolveObject(obj);
        if (rep != obj) {
            handles.setObject(passHandle, rep);
        }
        return rep;
    }
```

看上面的代码，可知道真正解析返回对象是：readString()、readArray()、readEnum()、readOrdinaryObject()方法； 

readString：

```java

    /**
     * Reads in and returns new string.  Sets passHandle to new string's
     * assigned handle.
     */
    private String readString(boolean unshared) throws IOException {
        String str;
        byte tc = bin.readByte();
        //根据不同的String，直接从流中读取字符串
        switch (tc) {
            case TC_STRING:
                str = bin.readUTF();
                break;

            case TC_LONGSTRING:
                str = bin.readLongUTF();
                break;

            default:
                throw new StreamCorruptedException(
                    String.format("invalid type code: %02X", tc));
        }
        passHandle = handles.assign(unshared ? unsharedMarker : str);
        handles.finish(passHandle);
        return str;
    }
```

解析数组和枚举的过程与上面解析字符串差不多；

下面来解析对象readOrdinaryObject()方法：

```java

    /**
     * Reads and returns "ordinary" (i.e., not a String, Class,
     * ObjectStreamClass, array, or enum constant) object, or null if object's
     * class is unresolvable (in which case a ClassNotFoundException will be
     * associated with object's handle).  Sets passHandle to object's assigned
     * handle.
     */
    private Object readOrdinaryObject(boolean unshared)
        throws IOException
    {
        //判断是否为对象类型
        if (bin.readByte() != TC_OBJECT) {
            throw new InternalError();
        }
	    //读取类描述信息对象
        ObjectStreamClass desc = readClassDesc(false);
        desc.checkDeserialize();

        //判断是否为非法类描述符
        Class<?> cl = desc.forClass();
        if (cl == String.class || cl == Class.class
                || cl == ObjectStreamClass.class) {
            throw new InvalidClassException("invalid class descriptor");
        }

        Object obj;
        try {
            //isInstantiable方法是判断是否有public的无参构造方法表示是否可以初始化对象,
            //然后通过Constructor的newInstance方法初始化对象(反射)
            obj = desc.isInstantiable() ? desc.newInstance() : null;
        } catch (Exception ex) {
            throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
        }

        passHandle = handles.assign(unshared ? unsharedMarker : obj);
        ClassNotFoundException resolveEx = desc.getResolveException();
        if (resolveEx != null) {
            handles.markException(passHandle, resolveEx);
        }
        
        if (desc.isExternalizable()) {
            //反序列化实现了Externalizable接口的对象
            readExternalData((Externalizable) obj, desc);
        } else {
            //反序列化实现了Serializable接口的对象
            readSerialData(obj, desc);
        }

        handles.finish(passHandle);

        if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
        {
            Object rep = desc.invokeReadResolve(obj);
            if (unshared && rep.getClass().isArray()) {
                rep = cloneArray(rep);
            }
            if (rep != obj) {
                handles.setObject(passHandle, obj = rep);
            }
        }

        return obj;
    }
```

```java

    /**
     * Reads (or attempts to skip, if obj is null or is tagged with a
     * ClassNotFoundException) instance data for each serializable class of
     * object in stream, from superclass to subclass.  Expects that passHandle
     * is set to obj's handle before this method is called.
     */
    private void readSerialData(Object obj, ObjectStreamClass desc)
        throws IOException
    {
        //读取其类描述信息，更高的超类放到数组的前面；
        ObjectStreamClass.ClassDataSlot[] slots = desc.getClassDataLayout();
        //遍历类描述信息
        for (int i = 0; i < slots.length; i++) {
            ObjectStreamClass slotDesc = slots[i].desc;

            if (slots[i].hasData) {
                if (obj == null || handles.lookupException(passHandle) != null) {
                    //没有数据可以解析
                    defaultReadFields(null, slotDesc); // skip field values
                } else if (slotDesc.hasReadObjectMethod()) {
                    //如果有自定义的readObejct方法则按自定义的逻辑执行
                    ThreadDeath t = null;
                    boolean reset = false;
                    SerialCallbackContext oldContext = curContext;
                    if (oldContext != null)
                        oldContext.check();
                    try {
                        curContext = new SerialCallbackContext(obj, slotDesc);

                        bin.setBlockDataMode(true);
                        slotDesc.invokeReadObject(obj, this);
                    } catch (ClassNotFoundException ex) {
                        /*
                         * In most cases, the handle table has already
                         * propagated a CNFException to passHandle at this
                         * point; this mark call is included to address cases
                         * where the custom readObject method has cons'ed and
                         * thrown a new CNFException of its own.
                         */
                        handles.markException(passHandle, ex);
                    } finally {
                        do {
                            try {
                                curContext.setUsed();
                                if (oldContext!= null)
                                    oldContext.check();
                                curContext = oldContext;
                                reset = true;
                            } catch (ThreadDeath x) {
                                t = x;  // defer until reset is true
                            }
                        } while (!reset);
                        if (t != null)
                            throw t;
                    }

                    /*
                     * defaultDataEnd may have been set indirectly by custom
                     * readObject() method when calling defaultReadObject() or
                     * readFields(); clear it to restore normal read behavior.
                     */
                    defaultDataEnd = false;
                } else {
                    //没有自定义的readObject方法则按默认的解析方法进行解析
                    defaultReadFields(obj, slotDesc);
                }

                if (slotDesc.hasWriteObjectData()) {
                    skipCustomData();
                } else {
                    bin.setBlockDataMode(false);
                }
            } else {
                if (obj != null &&
                    slotDesc.hasReadObjectNoDataMethod() &&
                    handles.lookupException(passHandle) == null)
                {
                    slotDesc.invokeReadObjectNoData(obj);
                }
            }
        }
    }
```

先判断是否自定义了反序列化的方法readObject，如果有就按自定义的执行，如果没有就执行默认的反序列化方法defaultReadFields方法执行：

```java

    /**
     * Reads in values of serializable fields declared by given class
     * descriptor.  If obj is non-null, sets field values in obj.  Expects that
     * passHandle is set to obj's handle before this method is called.
     */
    private void defaultReadFields(Object obj, ObjectStreamClass desc)
        throws IOException
    {
        Class<?> cl = desc.forClass();
        if (cl != null && obj != null && !cl.isInstance(obj)) {
            throw new ClassCastException();
        }

        int primDataSize = desc.getPrimDataSize();
        if (primVals == null || primVals.length < primDataSize) {
            primVals = new byte[primDataSize];
        }
        bin.readFully(primVals, 0, primDataSize, false);
        if (obj != null) {
            //设置基本数据类型的属性,只包括八大基本数据类型Integer、Long等不在内
            desc.setPrimFieldValues(obj, primVals);
        }

        int objHandle = passHandle;
        ObjectStreamField[] fields = desc.getFields(false);
        Object[] objVals = new Object[desc.getNumObjFields()];
        int numPrimFields = fields.length - objVals.length;
        for (int i = 0; i < objVals.length; i++) {
            ObjectStreamField f = fields[numPrimFields + i];
            //递归设置对象类型属性
            objVals[i] = readObject0(f.isUnshared());
            if (f.getField() != null) {
                handles.markDependency(objHandle, passHandle);
            }
        }
        if (obj != null) {
            desc.setObjFieldValues(obj, objVals);
        }
        passHandle = objHandle;
    }

```

## 自定义序列化和反序列化

JDK除了支持默认的序列化逻辑，还可以自行定义序列化的方式，只需要对应的类中重写writeObject方法和readObject方法即可：

```java
/**自定义序列化方法*/
private void writeObject(ObjectOutputStream outputStream)
            throws IOException
    {
        outputStream.writeLong(this.userId);
        outputStream.writeObject(this.userName);
        outputStream.writeObject(this.password);
    }

/**自定义反序列化方法*/
private void readObject(ObjectInputStream inputStream){
        try {
            this.userId = inputStream.readLong();
            this.password = (String)inputStream.readObject();
            this.userName = (String)inputStream.readObject();
//            this.addr = (String)inputStream.readObject();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

注：

1. writeObject和readObject方法必须同时重写，如果只重写一个，那么会抛异常，因为序列化之后的是字节序列，是严格按字节序列的顺序来解析的
2. writeObject和readObject方法中的序列化和反序列化字段必须完全一致，也就是序列化字段的数量和顺序（writeObejct可以多write几个字段，但是顺序必须在后面，readObject不可多字段，否则解析会抛异常）
3. writeObejct和readObject都必须是private修饰的，public修饰的不起作用

## 问题总结 

#### 序列化之前和反序列化之后的对象可能是同一个对象吗？

可能，反序列化是通过对象的构造方法进行初始化的，所以正常情况下不会是之前的同一个对象，而且序列化之前的对象可能都已经被垃圾回收的；
但是在单例模式下是可以通过自定义反序列化逻辑进行操作的，详情如下第二条。

#### 序列化和反序列化破坏单例模式的解决？

解决办法如下：在单例类中定义方法readResolve方法，并且返回Object对象

```
 private Object readResolve(){
        return user;
    }
```

 原理请参考：反序列化关键类ObjectInputStream类中的readOrdinaryObject方法 --> 先判断该类中是否包含有readResolveMethod方法，如果有就通过readResolve方法返回结果，再将返回的结果赋值给通过反序列化生成的对象，所以只需要readResolve方法返回的也是同样的单例对象即可。

#### 实现了Serializable接口的serialVersionUID作用是什么？

在反序列化时，JVM需要知道所属的class文件，在序列化的时候JVM会记录class文件的版本号，默认是JVM自动生成，也可以手动定义。反序列化时JVM会按版本号找指定版本的class文件进行反序列化，

如果class文件有版本号在序列化和反序列化时不一致就会导致反序列化失败，会抛异常提示版本号不一致;

#### JDK序列化的整体逻辑流程？

序列化流程：

1. 调用ObjectStreamClass.lookup方法得到类的描述对象（保护类的基本数据属性、对象属性、是否包含readObject、writeObject等和序列化相关的方法等）
2. 判断对象的类型如果是String、Array、Eunm则直接转化为字节序列，如果是对象类型（实现了Serializable接口的属性）则执行writeOrdinaryObject方法
3. 如果属性所属的类自定义了writeObject方法则通过反射的方式invoke自定义的writeObject方法，如果没有自定义则按默认的序列化方法defaultWriteFields方法
4. 递归所有属性，如果是基本数据类型则直接写入流中，如果不是基本数据类型则递归属性继续执行writeObject方法写入流中
5. 最终的结果是将对象转化成二进制流中，所以需要依次按顺序将对象的信息写入到流中

反序列化流程：

1. 从字节流中读取字节，判断字节对应所属的类型（Class、String、Object等等）
2. 根据不同的类型，继续读取对应的字节数据，如果不是对象类型那么就可以直接解析出基本数据类型的数据
3. 如果是对象类型，先读取字节TC_CLASSDESC，从而解析读取并解析出ObjectStreamClass对象，也就是类的描述信息对象
4. 通过ObjectStreamClass可以得到类的构造方法，并通过构造器的newInstance初始化对象，然后开始给属性赋值
5. 判断是否有自定义readObject，有的话通过反射执行readObject方法反序列化，如果没有则按默认的反序列化方法defaultReadFields方法
6. 依次从流中读取属性的值进行赋值，如果是对象类型则递归继续执行readObject方法进行读取流操作

### 各种序列化方式的对比

目前主流的序列化方式包括JDK序列化、Hessian、JSON、Kryo、ProtoBuf等，对比如下：

|                | JDK                                                          | Hessian                                                      | JSON                                                         | Kryo                                                         | ProtoBuf                                                     |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 序列化数据大小 | 很大                                                         | 较小                                                         | 较大                                                         | 很小                                                         | 很小                                                         |
| 序列化速度     | 很慢                                                         | 较快                                                         | 较快                                                         | 很快                                                         | 很快                                                         |
| 总结           | 无论是序列化的效率还是序列化的数据大小都没有竞争力，另外也不支持跨语言，唯一的优点是不需要依赖第三方Jar包就可以实现 | 效率比JDK要好，数据大小也比JDK要小，且均比JSON要好。且可以跨语言 | 效率和数据量大小相比于JDK序列化优化了很多，但是整体效率不高。优点是序列化后的数据是字符串，可观性较好且支持跨语言 | 效率非常高，序列化数据非常小，基本上和ProtoBuf整体性能一致，但是不支持跨语言。 | 效率非常高，序列化数据非常小，但是需要提前定义序列化的数据结构，且提前静态编译，使用成本变高 |

总结：

JDK序列化：JDK序列化会把对象类的描述和所有属性的元数据都序列化为字节流，另外继承的元数据也会序列化，所以导致序列化的元素较多且字节流很大，但是由于序列化了所有信息所以相对而言更可靠。但是如果只需要序列化属性的值时就比较浪费。

Hessian序列化：Hessian序列化将基本数据类型进行了标记，如Integer类型标记为I，字符串类型标记为S，Map类型标记为M，类描述信息标记为C，通过各种标记大大简化了属性类信息的序列化过程。另外对于自定义类型如果已经序列化过，会用R进行标记，并

加上引用的位置。通常类的描述信息比字段的值信息要大，所以通过简化类的描述信息来大幅度提升了序列化的效果以及字节流的减少。

Kryo序列化：Kryo序列化使用之前可以对涉及到的Class进行注册，注册之后Kryo对象会缓存Class的注册信息，后续序列化和反序列化时都只需要写入Class注册之后绑定的ID即可，比完整的Class信息会效率高很多。但是在注册的时候必须保证注册的顺序不允许发生改变，特别是在集群部署的情况下。A服务器注册的顺序和B服务器注册的顺序不一样，那么A服务器序列化的数据B服务器是无法反序列化的。如果需要新增注册的类只能在后面继续添加注册的类信息。

本文笔记于：

> https://blog.csdn.net/abc123lzf/article/details/82318148
>https://www.cnblogs.com/jackion5/p/11819393.html