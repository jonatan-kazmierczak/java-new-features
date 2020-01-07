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

Extended definition with an additional constructor and a method:
```java
public record Person( String name, Person partner ) {
  /** Second constructor referring to the first one. */
  public Person( String name ) { this( name, null ); }
  /** Usage of a field. */
  public String getNameInUppercase() { return name.toUpperCase(); }
}
```

Compiled as follows:
```java
public final class Person extends Record {
  private final String name;
  private final Person partner;
  
  public Person(String name);
  public Person(String name, Person partner);

  public String getNameInUppercase();
  public String toString();
  public final int hashCode();
  public final boolean equals(Object o);
  public String name();
  public Person partner();
}
```

Usage:
```java
var man = new Person("Adam");
var woman = new Person("Eve", man);
woman.toString() // ==> "Person[name=Eve, partner=Person[name=Adam, partner=null]]"

new Person("Eve", new Person("Adam")).equals( woman ) // ==> true
```

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
  - [Triple-double-quoted string](https://groovy-lang.org/syntax.html#_triple_double_quoted_string) from Groovy
  - Multi-line string literals from Scala
  - [Raw string literals](https://kotlinlang.org/docs/reference/basic-types.html#string-literals) from Kotlin

#### Examples

##### Example 1
```java
""" short text """
```

results with:
```
|  Error:
|  illegal text block open delimiter sequence, missing line terminator
|  """ short text """
|      ^
```

##### Example 2

### 361: 	Switch Expressions (Standard)
#### Description
Simplified variant of the switch statement.  
Initially introduced in Java 12.

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
developerRating( 4 ); // ==> "manager"
```

## JVM - general
### 358: 	Helpful NullPointerExceptions
### 352: 	Non-Volatile Mapped Byte Buffers
### 370: 	Foreign-Memory Access API (Incubator)
### 349: 	JFR Event Streaming

## JVM - Garbage Collectors
### 364: 	ZGC on macOS
### 365: 	ZGC on Windows
### 345: 	NUMA-Aware Memory Allocation for G1
### 363: 	Remove the Concurrent Mark Sweep (CMS) Garbage Collector
### 366: 	Deprecate the ParallelScavenge + SerialOld GC Combination

## Tools
### 343: 	Packaging Tool (Incubator)
### 367: 	Remove the Pack200 Tools and API

## Other
### 362: 	Deprecate the Solaris and SPARC Ports

## Other extensions to the Standard Library
