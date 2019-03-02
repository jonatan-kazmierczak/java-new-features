# Java 12 - new features 

Features with their ratings:
* :+1: useful
* :-1: removed
* :disappointed: not available
* :confused: no added value / not working as expected

## :+1: JShell usability improvements
See JShell help for more details
```
$ jshell
|  Welcome to JShell -- Version 12
|  For an introduction type: /help intro


jshell> /help
|  ...
|
|  Subjects:
|  
|  intro
|       an introduction to the jshell tool
|  keys
|       a description of readline-like input editing
|  id
|       a description of snippet IDs and how use them
|  shortcuts
|       a description of keystrokes for snippet and command completion,
|       information access, and automatic code generation
|  context
|       a description of the evaluation context options for /env /reload and /reset
|  rerun
|       a description of ways to re-evaluate previously entered snippets


jshell> /help keys
|  
|                                    keys
|                                    ====
|  
|  The jshell tool provides line editing support to allow you to navigate within
|  and edit snippets and commands. The current command/snippet can be edited,
|  or prior commands/snippets can be retrieved from history, edited, and executed.
|  This support is similar to readline/editline with simple emacs-like bindings.
|  There are also jshell tool specific key sequences.
|
|  ...
```

You can try the following code snippets in `jshell`.

## :-1: ~326: Raw String Literals (Preview)~
Feature removed. For good.  
There are 2 leftovers in `String`:
### :confused: transform
How to rewrite this snippet
```java
Integer.parseInt( "65536" )
```
in overcomplicated and overweighted way?

Here you are:
```java
"65536".transform( Integer::parseInt )
```

### :confused: indent
Would you like to format your JSON?  
You are expecting too much from Java.
But you can adjust indentation:

```java
var jsonInner = "'name': 'OpenJDK',\n'version': 12\n".replace( '\'', '"' );
System.out.println( jsonInner );
var jsonOuter = "{\n" + jsonInner.indent( 4 ) + "}\n";
System.out.println( jsonOuter );
System.out.println( jsonOuter.indent( -8 ) );
```

## :disappointed: 325: Switch Expressions (Preview)

By using switch expressions with default configuration, you will see an error:
```
|  Error:
|  switch expressions are a preview feature and are disabled by default.
|    (use --enable-preview to enable switch expressions)
```

You have to enable this feature first:
```
/usr/lib/jvm/java-12-openjdk/bin/jshell --enable-preview
```

Then you can use it:
```java
String developerRating( int numberOfChildren ) {
    return switch (numberOfChildren) {
        case 0 -> "open source contributor";
        case 1 -> "junior";
        case 2 -> "senior";
        case 3 -> "expert";
        default -> {
            if (numberOfChildren < 0) throw new IndexOutOfBoundsException( numberOfChildren );
            break "manager";
        }
    };
}

developerRating( 0 ); // ==> "open source contributor"
developerRating( 4 ); // ==> "manager"
```

## Other extensions to the standard library
### CompactNumberFormat

#### :confused: Short variant

In IT we often use multiplicators, i.e. 1K, 1M, 1G, 1T, 1P, etc.  
It would be useful to have these multiplicators available out-of-the-box, right?

Let's try a new feature - `CompactNumberFormat` - to see, if it helps.

```java
import java.text.*;
var cnf = NumberFormat.getCompactNumberInstance();

cnf.format( 1L << 10 ); // ==> 1K
cnf.format( 1920 );     // ==> 2K - instead of 1.9K
cnf.format( 1L << 20 ); // ==> 1M
cnf.format( 1L << 30 ); // ==> 1B - why?
cnf.format( 1L << 40 ); // ==> 1T
cnf.format( 1L << 50 ); // ==> 1126T
```

They are not, what you would expect - they are unicode formats :)  
Please see the official website for details: https://unicode.org/reports/tr35/tr35-numbers.html#Compact_Number_Formats

#### :+1: Long variant

```java
import java.text.*;
var cnf = NumberFormat.getCompactNumberInstance( Locale.GERMAN, NumberFormat.Style.LONG );

cnf.format( 1L << 10 ); // ==> 1 Tausend
cnf.format( 1L << 20 ); // ==> 1 Million
cnf.format( 1L << 21 ); // ==> 2 Million (!) - instead of 2 Millionen
cnf.format( 1L << 24 ); // ==> 17 Millionen
cnf.format( 1L << 30 ); // ==> 1 Milliarde
cnf.format( 1L << 31 ); // ==> 2 Milliarde (!) - instead of 2 Milliarden
cnf.format( 1L << 34 ); // ==> 17 Milliarden
cnf.format( 1L << 40 ); // ==> 1 Billion
cnf.format( 1L << 41 ); // ==> 2 Billion (!) - instead of 2 Billionen
cnf.format( 1L << 50 ); // ==> 1126 Billionen
```

### :confused: Files.mismatch

```java
Files.mismatch( Paths.get("doesn't exist"), Paths.get("doesn't exist") ); // ==> -1
```
gives result `-1` - means the 2 non existing files are equal.  
Is this what you would expect?

### :confused: Collectors.teeing

How to rewrite this snippet
```java
Stream.of( 2, 4, 8, 0x10 ).collect( Collectors.averagingInt( i -> i ) )
```
in overcomplicated and overweighted way?

Here you are:
```java
Stream.of( 2, 4, 8, 0x10 ).collect( Collectors.teeing(
    Collectors.summingDouble( i -> i ),
    Collectors.counting(),
    (s, c) -> s / c
) )
```

## :confused: 334: JVM Constants API
Please don't be surprise by 2 new interfaces implemented by some core classes  
(diagrams generated by [Class Visualizer](http://class-visualizer.net/)):

![Constable](diagrams/Constable.png "Constable")

![ConstantDesc](diagrams/ConstantDesc.png "ConstantDesc")

adding 2 new methods - maybe unexpected.  
They are not intended to add productivity to your daily work. Instead, they are supposed to simplify internal constant loading.

However, you are welcomed to try these new methods:
```java
"Java 12".describeConstable();
"Java 12".resolveConstantDesc( null )
```

## :confused: 341: Default CDS Archives
Nothing new - unless you are building custom JRE images.

## :disappointed: 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)

:+1: Included in RedHat builds.  
You can find it in OpenJDK since v1.8.0 on RedHat Linux 7.x.
You can turn it on as follows:
```
java -XX:+UseShenandoahGC
```

:+1: Included in Debian builds:
```
$ java -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -verbose:gc -version
[0.001s][info][gc] Consider -XX:+ClassUnloadingWithConcurrentMark if large pause times are observed on class-unloading sensitive workloads
[0.003s][info][gc] Heuristics ergonomically sets -XX:+ExplicitGCInvokesConcurrent
[0.003s][info][gc] Heuristics ergonomically sets -XX:+ShenandoahImplicitGCInvokesConcurrent
[0.005s][info][gc] Using Shenandoah
openjdk version "12" 2019-03-19
OpenJDK Runtime Environment (build 12+33-Debian-1)
OpenJDK 64-Bit Server VM (build 12+33-Debian-1, mixed mode, sharing)
```

:disappointed: Not included in the official build from Oracle.  
See this issue for details: https://bugs.openjdk.java.net/browse/JDK-8215030

Try of turning it on leads to an error:
```
$ java -XX:+UseShenandoahGC
Error: VM option 'UseShenandoahGC' is experimental and must be enabled via -XX:+UnlockExperimentalVMOptions.

$ java -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC
Error occurred during initialization of VM
Option -XX:+UseShenandoahGC not supported
```

## 344: Abortable Mixed Collections for G1

## :disappointed: 346: Promptly Return Unused Committed Memory from G1
Made in :switzerland:  
:disappointed: switched off by default.

## :confused: 230: Microbenchmark Suite
## :confused: 340: One AArch64 Port, Not Two

# Leftovers from version 11
## :-1: JDK Mission Control 7
:-1: Still in Early-Acces phase, all builds are removed.
Please check the download page: http://jdk.java.net/jmc/
