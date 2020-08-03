---
title: javac编译java文件
date: 2019-11-05 23:32:45
tags: Java
cover: true
categories: Java
---
# javac编译java文件

最近做银行项目，生产上新给了一个csv文件，程序需要读取内容进行清洗，但是拿不出来看，也不可能为了这一个小测试专门打审批给生产上传一个程序包，就想到了使用javac写个简单的测试看下数据。下面先来复习一下javac这个命令



下面测试的所有java源文件统一保存在 `/Users/xxx/Downloads/compiler`

## 编译单文件

现在写一段简单的Java代码

```java
public class Test1 {
    public static void main(String... args){
        System.out.println("hello world!!!");
    }
}
```

使用javac编译

```shell
➜  compiler pwd
/Users/xxx/Downloads/compiler # 当前路径
➜  compiler ls -lrt
total 8
-rw-r--r--@ 1 xxx  staff  119 11  5 22:11 Test1.java # 这里是源文件
➜  compiler javac Test1.java # 使用javac进行编译
➜  compiler ls -lrt
total 16
-rw-r--r--@ 1 xxx  staff  119 11  5 22:11 Test1.java
-rw-r--r--  1 xxx  staff  418 11  5 22:28 Test1.class # 编译以后的class文件
➜  compiler java Test1 # 运行结果
hello world!!!
```

可以看到结果就是我们正常的一个输出 hello world

## 编译两个相互依赖的Java源文件

- A.java

```java
public class A {
    public void sayHi() {
        System.out.println("我是 class A  hello world!!!");
    }
}
```

- B.java

```java
public class B {
    public static void main(String... args) {
        A a = new A();
        a.sayHi();
    }
}
```

使用javac分别进行编译，这里注意先后顺序，先编译A.java，在编译B.java，当然，因为B中引用了A，A.java也可以不用编译，在编译B.java的时候，会自动编译A.java的，后面我们会看到

```shell
➜  compiler ls -lrt
total 24
-rw-r--r--@ 1 xxx  staff  119 11  5 22:11 Test1.java
-rw-r--r--@ 1 xxx  staff  112 11  5 22:20 B.java
-rw-r--r--@ 1 xxx  staff  113 11  5 22:21 A.java
➜  compiler javac A.java
➜  compiler javac B.java
➜  compiler ls -lrt
total 40
-rw-r--r--@ 1 xxx  staff  119 11  5 22:11 Test1.java
-rw-r--r--@ 1 xxx  staff  112 11  5 22:20 B.java
-rw-r--r--@ 1 xxx  staff  113 11  5 22:21 A.java
-rw-r--r--  1 xxx  staff  402 11  5 22:39 A.class
-rw-r--r--  1 xxx  staff  297 11  5 22:39 B.class
➜  compiler java B # 运行B这个程序
我是 class A  hello world!!!
```

## 编译带外部依赖的jar

这里比如我们需要使用Google的guava-27.1-jre.jar，使用其中的api读取一个文本文件的内容。

为了方便，将guava-27.1-jre.jar复制到当前路径，当然也可以使用绝对路径，新建文件GuavaTest.java


```shell
➜  compiler ls -lrt
total 5400
-rw-r--r--  1 xxx  staff  2746671 11  5 09:33 guava-27.1-jre.jar # 新复制的外部jar文件
-rw-r--r--@ 1 xxx  staff      119 11  5 22:11 Test1.java
-rw-r--r--@ 1 xxx  staff      112 11  5 22:20 B.java
-rw-r--r--@ 1 xxx  staff      113 11  5 22:21 A.java
-rw-r--r--@ 1 xxx  staff      663 11  5 22:54 GuavaTest.java
```

GuavaTest.java内容

```java

import com.google.common.io.Files;

import java.io.File;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.List;

/**
 * Created by xxx on 2019-11-05.
 */
public class GuavaTest {

    public static void main(String[] args) {
        String path = "/Users/xxx/Downloads/compiler/A.java";
        try {
          	// 按行读取文件内容
            List<String> list = Files.readLines(new File(path), StandardCharsets.UTF_8);
            list.forEach(System.out::println);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

然后使用javac进行编译，和预期的结果不一致，引用了guava-27.1-jre.jar中的class不存在

```shell
➜  compiler javac GuavaTest.java
GuavaTest.java:2: 错误: 程序包com.google.common.io不存在
import com.google.common.io.Files;
                           ^
GuavaTest.java:19: 错误: 找不到符号
            List<String> list = Files.readLines(new File(path), StandardCharsets.UTF_8);
                                ^
  符号:   变量 Files
  位置: 类 GuavaTest
2 个错误
```

那么这个时候就不能单纯的使用`javac GuavaTest.java` 而是需要使用java -cp 指定外部依赖的jar路径，可以看到加了 -cp 参数指定外部依赖的jar以后编译成功了

```shell
➜  compiler javac -cp guava-27.1-jre.jar GuavaTest.java
➜  compiler ls -lrt
total 5408
-rw-r--r--  1 xxx  staff  2746671 11  5 09:33 guava-27.1-jre.jar
-rw-r--r--@ 1 xxx  staff      119 11  5 22:11 Test1.java
-rw-r--r--@ 1 xxx  staff      112 11  5 22:20 B.java
-rw-r--r--@ 1 xxx  staff      113 11  5 22:21 A.java
-rw-r--r--@ 1 xxx  staff      663 11  5 22:54 GuavaTest.java
-rw-r--r--  1 xxx  staff     1548 11  5 22:59 GuavaTest.class
```

这个时候使用`java GuavaTest`去运行程序是否可行呢？

```shell
➜  compiler java GuavaTest
Exception in thread "main" java.lang.NoClassDefFoundError: com/google/common/io/Files
	at GuavaTest.main(GuavaTest.java:19)
Caused by: java.lang.ClassNotFoundException: com.google.common.io.Files
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 1 more
```

答案当然是no

这里运行的时候，我们也需要使用 -cp 参数指定外部依赖的jar文件，但是我们发现依然报错，实际上当我们使用cp参数以后指定外部依赖jar以后，而并没有包含当前目录，就算编译成功，也是找不到主类的。

```shell
➜  compiler java -cp guava-27.1-jre.jar GuavaTest
错误: 找不到或无法加载主类 GuavaTest
```

接下来我们可以把当前路径也加进去，这里特别注意的就是，和上一次的命令相比，添加了`:.`，`:` 作为分隔，`.` 表示当前路径。当然也有博客指出，在Linux下面使用`:`分隔，但是在Windows下面使用`;`分隔，因为我是Mac，没有Windows去验证，这里如果有需要就自己验证一下即可。就可以看到当前读取除了A.java文件中的内容

```shell
➜  compiler java -cp guava-27.1-jre.jar:. GuavaTest
public class A {
    public void sayHi() {
        System.out.println("我是 class A  hello world!!!");
    }
}
```

## 编译源文件中带有package的文件

上面我们描述了编译单个文件，两个相互依赖的文件以及依赖外部文件的处理方式，如果在我们的源文件中包含package又该如何处理呢？接下来我们继续来看，首先我们给A.java和B.java分别添加上package

- A.java

```java
package compiler;

public class A {
    public void sayHi() {
        System.out.println("我是 class A  hello world!!!");
    }
}
```

- B.java

```java
package compiler;

public class B {
    public static void main(String... args) {
        A a = new A();
        a.sayHi();
    }
}
```

接下来我们使用javac进行编译，可以看到A.java可以正常编译，但是B.java编译就出错了。这是因为，添加了package语句后，编译器需要找的是compiler.A类，编译器会首先找到compiler目录，然后再从compiler目录中找到A，此时当前目录中(compiler目录内)不存在compiler子目录，因此，编译器找不到A类，编译失败。这说明我们包含的类路径需要包括类所在的包，**这里需要包含compiler目录的上一级子目录**。

```shell
➜  compiler javac A.java
➜  compiler javac B.java
B.java:5: 错误: 找不到符号
        A a = new A();
        ^
  符号:   类 A
  位置: 类 B
B.java:5: 错误: 找不到符号
        A a = new A();
                  ^
  符号:   类 A
  位置: 类 B
2 个错误
```

接下来我们编译的时候就需要指定compiler的上一级子目录

```shell
➜  compiler ls -lrt
total 5400
-rw-r--r--  1 xxx  staff  2746671 11  5 09:33 guava-27.1-jre.jar
-rw-r--r--@ 1 xxx  staff      119 11  5 22:11 Test1.java
-rw-r--r--@ 1 xxx  staff      663 11  5 22:54 GuavaTest.java
-rw-r--r--@ 1 xxx  staff      131 11  5 23:19 B.java
-rw-r--r--@ 1 xxx  staff      131 11  5 23:19 A.java
# 注意这里使用了两个.. 表示上一级目录，因为B里面依赖了A，所有也会把A.java自动编译，A.java可以不管
➜  compiler javac -cp .. B.java
➜  compiler ls -lrt
total 5416
-rw-r--r--  1 xxx  staff  2746671 11  5 09:33 guava-27.1-jre.jar
-rw-r--r--@ 1 xxx  staff      119 11  5 22:11 Test1.java
-rw-r--r--@ 1 xxx  staff      663 11  5 22:54 GuavaTest.java
-rw-r--r--@ 1 xxx  staff      131 11  5 23:19 B.java
-rw-r--r--@ 1 xxx  staff      131 11  5 23:19 A.java
-rw-r--r--  1 xxx  staff      315 11  5 23:28 B.class
-rw-r--r--  1 xxx  staff      411 11  5 23:28 A.class
# 运行是也一样，需要.. 指定到上一级目录，并且要使用compiler.B
➜  compiler java -cp .. compiler.B
我是 class A  hello world!!! # A.java中的输出语句
```



# 小结

以上内容虽然平时很少用到，应用场景虽然不多，但是也是知识点的一次巩固复习，有不足的地方，欢迎指正。


