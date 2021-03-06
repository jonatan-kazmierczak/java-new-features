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

The goal was to have a syntax inspired by template strings from ECMAScript 2015:
```javascript
var json = `\
{
    "name": "OpenJDK",
    "version": 12
}
`;
```

There are 2 leftovers in the `String` class:
### :confused: transform
Before - simpler:
```java
new BigInteger( "65536" ).getLowestSetBit(); // ==> 16
```

Now - more complicated:
```java
"65536".transform( BigInteger::new ).getLowestSetBit(); // ==> 16
```

#### Motivation
The method was intended to simplify String transformations. Is it really the case? Let's see.

Example 1, from [JEP 326 page](http://openjdk.java.net/jeps/326), removal of "line markers", is overcomplicated.  
It can be implemented much simpler since Java 1.4 using regular expressions:
```java
String removeLineMarkers(String text) {
    return Pattern.compile( "^\\|  ", Pattern.MULTILINE ).matcher( text ).replaceAll( "" );
}

var text = "|  Welcome to JShell -- Version 12\n|  For an introduction type: /help intro";
removeLineMarkers( text );
```

Example 2, from [JDK-8203703](https://bugs.openjdk.java.net/browse/JDK-8203703), words capitalizer, is not only overcomplicated, but also buggy.  
It can be implemented correctly and much simpler using regular expressions:
```java
String wordsCapitalizer(String text) {
    return Pattern.compile( "\\b\\p{L}" ).matcher( text ).replaceAll( m -> m.group().toUpperCase() );
}

wordsCapitalizer( " [ voxxed days zürich 2019 ] " );
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

### Before
```java
String developerRating( int numberOfChildren ) {
    switch (numberOfChildren) {
        case 0: return "open source contributor";
        case 1:
        case 2: return "junior";
        case 3: return "senior";
        default:
            if (numberOfChildren < 0) throw new IndexOutOfBoundsException( numberOfChildren );
            return "manager";
    }
}

developerRating( 0 ); // ==> "open source contributor"
developerRating( 4 ); // ==> "manager"
```

### Now
By using switch expressions with default configuration, you will see an error:
```
|  Error:
|  switch expressions are a preview feature and are disabled by default.
|    (use --enable-preview to enable switch expressions)
```

You have to enable this feature first:
```
jshell --enable-preview
```

Then you can use it:
```java
String developerRating( int numberOfChildren ) {
    return switch (numberOfChildren) {
        case 0 -> "open source contributor";
        case 1, 2 -> "junior";
        case 3 -> "senior";
        default -> {
            if (numberOfChildren < 0) throw new IndexOutOfBoundsException( numberOfChildren );
            break "manager";
        }
    };
}

developerRating( 0 ); // ==> "open source contributor"
developerRating( 4 ); // ==> "manager"
```

### JEP Motivation
JEP description includes example determining, how long is each day of the week - how many letters does it include.  
That case can be easily implemented with features existing since Java 1.5:
```java
enum Weekday { Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday, }

int weekdayLength( Weekday day ) { return day.name().length(); }

weekdayLength( Weekday.Monday ); // ==> 6
weekdayLength( Weekday.Saturday ); // ==> 8
```

Other popular example is to determine working/non-working weekdays.
It can be also implemented simply with features existing since Java 1.5:
```java
enum Weekday { Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday, }
var nonWorkingDays = EnumSet.of( Weekday.Saturday, Weekday.Sunday );

boolean isWorkingDay( Weekday day ) { return !nonWorkingDays.contains( day ); }

isWorkingDay( Weekday.Monday ); // ==> true
isWorkingDay( Weekday.Sunday ); // ==> false
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
cnf.format( 1L << 30 ); // ==> 1B - why? Billion, not Giga :)
cnf.format( 1L << 40 ); // ==> 1T
cnf.format( 1L << 50 ); // ==> 1126T
```

They are not, what you would expect - they are unicode formats :)  
Please see the official website for details: https://unicode.org/reports/tr35/tr35-numbers.html#Compact_Number_Formats

#### :confused: Long variant

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
Origin: https://bugs.openjdk.java.net/browse/JDK-8209685

Example of usage:
```java
Stream.of( "Devoxx", "Voxxed Days", "Code One", "Basel One" )
    .collect( Collectors.teeing(
        // first collector
        Collectors.filtering( n -> n.contains("xx"), Collectors.toUnmodifiableList() ),
        // second collector
        Collectors.filtering( n -> n.endsWith("One"), Collectors.toUnmodifiableList() ),
        // merger - automatic type inference doesn't work here
        (List<String> list1, List<String> list2) -> List.of( list1, list2 )
) );
```

#### Motivation
Example from referenced JIRA issue could be easily solved since Java 1.8:
```java
Stream.of( 2, 4, 8, 0x10 ).collect( Collectors.summarizingInt( i -> i ) )
// ==> IntSummaryStatistics{count=4, sum=30, min=2, average=7.500000, max=16}
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
"Java 12".describeConstable();         // ==> Optional[Java 12]
"Java 12".resolveConstantDesc( null ); // ==> "Java 12"
```

## :+1: 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)

Feature developed primarily by Aleksey Shipilëv from Red Hat.
Motivations:
1. application throughput - constant CG pauses regardless of heap size
2. optimization of RAM usage in cloud environments

### :+1: Included in Red Hat builds (productive)
You can find it in OpenJDK since v1.8.0 on Red Hat Linux 7.x.
You can turn it on as demonstrated below:
```
$ java -XX:+UseShenandoahGC -version
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
```

```
$ java -XX:+UseShenandoahGC -verbose:gc -version
[0.008s][info][gc] Using Shenandoah
openjdk version "11.0.1" 2018-10-16 LTS
OpenJDK Runtime Environment 18.9 (build 11.0.1+13-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.1+13-LTS, mixed mode)
```

### :+1: Included in Debian builds (experimental)
```
$ java -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -verbose:gc -version
[0.005s][info][gc] Using Shenandoah
openjdk version "12" 2019-03-19
OpenJDK Runtime Environment (build 12+33-Debian-1)
OpenJDK 64-Bit Server VM (build 12+33-Debian-1, mixed mode, sharing)
```

### :disappointed: Excluded in official builds from Oracle.  
See this issue for details: https://bugs.openjdk.java.net/browse/JDK-8215030  
Possible reason: Shenandoah is a production ready competitor of experimental ZGC, introduced in JDK 11.

Try of turning it on leads to an error:
```
$ java -XX:+UseShenandoahGC
Error: VM option 'UseShenandoahGC' is experimental and must be enabled via -XX:+UnlockExperimentalVMOptions.

$ java -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC
Error occurred during initialization of VM
Option -XX:+UseShenandoahGC not supported
```

## :+1: 346: Promptly Return Unused Committed Memory from G1
Feature initiated by Ruslan Synytsky from Jelastic. It is included in Shenandoah as well.
Motivation: optimization of RAM usage in cloud environments.

See [this very clear and informative presentation](https://www.slideshare.net/jelastic/elastic-jvm-automatic-vertical-scaling-of-the-java-heap) to learn how different Garbage Collectors work.

## :+1: 344: Abortable Mixed Collections for G1
Motivation: introduces G1 collection pauses of configured/predictible length.  
Until then, G1 pauses often exceed configured limits.

## :+1: 341: Default CDS Archives
- CDS feature improves startup time of JVM
- in JDK 11 "Class data sharing is enabled by default" - see https://docs.oracle.com/en/java/javase/11/vm/class-data-sharing.html#GUID-882DC523-706D-403E-8A06-FBBB0E1B2128
- however, required `classes.jsa` is not provided in OpenJDK 11 builds - so the feature doesn't work
- this JEP adds "dump classes to `classes.jsa`" step to OpenJDK 12 builds - so now the file exists and the feature works

## :confused: 230: Microbenchmark Suite
## :confused: 340: One AArch64 Port, Not Two

# Leftovers from version 11
## :-1: JDK Mission Control 7
:-1: Still in Early-Acces phase, all builds are removed.
Please check the download page: http://jdk.java.net/jmc/  

There are many unresolved issues, including the one reported by me:
https://bugs.openjdk.java.net/browse/JMC-6195?jql=project%20%3D%20JMC
