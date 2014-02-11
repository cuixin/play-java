## ReentrantLock和Synchronized在Jstack下的区别

-------

ReentrantLock和Synchronized的区别可以参看IBM的这篇文章，[猛戳这里](:http://www.ibm.com/developerworks/cn/java/j-jtp10264/index.html)

我的建议是，开发时采用Synchronized，万不得已性能提升的时候使用Reentrantlock，为什么这么做呢？

因为你在发生死锁的时候，Synchronized可以被jstack捕捉到，而Reentrantlock却不行。

下面是两个测试代码： 

这是使用ReentrantLock的:

```

import java.util.concurrent.locks.ReentrantLock;

public class RetreenLockTest {
    public static void main(String[] args) {
        final ReentrantLock objA = new ReentrantLock();
        final ReentrantLock objB = new ReentrantLock();

        new Thread() {
            public void run (){
                while (1==1) {
                    System.out.println ("Thread A synchronizing on objA");
                    objA.lock();
                    try {
                        System.out.println ("Thread A synchronized on objA");
                        System.out.println ("Thread A synchronizing on objB");
                        objB.lock();
                        try {

                            System.out.println ("Thread A synchronized on objB");
                        } finally {
                            objB.unlock();
                        }
                    } finally {
                        objA.unlock();
                    }
                }
            }
        }.start();

        new Thread() {
            public void run (){
                while (1==1) {
                    System.out.println ("Thread B synchronizing on objB");
                    objB.lock();
                    try {
                        System.out.println ("Thread B synchronized on objB");
                        System.out.println ("Thread B synchronizing on objA");
                        objA.lock();
                        try {
                            System.out.println ("Thread B synchronized on objA");
                        } finally {
                            objA.unlock();
                        }
                    } finally {
                        objB.unlock();
                    }
                }
            }
        }.start();
    }
}
```

```

~  jstack 1899
2014-02-11 12:55:43
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.45-b08 mixed mode):

"Attach Listener" daemon prio=5 tid=0x00007fa977005800 nid=0x5d03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"DestroyJavaVM" prio=5 tid=0x00007fa9740a5800 nid=0x1903 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Thread-1" prio=5 tid=0x00007fa9740a4800 nid=0x5b03 waiting on condition [0x00000001194b5000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007d568cf58> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:834)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:867)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1197)
	at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:214)
	at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:290)
	at RetreenLockTest$2.run(RetreenLockTest.java:41)

"Thread-0" prio=5 tid=0x00007fa9740a4000 nid=0x5903 waiting on condition [0x00000001193b2000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007d568cf88> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:834)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:867)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1197)
	at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:214)
	at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:290)
	at RetreenLockTest$1.run(RetreenLockTest.java:19)

"Monitor Ctrl-Break" daemon prio=5 tid=0x00007fa9740ff000 nid=0x5703 runnable [0x000000011905a000]
   java.lang.Thread.State: RUNNABLE
	at java.net.PlainSocketImpl.socketAccept(Native Method)
	at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:398)
	at java.net.ServerSocket.implAccept(ServerSocket.java:530)
	at java.net.ServerSocket.accept(ServerSocket.java:498)
	at com.intellij.rt.execution.application.AppMain$1.run(AppMain.java:82)
	at java.lang.Thread.run(Thread.java:744)

"Service Thread" daemon prio=5 tid=0x00007fa977008800 nid=0x5303 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" daemon prio=5 tid=0x00007fa977007800 nid=0x5103 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" daemon prio=5 tid=0x00007fa974803000 nid=0x4f03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" daemon prio=5 tid=0x00007fa976033800 nid=0x4d03 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" daemon prio=5 tid=0x00007fa977002000 nid=0x3903 in Object.wait() [0x00000001185f7000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d5505568> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
	- locked <0x00000007d5505568> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:189)

"Reference Handler" daemon prio=5 tid=0x00007fa976813000 nid=0x3703 in Object.wait() [0x00000001184f4000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d55050f0> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:503)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
	- locked <0x00000007d55050f0> (a java.lang.ref.Reference$Lock)

"VM Thread" prio=5 tid=0x00007fa976812800 nid=0x3503 runnable

"GC task thread#0 (ParallelGC)" prio=5 tid=0x00007fa97580d000 nid=0x2503 runnable

"GC task thread#1 (ParallelGC)" prio=5 tid=0x00007fa974003800 nid=0x2703 runnable

"GC task thread#2 (ParallelGC)" prio=5 tid=0x00007fa97400c000 nid=0x2903 runnable

"GC task thread#3 (ParallelGC)" prio=5 tid=0x00007fa976000000 nid=0x2b03 runnable

"GC task thread#4 (ParallelGC)" prio=5 tid=0x00007fa976001000 nid=0x2d03 runnable

"GC task thread#5 (ParallelGC)" prio=5 tid=0x00007fa976001800 nid=0x2f03 runnable

"GC task thread#6 (ParallelGC)" prio=5 tid=0x00007fa976002000 nid=0x3103 runnable

"GC task thread#7 (ParallelGC)" prio=5 tid=0x00007fa976002800 nid=0x3303 runnable

"VM Periodic Task Thread" prio=5 tid=0x00007fa977021000 nid=0x5503 waiting on condition

JNI global references: 127


Found one Java-level deadlock:
=============================
"Thread-1":
  waiting for ownable synchronizer 0x00000007d568cf58, (a java.util.concurrent.locks.ReentrantLock$NonfairSync),
  which is held by "Thread-0"
"Thread-0":
  waiting for ownable synchronizer 0x00000007d568cf88, (a java.util.concurrent.locks.ReentrantLock$NonfairSync),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007d568cf58> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:834)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:867)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1197)
	at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:214)
	at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:290)
	at RetreenLockTest$2.run(RetreenLockTest.java:41)
"Thread-0":
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007d568cf88> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:834)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:867)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1197)
	at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:214)
	at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:290)
	at RetreenLockTest$1.run(RetreenLockTest.java:19)

Found 1 deadlock.

```

上面根本不知道哪行代码出错了，只是知道死锁了。而换成Synchronized试下，下面是改成Synchronized的代码: 

```
public class SyncTest {
    public static void main(String[] args) {
        final Object objA = new Object();
        final Object objB = new Object();

        new Thread() {
            public void run (){
                while (1==1) {
                    System.out.println ("Thread A synchronizing on objA");
                    synchronized (objA){
                        System.out.println ("Thread A synchronized on objA");
                        System.out.println ("Thread A synchronizing on objB");
                        synchronized (objB){
                            System.out.println ("Thread A synchronized on objB");
                        }
                    }
                }
            }
        }.start();

        new Thread() {
            public void run (){
                while (1==1) {
                    System.out.println ("Thread B synchronizing on objB");
                    synchronized (objB){
                        System.out.println ("Thread B synchronized on objB");
                        System.out.println ("Thread B synchronizing on objA");
                        synchronized (objA){
                            System.out.println ("Thread B synchronized on objA");
                        }
                    }
                }
            }
        }.start();

    }
}
```

下面的输出详细的看到代码的死锁出现在哪行了  

```

jstack 1884
2014-02-11 12:51:35
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.45-b08 mixed mode):

"Attach Listener" daemon prio=5 tid=0x00007f8956002000 nid=0x3b07 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"DestroyJavaVM" prio=5 tid=0x00007f8954006000 nid=0x1903 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Thread-1" prio=5 tid=0x00007f8954005800 nid=0x5b03 waiting for monitor entry [0x000000011be00000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at SyncTest$2.run(SyncTest.java:32)
	- waiting to lock <0x00000007d568c660> (a java.lang.Object)
	- locked <0x00000007d568c670> (a java.lang.Object)

"Thread-0" prio=5 tid=0x00007f895782f000 nid=0x5903 waiting for monitor entry [0x000000011bcfd000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at SyncTest$1.run(SyncTest.java:17)
	- waiting to lock <0x00000007d568c670> (a java.lang.Object)
	- locked <0x00000007d568c660> (a java.lang.Object)

"Monitor Ctrl-Break" daemon prio=5 tid=0x00007f8955196000 nid=0x5703 runnable [0x000000011b9a5000]
   java.lang.Thread.State: RUNNABLE
	at java.net.PlainSocketImpl.socketAccept(Native Method)
	at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:398)
	at java.net.ServerSocket.implAccept(ServerSocket.java:530)
	at java.net.ServerSocket.accept(ServerSocket.java:498)
	at com.intellij.rt.execution.application.AppMain$1.run(AppMain.java:82)
	at java.lang.Thread.run(Thread.java:744)

"Service Thread" daemon prio=5 tid=0x00007f8955050000 nid=0x5303 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" daemon prio=5 tid=0x00007f8955808000 nid=0x5103 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" daemon prio=5 tid=0x00007f8955813000 nid=0x4f03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" daemon prio=5 tid=0x00007f8956003000 nid=0x4d03 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" daemon prio=5 tid=0x00007f895504c000 nid=0x3903 in Object.wait() [0x000000011af47000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d5505568> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
	- locked <0x00000007d5505568> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:189)

"Reference Handler" daemon prio=5 tid=0x00007f8955049800 nid=0x3703 in Object.wait() [0x000000011ae44000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d55050f0> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:503)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
	- locked <0x00000007d55050f0> (a java.lang.ref.Reference$Lock)

"VM Thread" prio=5 tid=0x00007f8955049000 nid=0x3503 runnable

"GC task thread#0 (ParallelGC)" prio=5 tid=0x00007f895580c800 nid=0x2503 runnable

"GC task thread#1 (ParallelGC)" prio=5 tid=0x00007f895580d000 nid=0x2703 runnable

"GC task thread#2 (ParallelGC)" prio=5 tid=0x00007f8955009800 nid=0x2903 runnable

"GC task thread#3 (ParallelGC)" prio=5 tid=0x00007f895500a000 nid=0x2b03 runnable

"GC task thread#4 (ParallelGC)" prio=5 tid=0x00007f895500b000 nid=0x2d03 runnable

"GC task thread#5 (ParallelGC)" prio=5 tid=0x00007f895500b800 nid=0x2f03 runnable

"GC task thread#6 (ParallelGC)" prio=5 tid=0x00007f895500c000 nid=0x3103 runnable

"GC task thread#7 (ParallelGC)" prio=5 tid=0x00007f895500c800 nid=0x3303 runnable

"VM Periodic Task Thread" prio=5 tid=0x00007f895581d000 nid=0x5503 waiting on condition

JNI global references: 127


Found one Java-level deadlock:
 =============================
"Thread-1":
  waiting to lock monitor 0x00007f8955810368 (object 0x00000007d568c660, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007f89558102b8 (object 0x00000007d568c670, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
 ===================================================
"Thread-1":
	at SyncTest$2.run(SyncTest.java:32)
	- waiting to lock <0x00000007d568c660> (a java.lang.Object)
	- locked <0x00000007d568c670> (a java.lang.Object)
"Thread-0":
	at SyncTest$1.run(SyncTest.java:17)
	- waiting to lock <0x00000007d568c670> (a java.lang.Object)
	- locked <0x00000007d568c660> (a java.lang.Object)

Found 1 deadlock.

```