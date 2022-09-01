java-callgraph: Java Call Graph Utilities
=========================================

A suite of programs for generating static and dynamic call graphs in Java.
一套用于在Java中生成静态和动态调用图的程序。

* javacg-static: Reads classes from a jar file, walks down the method bodies and
   prints a table of caller-caller relationships.
* javacg-static：从jar文件中读取类，遍历方法主体并打印一个调用者-调用者关系表。

* javacg-dynamic: Runs as a [Java agent](http://download.oracle.com/javase/6/docs/api/index.html?java/lang/instrument/package-summary.html) and instruments
  the methods of a user-defined set of classes in order to track their invocations.
  At JVM exit, prints a table of caller-callee relationships, along with a number
  of calls
* javacg-dynamic:作为一个[Java代理]运行，并且记录用户定义的一组类的方法，以便跟踪它们的调用。

#### Compile

The java-callgraph package is build with maven. Install maven and do:

```
mvn install
```

This will produce a `target` directory with the following three jars:
这将生成包含以下三个jar的“目标”目录
- javacg-0.1-SNAPSHOT.jar: This is the standard maven packaged jar with static and dynamic call graph generator classes 
- javacg-0.1-SNAPSHOT.jar：这是标准的maven打包jar，带有静态和动态调用图生成器类
- `javacg-0.1-SNAPSHOT-static.jar`: This is an executable jar which includes the static call graph generator
- `javacg-0.1-SNAPSHOT-static.jar`：这是一个包含静态调用图形生成器的可执行jar
- `javacg-0.1-SNAPSHOT-dycg-agent.jar`: This is an executable jar which includes the dynamic call graph generator
- `javacg-0.1-SNAPSHOT-dycg-agent.jar`：这是一个包含动态调用图生成器的可执行jar

#### Run

Instructions for running the callgraph generators
运行调用图生成器的说明

##### Static

`javacg-static` accepts as arguments the jars to analyze.
`javacg-static`接受要分析的jar文件作为参数

```
java -jar javacg-0.1-SNAPSHOT-static.jar lib1.jar lib2.jar...
```

`javacg-static` produces combined output in the following format:
以下列格式产生合并输出:

###### For methods

```
  M:class1:<method1>(arg_types) (typeofcall)class2:<method2>(arg_types)
```

The line means that `method1` of `class1` called `method2` of `class2`.
The type of call can have one of the following values (refer to the [JVM specification](http://java.sun.com/docs/books/jvms/second_edition/html/Instructions2.doc6.html) for the meaning of the calls):
调用类型可以有以下值之一

 * `M` for `invokevirtual` calls
 * `I` for `invokeinterface` calls
 * `O` for `invokespecial` calls
 * `S` for `invokestatic` calls
 * `D` for `invokedynamic` calls

For `invokedynamic` calls, it is not possible to infer the argument types.
对于“动态调用”调用，不可能推断参数类型。

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

“javacg-dynamic”使用[javassist]在方法入口点和出口点插入探针。为了能够分析类，“javassist”必须在插装时解析所有依赖类。
为此，它从JVM的引导类加载器读取类。默认情况下，JVM将引导类路径设置为使用Java的默认类路径实现。
可以使用' -Xbootclasspath '选项扩展引导类路径，该选项的工作原理与传统的' -classpath '选项相同。
建议“javacs -dynamic”按预期工作，将引导类路径设置为与常规应用程序类路径相同或适当的条目子集。

Moreover, since instrumenting all methods will produce huge callgraphs which
are not necessarily helpful (e.g. it will include Java's default classpath
entries), `javacg-dynamic` includes support for restricting the set of classes
to be instrumented through include and exclude statements. The options are
appended to the `-javaagent` argument and has the following format

此外，由于插装所有方法将产生巨大的调用图，而这些调用图不一定有帮助(例如，它将包括Java的默认类路径条目)，
“javacs -dynamic”支持通过include和exclude语句限制要插装的类集。
这些选项被附加到' -javaagent '参数中，并具有以下格式

```
-javaagent:javacg-dycg-agent.jar="incl=mylib.*,mylib2.*,java.nio.*;excl=java.nio.charset.*"
```

The example above will instrument all classes under the the `mylib`, `mylib2` and
`java.nio` namespaces, except those that fall under the `java.nio.charset` namespace.

上面的示例将检测' mylib '、' mylib2 '和' java '下的所有类。
nio名称空间，除了那些属于java.nio的名称空间。字符集的名称空间。

```
java
-Xbootclasspath:/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar:mylib.jar
-javaagent:javacg-0.1-SNAPSHOT-dycg-agent.jar="incl=mylib.*;"
-classpath mylib.jar mylib.Mainclass
```

`javacg-dynamic` produces two kinds of output. On the standard output, it
writes method call pairs as shown below:
“javacs -dynamic”生成两种输出。在标准输出中，它写入方法调用对，如下所示:

```
class1:method1 class2:method2 numcalls
```

It also produces a file named `calltrace.txt` in which it writes the entry
and exit timestamps for methods, thereby turning `javacg-dynamic` into
a poor man's profiler. The format is the following:

它还生成一个名为“calltrace.txt”的文件，其中编写方法的进入和退出时间戳，从而将“javacg-dynamic”变成了一个穷人的剖析器。格式如下:

```
<>[stack_depth][thread_id]fqdn.class:method=timestamp_nanos
```

The output line starts with a `<` or `>` depending on whether it is a method
entry or exit. It then writes the stack depth, thread id and the class and
method name, followed by a timestamp. The provided `process_trace.rb`
script processes the callgraph output to generate total time per method
information.

输出行以' &lt; '或' &gt; '开始，这取决于它是一个方法入口还是出口。然后写入堆栈深度、线程id、类和方法名，后面是时间戳。提供的process_trace。rb的脚本处理callgraph输出，以生成每个方法信息的总时间。

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

* The static call graph generator does not account for methods invoked via reflection.
静态调用图形生成器不考虑通过反射调用的方法。
* The dynamic call graph generator will not work reliably (or at all) for
  multithreaded programs
* The dynamic call graph generator does not handle exceptions very well, so some
methods might appear as having never returned

#### Author

Georgios Gousios <gousiosg@gmail.com>

#### License

[2-clause BSD](http://www.opensource.org/licenses/bsd-license.php)
