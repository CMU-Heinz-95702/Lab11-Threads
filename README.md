# 95-702 Distributed Systems for ISM                         
# Lab12-Threads

The motivation for this lab comes from the Coulouris text on distributed systems. To understand transactions and transaction processing, it is essential to be exposed to threads, communicating threads, deadlock, and isolation. The first part of this lab, Part 1, illustrates deadlock. The second part, Part 2, explores the important issue of isolation. The third part, Part 3, introduces the student to communicating threads via wait and notify.

You will submit a single ".pdf" file to your TA that includes your answers to the questions from each part.

# Part 1 DeadLock

Study the following code, copy it into IntelliJ, and get it running.

```


// Working with deadlock.
public class DeadLockLabPart1 {

    final Object resource1 = new Object();
    final Object resource2 = new Object();

    final int n = 1000;

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

# Part 1 Questions

Study the code above, run it on IntelliJ, and write down answers to the following questions.

1. What is the largest value of n (choosing n as a power of 10) that you can use that mostly stays out of deadlock? Your answer only needs to be approximate.

2. What is the smallest value of n (choosing n as a power of 10) that you can use to reliably reach deadlock? Your answer only needs to be approximate.

3. How does this system behave if t1 and t2 access their shared resources in the exact same order? Make the necessary changes to the code and test it. Explain why you see what you see. Does the system still reach deadlock? Why or why not?

4. Remove all of the synchronization from the code. Remove the "synchronized (resourceX)" coding. What do you see when you run the program?

:checkered_flag:**This is the checkpoint for this lab.**


# Part 2 Isolation

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

    int n = 1000;

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


# Part 2 Questions   

Study the code above, run it on IntelliJ, and write down answers to the following questions.

5. Experimenting only with the value n, does the program work with n = 10, 100, 1000, 10000, 100000, 100000000 ?

6. Swap the lines "t1.start();" with "t2.start()". Does the program work with n = 10, 100, 1000, 10000, 100000, 100000000 ? Explain what you find.

7. Remove the synchronized keywords from the Account class methods. Does the program work with  n = 10 ? Describe what happens.

8. With the synchronized keywords removed from the Account class methods. Does the program work with  n = 100, 1000, 10000, 100000, 1000000 ? Describe what happens.



# Part 3 Communicating threads

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

# Part 3 Questions

Study the code above, run it on IntelliJ, and write down answers to the following questions.

9. Currently, we are adding n elements to the queue and removing n elements from the queue. Explain what happens if we add 2n (1,2,3,...,n,n+1,...,2n) elements to the queue rather than n.The removal thread is left unchanged.

10. Currently, we are adding n elements to the queue and removing n elements from the queue. Explain what happens if we only add n-1 elements to the queue rather than n. The removal thread is left unchanged.

11. This is the same as question 10 but after adding 1,2,3,...,n-1 to the queue, we would like to put the thread to sleep for 5 seconds and then add n to the queue. The queue ends up taking on 1,2,3,...,n but it does so in two parts. First, it adds 1,2,3,...,n-1 to the queue, and then it sleeps. When it awakes, it adds one value, n, to the queue. Use this code
```
 try{ java.lang.Thread.sleep(5000); } catch(Exception e) {}
```
to put the thread to sleep. Explain what happens.

:checkered_flag:**Provide to your TA a single PDF file with your answers to the questions one through eleven. Name the pdf file \<yourandrewid\>Lab12.pdf**
