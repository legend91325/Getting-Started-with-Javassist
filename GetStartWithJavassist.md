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

###3. Class loader

###3. 类加载器

If what classes must be modified is known in advance, the easiest way for modifying the classes is as follows:

如果提前知道哪些类是要修改的，最简单的操作是如下：

1. Get a CtClass object by calling ClassPool.get(),
2. Modify it, and
3. Call writeFile() or toBytecode() on that CtClass object to obtain a modified class file.
If whether a class is modified or not is determined at load time, the users must make Javassist collaborate with a class loader. Javassist can be used with a class loader so that bytecode can be modified at load time. The users of Javassist can define their own version of class loader but they can also use a class loader provided by Javassist.


1. 通过调用ClassPool.get()来获取一个CtClass对象，
2. 修改它然后
3. 调用CtClass对象的writeFile()或者toBytecode()方法获取被修改的类文件。
无论是否一个类在加载期间要被修改，用户都必须让Javassist关联上一个类加载器。这么做以便在加载期间可以修改字节码内容，用户可以提供自己定义的类加载器或者使用Javassist提供的。


#### 3.1 The toClass method in CtClass

#### 3.1 CtClass的toClass方法

The CtClass provides a convenience method toClass(), which requests the context class loader for the current thread to load the class represented by the CtClass object. To call this method, the caller must have appropriate permission; otherwise, a SecurityException may be thrown.

CtClass提供了一个便捷的方法toClass()，用来请求当前线程上下文的类加载器来加载CtClass对象所代表的类。为了调用这个方法，调用者必须有合适的权限，否则会有SecurityException异常抛出。

The following program shows how to use toClass():

下面程序展示如何使用toClass()：

```java 
public class Hello {
    public void say() {
        System.out.println("Hello");
    }
}

public class Test {
    public static void main(String[] args) throws Exception {
        ClassPool cp = ClassPool.getDefault();
        CtClass cc = cp.get("Hello");
        CtMethod m = cc.getDeclaredMethod("say");
        m.insertBefore("{ System.out.println(\"Hello.say():\"); }");
        Class c = cc.toClass();
        Hello h = (Hello)c.newInstance();
        h.say();
    }
}
```

Test.main() inserts a call to println() in the method body of say() in Hello. Then it constructs an instance of the modified Hello class and calls say() on that instance.

Test.main()中将一个println()方法插入到Hello的say()方法体中。然后构造了一个修改后的Hello类的实例，调用say()方法。

Note that the program above depends on the fact that the Hello class is never loaded before toClass() is invoked. If not, the JVM would load the original Hello class before toClass() requests to load the modified Hello class. Hence loading the modified Hello class would be failed (LinkageError is thrown). For example, if main() in Test is something like this:

注意这段程序是依赖条件是，在调用toClass()方法前，Hello类从来没有被加载过。如果不是这样的话，而是在toClass()请求加载修改的Hello类之前，JVM已经加载过原始的Hello类的话。请求JVM加载修改之后的Hello类将会不成功的（LinkageError 错误会被抛出）。例如，如果Test文件中main()如下面所示：

```java 
public static void main(String[] args) throws Exception {
    Hello orig = new Hello();
    ClassPool cp = ClassPool.getDefault();
    CtClass cc = cp.get("Hello");
        :
}
```

then the original Hello class is loaded at the first line of main and the call to toClass() throws an exception since the class loader cannot load two different versions of the Hello class at the same time.

原始的Hello类在main第一行被加载，然后调用toClass()方法会抛出异常，因为类加载器不能同时加载两个不同版本的Hello类。

If the program is running on some application server such as JBoss and Tomcat, the context class loader used by toClass() might be inappropriate. In this case, you would see an unexpected ClassCastException. To avoid this exception, you must explicitly give an appropriate class loader to toClass(). For example, if bean is your session bean object, then the following code:

如果程序运行在应用服务器上，例如JBoss，Tomcat，通过当前线程上下文调用toClass可能不合适，在这种场景下，你将会接收到一个不期望的ClassCastException异常。为了消除这种异常，你必须明确的指定类加载器来做toClass()操作。如果bean属于你的会话bean对象，按下面的代码：

```java 
CtClass cc = ...;
Class c = cc.toClass(bean.getClass().getClassLoader());
```

would work. You should give toClass() the class loader that has loaded your program (in the above example, the class of the bean object).

这种方式将会奏效，你应该给toClass()方法那个加载这个程序的类加载器（在上面这个例子，指定是加载bean对象的类）

toClass() is provided for convenience. If you need more complex functionality, you should write your own class loader.

toClass()方法提供了便捷，但是如果你需要复杂的功能，你应该写自己的类加载器。

####3.2 Class loading in Java

####3.2 java中的类加载

In Java, multiple class loaders can coexist and each class loader creates its own name space. Different class loaders can load different class files with the same class name. The loaded two classes are regarded as different ones. This feature enables us to run multiple application programs on a single JVM even if these programs include different classes with the same name.

在JAVA中，多个类加载器可以共存，每个类加载器可以其创建自己的命名空间。不同的类加载器可以加载同类名的不同类文件。被加载的两个类文件是被当做不同的类存在的。这个特点可以让我们在单一的JVM环境中运行不同的应用程序，即使这些应用程序包含同名的不同类。

Note: The JVM does not allow dynamically reloading a class. Once a class loader loads a class, it cannot reload a modified version of that class during runtime. Thus, you cannot alter the definition of a class after the JVM loads it. However, the JPDA (Java Platform Debugger Architecture) provides limited ability for reloading a class. See Section 3.6.

注意：JVM不允许动态重新加载类，一旦一个类被类加载器加载，在JVM运行期间不能再被另一个修改的版本。因此你无法再JVM加载之后去修改一个类的定义。但是JPDA(JAVA Platform Debugger Architecture)提供受限制的重新加载能力。看3.6部分了解详情。

If the same class file is loaded by two distinct class loaders, the JVM makes two distinct classes with the same name and definition. The two classes are regarded as different ones. Since the two classes are not identical, an instance of one class is not assignable to a variable of the other class. The cast operation between the two classes fails and throws a ClassCastException.

如果同一个类文件被两个独立的类加载器加载，JVM将会创建两个同名同样的定义但是不同的类。这两个类是被认为不同的。因为这两个类不是同一个，因此一个类的类实例是不能"assignable"另一个类的变量。这cast操作对于这两个类之间是会失败，抛出ClassCastException异常。

For example, the following code snippet throws an exception:

例如，下面代码片段会抛出异常：

```java 
MyClassLoader myLoader = new MyClassLoader();
Class clazz = myLoader.loadClass("Box");
Object obj = clazz.newInstance();
Box b = (Box)obj;    // this always throws ClassCastException.
```

The Box class is loaded by two class loaders. Suppose that a class loader CL loads a class including this code snippet. Since this code snippet refers to MyClassLoader, Class, Object, and Box, CL also loads these classes (unless it delegates to another class loader). Hence the type of the variable b is the Box class loaded by CL. On the other hand, myLoader also loads the Box class. The object obj is an instance of the Box class loaded by myLoader. Therefore, the last statement always throws a ClassCastException since the class of obj is a different verison of the Box class from one used as the type of the variable b.

Box类有两个类加载器加载。假设在这代码片段中包含一个由CL类加载器加载的类。因为这代码中引用了MyClassLoader类加载器，类，对象和Box类型，CL加载器同样也加载了这些操作（除非它有委托另一个加载器加载）。因此变量b是由CL加载器加载的Box类型。另一个边，myLoader加载器同样加载了Box类。obj对象是由myLoader加载器加载的Box类的实例。因此最后一段代码会抛出ClassCastException异常，因为obj对象的类和b变量所归属的类不是同一个版本。

Multiple class loaders form a tree structure. Each class loader except the bootstrap loader has a parent class loader, which has normally loaded the class of that child class loader. Since the request to load a class can be delegated along this hierarchy of class loaders, a class may be loaded by a class loader that you do not request the class loading. Therefore, the class loader that has been requested to load a class C may be different from the loader that actually loads the class C. For distinction, we call the former loader the initiator of C and we call the latter loader the real loader of C.

多个类加载器形成树形结构。除了启动类加载器其他都会有父类加载器， 。因为请求加载一个类会沿着这层级结构去委托加载，一个类的加载可能不是由你所请求加载的类加载器所加载。因此请求加载类C的类加载器可能和实际加载C类的类加载器是不相同的。为了区分，我们称前一个类加载器为C类的初始化者，后一个为C类的实际加载者。

Furthermore, if a class loader CL requested to load a class C (the initiator of C) delegates to the parent class loader PL, then the class loader CL is never requested to load any classes referred to in the definition of the class C. CL is not the initiator of those classes. Instead, the parent class loader PL becomes their initiators and it is requested to load them. The classes that the definition of a class C referes to are loaded by the real loader of C.

此外，如果一个CL类加载器请求加载类C(C的初始化类加载器)，将请求委托给父类PL类加载器，后续CL类加载器

To understand this behavior, let's consider the following example.

为了理解这个行为，让我们看下面的例子.

```java 
public class Point {    // loaded by PL
    private int x, y;
    public int getX() { return x; }
        :
}

public class Box {      // the initiator is L but the real loader is PL
    private Point upperLeft, size;
    public int getBaseX() { return upperLeft.x; }
        :
}

public class Window {    // loaded by a class loader L
    private Box box;
    public int getBaseX() { return box.getBaseX(); }
}
```

Suppose that a class Window is loaded by a class loader L. Both the initiator and the real loader of Window are L. Since the definition of Window refers to Box, the JVM will request L to load Box. Here, suppose that L delegates this task to the parent class loader PL. The initiator of Box is L but the real loader is PL. In this case, the initiator of Point is not L but PL since it is the same as the real loader of Box. Thus L is never requested to load Point.

假如类Window是由类加载器L所加载。初始化和真正加载Window都是L。因为Window的定义中引用了Box类，JVM请求L类加载器加载Box。假如这里，L类加载器将这个任务委托给PL类加载器。初始化Box的类加载器是L,实际加载Box的是PL类加载器。在这个例子中，Point类的初始化不是由L类加载器而是由PL类加载器，是和Box的实际加载器相同。因此L类加载器从没请求加载过Point类。

Next, let's consider a slightly modified example.

接下来让我们看下稍做修改的例子。

```java 
public class Point {
    private int x, y;
    public int getX() { return x; }
        :
}

public class Box {      // the initiator is L but the real loader is PL
    private Point upperLeft, size;
    public Point getSize() { return size; }
        :
}

public class Window {    // loaded by a class loader L
    private Box box;
    public boolean widthIs(int w) {
        Point p = box.getSize();
        return w == p.getX();
    }
}
```

Now, the definition of Window also refers to Point. In this case, the class loader L must also delegate to PL if it is requested to load Point. You must avoid having two class loaders doubly load the same class. One of the two loaders must delegate to the other.

现在Window类的定义同样引用了Point类，在这个案例中，当L类加载器被请求加载Point时，它必须同样委托给PL类加载器。你必须消除同一个类被两个类加载器加载。两个类加载器中其一必须委托给其他类加载器去加载。

If L does not delegate to PL when Point is loaded, widthIs() would throw a ClassCastException. Since the real loader of Box is PL, Point referred to in Box is also loaded by PL. Therefore, the resulting value of getSize() is an instance of Point loaded by PL whereas the type of the variable p in widthIs() is Point loaded by L. The JVM regards them as distinct types and thus it throws an exception because of type mismatch.

如果Point类要求加载的请求没有被L类加载器委托给PL类加载器的话，widthIs()方法会抛出ClassCastException异常。因为实际加载Box类的类加载器是PL,Point类在Box中有引用，所以同样也被PL类加载器加载。因此getSize()方法返回的结果值是由PL加载的Point实例，同样widthIs()方法的Point类的变量p是由L类加载器加载的。JVM认为这是两个不同类型的类，所以会抛出一个类型一致的异常。

This behavior is somewhat inconvenient but necessary. If the following statement:

这种行为某种程度上不是便利的，但是必要的限制。如果下面的代码：

```java 
Point p = box.getSize();
```

did not throw an exception, then the programmer of Window could break the encapsulation of Point objects. For example, the field x is private in Point loaded by PL. However, the Window class could directly access the value of x if L loads Point with the following definition:

不再抛出一个异常，Window类的开发者将会到Point对象的封装性。比如，由PL类加载器的Point中属性x是私有的。但是如果L类加载器按下面来加载Point类的话，Window类可以直接访问x属性的值。

```java 
public class Point {
    public int x, y;    // not private
    public int getX() { return x; }
        :
}
```

For more details of class loaders in Java, the following paper would be helpful:

为了了解更多关于Java的类加载器的细节，下面的论文是很有帮助的：

Sheng Liang and Gilad Bracha, "Dynamic Class Loading in the Java Virtual Machine", 
ACM OOPSLA'98, pp.36-44, 1998.

#### 3.3 Using javassist.Loader

#### 3.3 使用 javassist.Loader

Javassist provides a class loader javassist.Loader. This class loader uses a javassist.ClassPool object for reading a class file.

Javassist提供了一个javassist.Loader类加载器。这个类加载器使用javassist.ClassPool对象来加载类文件。

For example, javassist.Loader can be used for loading a particular class modified with Javassist.

例如，javassist.Loader可以用来加载由Javassist修改过的类。

```java 
import javassist.*;
import test.Rectangle;

public class Main {
  public static void main(String[] args) throws Throwable {
     ClassPool pool = ClassPool.getDefault();
     Loader cl = new Loader(pool);

     CtClass ct = pool.get("test.Rectangle");
     ct.setSuperclass(pool.get("test.Point"));

     Class c = cl.loadClass("test.Rectangle");
     Object rect = c.newInstance();
         :
  }
}
```

This program modifies a class test.Rectangle. The superclass of test.Rectangle is set to a test.Point class. Then this program loads the modified class, and creates a new instance of the test.Rectangle class.

这段程序修改了test.Rectangle类。test.Rectangle的父类设置为test.Point。然后程序加载修改后的类，创建了一个test.Rectangle类型的新实例。

If the users want to modify a class on demand when it is loaded, the users can add an event listener to a javassist.Loader. The added event listener is notified when the class loader loads a class. The event-listener class must implement the following interface:

如果用户想在类被加载的时候按照要求来修改，可以通过网javassist.Loader添加一个时间监听来实现这个功能。当一个类被类加载器加载时，添加的事件监听会被通知到。事件监听类必须实现下面的接口：

```java 
public interface Translator {
    public void start(ClassPool pool)
        throws NotFoundException, CannotCompileException;
    public void onLoad(ClassPool pool, String classname)
        throws NotFoundException, CannotCompileException;
}
```

The method start() is called when this event listener is added to a javassist.Loader object by addTranslator() in javassist.Loader. The method onLoad() is called before javassist.Loader loads a class. onLoad() can modify the definition of the loaded class.

通过javassist.Loader的addTranslator()方法来添加一个事件监听，Translator的start()方法会被回调。在javassist.Loader加载一个类之前，onLoad()方法会被调用。onLoad()方法中可以修改被加载类的定义。

For example, the following event listener changes all classes to public classes just before they are loaded.

例如，接下来的事件监听，将类被加载之前改成公有访问。

```java 
public class MyTranslator implements Translator {
    void start(ClassPool pool)
        throws NotFoundException, CannotCompileException {}
    void onLoad(ClassPool pool, String classname)
        throws NotFoundException, CannotCompileException
    {
        CtClass cc = pool.get(classname);
        cc.setModifiers(Modifier.PUBLIC);
    }
}
```

Note that onLoad() does not have to call toBytecode() or writeFile() since javassist.Loader calls these methods to obtain a class file.

注意onLoad()方法不必调用toBytecode()或者writeFile()方法，因为javasssist.Loader调用了这些方法来获取类文件。

To run an application class MyApp with a MyTranslator object, write a main class as following:

为了通过MyTranslator对象，运行MyApp类的程序，可以写一个如下的Main类：

```java 
import javassist.*;

public class Main2 {
  public static void main(String[] args) throws Throwable {
     Translator t = new MyTranslator();
     ClassPool pool = ClassPool.getDefault();
     Loader cl = new Loader();
     cl.addTranslator(pool, t);
     cl.run("MyApp", args);
  }
}
```
To run this program, do:

通过下面命令，运行这个程序：

```shell 
% java Main2 arg1 arg2...
```

The class MyApp and the other application classes are translated by MyTranslator.

MyApp类和其他应用程序类是被MyTranslator转化过的。

Note that application classes like MyApp cannot access the loader classes such as Main2, MyTranslator, and ClassPool because they are loaded by different loaders. The application classes are loaded by javassist.Loader whereas the loader classes such as Main2 are by the default Java class loader.

需要注意应用程序类如MyApp不能访问那些加载类，诸如Main2,MyTranslator,ClassPool类，因为它们是由不同加载器所加载。应用程序类是由javassist.Loader所加载，那些加载类如Main2 是由Java默认的加载器所加载。

javassist.Loader searches for classes in a different order from java.lang.ClassLoader. ClassLoader first delegates the loading operations to the parent class loader and then attempts to load the classes only if the parent class loader cannot find them. On the other hand, javassist.Loader attempts to load the classes before delegating to the parent class loader. It delegates only if:

javassist.Loader查找类的顺序与java.lang.ClassLoader不同。ClassLoader首先将加载操作委托给父类加载器，然后如果父类查找不到，才会尝试自己加载。另一边，javassist.Loader在委托之前会先尝试自己来加载这个类。仅仅下面情况才会委托父类：

the classes are not found by calling get() on a ClassPool object, or
the classes have been specified by using delegateLoadingOf() to be loaded by the parent class loader.
This search order allows loading modified classes by Javassist. However, it delegates to the parent class loader if it fails to find modified classes for some reason. Once a class is loaded by the parent class loader, the other classes referred to in that class will be also loaded by the parent class loader and thus they are never modified. Recall that all the classes referred to in a class C are loaded by the real loader of C. If your program fails to load a modified class, you should make sure whether all the classes using that class have been loaded by javassist.Loader.

通过ClassPool对象调用get()方法没有发现的类，或者通过使用delegateLoadingOf()方法描述的类将由父类加载器加载。查找顺序可以允许javassist来加载修改过的类，但是如果它因为某种原因查找修改的类失败，那么它会委托给父类加载器架子啊。一旦类由父类加载器加载，这个类中引用的其他类也将会由父类加载器来加载，因此他们的类文件是从没被修改过的。C类中所有类的引用，将由C类的真实加载器来加载。如果你的程序没有成功加载一个修改过的类，你应该去确信是否所有引用过这个类的类文件已经被javassist.Loader所加载。

####3.4 Writing a class loader

####3.4 写一个类加载器

A simple class loader using Javassist is as follows:

使用Javassist写一个简单的类加载器如下：

```java 
import javassist.*;

public class SampleLoader extends ClassLoader {
    /* Call MyApp.main().
     */
    public static void main(String[] args) throws Throwable {
        SampleLoader s = new SampleLoader();
        Class c = s.loadClass("MyApp");
        c.getDeclaredMethod("main", new Class[] { String[].class })
         .invoke(null, new Object[] { args });
    }

    private ClassPool pool;

    public SampleLoader() throws NotFoundException {
        pool = new ClassPool();
        pool.insertClassPath("./class"); // MyApp.class must be there.
    }

    /* Finds a specified class.
     * The bytecode for that class can be modified.
     */
    protected Class findClass(String name) throws ClassNotFoundException {
        try {
            CtClass cc = pool.get(name);
            // modify the CtClass object here
            byte[] b = cc.toBytecode();
            return defineClass(name, b, 0, b.length);
        } catch (NotFoundException e) {
            throw new ClassNotFoundException();
        } catch (IOException e) {
            throw new ClassNotFoundException();
        } catch (CannotCompileException e) {
            throw new ClassNotFoundException();
        }
    }
}
```

The class MyApp is an application program. To execute this program, first put the class file under the ./class directory, which must not be included in the class search path. Otherwise, MyApp.class would be loaded by the default system class loader, which is the parent loader of SampleLoader. The directory name ./class is specified by insertClassPath() in the constructor. You can choose a different name instead of ./class if you want. Then do as follows:

MyApp类是一个应用程序，为了执行这个程序，首先将类文件放在./class路径下，同时类查找路径不包含这个路径。另外，MyApp.class将会由系统默认的类加载器所加载，是SampleLoader加载器的父类加载器。路径名./class通调用用insertClassPath()方法，在构造函数中设置。你可以提供一个不同的路径名来代替./class路径，然后做如下操作：

```shell 
% java SampleLoader
```

The class loader loads the class MyApp (./class/MyApp.class) and calls MyApp.main() with the command line parameters.

类加载器加载MyApp类（./class/MyApp.class），使用命令行调用MyApp.main()方法。

This is the simplest way of using Javassist. However, if you write a more complex class loader, you may need detailed knowledge of Java's class loading mechanism. For example, the program above puts the MyApp class in a name space separated from the name space that the class SampleLoader belongs to because the two classes are loaded by different class loaders. Hence, the MyApp class cannot directly access the class SampleLoader.

这是最简单的使用Javassist方式。但是，如果你写个更复杂的类加载器，你可能需要更多关于Java类加载机制的细节知识。例如，程序将MyApp类的命名空间和类SampleLoader类的命名空间区分出来，因为这两个类是由不同类加载器所加载。因此MyApp类不能直接访问到SampleLoader类。

####3.5 Modifying a system class

####3.5 修改一个系统类

The system classes like java.lang.String cannot be loaded by a class loader other than the system class loader. Therefore, SampleLoader or javassist.Loader shown above cannot modify the system classes at loading time.

系统类如java.lang.String不能有除了系统加载器之外的其他加载器所加载。因此，SampleLoadr或者javassist.Loader不能在加载期间加载一个系统类。

If your application needs to do that, the system classes must be statically modified. For example, the following program adds a new field hiddenValue to java.lang.String:

如果你的应用须有这么做，系统类文件必须是静态被修改，例如下面的程序将会往java.lang.String中添加一个新的hiddenValue属性。

```java 
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("java.lang.String");
CtField f = new CtField(CtClass.intType, "hiddenValue", cc);
f.setModifiers(Modifier.PUBLIC);
cc.addField(f);
cc.writeFile(".");
This program produces a file "./java/lang/String.class".
```

To run your program MyApp with this modified String class, do as follows:

为了运行这个执行修改String类的程序，执行下面命令：

```java 
% java -Xbootclasspath/p:. MyApp arg1 arg2...
Suppose that the definition of MyApp is as follows:

public class MyApp {
    public static void main(String[] args) throws Exception {
        System.out.println(String.class.getField("hiddenValue").getName());
    }
}
```
If the modified String class is correctly loaded, MyApp prints hiddenValue.、

如果修改过的String类被正确加载，MyApp将会戴颖hiddenValue。

Note: Applications that use this technique for the purpose of overriding a system class in rt.jar should not be deployed as doing so would contravene the Java 2 Runtime Environment binary code license.

注意：应用程序通过使用技术来达到覆盖rt.jar中的系统类，但是不应该被部署，因为这违背了Java 2 Runtime Environment字节码许可。

####3.6 Reloading a class at runtime

####3.6 在运行期间重载类

If the JVM is launched with the JPDA (Java Platform Debugger Architecture) enabled, a class is dynamically reloadable. After the JVM loads a class, the old version of the class definition can be unloaded and a new one can be reloaded again. That is, the definition of that class can be dynamically modified during runtime. However, the new class definition must be somewhat compatible to the old one. The JVM does not allow schema changes between the two versions. They have the same set of methods and fields.

如果JVM在启动时JPDA(Java 平台debugger 体系)属性开启，类是可以动态重复加载。JVM加载一个类后，类的老版本定义被卸载，新的类被再一次重载。换言之，类的定义可以在运行期间动态的被修改。但是新类的定义某种程度上要兼容旧的类定义。JVM不允许两个版本之间schema发生改变。两个版本要有相同的方法和属性。

Javassist provides a convenient class for reloading a class at runtime. For more information, see the API documentation of javassist.tools.HotSwapper.

Javassist提供了一个便捷的类，用来在运行期间重载一个列，了解javassist.tools.HotSwapper的API文档。

### 4. Introspection and customization

### 4. 内省和定制化

CtClass provides methods for introspection. The introspective ability of Javassist is compatible with that of the Java reflection API. CtClass provides getName(), getSuperclass(), getMethods(), and so on. CtClass also provides methods for modifying a class definition. It allows to add a new field, constructor, and method. Instrumenting a method body is also possible.

CtClass 提供内省的方法。Javassist的内省能力是兼容Java反射API。CtClass提供getName()，getSuperClass()，getMethod()诸如此类的方法。CtClass同样也提供修改类的定义方法，允许添加往类中添加属性，构造函数，方法等。操作方法体逻辑也成为了可能。

Methods are represented by CtMethod objects. CtMethod provides several methods for modifying the definition of the method. Note that if a method is inherited from a super class, then the same CtMethod object that represents the inherited method represents the method declared in that super class. A CtMethod object corresponds to every method declaration.

CtMethod对象代表类中的方法。CtMethod提供了几种方式来修改方法的定义。注意一个方法如果是来自继承的父类，同一个CtMethod对象可以代表来自父类继承的方法。一个CtMethod可以响应每个声明的方法。

For example, if class Point declares method move() and a subclass ColorPoint of Point does not override move(), the two move() methods declared in Point and inherited in ColorPoint are represented by the identical CtMethod object. If the method definition represented by this CtMethod object is modified, the modification is reflected on both the methods. If you want to modify only the move() method in ColorPoint, you first have to add to ColorPoint a copy of the CtMethod object representing move() in Point. A copy of the the CtMethod object can be obtained by CtNewMethod.copy().

例如，如果Point类声明了move()方法，子类ColorPoint没有重载这个方法。在Point中声明的move()方法和在ColorPoint中继承的move()方法是由同一个CtMethod对象表示。如果由CtMethod对代表的方法被修改，两个类中的方法都会被影响。如果你仅仅想修改ColorPoint中的move()方法，首先你需要先添加一个代表move()方法的CtMethod对象的拷贝到ColorPoint中，可以通过CtNewMethod.copy()获取这个拷贝。

Javassist does not allow to remove a method or field, but it allows to change the name. So if a method is not necessary any more, it should be renamed and changed to be a private method by calling setName() and setModifiers() declared in CtMethod.

Javassist不允许删除一个方法或者属性，但是它允许改变名称。所以如果一个方法不再需要，可以通过CtMethod的setName()方法重命名并通过setModifiers()方法设置为私有。

Javassist does not allow to add an extra parameter to an existing method, either. Instead of doing that, a new method receiving the extra parameter as well as the other parameters should be added to the same class. For example, if you want to add an extra int parameter newZ to a method:

Javassist不允许往一个存在的方法中添加扩展参数，或者

```java 
void move(int newX, int newY) { x = newX; y = newY; }
```

in a Point class, then you should add the following method to the Point class:

在Point类中，你应该添加下面的方法到Point类中：

```java 
void move(int newX, int newY, int newZ) {
    // do what you want with newZ.
    move(newX, newY);
}
```

Javassist also provides low-level API for directly editing a raw class file. For example, getClassFile() in CtClass returns a ClassFile object representing a raw class file. getMethodInfo() in CtMethod returns a MethodInfo object representing a method_info structure included in a class file. The low-level API uses the vocabulary from the Java Virtual machine specification. The users must have the knowledge about class files and bytecode. For more details, the users should see the javassist.bytecode package.

Javassist同样也提供底层API用来直接编辑原生类文件。例如，CtClass中的getClassFile()返回代表原生类的ClassFile对象。CtMethod中的getMethodInfo()方法返回一个MethodInfo对象 代表着类文件中方法信息结构。底层API使用的是Java虚拟机规范的字典。使用者必须对类文件规范以及字节码有了解，更多细节，可以查看javassist.bytecode包。

The class files modified by Javassist requires the javassist.runtime package for runtime support only if some special identifiers starting with $ are used. Those special identifiers are described below. The class files modified without those special identifiers do not need the javassist.runtime package or any other Javassist packages at runtime. For more details, see the API documentation of the javassist.runtime package.



4.1 Inserting source text at the beginning/end of a method body

CtMethod and CtConstructor provide methods insertBefore(), insertAfter(), and addCatch(). They are used for inserting a code fragment into the body of an existing method. The users can specify those code fragments with source text written in Java. Javassist includes a simple Java compiler for processing source text. It receives source text written in Java and compiles it into Java bytecode, which will be inlined into a method body.

Inserting a code fragment at the position specified by a line number is also possible (if the line number table is contained in the class file). insertAt() in CtMethod and CtConstructor takes source text and a line number in the source file of the original class definition. It compiles the source text and inserts the compiled code at the line number.

The methods insertBefore(), insertAfter(), addCatch(), and insertAt() receive a String object representing a statement or a block. A statement is a single control structure like if and while or an expression ending with a semi colon (;). A block is a set of statements surrounded with braces {}. Hence each of the following lines is an example of valid statement or block:

System.out.println("Hello");
{ System.out.println("Hello"); }
if (i < 0) { i = -i; }
The statement and the block can refer to fields and methods. They can also refer to the parameters to the method that they are inserted into if that method was compiled with the -g option (to include a local variable attribute in the class file). Otherwise, they must access the method parameters through the special variables $0, $1, $2, ... described below. Accessing local variables declared in the method is not allowed although declaring a new local variable in the block is allowed. However, insertAt() allows the statement and the block to access local variables if these variables are available at the specified line number and the target method was compiled with the -g option.
The String object passed to the methods insertBefore(), insertAfter(), addCatch(), and insertAt() are compiled by the compiler included in Javassist. Since the compiler supports language extensions, several identifiers starting with $ have special meaning:

$0, $1, $2, ...    	this and actual parameters
$args	An array of parameters. The type of $args is Object[].
$$	All actual parameters.
For example, m($$) is equivalent to m($1,$2,...)
 
$cflow(...)	cflow variable
$r	The result type. It is used in a cast expression.
$w	The wrapper type. It is used in a cast expression.
$_	The resulting value
$sig	An array of java.lang.Class objects representing the formal parameter types.
$type	A java.lang.Class object representing the formal result type.
$class	A java.lang.Class object representing the class currently edited.
$0, $1, $2, ...

The parameters passed to the target method are accessible with $1, $2, ... instead of the original parameter names. $1 represents the first parameter, $2 represents the second parameter, and so on. The types of those variables are identical to the parameter types. $0 is equivalent to this. If the method is static, $0 is not available.

These variables are used as following. Suppose that a class Point:

class Point {
    int x, y;
    void move(int dx, int dy) { x += dx; y += dy; }
}
To print the values of dx and dy whenever the method move() is called, execute this program:

ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
CtMethod m = cc.getDeclaredMethod("move");
m.insertBefore("{ System.out.println($1); System.out.println($2); }");
cc.writeFile();
Note that the source text passed to insertBefore() is surrounded with braces {}. insertBefore() accepts only a single statement or a block surrounded with braces.

The definition of the class Point after the modification is like this:

class Point {
    int x, y;
    void move(int dx, int dy) {
        { System.out.println(dx); System.out.println(dy); }
        x += dx; y += dy;
    }
}
$1 and $2 are replaced with dx and dy, respectively.

$1, $2, $3 ... are updatable. If a new value is assigend to one of those variables, then the value of the parameter represented by that variable is also updated.

$args

The variable $args represents an array of all the parameters. The type of that variable is an array of class Object. If a parameter type is a primitive type such as int, then the parameter value is converted into a wrapper object such as java.lang.Integer to store in $args. Thus, $args[0] is equivalent to $1 unless the type of the first parameter is a primitive type. Note that $args[0] is not equivalent to $0; $0 represents this.

If an array of Object is assigned to $args, then each element of that array is assigned to each parameter. If a parameter type is a primitive type, the type of the corresponding element must be a wrapper type. The value is converted from the wrapper type to the primitive type before it is assigned to the parameter.

$$

The variable $$ is abbreviation of a list of all the parameters separated by commas. For example, if the number of the parameters to method move() is three, then

move($$)
is equivalent to this:

move($1, $2, $3)
If move() does not take any parameters, then move($$) is equivalent to move().

$$ can be used with another method. If you write an expression:

exMove($$, context)
then this expression is equivalent to:

exMove($1, $2, $3, context)
Note that $$ enables generic notation of method call with respect to the number of parameters. It is typically used with $proceed shown later.

$cflow

$cflow means "control flow". This read-only variable returns the depth of the recursive calls to a specific method.

Suppose that the method shown below is represented by a CtMethod object cm:

int fact(int n) {
    if (n <= 1)
        return n;
    else
        return n * fact(n - 1);
}
To use $cflow, first declare that $cflow is used for monitoring calls to the method fact():

CtMethod cm = ...;
cm.useCflow("fact");
The parameter to useCflow() is the identifier of the declared $cflow variable. Any valid Java name can be used as the identifier. Since the identifier can also include . (dot), for example, "my.Test.fact" is a valid identifier.

Then, $cflow(fact) represents the depth of the recursive calls to the method specified by cm. The value of $cflow(fact) is 0 (zero) when the method is first called whereas it is 1 when the method is recursively called within the method. For example,

cm.insertBefore("if ($cflow(fact) == 0)"
              + "    System.out.println(\"fact \" + $1);");
translates the method fact() so that it shows the parameter. Since the value of $cflow(fact) is checked, the method fact() does not show the parameter if it is recursively called within fact().

The value of $cflow is the number of stack frames associated with the specified method cm under the current topmost stack frame for the current thread. $cflow is also accessible within a method different from the specified method cm.

$r

$r represents the result type (return type) of the method. It must be used as the cast type in a cast expression. For example, this is a typical use:

Object result = ... ;
$_ = ($r)result;
If the result type is a primitive type, then ($r) follows special semantics. First, if the operand type of the cast expression is a primitive type, ($r) works as a normal cast operator to the result type. On the other hand, if the operand type is a wrapper type, ($r) converts from the wrapper type to the result type. For example, if the result type is int, then ($r) converts from java.lang.Integer to int.

If the result type is void, then ($r) does not convert a type; it does nothing. However, if the operand is a call to a void method, then ($r) results in null. For example, if the result type is void and foo() is a void method, then

$_ = ($r)foo();
is a valid statement.

The cast operator ($r) is also useful in a return statement. Even if the result type is void, the following return statement is valid:

return ($r)result;
Here, result is some local variable. Since ($r) is specified, the resulting value is discarded. This return statement is regarded as the equivalent of the return statement without a resulting value:

return;
$w

$w represents a wrapper type. It must be used as the cast type in a cast expression. ($w) converts from a primitive type to the corresponding wrapper type. The following code is an example:

Integer i = ($w)5;
The selected wrapper type depends on the type of the expression following ($w). If the type of the expression is double, then the wrapper type is java.lang.Double.

If the type of the expression following ($w) is not a primitive type, then ($w) does nothing.

$_

insertAfter() in CtMethod and CtConstructor inserts the compiled code at the end of the method. In the statement given to insertAfter(), not only the variables shown above such as $0, $1, ... but also $_ is available.

The variable $_ represents the resulting value of the method. The type of that variable is the type of the result type (the return type) of the method. If the result type is void, then the type of $_ is Object and the value of $_ is null.

Although the compiled code inserted by insertAfter() is executed just before the control normally returns from the method, it can be also executed when an exception is thrown from the method. To execute it when an exception is thrown, the second parameter asFinally to insertAfter() must be true.

If an exception is thrown, the compiled code inserted by insertAfter() is executed as a finally clause. The value of $_ is 0 or null in the compiled code. After the execution of the compiled code terminates, the exception originally thrown is re-thrown to the caller. Note that the value of $_ is never thrown to the caller; it is rather discarded.

$sig

The value of $sig is an array of java.lang.Class objects that represent the formal parameter types in declaration order.

$type

The value of $type is an java.lang.Class object representing the formal type of the result value. This variable refers to Void.class if this is a constructor.

$class

The value of $class is an java.lang.Class object representing the class in which the edited method is declared. This represents the type of $0.

addCatch()

addCatch() inserts a code fragment into a method body so that the code fragment is executed when the method body throws an exception and the control returns to the caller. In the source text representing the inserted code fragment, the exception value is referred to with the special variable $e.

For example, this program:

CtMethod m = ...;
CtClass etype = ClassPool.getDefault().get("java.io.IOException");
m.addCatch("{ System.out.println($e); throw $e; }", etype);
translates the method body represented by m into something like this:

try {
    the original method body
}
catch (java.io.IOException e) {
    System.out.println(e);
    throw e;
}
Note that the inserted code fragment must end with a throw or return statement.



4.2 Altering a method body

CtMethod and CtConstructor provide setBody() for substituting a whole method body. They compile the given source text into Java bytecode and substitutes it for the original method body. If the given source text is null, the substituted body includes only a return statement, which returns zero or null unless the result type is void.

In the source text given to setBody(), the identifiers starting with $ have special meaning

$0, $1, $2, ...    	this and actual parameters
$args	An array of parameters. The type of $args is Object[].
$$	All actual parameters.
$cflow(...)	cflow variable
$r	The result type. It is used in a cast expression.
$w	The wrapper type. It is used in a cast expression.
$sig	An array of java.lang.Class objects representing the formal parameter types.
$type	A java.lang.Class object representing the formal result type.
$class	A java.lang.Class object representing the class that declares the method
currently edited (the type of $0).
 
Note that $_ is not available.
Substituting source text for an existing expression

Javassist allows modifying only an expression included in a method body. javassist.expr.ExprEditor is a class for replacing an expression in a method body. The users can define a subclass of ExprEditor to specify how an expression is modified.

To run an ExprEditor object, the users must call instrument() in CtMethod or CtClass. For example,

CtMethod cm = ... ;
cm.instrument(
    new ExprEditor() {
        public void edit(MethodCall m)
                      throws CannotCompileException
        {
            if (m.getClassName().equals("Point")
                          && m.getMethodName().equals("move"))
                m.replace("{ $1 = 0; $_ = $proceed($$); }");
        }
    });
searches the method body represented by cm and replaces all calls to move() in class Point with a block:

{ $1 = 0; $_ = $proceed($$); }
so that the first parameter to move() is always 0. Note that the substituted code is not an expression but a statement or a block. It cannot be or contain a try-catch statement.

The method instrument() searches a method body. If it finds an expression such as a method call, field access, and object creation, then it calls edit() on the given ExprEditor object. The parameter to edit() is an object representing the found expression. The edit() method can inspect and replace the expression through that object.

Calling replace() on the parameter to edit() substitutes the given statement or block for the expression. If the given block is an empty block, that is, if replace("{}") is executed, then the expression is removed from the method body. If you want to insert a statement (or a block) before/after the expression, a block like the following should be passed to replace():

{ before-statements;
  $_ = $proceed($$);
  after-statements; }
whichever the expression is either a method call, field access, object creation, or others. The second statement could be:

$_ = $proceed();
if the expression is read access, or

$proceed($$);
if the expression is write access.

Local variables available in the target expression is also available in the source text passed to replace() if the method searched by instrument() was compiled with the -g option (the class file includes a local variable attribute).

javassist.expr.MethodCall

A MethodCall object represents a method call. The method replace() in MethodCall substitutes a statement or a block for the method call. It receives source text representing the substitued statement or block, in which the identifiers starting with $ have special meaning as in the source text passed to insertBefore().

$0	The target object of the method call.
This is not equivalent to this, which represents the caller-side this object.
$0 is null if the method is static.
 
 
$1, $2, ...    	The parameters of the method call.
$_	The resulting value of the method call.
$r	The result type of the method call.
$class    	A java.lang.Class object representing the class declaring the method.
$sig    	An array of java.lang.Class objects representing the formal parameter types.
$type    	A java.lang.Class object representing the formal result type.
$proceed    	The name of the method originally called in the expression.
Here the method call means the one represented by the MethodCall object.

The other identifiers such as $w, $args and $$ are also available.

Unless the result type of the method call is void, a value must be assigned to $_ in the source text and the type of $_ is the result type. If the result type is void, the type of $_ is Object and the value assigned to $_ is ignored.

$proceed is not a String value but special syntax. It must be followed by an argument list surrounded by parentheses ( ).

javassist.expr.ConstructorCall

A ConstructorCall object represents a constructor call such as this() and super included in a constructor body. The method replace() in ConstructorCall substitutes a statement or a block for the constructor call. It receives source text representing the substituted statement or block, in which the identifiers starting with $ have special meaning as in the source text passed to insertBefore().

$0	The target object of the constructor call. This is equivalent to this.
$1, $2, ...    	The parameters of the constructor call.
$class    	A java.lang.Class object representing the class declaring the constructor.
$sig    	An array of java.lang.Class objects representing the formal parameter types.
$proceed    	The name of the constructor originally called in the expression.
Here the constructor call means the one represented by the ConstructorCall object.

The other identifiers such as $w, $args and $$ are also available.

Since any constructor must call either a constructor of the super class or another constructor of the same class, the substituted statement must include a constructor call, normally a call to $proceed().

$proceed is not a String value but special syntax. It must be followed by an argument list surrounded by parentheses ( ).

javassist.expr.FieldAccess

A FieldAccess object represents field access. The method edit() in ExprEditor receives this object if field access is found. The method replace() in FieldAccess receives source text representing the substitued statement or block for the field access.

In the source text, the identifiers starting with $ have special meaning:

$0	The object containing the field accessed by the expression. This is not equivalent to this.
this represents the object that the method including the expression is invoked on.
$0 is null if the field is static.
 
 
$1	The value that would be stored in the field if the expression is write access. 
Otherwise, $1 is not available.
 
$_	The resulting value of the field access if the expression is read access. 
Otherwise, the value stored in $_ is discarded.
 
$r	The type of the field if the expression is read access. 
Otherwise, $r is void.
 
$class    	A java.lang.Class object representing the class declaring the field.
$type	A java.lang.Class object representing the field type.
$proceed    	The name of a virtual method executing the original field access. .
The other identifiers such as $w, $args and $$ are also available.

If the expression is read access, a value must be assigned to $_ in the source text. The type of $_ is the type of the field.

javassist.expr.NewExpr

A NewExpr object represents object creation with the new operator (not including array creation). The method edit() in ExprEditor receives this object if object creation is found. The method replace() in NewExpr receives source text representing the substitued statement or block for the object creation.

In the source text, the identifiers starting with $ have special meaning:

$0	null.
$1, $2, ...    	The parameters to the constructor.
$_	The resulting value of the object creation. 
A newly created object must be stored in this variable.
 
$r	The type of the created object.
$sig    	An array of java.lang.Class objects representing the formal parameter types.
$type    	A java.lang.Class object representing the class of the created object.
$proceed    	The name of a virtual method executing the original object creation. .
The other identifiers such as $w, $args and $$ are also available.

javassist.expr.NewArray

A NewArray object represents array creation with the new operator. The method edit() in ExprEditor receives this object if array creation is found. The method replace() in NewArray receives source text representing the substitued statement or block for the array creation.

In the source text, the identifiers starting with $ have special meaning:

$0	null.
$1, $2, ...    	The size of each dimension.
$_	The resulting value of the array creation. 
A newly created array must be stored in this variable.
 
$r	The type of the created array.
$type    	A java.lang.Class object representing the class of the created array.
$proceed    	The name of a virtual method executing the original array creation. .
The other identifiers such as $w, $args and $$ are also available.

For example, if the array creation is the following expression,

String[][] s = new String[3][4];
then the value of $1 and $2 are 3 and 4, respectively. $3 is not available.
If the array creation is the following expression,

String[][] s = new String[3][];
then the value of $1 is 3 but $2 is not available.
javassist.expr.Instanceof

A Instanceof object represents an instanceof expression. The method edit() in ExprEditor receives this object if an instanceof expression is found. The method replace() in Instanceof receives source text representing the substitued statement or block for the expression.

In the source text, the identifiers starting with $ have special meaning:

$0	null.
$1	The value on the left hand side of the original instanceof operator.
$_	The resulting value of the expression. The type of $_ is boolean.
$r	The type on the right hand side of the instanceof operator.
$type	A java.lang.Class object representing the type on the right hand side of the instanceof operator.
$proceed    	The name of a virtual method executing the original instanceof expression. 
It takes one parameter (the type is java.lang.Object) and returns true 
if the parameter value is an instance of the type on the right hand side of 
the original instanceof operator. Otherwise, it returns false.
 
 
 
The other identifiers such as $w, $args and $$ are also available.

javassist.expr.Cast

A Cast object represents an expression for explicit type casting. The method edit() in ExprEditor receives this object if explicit type casting is found. The method replace() in Cast receives source text representing the substitued statement or block for the expression.

In the source text, the identifiers starting with $ have special meaning:

$0	null.
$1	The value the type of which is explicitly cast.
$_	The resulting value of the expression. The type of $_ is the same as the type 
after the explicit casting, that is, the type surrounded by ( ).
 
$r	the type after the explicit casting, or the type surrounded by ( ).
$type	A java.lang.Class object representing the same type as $r.
$proceed    	The name of a virtual method executing the original type casting. 
It takes one parameter of the type java.lang.Object and returns it after 
the explicit type casting specified by the original expression.
 
 
The other identifiers such as $w, $args and $$ are also available.

javassist.expr.Handler

A Handler object represents a catch clause of try-catch statement. The method edit() in ExprEditor receives this object if a catch is found. The method insertBefore() in Handler compiles the received source text and inserts it at the beginning of the catch clause.

In the source text, the identifiers starting with $ have meaning:

$1	The exception object caught by the catch clause.
$r	the type of the exception caught by the catch clause. It is used in a cast expression.
$w	The wrapper type. It is used in a cast expression.
$type    	A java.lang.Class object representing 
the type of the exception caught by the catch clause.
 
If a new exception object is assigned to $1, it is passed to the original catch clause as the caught exception.



4.3 Adding a new method or field

Adding a method

Javassist allows the users to create a new method and constructor from scratch. CtNewMethod and CtNewConstructor provide several factory methods, which are static methods for creating CtMethod or CtConstructor objects. Especially, make() creates a CtMethod or CtConstructor object from the given source text.

For example, this program:

CtClass point = ClassPool.getDefault().get("Point");
CtMethod m = CtNewMethod.make(
                 "public int xmove(int dx) { x += dx; }",
                 point);
point.addMethod(m);
adds a public method xmove() to class Point. In this example, x is a int field in the class Point.

The source text passed to make() can include the identifiers starting with $ except $_ as in setBody(). It can also include $proceed if the target object and the target method name are also given to make(). For example,

CtClass point = ClassPool.getDefault().get("Point");
CtMethod m = CtNewMethod.make(
                 "public int ymove(int dy) { $proceed(0, dy); }",
                 point, "this", "move");
this program creates a method ymove() defined below:

public int ymove(int dy) { this.move(0, dy); }
Note that $proceed has been replaced with this.move.

Javassist provides another way to add a new method. You can first create an abstract method and later give it a method body:

CtClass cc = ... ;
CtMethod m = new CtMethod(CtClass.intType, "move",
                          new CtClass[] { CtClass.intType }, cc);
cc.addMethod(m);
m.setBody("{ x += $1; }");
cc.setModifiers(cc.getModifiers() & ~Modifier.ABSTRACT);
Since Javassist makes a class abstract if an abstract method is added to the class, you have to explicitly change the class back to a non-abstract one after calling setBody().

Mutual recursive methods

Javassist cannot compile a method if it calls another method that has not been added to a class. (Javassist can compile a method that calls itself recursively.) To add mutual recursive methods to a class, you need a trick shown below. Suppose that you want to add methods m() and n() to a class represented by cc:

CtClass cc = ... ;
CtMethod m = CtNewMethod.make("public abstract int m(int i);", cc);
CtMethod n = CtNewMethod.make("public abstract int n(int i);", cc);
cc.addMethod(m);
cc.addMethod(n);
m.setBody("{ return ($1 <= 0) ? 1 : (n($1 - 1) * $1); }");
n.setBody("{ return m($1); }");
cc.setModifiers(cc.getModifiers() & ~Modifier.ABSTRACT);
You must first make two abstract methods and add them to the class. Then you can give the method bodies to these methods even if the method bodies include method calls to each other. Finally you must change the class to a not-abstract class since addMethod() automatically changes a class into an abstract one if an abstract method is added.

Adding a field

Javassist also allows the users to create a new field.

CtClass point = ClassPool.getDefault().get("Point");
CtField f = new CtField(CtClass.intType, "z", point);
point.addField(f);
This program adds a field named z to class Point.

If the initial value of the added field must be specified, the program shown above must be modified into:

CtClass point = ClassPool.getDefault().get("Point");
CtField f = new CtField(CtClass.intType, "z", point);
point.addField(f, "0");    // initial value is 0.
Now, the method addField() receives the second parameter, which is the source text representing an expression computing the initial value. This source text can be any Java expression if the result type of the expression matches the type of the field. Note that an expression does not end with a semi colon (;).

Furthermore, the above code can be rewritten into the following simple code:

CtClass point = ClassPool.getDefault().get("Point");
CtField f = CtField.make("public int z = 0;", point);
point.addField(f);
Removing a member

To remove a field or a method, call removeField() or removeMethod() in CtClass. A CtConstructor can be removed by removeConstructor() in CtClass.



4.4 Annotations

CtClass, CtMethod, CtField and CtConstructor provides a convenient method getAnnotations() for reading annotations. It returns an annotation-type object.

For example, suppose the following annotation:

public @interface Author {
    String name();
    int year();
}
This annotation is used as the following:

@Author(name="Chiba", year=2005)
public class Point {
    int x, y;
}
Then, the value of the annotation can be obtained by getAnnotations(). It returns an array containing annotation-type objects.

CtClass cc = ClassPool.getDefault().get("Point");
Object[] all = cc.getAnnotations();
Author a = (Author)all[0];
String name = a.name();
int year = a.year();
System.out.println("name: " + name + ", year: " + year);
This code snippet should print:

name: Chiba, year: 2005
Since the annoation of Point is only @Author, the length of the array all is one and all[0] is an Author object. The member values of the annotation can be obtained by calling name() and year() on the Author object.

To use getAnnotations(), annotation types such as Author must be included in the current class path. They must be also accessible from a ClassPool object. If the class file of an annotation type is not found, Javassist cannot obtain the default values of the members of that annotation type.



4.5 Runtime support classes

In most cases, a class modified by Javassist does not require Javassist to run. However, some kinds of bytecode generated by the Javassist compiler need runtime support classes, which are in the javassist.runtime package (for details, please read the API reference of that package). Note that the javassist.runtime package is the only package that classes modified by Javassist may need for running. The other Javassist classes are never used at runtime of the modified classes.



4.6 Import

All the class names in source code must be fully qualified (they must include package names). However, the java.lang package is an exception; for example, the Javassist compiler can resolve Object as well as java.lang.Object.

To tell the compiler to search other packages when resolving a class name, call importPackage() in ClassPool. For example,

ClassPool pool = ClassPool.getDefault();
pool.importPackage("java.awt");
CtClass cc = pool.makeClass("Test");
CtField f = CtField.make("public Point p;", cc);
cc.addField(f);
The seconde line instructs the compiler to import the java.awt package. Thus, the third line will not throw an exception. The compiler can recognize Point as java.awt.Point.

Note that importPackage() does not affect the get() method in ClassPool. Only the compiler considers the imported packages. The parameter to get() must be always a fully qualified name.



4.7 Limitations

In the current implementation, the Java compiler included in Javassist has several limitations with respect to the language that the compiler can accept. Those limitations are:

The new syntax introduced by J2SE 5.0 (including enums and generics) has not been supported. Annotations are supported by the low level API of Javassist. See the javassist.bytecode.annotation package (and also getAnnotations() in CtClass and CtBehavior). Generics are also only partly supported. See the latter section for more details.
Array initializers, a comma-separated list of expressions enclosed by braces { and }, are not available unless the array dimension is one.
Inner classes or anonymous classes are not supported. Note that this is a limitation of the compiler only. It cannot compile source code including an anonymous-class declaration. Javassist can read and modify a class file of inner/anonymous class.
Labeled continue and break statements are not supported.
The compiler does not correctly implement the Java method dispatch algorithm. The compiler may confuse if methods defined in a class have the same name but take different parameter lists.
For example,

class A {} 
class B extends A {} 
class C extends B {} 

class X { 
    void foo(A a) { .. } 
    void foo(B b) { .. } 
}
If the compiled expression is x.foo(new C()), where x is an instance of X, the compiler may produce a call to foo(A) although the compiler can correctly compile foo((B)new C()).

The users are recommended to use # as the separator between a class name and a static method or field name. For example, in regular Java,
javassist.CtClass.intType.getName()
calls a method getName() on the object indicated by the static field intType in javassist.CtClass. In Javassist, the users can write the expression shown above but they are recommended to write:

javassist.CtClass#intType.getName()
so that the compiler can quickly parse the expression.
