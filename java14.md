# Java 14 - new features

## Language syntax
### 305: 	Pattern Matching for instanceof (Preview)
Inspired by [Smart Casts](https://kotlinlang.org/docs/reference/typecasts.html#smart-casts) from Kotlin.

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
Inspired by [Case Classes](https://docs.scala-lang.org/tour/case-classes.html) from Scala, [Data Classes](https://kotlinlang.org/docs/reference/data-classes.html) from Kotlin and [Record Types](https://www.freepascal.org/docs-html/ref/refsu15.html) from Pascal.

#### Example
Definition:
```java
record Person(String name, Person partner) {}
```

Compiled as:
```
```

Representation by Class Visualizer:


Usage:
```java
var man = new Person("Adam", null);
var woman = new Person("Eve", man);
```

### 368: 	Text Blocks (Second Preview)
Inspired by Text Blocks from Scala, Kotlin and Groovy.

### 361: 	Switch Expressions (Standard)
Inspired by [When Expression](https://kotlinlang.org/docs/reference/control-flow.html#when-expression) from Kotlin.

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
