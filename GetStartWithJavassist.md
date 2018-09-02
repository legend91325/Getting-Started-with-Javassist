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

To disallow pruning a particular CtClass, stopPruning() must be called on that object in advance:

CtClasss cc = ...;
cc.stopPruning(true);
    :
cc.writeFile();                             // convert to a class file.
// cc is not pruned.
The CtClass object cc is not pruned. Thus it can be defrost after writeFile() is called.

Note: While debugging, you might want to temporarily stop pruning and freezing and write a modified class file to a disk drive. debugWriteFile() is a convenient method for that purpose. It stops pruning, writes a class file, defrosts it, and turns pruning on again (if it was initially on).
Class search path

The default ClassPool returned by a static method ClassPool.getDefault() searches the same path that the underlying JVM (Java virtual machine) has. If a program is running on a web application server such as JBoss and Tomcat, the ClassPool object may not be able to find user classes since such a web application server uses multiple class loaders as well as the system class loader. In that case, an additional class path must be registered to the ClassPool. Suppose that pool refers to a ClassPool object:

pool.insertClassPath(new ClassClassPath(this.getClass()));
This statement registers the class path that was used for loading the class of the object that this refers to. You can use any Class object as an argument instead of this.getClass(). The class path used for loading the class represented by that Class object is registered.

You can register a directory name as the class search path. For example, the following code adds a directory /usr/local/javalib to the search path:

ClassPool pool = ClassPool.getDefault();
pool.insertClassPath("/usr/local/javalib");
The search path that the users can add is not only a directory but also a URL:

ClassPool pool = ClassPool.getDefault();
ClassPath cp = new URLClassPath("www.javassist.org", 80, "/java/", "org.javassist.");
pool.insertClassPath(cp);
This program adds "http://www.javassist.org:80/java/" to the class search path. This URL is used only for searching classes belonging to a package org.javassist. For example, to load a class org.javassist.test.Main, its class file will be obtained from:

http://www.javassist.org:80/java/org/javassist/test/Main.class
Furthermore, you can directly give a byte array to a ClassPool object and construct a CtClass object from that array. To do this, use ByteArrayClassPath. For example,

ClassPool cp = ClassPool.getDefault();
byte[] b = a byte array;
String name = class name;
cp.insertClassPath(new ByteArrayClassPath(name, b));
CtClass cc = cp.get(name);
The obtained CtClass object represents a class defined by the class file specified by b. The ClassPool reads a class file from the given ByteArrayClassPath if get() is called and the class name given to get() is equal to one specified by name.

If you do not know the fully-qualified name of the class, then you can use makeClass() in ClassPool:

ClassPool cp = ClassPool.getDefault();
InputStream ins = an input stream for reading a class file;
CtClass cc = cp.makeClass(ins);
makeClass() returns the CtClass object constructed from the given input stream. You can use makeClass() for eagerly feeding class files to the ClassPool object. This might improve performance if the search path includes a large jar file. Since a ClassPool object reads a class file on demand, it might repeatedly search the whole jar file for every class file. makeClass() can be used for optimizing this search. The CtClass constructed by makeClass() is kept in the ClassPool object and the class file is never read again.

The users can extend the class search path. They can define a new class implementing ClassPath interface and give an instance of that class to insertClassPath() in ClassPool. This allows a non-standard resource to be included in the search path.
