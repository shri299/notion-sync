# Page

Network protcols define the rule, where two system can communicate with each other.

## Application Layer :
1. client-server: HTTP, FTP, SMTP, websockets
  1. client initiates a request and the server responds.
  1. websocket is a little exceptional, it is bi-directional communication.(used in messaging apps)
1. peer to peer : webRTC
  1. client can talk to server and other clients, the server can also talk to multiple clients.

## Transport Layer :
1. TCP/IP
1. UDP/IP(live streaming)



## Resources :
1. https://www.youtube.com/watch?v=JwTiZ9ENquI&list=PL6W8uoQQ2c63W58rpNFDwdrBnq5G3EfT7&index=2
1. https://www.youtube.com/watch?v=aXYleYRa4QU
1. https://www.youtube.com/watch?v=SzwjnoPI--M
1. 

---

# Page

It is a desirable property of a distributed system with replicated data.
c → consistency
a → availability
p → partition tolerence
All three can never be used together.

---

# Page



---

# Page



---

# Page

### Definition:
1. A Stream in Java represents a sequence of elements supporting sequential and parallel aggregate operations. It allows functional-style operations on collections of elements, such as map, filter, reduce, etc.
1. It does not store data; instead, it operates on data from a source (like a Collection, List, Set, Map, or array), processes it, and produces a result.
1. We can also consider stream as a pieline through which elements are passed and openrations are performed on that.
1. Useful when dealing with bulk processing(large data size)(can deal with parellel processing).
1. Introduced in java 8

### Steps:

### Intermediate operations:

### Sequential processing of streams:
```java
List<Integer> salary = Arrays.asList(3,2,1,6,4,4);
        Stream<Integer> salaryStream = salary.stream();

        salaryStream.filter((Integer val) -> val>2)
                .peek((Integer val) -> System.out.println("filtering: "+val))
                .map((Integer val) -> val+10)
                .peek((Integer val) -> System.out.println("mapping: "+val))
                .sorted()
                .peek((Integer val) -> System.out.println("sorted: "+val))
                .collect(Collectors.toList());
```
### Parellel Stream:
1. helps to perform operations on streams concurrently, taking advantage of multicode CPU.
1. prallelStream() method is used instead of stream() method.
1. Internally used forkJoinPool to divide task into subtask, and doing them parellely.


---

# Page

### ✅ volatile
Definition:
The volatile keyword in Java is used to mark a variable such that:
1. Changes to it are always immediately visible to other thread.
1. It prevents caching of the variable value by threads.
Usage:
```java
private volatile boolean isRunning = true;
```
When to use:
- When you have one thread writing, and many threads reading a variable.
- When atomicity isn't required, but visibility is.
- A typical example is flag variables, like stopping a thread.
Limitation:
- Does not guarantee atomicity. For example, counter++ is not safe with just volatile.
Use volatile for status flags, where you're only reading/writing one variable.

```java
class MyTask implements Runnable {
    private volatile boolean isRunning = true;

    public void run() {
        while (isRunning) {
            // do work
        }
    }

    public void stop() {
        isRunning = false;
    }
}
```
Here’s what happens without volatile:
- Thread A is running the loop.
- Thread B sets isRunning = false.
- But Thread A might never see the update, because:
  - The value of isRunning might be cached in CPU registers or local memory (due to compiler/CPU optimization).
  - Without volatile, there’s no happens-before guarantee, so Thread A can just keep looping forever.

“Why wouldn’t other threads immediately see the change?”
## 🔍 The Real Answer: Caching + Compiler/CPU Reordering
Even though the object is in the heap, here’s what happens in reality:
### 1. Thread-local CPU caches
- Modern CPUs and JVMs optimize for speed.
- So when a thread reads a value from memory (e.g., flag), it may cache it in a CPU register or local cache.
- Subsequent reads might just read from the cache — not the heap.
- If another thread updates the flag, the first thread’s cache might still hold the old value.
### 2. Memory Reordering
- The JVM and CPU might reorder instructions for optimization.
- For example, setting flag = false could be reordered or delayed in visibility to other threads.
## 🧨 Why it breaks: No volatile
Without volatile or proper synchronization:
- There is no guarantee that:
  - Writes to flag by Thread A will be flushed to main memory
  - Reads by Thread B will see the fresh value instead of the cached one
So even though both threads technically read/write from the heap, the JVM/CPU optimization layers break that illusion without proper coordination.
## ✅ What volatile Does
Adding volatile tells the JVM:
1. No caching of this variable — always read/write from main memory.
1. Establish a "happens-before" relationship:
  - A write to a volatile variable happens-before a subsequent read of that variable.
So if Thread A writes flag = false, and Thread B reads flag, Thread B is guaranteed to see the updated value.
## 🔁 In Simple Terms
Even with a shared heap:
- ❌ Threads can see stale values due to caching and reordering.
- ✅ volatile disables this caching for that specific variable and ensures memory visibility.

### 🧠 Think of it like this:

## 🧠 What "happens-before" really means (in JMM)
The happens-before relationship defines the visibility guarantee — not literal ordering of every read/write.
✅ It ensures that:
- If Thread A writes to a volatile variable
- And Thread B later reads the same volatile variable
- Then all writes that Thread A did before the volatile write become visible to Thread B after the read
### 🧪 Concrete Example
```java
java
CopyEdit
class SharedData {
    int data = 0;
    volatile boolean ready = false;

    void writer() {
        data = 42;          // (1)
        ready = true;       // (2) volatile write
    }

    void reader() {
        if (ready) {        // (3) volatile read
            System.out.println(data); // (4)
        }
    }
}
```
In this example:
- If Thread A runs writer()
- And Thread B runs reader()
Then:
✅ If ready == true in Thread B (i.e., volatile read at step 3),
  Then data == 42 is guaranteed to be visible in step 4


Yes — 💯 volatile does prevent reordering, and this is one of its most important powers in concurrent programming!
But let's clarify what kind of reordering it prevents — because that's the nuance that really matters.
## ✅ What Reordering Does Volatile Prevent?
### 🔸 The Java Memory Model (JMM) allows:
- Instruction reordering (by JVM or CPU)
- As long as single-threaded semantics are preserved
But in multithreaded programs, this can break things unless we control memory visibility.
### 🔥 volatile prevents:
1. Writes before a volatile write can’t be reordered after the volatile write
1. Reads after a volatile read can’t be reordered before the volatile read
### 🧪 Let’s look at an example:
```java
java
CopyEdit
// Thread A
data = 100;           // (1)
flag = true;          // (2) volatile write

// Thread B
if (flag) {           // (3) volatile read
    System.out.println(data); // (4)
}
```
Without volatile, the JVM/CPU might reorder (1) and (2), meaning flag could be set before data = 100 — and another thread might see flag = true but data = 0 😱
### ✅ With volatile flag:
- The write to data (1) is guaranteed to happen-before the write to flag (2)
- The read of flag (3) happens-before the read of data (4)
🧠 So: If Thread B sees flag == true, then it must also see the updated data = 100

## ✅ Summary

## 👀 The Scenario
```java
java
CopyEdit
// Thread A
data = 100;            // (1)
flag = true;           // (2) volatile write

// Thread B (⚠ does NOT read flag)
System.out.println(data); // (3)
```
## ❓ The Question
If Thread A writes to data and then performs a volatile write to flag,
  but Thread B does NOT read flag,
  will it still see the updated value of data?
## ❌ Answer: NO, there is NO guarantee that Thread B will see data = 100.

---

