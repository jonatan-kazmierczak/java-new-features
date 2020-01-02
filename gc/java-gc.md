# High performance at low cost - choose the right JVM and the Garbage Collector for your needs

## Abstract
Garbage Collector (GC) is an integral part of every running Java Virtual Machine (JVM) - running in the Cloud, on a dedicated server or on your desktop/laptop.  
Are you aware, what impact has used GC on an application - on its performance, resources consumption and, finally, on its operational cost?  
Which GC will allow you to maximize the performance and minimize the cost of a running Java application?  
This article provides a complete answer to the above questions.  

## What is Garbage Collector?
As explained in [Introduction to Garbage Collection Tuning](https://docs.oracle.com/en/java/javase/13/gctuning/introduction-garbage-collection-tuning.html#GUID-8A443184-7E07-4B71-9777-4F12947C8184):

> The garbage collector (GC) automatically manages the application's dynamic memory allocation requests.  
> A garbage collector performs automatic dynamic memory management through the following operations:  

> - Allocates from and gives back memory to the operating system.
> - Hands out that memory to the application as it requests it.
> - Determines which parts of that memory is still in use by the application.
> - Reclaims the unused memory for reuse by the application.

## Considered Java Virtual Machines and Garbage Collectors
In this article I consider the following Java distributions, Virtual Machines and Garbage Collectors:

### AdoptOpenJDK 13 with HotSpot VM
#### Version
```
openjdk version "13.0.1" 2019-10-15
OpenJDK Runtime Environment AdoptOpenJDK (build 13.0.1+9)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 13.0.1+9, mixed mode, sharing)
```
#### Collectors
- **Epsilon**  
  - So called No-Op collector, which only allocates memory, but does not perform any memory reclamation.
  - Intended to have lowest possible overhead
  - Used as a reference - any collector with better performance or lower CPU consumption, than Epsilon, is the outperforming winner.
  - You can enable it with options: `-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC`
- **Serial**  
  - Default collector on HotSpot VM in environments with limited resources:  
    1 CPU or less than 1792 MB of RAM
  - Intended for small heaps (up to approximately 100 MB).
  - Unlike other collectors, it performs all collections in a single thread.  
  - You can enable it explicitly with option: `-XX:+UseSerialGC`
- **Parallel**  
  - Was a default collector in JDK 8.
  - Intended to provide the best overall performance in a multiprocessor environment.
  - You can enable it with option: `-XX:+UseParallelGC`
- **G1 (Garbage-First)**  
  - Default collector on HotSpot VM (since JDK 9) in environments without restrictions mentioned at **Serial** GC.  
  - Concurrent collector intended to minimize collection pauses in multiprocessor environments with a large amount of memory.
  - You can enable it explicitly with option: `-XX:+UseG1GC`
- **Shenandoah**  
  - Concurrent collector intended to keep very low collection pauses in all applications, regardless of the heap size.
  - Excluded from OpenJDK builds distributed by Oracle.
  - You can enable it with options: `-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC`
- **Z**  
  - Concurrent collector intended for applications which require low latency (less than 10ms pauses) and/or use a very large heap (multi-terabytes).
  - You can enable it with options: `-XX:+UnlockExperimentalVMOptions -XX:+UseZGC`

### AdoptOpenJDK 13 with OpenJ9 VM
#### Version
```
openjdk version "13.0.1" 2019-10-15
OpenJDK Runtime Environment AdoptOpenJDK (build 13.0.1+9)
Eclipse OpenJ9 VM AdoptOpenJDK (build openj9-0.17.0, JRE 13 Linux amd64-64-Bit Compressed References 20191021_96 (JIT enabled, AOT enabled)
OpenJ9   - 77c1cf708
OMR      - 20db4fbc
JCL      - 74a8738189 based on jdk-13.0.1+9)
```
#### Collectors
- **nogc**
  - So called No-Op collector, which only allocates memory, but does not perform any memory reclamation.
  - Intended to have lowest possible overhead
  - Used as a reference - any collector with better performance or lower CPU consumption, than Epsilon, is the outperforming winner.
  - You can enable it with option: `-Xgcpolicy:nogc`
- **gencon**
  - Default collector on OpenJ9 VM.
  - Concurrent collector intended for applications creating many short-lived objects.
  - You can enable it explicitly with option: `-Xgcpolicy:gencon`
- **balanced**
  - Collector intended to handle large heaps (greater than 4 GB).
  - You can enable it with option: `-Xgcpolicy:balanced`
- **optavgpause**
  - Concurrent collector intended to be best suited to short-lived applications and to long-running services involving concurrent sessions that have short lifespans.
  - Its priority are short pause times.
  - You can enable it with option: `-Xgcpolicy:optavgpause`
- **optthruput**
  - Collector intended to be best suited to short-lived applications and to long-running services involving concurrent sessions that have short lifespans.
  - Its priority is high application throughput.
  - You can enable it with option: `-Xgcpolicy:optthruput`
- **metronome**
  - Collector intended for use in applications that require precise response times - i.e. real-time applications.
  - Its priority are very short pause times (3ms).
  - You can enable it with option: `-Xgcpolicy:metronome`

### JDK 11 on Zing VM 19.10 (commercial)
#### Version
```
java version "11.0.4.0.101" 2019-11-06 LTS
Zing Runtime Environment for Java Applications 19.10.1.0+1 (product build 11.0.4.0.101+11-LTS)
Zing 64-Bit Tiered VM 19.10.1.0+1 (product build 11.0.4.0.101-zing_19.10.1.0-b3-product-linux-X86_64, mixed mode)
```
#### Collector
- **C4** - Concurrent collector designed to avoid collection pauses.

### Substrate VM (from GraalVM 19.3 CE)
A simple VM, embedded in a platform-specific executable together with ahead-of-time compiled Java code, produced by `native-image` utility from GraalVM.

#### Version
```
com.oracle.svm.core.VM  GraalVM 19.3.0 CE
```
#### Collector
- Simple no-name collector; in the below tests it is called **SVM** (shortcut for Substrate VM).

## Tests
### Environment used for testing
- CPU: Intel(R) Core(TM) i7-7500U @ 3.50GHz, 2 cores, 4 logical CPUs
- RAM: 16GB (no Swap)
- OS: Ubuntu Linux 18.04 LTS (Linux 4.15.0-72-generic #81-Ubuntu SMP)

### Explanation of used metrics
**Note**: smaller values are better

- **Elapsed Time [s]**  
  Execution time in seconds - it reflects the overall performance.
- **CPU [%]**  
  Average CPU utilization in percents (100% = 1 full core).
- **CPU Time [s]**  
  Execution time in seconds, scaled per 1 core (`Elapsed Time * CPU / 100`).  
  This is the main factor of the operational cost of the application, running on a server or on a desktop.
- **RAM Max [MB]**  
  Maximum used memory in MB.  
- **Cloud Cost [unit * s]**  
  Approximate cost of running the application in a container in the cloud.  
  It is calculated to reflect the operational cost of running the application.  
  For the biggest cloud providers, cost of 1 GB of RAM per second is approximately equal to 0.11 of the cost of 1 vCPU per second.  
  The Cloud Cost is calculated as follows (all variables are rounded up to the next integral value):
  `Elapsed Time [s] * (CPU cores + 0.11 * RAM Max [GB])`

## Test 1 - parallel, CPU- and RAM-intensive request processing application

For this test I've selected a multi-threaded, CPU- and RAM-intensive request processing application - [Request Processor](https://github.com/jonatan-kazmierczak/request-processor).  
Statistics, collected by `collect_gc_stats.sh` script, are presented below.  

### Elapsed Time
Execution time in seconds

![chart](img/uc2-perf.png)

#### Results
- Execution time varies from about 0:09 (HotSpot's G1 and Serial) to almost 3:15 (OpenJ9's balanced).
- **G1** is the leader, following closely by **Serial** and **Shenandoah**.

Let's drill the times down.

### Distribution of Request Processing Times
in seconds

Request processing times are directly related to the application throughput and user experience:  
shorter processing times mean better throughput and happier user.

Request processing times are presented on the following box plot.  
[Here](https://pythontic.com/pandas/dataframe-plotting/box%20and%20whisker%20plot) you can find an explanation of the plot and considered statistics.  
The values (numbers) are also presented below the chart.

![chart](img/uc2-res.png)

```
       G1             Serial         Shenandoah        Parallel     
 Min.   :0.1660   Min.   :0.1830   Min.   :0.1770   Min.   :0.2260  
 1st Qu.:0.1920   1st Qu.:0.1910   1st Qu.:0.1950   1st Qu.:0.2900  
 Median :0.2630   Median :0.2980   Median :0.2990   Median :0.3100  
 Mean   :0.2636   Mean   :0.2703   Mean   :0.2797   Mean   :0.3156  
 3rd Qu.:0.3120   3rd Qu.:0.3305   3rd Qu.:0.3135   3rd Qu.:0.3300  
 Max.   :0.6150   Max.   :0.4250   Max.   :0.7140   Max.   :0.5500  
 
       Z              gencon             C4          optavgpause   
 Min.   :0.2280   Min.   :0.3250   Min.   :0.2080   Min.   :1.131  
 1st Qu.:0.2680   1st Qu.:0.4630   1st Qu.:0.4875   1st Qu.:1.728  
 Median :0.3120   Median :0.5560   Median :0.5400   Median :1.850  
 Mean   :0.3198   Mean   :0.5394   Mean   :0.5723   Mean   :1.893  
 3rd Qu.:0.3455   3rd Qu.:0.6105   3rd Qu.:0.6190   3rd Qu.:2.038  
 Max.   :0.6840   Max.   :0.7600   Max.   :1.0680   Max.   :2.583  
 
      SVM          optthruput      metronome        balanced    
 Min.   :1.700   Min.   :1.579   Min.   :3.022   Min.   :5.211  
 1st Qu.:1.742   1st Qu.:1.982   1st Qu.:3.534   1st Qu.:5.760  
 Median :2.231   Median :2.164   Median :4.208   Median :5.881  
 Mean   :2.040   Mean   :2.150   Mean   :4.114   Mean   :5.880  
 3rd Qu.:2.247   3rd Qu.:2.297   3rd Qu.:4.638   3rd Qu.:6.007  
 Max.   :2.256   Max.   :2.595   Max.   :5.207   Max.   :6.435  
```

#### Results
- Recorded response times vary from 0.166s (shortest) to 6.435s (longest).  
- The top results are achieved by **5 HotSpot's GCs**, following by OpenJ9's **gencon** and Zing's **C4**.  
  Let's take a closer look at them.

![chart](img/uc2-res-hs.png)

#### Best results
- **G1** is the overall winner, **Serial** has the best maximal response time = 0.425s.  
- All **5 HotSpot's GCs** and OpenJ9's **gencon** have maximal response times below 0.8s.

### CPU
Average CPU utilization in percents

![chart](img/uc2-cpu.png)

#### Results
- Average CPU utilization varies from 256% (OpenJ9's balanced) to 323% (OpenJ9's gencon).
- OpenJ9's **balanced** is the winner, following by Zing's **C4**.

### CPU Time
Used CPU time in seconds

![chart](img/uc2-cpu-t.png)

#### Results
- CPU time varies from 0:26 (HotSpot's Serial and G1) to almost 8:19 (OpenJ9's balanced).
- HotSpot's **Serial** is the winner, following closely by **G1** and **Shenandoah**.  
- Another **2 HotSpot's GCs** are following them.  
- Next ones, Zing's **C4** and OpenJ9's **gencon**, used over 2x more CPU time than Serial and G1.

### RAM Max
Maximum RAM usage in MB

![chart](img/uc2-ram.png)

#### Results
- Maximum RAM usage varies from 106 MB (OpenJ9's optthruput) to over 12.7 GB (Zing's C4).
- OpenJ9's optthruput is the leader, following closely by optavgpause, HotSpot's Serial, and balanced.
- Next SVM used over 2x more RAM, than the top 4 listed above.

### Cloud Cost
Approximate cost of running the application in a container in a cloud - in price units * seconds

![chart](img/uc2-cloud.png)

#### Results
- Approximate Cloud cost varies from 29 (HotSpot's G1) to over 606 (OpenJ9's balanced).
- HotSpot's **G1** is the leader, following closely by **Serial** and **Shenandoah**.
- Another **2 HotSpot's GCs** are following them.  
- Next ones, OpenJ9's gencon and Zing's C4, are, respectively, over 2x and 3x more expensive, than G1.

### Recommendations for parallel, CPU- and RAM-intensive request processing application
starting from the most recommended

- JVM: HotSpot
- GC:
  - when the top performance or the low cost of running in a container in the Cloud is most important:  
    G1, Serial, Shenandoah, Parallel, Z
  - when the low cost of running on an on-premise server or a local machine is most important:
    Serial, G1, Shenandoah, Parallel, Z

## Test 2 - single-threaded batch processing application
For this test I've selected a single-threaded batch application - [Words Frequency Calculator](https://github.com/jonatan-kazmierczak/words-frequency-calculator).  
It performs a heavy processing of a big text file.  
Statistics, collected by `collect_gc_stats.sh` script, are presented below.  

### Elapsed Time
Execution time in seconds

![chart](img/uc1-perf.png)

#### Results
- Execution time varies from 2.7s (HotSpot's Parallel) to over 9.6s (OpenJ9's optthruput).
- HotSpot's Parallel is the ultimate winner, outperforming all other collectors - including 2 No-Ops: Epsilon and nogc.
- Parallel is followed by other HotSpot's collectors: Serial, Shenandoah, G1, Z.
- SVM is next, following closely by OpenJ9's gencon.

### CPU
Average CPU utilization in percents

![chart](img/uc1-cpu.png)

#### Results
- Average CPU utilization varies from 99% (SVM) to 234% (Zing's C4).
- SVM is the winner.

### CPU Time
Used CPU time in seconds

![chart](img/uc1-cpu-t.png)

#### Results
- Total CPU time varies from 4.6s (HotSpot's Parallel) to almost 16.2s (Zing's C4).
- HotSpot's Parallel is the ultimate winner, outperforming all other collectors - including 2 No-Ops: Epsilon and nogc.
- HotSpot's Serial is next, following by SVM and Shenandoah.

### RAM Max
Maximum RAM usage in MB

![chart](img/uc1-ram.png)

#### Results
- Max RAM usage varies from 281 MB (OpenJ9's optavgpause) to almost 12.8 GB (Zing's C4).
- Interestingly, No-Op Epsilon didn't use the biggest amount of memory; OpenJ9's metronome, HotSpot's Z and Zing's C4 used even more RAM.

### Cloud Cost
Approximate cost of running the application in a container in a cloud - in price units * seconds

![chart](img/uc1-cloud.png)

#### Results
- Approximate Cloud cost varies from 6.33 (HotSpot's Parallel) to 31 (Zing's C4).
- HotSpot's Parallel is the leader, followed closely by SVM.
- HotSpot's Serial, Shenandoah and Z are next.

### Recommendations for single-threaded batch processing application
starting from the most recommended:

- JVM: HotSpot, Substrate
- GC:
  - when the top performance is most important:  
    Parallel, Serial, Shenandoah, G1, Z
  - when the low cost of running on an on-premise server or a local machine is most important:  
    Parallel, Serial, SVM, Shenandoah
  - when the low cost of running in a container in the Cloud is most important:  
    Parallel, SVM, Serial, Shenandoah, Z

## Final Recommendations
Here you can find the final, summarized list of recommended Java Virtual Machines and Garbage Collectors - starting from the most recommended:

| VM | GC | Parallel, CPU- and RAM-intensive request processing | Single-threaded batch processing | High performance | Low execution cost | Low RAM usage |
| --- | --- | :---: | :---: | :---: | :---: | :---: |
| HotSpot | Serial | + | + | + | + | + |
| HotSpot | Shenandoah | + | + | + | + | - | 
| HotSpot | G1 | + | - | + | + | - |
| HotSpot | Parallel | - | + | + | + | - |
| Substrate | SVM | - | + | + | + | + |
| HotSpot | Z | + | + | + | - | - |

## References
- [HotSpot Virtual Machine Garbage Collection Tuning Guide](https://docs.oracle.com/en/java/javase/13/gctuning/)
- [JEP 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)](http://openjdk.java.net/jeps/189)
- [JEP 318: Epsilon: A No-Op Garbage Collector (Experimental)](http://openjdk.java.net/jeps/318)
- https://www.eclipse.org/openj9/docs/gc/
- https://developer.ibm.com/articles/garbage-collection-tradeoffs-and-tuning-with-openj9/
- https://www.ibm.com/support/knowledgecenter/SSYKE2_8.0.0/com.ibm.java.vm.80.doc/docs/mm_memory_management.html
- https://www.graalvm.org/docs/reference-manual/native-image/
