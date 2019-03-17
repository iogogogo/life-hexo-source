---
title: Java拆分shell命令成数组
date: 2019-01-29 10:39:33
tags: Java
---

# 命令行拆分成string数组

- 原始命令如下


```shell
-p /path -d "here's my description" --verbose other args
```

- 需要的拆分结果如下

```
Array[0] = -p
Array[1] = /path
Array[2] = -d
Array[3] = here's my description
Array[4] = --verbose
Array[5] = other
Array[6] = args
```

[stackoverflow上的问题以及解决方案](https://stackoverflow.com/questions/3259143/split-a-string-containing-command-line-parameters-into-a-string-in-java#)

[依赖的ant jar](https://mvnrepository.com/artifact/org.apache.ant/ant)

scala 可以直接使用scala.tools.cmd提供的api

- 解决方案

```java
/**
 * [code borrowed from ant.jar]
 * Crack a command line.
 * @param toProcess the command line to process.
 * @return the command line broken into strings.
 * An empty or null toProcess parameter results in a zero sized array.
 */
public static String[] translateCommandline(String toProcess) {
    if (toProcess == null || toProcess.length() == 0) {
        //no command? no string
        return new String[0];
    }
    // parse with a simple finite state machine

    final int normal = 0;
    final int inQuote = 1;
    final int inDoubleQuote = 2;
    int state = normal;
    final StringTokenizer tok = new StringTokenizer(toProcess, "\"\' ", true);
    final ArrayList<String> result = new ArrayList<String>();
    final StringBuilder current = new StringBuilder();
    boolean lastTokenHasBeenQuoted = false;

    while (tok.hasMoreTokens()) {
        String nextTok = tok.nextToken();
        switch (state) {
        case inQuote:
            if ("\'".equals(nextTok)) {
                lastTokenHasBeenQuoted = true;
                state = normal;
            } else {
                current.append(nextTok);
            }
            break;
        case inDoubleQuote:
            if ("\"".equals(nextTok)) {
                lastTokenHasBeenQuoted = true;
                state = normal;
            } else {
                current.append(nextTok);
            }
            break;
        default:
            if ("\'".equals(nextTok)) {
                state = inQuote;
            } else if ("\"".equals(nextTok)) {
                state = inDoubleQuote;
            } else if (" ".equals(nextTok)) {
                if (lastTokenHasBeenQuoted || current.length() != 0) {
                    result.add(current.toString());
                    current.setLength(0);
                }
            } else {
                current.append(nextTok);
            }
            lastTokenHasBeenQuoted = false;
            break;
        }
    }
    if (lastTokenHasBeenQuoted || current.length() != 0) {
        result.add(current.toString());
    }
    if (state == inQuote || state == inDoubleQuote) {
        throw new RuntimeException("unbalanced quotes in " + toProcess);
    }
    return result.toArray(new String[result.size()]);
}
```