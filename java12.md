# Java 12 - new features 

## JShell usability improvements
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

## 325: Switch Expressions (Preview)

## 326: Raw String Literals (Preview)
Feature removed.  
There are 2 leftovers in `String`:
### transform
### indent

## Other extensions to the standard library
### CompactNumberFormat
### Files.mismatch
### Collectors.teeing

## 334: JVM Constants API

## Prformance improvements
### 341: Default CDS Archives
### 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)
### 344: Abortable Mixed Collections for G1
### Performance comparison: 1.8 vs. 12

## 346: Promptly Return Unused Committed Memory from G1

## 230: Microbenchmark Suite
## 340: One AArch64 Port, Not Two




# Leftovers from version 11
## JMC
