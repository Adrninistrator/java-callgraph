[![Maven Central](https://img.shields.io/maven-central/v/com.github.adrninistrator/java-callgraph.svg)](https://search.maven.org/artifact/com.github.adrninistrator/java-callgraph/)

该项目已移至[https://github.com/Adrninistrator/java-callgraph2](https://github.com/Adrninistrator/java-callgraph2)

# 使用说明

编译命令：

```
gradlew jar
```

执行参数：

VM options: 用于指定输出文件路径

```
-Doutput.file=a.txt
```

Program arguments:用于指定需要解析的jar包路径列表

```
build/libs/a.jar build/libs/b.jar
```

# 增加调用关系类型及调用者代码行号

增强后的java-callgraph输出的方法调用关系的格式与原始java-callgraph基本一致，如下所示：

```
M:class1:<method1>(arg_types) (typeofcall)class2:<method2>(arg_types) line_number jar_number
```

原始java-callgraph支持的调用类型arg_types如下：

 * `M` for `invokevirtual` calls
 * `I` for `invokeinterface` calls
 * `O` for `invokespecial` calls
 * `S` for `invokestatic` calls
 * `D` for `invokedynamic` calls

增强后的java-callgraph增加的调用类型arg_types如下：

|typeofcall|含义|
|---|---|
|ITF|接口与实现类方法|
|RIR|Runnable实现类线程调用|
|CIC|Callable实现类线程调用|
|TSR|Thread子类线程调用|
|LM|lambda表达式（含线程调用等）|
|SCC|父类调用子类的实现方法|
|CCS|子类调用父类的实现方法|

- line_number

为当前调用者方法源代码对应行号

- jar_number

jar包序号，从1开始

# 增加jar包文件路径

增强后的java-callgraph会输出当前jar包文件路径，如下所示：

```
J:jar_number jar_file_path
```

- jar_number

jar包序号，从1开始

- jar_file_path

jar包文件路径

# 原始java-callgraph调用关系缺失的场景

原始java-callgraph在多数场景下能够获取到Java方法调用关系，但以下场景的调用关系会缺失：

- 接口与实现类方法

假如存在接口Interface1，及其实现类Impl1，若在某个类Class1中引入了接口Interface1，实际为实现类Impl1的实例（使用Spring时的常见场景），在其方法Class1.func1()中调用了Interface1.fi()方法；

原始java-callgraph生成的方法调用关系中，只包含Class1.func1()调用Interface1.fi()的关系，Class1.func1()调用Impl1.fi()，及Impl1.fi()向下调用的关系会缺失。

- Runnable实现类线程调用

假如f1()方法中使用内部匿名类形式的Runnable实现类在线程中执行操作，在线程中执行了f2()方法，如下所示：

```java
private void f1() {
    new Thread(new Runnable() {
        @Override
        public void run() {
            f2();
        }
    }).start();
}
```

原始java-callgraph生成的方法调用关系中，f1()调用f2()，及f2()向下调用的关系会缺失；

对于使用命名类形式的Runnable实现类在线程中执行操作的情况，存在相同的问题，原方法调用线程中执行的方法，及继续向下的调用关系会缺失。

- Callable实现类线程调用

与Runnable实现类线程调用情况类似，略。

- Thread子类线程调用

与Runnable实现类线程调用情况类似，略。

- lambda表达式（含线程调用等）

假如f1()方法中使用lambda表达式的形式在线程中执行操作，在线程中执行了f2()方法，如下所示：

```java
private void f1() {
    new Thread(() -> f2()).start();
}
```

原始java-callgraph生成的方法调用关系中，f1()调用f2()，及f2()向下调用的关系会缺失；

对于其他使用lambda表达式的情况，存在相同的问题，原方法调用lambda表达式中执行的方法，及继续向下的调用关系会缺失。

- 父类调用子类的实现方法

假如存在抽象父类Abstract1，及其非抽象子类ChildImpl1，若在某个类Class1中引入了抽象父类Abstract1，实际为子类ChildImpl1的实例（使用Spring时的常见场景），在其方法Class1.func1()中调用了Abstract1.fa()方法；

原始java-callgraph生成的方法调用关系中，只包含Class1.func1()调用Abstract1.fa()的关系，Class1.func1()调用ChildImpl1.fa()的关系会缺失。

- 子类调用父类的实现方法

假如存在抽象父类Abstract1，及其非抽象子类ChildImpl1，若在ChildImpl1.fc1()方法中调用了父类Abstract1实现的方法fi()；

原始java-callgraph生成的方法调用关系中，ChildImpl1.fc1()调用Abstract1.fi()的关系会缺失。

针对以上问题，增强后的java-callgraph都进行了优化，能够生成缺失的调用关系。

增强后的java-callgraph地址为#github#

对于更复杂的情况，例如存在接口Interface1，及其抽象实现类Abstract1，及其子类ChildImpl1，若在某个类中引入了抽象实现类Abstract1并调用其方法的情况，生成的方法调用关系中也不会出现缺失。

# 原始文档

java-callgraph: Java Call Graph Utilities
=========================================

A suite of programs for generating static and dynamic call graphs in Java.

* javacg-static: Reads classes from a jar file, walks down the method bodies and
   prints a table of caller-caller relationships.
* javacg-dynamic: Runs as a [Java agent](http://download.oracle.com/javase/6/docs/api/index.html?java/lang/instrument/package-summary.html) and instruments
  the methods of a user-defined set of classes in order to track their invocations.
  At JVM exit, prints a table of caller-callee relationships, along with a number
  of calls

#### Compile

The java-callgraph package is build with maven. Install maven and do:

```
mvn install
```

This will produce a `target` directory with the following three jars:
- javacg-0.1-SNAPSHOT.jar: This is the standard maven packaged jar with static and dynamic call graph generator classes
- `javacg-0.1-SNAPSHOT-static.jar`: This is an executable jar which includes the static call graph generator
- `javacg-0.1-SNAPSHOT-dycg-agent.jar`: This is an executable jar which includes the dynamic call graph generator

#### Run

Instructions for running the callgraph generators

##### Static

`javacg-static` accepts as arguments the jars to analyze.

```
java -jar javacg-0.1-SNAPSHOT-static.jar lib1.jar lib2.jar...
```

`javacg-static` produces combined output in the following format:

###### For methods

```
  M:class1:<method1>(arg_types) (typeofcall)class2:<method2>(arg_types)
```

The line means that `method1` of `class1` called `method2` of `class2`.
The type of call can have one of the following values (refer to
the [JVM specification](http://java.sun.com/docs/books/jvms/second_edition/html/Instructions2.doc6.html)
for the meaning of the calls):

 * `M` for `invokevirtual` calls
 * `I` for `invokeinterface` calls
 * `O` for `invokespecial` calls
 * `S` for `invokestatic` calls
 * `D` for `invokedynamic` calls

For `invokedynamic` calls, it is not possible to infer the argument types.

###### For classes

```
  C:class1 class2
```

This means that some method(s) in `class1` called some method(s) in `class2`.

##### Dynamic

`javacg-dynamic` uses
[javassist](http://www.csg.is.titech.ac.jp/~chiba/javassist/) to insert probes
at method entry and exit points. To be able to analyze a class `javassist` must
resolve all dependent classes at instrumentation time. To do so, it reads
classes from the JVM's boot classloader. By default, the JVM sets the boot
classpath to use Java's default classpath implementation (`rt.jar` on
Win/Linux, `classes.jar` on the Mac). The boot classpath can be extended using
the `-Xbootclasspath` option, which works the same as the traditional
`-classpath` option. It is advisable for `javacg-dynamic` to work as expected,
to set the boot classpath to the same, or an appropriate subset, entries as the
normal application classpath.

Moreover, since instrumenting all methods will produce huge callgraphs which
are not necessarily helpful (e.g. it will include Java's default classpath
entries), `javacg-dynamic` includes support for restricting the set of classes
to be instrumented through include and exclude statements. The options are
appended to the `-javaagent` argument and has the following format

```
-javaagent:javacg-dycg-agent.jar="incl=mylib.*,mylib2.*,java.nio.*;excl=java.nio.charset.*"
```

The example above will instrument all classes under the the `mylib`, `mylib2` and
`java.nio` namespaces, except those that fall under the `java.nio.charset` namespace.

```
java
-Xbootclasspath:/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar:mylib.jar
-javaagent:javacg-0.1-SNAPSHOT-dycg-agent.jar="incl=mylib.*;"
-classpath mylib.jar mylib.Mainclass
```

`javacg-dynamic` produces two kinds of output. On the standard output, it
writes method call pairs as shown below:

```
class1:method1 class2:method2 numcalls
```

It also produces a file named `calltrace.txt` in which it writes the entry
and exit timestamps for methods, thereby turning `javacg-dynamic` into
a poor man's profiler. The format is the following:

```
<>[stack_depth][thread_id]fqdn.class:method=timestamp_nanos
```

The output line starts with a `<` or `>` depending on whether it is a method
entry or exit. It then writes the stack depth, thread id and the class and
method name, followed by a timestamp. The provided `process_trace.rb`
script processes the callgraph output to generate total time per method
information.

#### Examples

The following examples instrument the
[Dacapo benchmark suite](http://dacapobench.org/) to produce dynamic call graphs.
The Dacapo benchmarks come in a single big jar archive that contains all dependency
libraries. To build the boot class path required for the javacg-dyn program,
extract the `dacapo.jar` to a directory: all the required libraries can be found
in the `jar` directory.

Running the batik Dacapo benchmark:

```
java -Xbootclasspath:/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar:jar/batik-all.jar:jar/xml-apis-ext.jar -javaagent:target/javacg-0.1-SNAPSHOT-dycg-agent.jar="incl=org.apache.batik.*,org.w3c.*;" -jar dacapo-9.12-bach.jar batik -s small |tail -n 10
```
<br/>

```
[...]
org.apache.batik.dom.AbstractParentNode:appendChild org.apache.batik.dom.AbstractParentNode:fireDOMNodeInsertedEvent 6270<br/>
org.apache.batik.dom.AbstractParentNode:fireDOMNodeInsertedEvent org.apache.batik.dom.AbstractDocument:getEventsEnabled 6280<br/>
org.apache.batik.dom.AbstractParentNode:checkAndRemove org.apache.batik.dom.AbstractNode:getOwnerDocument 6280<br/>
org.apache.batik.dom.util.DoublyIndexedTable:put org.apache.batik.dom.util.DoublyIndexedTable$Entry:DoublyIndexedTable$Entry 6682<br/>
org.apache.batik.dom.util.DoublyIndexedTable:put org.apache.batik.dom.util.DoublyIndexedTable:hashCode 6693<br/>
org.apache.batik.dom.AbstractElement:invalidateElementsByTagName org.apache.batik.dom.AbstractElement:getNodeType 7198<br/>
org.apache.batik.dom.AbstractElement:invalidateElementsByTagName org.apache.batik.dom.AbstractDocument:getElementsByTagName 14396<br/>
org.apache.batik.dom.AbstractElement:invalidateElementsByTagName org.apache.batik.dom.AbstractDocument:getElementsByTagNameNS 28792<br/>
```

Running the lucene Dacapo benchmark:

```
java -Xbootclasspath:/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar:jar/lucene-core-2.4.jar:jar/luindex.jar -javaagent:target/javacg-0.1-SNAPSHOT-dycg-agent.jar="incl=org.apache.lucene.*;" -jar dacapo-9.12-bach.jar luindex -s small |tail -n 10
```
<br/><br/>

```
[...]
org.apache.lucene.analysis.Token:setTermBuffer org.apache.lucene.analysis.Token:growTermBuffer 43449<br/>
org.apache.lucene.analysis.CharArraySet:getSlot org.apache.lucene.analysis.CharArraySet:getHashCode 43472<br/>
org.apache.lucene.analysis.CharArraySet:getSlot org.apache.lucene.analysis.CharArraySet:equals 46107<br/>
org.apache.lucene.index.FreqProxTermsWriter:appendPostings org.apache.lucene.store.IndexOutput:writeVInt 46507<br/>
org.apache.lucene.store.IndexInput:readVInt org.apache.lucene.index.ByteSliceReader:readByte 63927<br/>
org.apache.lucene.index.TermsHashPerField:writeVInt org.apache.lucene.index.TermsHashPerField:writeByte 63927<br/>
org.apache.lucene.store.IndexOutput:writeVInt org.apache.lucene.store.BufferedIndexOutput:writeByte 94239<br/>
org.apache.lucene.index.TermsHashPerField:quickSort org.apache.lucene.index.TermsHashPerField:comparePostings 107343<br/>
org.apache.lucene.analysis.Token:termBuffer org.apache.lucene.analysis.Token:initTermBuffer 162115<br/>
org.apache.lucene.analysis.Token:termLength org.apache.lucene.analysis.Token:initTermBuffer 205554<br/>
```

#### Known Restrictions

* The static call graph generator does not account for methods invoked via
  reflection.
* The dynamic call graph generator will not work reliably (or at all) for
  multithreaded programs
* The dynamic call graph generator does not handle exceptions very well, so some
methods might appear as having never returned

#### Author

Georgios Gousios <gousiosg@gmail.com>

#### License

[2-clause BSD](http://www.opensource.org/licenses/bsd-license.php)
