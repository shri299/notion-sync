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

# Page

## 🔐 What is synchronized?
synchronized is a keyword that ensures:
  - Only one thread at a time can execute a block or method protected by a monitor lock
  - It also ensures visibility: changes made by one thread inside the lock are visible to others after the lock is released
## 🔧 Syntax Variants
### 1. Synchronized Method
```java
java
public synchronized void doWork() {
    // synchronized on `this`
}
```
### 2. Synchronized Block
```java
java
public void doWork() {
    synchronized (this) {
        // critical section
    }
}
```
### 3. Static Synchronized Method
```java
java
public static synchronized void doStaticWork() {
    // synchronized on Class object
}
```
## 🧠 What Does synchronized Guarantee?
## 🔁 How It Works Internally
### Behind the scenes:
- JVM uses monitorenter (when entering synchronized) and monitorexit (when exiting)
- These act as memory fences:
  - Before entering, thread flushes its cache to read fresh values
  - On exit, it flushes writes to main memory
This ensures visibility + ordering, just like volatile — but also mutual exclusion
## 🧪 Classic Example: Thread-safe Counter
```java
public class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++; // atomic now
    }

    public synchronized int getCount() {
        return count;
    }
}
```
Multiple threads can now safely call increment() and getCount() — no lost updates, no stale reads.
## 💥 Happens-Before in synchronized
Java guarantees that:
✅ An unlock (monitorexit) in one thread happens-before a lock (monitorenter) in another thread on the same monitor
So any thread that acquires the lock is guaranteed to:
- See all updates made by previous threads that released the same lock
## 🔄 Comparison with volatile and AtomicInteger
## ⚠️ Downsides of synchronized
## ✅ When to Use synchronized
Use it when:
- You need atomicity + visibility across multiple operations
- You're modifying multiple shared variables
- You need to ensure consistency, not just fast updates


## 👇 Scenario Recap
```java
java
CopyEdit
class MyClass {
    public synchronized void m1() {
        // some work
    }

    public synchronized void m2() {
        // some work
    }
}
```
Now:
```java
java
CopyEdit
MyClass obj = new MyClass();

Thread t1 = new Thread(() -> obj.m1());
Thread t2 = new Thread(() -> obj.m2());
```
## ❓ Will t2 be able to enter m2() while t1 is in m1()?
❌ NO, t2 will have to wait — because both m1() and m2() are synchronized on the same object monitor (obj)
## 🔐 Explanation:
### 🔸 synchronized methods implicitly lock on this (the object instance)
So:
- m1() is equivalent to synchronized (this) { ... }
- m2() is also synchronized (this) { ... }
### 🔸 Only one thread at a time can hold the lock on this
So when:
- t1 enters obj.m1(), it acquires the monitor lock on obj
- t2 then tries to enter obj.m2(), but it also needs the same lock on obj
- So t2 gets blocked until t1 exits m1() and releases the lock

## ✅ How to Allow Parallel Execution?
If you want t1 and t2 to execute simultaneously, you'd need:
1. Remove synchronized from at least one method, OR
1. Synchronize on different locks
Example with different locks:
```java
java
CopyEdit
class MyClass {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void m1() {
        synchronized (lock1) {
            // work
        }
    }

    public void m2() {
        synchronized (lock2) {
            // work
        }
    }
}
```
Now t1 and t2 can execute m1() and m2() independently 🚀

---

# Page

## ✅ What is AtomicInteger?
AtomicInteger is a thread-safe, lock-free class used to perform atomic operations on integers.
It solves problems like:
- Lost updates from count++
- Need for simple atomic operations without using synchronized
## 📦 Package & Class
```java
java
CopyEdit
import java.util.concurrent.atomic.AtomicInteger;
```
It’s part of the java.util.concurrent.atomic package introduced in Java 5.
## ⚙️ How It Works Internally
It uses low-level hardware primitives, specifically the Compare-And-Swap (CAS) instruction:
- If current value == expected value → update it
- Else → try again (retry loop)
No locking → no thread blocking → high performance in multithreaded scenarios.
## 🛠 Common Methods
## 🧪 Example Usage
```java
java
CopyEdit
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicDemo {
    private static AtomicInteger counter = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.incrementAndGet();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.incrementAndGet();
            }
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final counter value: " + counter.get()); // ✅ 2000
    }
}
```
Without AtomicInteger, using count++, this may print less than 2000 due to race conditions.
## 🧠 When to Use AtomicInteger
Use it when:
- You need a simple counter or flag
- You want lock-free, high-performance atomic operations
- You don’t need to perform multiple actions atomically (e.g., if A then B — that needs synchronized)
## ❗ When NOT to Use
- When your logic spans multiple shared variables
- When you need to perform compound actions atomically
- When fairness or ordering between threads matters (use synchronized or ReentrantLock)
## 🔄 compareAndSet Example
```java
java
CopyEdit
AtomicInteger stock = new AtomicInteger(1);

if (stock.compareAndSet(1, 0)) {
    System.out.println("Item purchased");
} else {
    System.out.println("Item already sold");
}
```
This is perfect for scenarios like:
- Distributed locks
- Resource reservation
- Optimistic concurrency control

## 🧠 Yes — AtomicInteger is limited in functionality, but extremely efficient for what it does.
### ✅ When to Use AtomicInteger
- You just need to increment, decrement, set, or compare-and-set a single variable
- You want performance — no thread blocking, just fast retries via CAS
- You're doing very lightweight logic
### ❌ Where AtomicInteger Falls Short
- You can't do:
  - if (x > 5) { x -= 3; } atomically
  - Update two or more variables atomically
  - Do any complex conditional mutation
Example that AtomicInteger can't do safely:
```java
java
CopyEdit
if (counter.get() < 5) {
    counter.incrementAndGet(); // ❌ race condition risk between get() and increment
}
```
This needs a synchronized block because it's a read-then-write pattern with a condition.
### ✅ When synchronized Wins
- You need to perform multiple actions atomically
- You need to coordinate multiple shared variables
- You need blocking behavior or fairness
- You want more general-purpose control
```java
java
CopyEdit
synchronized (lock) {
    if (balance >= amount) {
        balance -= amount;
        history.add("Debited: " + amount);
    }
}
```
This can’t be done with AtomicInteger.
### ⚡ Performance Tradeoff
## ✅ So your statement:
“We could just use synchronized here too, but that would cause locking”
💯 True. You're just choosing between:
- Performance & simplicity (AtomicInteger)
- Power & flexibility (synchronized)
Both have their place — use the right tool for the job.


## ✅ 1. Does AtomicInteger provide atomicity?
✔️ Yes
That’s its main purpose — atomicity of individual operations, like incrementAndGet(), compareAndSet(), getAndAdd(), etc.
These are lock-free atomic instructions done via Compare-And-Swap (CAS) under the hood.
## ✅ 2. Does AtomicInteger provide visibility?
✔️ Yes
Every get(), set(), compareAndSet():
- Reads and writes are done in a way that ensures visibility between threads
- Like volatile, it ensures that:
  - Writes are flushed to main memory
  - Reads are pulled from main memory, not from stale thread-local caches
In fact, internally, the value is stored as a volatile int value — so you get the same visibility semantics as volatile.
## ✅ 3. Does AtomicInteger prevent reordering?
✔️ Yes — for the atomic operations it provides.
Each atomic method (like incrementAndGet, set, get, compareAndSet) includes memory fences/barriers that:
- Prevent reordering around the atomic operation
- Ensure that other memory writes before/after it are properly ordered
It behaves similarly to how volatile inserts LoadLoad / StoreStore / StoreLoad barriers.
## 🔁 4. What about happens-before relationships?
Absolutely!
The Java Memory Model (JMM) defines:
If compareAndSet(expected, update) returns true, then:
  ✅ A happens-before relationship is established between the write (by one thread) and any subsequent read (by another thread) of the updated value.
Similarly:
- A successful set() or incrementAndGet() happens-before any subsequent get() that sees that update.
So yes, happens-before does apply, just like it does for volatile.

---

# Page

Consistent Hashing is a strategy to distribute data across multiple nodes (servers, databases, caches) in a way that minimizes reorganization when nodes are added or removed.
It was introduced to solve the load balancing problem in distributed systems, especially in distributed caching (like Memcached, Redis) or sharded databases.
Let’s say you have a service connected to three database instances. How will you know on which instance to route the request?
1. 1st Method:
  1. Hash the event ID and mod the result with the number of servers.
  1. The problem here, that if you will wish to add a new node or decomission a node, you will have to redistribute the entire data across the nodes.
1. Consistent Hashing:
  1. Hash the event ID and get a number. Place the servers at equal distance at cetain numbers on the hash ring.
  1. If the hash resultant exactly matches any server number route it there.
  1. If not, move the request clocl wise until it reaches a server.
  1. Adding/decomission becomes better(only certain part of the events/rows need to be moved).
  1. A drawback is that data will not be evenly distributed.
  1. 
1. Consistent hashing : Virtual Nodes:
  1. To solve the above drawback we introduce virtual nodes.

What uses consistent hashing?
1. Redis
1. Cassandra
1. DynamoDB
1. Many CDNs

---

# Page

1. Thread pool is a collection of thread(worker threads), which are available to perform the submitted task.
1. Once task completed, worker thread get back to thread pool and wait for for new task to be assigned.
1. Hence they can be reused.
1. Application submit the task in the queue, and once any threads is availbale it picks it from there.

## Advantages of thread pool :
1. Thread creation time can be saved.
  1. Every time we create a new thread, it allocates space, and this takes time.
  1. With thread pool this can be avoided using thread pool.
1. Overhead of managing the thread life-cycle is removed.
  1. With thread pool, we do not have to do this manuaaly, the executer service will do this for you.
1. Increased performance
  1. If you will create new thread manully for every task, there will. be a lot of context switching. This issue is resolved by reusing threads, i.e thread pools.

## Threadpool Executor:
```java
public ThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, 
									TimeUnit unit, BlockingQueue<Runnable> workQueue, 
									ThreadFactory threadFactory, RejectedExecutionHandler handler){}
```
1. Core pool size : Number of thread initially created and kept in the pool even if they are idle.
1. Allow core thread timeout : If this flag is set to true, then we will. check the keep alive time, if any thread are idle for longe than this, it will kill. that thread.
1. Keep alive time : Threads which are idle after this period, get terminated.
1. Max pool size : When all the core thread pools are busy and the queue is also full, then inorder to accept a new task, a new thread is created, only after checking if the max pool size is not breached. If the threashold is breached, we reject the task.
1. Time unit : Tells whether the keep alive time is in sec, ms, mins, or hours
1. Blocking queue : It is queue use to hold task before the get picked by a worker thread.
  1. Bounded Queue : Queue with fixed capacity, like ArrayBlockingQueue.
  1. Unbounded Queue : Queue with no fixed capacity, like LinkedBlockingQueue.
1. Thread factory : Give custom thread name, priority etc.
1. Reject handler : Hnadler for tasks that are rejected. We can add logging here.
  1. AbortPolicy
  1. DiscardPolicy
  1. CallersRunPolicy
  1. DiscardPolicy
  1. DiscardOldestPolicy

## Lifecycle of threadpool executor:
1. Running : Executor is in running state and submit() method is used to add new task.
1. Shutdown : Does not accept any new taks, but let the already running task continue.
1. Stop : Does not accept any new taks, and also teminate the already running tasks.
1. Terminate : End of life for a particular threadpool executor.

Generally the thread pool min max size depend on a lot of factors:
1. CPU cores.
1. JVM Memory.
1. Task nature CPU intensive or IO intensive.
1. High/Low concurrebcy requirement.
1. Memory required tp process a request.
1. Throghput etc.

---

# Page

## Introduction :
1. Every language has its implementation of the hash table, like a hashmap in Java, a dictionary in Python, etc.
1. A hashtable provides constant lookup, insertion, deletion, and updation time.
1. They store key-value pairs.

## Conflict Resolution in Hash Tables :
How can we solve this issue?
1. Chaining
1. Open Addressing
## Chaining :
1. The code idea is to form a chain of keys that hash to the same slot.
1. Put the colliding key in a data structure and link them with each other. (Linked List)
1. Operations to be performed :
1. Implementations
1. Insertion
1. Deletion
1. Lookups
  1. Lookups are similar to delete.
  1. Get the hash key from the hash function.
  1. Linear iterate the linked list to get the value.
  1. We might want to resize the array to optimize lookups in case of high collisions.
  1. In case of high collisions, we can also use a self-balancing BT, which gives a lookup of log(h) (costly insertion and deletion).

## Open Addressing :
1. Instead of using an extra data structure, we can use the empy slots of the hashing array.

---

# Page

# 📝 Notes on Tomcat Threads
## 1. What are Tomcat Threads?
- Tomcat is a servlet container that manages HTTP connections.
- It creates and manages several thread types:
  - Acceptor threads → Accept new TCP connections.
  - Poller threads → Monitor sockets for readiness (in NIO).
  - Worker threads (-exec-*) → Handle actual HTTP requests (execute servlet/filter/controller code).
## 2. Worker Threads (Main Focus)
- Worker threads come from a thread pool owned by Tomcat.
- Each HTTP request → one worker thread (for synchronous code).
- Default pool size: maxThreads = 200.
- Thread names look like: http-nio-8080-exec-1.
- Configurable via Spring Boot:
```plain text
server.tomcat.max-threads=200
server.tomcat.min-spare-threads=10
```
## 3. Queuing & Connection Handling
- Tomcat uses multiple queues/limits:
  - maxConnections → Max concurrent open connections (default ~8192).
  - acceptCount → Queue for new requests when all worker threads are busy (default ~100).
  - OS backlog → Queue at the OS level before Tomcat accepts sockets.
Flow:
1. Client connects → goes to OS backlog.
1. Acceptor thread accepts if maxConnections not exceeded.
1. If worker available → request runs immediately.
1. If all busy → request waits in acceptCount.
1. If acceptCount full → client gets connection refused/timeout.
## 4. Concurrency Model
- With synchronous/blocking code, concurrency = maxThreads (default 200).
- If each request is long-running, threads stay busy and throughput drops.
- Extra requests wait in acceptCount.
## 5. Interaction with Spring Boot
- Tomcat threads handle request → run your controller/service code.
- If you call an @Async method:
  - Tomcat thread can return quickly.
  - Async work moves to a separate executor pool (not Tomcat’s pool, unless explicitly wired).
- This separation prevents Tomcat threads from being blocked by long background jobs.
## 6. Tuning Guidelines
- CPU-bound apps → keep maxThreads close to available CPU cores × 2–4.
- I/O-bound apps → allow higher maxThreads (since threads wait often).
- Always monitor metrics (busy threads, request latency, queue depth).
- Consider:
```plain text
server.tomcat.max-threads=500
server.tomcat.accept-count=1000
server.tomcat.max-connections=10000
```
  (values depend on workload and hardware).
## 7. Key Takeaways
- Tomcat threads = request handling threads in the Tomcat worker pool.
- Max concurrency (for sync code) = maxThreads.
- Beyond that → acceptCount queue.
- If queue full → connections refused.
- @Async and custom ThreadPoolExecutors are separate pools unless explicitly shared.
- Use async/non-blocking (WebFlux, DeferredResult, CompletableFuture) to scale beyond maxThreads.


# Who creates Tomcat threads?
Tomcat (specifically its Connector / ProtocolHandler) creates and manages several kinds of threads:
- Acceptor threads
  - Names like: http-nio-8080-Acceptor-0
  - Purpose: accept new TCP connections from the OS listen socket.
- Poller (selector) threads (for NIO/NIO2)
  - Names like: http-nio-8080-Poller-1
  - Purpose: watch many open sockets for readiness (read/write) without blocking.
- Worker (executor) threads
  - Names like: http-nio-8080-exec-1
  - Purpose: actually run your servlet/filter/controller code for a request.
  - Pool size controlled by maxThreads (default ~200).
Spring Boot (with embedded Tomcat) boots Tomcat and passes it your config, but Tomcat itself owns these threads.
# Where’s the queue?
There are two relevant queues on the request path:
1. OS listen backlog (outside Tomcat)
  - Before Tomcat even accepts the socket, the OS maintains a backlog for incoming SYNs/accepted sockets waiting for accept().
  - Size is OS-tuned; Tomcat’s acceptCount interacts with this (see #2).
1. Tomcat Connector “acceptCount” queue (inside Tomcat)
  - When all worker threads are busy (i.e., you’ve hit maxThreads), newly accepted connections wait here until a worker is free.
  - Controlled by acceptCount (default often 100).
  - If this queue is full, further connections are rejected (clients may see connection refused/timeout).
Also relevant:
- maxConnections
  - Caps the number of concurrent open connections Tomcat will keep (active + queued).
  - If exceeded, Tomcat stops accepting more until below the limit.
- minSpareThreads
  - Tomcat tries to keep at least this many idle worker threads ready.
# Putting it together (flow)
1. Client connects → OS backlog (briefly if needed).
1. Acceptor takes the connection if maxConnections not exceeded.
1. If a worker is free, request goes straight to a worker thread (-exec-*).
1. If all workers are busy, the connection waits in acceptCount queue.
1. If acceptCount is also full → connection refused/timeout.

---

# Page



---

# Page



---

# Page

## 1. Tomcat Threads
- Tomcat is a servlet container.
- When you run a Spring Boot application (with embedded Tomcat), Tomcat creates a thread pool to handle HTTP requests.
- Each incoming request is handled by one of these Tomcat worker threads.
- By default, this is a fixed-size pool (maxThreads=200 by default, configurable in application.properties via server.tomcat.max-threads).
- These threads are typically named like http-nio-8080-exec-1.
👉 So, Tomcat threads = request handling threads created and managed by the Tomcat web server.
## 2. Spring Boot Threads
- Spring Boot itself doesn’t create its own "special threads."
- Instead, it relies on:
  - The servlet container (Tomcat/Jetty/Undertow) for request threads.
  - Spring’s Task Executors for async operations (@Async, @Scheduled, etc.).
- For example, if you annotate a method with @Async, Spring uses a TaskExecutor (backed by a ThreadPoolExecutor) to run it on a separate thread, not a Tomcat request thread.
👉 So, Spring Boot threads are usually just threads coming from either Tomcat (for requests) or a custom executor (for async tasks).
## 3. ThreadPoolExecutor Threads (Java’s ExecutorService)
- A ThreadPoolExecutor is a Java utility that manages a pool of worker threads.
- You can create one manually or configure it via Spring Boot’s TaskExecutor.
- For example:
```java
@Bean
public Executor taskExecutor() {
    return Executors.newFixedThreadPool(10);
}
```
- If you use this executor, tasks will run on its worker threads, completely independent from Tomcat’s request-handling threads.
👉 So, ThreadPoolExecutor threads are general-purpose worker threads you (or Spring) define for background/parallel work.

- A client sends an HTTP request → handled by a Tomcat thread.
- Inside the controller, you call a method annotated with @Async → that spawns work on a ThreadPoolExecutor thread (Spring’s task executor).
- The Tomcat thread is free to return the HTTP response (if async), while the background task continues separately.

# 📝 What are Spring Boot Threads?
- Spring Boot doesn’t invent a new type of thread.
- They are just normal Java threads under the hood, managed via:
  - Tomcat worker threads → handle HTTP requests.
  - TaskExecutor threads (@Async) → backed by a ThreadPoolExecutor.
  - TaskScheduler threads (@Scheduled) → backed by a ScheduledThreadPoolExecutor.
# 📝 Do they use the same resources as Java worker threads?
✅ Yes — Spring Boot threads are Java threads.
- They come from the same JVM thread system and consume the same OS resources (stack memory, CPU, scheduling by the OS).
- A Tomcat thread, a Spring @Async thread, or a raw ThreadPoolExecutor thread are all just Java threads.
🚫 No automatic sharing of pools/queues:
- Tomcat threads belong to Tomcat’s pool.
- @Async threads belong to Spring’s executor.
- A raw ThreadPoolExecutor you create has its own pool.
- They are separate unless you explicitly configure them to use the same executor.

Exactly ✅ — you’ve understood it correctly.
- Spring Boot has no special kind of thread of its own.
- The threads you see running in a Spring Boot app always come from one of these sources:
  1. Tomcat (or Jetty/Undertow) worker threads → handle HTTP requests.
  1. Spring-managed executors (e.g., @Async, @Scheduled) → which are just wrappers around Java’s ThreadPoolExecutor / ScheduledThreadPoolExecutor.
  1. Any thread pools you create manually (raw ThreadPoolExecutor, Executors.newFixedThreadPool, etc.).
So:
- No “Spring Boot threads” in a unique sense — they are just plain Java threads.
- The difference is only who manages the pool (Tomcat, Spring, or you).
- All of them use the same JVM + OS-level thread resources (stack memory, CPU scheduling, etc.).
👉 In other words: Spring Boot doesn’t invent a new threading model. It just organizes and manages plain Java threads for common application tasks.

---

# Page

### We know that synchronized block is aquired on an object. So if two different threads acquire two different objects, both can go inside the block at the same time.

### You want know matter what, only one thread should execute the critical section:
We use custom locks in such cases, they do not depend on the object.
1. Reentrant Lock : (Refer github for example)
  1. Use a reentrant object and keeps it common. Even if you use multiple objects of class, you will be able to put only one lock at a time.
1. ReadWrite Lock : (Refer github for example)
  1. It has read lock(shared lock) and write lock(exclusive lock).
  1. Multiple thread can have shared locks, and read the data or whatever.
  1. You cannot put write lock if an read lock already exists on it.
  1. If you put write lock first, you can put niether read nor write, until the lock is relased.
  1. Generally used when read is very high. Since multiple reading locks can be acquired, there will not be an issue.
1. Stamped Lock. : (Refer github for example)
  1. Supports readwrite lock functionality and occ functionality(version check).
  1. When putting lock it return a stamp, and when unlocking you have to pass that stamp.
  1. That stamp basically stores the read version state.
1. Semaphore Lock. : (Refer github for example)
  1. We pass a permit argument when creating the lock object.
  1. It basically signifies the number of lock permitted inside the critical section at a time.

1. awaity() (used for above 4) → wait() (used for synchronized)
1. signal() (used for above 4) → notify() (used for synchronized)

---

# Page



---

# Page



---

# Page

Future :
1. Interface which represents the result of async task(thread).
1. Means it allows to check if :
  1. computation is complete.
  1. get the result.
  1. take care of any exception etc.
1. Study various methods of future interface (done, get, cancel etc)



---

# Page

1. Monolithic application overloads the IDE, since the file size is large
1. Very tightly coupled
1. even if you wish to scale for one API, you’ll have to scale the entire service.

1. divide a big application into small services
1. scailing is easy
1. loosely coupled

1. hops may increase.
1. integration overhead
1. transaction management

decompositon(buisness capability, subdomain), database(common, different), communication(via api, via events), integration

strangler pattern : refacotring the code from monolithic to microservice pattern. moving the traffic on a api slowly from monolithic service to microservice. you can never move 100 percent traffic in one go.

---

# Page

## Collection cheat sheet:


---

# Page

Thread-local storage in Java provides a way to create variables that are local to each thread, meaning each thread has its own independent copy of the variable. This is implemented through the ThreadLocal class.
## How ThreadLocal Works
When you create a ThreadLocal variable, each thread that accesses it gets its own separate instance of that variable. Threads cannot see or modify each other's copies, making it useful for maintaining thread-specific data without synchronization.
## Basic Usage
```java
java
// Create a ThreadLocal variable
ThreadLocal<String> threadLocalString = new ThreadLocal<>();

// Set value for current thread
threadLocalString.set("Hello from " + Thread.currentThread().getName());

// Get value for current thread
String value = threadLocalString.get();
```
## Common Patterns
With Initial Value:
```java
java
ThreadLocal<Integer> threadLocalInt = ThreadLocal.withInitial(() -> 0);

// Or using the older approach
ThreadLocal<Integer> threadLocalInt = new ThreadLocal<Integer>() {
    @Override
    protected Integer initialValue() {
        return 0;
    }
};
```
Typical Use Case - Database Connections:
```java
java
public class DatabaseManager {
    private static final ThreadLocal<Connection> connectionHolder =
        new ThreadLocal<>();

    public static Connection getConnection() {
        Connection conn = connectionHolder.get();
        if (conn == null) {
            conn = createNewConnection();
            connectionHolder.set(conn);
        }
        return conn;
    }

    public static void closeConnection() {
        Connection conn = connectionHolder.get();
        if (conn != null) {
            conn.close();
            connectionHolder.remove();// Important!
        }
    }
}
```
## Important Considerations
Memory Leaks: Always call remove() when done with a ThreadLocal, especially in application servers where threads are reused. The thread holds a reference to the ThreadLocal value, which can prevent garbage collection.
Inheritance: Regular ThreadLocal values aren't inherited by child threads. Use InheritableThreadLocal if you need child threads to inherit the parent's value.
Performance: ThreadLocal access is generally fast, but creating many ThreadLocal instances can impact performance.
## When to Use ThreadLocal
ThreadLocal is particularly useful for storing context information that needs to be available throughout a request or operation without passing it as parameters everywhere. Common examples include user authentication details, transaction contexts, or request-specific configuration.
The key benefit is avoiding the complexity of synchronization while still maintaining thread safety through isolation.




Virtual threads and platform threads represent Java's evolution in concurrency models, with virtual threads being a major addition in Java 19+ to address scalability limitations of traditional platform threads.
## Platform Threads (Traditional Java Threads)
Platform threads are what we've been using since Java's beginning. They're heavyweight threads that map 1:1 to operating system threads.
```java
java
// Creating a platform thread
Thread platformThread = new Thread(() -> {
    System.out.println("Running on platform thread: " +
                      Thread.currentThread());
});
platformThread.start();

// Or using Thread.ofPlatform() (Java 19+)
Thread platformThread2 = Thread.ofPlatform()
    .name("my-platform-thread")
    .start(() -> {
        System.out.println("Platform thread task");
    });
```
Platform Thread Characteristics:
- Each thread consumes ~2MB of stack memory
- Limited by OS thread limits (typically thousands, not millions)
- Expensive to create and context switch
- When they block (I/O, sleep), the OS thread blocks too
## Virtual Threads (Project Loom)
Virtual threads are lightweight threads managed by the JVM rather than the OS. They're designed for high-concurrency applications.
```java
java
// Creating virtual threads (Java 19+)
Thread virtualThread = Thread.ofVirtual()
    .name("my-virtual-thread")
    .start(() -> {
        System.out.println("Running on virtual thread: " +
                          Thread.currentThread());
    });

// Or using factory
ThreadFactory factory = Thread.ofVirtual().factory();
Thread vt = factory.newThread(() -> {
    System.out.println("Virtual thread task");
});
vt.start();
```
Virtual Thread Characteristics:
- Extremely lightweight (~few KB of memory each)
- Can create millions of them
- Managed by JVM, not OS
- When they block, they're "parked" and the carrier thread can run other virtual threads
## Key Differences Illustrated
```java
java
// Platform threads - limited scalability
public class PlatformThreadExample {
    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();

// Creating 1000 platform threads
        for (int i = 0; i < 1000; i++) {
            Thread.ofPlatform().start(() -> {
                try {
                    Thread.sleep(1000);// Simulates I/O
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        System.out.println("Created 1000 platform threads in: " +
                          (System.currentTimeMillis() - start) + "ms");
    }
}

// Virtual threads - massive scalability
public class VirtualThreadExample {
    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();

// Creating 100,000 virtual threads!
        for (int i = 0; i < 100_000; i++) {
            Thread.ofVirtual().start(() -> {
                try {
                    Thread.sleep(1000);// When virtual thread sleeps,// carrier thread can run others
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        System.out.println("Created 100K virtual threads in: " +
                          (System.currentTimeMillis() - start) + "ms");
    }
}
```
## Virtual Thread Architecture
Virtual threads run on a small pool of platform threads called "carrier threads":
```plain text
Virtual Threads:    [VT1] [VT2] [VT3] [VT4] [VT5] [VT6] ...
                      |     |     |     |     |     |
Carrier Threads:   [PT1]  [PT2]  [PT1]  [PT3]  [PT2]  [PT1]
                      |     |     |     |     |     |
OS Threads:        [OS1]  [OS2]  [OS3]  [OS4]  [OS2]  [OS1]
```
When a virtual thread blocks:
1. It's unmounted from its carrier thread
1. The carrier thread picks up another virtual thread
1. When the blocked operation completes, the virtual thread is remounted (possibly on a different carrier)
## Practical Usage
Web Server Example:
```java
java
// Traditional approach with platform threads
public class TraditionalWebServer {
    private final ExecutorService executor =
        Executors.newFixedThreadPool(200);// Limited!

    public void handleRequest(Request request) {
        executor.submit(() -> {
// Handle request - blocks on database/network calls
            processRequest(request);
        });
    }
}

// With virtual threads
public class VirtualThreadWebServer {
    private final ExecutorService executor =
        Executors.newVirtualThreadPerTaskExecutor();// Unlimited!

    public void handleRequest(Request request) {
        executor.submit(() -> {
// Same code, but can handle millions of concurrent requests
            processRequest(request);
        });
    }
}
```
## When to Use Each
Use Virtual Threads for:
- I/O-intensive applications (web servers, microservices)
- High-concurrency scenarios
- Applications with many blocking operations
- When you need thousands/millions of concurrent tasks
Use Platform Threads for:
- CPU-intensive tasks
- When you need thread-local storage that persists
- Legacy code that depends on platform thread behavior
- When using thread pools with careful sizing
## Limitations and Considerations
Virtual Thread Limitations:
- Don't use large thread pools with virtual threads (defeats the purpose)
- Avoid synchronized blocks (can pin virtual threads to carriers)
- Some native code interactions may not work as expected


Good question! This refers to the fact that operating systems have built-in limits on how many threads a single process can create, and these limits are much lower than what modern applications often need.
## OS Thread Limits in Practice
Typical Limits:
- Linux: Usually 1000-4000 threads per process (depending on available memory)
- Windows: Similar range, often around 2048 threads per process
- macOS: Typically around 2000-8000 threads
These aren't arbitrary numbers - they're constrained by several factors:
## Why These Limits Exist
1. Memory Consumption:
```java
java
// Each platform thread consumes significant memory
Thread thread = new Thread(() -> {
// This thread gets ~2MB of stack space by default// Plus additional overhead for thread control blocks
});

// With 1000 threads = ~2GB just for stack space!// With 10,000 threads = ~20GB (often exceeds available RAM)
```
2. Kernel Resources:
Each OS thread requires kernel data structures:
- Thread control blocks
- Scheduling information
- Context switching overhead
3. Address Space Limitations:
On 32-bit systems, the virtual address space is limited, making it impossible to allocate stack space for too many threads.

The OS thread limits were a major bottleneck for highly concurrent applications, which is exactly why virtual threads were introduced - to break free from these OS-imposed constraints while maintaining the familiar threading programming model.

You're absolutely right! Web servers like Tomcat do have their own configured thread limits, and these are typically set much lower than the OS maximum for good reasons.
## Tomcat's Thread Configuration
Tomcat has configurable thread pools with default limits:
```xml
xml
<!-- server.xml configuration -->
<Connector port="8080"
           protocol="HTTP/1.1"
           maxThreads="200"          <!-- Default: 200 threads -->
           minSpareThreads="10"<!-- Minimum idle threads -->
           maxSpareThreads="50"<!-- Maximum idle threads -->
           acceptCount="100"<!-- Queue size when all threads busy -->
           connectionTimeout="20000" />
```
Default Tomcat Settings:
- maxThreads: 200 (can handle 200 concurrent requests)
- acceptCount: 100 (additional requests wait in queue)
- If all 200 threads are busy AND queue is full → new requests get rejected
## Why Not Use OS Maximum?
Even though your OS might allow 2000+ threads, Tomcat defaults to 200 because:
1. Resource Management:
```java
java
// Each thread consumes memory even when idle// 200 threads ≈ 400MB of stack space// 2000 threads ≈ 4GB of stack space
```
2. Performance Degradation:
Too many threads cause excessive context switching:
```java
java
// Context switching overhead increases with thread count// Sweet spot is usually around number of CPU cores × 2-4
int optimalThreads = Runtime.getRuntime().availableProcessors() * 2;
```

---

# Page

JWT (JSON Web Token) is a compact, secure, and URL-safe way of transmitting information between two parties — typically a client and a server — as a digitally signed JSON object.
It’s commonly used for authentication and authorization in modern web applications.
Where to use JWT:
Used for AUTHENTICATING (confirming the user identity)
Used for AUTHORIZATION (checks the user permission before providing access to resources)
Used for SSO (Single Sign On) i.e. Authenticate once and access multiple applications.

### 🔹 Structure of a JWT
A JWT consists of three parts, separated by dots (.):
```plain text
xxxxx.yyyyy.zzzzz
```
Each part is Base64URL-encoded:
1. Header – contains metadata about the token
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
  - alg: signing algorithm (e.g., HS256 for HMAC-SHA256)
  - typ: token type (JWT)
1. Payload – contains user data or claims
```json
{
  "sub": "1234567890",
  "name": "Srishti Srivastava",
  "role": "admin",
  "exp": 1736034560
}
```
  - Claims can be registered (iss, exp, sub), public, or custom
1. Signature – ensures integrity and authenticity
  Created by encoding the header and payload and signing them:
```plain text
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```
### 🔹 How JWT Works (in authentication)
1. User login:
  The user logs in with credentials (e.g., email + password).
1. Server verifies credentials:
  If valid, the server creates a JWT containing user info and signs it with a secret key.
1. Client stores JWT:
  The client (browser or app) stores the token (often in localStorage or a cookie).
1. Subsequent requests:
  The client sends the JWT in the Authorization header:
```plain text
Authorization: Bearer <jwt_token>
```
1. Server verifies token:
  The server validates the token’s signature and checks expiry before granting access.
### 🔹 Advantages
✅ Stateless: No need to store session info on the server.
✅ Compact: URL-safe and small enough to send in headers.
✅ Cross-platform: Works with web, mobile, and APIs easily.
✅ Self-contained: Carries user data within itself.
### 🔹 Example JWT
Header:
```json
{ "alg": "HS256", "typ": "JWT" }
```
Payload:
```json
{ "userId": 101, "email": "user@example.com", "role": "user" }
```
Encoded token:
```plain text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJ1c2VySWQiOjEwMSwiZW1haWwiOiJ1c2VyQGV4YW1wbGUuY29tIiwicm9sZSI6InVzZXIifQ
.0rj8jZB_3T4RHeXoyzHAdb9_y2fxfLp7tkgOyLQ9sKM
```


### What was popular before JWT and what was the problem with it?
Before JWT (JSON Web Tokens) became popular, most web applications used session-based authentication — often with server-side sessions and cookies.
Let’s break this down clearly 👇
### 🧩 How it worked
1. A user logs in with credentials.
1. The server validates them and creates a session (a record in memory or a database) containing user info.
1. The server sends back a session ID, which the client stores in a cookie.
1. On every subsequent request, the browser automatically sends the cookie (with the session ID) back to the server.
1. The server looks up the session ID in its store to identify the user.
### 🧠 Example
## 🔸 Problems with Session-Based Auth
### 1. Server State Dependency
- The server must store every active session in memory or a database.
- As your user base grows, maintaining sessions becomes heavy and hard to scale.
- In distributed systems (multiple servers), you must share session state across all instances — often using Redis or sticky sessions.
➡️ JWT solves this by being stateless — no need to store anything on the server.
### 2. Scaling Issues
- When you deploy multiple servers (load balancing), each server must have access to the same session data.
- This requires session replication or a centralized session store (like Redis), which increases complexity.
➡️ With JWT, every request carries the user’s data itself, so any server can validate it independently.
### 3. Cross-Domain / API Usage
- Session cookies work well in browsers but not easily in mobile apps or APIs.
- Sharing cookies between subdomains or third-party clients can be tricky.
➡️ JWT works well with APIs, mobile clients, and cross-domain requests using headers.
### 4. Serialization Overhead
- Session objects can be complex (nested user roles, permissions, etc.), requiring serialization/deserialization each time.
➡️ JWT stores data as JSON, lightweight and already structured.
### 5. Logout and Revocation
- JWTs are stateless, but that’s a trade-off:
  - Easier scaling.
  - Harder to revoke tokens early (e.g., force logout before expiry).
(Session-based systems can easily destroy a session record on logout.)

---

# Page



---

# Page

In Java, the final keyword is used to declare constants or prevent modification. It can be applied to variables, methods, and classes, and its behavior differs based on the context:(you cannot make the reference in stack point to some other obj in heap, always point to same obj, you can modify things inside that obj only)
### 1. final Variable:
- Once assigned, the value cannot be changed (i.e., it becomes a constant).
```java
java
CopyEdit
final int x = 10;
x = 20; // ❌ Compilation error
```
- For objects: the reference can't be changed, but the object's internal state can be modified.
```java
java
CopyEdit
final List<String> list = new ArrayList<>();
list.add("Hi"); // ✅ Allowed
list = new ArrayList<>(); // ❌ Error
```
### 2. final Method:
- Prevents method overriding in subclasses.
```java
java
CopyEdit
class Parent {
    final void show() {
        System.out.println("Can't override me");
    }
}
class Child extends Parent {
    void show() { } // ❌ Compilation error
}
```
### 3. final Class:
- The class cannot be extended/inherited.
```java
java
CopyEdit
final class Animal { }
class Dog extends Animal { } // ❌ Error
```
### 🔹 static final — What does it mean?
- *static**: belongs to the class, not an instance.
- *final**: the value cannot change after assignment.
So static final defines a constant value that:
- Is shared across all instances (because it’s static) : whats the point of  having a constant value in all objects of class, better to keep it as a class variable
- Can’t be modified (because it’s final), if it is obj the contents of it is still mutable, though cannot be reassigned

---

# Page



---

# Page

1. Out of order execution : JVM reorder command for certain optimizations.
1. Field visibility
1. Happens Before relationship

JMM is a specification which gaurentees visibility of fields amidst reordering of instructions.

### Types of memory :
Both stack and heap are created by JVM and stored in RAM.
1. Stack Memory :
  1. All temporary variable and seperate memeory block are sotred in the stack.
  1. Stores primitive data type.
  1. Stroes reference to an object present in heap.
  1. Each thread has its own stack memory, but common heap memory.
  1. As soon as a variable becomes out of scope, it is deleted in LIFO order.
  1. When stack is full → stack overflow error.
1. Heap Memory :
  1. Store object.
  1. Garbage collector sweeps unreferenced object in the heap.
  1. The GC runs periodically, jvm controls this.
  1. The GC used mark and sweep algo.
  1. Heap memory is divided into two parts :
    1. Young generation : Minor GC, cleanup runs frequently.
    1. Old generation : Major GC, cleanup not that frequent.
    1. Young gen is divided into → eden, s0 , s1
    1. Once the age of a object crosses a certain threshold, it is marked as old.
1. Non Heap :
  1. Stores class data, static, final and constants.

### Versions of GC :
1. Serial GC : Only one GC thread runs(slow, since gc is expensive, the app may slow down).
1. Parellel GC(default in java 8) : Multiple GC threads.
1. Concurrent Mark and Sweep : Application threads do not get impacted because of GC threads(Not 100 percent true still. some impact).
1. GI garbage collectore : A little than CMS.

### Type of reference :
1. Strong : GC cannot remove reference from the heap, until there is strong reference pointing from the stack.
1. Weak : GC reamove weak reference objects.
1. Soft : Only removed when very urgent or memory issue.

## Chat GPT notes :
1. You write .java file ➜ Human-readable source code
1. javac compiles it ➜ Creates .class file with bytecode
1. JVM loads the .class ➜ Verifies and links it
1. Execution Engine runs bytecode ➜ Interpreted or JIT compiled

---

# Page

## ✅ 1. S - Single Responsibility Principle (SRP)
Definition:
A class should have only one reason to change, i.e., it should only do one thing.
### 💡 Example:
```java
// BAD: One class handling multiple responsibilities
public class User {
    public void saveToDatabase() { /*...*/ }
    public void sendEmail() { /*...*/ }
}
```
```java
// GOOD: Separate responsibilities into different classes
public class User {
    // only user data and behavior
}

public class UserRepository {
    public void saveToDatabase(User user) { /*...*/ }
}

public class EmailService {
    public void sendEmail(User user) { /*...*/ }
}
```
## ✅ 2. O - Open/Closed Principle (OCP)
Definition:
Software entities (classes, modules, functions) should be open for extension, but closed for modification.
### 💡 Example:
```java
// BAD: You need to change the code every time you add a new shape
public class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            return Math.PI * ((Circle) shape).radius * ((Circle) shape).radius;
        } else if (shape instanceof Rectangle) {
            return ((Rectangle) shape).height * ((Rectangle) shape).width;
        }
        return 0;
    }
}
```
```java
// GOOD: Use abstraction to extend behavior
interface Shape {
    double area();
}

class Circle implements Shape {
    double radius;
    public Circle(double radius) { this.radius = radius; }
    public double area() { return Math.PI * radius * radius; }
}

class Rectangle implements Shape {
    double height, width;
    public Rectangle(double height, double width) {
        this.height = height;
        this.width = width;
    }
    public double area() { return height * width; }
}

class AreaCalculator {
    public double calculateArea(Shape shape) {
        return shape.area();
    }
}
```
## ✅ Liskov Substitution Principle (LSP) – Final Notes
### 🔹 Definition:
Subtypes must be substitutable for their base types.
  That is, if S is a subclass of T, then objects of type T can be replaced with objects of type S without breaking the behavior of the program.
### 🔹 Why It Matters:
LSP ensures that inheritance is used correctly. A subclass shouldn’t override behavior in a way that violates expectations set by the parent class.
### 🔹 ❌ Violating LSP – Example:
```java
class Bird {
    void fly() { System.out.println("Flying"); }
}

class Ostrich extends Bird {
    @Override
    void fly() {
        throw new UnsupportedOperationException("Can't fly!");
    }
}

Bird b = new Ostrich();
b.fly(); // ❌ Runtime failure → Violates LSP
```
### 🔹 ✅ Correct Approach – Redesign Abstraction
```java
interface Bird {
    void eat();
}

interface FlyingBird extends Bird {
    void fly();
}

class Sparrow implements FlyingBird {
    public void eat() {}
    public void fly() {}
}

class Ostrich implements Bird {
    public void eat() {}
}
```
- ✅ Now Ostrich is not forced to fly, so no contract is violated.
- ✅ You can safely pass either Bird or FlyingBird wherever required, and behavior remains valid.
### 🔹 Reference vs Object in Memory (Parent/Child Objects)
### ✅ Allowed:
```java
Animal a = new Dog(); // ✅ Parent reference → Child object (Polymorphism)
a.speak();            // Dog's overridden method is called
```
- You can only call methods defined in the parent class/interface
- Child methods are hidden unless downcasted:
```java
((Dog) a).wagTail(); // ✅ But only if a is actually a Dog
```
### ❌ Not Allowed:
```java
Dog d = new Animal(); // ❌ Compilation Error
```
- You cannot assign a parent object to a child reference because the parent does not have child-specific behavior.
### 🔹 Summary Table

## ✅ 4. I - Interface Segregation Principle (ISP)
Definition:
Clients should not be forced to depend on interfaces they do not use.
### 💡 Example:
```java
// BAD: Fat interface
interface Worker {
    void work();
    void eat();
}

class Robot implements Worker {
    public void work() {}
    public void eat() {} // But robot doesn’t eat! Violates ISP
}
```
```java
// GOOD: Split into smaller interfaces
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

class Human implements Workable, Eatable {
    public void work() {}
    public void eat() {}
}

class Robot implements Workable {
    public void work() {}
}
```
## ✅ 5. D - Dependency Inversion Principle (DIP)
Definition:
High-level modules should not depend on low-level modules. Both should depend on abstractions.
### 💡 Example:
```java
// BAD: High-level class depends directly on low-level class
class LightBulb {
    public void turnOn() {}
}

class Switch {
    private LightBulb bulb;

    public Switch(LightBulb bulb) {
        this.bulb = bulb;
    }

    public void operate() {
        bulb.turnOn();
    }
}
```
```java
// GOOD: Depend on abstraction
interface Switchable {
    void turnOn();
}

class LightBulb implements Switchable {
    public void turnOn() {
        System.out.println("Bulb turned on");
    }
}

class Switch {
    private Switchable device;

    public Switch(Switchable device) {
        this.device = device;
    }

    public void operate() {
        device.turnOn();
    }
}
```
## 

---

# Page



---

# Page



---

# Page

### Command design pattern :
1. It is a behavioural design pattern.
1. It separates the logic of invoker, command , reciever.

---

# Page

1. Refer notes for diagram and API
1. Refer code for schema and stuff.

## Follow up Questions :
### ✅ How to Handle 2-3 Day Delayed Transactions in LLD
### 1. Transaction Life cycle with Status States
Model a transaction as a state machine:
```plain text
nginx
CopyEdit
PENDING → PROCESSING → SETTLEMENT_INITIATED → SETTLED / FAILED
```
```java
enum TransactionStatus {
    PENDING,
    PROCESSING,
    SETTLEMENT_INITIATED,
    SETTLED,
    FAILED,
    TIMED_OUT
}
```
### 2. Designing the Transaction Table
```sql
Transaction (
    id UUID PRIMARY KEY,
    from_user_id UUID,
    to_user_id UUID,
    amount DECIMAL,
    method ENUM('NEFT', 'UPI', 'CARD', ...),
    status ENUM,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    settlement_attempts INT,
    failure_reason TEXT
)
```
### 
### 3. Event-Driven Architecture for Settlements
Use an event queue (e.g., Kafka/SQS) to asynchronously handle delayed flows:
- On initiation: push event TransactionInitiated.
- A NEFTProcessor picks the message and contacts the external bank API.
- Store transaction in a "pending settlement" state.
- Schedule retry mechanism or poll for status updates.
### 6. Timeouts and Fallbacks
- If a transaction remains in PENDING or PROCESSING beyond a certain threshold (e.g., 3 days), move it to TIMED_OUT or FAILED.
- Inform the user and provide refund (manual or automated depending on business logic).
### 🧩 Example Flow
1. u1 initiates NEFT payment to u2.
1. System records PENDING transaction.
1. Background worker pushes it to NEFT gateway → mark as SETTLEMENT_INITIATED.
1. Periodic job or callback updates it to SETTLED or FAILED.
1. Notify both users.

---

# Page

## 1. Shallow Copy
A shallow copy creates a new object, but does not clone the nested objects inside it — instead, it copies their references.
### Key Points
- The top-level object is duplicated.
- Any mutable nested objects (arrays, lists, maps, other objects) are shared between the copy and the original.
- Changes in nested objects affect both copies.
- Fast, but can lead to unexpected side effects.
Java Example:
```java
java
CopyEdit
import java.util.*;

class User {
    String name;
    List<String> skills;

    User(String name, List<String> skills) {
        this.name = name;
        this.skills = skills;
    }
}

public class Main {
    public static void main(String[] args) {
        List<String> skills = new ArrayList<>(Arrays.asList("Java", "Spring"));
        User u1 = new User("Srishti", skills);

        // Shallow copy (only top-level copy)
        User u2 = new User(u1.name, u1.skills);

        u2.skills.add("Elasticsearch");

        System.out.println(u1.skills); // [Java, Spring, Elasticsearch] — modified!
    }
}
```
## 2. Deep Copy
A deep copy creates a new object and recursively copies all nested objects so that the new object is completely independent.
### Key Points
- Everything is duplicated — the top-level object and all objects it refers to.
- No shared references.
- Slower and more memory-consuming than shallow copy, but safer.
- Often implemented via:
  - Copy constructors
  - Serialization/Deserialization
  - Manual recursion
Java Example:
```java
java
CopyEdit
class User {
    String name;
    List<String> skills;

    User(String name, List<String> skills) {
        this.name = name;
        this.skills = new ArrayList<>(skills); // new list = deep copy
    }
}

public class Main {
    public static void main(String[] args) {
        User u1 = new User("Srishti", Arrays.asList("Java", "Spring"));
        User u2 = new User(u1.name, u1.skills);

        u2.skills.add("Elasticsearch");

        System.out.println(u1.skills); // [Java, Spring] — unchanged!
    }
}
```
## 3. Other Copy Types
### a) Lazy Copy (a.k.a. Copy-on-Write)
- Initially, shallow copy is made (sharing the same data).
- When either copy is modified, then a deep copy of the affected part is made.
- Used in immutable collections, strings in some languages, or CopyOnWriteArrayList in Java.
Benefit: Saves memory until a write occurs.
### b) Clone (Java clone() method)
- Java’s built-in cloning via Object.clone().
- By default, it does shallow copy.
- Can be overridden for deep copy.
- Requires implementing Cloneable.
### c) Serialization Copy
- Serialize the object to a byte stream and deserialize it back to create a full deep copy.
- Slower but easy for complex object graphs.
### Comparison Table

---

# Page

## 🔍 What is a Race Condition?
A race condition happens when two or more threads access a shared resource at the same time, and the result depends on the timing of their execution.
Livelock, Starvation, Fairness

Disruptor Pattern / High-Performance Concurrency
- Ring buffer concepts (used in trading systems, Kafka-like designs)

Practical Concurrency Patterns
- Producer-consumer, reader-writer, dining philosophers
- Real-world usage in backend systems (queue processing, async pipelines)

---

# Page

## 1. What is ConcurrentHashMap?
ConcurrentHashMap is a thread-safe variant of HashMap introduced in Java 5.
It allows safe concurrent access by multiple threads without external synchronization.
Unlike Collections.synchronizedMap() (which uses a single global lock),
ConcurrentHashMap uses fine-grained locking to achieve high performance and concurrency.
## 2. Why it exists
- A normal HashMap is not thread-safe — concurrent modifications can cause data corruption or infinite loops.
- A synchronized map (Collections.synchronizedMap()) is thread-safe but slow — every operation locks the entire map.
- ConcurrentHashMap gives thread-safety + high concurrency by locking only small parts (buckets/segments) instead of the whole map.
## 3. Internal Working (Simplified)
## 4. Thread Safety Rules
## 5. The Core Confusion — Atomic vs Thread-safe
✅ Each individual method (get, put, remove) is thread-safe.
❌ But a sequence of operations (like get + modify + put) is not atomic.
Example:
```java
map.put("A", map.get("A") + 1);
```
This looks simple but is not safe, because:
1. Thread T1 does get("A") → 10
1. Thread T2 does get("A") → 10
1. Both compute +1 → 11
1. Both do put("A", 11)
  ➡️ Result: One update lost (should’ve been 12)
Reason: get() and put() are two independent atomic operations, not a single transaction.
## 6. When is it Safe?
✅ Safe: Single atomic method calls
- put(key, val)
- get(key)
- remove(key)
- putIfAbsent()
- replace()
❌ Unsafe: Combined operations
- if (map.get(k) == null) map.put(k, val);
- map.put(k, map.get(k) + 1);
Because between the two calls, another thread can modify the same key.
## 7. Why AtomicInteger is Used as Value
### Example:
```java
ConcurrentHashMap<String, AtomicInteger> voteCount = new ConcurrentHashMap<>();
voteCount.get(candidateId).incrementAndGet();
```
✅ Each candidate’s votes are stored in an AtomicInteger, which guarantees atomic read-modify-write using CAS (Compare-And-Swap).
If we used:
```java
ConcurrentHashMap<String, Integer> voteCount;
voteCount.put(id, voteCount.get(id) + 1);
```
it’s not thread-safe because get and put are separate.
### So:
## 8. Alternative: Using Map Atomic Methods (Java 8+)
If you don’t want to use AtomicInteger, you can use:
```java
voteCount.merge(candidateId, 1, Integer::sum);
```
or
```java
voteCount.compute(candidateId, (k, v) -> v == null ? 1 : v + 1);
```
✅ These perform the “read + modify + write” under internal lock for that key — fully atomic.
⚙️ Slightly slower than AtomicInteger.incrementAndGet() when updates are frequent.
## 9. Locking Behavior (Deep Explanation)
### When you call put():
- It locks only the bucket corresponding to that key.
- The lock is released immediately after that write.
- Other buckets can be modified simultaneously by other threads.
### When you call get():
- It does not acquire any lock.
- Uses volatile read — fast and non-blocking.
So yes — your understanding was exactly right:
“Once the get() completes, it releases any internal lock (actually none for read).
  Then when put() happens, it locks again —
  between those two calls, another thread can slip in.”
## 10. Common Use Cases
## 11. Example Summary
### ❌ Unsafe Increment:
```java
map.put("A", map.get("A") + 1);
```
→ Race condition, lost updates.
### ✅ Safe (using AtomicInteger):
```java
map.computeIfAbsent("A", k -> new AtomicInteger(0)).incrementAndGet();
```
### ✅ Safe (using atomic map method):
```java
map.merge("A", 1, Integer::sum);
```
## 12. Analogy
Imagine ConcurrentHashMap as a building of lockers:
- Each locker = one key
- Each locker has its own mini-lock (fine-grained locking)
- get() → peek through glass → no lock
- put() → open just one locker → lock only that one
- get + put → open, think, close → another person can sneak in between (race!)
## 13. Key Takeaways
✅ Each individual operation (get, put, remove) is thread-safe.
⚠️ A sequence of operations is not atomic — race conditions can occur.
✅ Use AtomicInteger as value when frequent updates happen.
✅ Or use merge() / compute() for atomic map updates.
✅ No need to synchronize externally — ConcurrentHashMap handles internal locking.
✅ Reads are non-blocking; writes lock only per-key segments.
✅ Perfect for high-concurrency systems (like counters, voting, caching).
## 14. TL;DR Summary Table

---

# Page



---

# Page

### What is DB Locking?
DB locking ensures that no other transactions update the locked rows.
### What is a shared lock?
1. It is a read lock, aquired for reading.
1. Let’s say a transactions has a shred lock a row, and another transaction ties to update it,it won;t allow.
1. But other transaction can put a shared lock on same row and read it.
### What is exclusice lock?
1. If a trans has exc lock on a row, no other trans can either read or write.

Below are the three problems we usually see when there is no isolation:
### Dirty Read problem :
A dirty read is a situation in database systems where a transaction reads data that has been written by another transaction but not yet committed.
### 🧾 Explanation:
In a dirty read, one transaction (let’s say T1) modifies a record, and another transaction (T2) reads that modified record before T1 commits or rolls back. If T1 later rolls back (i.e., undoes its changes), then T2 has read data that never really existed in the database — this is the "dirty" part.
### 🔁 Example:
1. T1 starts and updates a row:
  UPDATE accounts SET balance = 1000 WHERE id = 1;
  (But it doesn't commit yet)
1. T2 starts and reads the balance:
  SELECT balance FROM accounts WHERE id = 1; → gets 1000 (dirty read)
1. T1 rolls back due to some error:
  The update is undone, and the balance is back to the old value, say 500.
Now T2 has read a balance that never actually existed in a consistent state.
### 🛡️ How to prevent dirty reads:
Use higher isolation levels like:
- READ COMMITTED (prevents dirty reads)
- REPEATABLE READ
- SERIALIZABLE (most strict)
Dirty reads are only possible under the lowest isolation level, which is READ UNCOMMITTED.




### Non Repeatable Reads :
If T1 reads the same row several times, there is a change that it might get different values at each read, as some other T2 might have written and committed a different value in the row.
### 🔄 What is a Non-Repeatable Read?
A non-repeatable read occurs when a transaction reads the same row twice and gets different values because another transaction modified and committed changes to that row in between the two reads.
### 🧾 Detailed Explanation:
- A transaction (T1) reads a row.
- Another transaction (T2) updates or deletes that same row and commits the change.
- T1 reads the row again and sees different data.
This breaks the expectation that if you read the same data twice within a transaction, it should stay the same — hence, non-repeatable.
### 🧪 Example:
1. T1 starts
  SELECT balance FROM accounts WHERE id = 1; → gets 500
1. T2 starts
  UPDATE accounts SET balance = 1000 WHERE id = 1;
  COMMIT
1. T1 reads again
  SELECT balance FROM accounts WHERE id = 1; → gets 1000
❗ Now T1 has read the same row twice and got two different values.
### 🔐 How to Prevent:
- READ COMMITTED isolation level does not prevent this.
- Use REPEATABLE READ or SERIALIZABLE isolation level to prevent non-repeatable reads:
  - REPEATABLE READ ensures that rows read once cannot change during the transaction.




### Phantom reads:
If transaction A returns the same query several times, then there is chance that number of rwos returned are different several times.
### 👻 What is a Phantom Read?
A phantom read happens when a transaction executes the same query twice and gets different sets of rows because another transaction inserted or deleted rows that match the query condition in the meantime.
The "phantom" is the new row that "appears" (or disappears) in the second read — it wasn’t there the first time.
### 🧪 Example:
1. T1 starts a transaction
  SELECT * FROM orders WHERE amount > 100; → returns 5 rows
1. T2 inserts a new matching row
  INSERT INTO orders (id, amount) VALUES (10, 200);
  COMMIT
1. T1 runs the same query again
  SELECT * FROM orders WHERE amount > 100; → now returns 6 rows
✅ All rows are committed, so no dirty read.
🔁 But T1 sees extra rows (phantoms) that didn’t exist earlier in the same transaction.
### 🔐 How to Prevent:
- REPEATABLE READ stops dirty + non-repeatable reads, but not phantom reads.
- Only SERIALIZABLE isolation level prevents phantom reads.
  - It locks the entire range of values, not just individual rows.

## How to resolve the above problems? → ISOLATION LEVELS
## READ UNCOMMITTED:
No locking while readind and writing. Here all above three problems can happen
Read Uncommitted is the lowest isolation level in database systems.
It allows a transaction to read data that has been modified by other transactions but not yet committed — also known as a dirty read.
⚠️ You're allowed to "peek" at data that might never exist permanently.
### ✅ What It Allows:
- Dirty reads
- Non-repeatable reads
- Phantom reads
### ❌ What It Prevents:
- Almost nothing — it's extremely permissive and unsafe for critical systems.
### 🔧 When to Use (rare cases):
- For read-only or non-critical analytics where performance matters more than correctness.
- When stale or temporary values are acceptable.



### READ COMMITTED:
Read : Shared lock is aquiread and released as soon as the read is done.
Write : Exclusive lock aquired and keep till the end of the transaction.
### 📖 What is Read Committed Isolation Level?
Read Committed is a database isolation level where a transaction can only read data that has been committed by other transactions.
In simpler terms:
🛑 You cannot read uncommitted/dirty data from other transactions.
  ✅ But you can see changes if other transactions commit between your reads.
### 🧪 Example:
Let’s say we have two transactions, T1 and T2.
1. T1 starts and updates a row:
  UPDATE accounts SET balance = 1000 WHERE id = 1;
  (T1 does not commit yet)

---

# Page

### 📌 Scenario Recap:
- ₹100 is debited from Account A.
- ₹100 is not credited to Account B.
- No rollback occurred.
- All technical constraints (PKs, FKs, data types, CHECKs) are satisfied.
- So the data is valid structurally — but logically broken.
### ✅ What Properties Are Affected?
### 🔎 Deeper View
- ✅ Consistency is satisfied from the ACID perspective:
  - All database rules (foreign keys, checks, etc.) are followed.
  - The database is in a valid state, just not the right one.
- ❌ Integrity is violated:
  - You've lost money — this violates business rules, even if SQL constraints are okay.
  - This is what makes data integrity broader than consistency.
### 💡 Real-World Analogy:
Imagine you’re balancing a ledger:
- You subtract ₹100 from one account.
- You forget to add it to another.
- The math doesn’t add up anymore, even though your page is neat and has no typos.
It’s structurally clean, but logically corrupted.
### ✅ Final Summary:


You're absolutely right — data integrity is not explicitly listed in the ACID acronym.
However... here's the key point:
🔍 Data integrity is a broader concept, while ACID properties are a set of guarantees that help enforce data integrity in databases.
### ✅ So, let's clarify the relationship:
### 🧠 Think of it like this:
- Data Integrity = Health of the data (is it trustworthy?)
- ACID = How the system protects that health during operations
### 🚦 Example Mapping:
Let’s see how ACID contributes to integrity:
So even though integrity isn't part of the acronym, it’s what ACID protects.
### 🎯 Final Thought:
ACID is the means.
Data integrity is the end.
Let me know if you'd like a visual diagram or real-world system example to solidify this!

Just like integrity, the term reliability is not part of ACID, but...
✅ ACID properties enable reliability — especially in systems where correctness, fault tolerance, and recoverability matter.
### 📌 What is Reliability in a system?
A system is reliable if it consistently performs as expected — even in the presence of:
- Failures (e.g., power loss, crash)
- Concurrent users
- Network glitches

---

# Page

# Optimistic concurrency control:
Isolation level used to achieve this repeatable read.
Very high concurrency.

---

# Page

## What is a transaction in DB?
Transaction helps us to achieve INTEGRITY. It helps us to avoid inconsistency in pur database.

A transaction in a database is a sequence of one or more operations (such as read, write, update, delete) that are treated as a single logical unit of work. The primary goal of a transaction is to ensure data integrity and consistency, especially in the presence of failures or concurrent access.
### 🔑 Key Characteristics (ACID Properties):
A valid transaction must satisfy ACID properties:
1. Atomicity
  - All or nothing: Either all operations in the transaction are completed successfully, or none are.
  - If something fails, the database is rolled back to its previous consistent state.
1. Consistency
  - A transaction takes the database from one valid state to another valid state, maintaining data integrity constraints (e.g., primary keys, foreign keys).
1. Isolation
  - Transactions are independent of each other. The intermediate state of one transaction is invisible to others.
  - Prevents issues like dirty reads, non-repeatable reads, and phantom reads (handled via isolation levels).
1. Durability
  - Once a transaction is committed, its changes are permanent, even if the system crashes immediately afterward.
### ✅ Example (Bank Transfer)
Transferring ₹1000 from Account A to Account B:
```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 1000 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 1000 WHERE account_id = 'B';

COMMIT;
```
If something fails between the two updates, a ROLLBACK will undo the first operation to keep the data consistent.

The meaning of consistency below is only for traditional databases, not the CAP theoram on
- Consistency → Your database rules are followed after every transaction.
- Data Integrity → Your data is correct and makes sense at all times.

### Transactions are most commonly associated with:
### ✅ Relational Databases (like MySQL, PostgreSQL, Oracle, SQL Server)
- Transactions are built-in, with full ACID guarantees.
- You use SQL keywords like BEGIN, COMMIT, ROLLBACK.
- Designed for strong data integrity and consistency.
### 💡 Analogy:
- Relational DB = strict accountant who double-checks every number and never breaks the rules (ACID).
- NoSQL DB = fast-moving clerk who may skip some checks for speed — unless you explicitly ask for more guarantees.

---

# Page

If your use case requires strong data reliability, with no room for:
- Incomplete transactions
- Lost updates due to concurrency
- Data inconsistency or corruption
Then SQL (relational databases) like PostgreSQL, MySQL, Oracle, or SQL Server are the right choice. Here's why:
### ✅ Why Choose SQL (Relational DBs) in Such Cases?
1. Built-in ACID Transactions
  - Full support for Atomicity, Consistency, Isolation, and Durability.
  - You don’t need to write complex logic to handle failure cases.
1. Strong Concurrency Control
  - Uses locks, MVCC (multi-version concurrency control), and isolation levels to avoid race conditions and data anomalies (like dirty reads, lost updates).
1. Reliable Rollbacks
  - If part of a transaction fails (e.g., debit succeeds but credit fails), the whole operation rolls back automatically.
1. Enforced Constraints
  - Foreign keys, unique constraints, and checks ensure data integrity without manual validation.

### Go for SQL when:
- You're building a banking system, payment gateway, inventory control, or any financial or critical data app.
- Your app must never lose or corrupt data — even under concurrent users or crashes.
### 📝 Real-World Analogy
- SQL is like a trained accountant — double-checks everything, logs every move, won't let anything slip.
- NoSQL is like a fast-moving assistant — very quick, but you have to trust or verify more yourself.

---

# Page

## ✅ What are Enums?
In Java (and many other languages), enum (short for enumeration) is a special data type that enables a variable to be a set of predefined constants.
### Syntax:
```java
public enum Status {
    OPEN, IN_PROGRESS, CLOSED;
}
```
Here, Status can only have one of these three values.
## ✅ Why Use Enums Instead of Strings?
### 1. Type Safety
- If you use Strings for something like status, it's easy to mistype:
```java
String status = "In_Progress"; // typo
```
- But with enums:
```java
Status status = Status.IN_PROGRESS; // compiler-enforced
```
### 2. Code Completion and Readability
- IDEs can suggest enum constants automatically.
- Improves readability and reduces magic strings scattered across code.
### 3. Avoid Runtime Errors
- Typos in strings lead to runtime bugs.
- Enums prevent this at compile-time.
### 4. Scoped Namespace
- Enum values belong to the enum type. You can’t mix Status.OPEN with some other enum's values.
- Prevents pollution of the global namespace (unlike constant strings).
### 5. Can Have Methods
- Enums can contain methods, constructors, and fields.
- Example:
```java
java
CopyEdit
public enum Status {
    OPEN("Open case"),
    IN_PROGRESS("Work ongoing"),
    CLOSED("Case closed");

    private final String description;

    Status(String desc) {
        this.description = desc;
    }

    public String getDescription() {
        return description;
    }
}
```
### 6. Switch-Case Friendly
```java
java
CopyEdit
switch (status) {
    case OPEN: // do something
        break;
    case IN_PROGRESS:
        break;
    case CLOSED:
        break;
}
```
## ✅ Use Case in Objects with Limited Known Values
Whenever you have an object field that can only take a fixed, well-known set of values, enums are preferred:
```java
java
CopyEdit
class Ticket {
    private Status status; // ✅ not String
}
```
This gives you all the advantages mentioned above—validation, safety, clarity, and maintainability.
## Summary: When to Use Enums
Use enums when:
- Values are fixed and known ahead of time
- You want to avoid string-based bugs
- You care about clean, maintainable, and type-safe code



## 🔧 JVM-Level Optimization for Enums
### ✅ 1. Singleton Instances
Each enum constant is implemented as a public static final singleton instance, created only once per classloader. This means:
- Memory-efficient: Only one object per enum constant.
- Fast comparisons: You can use == instead of .equals().
Example:
```java
java
CopyEdit
Status a = Status.OPEN;
Status b = Status.OPEN;

System.out.println(a == b); // true ✅
```
So instead of allocating memory for a new string every time (like "OPEN"), enums are shared objects.
### ✅ 2. Efficient Switching
When using enums in a switch statement, Java compiles this into a tableswitch or lookupswitch bytecode. This is much faster than doing string comparisons.
So this:
```java
java
CopyEdit
switch (status) {
    case OPEN:
        // ...
}
```
is compiled into optimized integer-based branching, not string comparison.
## 🧠 Where Are Enums Stored?
Enums are stored like regular Java classes with some special behavior. Here's how:
### ✅ 1. Enum Class Bytecode
Every enum is compiled into a .class file like any other class, but:
- It extends java.lang.Enum<T>
- Enum constants are compiled into static final fields
Example:
```java
public enum Status {
    OPEN, CLOSED;
}
```
Is compiled roughly like:
```java
public final class Status extends Enum<Status> {
    public static final Status OPEN = new Status("OPEN", 0);
    public static final Status CLOSED = new Status("CLOSED", 1);
    private static final Status[] VALUES = {OPEN, CLOSED};
}
```
### ✅ 2. Stored in Method Area / Metaspace
- In modern JVMs (post-Java 8), class metadata (including enum constants) are stored in the Metaspace (native memory).
- The actual static instances (like Status.OPEN) reside in the Heap, as they are objects.
- Constants are created at class initialization (<clinit>).
## 🧪 Final Performance & Optimization Points

---

# Page

Partitioned locking (also known as lock striping) is a concurrency control technique used to reduce contention and improve scalability in multi-threaded environments, especially in data structures like ConcurrentHashMap.
### 🔒 What is the problem?
In a naive approach, if a single global lock is used to guard a shared structure (like a map), then only one thread can operate on it at a time — even if multiple threads want to access different parts of the map. This causes:
- Reduced parallelism
- Performance bottlenecks
### 🧩 Partitioned Locking: The Solution
Instead of using one big lock, the structure is divided into smaller independent partitions, and each partition is guarded by its own lock.
- In ConcurrentHashMap, the map is internally divided into segments or buckets.
- Each segment/bucket can be locked independently.
- Threads accessing different segments can work in parallel without blocking each other.
### ✅ Example (Conceptually)
Let’s say you have 16 segments (buckets):
- key1.hashCode() % 16 = 3 → goes to segment 3
- key2.hashCode() % 16 = 7 → goes to segment 7
If two threads try to update key1 and key2 at the same time:
- They acquire locks on different segments (3 and 7),
- So they don’t block each other ✅
### 🔁 Benefits
- Greatly increases throughput
- Lock contention is minimized
- Operations like put, get, remove are thread-safe without blocking the whole map

---

# Page

### 1. ✅ Security
Many Java classes use Strings for sensitive data:
- File paths
- Network URLs
- Database usernames/passwords
If String were mutable, someone could change the path or credentials after validation, causing serious security risks.
### 2. ✅ Thread Safety
Since String objects can't change, they are automatically thread-safe — multiple threads can safely share and use the same String instance without synchronization.
### 3. ✅ HashCode Caching
String is heavily used as a key in HashMap. Since it’s immutable:
- The hash code never changes,
- It can safely cache its hashCode, improving performance.
### 4. ✅ Class Loading & String Pool
Java maintains a String pool:
```java
java
CopyEdit
String a = "hello";
String b = "hello";  // a and b point to same object!
```
Because Strings don’t change, the JVM can reuse them safely — saving memory and speeding up comparisons.
## 🛠️ How is it implemented?
- The String class has a final character array:
```java
java
CopyEdit
private final char value[];
```
- Being final, it cannot be reassigned.
- Also, String itself is marked final, so it cannot be subclassed and modified.
## 🔁 So what happens when you "modify" a string?
You’re not modifying the string — you’re creating a new one.
```java
java
CopyEdit
String s = "hello";
s = s + " world";  // creates a new String object "hello world"
```
## ✅ Summary
## ✅ Is String thread-safe in Java?
Yes, String is thread-safe — because it is immutable.
### 🔐 Why?
Since a String object cannot be changed after it's created, there is no risk of one thread modifying it while another reads it.
You can safely:
- Share the same String between threads
- Read from it in multiple threads
- Pass it around freely
### ⚠️ But...
If you’re modifying a reference to a String, the reference itself is not thread-safe — just like with any object reference.
```java
java
CopyEdit
String s = "abc";

// This assignment is not thread-safe:
s = s + "d";  // creates a new String
```
Multiple threads changing the reference to s can cause race conditions — but that's about thread-safety of the variable, not the String object itself.
## ✅ Summary:













## 🧵 What is StringBuffer?
StringBuffer is a mutable sequence of characters (like StringBuilder) that is also thread-safe because its methods are synchronized.
## 🔍 Key Properties of StringBuffer






## 🚀 What is StringBuilder?
StringBuilder is a mutable class for creating and modifying strings efficiently.
It’s not thread-safe, but that’s exactly why it’s faster than StringBuffer in most single-threaded use cases.
## 🔍 Key Properties
## ⚙️ Performance
- String ➝ slow in loops (creates new object every time)
- StringBuffer ➝ synchronized = thread-safe = slower
- ✅ StringBuilder ➝ no synchronization = fastest

## ✅ Why is StringBuilder faster than String?
Because of immutability vs mutability:
### 🔴 String is immutable:
Every modification creates a new object in memory.
```java
java
CopyEdit
String s = "A";
s += "B";  // Creates a new "AB" String object
s += "C";  // Creates a new "ABC" String object
```
- Each time you append, Java must:
  - Allocate a new char[]
  - Copy all existing characters
  - Add the new character(s)
- This is O(n) for each append ➝ O(n²) total in loops
### 🟢 StringBuilder is mutable:
It uses an internal char[] buffer that grows automatically, and modifications are done in-place.
```java
java
CopyEdit
StringBuilder sb = new StringBuilder("A");
sb.append("B");  // modifies internal array
sb.append("C");  // still no object creation
```
- It avoids creating new objects on each append

---

# Page

Contention in computing refers to a situation where multiple threads or processes try to access the same shared resource at the same time, and only one can succeed at a time, forcing the others to wait.
### 🔁 In Multithreading:
Contention happens when threads compete for:
- CPU cores
- Locks (e.g. synchronized blocks)
- Memory or cache
- Disk I/O
- Network resources
### 🔒 In Context of Locks:
When two or more threads try to acquire the same lock, but only one can succeed, the others must:
- Wait (blocked),
- Retry (in a spinlock),
- Or get scheduled out.
This reduces performance and causes thread contention.
### 🔧 Example:
```java
java
CopyEdit
synchronized void update() {
   // only one thread can execute this at a time
}
```
If 10 threads try to call update() simultaneously, only one can proceed. The other 9 must wait ⇒ high contention.
### 🧠 Analogy:
Imagine 10 people trying to get through a single narrow door at once — only one can go through, and the rest form a queue. That’s contention.
### 🚩 Effects of High Contention:
- Increased waiting time
- Reduced parallelism
- Lower CPU utilization
- Thread starvation or deadlock in extreme cases

---

# Page

## 🔹 1. What is Exception Handling?
Exception Handling is a mechanism to handle runtime errors so that the normal flow of the application can be maintained.
- Exception: An event that disrupts the normal flow of the program.
- Throwable: The superclass for all errors and exceptions.

## 🔹 2. Exception Hierarchy
```plain text
Object
                  |
              Throwable
              /        \
         Error        Exception
                         /      \
                    Checked      Unchecked (RuntimeException)
```
### ✅ Checked Exceptions
- Known at compile-time.
- Must be either caught or declared.
- Examples: IOException, SQLException, ParseException
### ❌ Unchecked Exceptions
- Known at runtime.
- Not required to handle explicitly.
- Examples: NullPointerException, ArrayIndexOutOfBoundsException, IllegalArgumentException
### 🔥 Error
- Serious issues beyond application control.
- Example: OutOfMemoryError, StackOverflowError
## 🔹 3. Keywords
## 🔹 4. Flow of Exception Handling
```java
java
CopyEdit
try {
    // code that may throw
} catch (ExceptionType name) {
    // handler
} finally {
    // cleanup code
}
```
### Internal Flow:
1. If exception occurs in try → JVM looks for matching catch block.
1. If match found → that catch block executes.
1. If not → exception propagates up the call stack.
1. finally executes regardless of what happened in try/catch.
1. If still not handled → JVM terminates the program and prints stack trace.

## 🔹 6. Custom Exception
```java
java
CopyEdit
class MyException extends Exception {
    public MyException(String msg) {
        super(msg);
    }
}
```
Throwing:
```java
java
CopyEdit
throw new MyException("Something went wrong");
```


















## 🔹 9. Try-with-Resources (Java 7+)
For handling resources like file streams:
```java
java
CopyEdit
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    return br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
```
- Automatically closes resources (implements AutoCloseable).
## The Problem: What’s the issue with manual resource handling?
In Java, certain resources (like files, streams, DB connections) must be closed manually after you're done using them. If you forget to close them, you get:
- Memory leaks
- File locks
- Too many open connections
### 🔻 Traditional Way (before Java 7):
```java
java
CopyEdit
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("file.txt"));
    String line = br.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        if (br != null)
            br.close();  // ✅ YOU must remember to close this
    } catch (IOException ex) {
        ex.printStackTrace();
    }
}
```
Problems here:
- Verbose
- Easy to forget closing
- Needs nested try-catch in finally
## 🔹 Solution: try-with-resources (Java 7+)
### ✅ It lets you declare and use the resource inside try(), and auto-closes it at the end — no need for manual close.
```java
java
CopyEdit
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
}
// br.close() is automatically called here!
```
No need for finally, no memory leak, less code.
## 🔸 How does it work internally?
The object inside try() must implement the interface:
```java
java
CopyEdit
java.lang.AutoCloseable
```
### So when your code finishes the try block:
- JVM calls close() on the resource automatically — even if an exception occurred.
### ✅ Examples of AutoCloseable resources:
## 🔸 Real Analogy
Imagine:
You walk into a library and borrow a book (open resource).
  With manual code: YOU must remember to return the book (close it).
  With try-with-resources: The system automatically returns the book after you’re done reading.
## 🔸 Can I use multiple resources?
Yes!
```java
java
CopyEdit
try (
    BufferedReader br = new BufferedReader(new FileReader("file.txt"));
    PrintWriter pw = new PrintWriter("output.txt")
) {
    String line = br.readLine();
    pw.println(line);
}
```
Both br and pw are automatically closed in reverse order of declaration.
## 🔸 Can I use it with my own classes?
Yes! Just implement AutoCloseable:
```java
java
CopyEdit
class MyResource implements AutoCloseable {
    public void doSomething() {
        System.out.println("Working...");
    }

    @Override
    public void close() {
        System.out.println("Cleaning up!");
    }
}
```
Then:
```java
java
CopyEdit
try (MyResource r = new MyResource()) {
    r.doSomething();
}
// prints: Cleaning up!
```
## 🔚 Summary

---

# Page

Immutability means once an object is created, its state (data) cannot be changed. In the context of collections in Java, it means:
✅ You cannot add, remove, or update elements in the collection after it is created.
### 🔍 Example
```java
// Immutable
List<String> immList = List.of("a", "b");
immList.add("c"); // ❌ Throws UnsupportedOperationException
immList = new ArrayList<>(); //Throws exception
//cannot assign to mutable object but can be assigned to immutable

// Unmodifiable (but original list is modifiable)
List<String> base = new ArrayList<>();
base.add("a");
List<String> unmod = Collections.unmodifiableList(base);
base.add("b"); // ✅ unmod list also sees the change!
//why unmod also see change?
//Collections.unmodifiableList() does not create a new independent immutable copy — it creates a read-only view of the original list.
//So any changes to the original list (base) are reflected in unmod, even though you can't mutate it through unmod.
```
### 🛠️ Why Use Immutable Collections?
1. Thread safety – no need for synchronization
1. Readability – ensures no accidental modification
### ✅ Java ways to create immutable collections:
```java
List<String> list = List.of("A", "B");
Set<Integer> set = Set.of(1, 2, 3);
Map<String, Integer> map = Map.of("a", 1, "b", 2);
```

### 🔍 Question:
If I assign an immutable list to another list reference, can I modify the new list?
The answer is:
🟥 No — if you just assign it, both variables point to the same immutable list, and modifications will throw an exception.
### 💡 Example:
```java
java
CopyEdit
List<String> immutableList = List.of("A", "B", "C");  // Immutable list
List<String> newList = immutableList;                 // Just another reference

newList.add("D");  // ❌ Throws UnsupportedOperationException
```
Both immutableList and newList point to the same immutable object in memory. So, newList is not modifiable either.
### ✅ If you want to copy it into a modifiable list:
```java
List<String> immutableList = List.of("A", "B", "C");
// Create a modifiable list from it
List<String> modifiableList = new ArrayList<>(immutableList); //we are a creating a whole new obj here in the heap
modifiableList.add("D");  // ✅ Works fine
```
Now:
- immutableList → remains unchanged and immutable
- modifiableList → is a new, independent list you can modify freely


---

# Page

Before we talk about servlets and servlet containers :
Servlet is simply a java class, which takes a request process and returns it.
Servlet container is something which manage the servlets.
Web.xml had the mapping for all the servlets. As the classes grew, it was difficult managing the xml.

There are certain problems with above approach which spring MVC solves :
1. Removal of web.xml
  1. Difficult to manage over time.
  1. Spring introduced annotation based configuration.
1. Inversion of control
  1. Used to manage object dependency and its life cycle.
1. Unit testing is harder in servlet.
1. Difficult to manage rest APIs
1. Integration of other tools/frameworks is really easy in spring.

Why Spring boot?
1. No need to add seperate depedencies for spring like servlet, junit etc. Just adding starter dependecy, rest would be managed.
1. Auto Configuration: No need for separately configuring "DispatcherServlet", "AppConfig", "EnableWebMvc", "ComponentScan". Spring boot add internally by-default.
1. In traditional Spring MVC application, we need to build a WAR file, which is a packaged file containing your application's classes, JSP pages, configuration files, and dependencies. Then we need to deploy this WAR file to a servlet container like Tomcat. But in Spring boot, Servlet container is already embedded, we don't have to do all this stuff. Just run the application, that's all.

Layered architecture in spring boot :


---

# Page

Maven is a build automation and dependency management tool primarily used for Java projects. It’s widely used in Spring Boot and other Java-based enterprise applications.
### ✅ What Is Maven?
At its core, Maven:
- Builds your project (compiles, tests, packages).
- Manages dependencies (automatically downloads required libraries).
- Uses a project descriptor file: pom.xml (Project Object Model).
### 💡 Key Concepts to Know for Interviews
### 1. What is pom.xml?
- It's the central configuration file in Maven.
- Defines project info, dependencies, plugins, build settings.
Example:
```xml
xml
CopyEdit
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>myapp</artifactId>
  <version>1.0</version>
</project>
```
### 2. Dependencies
- Defined in <dependencies> block.
- Maven fetches them from Maven Central or other repositories.
- Each dependency includes: groupId, artifactId, version.
Example:
```xml
xml
CopyEdit
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <version>2.7.0</version>
</dependency>
```
### 3. Maven Lifecycle Phases
You should know the three built-in lifecycles:
- clean: cleans compiled files.
- default: handles compilation, testing, packaging.
- site: generates documentation.
Key default phases:
### 4. Transitive Dependencies
- Maven automatically pulls in dependencies of your dependencies.
- Can be excluded if they cause conflicts.
### 5. Scopes
Defines when a dependency is used:
- compile – default; needed at compile/runtime.
- provided – available at runtime (e.g. servlet-api).
- runtime – not needed at compile time (e.g. JDBC drivers).
- test – only for tests.
### 6. Plugins
- Provide additional build goals (e.g., for compilation, test, or packaging).
Example: maven-compiler-plugin, spring-boot-maven-plugin.
### 7. Maven Repositories
- Local: ~/.m2/repository
- Central: Maven Central Repository (public)
- Remote/Private: Nexus, Artifactory
### 8. Maven Commands
### 9. Dependency Conflicts
- Multiple versions of the same dependency → Maven picks the closest one in the dependency tree (nearest-first strategy).
### 📌 Maven vs Gradle
### 
### ✅ 1. What is Maven and why is it used?
Answer:
Maven is a build automation and dependency management tool used primarily for Java projects. It simplifies the project build process by using a pom.xml file where we can define project structure, dependencies, plugins, and build lifecycle configurations.
In Spring Boot, Maven helps manage libraries like Spring Web, JPA, and Test dependencies, ensuring consistent builds and easier deployment.
### ✅ 2. What is pom.xml? What are its main elements?
Answer:
pom.xml (Project Object Model) is the core configuration file in a Maven project. It defines:
- Project metadata (groupId, artifactId, version)
- Dependencies
- Plugins
- Build lifecycle settings
Example:
```xml
xml
CopyEdit
<groupId>com.example</groupId>
<artifactId>demo</artifactId>
<version>1.0.0</version>
```
### ✅ 3. What are Maven build lifecycles and key phases?
Answer:
Maven has three lifecycles: clean, default, and site.
The default lifecycle is the most used and includes phases like:
- compile – compiles Java code
- test – runs unit tests
- package – packages compiled code into JAR/WAR
- install – installs artifact in local .m2 repo
- deploy – uploads it to remote repo
### ✅ 4. What are dependency scopes in Maven?
Answer:
Scopes define when a dependency is available:
- compile (default): available at all stages.
- provided: available at compile/runtime but not bundled in the JAR (e.g., Servlet API).
- runtime: only required during runtime.
- test: used only in testing.
### ✅ 5. What are transitive dependencies? How does Maven handle them?
Answer:
Transitive dependencies are dependencies of your dependencies. Maven automatically resolves them, but if there's a conflict (two versions), Maven uses the nearest version in the dependency tree. You can exclude transitive dependencies if needed.
### ✅ 6. How do you handle dependency conflicts in Maven?
Answer:
Maven uses a nearest-first strategy in the dependency tree. If two versions exist, it picks the one closest to the root.

---

# Page

### ✅ 1. Core Spring Boot Annotations
### ✅ 2. Component Annotations (for Dependency Injection)
### ✅ 3. Web Annotations
### ✅ 4. Dependency Injection & Bean Management
### ✅ 5. Spring Data JPA Annotations
### ✅ 6. Spring Boot Configuration Annotations
### ✅ 7. Testing Annotations
## ✅ Error Handling Annotations
## ✅ Asynchronous Processing Annotations

---

# Page

Bean is a java object, that is managed by the spring container(IOC container)
IOC container → creation, intializaion, lifecycle management of beans.
We can create bean using:
1. component annotation :
  1. default constructor of a class is used here to create object.(controller, service, repository)
  1. If explicit cons mentioned in class, then that will be called.
1. bean annotation

at what time beans get created? eager and lazy
life cycle of bean:
1. application start → invoke ioc(web application context class) → scan beans via component and configuration → construct beans → inject dependency into the beans(if dependecy bean already present use it, or else create a new one and then inject it) → post contruct task is performaed → use bean → in predestry annotation performa any task before the bean is destroyed → bean destroyed
1. 
```javascript
@Component
public class User {
@Autowired
Order order;
public User() {
System.out.println("initializing user");}}


@Lazy
@Component
public class Order {
public Order() {
System.out.println("Lazy: initializing Order");}}



2024-04-09T00:53:11.751+05:30 INFO 60872 --- main] o.s.b.w.embedded.tomcat. TomcatWebServer Tomcat initialized with port 8080 (http)
2024-04-09T00:53:11.757+05:30 INFO 68872 [ main] o.apache.catalina.core.StandardService Starting service [Toncat]
2024-04-09T00:53:11.758+05:30 INFO 68872 [ main] o.apache.catalina.core.StandardEngine Starting Servlet engine: [Apache Tomcat/10.1.19]
2024-04-09T00:53:11.782+05:30 INFO 68872 --- [ main] o.a.c.c.c.[Tomcat]. [localhost].[/] Initializing Spring embedded WebApplicationContext
2024-04-09T00:53:11.82-05:30 INFO 68872 [ main] w.s.c.ServletWebServerApplicationContext: Root WebApplicationContext: initialization completed in 416 ms
initializing user
Lazy: initializing Order
2024-04-09T00:53:11.931+05:30 INFO 60872 --- [ main] o.s.b.w.embedded.tomcat. TomcatWebServer Tomcat started on port 8888 (http) with context path **
2024-04-09T00:53:11.936+05:30 INFO 68872 --- [ main] c.c.1.SpringbootApplication : Started SpringbootApplication in 8.76 seconds (process running o


EVEN THOUGH THERE IS A LAZY ANNOTATION, STILL ORDER BEAN WILL BE CREATED IN AN EAGER MANNER BECAUSE IT IS REQUIRED TO BE INJECTED IN ANOTHER EAGER BEAN.
```


---

# Page

### ✅ What is the Factory Design Pattern?
The Factory Design Pattern is a creational pattern that provides an interface for creating objects in a superclass but allows subclasses to alter the type of objects that will be created.
Instead of calling a constructor directly to create an object, you use a factory method.
### ✅ When to Use It?
Use the Factory Pattern when:
- You have a superclass/interface with multiple subclasses and need to return one of them based on input.
- Object creation logic is complex, and centralizing it in a factory improves readability and maintainability.
- You want to decouple object creation from the client code.
- You want to follow the Open/Closed Principle — new types can be added without modifying the existing code.
### ✅ Real-life Analogy
Think of a pizza store:
- You don’t make your own pizza (i.e., you don't call new CheesePizza() directly).
- You ask the store: "Give me a Cheese Pizza".
- The store (factory) decides which exact subclass to return.
### ✅ Basic Java Template
### Step 1: Define the common interface
```java
java
CopyEdit
public interface Shape {
    void draw();
}
```
### Step 2: Implement concrete classes
```java
java
CopyEdit
public class Circle implements Shape {
    public void draw() {
        System.out.println("Drawing a Circle");
    }
}

public class Square implements Shape {
    public void draw() {
        System.out.println("Drawing a Square");
    }
}
```
### Step 3: Create the Factory class
```java
java
CopyEdit
public class ShapeFactory {

    public Shape getShape(String type) {
        if (type == null) return null;
        switch (type.toUpperCase()) {
            case "CIRCLE": return new Circle();
            case "SQUARE": return new Square();
            default: return null;
        }
    }
}
```
### Step 4: Client code
```java
java
CopyEdit
public class Main {
    public static void main(String[] args) {
        ShapeFactory factory = new ShapeFactory();

        Shape shape1 = factory.getShape("CIRCLE");
        shape1.draw();

        Shape shape2 = factory.getShape("SQUARE");
        shape2.draw();
    }
}
```
### ✅ Benefits
- Loose Coupling: Client code is not tightly bound to concrete classes.
- Single Responsibility: Factory handles object creation.
- Scalability: Easy to introduce new types by adding new classes and updating the factory.

---

# Page

### 1. Creational Design Patterns
Definition:
Creational patterns focus on object creation mechanisms, trying to create objects in a manner suitable to the situation. They abstract the instantiation process, making the system independent of how its objects are created, composed, and represented.
Purpose:
To control object creation to reduce complexity and increase flexibility in code.
Examples:
- Singleton
- Factory Method
- Abstract Factory
- Builder
- Prototype
### 2. Structural Design Patterns
Definition:
Structural patterns deal with object composition and typically help ensure that components are connected in a flexible and efficient manner. They focus on how classes and objects are composed to form larger structures.
Purpose:
To simplify the design by identifying simple ways to realize relationships between entities.
Examples:
- Adapter
- Decorator
- Proxy
- Composite
- Bridge
- Flyweight
- Facade
### 3. Behavioral Design Patterns
Definition:
Behavioral patterns are concerned with communication between objects. They focus on how objects interact and distribute responsibility.
Purpose:
To define how objects communicate and assign responsibility, promoting loose coupling and flexibility.
Examples:
- Observer
- Strategy
- Command
- Chain of Responsibility
- State
- Template Method
- Mediator
- Memento
- Visitor
- Iterator
- Interpreter

---

# Page

### 🔷 What is the Builder Design Pattern?
The Builder Design Pattern is a creational design pattern used to construct complex objects step by step. Unlike other creational patterns, the builder pattern doesn’t require all parameters at once—instead, it allows you to create different representations of an object using the same construction process.
### 🔧 When to Use It
Use the Builder Pattern when:
- An object has many optional fields or parameters.
- You want to avoid telescoping constructors (constructors with many parameters).
- You need to construct different variants of an object.
### 🧱 Structure
There are generally four main components:
1. Product – The object being built.
1. Builder – Abstract interface defining the steps.
1. ConcreteBuilder – Implements the steps to construct and assemble parts.
1. Director (optional) – Controls the building process.
### ✅ Example in Java (Without Director)
Let’s create a User object with optional fields:
```java
java
CopyEdit
public class User {
    // Required parameters
    private final String firstName;
    private final String lastName;

    // Optional parameters
    private final int age;
    private final String phone;
    private final String address;

    private User(UserBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }

    public static class UserBuilder {
        private final String firstName;
        private final String lastName;

        private int age;
        private String phone;
        private String address;

        public UserBuilder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }

        public UserBuilder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }

    @Override
    public String toString() {
        return "User{" +
               "firstName='" + firstName + '\'' +
               ", lastName='" + lastName + '\'' +
               ", age=" + age +
               ", phone='" + phone + '\'' +
               ", address='" + address + '\'' +
               '}';
    }
}
```
### 🧪 Usage
```java
java
CopyEdit
public class Demo {
    public static void main(String[] args) {
        User user = new User.UserBuilder("John", "Doe")
                          .age(30)
                          .phone("1234567890")
                          .address("123 Main St")
                          .build();

        System.out.println(user);
    }
}
```
### 🎯 Key Advantages
- Readable and maintainable object creation.
- No need to pass nulls for optional parameters.
- Helps enforce immutability.
## 🤔 What Problem Are We Solving?
Imagine you’re trying to create a User object. You might have:
- Required fields: firstName, lastName
- Optional fields: age, phone, address, etc.
### 🚫 Traditional Way – Telescoping Constructor Problem
If you go with constructors, you might end up with something like this:
```java
java
CopyEdit
// One constructor:
User(String firstName, String lastName)

// Another with more:
User(String firstName, String lastName, int age)

// Another:
User(String firstName, String lastName, int age, String phone)

// And so on...
```
As you add more fields, you get many overloaded constructors. This becomes messy and hard to maintain.
## ✅ Builder Pattern to the Rescue
The Builder Pattern solves this problem by allowing step-by-step construction and only setting the fields you care about.
## 🧱 Components of the Builder Pattern (with your example)
Let’s revisit the same example with roles labeled:
### 1. Product – The object we are building
```java
java
CopyEdit
public class User {
    // This is the Product
    ...
}
```
This is the final object we want to build, and it can be complex (with optional parts). It is immutable once built.
### 2. Builder – Interface or nested class defining how to build the product
```java
java
CopyEdit
public static class UserBuilder {
    // This is the Builder
    ...
}
```
It defines methods like age(), phone(), address(), and a final build().
### 3. Concrete Builder – The actual class that implements the builder steps
```java
java
CopyEdit
public static class UserBuilder {
    // Same as Builder here, since it's nested and concrete
}
```
In this example, Builder and ConcreteBuilder are the same class, because it’s a nested static class. In other cases (especially when Builder is an interface), you’d have a separate ConcreteBuilder class.
### 4. Director – Orchestrates building process (optional)
In our example:
```java
java
CopyEdit
User user = new User.UserBuilder("John", "Doe")
                .age(30)
                .phone("1234567890")
                .address("123 Main St")
                .build();
```
The code calling this (main()) acts as the Director, deciding which parts to build.
In more complex systems (like system design), the Director is a separate class that calls builder methods in a fixed order to build a standard version of the object.
## 🎯 What Are We Achieving?
### Without Builder:
- Lots of constructors.
- Hard to know which parameter is which (User("John", "Doe", 30, "1234567890", "123 Main St") is not readable).
- Need to pass null for fields you don’t want.
### With Builder:
- Much more readable.
- Set only what you need.
- Can make the object immutable.
- Cleaner, more maintainable code.
## 🔁 Summary

---

# Page

# 🧵 Low-Level Concurrency Handling in Java – Notes
## 🔹 1. Thread Safety Fundamentals
## 🔹 2. Singleton vs Multi-instance in Threaded Context
### ✅ Singleton Class:
- Ensures only one instance per JVM
- Guarantees that shared data (even non-static) remains globally consistent
- Good for shared services like BookingService, CacheManager, Logger
### ❌ Multi-instance Class:
- Each object has its own lock and fields
- Shared consistency must be managed via static fields or external locking
## 🔹 3. Static vs Non-Static Fields
✅ If you want multiple instances to share state → make the field static
## 🔹 4. Synchronization Options
### 🔒 Synchronized Instance Method:
```java
java
CopyEdit
public synchronized void bookSeat() { ... }
```
- Locks on this object
- Only ensures one thread per instance is executing it
### 🔒 Synchronized Static Method:
```java
java
CopyEdit
public static synchronized void bookSeat() { ... }
```
- Locks on the Class object
- Ensures mutual exclusion across all instances
### 🔒 Synchronized Block on Shared Lock:
```java
java
CopyEdit
private static final Object lock = new Object();

public void bookSeat() {
    synchronized (lock) {
        // critical section
    }
}
```
- Fine-grained control over locking
- Allows syncing only specific sections of code
## 🔹 5. Thread-Safe Data Structures
✅ These often eliminate the need for synchronized in many cases
## 🔹 6. Double-Checked Locking Singleton Pattern
```java
java
CopyEdit
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```
### 🔐 Why volatile?
- Prevents instruction reordering
- Ensures visibility across threads
- Without it, another thread might see a partially constructed object
## 🔹 7. When synchronized is Not Enough
- Synchronizing instance methods in a multi-instance class does not guarantee safety
- Each instance has its own lock — threads can still enter at the same time
### ✅ Safer Designs:
- Use static shared resources with thread-safe classes
- Or use Singleton + thread-safe DS
- Or synchronize on shared class-level locks
## 🧠 Key Takeaways
- Thread-safety is more about shared state than number of instances
- Singleton + non-static DS = safe if thread-safe DS used
- Multi-instance class needs static shared DS to maintain consistency
- Synchronize only where needed — don’t block the world!

---

# Page



---

# Page

The State Design Pattern allows an object to change its behavior based on its internal state, as if the object itself changed its class.
Instead of using lots of if-else or switch conditions to handle different states, we represent each state as a separate class that encapsulates the behavior for that state. The context object delegates the work to the current state, and the state itself can trigger transitions.
For example, think of a document in a workflow. It can be in a Draft, Review, or Published state.
  Each state allows or disallows different actions. When in Draft, I can submit it for review. In Review, I can approve or reject it. Once Published, I can’t modify it anymore.
### Tempelate:
```java
// 1. State Interface
public interface State {
    void handle(Context context);
}

// 2. Concrete States
public class StateA implements State {
    public void handle(Context context) {
        System.out.println("Currently in State A. Moving to State B.");
        context.setState(new StateB());
    }
}

public class StateB implements State {
    public void handle(Context context) {
        System.out.println("Currently in State B. Moving to State A.");
        context.setState(new StateA());
    }
}

// 3. Context Class
public class Context {
    private State currentState;

    public Context(State initialState) {
        this.currentState = initialState;
    }

    public void setState(State state) {
        this.currentState = state;
    }

    public void request() {
        currentState.handle(this);
    }
}

// 4. Demo
public class StatePatternDemo {
    public static void main(String[] args) {
        Context context = new Context(new StateA());

        context.request(); // State A handles and transitions to B
        context.request(); // State B handles and transitions to A
    }
}
```
Please refer github for examples.

---

# Page

## ✅ 1. What are annotations in Java?
“Annotations in Java are a form of metadata that provide additional information about the code to the compiler, tools, or runtime environments. They don’t directly affect program execution but are heavily used in Java tooling and frameworks to drive behavior — for example, telling the compiler to check something, or frameworks like Spring how to wire components.”
## ✅ 2. Why are annotations useful?
“They make code more declarative, reducing boilerplate. Instead of writing logic to mark a method as deprecated, for example, I can just use @Deprecated. They’re also used in Java frameworks like Spring, JPA, and JUnit for configuration, validation, dependency injection, and more — making development faster and cleaner.”
## ✅ 3. Common Annotations in Core Java
You can list and explain these as:
### 🔸 @Override
“Used when overriding a method from a superclass or interface. It helps the compiler catch errors — for example, if I mistype the method name or signature, it’ll throw a compile-time error.”
### 🔸 @Deprecated
“Marks an element (method, class, etc.) as outdated. If someone tries to use it, the compiler will show a warning. This is useful when we want to maintain backward compatibility but discourage usage.”
### 🔸 @SuppressWarnings
“Tells the compiler to ignore specific warnings — like unchecked type casts, deprecated method usage, or unused variables. I usually use it when working with legacy code or raw types.”
### 🔸 @FunctionalInterface (Java 8+)
“Used to mark interfaces with a single abstract method — these are intended for lambda expressions. The annotation ensures that no additional abstract methods are accidentally added.”
### 🔸 @SafeVarargs
“Used to suppress warnings related to varargs and generics. This is particularly useful in utility methods that take variable arguments of generic types.”
### 🔸 @Native
“Marks constants to be accessible in native code via JNI. It’s more of a low-level use case.”
## ✅ 4. Meta-Annotations (Used when writing custom annotations)
“Java provides a few meta-annotations which we use to define the behavior of custom annotations.”
## ✅ 5. Retention Policies (Interview-ready explanation)
“There are 3 types of retention policies defined by @Retention:
- SOURCE: discarded by the compiler, used only during development
- CLASS: present in the compiled .class file but not accessible at runtime
- RUNTIME: available through reflection at runtime — which is what frameworks like Spring or Hibernate often rely on.”
## ✅ 6. When would you use annotations in your code?
“In my own code, I regularly use annotations like @Override and @SuppressWarnings for clarity and avoiding bugs. But more importantly, I understand how custom annotations combined with @Retention(RUNTIME) and reflection can help drive extensible designs — like building a rule engine or plug-and-play validators.”
## ✅ 7. Summary if you’re wrapping up an answer
“So in short, annotations let us add declarative, tool-readable information to our code. They’re deeply integrated into modern Java development — from standard annotations like @Override, to how frameworks like Spring and JUnit work under the hood.”

---

# Page



---

# Page

## 🔑 What does static mean in Java?
- A static member (variable, method, block, class) belongs to the class itself, not to any specific object.
- It is shared across all instances of that class.
- It can be accessed without creating an object of the class.

## 🧮 1. Static Primitive Variables
```java
class Example {
    static int count = 0;
}
```
- ✅ Shared across all objects.
- ✅ Initialized only once when the class is loaded (not every time an object is created).
- ✅ Stored in Method Area (class memory), not on the heap.
### Real-world example:
```java
class User {
    static int totalUsers = 0;

    User() {
        totalUsers++;
    }
}
```
All users share the same totalUsers counter.
## 📦 2. Static Object Reference
```java
class Example {
    static MyClass obj = new MyClass();
}
```
- Same logic as primitive: the reference is shared across all objects.
- If you mutate obj, the change is reflected across all usages.
```java
class Config {
    static Logger logger = new Logger("INFO");
}
```
All classes using Config.logger share the same Logger instance.
## 🧰 3. Static Methods
```java
class MathUtils {
    static int square(int x) {
        return x * x;
    }
}
```
- ✅ Can be called without object creation: MathUtils.square(5)
- ❌ Cannot access non-static variables or methods directly because static methods do not belong to any instance.
### Use cases:
- Utility/helper classes like Math, Collections, etc.
- Factory methods (public static YourClass create(...))
## 🧱 4. Static Blocks
```java
class Example {
    static {
        System.out.println("Static block called");
    }
}
```
- Used to run initialization code only once, when the class is loaded.
- Multiple static blocks are allowed; they execute in order.
## 🧭 5. Static Classes (Nested Static Class)
```java
class Outer {
    static class Inner {
        void print() {
            System.out.println("Inside static nested class");
        }
    }
}
```
- Only nested classes can be static.
- A static nested class:
  - ❌ Cannot access non-static members of outer class directly.
  - ✅ Can be instantiated without an instance of outer class:
    Outer.Inner obj = new Outer.Inner();
## 🚫 What you cannot do with static:
- ❌ Cannot make a top-level class static (only inner/nested).
- ❌ Cannot use this inside a static context.
- ❌ Static methods can't override instance methods (they can hide them though — static method hiding).
## 📍 Why and When to Use Static?
### ✅ Use static when:
- The variable or method should be shared across all instances.
- You’re building a utility or helper class.
- You need a global state (e.g., Constants, Logger, counters).
- You want to ensure something runs once per class, not per object (e.g., static block or singleton instance).
## 📎 Summary Table
Yes, you can reassign a static variable or object reference to a new object in the heap — absolutely.

## ❌ Why You Cannot Use Non-Static Variables in Static Methods
### 🔴 The rule:
Static methods cannot access non-static (instance) variables or methods directly.
### 🧠 Why?
Because:
- A static method belongs to the class itself.
- A non-static variable belongs to a specific object (instance).
So, when you call a static method — the JVM has no idea which object’s instance variable you’re talking about.

---

# Page

## 1. Encapsulation
Definition:
Wrapping data (variables) and methods (functions) that operate on that data into a single unit (class), and restricting direct access to the data to protect it from unauthorized changes.
Key Points for Interviews:
- Achieved by private fields and public getters/setters.
- Improves data security and maintainability.
- Lets you control how data is accessed/modified.
- Not the same as data hiding, but data hiding is a part of encapsulation.
Java Example:
```java
public class BankAccount {
    private double balance; // private → cannot access directly

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }

    public double getBalance() {
        return balance; // controlled access
    }

    public void deposit(double amount) {
        if (amount > 0) balance += amount;
    }
}
```
💡 Interview Tip:
If asked:
"Is data hiding and encapsulation the same?"
  Answer: "No — data hiding is a part of encapsulation. Encapsulation is about binding data and behavior, while data hiding focuses on restricting direct access to that data."

## 2. Abstraction
Definition:
Hiding implementation details and exposing only the necessary functionality to the user.
Key Points:
- Achieved through abstract classes and interfaces in Java.
- Focuses on "what" to do, not "how" to do it.
- Improves modularity and reduces complexity.
Java Example:
```java
abstract class Vehicle {
    abstract void start(); // only method signature
}

class Car extends Vehicle {
    void start() {
        System.out.println("Car starting with key...");
    }
}
```

## 3. Inheritance
Definition:
Mechanism by which one class can acquire the properties and methods of another class.
Key Points:
- Promotes code reusability.
- “is-a” relationship (Dog is-a Animal).
- Types: Single, Multilevel, Hierarchical (Java doesn't support multiple inheritance with classes).
Java Example:
```java
class Animal {
    void eat() { System.out.println("Eating..."); }
}

class Dog extends Animal {
    void bark() { System.out.println("Barking..."); }
}
```

## 4. Polymorphism
Definition:
Ability of a single entity (method or object) to take many forms.
Types:
- Compile-time polymorphism (Static binding) → Method Overloading.
- Runtime polymorphism (Dynamic binding) → Method Overriding.
Java Example – Overloading:
```java
class MathUtils {
    int sum(int a, int b) { return a + b; }
    double sum(double a, double b) { return a + b; }
}
```
Java Example – Overriding:
```java
class Animal {
    void sound() { System.out.println("Animal sound"); }
}
class Dog extends Animal {
    @Override
    void sound() { System.out.println("Bark"); }
}
```

## 1. Encapsulation – Follow-up Q&A
Q1: How is encapsulation different from abstraction?
A:
Encapsulation is about binding data and methods together in one unit and controlling access to that data.
Abstraction is about exposing only necessary details and hiding the implementation.
Example: In a car, encapsulation is having the engine inside a protected casing, abstraction is letting the driver just use the accelerator without knowing engine details.
Q2: Why use getters/setters instead of public variables?
A:
- Getters/setters let you validate data before changing it.
- You can make fields read-only or write-only.
- You can change the internal representation later without breaking outside code.
- Public variables break data integrity and make maintenance harder.
## 2. Abstraction – Follow-up Q&A
Q1: How is abstraction different from encapsulation?
A:
Encapsulation → “How to hide data” (restrict access).
Abstraction → “How to hide complexity” (expose only what’s needed).
Encapsulation is about access control; abstraction is about design simplification.
Q2: Which is better for abstraction in Java — interface or abstract class?
A:
- Interface: Use when you want to define a contract for classes to follow, multiple inheritance of type allowed.
- Abstract class: Use when you want to share partial implementation along with the abstraction.
- Java 8+ interfaces can have default & static methods, reducing the gap between them.
## 3. Inheritance – Follow-up Q&A
Q1: Why doesn’t Java support multiple inheritance with classes?
A:
To avoid ambiguity in the Diamond Problem — where two parent classes have the same method and the child doesn’t know which one to use.
Java allows multiple inheritance via interfaces because they only define methods, not maintain state.
Q2: When to use composition instead of inheritance?
A:
- Use inheritance when there is a strict "is-a" relationship (Dog is-a Animal).
- Use composition when there is a "has-a" relationship (Car has-a Engine).
- Composition is preferred for flexibility and avoiding tight coupling.
## 4. Polymorphism – Follow-up Q&A
Q1: How does JVM achieve runtime polymorphism?
A:
Through dynamic method dispatch:
At runtime, JVM looks at the actual object’s type (not reference type) and uses the virtual method table (vtable) to decide which overridden method to call.
Q2: Can we override static methods?
A:
No. Static methods are bound at compile time (static binding) and belong to the class, not the object.
If a subclass defines a static method with the same signature, it hides the parent method (method hiding), not overrides it.

---

# Page

The KISS principle stands for "Keep It Simple, Stupid" (sometimes phrased as "Keep It Short and Simple").
It’s a software design and problem-solving principle that emphasizes simplicity over unnecessary complexity.
### Core Idea
- Simple solutions are better: A system works best when it is kept simple rather than made complicated.
- Avoid over-engineering: Don’t add extra logic, layers, or abstractions unless they are absolutely necessary.
- Clarity > Cleverness: Code should be straightforward and easy to understand, even if it seems less “smart” or “fancy.”
### Example in Software
Suppose you need a function to calculate the sum of numbers in a list.
❌ Complex way (violates KISS):
```java
public int sum(List<Integer> numbers) {
    return numbers.stream().reduce(0, (a, b) -> a + b);
}
```
(While functional, it may be harder for beginners to read.)
✅ Simple way (follows KISS):
```java
public int sum(List<Integer> numbers) {
    int total = 0;
    for (int num : numbers) {
        total += num;
    }
    return total;
}
```
This is clear, direct, and easy to maintain.
### Benefits of KISS
- Easier to read, test, and debug
- Lower risk of bugs
- Faster onboarding for new developers
- Improves maintainability and scalability in the long run
👉 In short: Always prefer the simplest solution that gets the job done.

---

# Page

The DRY Principle stands for Don’t Repeat Yourself.
It’s one of the most important principles in software development and is part of clean code practices.
### 🔹 Definition
The DRY principle says:
"Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."
In simple words:
👉 Don’t write the same logic/code/data in multiple places.
👉 If something changes, you should only need to change it in one place.
### 🔹 Why is DRY important?
1. Maintainability – If you duplicate logic and a bug is found, you’ll need to fix it in multiple places. DRY reduces effort.
1. Readability – Cleaner and more understandable code.
1. Scalability – Easy to extend features without worrying about duplicated changes.
1. Consistency – Avoids mismatched behavior caused by missed updates in duplicated code.
### 🔹 Examples
### ❌ Bad (Violation of DRY):
```java
public class UserService {
    public void validateEmail(String email) {
        if (!email.contains("@") || email.length() < 5) {
            throw new IllegalArgumentException("Invalid Email");
        }
    }

    public void registerUser(String name, String email) {
        if (!email.contains("@") || email.length() < 5) {
            throw new IllegalArgumentException("Invalid Email");
        }
        // registration logic
    }

    public void updateUser(String userId, String email) {
        if (!email.contains("@") || email.length() < 5) {
            throw new IllegalArgumentException("Invalid Email");
        }
        // update logic
    }
}
```
👉 The email validation logic is repeated 3 times.
### ✅ Good (Following DRY):
```java
public class UserService {
    private void validateEmail(String email) {
        if (!email.contains("@") || email.length() < 5) {
            throw new IllegalArgumentException("Invalid Email");
        }
    }

    public void registerUser(String name, String email) {
        validateEmail(email);
        // registration logic
    }

    public void updateUser(String userId, String email) {
        validateEmail(email);
        // update logic
    }
}
```
👉 Validation is centralized in one method. If rules change, you only update once.
### 🔹 DRY Beyond Code
DRY applies not only in functions but also in:
- Database design → Normalization (avoiding redundant data).
- Configuration files → Using constants or environment variables instead of hardcoding.
- Documentation → Generate docs from code/comments instead of maintaining them separately.
✅ In short: DRY = One place, one truth.

---

# Page

The YAGNI principle stands for:
"You Aren’t Gonna Need It"
### ✅ Definition:
YAGNI is a software development principle that advises:
Don’t implement something until you actually need it.
### 📌 Purpose:
To avoid unnecessary complexity and premature features in your codebase.
### 🔍 Example:
### ❌ Without YAGNI:
```java
public class ReportGenerator {
    public void generatePDFReport() {
        // code to generate PDF
    }

    public void generateExcelReport() {
        // code to generate Excel report
    }

    public void generateHtmlReport() {
        // code to generate HTML report
    }
}
```
Here, you've implemented 3 types of reports even if only PDF is currently needed.
### ✅ With YAGNI:
```java
public class ReportGenerator {
    public void generatePDFReport() {
        // code to generate PDF
    }
}
```
Only implement what's needed right now. Add Excel/HTML later if and when required.
### 🎯 Why Follow YAGNI?
- Reduces code complexity
- Saves development time
- Makes code easier to test and maintain
- Encourages agile, incremental development
### 🛑 Common Violations of YAGNI:
- Writing utility/helper methods “just in case”
- Creating elaborate configurations or plugin systems no one needs yet
- Designing overly generic classes early on
### 🧠 Remember:
Just because you can write the code, doesn’t mean you should — unless there's a clear, current need.


---

# Page

## 🔹 Definition
The Adapter Design Pattern is a structural design pattern that allows incompatible interfaces to work together.
It acts like a bridge between two classes.
Think of it as a translator:
- You have some existing code (a client) expecting one interface.
- You want to use a different class (adaptee) that has a different interface.
- Instead of changing either, you introduce an Adapter that converts one interface into the other.

## 🔹 Example in Java
Suppose we have an app that needs to play MP3 files, but we also want to support MP4 (different interface).
```java
// Step 1: Target interface
interface MediaPlayer {
    void play(String filename);
}

// Step 2: Adaptee (incompatible interface)
class AdvancedMediaPlayer {
    public void playMp4(String filename) {
        System.out.println("Playing mp4 file: " + filename);
    }
}

// Step 3: Adapter (bridge between the two)
class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;

    public MediaAdapter(AdvancedMediaPlayer advancedPlayer) {
        this.advancedPlayer = advancedPlayer;
    }

    @Override
    public void play(String filename) {
        // adapting MP3 request to MP4
        advancedPlayer.playMp4(filename);
    }
}

// Step 4: Client code
public class AdapterPatternDemo {
    public static void main(String[] args) {
        MediaPlayer player = new MediaAdapter(new AdvancedMediaPlayer());
        player.play("song.mp4");
    }
}
```
## 🔹 How to Explain to Interviewer
"Adapter Design Pattern allows incompatible interfaces to work together. Instead of modifying existing code, we create an Adapter that translates one interface into another. A common real-world example is a power adapter that converts a plug type so it can fit into a socket. In code, it helps us integrate legacy or third-party code into our system smoothly."

---

# Page

The Chain of Responsibility Design Pattern is a behavioral design pattern used to pass a request along a chain of handlers. Each handler can either handle the request or pass it to the next handler in the chain.
### 🔧 Intent
Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request.
### 📦 Real-World Analogy
A customer service request flows through:
- Level 1 support → Level 2 support → Manager
  Only the person equipped to handle it will take action.
### 🧩 Structure
- Handler (Abstract class/interface): Declares method for handling the request and holds a reference to the next handler.
- ConcreteHandler: Handles request or forwards it to the next handler.
- Client: Initiates the request.

### 🧪 Template Code (Java)
```java
// Step 1: Abstract Handler
abstract class Handler {
    protected Handler next;

    public void setNext(Handler next) {
        this.next = next;
    }

    public abstract void handleRequest(String request);
}

// Step 2: Concrete Handlers
class ConcreteHandlerA extends Handler {
    public void handleRequest(String request) {
        if (request.equals("A")) {
            System.out.println("Handler A processed the request");
        } else if (next != null) {
            next.handleRequest(request);
        }
    }
}

class ConcreteHandlerB extends Handler {
    public void handleRequest(String request) {
        if (request.equals("B")) {
            System.out.println("Handler B processed the request");
        } else if (next != null) {
            next.handleRequest(request);
        }
    }
}

// Step 3: Client
public class Client {
    public static void main(String[] args) {
        Handler handlerA = new ConcreteHandlerA();
        Handler handlerB = new ConcreteHandlerB();

        handlerA.setNext(handlerB); // Chain: A -> B

        handlerA.handleRequest("A"); // Output: Handler A processed the request
        handlerA.handleRequest("B"); // Output: Handler B processed the request
        handlerA.handleRequest("C"); // No output (no handler could process)
    }
}
```
### 

---

# Page

### 🔹 Definition:
The Memento Pattern is a behavioral design pattern that lets you capture and store the internal state of an object without violating encapsulation, so that it can be restored later.
### 🔹 When to Use:
- When you need to implement undo/rollback functionality.
- When you want to keep a history of states of an object (like in text editors).
- When you want to maintain encapsulation (i.e., the object’s internal state should not be exposed to others).
### 🔹 Key Components:
### 🔹 UML Diagram:
```plain text
+---------------+         +---------------+         +---------------+
|   Originator  | <-----> |   Memento     |         |  Caretaker    |
+---------------+         +---------------+         +---------------+
| - state       |         | - state       |         | - mementos[]  |
| + save()      |         | + getState()  |         | + saveMemento |
| + restore()   |         |               |         | + undo()      |
+---------------+         +---------------+         +---------------+
```
### 🔹 Java Template Code:
```java
// Memento class — stores state
class Memento {
    private final String state;

    public Memento(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }
}

// Originator class — whose state we want to save/restore
class Originator {
    private String state;

    public void setState(String state) {
        System.out.println("Setting state to: " + state);
        this.state = state;
    }

    public String getState() {
        return state;
    }

    // Save current state into a Memento
    public Memento save() {
        return new Memento(state);
    }

    // Restore from a Memento
    public void restore(Memento memento) {
        state = memento.getState();
        System.out.println("Restored state to: " + state);
    }
}

// Caretaker class — keeps track of Memento history
import java.util.Stack;

class Caretaker {
    private Stack<Memento> history = new Stack<>();

    public void saveState(Originator originator) {
        history.push(originator.save());
    }

    public void undo(Originator originator) {
        if (!history.isEmpty()) {
            originator.restore(history.pop());
        } else {
            System.out.println("No states to restore!");
        }
    }
}

// Demo usage
public class MementoPatternDemo {
    public static void main(String[] args) {
        Originator originator = new Originator();
        Caretaker caretaker = new Caretaker();

        originator.setState("State 1");
        caretaker.saveState(originator);

        originator.setState("State 2");
        caretaker.saveState(originator);

        originator.setState("State 3");
        System.out.println("Current state: " + originator.getState());

        caretaker.undo(originator); // should go back to State 2
        caretaker.undo(originator); // should go back to State 1
        caretaker.undo(originator); // nothing left
    }
}
```
### 🧠 Interview Tip:
When asked about Memento Pattern, emphasize:
- Encapsulation is preserved.
- It’s used for undo mechanisms.
- You can store snapshots of an object's state without exposing internals.

---

# Page



---

# Page



---

# Page

## 🔹 Marshalling vs Serialization
- Serialization:
  - Java-specific term.
  - Converting a Java object into a byte stream so it can be stored or transferred.
  - Focus is on persistence and transport.
- Marshalling:
  - More general term used in distributed systems (RMI, CORBA, Web Services, etc.).
  - Means packing an object (its state + metadata) into a format suitable for transmission over a network.
  - It’s like serialization, but often includes extra info like type, structure, version.
  - Unmarshalling is the reverse → reconstructing the object on the other side.
- Difference in short:
  Serialization is about converting an object to bytes.
    Marshalling is about preparing an object to move across systems (including serialization + metadata).
## 🔹 Where You’ll Hear It
- Java RMI (Remote Method Invocation):
  Objects are marshalled before being sent to another JVM, and unmarshalled at the receiving end.
- CORBA, SOAP, gRPC:
  Use marshalling/unmarshalling to send structured data across languages.
- REST APIs:
  When you convert an object to JSON (marshalling) and back (unmarshalling).
  
## 🔹 How to Frame in Interview
  If asked: “What is marshalling, how is it different from serialization?”
  👉
  Serialization is Java-specific and only converts an object to bytes. Marshalling is a broader concept — it not only serializes the object but also includes metadata, so the object can be reconstructed across different systems or languages. The reverse process is called unmarshalling. For example, in Java RMI, objects are marshalled before being sent over the network and unmarshalled on the receiving side.
  
  
## What kind of metadata are we talking about?
  Metadata = information that helps reconstruct the object properly on the other side.
  It depends on the protocol/format:
  - Java Serialization → class name, field names, types, version (serialVersionUID).
  - JSON → key names and their types (loosely enforced).
  - XML → tags, schema definitions (XSD).
  - Avro/Protobuf → strict schema (field id, type, default values).
  - SOAP (XML-based) → includes namespaces, type definitions.
  👉 The important bit:
  - If you use Java serialization, metadata works only with Java (tightly coupled).
  - If you use JSON/Protobuf/Avro, metadata is cross-platform → Java, Python, JS, Go can all understand it.
## 🔹 6. Final Wrap-up (Interview framing)
  If they ask: “Does marshalling/unmarshalling happen everywhere?”
  👉
  Yes, whenever data crosses a boundary — between app & DB, service & browser, or producer & consumer in event-driven systems — the object has to be converted into some transportable format (marshalling) and back (unmarshalling). The metadata included depends on the protocol/format: in Java serialization it’s class info, in JSON it’s key names, in Avro/Protobuf it’s schema definitions. For interoperability, we prefer formats with schemas like Avro/Protobuf because they work across platforms and ensure strong typing.

## 🔹 1. Does marshalling/unmarshalling happen in DB calls?
- Not in the same sense as network marshalling.
- When you push/fetch data from a relational DB:
  - Your Java object → converted into SQL statement / protocol format → DB engine stores it.
  - When reading: DB rows → JDBC driver converts them into Java types (int, String, ResultSet, etc.).
- This isn’t called “marshalling” in strict terms, but conceptually it’s similar: object ↔ transportable format ↔ object.
👉 So yes, conversion happens, but it’s JDBC (or ORM like Hibernate) doing object-relational mapping, not raw marshalling.
## 🔹 2. What about APIs (REST / gRPC / SOAP)?
- Absolutely yes.
- REST:
  - Java object → JSON/XML (marshalling).
  - JSON/XML → Java object (unmarshalling).
  - Frameworks: Jackson, Gson, JAXB, etc.
- gRPC/Protocol Buffers:
  - Java object → Protobuf binary stream (marshalling).
  - Protobuf stream → Java object (unmarshalling).
👉 Here, marshalling/unmarshalling is central, since the format must be understood across different systems/languages.
## 🔹 3. Event-driven systems (Kafka, RabbitMQ, etc.)
- Yes, happens here too.
- Producer:
  - Java object → serialized (JSON, Avro, Protobuf, or raw Java serialization) → sent to broker.
- Consumer:
  - Byte stream → deserialized back into object.
👉 Example:
In Kafka you might send an event as Avro. The Avro schema acts like metadata so any consumer (Java, Python, Go) can decode it.
## 🔹 4. Browser communication
- Browser <-> Server (HTTP, WebSocket):
  - Browser sends JSON, XML, Form-Data, Multipart, etc.
  - Server unmarshals that into objects.
  - Server response is marshalled back into JSON/HTML.
- Example:
```json
{ "id": 101, "name": "Srishti" }
```
  Browser sees this as JSON, backend unmarshals it into an Employee object.
👉 Yes, marshalling/unmarshalling happens in every API call from the browser.

---

# Page

### 1. What is Serialization?
- Definition:
  Serialization is the process of converting an object’s state (its data) into a byte stream so that it can be easily stored (e.g., in a file, DB) or transferred (e.g., over a network).
- In Java:
  Implemented via the java.io.Serializable interface (a marker interface, i.e., no methods).
  Example:
### 2. What is Deserialization?
- Definition:
  Deserialization is the reverse process: converting the byte stream back into a Java object.
### 3. Why do we need it?
- To persist objects (store them in files/databases).
- To send objects over a network (e.g., RMI, sockets, REST APIs in older systems).
- To cache or transfer state between different JVMs.
### 4. Key Interview Points
- Serializable interface: marker interface, tells JVM that this object can be serialized.
- transient keyword: used to skip serialization of sensitive/non-serializable fields (e.g., password).
- serialVersionUID:
  - A unique identifier for each class.
  - Ensures compatibility during deserialization.
  - If not present, JVM generates one automatically (but changes when class changes).
- Inheritance: If superclass implements Serializable, subclass automatically becomes serializable.
- Customization: Can override writeObject() and readObject() to control serialization.
- Externalizable: Interface that gives complete control (implements writeExternal() & readExternal()).
### 5. Common Interview Questions
1. What happens if a class is not Serializable but its object is serialized?
  → NotSerializableException.
1. Difference between Serializable and Externalizable?
  - Serializable: automatic, JVM handles it.
  - Externalizable: manual control, more efficient but more code.
1. What is the role of serialVersionUID?
  → Ensures compatibility between serialized object and class definition.
1. What fields are not serialized?
  → static and transient fields.
🔑 Interview-ready one-liner:
Serialization is converting an object into a byte stream for storage/transfer, and deserialization is reconstructing the object from that byte stream. In Java, it’s done using the Serializable interface, with transient and serialVersionUID playing key roles in controlling behavior and versioning.

---

# Page

API Gateway provides a single entry point for multiple endpoints.
Enables routing by forwarding request to the correct microservice.
Facilitates load balancing.
Authentication like JWT.
Rate limiting/Circuit breaker/Retry etc.
Request and response processing(post/pre process)
Monitoring and logging.

---

# Page



---

# Page



---

# Page



---

# Page



---

# Page

Bean is created conditionally based on true and false.

---

# Page

In Spring, a bean scope defines how long a bean lives and how many instances of it exist in the Spring container.
Here are the commonly used scopes:
### 🔹 1. Singleton (Default)
- Definition: Only one instance of the bean per Spring container.
- Lifecycle: Created at container startup (eager by default), reused everywhere.
- Use case: Stateless services, utility classes.
```java
@Component
@Scope("singleton")  // optional, since it's the default
public class MyService { }
```
### 🔹 2. Prototype
- Definition: A new instance is created every time it is requested.
- Lifecycle: Container creates it, then hands control to the caller (no automatic destruction).
- Use case: Stateful beans, per-request objects.
- It is lazily intialized, created when needed instead of keeping it the start itself.
```java
@Component
@Scope("prototype")
public class MyPrototypeBean { }
```
### 🔹 3. Request (Web-aware scope)
- Definition: One instance per HTTP request.
- Lifecycle: Created when a request comes in, destroyed at request completion.
- Use case: Web request-specific data like user session objects.
- Lazily intialized
- singleton controller with a request scope dependecy will result in failure, to resolve this use proxy mode.
- proxy mode tells spring boot to create s proxy object of that dpeendency for time being and injext in controller. When request comes use a new dependecy object every time.
```java
@Component
@Scope("request")
public class MyRequestBean { }
```
### 🔹 4. Session (Web-aware scope)
- Definition: One instance per HTTP session.
- Lifecycle: Tied to the user session.
- Use case: User preferences, shopping cart.
- when user access any endpoint the session is created, remains active till it not expires(default, can aslo configure)
```java
@Component
@Scope("session")
public class MySessionBean { }
```
### 🔹 5. Application (Web-aware scope)
- Definition: One instance per ServletContext (shared across all sessions/requests).
- Use case: Application-wide caches, shared resources.
- One instance across multiple IOC
```java
@Component
@Scope("application")
public class MyAppBean { }
```
### 🔹 6. WebSocket (Web-aware scope)
- Definition: One instance per WebSocket session.
- Use case: WebSocket-specific state/data.
```java
@Component
@Scope("websocket")
public class MyWebSocketBean { }
```
📌 Summary Table


---

# Page



---

# Page



---

# Page

To protect our system from different kinds of attacks we need two things :
1. Authentication : verify who you are
1. Authorization : what you are allowed to do
Thats where spring security comes into picture.

---

# Page



---

# Page

### CSRF(cross site request frogery)
### XSS(Cross site scripting)
1. Allows attacker to put a mallicious script viwed by other user.
1. Commponly used for stealing session and deforming website.
1. GET "/xss" endpoint, which loads all the comments. It returns "xss", since it's a controller class (not RestController) so, by-default it will try to look for "xss.html" file and try to render it.
1. POST "/comment" endpoint, which is not sanitizing any user input and simply stores this comment say in DB and then displays it during GET call.
1. So, if attacker put malicious script using this POST request, then during every GET call, this script will run for all the users who will make a call.
1. solution : proper escaping of user inputs also validate the data when rendering.

### CORS(Cross origin resouce sharing)
### SQL Injection
1. In this attacker manipulates SQL Query by inserting mallicious input in the fields.

---

# Page



---

# Page

Used to mark a menthod that should run asynchronously(In a separate thread)
Runs in a new thread withought blocking the main thread.
Enable async annotation : Put this in springboot application, it tells spring boot to create all the beans the are internally required to perform async operations.

---

# Page

### ✅ Short and Clear Interview Answer
"Optional in Java is a container class introduced in Java 8 that may or may not contain a non-null value. It helps to avoid NullPointerException and makes the code more readable by explicitly handling the absence of a value. Instead of returning null, we return an Optional, and the caller can safely check or retrieve the value using methods like isPresent(), orElse(), or ifPresent()."
### ✅ If They Ask for an Example
You can say:
For example, instead of returning null from a method:
```java
public Optional<String> getEmail(User user) {
    return Optional.ofNullable(user.getEmail());
}
```
  Then the caller can handle it safely:
```java
email.ifPresent(e -> System.out.println("Email: " + e));
```
  or
```java
String emailValue = email.orElse("No email found");
```
### ✅ If They Ask “Why do we need Optional?”
Say something like:
Traditionally, Java methods would return null for missing values, forcing developers to write multiple null checks. Optional makes the presence or absence of a value explicit and provides built-in methods to handle it gracefully, reducing the risk of NullPointerException and making APIs cleaner.
### ✅ If They Ask “When not to use Optional?”
You should mention:
We should avoid using Optional for fields in entities, DTOs, or method parameters — it’s best suited for method return types, especially when a value may or may not exist.
### ✅ If They Ask About Methods of Optional
You can mention a few key ones:
- of(), ofNullable(), empty() → for creation
- isPresent(), isEmpty() → to check value
- get(), orElse(), orElseGet(), orElseThrow() → to retrieve value
- map() and flatMap() → for transforming the contained value

---

# Page

## 1. What is a ReadWriteLock?
A ReadWriteLock is a synchronization mechanism that allows:
- Multiple threads to read simultaneously, but
- Only one thread to write at a time, and
- When a writer is writing, no reader or writer can access the resource.
It is implemented in Java by ReentrantReadWriteLock.
## 2. Basic Behavior
## 3. Why use ReadWriteLock instead of synchronized?
### synchronized
- Only one thread can access the synchronized block at a time (either read or write).
- All threads must wait even if they only want to read.
### ReadWriteLock
- Allows concurrent reads (no blocking between readers).
- Writers still get exclusive access to avoid data inconsistency.
✅ Best when reads are more frequent than writes.
## 4. Example: Basic ReadWriteLock Usage
```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockExample {

    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private int data = 0;

    public void readData() {
        lock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + " reading: " + data);
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            lock.readLock().unlock();
            System.out.println(Thread.currentThread().getName() + " finished reading");
        }
    }

    public void writeData(int value) {
        lock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + " writing...");
            data = value;
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " wrote: " + data);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            lock.writeLock().unlock();
            System.out.println(Thread.currentThread().getName() + " finished writing");
        }
    }
}
```
## 5. Shared Lock Behavior Across Methods
All functions using the same ReadWriteLock instance share locking behavior.
Example:
```java
class VotingSystem {
    private final ReadWriteLock lock = new ReentrantReadWriteLock();

    public void viewResults() {
        lock.readLock().lock();
        try { System.out.println("Reading results..."); }
        finally { lock.readLock().unlock(); }
    }

    public void viewSomething() {
        lock.readLock().lock();
        try { System.out.println("Viewing something else..."); }
        finally { lock.readLock().unlock(); }
    }

    public void endVoting() {
        lock.writeLock().lock();
        try { System.out.println("Ending voting..."); }
        finally { lock.writeLock().unlock(); }
    }
}
```
## 6. Scenarios
## 7. Example Run
```java
public static void main(String[] args) {
    VotingSystem system = new VotingSystem();
    Thread t1 = new Thread(system::viewResults, "T1");
    Thread t2 = new Thread(system::endVoting, "T2");
    Thread t3 = new Thread(system::viewSomething, "T3");

    t1.start();
    t3.start();
    t2.start();
}
```
### Output (sample)
```plain text
T1 reading results...
T3 viewing something else...
T2 trying to acquire WRITE lock... (blocked)
T1 finished reading.
T3 finished reading.
T2 ending voting... (now exclusive)
T2 finished ending voting.
```
## 8. How the Lock Applies
🧠 A ReadWriteLock works on the lock object, not on functions.
That means:
- If lock is shared across functions,
  → all methods using it are synchronized together (read/write control is shared).
- If each function had its own lock,
  → they would act independently (but might corrupt shared data).
## 9. Doubts Explained (from your earlier questions)
## 10. Summary Table
## 11. Visualization Analogy
Think of a library:
- Many people can read books together → readLock.
- But when a librarian wants to rearrange shelves (write) → everyone must leave → writeLock.
- No one else can enter until the librarian finishes rearranging.
## 12. Key Takeaways
✅ ReadWriteLock increases concurrency for read-heavy workloads.
✅ Writers always get exclusive access.
✅ Lock applies at the object level, not per method.
✅ Keep locking encapsulated inside the class.
✅ Use synchronized only if you have simple, low-concurrency needs.

---

# Page



---

