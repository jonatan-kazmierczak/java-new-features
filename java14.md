# Java 14 - new features

## Language syntax
### 305: 	Pattern Matching for instanceof (Preview)
Preview feature - requires `--enable-preview` parameter during compilation.

#### Description
Extended `instanceof` operator allowing to declare local variable of the checked type.

Inspired by
[Smart Casts](https://kotlinlang.org/docs/reference/typecasts.html#smart-casts) from Kotlin.

#### Example
Method checking, if the parameter is null, blank String or empty collection.

Before:
```java
boolean isNullOrEmpty( Object o ) {
  return
    o == null ||
    o instanceof String && ((String) o).isBlank() ||
    o instanceof Collection && ((Collection) o).isEmpty();
}
```

Now:
```java
boolean isNullOrEmpty( Object o ) {
  return
    o == null ||
    o instanceof String s && s.isBlank() ||
    o instanceof Collection c && c.isEmpty();
}
```

### 359: 	Records (Preview)
Preview feature - requires `--enable-preview` parameter during compilation.

#### Description
Immutable value-holding class.

Inspired by:
- [Case Classes](https://docs.scala-lang.org/tour/case-classes.html) from Scala
- [Data Classes](https://kotlinlang.org/docs/reference/data-classes.html) from Kotlin
- [Record Type](https://www.freepascal.org/docs-html/ref/refsu15.html) from Pascal

#### Example
Class holding 2 information about the person: name (mandatory) and an optional reference to a partner.

Simplest definition with 2 fields:
```java
public record Person( String name, Person partner ) {}
```

Extended definition with an additional constructor, requiring only name, and an additional method returning name in upper case:
```java
public record Person( String name, Person partner ) {
  public Person( String name ) { this( name, null ); }
  public String getNameInUppercase() { return name.toUpperCase(); }
}
```

The compiler turns it into the following class:
```java
public final class Person extends Record {
  private final String name;
  private final Person partner;
  
  public Person(String name) { this( name, null ); }
  public Person(String name, Person partner) { this.name = name; this.partner = partner; }

  public String getNameInUppercase() { return name.toUpperCase(); }
  public String toString() { /* ... */ }
  public final int hashCode() { /* ... */ }
  public final boolean equals(Object o) { /* ... */ }
  public String name() { return name; }
  public Person partner() { return partner; }
}
```

Usage:
```java
var man = new Person("Adam");
var woman = new Person("Eve", man);
woman.toString(); // ==> "Person[name=Eve, partner=Person[name=Adam, partner=null]]"

woman.partner().name(); // ==> "Adam"
woman.getNameInUppercase(); // ==> "EVE"

// Deep equals
new Person("Eve", new Person("Adam")).equals( woman ); // ==> true
```

#### Summary
- unlike Data Classes from Kotlin, Records are not Java Beans - they don't contain getters
- field readers in Records are generated in in the same way, as in Case Classes from Scala - they have the same names as fields
- unlike in Scala and Kotlin, there is no way to define mutable properties

### 368: 	Text Blocks (Second Preview)
Preview feature - requires `--enable-preview` parameter during compilation.

#### Description
Multi-line string literals.  
Initially introduced in Java 13.

Inspiration:
- Functionality:
  - [Triple-single-quoted string](https://groovy-lang.org/syntax.html#_triple_single_quoted_string) from Groovy
  - [Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) from JavaScript
- Syntax:
  - Multi-line string literals from Scala
  - [Triple-double-quoted string](https://groovy-lang.org/syntax.html#_triple_double_quoted_string) from Groovy
  - [Raw string literals](https://kotlinlang.org/docs/reference/basic-types.html#string-literals) from Kotlin

#### Examples

##### Example 1
Simple JSON object.

```java
var json = """{ "language": "Java", "version": 14 }"""
```

Result:
```
|  Error:
|  illegal text block open delimiter sequence, missing line terminator
|  var json = """{ "language": "Java", "version": 14 }"""
|                ^
```
-> one-line texts cannot be used (unlike in other mentioned languages)

##### Example 2
SQL select query - with required line terminator after opening `"""` sequence.

```java
var table = "person";
var field = "name";
var sql = """
    select ${field}, "\t" as test
    from ${table}
    order by ${field}
""";
// sql ==> "    select ${field}, \"\t\" as test\n    from ${table}\n    order by ${field}\n"
```

Let's try to use the `sql` variable with stripping of the indentation:
```java
System.out.println( sql.stripIndent() );
```

Result:
```
    select ${field}, "  " as test
    from ${table}
    order by ${field}

```
- `stripIndent` didn't remove the indentation and the empty trailing line - unlike `trimIndent` in Kotlin
- `\t` was replaced with invisible tab character - like in Groovy and JavaScript
- variables are not interpolated - like in Scala and in Triple-single-quoted string in Groovy, but unlike Kotlin, JavaScript and Triple-double-quoted string in Groovy

Let's try to strip the indentation, remove the trailing blank line and get the variables replaced with their values:
```java
System.out.println(
  sql
    .trim()
    .indent( -4 ) // it will not delete any non-white-space character, so the value can be also less - i.e. -128
    .replace( "${table}", table )
    .replace( "${field}", field )
);
```

Result:
```
select name, "  " as test
from person
order by name
```
as expected - no trailing empty line, the indent is stripped and variables are replaced with values.

##### Example 3
Another variant of the SQL query from the previous example:

```java
var sql = """
    select *
    from %s
    order by %s \
""";
// sql ==> "    select *\n    from %s\n    order by %s "
```

Let's try to strip the indentation:
```java
System.out.println(
  sql
    .stripIndent()
    .formatted( table, field )
);
```

Result:
```
select *
from person
order by name
```
as expected - no trailing empty line, the indent is stripped.

### 361: 	Switch Expressions (Standard)
#### Description
Simplified variant of the switch statement.  
Initially introduced in Java 12, adjusted in Java 13.

Inspired by
[When Expression](https://kotlinlang.org/docs/reference/control-flow.html#when-expression) from Kotlin.

#### Example
Method rating developer based on a number of children:

```java
String developerRating( int numberOfChildren ) {
  return switch (numberOfChildren) {
    case 0 -> "open source contributor";
    case 1, 2 -> "junior";
    case 3 -> "senior";
    default -> {
      if (numberOfChildren < 0) throw new IndexOutOfBoundsException( numberOfChildren );
      yield "manager";
    }
  };
}
```

Usage:
```java
developerRating( 0 ); // ==> "open source contributor"
developerRating( 2 ); // ==> "junior"
developerRating( 4 ); // ==> "manager"
```

## JVM - Garbage Collectors
The following GCs are currently available (on HotSpot VM):
- Epsilon (Experimental)
- G1
- Parallel
- ParallelOld
- Serial
- Shenandoah (Experimental; productive in RedHat OpenJDK, excluded from Oracle builds)
- Z (Experimental)

You can enable any of them with option: `-XX:+Use${name}GC` (`${name}` is one of above).  
Enabling an experimental GC require additionally option `-XX:+UnlockExperimentalVMOptions`.  
See an example below.

**Note**: The blog post [High performance at low cost – choose the best JVM and the best Garbage Collector for your needs](https://blog.oio.de/2020/01/13/high-performance-at-low-cost-choose-the-best-jvm-and-the-best-garbage-collector-for-your-needs/) contains extensive comparison of different JVMs and the GCs included in them.

### 364, 365: ZGC on macOS and Windows
ZGC is now available on all 3 main platforms: Linux, macOS and Windows.  
You can enable it with options: `-XX:+UnlockExperimentalVMOptions -XX:+UseZGC`

Example of usage:
```
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -verbose:gc -version
```

Result:
```
[0.105s][info][gc] Using The Z Garbage Collector
...
```

### 345: 	NUMA-Aware Memory Allocation for G1
### 363: 	Remove the Concurrent Mark Sweep (CMS) Garbage Collector
### 366: 	Deprecate the ParallelScavenge + SerialOld GC Combination

## JVM - general
### 358: 	Helpful NullPointerExceptions
Adds to the `NullPointerException` an informative message about the cause.  
This feature has to be enabled with option: `-XX:+ShowCodeDetailsInExceptionMessages`

The below examples extend the one from the point [359: Records (Preview)](#359-records-preview).

#### Example 1
One method in the invocation chain returns null.

```java
man.partner().name()
```

Result:
```
java.lang.NullPointerException: Cannot invoke "Person.name()" because the return value of "Person.partner()" is null
```

#### Example 2
A parameter passed to a lambda function is null.

```java
Stream.of( man, woman )
  .map( p -> p.partner() )
  .map( p -> p.name() )
  .collect( Collectors.toUnmodifiableList() )
```

Result:
```
java.lang.NullPointerException: Cannot invoke "Person.name()" because "<parameter1>" is null
```

After compilation of the example with parameter `-g:vars`, we get the name of the lambda parameter as well:
```
java.lang.NullPointerException: Cannot invoke "Person.name()" because "p" is null
```

**Note**: keeping one method invocation per line always helps to narrow down the problem.

#### Example 3
A local variable is null.

```java
int calculate() {
  Integer a = 2, b = 4, x = null;
  return a + b * x;
}
calculate();
```

Result:
```
java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because "<local2>" is null
```

Which variable is `local2`?  
And why is `Integer.intValue()` invoked? Reason: unboxing.  
And to get the variable name, the code must be compiled with parameter `-g:vars`. Afterwards we get the name of the local variable as well:
```
java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because "x" is null
```

#### Example 4
An array is null.

```java
int[] arr = null;
arr[2] = 4;
```

Result:
```
java.lang.NullPointerException: Cannot store to int array because "arr" is null
```

#### Example 5
Sorting a list containing null.

```java
var list = Arrays.asList( 2, null, 4 );
list.sort( null );
```

Result:
```
java.lang.NullPointerException: Cannot invoke "java.lang.Comparable.compareTo(Object)" because "a[runHi]" is null
      at ComparableTimSort.countRunAndMakeAscending (ComparableTimSort.java:320)
      at ComparableTimSort.sort (ComparableTimSort.java:188)
      at Arrays.sort (Arrays.java:1040)
      at Arrays.sort (Arrays.java:1227)
      at Arrays$ArrayList.sort (Arrays.java:4218)
```

#### Example 6
Null passed to `List.of`.

```java
Integer x = null;
var list = List.of( 2, x, 4 );
```

Result:
```
java.lang.NullPointerException
      at Objects.requireNonNull (Objects.java:222)
      at ImmutableCollections$ListN.<init> (ImmutableCollections.java:483)
      at List.of (List.java:843)
```
no message in that case; Java standard library should be adjusted.

### 352: 	Non-Volatile Mapped Byte Buffers
### 370: 	Foreign-Memory Access API (Incubator)
API allowing to allocate memory outside of heap and to access it in a safe way.  
It is intended to replace the similar unsafe functionality existing in `Unsafe` class, and to be an alternative to `ByteBuffer` API.

Example - allocation of 4 GB of RAM outside of heap:
```java
import jdk.incubator.foreign.MemorySegment;

try ( var ms = MemorySegment.allocateNative( 1L << 32 ) ) {
  // use allocated RAM
}
```

### 349: 	JFR Event Streaming
#### Available before
##### Java Flight Recorder (JFR)
It is recommended to run every critical Java application with flight recording turned on.

In the following way you can start Java with flight recording, keeping last 1 day of data and dumping the recording into the file `recording.jfr`:
```
java \
-XX:+FlightRecorder \
-XX:StartFlightRecording=disk=true,filename=recording.jfr,dumponexit=true,maxage=1d \
...
```

##### JDK Mission Control (JMC)
You can load flight recording to JMC and analyze it.  
JMC can be downloaded from [here](https://www.oracle.com/technetwork/java/javaseproducts/downloads/jmc7-downloads-5868868.html).  

#### New functionalities
Now it is possible to asynchronously subscribe for the events from within running Java application.

Example - monitoring total CPU usage in 1 second intervals:
```java
import jdk.jfr.consumer.RecordingStream;
import java.time.Duration;

try ( var rs = new RecordingStream() ) {
  rs.enable( "jdk.CPULoad" ).withPeriod( Duration.ofSeconds( 1 ) );
  rs.onEvent( "jdk.CPULoad", event -> {
    System.out.printf( "%.1f %% %n", event.getFloat( "machineTotal" ) * 100 );
  });
  rs.start();
}
```

Result:
```
7.2 % 
7.0 % 
2.5 % 
5.3 % 
```

## Tools
### 343: 	Packaging Tool (Incubator)
### 367: 	Remove the Pack200 Tools and API

## Other
### 362: 	Deprecate the Solaris and SPARC Ports

## Other extensions to the Standard Library
