# 1. Java Threads

**Lecture Summary**: In this lecture, we learned the concept of threads as lower-level building blocks for concurrent programs. A unique aspect of Java compared to prior mainstream programming languages is that Java included the notions of threads (as instances of the `java.lang.Thread` class) in its language definition right from the start.

When an instance of `Thread` is created (via a `new` operation), it does not start executing right away; instead, it can only start executing when its `start()` method is invoked. The statement or computation to be executed by the thread is specified as a parameter to the constructor.

The Thread class also includes a wait operation in the form of a `join()` method. If thread `t0` performs a `t1.join()` call, thread `t0` will be forced to wait until thread `t1` completes, after which point it can safely access any values computed by thread `t1`. Since there is no restriction on which thread can perform a `join` on which other thread, it is possible for a programmer to erroneously create a deadlock cycle with `join` operations. (A deadlock occurs when two threads wait for each other indefinitely, so that neither can make any progress.)

The `Thread` class also defines a number of methods for thread management. These include `static` methods which include information about, or affect the status of, the thread invoking the method. There are other non-static methods as well which are invoked from other threads involved in managing the thread and the `Thread` object.

### Defining and starting a thread
An application that creates an instance of `Thread` must also provide the code that executes within that thread. There are two ways to provide the code for execution:
- By implementing the `Runnable` interface.

```java
// Example illustrating thread creation and task execution by implementing java.lang.Runnable interface.

public class MyTask implements Runnable {
    public void run() {
        System.out.println("Hello from a thread! (Running MyTask implementing java.lang.Runnable)");
    }

    public static void main(String args[]) {
        Thread t0 = new Thread(new MyTask());
        t0.start();
    }
};
```

Output:

	Hello from a thread! (Running MyTask implementing java.lang.Runnable)
  
  
- By extending the `Thread` class

`java.lang.Thread` itself implements `java.lang.Runnable`, but its `run()` method does nothing. A class which subclasses `java.lang.Thread` can implement its own `run()` method.

```java
// Example illustrating the thread creation and task execution by extending java.lang.Thread class and overriding it's run() method.

public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Hello from MyThread! (Extended from java.lang.Thread)");
    }

    public static void main(String args[]) {
        Thread t0 = new MyThread();
        t0.start();
    }
};
```

Output:

	Hello from MyThread! (Extended from java.lang.Thread)



### Waiting on a thread

`join()` method is used when a thread wants another thread to finish executing before the calling thread can proceed further. A thread `t0` which invokes the `join` operation on another thread `t1` is said to be waiting on `t1`.

```java
// Example illustrating the use of join() method of java.lang.Thread class

public class MyThread {

    public static void main(String args[]) {
        Thread t1 = new Thread(new MyTask(), "Thread 1");
        Thread t2 = new Thread(new MyTask(), "Thread 2");

        // Start t1 immediately
         t1.start();
         try {
             t1.join();
         } catch (InterruptedException ie) {
             ie.printStackTrace();
         }

         // Start t2 once t1 has finished execution
         t2.start();
         try {
             t2.join();
         } catch (InterruptedException ie) {
             ie.printStackTrace();
         }

         System.out.println("Both threads finished execution");
    }
};

class MyTask implements Runnable {

    @Override
    public void run() {
        Thread t = Thread.currentThread();
        System.out.println("Thread started: " + t.getName());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException ie) {
            ie.printStackTrace();
        }
        System.out.println("Thread ended: " + t.getName());
    }
};
```
Output:

    Thread started: Thread 1
    Thread ended: Thread 1
    Thread started: Thread 2
    Thread ended: Thread 2
    Both threads finished execution


### Deadlock
Deadlock is a condition in which two (or more) threads cannot make progress because one thread is holding a resource and is waiting for another resource which is held by another waiting process. 

```java
// Example illustrating deadlock situation

public class Deadlock {
    static String resource1 = "resource1";
    static String resource2 = "resource2";

    public static void main(String args[]) {
        // t1 first tries to lock resource1 and then resource2
        Thread t1 = new Thread("Thread 1") {
            public void run() {
                System.out.println("Started: " + Thread.currentThread().getName());
                synchronized (resource1) {
                    System.out.println(Thread.currentThread().getName() + ": Locked resource 1");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException ie) {
                        ie.printStackTrace();
                    }

                    synchronized (resource2) {
                        System.out.println(Thread.currentThread().getName() + ": Locked resource 2");
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException ie) {
                            ie.printStackTrace();
                        }
                    }
                }
                System.out.println("Ended: Thread 1");
            }
        };

        //t2 first tries to lock resource2 and then resource1
        Thread t2 = new Thread("Thread 2") {
            public void run() {
                System.out.println("Started: " + Thread.currentThread().getName());
                synchronized (resource2) {
                    System.out.println(Thread.currentThread().getName() + ": Locked resource 2");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException ie) {
                        ie.printStackTrace();
                    }

                    synchronized (resource1) {
                        System.out.println(Thread.currentThread().getName() + ": Locked resource 1");
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException ie) {
                            ie.printStackTrace();
                        }
                    }
                }
                System.out.println("Ended: Thread 1");
            }
        };

        t1.start();
        t2.start();
    }
}
```
Output:

    Started: Thread 2
    Started: Thread 1
    Thread 2: Locked resource 2
    Thread 1: Locked resource 1
    
### Further reading:
- Wikipedia article on [Threads](https://en.wikipedia.org/wiki/Thread_(computing) "Threads")
- [Java documentation on Thread class](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html "Java documentation on Thread class")

<br>
<br>

# 2. Structured Locks

**Lecture Summary**: In this lecture, we learned about structured locks, and how they can be implemented using `synchronized` statements and methods in Java. Structured locks can be used to enforce mutual exclusion and avoid data races, as illustrated by the `incr()` method in the `A.count` example, and the `insert()` and `remove()` methods in the the `Buffer` example. A major benefit of structured locks is that their *acquire* and *release* operations are implicit, since these operations are automatically performed by the Java runtime environment when entering and exiting the scope of a `synchronized` statement or method, even if an exception is thrown in the middle.

We also learned about `wait()` and `notify()` operations that can be used to block and resume threads that need to wait for specific conditions. For example, a producer thread performing an `insert()` operation on a bounded buffer can call `wait()` when the buffer is full, so that it is only unblocked when a consumer thread performing a `remove()` operation calls `notify()`. Likewise, a consumer thread performing a `remove()` operation on a bounded buffer can call `wait()` when the buffer is empty, so that it is only unblocked when a producer thread performing an `insert()` operation calls `notify()`. Structured locks are also referred to as  *intrinsic locks* or *monitors*.

### Data race
A data race is a condition in which two or more threads concurrently access a shared memory location and atleast one of the accesses is for writing, and the threads are not using any exclusive locks for controlling their accesses to that memory location.

```java
// Example illustrating data race.

public class DataRace {

    private static final int MAX_THREADS = 5;
    public static void main(String args[]) {
        MyCounter counter = new MyCounter();
        Thread[] threads = new Thread[MAX_THREADS];
        for(int i = 0; i < MAX_THREADS; i++) {
            threads[i] = new Thread(new MyTask(counter));
            threads[i].start();
        }
        for(int i = 0; i < MAX_THREADS; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException ie) {
                ie.printStackTrace();
            }
        }

        System.out.println("Counter value (actual): " + counter.getValue() + " Counter value (expected): " + MAX_THREADS);
    }
};

class MyCounter {
    private int counter = 0;

    public void increment() {
        this.counter++;
    }

    public int getValue() {
        return this.counter;
    }
}

class MyTask implements Runnable {
    private MyCounter counter;
    
    public MyTask(MyCounter counter) {
        this.counter = counter;
    }

    @Override
    public void run() {
        try {
            Thread.sleep(10);
        } catch (InterruptedException ie) {
            ie.printStackTrace();
        }
        this.counter.increment();
    }
};
```

Output:

	Counter value (actual): 3 Counter value (expected): 5

### Synchronized construct 
The `synchronized` construct ensures that no two threads execute the synchronized block at the same time. Any accesses to the resource on which the synchronized construct is placed happen in mutual exclusion.

Syntax:

    synchronized (Object resource) {
    	// Some computation on the shared resource
    }

The `synchronized` construct is responsible for acquiring the lock on the shared resource before entering the `synchronized` block and releasing the lock upon exiting the block. It also provides a guarantee of lock release even when an exception occurs inside the synchronized block.

The above data race condition can be prevented by synchronizing the accesses to the shared resource `counter` as follows:

```java
// Example illustrating the use of synchronized blocks to prevent data race.

public class DataRace {

    private static final int MAX_THREADS = 5;
    public static void main(String args[]) {
        MyCounter counter = new MyCounter();
        Thread[] threads = new Thread[MAX_THREADS];
        for(int i = 0; i < MAX_THREADS; i++) {
            threads[i] = new Thread(new MyTask(counter));
            threads[i].start();
        }
        for(int i = 0; i < MAX_THREADS; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException ie) {
                ie.printStackTrace();
            }
        }

        System.out.println("Counter value (actual): " + counter.getValue() + " Counter value (expected): " + MAX_THREADS);
    }
};

class MyCounter {
    private int counter = 0;

    public void increment() {
        synchronized (this) {
            this.counter++;
        }
    }

    public int getValue() {
        return this.counter;
    }
}

class MyTask implements Runnable {
    
    private MyCounter counter;
    
    public MyTask(MyCounter counter) {
        this.counter = counter;
    }

    @Override
    public void run() {
        try {
            Thread.sleep(10);
        } catch (InterruptedException ie) {
            ie.printStackTrace();
        }
        this.counter.increment();
    }
};
```

Output:

	Counter value (actual): 5 Counter value (expected): 5

### wait() and notify()

### Producer-Consumer example
```java
// Example producer-consumer program

public class ProducerConsumer {

    public static void main(String[] args) {
        Buffer buf = new Buffer();
        Thread producer = new Producer(buf);
        Thread consumer = new Consumer(buf);
    
        producer.start();
        consumer.start();
    }
};

class Buffer {
    private static final int MAX_BUFFER_LENGTH = 5;
    private String[] messages;
    private int in;
    private int out;
    private int n_elem;

    public Buffer() {
        this.messages = new String[MAX_BUFFER_LENGTH];
        this.in = 0;
        this.out = 0;
        this.n_elem = 0;
    }

    public synchronized void insert(String message) {
        while(n_elem >= MAX_BUFFER_LENGTH) {
            try {
                wait();
            } catch (InterruptedException ie) {
                // Do nothing
            }
        }

        messages[in] = message;
        in = (in + 1) % MAX_BUFFER_LENGTH;
        n_elem++;
        notifyAll();
    }

    public synchronized String remove() {
        while(n_elem <= 0) {
            try {
                wait();
            } catch (InterruptedException ex) {
                // Do nothing
            }
        }

        String msg = messages[out];
        out = (out + 1) % MAX_BUFFER_LENGTH;
        n_elem--;
        notifyAll();
        return msg;
    }

};

class Producer extends Thread {
    private Buffer buf;

    public Producer(Buffer buf) {
        this.buf = buf;
    }

    @Override
    public void run() {
        String[] messages = {"I", "am", "a", "producer", "thread", "and", "my", "task", "is", "to", "produce", "strings", "DONE"};
        for(int i = 0; i < messages.length; i++) {
            buf.insert(messages[i]);
            try {
                Thread.sleep(100);
            } catch (InterruptedException ie) {
                ie.printStackTrace();
            }
        }
    }
};

class Consumer extends Thread {
    private Buffer buf;

    public Consumer(Buffer buf) {
        this.buf = buf;
    }

    @Override
    public void run() {
        for(String message = buf.remove(); !message.equals("DONE"); message = buf.remove()) {
            System.out.println("RECEIVED: " + message);
            try {
                Thread.sleep(100);
            } catch (InterruptedException ie) {
                ie.printStackTrace();
            }
        }
    }
};
```
Output:

    RECEIVED: I
    RECEIVED: am
    RECEIVED: a
    RECEIVED: producer
    RECEIVED: thread
    RECEIVED: and
    RECEIVED: my
    RECEIVED: task
    RECEIVED: is
    RECEIVED: to
    RECEIVED: produce
    RECEIVED: strings
```
