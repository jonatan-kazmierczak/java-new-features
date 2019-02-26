# Java 12 - new features 

Features with their ratings:
* :+1: useful
* :-1: removed / useless / not useful
* :disappointed: disappointing

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

## :disappointed: 325: Switch Expressions (Preview)

## :-1: ~326: Raw String Literals (Preview)~
Feature removed.  
There are 2 leftovers in `String`:
### :-1: transform
How to rewrite this snippet
```java
Integer.parseInt( "65536" )
```
in overcomplicated and overweighted way?

Here you are:
```java
"65536".transform( Integer::parseInt )
```

### :-1: indent
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

## Other extensions to the standard library
### CompactNumberFormat
### Files.mismatch
### Collectors.teeing

## 334: JVM Constants API

## Prformance improvements
### :disappointed: 341: Default CDS Archives
Nothing new - unless you are building custom JDK images.

### :disappointed: 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)
Included in RedHat build.

:disappointed: Not included in the official build from Oracle.

```
$ java -XX:+UseShenandoahGC
Error: VM option 'UseShenandoahGC' is experimental and must be enabled via -XX:+UnlockExperimentalVMOptions.

$ java -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC
Error occurred during initialization of VM
Option -XX:+UseShenandoahGC not supported
```

### 344: Abortable Mixed Collections for G1
### Performance comparison: 1.8 vs. 12

## 346: Promptly Return Unused Committed Memory from G1
Made in :switzerland:

## 230: Microbenchmark Suite
## 340: One AArch64 Port, Not Two

# Leftovers from version 11
## :disappointed: JDK Mission Control 7
:disappointed: Still not available.
