# Getting Started with Javassist
**Shigeru Chiba**

1. Reading and writing bytecode 读写字节码

2. ClassPool ClassPool

3. Class loader  类加载器

4. Introspection and customization 

5. Bytecode level API  字节码级API

6. Generics  泛型

7. Varargs 
8. J2ME 
9. Boxing/Unboxing 装箱/拆箱
10. Debug



### 1. Reading and writing bytecode
### 1. 读写字节码

Javassist is a class library for dealing with Java bytecode. Java bytecode is stored in a binary file called a class file. Each class file contains one Java class or interface.

Javassist 是一个可以处理字节码的类库。Java字节码存储在二进制文件中，我们称为class文件。每个class文件包含一个Java类或者接口。

The class Javassist.CtClass is an abstract representation of a class file. A CtClass (compile-time class) object is a handle for dealing with a class file. The following program is a very simple example:

Javassist.CtClass 类是一个类文件的抽象表现。一个CtClass（编译时的类）对象是一个可用来处理一个类文件的句柄。下面的程序是一个简单的例子

```java 
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("test.Rectangle");
cc.setSuperclass(pool.get("test.Point"));
cc.writeFile();
```

This program first obtains a ClassPool object, which controls bytecode modification with Javassist. 

程序首先获取一个ClassPool对象，可以通过Javassist来控制字节码的修改。

The ClassPool object is a container of CtClass object representing a class file.

ClassPool对象是一个容器，用来存储代表类文件的CtClass对象。

It reads a class file on demand for constructing a CtClass object and records the constructed object for responding later accesses.

它读取一个class文件，按需构造一个CtClass对象，记录着构造好的对象以便后续访问返回。

To modify the definition of a class, the users must first obtain from a ClassPool object a reference to a CtClass object representing that class.

为了修改一个定义好的类，用户必须首先从ClassPool对象中获取一个代表那个类的CtClass对象的引用。

get() in ClassPool is used for this purpose.

ClassPool 的get()方法就是用来做这件事情的。

In the case of the program shown above, the CtClass object representing a class test.Rectangle is obtained from the ClassPool object and it is assigned to a variable cc. The ClassPool object returned by getDefault() searches the default system search path.

在这个程序例子中，这CtClass对象代表着一个test.Rectangle类，这个CtClass对象是通过ClassPool对象获取到的。赋值到一个cc变量上。通过调用getDefault()方法，ClassPool对象会咋默认的系统搜索路径来查找这个类文件。

From the implementation viewpoint, ClassPool is a hash table of CtClass objects, which uses the class names as keys. get() in ClassPool searches this hash table to find a CtClass object associated with the specified key. If such a CtClass object is not found, get() reads a class file to construct a new CtClass object, which is recorded in the hash table and then returned as the resulting value of get().

从内部实现角度，ClassPool 是一个CtClass对象的hash表，使用类名作为关键字。调用get()方法，ClassPool会通过关键字来在hash表中查找对应的CtClass 对象。如果CtClass对象没有找到，那么会读取类文件来构造一个新的CtClas对象。并且记录到hash表中，同时CtClass对象作为get()方法的返回值返回。

The CtClass object obtained from a ClassPool object can be modified (details of how to modify a CtClass will be presented later). In the example above, it is modified so that the superclass of test.Rectangle is changed into a class test.Point. This change is reflected on the original class file when writeFile() in CtClass() is finally called.

通过ClassPool获取到的CtClass是可以修改的（具体如何修改一个CtClass对象的细节将在后续呈现）。在这里例子中，是把test.Rectangle的父类改成了test.Point类。当最后调用了CtClass的writeFile()方法，这个改变影响到原始的类文件。

writeFile() translates the CtClass object into a class file and writes it on a local disk. Javassist also provides a method for directly obtaining the modified bytecode. To obtain the bytecode, call toBytecode():

writeFile()方法将CtClass对象传输到类文件中并写入本地磁盘。Javassist也提供了直接获取到被修改的字节码方法，为了获取字节码可以调用toBytecode()方法。

```java 
byte[] b = cc.toBytecode();
```

You can directly load the CtClass as well:

你也可以同样的加载CtClass。

```java 
Class clazz = cc.toClass();
```

toClass() requests the context class loader for the current thread to load the class file represented by the CtClass. It returns a java.lang.Class object representing the loaded class. For more details, please see this section below.

toClass()方法请求当前线程上下文的类加载器来加载这个CtClass代表的类文件。这个方法返回一个被要求加载的类的java.lang.Class对象。更多细节，请看这部分内容。

#### Defining a new class

#### 定义一个新类

To define a new class from scratch, makeClass() must be called on a ClassPool.

为了从零开始定义一个新类，必须要在一个ClassPool上调用makeClass()方法。

```java 
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.makeClass("Point");
```

This program defines a class Point including no members. Member methods of Point can be created with factory methods declared in CtNewMethod and appended to Point with addMethod() in CtClass.

这个程序定义了一个没有成员变量的Point类，Point的成员方法可以用CtNewMethod来声明工厂方法创建。通过CtClass的addMethod()来添加到Point类中。

makeClass() cannot create a new interface; makeInterface() in ClassPool can do. Member methods in an interface can be created with abstractMethod() in CtNewMethod. Note that an interface method is an abstract method.

makeClass()方法不能创建一个接口；ClassPool的makeInterface()方法可以。可以通过CtNewMehtod的abstractMethod()方法来创建接口中的成员方法。注意接口中的方法是抽象方法。

#### Frozen classes

#### 冻结类

If a CtClass object is converted into a class file by writeFile(), toClass(), or toBytecode(), Javassist freezes that CtClass object. Further modifications of that CtClass object are not permitted. This is for warning the developers when they attempt to modify a class file that has been already loaded since the JVM does not allow reloading a class.

如果一个CtClass对象通过调用writeFile()，toClass()，或者toBytecode()方法 转换到类文件中，Javassist冻结这个CtClass对象。后续关于这个CtClass对象的修改是不被允许的。这个机制会警告开发者，当他们尝试修改一个已经被加载过的类文件，理由是JVM不允许重复加载同一个类。

A frozen CtClass can be defrost so that modifications of the class definition will be permitted. For example,

一个冻结的CtClass可以解冻以便允许修改这个类的定义。例如,

```java 
CtClasss cc = ...;
    :
cc.writeFile();
cc.defrost();
cc.setSuperclass(...);    // OK since the class is not frozen.
```

After defrost() is called, the CtClass object can be modified again.

defrost()方法被调用后，这CtClass对象再一次允许被修改。

If ClassPool.doPruning is set to true, then Javassist prunes the data structure contained in a CtClass object when Javassist freezes that object. To reduce memory consumption, pruning discards unnecessary attributes (attribute_info structures) in that object. For example, Code_attribute structures (method bodies) are discarded. Thus, after a CtClass object is pruned, the bytecode of a method is not accessible except method names, signatures, and annotations. The pruned CtClass object cannot be defrost again. The default value of ClassPool.doPruning is false.

如果ClassPool.doPruning设置为true,当Javassist冻结了对象，Javassist将会清理CtClass对象中包含的数据结构。清理对象中比必要的属性（属性信息结构）是为了较少内存的消耗。例如Code_attribute结构（方法体）会被丢弃。因此，一个CtClass对象被清洗后，除了方法名，签名，注解其他的方法字节内容是不能访问到了。被清洗的CtClass对象不能再次被解冻。ClassPool.doPruning默认值是false.

To disallow pruning a particular CtClass, stopPruning() must be called on that object in advance:

为了拒绝清理一个特别的CtClass对象那个，stopPruning()必须提前调用：

```java 
CtClasss cc = ...;
cc.stopPruning(true);
    :
cc.writeFile();                             // convert to a class file.
// cc is not pruned.
```

The CtClass object cc is not pruned. Thus it can be defrost after writeFile() is called.

cc的CtClass对象不会被清理，因此它可以在执行了writeFile()方法后再次解冻。

Note: While debugging, you might want to temporarily stop pruning and freezing and write a modified class file to a disk drive. debugWriteFile() is a convenient method for that purpose. It stops pruning, writes a class file, defrosts it, and turns pruning on again (if it was initially on).
Class search path

注意：debug期间，你可能想要临时停止清理，冻结对象，把修改过的类文件写到本地磁盘上。debugWriteFile()方法可以便捷的实现这个逻辑。调用这个方法可以停止清理，写类文件到磁盘，解冻类文件，再一次打开清理（如果起初清理开关是打开着）。

The default ClassPool returned by a static method ClassPool.getDefault() searches the same path that the underlying JVM (Java virtual machine) has. If a program is running on a web application server such as JBoss and Tomcat, the ClassPool object may not be able to find user classes since such a web application server uses multiple class loaders as well as the system class loader. In that case, an additional class path must be registered to the ClassPool. Suppose that pool refers to a ClassPool object:

默认的ClassPool对象是由ClassPool.getDefault静态方法返回的。它会优先搜素JVM(Java 虚拟机)的基本路径。***如果一个程序运行在web服务器中，如JBoss、Tomcat，这ClassPool对象可能查找不到用户的类文件,*** 因为一个Web应用服务器使用多个类加载器作为系统类加载器。在这种情况下，一个附加的类路径必须被注册到ClassPool上。假设pool引用一个ClassPool对象：

```java 
pool.insertClassPath(new ClassClassPath(this.getClass()));
```

This statement registers the class path that was used for loading the class of the object that this refers to. You can use any Class object as an argument instead of this.getClass(). The class path used for loading the class represented by that Class object is registered.

这段程序注册了类路径，使用的是加载这个对象所指的类引用。你可以使用任意一个Class对象来代替this.getClass()作为参数。由任意类对象所代表的类路径被注册上用来加载类。

You can register a directory name as the class search path. For example, the following code adds a directory /usr/local/javalib to the search path:

你可以注册一个文件夹名来作为类的查找路径。例如，下面代码添加 /usr/local/javalib 到查找路径:

```java  
ClassPool pool = ClassPool.getDefault();
pool.insertClassPath("/usr/local/javalib");
```

The search path that the users can add is not only a directory but also a URL:

用户添加的查找路径不仅可以是一个文件夹也可以使一个URL:

```java 
ClassPool pool = ClassPool.getDefault();
ClassPath cp = new URLClassPath("www.javassist.org", 80, "/java/", "org.javassist.");
pool.insertClassPath(cp);
```

This program adds "http://www.javassist.org:80/java/" to the class search path. This URL is used only for searching classes belonging to a package org.javassist. For example, to load a class org.javassist.test.Main, its class file will be obtained from:
http://www.javassist.org:80/java/org/javassist/test/Main.class

这段程序添加 "http://www.javassist.org:80/java/" 到类的查找路径。这个URL仅在查找属于org.javassist包下的类时使用。例如，为了加载org.javassist.test.Main类，它的类文件将会从这个地址获取：http://www.javassist.org:80/java/org/javassist/test/Main.class

Furthermore, you can directly give a byte array to a ClassPool object and construct a CtClass object from that array. To do this, use ByteArrayClassPath. For example,

此外，你也可以直接把字节数组传输给ClassPool对象，让其通过字节数组构造CtClass对象。为了可以传输字节数组，使用ByteArrayClassPath,例如，

```java 
ClassPool cp = ClassPool.getDefault();
byte[] b = a byte array;
String name = class name;
cp.insertClassPath(new ByteArrayClassPath(name, b));
CtClass cc = cp.get(name);
```

The obtained CtClass object represents a class defined by the class file specified by b. The ClassPool reads a class file from the given ByteArrayClassPath if get() is called and the class name given to get() is equal to one specified by name.

获取到CtClass对象，其类的定义是由字节数组b来说明的。当ByteArrayClassPath的get方法被调用，ClassPool通过内容读取类文件,get()方法调用的类名要和ByteArrayClassPath构造的保持一致。

If you do not know the fully-qualified name of the class, then you can use makeClass() in ClassPool:

如果你不知道类的全名是什么，你可以使用ClassPool的makeClass()方法构造：

```java 
ClassPool cp = ClassPool.getDefault();
InputStream ins = an input stream for reading a class file;
CtClass cc = cp.makeClass(ins);
```

makeClass() returns the CtClass object constructed from the given input stream. You can use makeClass() for eagerly feeding class files to the ClassPool object. This might improve performance if the search path includes a large jar file. Since a ClassPool object reads a class file on demand, it might repeatedly search the whole jar file for every class file. makeClass() can be used for optimizing this search. The CtClass constructed by makeClass() is kept in the ClassPool object and the class file is never read again.

makeClass()方法返回CtClass对象，通过传入的输入流来构造。如果急切需要将类文件传输到ClassPool对象，你可以使用makeClass()方法，在一个查找路径包含大量jar文件的场景下，这种方式可以提高构造性能。因为如果ClassPool按要求读取一个class文件，它可能会重复搜素全量jar文件对于每个要构造的类文件。使用makeClass()方法可以优化这个搜索。通过makeClass()构造的CtClass会保存在ClassPool中，其所代表的类文件不会被重复调用。

The users can extend the class search path. They can define a new class implementing ClassPath interface and give an instance of that class to insertClassPath() in ClassPool. This allows a non-standard resource to be included in the search path.

用户可以扩展查找路径，他们可以通过实现ClassPath接口来实现一个新类，通过insertClassPath()方法，把一个实例存储到ClassPool中。这种方式允许把非标准资源包含到查找路径中。

### 2. ClassPool

### 2. ClassPool

A ClassPool object is a container of CtClass objects. Once a CtClass object is created, it is recorded in a ClassPool for ever. This is because a compiler may need to access the CtClass object later when it compiles source code that refers to the class represented by that CtClass.

一个ClassPool对象就是关于CtClass对象的容器，一旦一个CtClass对象被创建，它就会永久的记录在ClassPool里。原因是当编译器要编译CtClass对象所代表的类的字节码时，可能会需要访问CtClass 对象。

For example, suppose that a new method getter() is added to a CtClass object representing Point class. Later, the program attempts to compile source code including a method call to getter() in Point and use the compiled code as the body of a method, which will be added to another class Line. If the CtClass object representing Point is lost, the compiler cannot compile the method call to getter(). Note that the original class definition does not include getter(). Therefore, to correctly compile such a method call, the ClassPool must contain all the instances of CtClass all the time of program execution.

例如,当一个getter()方法被添加到一个表示Point类的CtClass对象里，后续编译器试图编译getter()方法在Point类中的源码，使用编译后的字节码作为方法体，将会添加到类的另一行里。如果代表Point类的CtClass查找不到，编译器将无法编译getter()方法，注意原始的类的定义是不包含这个方法。因此为了在程序执行的生命周期期间，编译器可以编译这种方法，ClassPool必须包含所有实例了的CtClass对象。

####Avoid out of memory

####消除内存溢出

This specification of ClassPool may cause huge memory consumption if the number of CtClass objects becomes amazingly large (this rarely happens since Javassist tries to reduce memory consumption in various ways). To avoid this problem, you can explicitly remove an unnecessary CtClass object from the ClassPool. If you call detach() on a CtClass object, then that CtClass object is removed from the ClassPool. For example,

如果ClassPool记录的CtClass对象的数量变的惊人的大的话，可能会发生巨大的内存消耗（这种情况很少会发生，因为Javassist尝试用多种方式减少内存消耗）。为了避免这种问题的发生，你可以明确的把不需要的CtClass对象从ClassPool中移除。你调用CtClass对象上的detach()方法，它就会从ClassPool中移除。示例，

```java 
CtClass cc = ... ;
cc.writeFile();
cc.detach();
```

You must not call any method on that CtClass object after detach() is called. However, you can call get() on ClassPool to make a new instance of CtClass representing the same class. If you call get(), the ClassPool reads a class file again and newly creates a CtClass object, which is returned by get().

调用detach()方法后，你不能再调用关于此CtClass对象的任何其他方法了.然而你可以通过调用ClassPool的get()方法，重新创建一个CtClass实例，代表同样的类文件。当调用get()方法时，ClassPool会再一次读取类文件，崭新的创建一个CtClass对象，作为get()方法的返回值。

Another idea is to occasionally replace a ClassPool with a new one and discard the old one. If an old ClassPool is garbage collected, the CtClass objects included in that ClassPool are also garbage collected. To create a new instance of ClassPool, execute the following code snippet:

另一个种思路，偶尔我们可以用一个新的ClassPool来取代旧的。如果一个旧的ClassPool被垃圾回收，它所包含的CtClass对象同样也会被垃圾回收。创建一个新的ClassPool实例，代码片段如下：

```java 
ClassPool cp = new ClassPool(true);
// if needed, append an extra search path by appendClassPath()
```

This creates a ClassPool object that behaves as the default ClassPool returned by ClassPool.getDefault() does. Note that ClassPool.getDefault() is a singleton factory method provided for convenience. It creates a ClassPool object in the same way shown above although it keeps a single instance of ClassPool and reuses it. A ClassPool object returned by getDefault() does not have a special role. getDefault() is a convenience method.

这种方式和通过调用ClassPool.getDefault()方法创建时一样的。ClassPool.getDefault()是为了便捷，提供的一个单例工厂方法。如上面代码所示，这种方式同样创建一个ClassPool 对象，保持一个单一实例，重复使用。通过getDefault()方法返回的ClassPool没有特别的功能，只是调用getDefault()方法更便利。

Note that new ClassPool(true) is a convenient constructor, which constructs a ClassPool object and appends the system search path to it. Calling that constructor is equivalent to the following code:

new ClassPool(true) 是一种便利的构造函数，构造ClassPool对象的同时，添加系统查找路径到ClassPool中。等同于下面代码构造：

```java 
ClassPool cp = new ClassPool();
cp.appendSystemPath();  // or append another path by appendClassPath()
Cascaded ClassPools
```

If a program is running on a web application server, creating multiple instances of ClassPool might be necessary; an instance of ClassPool should be created for each class loader (i.e. container). The program should create a ClassPool object by not calling getDefault() but a constructor of ClassPool.

如果一个程序运行在web应用服务器上，创建多个ClassPool实例可能是必要的；每一个类加载器（即容器）应该创建一个ClassPool实例。程序应该使用ClassPool的构造函数创建，而不是调用getDefault()方法。

Multiple ClassPool objects can be cascaded like java.lang.ClassLoader. For example,

多个ClassPool对象可以被级联在一起，如果java.lang.ClassLoader那样。例如，

```java 
ClassPool parent = ClassPool.getDefault();
ClassPool child = new ClassPool(parent);
child.insertClassPath("./classes");
```

If child.get() is called, the child ClassPool first delegates to the parent ClassPool. If the parent ClassPool fails to find a class file, then the child ClassPool attempts to find a class file under the ./classes directory.

如果child.get()方法被调用，child ClassPool首先委托parent的ClassPool去查找class file ,如果parent没有查找到，child的ClassPool才会尝试在./classes目录下查找。

If child.childFirstLookup is true, the child ClassPool attempts to find a class file before delegating to the parent ClassPool. For example,

如果child.childFirstLookup 属性设置为true,child ClassPool 会在委托给parent ClassPool查找之前先行查找。例如， 

```java 
ClassPool parent = ClassPool.getDefault();
ClassPool child = new ClassPool(parent);
child.appendSystemPath();         // the same class path as the default one.
child.childFirstLookup = true;    // changes the behavior of the child.
```

Changing a class name for defining a new class

通过改变类名来定义以个新类

A new class can be defined as a copy of an existing class. The program below does that:

一个新类的定义可以通过已存在的类文件的拷贝来操作。下面程序示例：

```java 
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
cc.setName("Pair");
```

This program first obtains the CtClass object for class Point. Then it calls setName() to give a new name Pair to that CtClass object. After this call, all occurrences of the class name in the class definition represented by that CtClass object are changed from Point to Pair. The other part of the class definition does not change.

程序首先获取到Point类的CtClass对象。然后调用setName()方法，给CtClass对象赋值了一个新的名字 Pair. 方法调用后，由CtClass对象所代表的类中，所有类名出现的地方都有Point变成了Pair。类的其他部分定义没有改动。

Note that setName() in CtClass changes a record in the ClassPool object. From the implementation viewpoint, a ClassPool object is a hash table of CtClass objects. setName() changes the key associated to the CtClass object in the hash table. The key is changed from the original class name to the new class name.

注意CtClass的setName()方法改变了ClassPool对象的映射。从实现角度来看，一个ClassPool对象一个关于CtClass 对象的hash表。setName()改变了hash表中映射这个CtClass对象的关键字。这个关键字从原始类名变更为新的类名。
  
Therefore, if get("Point") is later called on the ClassPool object again, then it never returns the CtClass object that the variable cc refers to. The ClassPool object reads a class file Point.class again and it constructs a new CtClass object for class Point. This is because the CtClass object associated with the name Point does not exist any more. See the followings:

因此，如果后续再通过ClassPool对象调用get("Point") 方法，它不会再返回cc变量所引用的CtClass对象。ClassPool对象会再一次读取Point.class类文件，构造一个新的代表Point类的CtClass对象。这是因为关联名为Point的CtClass对象不再存在。看下面的代码：

```java 
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
CtClass cc1 = pool.get("Point");   // cc1 is identical to cc.
cc.setName("Pair");
CtClass cc2 = pool.get("Pair");    // cc2 is identical to cc.
CtClass cc3 = pool.get("Point");   // cc3 is not identical to cc.
```

cc1 and cc2 refer to the same instance of CtClass that cc does whereas cc3 does not. Note that, after cc.setName("Pair") is executed, the CtClass object that cc and cc1 refer to represents the Pair class.

cc1变量和cc2变量引用的是相同实例的CtClass对象，和cc相同，不和cc3变量相同。注意，cc.setName("Pair")方法被之后，cc变量和cc1变量所引用的CtClass对象代表着Pair类。

The ClassPool object is used to maintain one-to-one mapping between classes and CtClass objects. Javassist never allows two distinct CtClass objects to represent the same class unless two independent ClassPool are created. This is a significant feature for consistent program transformation.

ClassPool对象是用来维护类和CtClass对象一对一的映射关系。Javassist重来不会允许两个不同的CtClass对象代表相同的类文件，除非是由两个互相独立的ClassPool所创建。对于程序的一致性转变来说，这是一个重要的特征。

To create another copy of the default instance of ClassPool, which is returned by ClassPool.getDefault(), execute the following code snippet (this code was already shown above):

为了创建由ClassPool.getDefault()方法返回的ClassPool实例的另外一个副本，执行下面的代码片段（这个代码已经展示过了）：

```java 
ClassPool cp = new ClassPool(true);
```

If you have two ClassPool objects, then you can obtain, from each ClassPool, a distinct CtClass object representing the same class file. You can differently modify these CtClass objects to generate different versions of the class.

如果你有两个ClassPool对象，你可以从每一个ClassPool对象中获取一个唯一的CtClass对象，代表相同的类文件。你可以差异地修改这些CtClass对象来产生不同版本的类。

Renaming a frozen class for defining a new class

通过重命名一个冻结的类来定义一个新类

Once a CtClass object is converted into a class file by writeFile() or toBytecode(), Javassist rejects further modifications of that CtClass object. Hence, after the CtClass object representing Point class is converted into a class file, you cannot define Pair class as a copy of Point since executing setName() on Point is rejected. The following code snippet is wrong:

一旦CtClass对象通过调用writeFile()方法或者toBytecode()方法，转化到类文件中。Javassist会拒绝关于CtClass对象的后续的修改。因此，CtClass对象所代表的Point类转化到类文件中后，你不能再定义Pair类作为Point类的一个拷贝，因为执行setName()方法会被拒绝。接下来的代码片段是错误的：

```java 
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
cc.writeFile();
cc.setName("Pair");    // wrong since writeFile() has been called.
```

To avoid this restriction, you should call getAndRename() in ClassPool. For example,

为了消除这个约束，你应该在ClassPool中调用getAndRename()方法，例如,

```java 
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
cc.writeFile();
CtClass cc2 = pool.getAndRename("Point", "Pair");
```

If getAndRename() is called, the ClassPool first reads Point.class for creating a new CtClass object representing Point class. However, it renames that CtClass object from Point to Pair before it records that CtClass object in a hash table. Thus getAndRename() can be executed after writeFile() or toBytecode() is called on the the CtClass object representing Point class.

如果getAndRename()方法被调用，ClassPool首先会读取Point.class创建一个代表Point类的新的CtClass对象。但是，在记录在hash表中将CtClass对象有Point重命名为Pair。因此getAndRename()方法可以在执行writeFile()或者toBytecode()后调用。