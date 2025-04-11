# 95-702 Distributed Systems for ISM                         
# **Lab11: Threads**

Complete Lab11_Quiz on Canvas as you work on this lab. See the checkered flags for references to the quiz questions. In addition, each section has several questions that you should answer for yourself - these are not collected and graded, but they will serve as the basis for exam questions.

The motivation for this lab comes from the Coulouris text on distributed systems. To understand transactions and transaction processing, it is essential to be exposed to threads, communicating threads, deadlock, and isolation. The first part of this lab, Part 1, illustrates **deadlock** - when two or more threads cannot continue running because they each have a resource the other wants, and neither will give up the resource they have. The second part, Part 2, explores the important issue of **isolation** - keeping data locked when more than one thread shares that data. The third part, Part 3, introduces the student to communicating threads via **wait** and **notify** - Java instructions that force a thread to sleep or wakeup based on some boolean condition.

There are questions about each part (see below) in addition to the quiz. You should try to answer all those questions, but only the lab quiz is graded.

# **Part 1 DeadLock**

Study the following code, copy it into IntelliJ, and get it running.

```


// Working with deadlock.
public class DeadLockLabPart1 {

    final Object resource1 = new Object();
    final Object resource2 = new Object();

    final int n = 100;

    Thread t1 = new Thread( new Runnable() {
        public void run() {
            synchronized (resource1) {
                for (int x = 1; x <= n; x++) {
                }
                synchronized (resource2) {
                    System.out.println();
                    for (int i = 1; i <= 10; i++) {
                        System.out.print(" t1: " + i);
                    }

                }
            }
        }
    });

    Thread t2 = new Thread( new Runnable() {
        public void run() {
                synchronized (resource2) {
                    for (int x = 1; x <= n; x++) {
                    }
                    synchronized (resource1) {
                        System.out.println();
                        for (int i = 1; i <= 10; i++) {
                            System.out.print(" t2: " + i);

                    }
                }
            }
        }
    });

    public void foo() {
        // Start up two threads that may become deadlocked.
        t1.start();
        t2.start();
    }
    public static void main(String[] args) {
	     System.out.println("About to startup and run two threads.");
	     new DeadLockLabPart1().foo();
       System.out.println("Startup complete but threads may still be running.");
    }
}

```

# **Part 1 Questions**

Study the code above, run it on IntelliJ, and answer the following questions (no submission required).

1. What is the largest value of n (choosing n as a power of 10) that you can use that mostly stays out of deadlock? Your answer only needs to be approximate.

2. What is the smallest value of n (choosing n as a power of 10) that you can use to reliably reach deadlock? Your answer only needs to be approximate.

### :checkered_flag: Answer question 1 on the Canvas quiz named Lab11_Quiz.


3. How does this system behave if t1 and t2 access their shared resources in the exact same order? Make the necessary changes to the code and test it. Explain why you see what you see. Does the system still reach deadlock? Why or why not?

### :checkered_flag: Answer question 2 on the Canvas quiz named Lab11_Quiz.

4. Remove all of the synchronization from the code. Remove the "synchronized (resourceX)" coding. What do you see when you run the program?

# **Part 2 Isolation**

Study the following code, copy it into IntelliJ, and get it running.

```

// Isolation

// A shared account
class Account {
    private double balance = 0;

    public synchronized void deposit(double amount) {
        balance = balance + amount;
    }
    public synchronized void  withdraw(double amount) {
        balance = balance - amount;
    }
    public synchronized double getBalance() {
        return balance;
    }

}

public class SynchronizedLabPart2 {

    int n = 100;

    Account acct = new Account();

    // deposit 1,2,3,...,n to the account
    Thread t1 = new Thread( new Runnable() {
        public void run() {
            for(int j = 1; j <= n; j++) {
                acct.deposit(1.0);
            }
        }});

    // withdraw 1,2,3,...,n from the account
    Thread t2 = new Thread( new Runnable() {
            public void run() {
                for(int j = 1; j <= n; j++) {
                    acct.withdraw(1.0);
                }
            }});

    public void doBanking() {

        t1.start();
        t2.start();


        // wait for t1 to finish
        try { t1.join(); } catch(InterruptedException e){}
        // wait for t2 to finish
        try { t2.join(); } catch(InterruptedException e){}
        System.out.println("Done waiting");
    }

    public static void main(String[] args) {
         SynchronizedLabPart2 m = new SynchronizedLabPart2();
	     m.doBanking();
	     System.out.println("Balance should be 0.0  balance = " + m.acct.getBalance());

    }
}

```


# **Part 2 Questions**

Study the code above, run it on IntelliJ, and answer the following questions (no submission required).

5. Experimenting only with the value n, does the program work with n = 10, 100, 1000, 10000, 100000, 100000000 ?

6. Swap the lines "t1.start();" with "t2.start()". Does the program work with n = 10, 100, 1000, 10000, 100000, 100000000 ? Explain what you find.

### :checkered_flag: Answer question 3 on the Canvas quiz named Lab11_Quiz.


7. Remove the synchronized keywords from the Account class methods. Does the program work with  n = 10 ? Describe what happens.

8. With the synchronized keywords removed from the Account class methods. Does the program work with  n = 100, 1000, 10000, 100000, 1000000 ? Describe what happens.

### :checkered_flag: Answer question 4 on the Canvas quiz named Lab11_Quiz.

# **Part 3 Communicating threads**

Study the following code, copy it into IntelliJ, and get it running.

```

import java.util.*;

class QueueWithWaitAndNotify {

    // A linked list is used to hold the queue.
    LinkedList q = new LinkedList();

    // Add an element to the end of the line
    public synchronized void addAtEnd(Object o) {
        q.addLast(o);
        this.notify();
    }

    // Remove the first in line
    public synchronized Object removeFromFront() {
        while(q.size() == 0) {
            try { this.wait(); }
            catch (InterruptedException e) { /* ignore this exception */ }
        }
        return q.removeFirst();
    }
}

public class WaitAndNotify {

    int n = 100;

    QueueWithWaitAndNotify myQueue = new QueueWithWaitAndNotify();

    // add at end of queue 1,2,...,n
    Thread t1 = new Thread( new Runnable() {
        public void run() {
            for(int j = 1; j <= n; j++) {
                myQueue.addAtEnd(j);
            }
        }
    });

    // remove n elements from the queue
    Thread t2 = new Thread( new Runnable() {
        public void run() {
            for(int j = 1; j <= n; j++) {
                System.out.println(myQueue.removeFromFront());
            }
        }
    });

    public void playWithQueue()
    {
        t1.start();
        t2.start();
        // Wait for both threads to complete.
        try { t1.join(); } catch(Exception e) {}
        try { t2.join(); } catch(Exception e) {}

    }

    public static void main(String args[]) {
        WaitAndNotify w = new WaitAndNotify();
        w.playWithQueue();
        System.out.println("The wait and notify demonstration is complete");
    }
}

```

# **Part 3 Questions**

Study the code above, run it on IntelliJ, and answer the following questions (no submission required).

9. Currently, we are adding n elements to the queue and removing n elements from the queue. Explain what happens if we add 2n (1,2,3,...,n,n+1,...,2n) elements to the queue rather than n.The removal thread is left unchanged.

10. Currently, we are adding n elements to the queue and removing n elements from the queue. Explain what happens if we only add n-1 elements to the queue rather than n. The removal thread is left unchanged.

### :checkered_flag: Answer question 5 on the Canvas quiz named Lab11_Quiz.


11. This is the same as question 10 but after adding 1,2,3,...,n-1 to the queue, we would like to put the thread to sleep for 5 seconds and then add n to the queue. The queue ends up taking on 1,2,3,...,n but it does so in two parts. First, it adds 1,2,3,...,n-1 to the queue, and then it sleeps. When it awakes, it adds one value, n, to the queue. Use this code
```
 try{ java.lang.Thread.sleep(5000); } catch(Exception e) {}
```
to put the thread to sleep. Explain what happens.