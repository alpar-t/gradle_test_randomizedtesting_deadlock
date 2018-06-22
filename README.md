```
git clone git@github.com:atorok/elasticsearch.git
cd elasticsearch
git checkout feature/31496_run_test_with_gradle
./gradlew :server:test --max-workers=1
```

Hags at:
```
> Task :server:processTestResources
> Task :server:testClasses
> Task :server:test
```

This inspired this reproduction:
```
git checkout git@github.com:atorok/gradle_test_randomizedtesting_deadlock.git
cd gradle_test_randomizedtesting_deadlock
./gradlew test
```

Thread dumps from server:test elatic repo
-----------------------------------------
```
[:~/work/elastic/elasticsearch] feature/31496_run_test_with_gradle ± jps
19649 GradleWorkerMain
20611 Jps
18118 GradleWrapperMain
18248 GradleWorkerMain
888 Main
2427 RemoteMavenServer
18175 GradleDaemon
```

A worker:
```
[:~/work/elastic/elasticsearch] feature/31496_run_test_with_gradle 1 ± jstack 18248
2018-06-22 16:42:33
Full thread dump OpenJDK 64-Bit Server VM (10.0.1+10 mixed mode):

Threads class SMR info:
_java_thread_list=0x00007fcf6c0028e0, length=16, elements={
0x00007fd00c011000, 0x00007fd00c249000, 0x00007fd00c24b000, 0x00007fd00c254800,
0x00007fd00c25e800, 0x00007fd00c260800, 0x00007fd00c262800, 0x00007fd00c264800,
0x00007fd00c266800, 0x00007fd00c2f3000, 0x00007fd00c30c000, 0x00007fd00cbdc000,
0x00007fd00cc0a000, 0x00007fd00cba7800, 0x00007fd00cba9000, 0x00007fcf6c001000
}

"main" #1 prio=5 os_prio=0 tid=0x00007fd00c011000 nid=0x4749 waiting on condition  [0x00007fd014df7000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000005cad8a3d0> (a java.util.concurrent.CountDownLatch$Sync)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(java.base@10.0.1/AbstractQueuedSynchronizer.java:883)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(java.base@10.0.1/AbstractQueuedSynchronizer.java:1037)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(java.base@10.0.1/AbstractQueuedSynchronizer.java:1343)
	at java.util.concurrent.CountDownLatch.await(java.base@10.0.1/CountDownLatch.java:232)
	at org.gradle.process.internal.worker.request.WorkerAction.execute(WorkerAction.java:69)
	at org.gradle.process.internal.worker.request.WorkerAction.execute(WorkerAction.java:37)
	at org.gradle.process.internal.worker.child.ActionExecutionWorker.execute(ActionExecutionWorker.java:83)
	at org.gradle.process.internal.worker.child.ActionExecutionWorker.execute(ActionExecutionWorker.java:35)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:119)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:64)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.run(GradleWorkerMain.java:62)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.main(GradleWorkerMain.java:67)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007fd00c249000 nid=0x4769 waiting on condition  [0x00007fcfa4a6f000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ref.Reference.waitForReferencePendingList(java.base@10.0.1/Native Method)
	at java.lang.ref.Reference.processPendingReferences(java.base@10.0.1/Reference.java:174)
	at java.lang.ref.Reference.access$000(java.base@10.0.1/Reference.java:44)
	at java.lang.ref.Reference$ReferenceHandler.run(java.base@10.0.1/Reference.java:138)

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007fd00c24b000 nid=0x476a in Object.wait()  [0x00007fcfa496e000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000005cad50e58> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:151)
	- waiting to re-lock in wait() <0x00000005cad50e58> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:172)
	at java.lang.ref.Finalizer$FinalizerThread.run(java.base@10.0.1/Finalizer.java:216)

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007fd00c254800 nid=0x476b runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007fd00c25e800 nid=0x476c waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C2 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007fd00c260800 nid=0x476d waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C2 CompilerThread2" #7 daemon prio=9 os_prio=0 tid=0x00007fd00c262800 nid=0x476e waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C1 CompilerThread3" #8 daemon prio=9 os_prio=0 tid=0x00007fd00c264800 nid=0x476f waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"Sweeper thread" #9 daemon prio=9 os_prio=0 tid=0x00007fd00c266800 nid=0x4770 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #10 daemon prio=9 os_prio=0 tid=0x00007fd00c2f3000 nid=0x4771 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Common-Cleaner" #11 daemon prio=8 os_prio=0 tid=0x00007fd00c30c000 nid=0x4773 in Object.wait()  [0x00007fcf57dfb000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000005cad56680> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:151)
	- waiting to re-lock in wait() <0x00000005cad56680> (a java.lang.ref.ReferenceQueue$Lock)
	at jdk.internal.ref.CleanerImpl.run(java.base@10.0.1/CleanerImpl.java:148)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)
	at jdk.internal.misc.InnocuousThread.run(java.base@10.0.1/InnocuousThread.java:134)

"Memory manager" #13 prio=5 os_prio=0 tid=0x00007fd00cbdc000 nid=0x4779 waiting on condition  [0x00007fcf574ce000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000005cabe8f40> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(java.base@10.0.1/LockSupport.java:234)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(java.base@10.0.1/AbstractQueuedSynchronizer.java:2117)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:1182)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:899)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:53316 to /0:0:0:0:0:0:0:1:41677 workers" #14 prio=5 os_prio=0 tid=0x00007fd00cc0a000 nid=0x4782 waiting on condition  [0x00007fcf571c8000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000005cb369090> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$Handler.run(MessageHub.java:393)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:53316 to /0:0:0:0:0:0:0:1:41677 workers Thread 2" #15 prio=5 os_prio=0 tid=0x00007fd00cba7800 nid=0x4783 waiting on condition  [0x00007fcf56ec7000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000005cad531f8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionDispatch.run(MessageHub.java:314)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:53316 to /0:0:0:0:0:0:0:1:41677 workers Thread 3" #16 prio=5 os_prio=0 tid=0x00007fd00cba9000 nid=0x4784 runnable  [0x00007fcf56dc6000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.EPollArrayWrapper.epollWait(java.base@10.0.1/Native Method)
	at sun.nio.ch.EPollArrayWrapper.poll(java.base@10.0.1/EPollArrayWrapper.java:265)
	at sun.nio.ch.EPollSelectorImpl.doSelect(java.base@10.0.1/EPollSelectorImpl.java:92)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(java.base@10.0.1/SelectorImpl.java:89)
	- locked <0x00000005cad5a370> (a sun.nio.ch.Util$2)
	- locked <0x00000005cad5a380> (a java.util.Collections$UnmodifiableSet)
	- locked <0x00000005cad5a328> (a sun.nio.ch.EPollSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:100)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:104)
	at org.gradle.internal.remote.internal.inet.SocketConnection$SocketInputStream.read(SocketConnection.java:178)
	at com.esotericsoftware.kryo.io.Input.fill(Input.java:139)
	at com.esotericsoftware.kryo.io.Input.require(Input.java:159)
	at com.esotericsoftware.kryo.io.Input.readByte(Input.java:255)
	at org.gradle.internal.serialize.kryo.KryoBackedDecoder.readByte(KryoBackedDecoder.java:80)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:63)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:52)
	at org.gradle.internal.remote.internal.inet.SocketConnection.receive(SocketConnection.java:79)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionReceive.run(MessageHub.java:263)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Attach Listener" #17 daemon prio=9 os_prio=0 tid=0x00007fcf6c001000 nid=0x586e waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"VM Thread" os_prio=0 tid=0x00007fd00c23f000 nid=0x4768 runnable  

"GC Thread#0" os_prio=0 tid=0x00007fd00c027800 nid=0x474b runnable  

"GC Thread#1" os_prio=0 tid=0x00007fd00c029000 nid=0x474c runnable  

"GC Thread#2" os_prio=0 tid=0x00007fd00c02b000 nid=0x474d runnable  

"GC Thread#3" os_prio=0 tid=0x00007fd00c02c800 nid=0x474e runnable  

"GC Thread#4" os_prio=0 tid=0x00007fd00c02e000 nid=0x474f runnable  

"GC Thread#5" os_prio=0 tid=0x00007fd00c030000 nid=0x4750 runnable  

"GC Thread#6" os_prio=0 tid=0x00007fd00c031800 nid=0x4751 runnable  

"GC Thread#7" os_prio=0 tid=0x00007fd00c033800 nid=0x4752 runnable  

"GC Thread#8" os_prio=0 tid=0x00007fd00c035000 nid=0x4753 runnable  

"GC Thread#9" os_prio=0 tid=0x00007fd00c036800 nid=0x4754 runnable  

"G1 Main Marker" os_prio=0 tid=0x00007fd00c061800 nid=0x4756 runnable  

"G1 Conc#0" os_prio=0 tid=0x00007fd00c063800 nid=0x4757 runnable  

"G1 Conc#1" os_prio=0 tid=0x00007fd00c065000 nid=0x4758 runnable  

"G1 Conc#2" os_prio=0 tid=0x00007fd00c066800 nid=0x4759 runnable  

"G1 Refine#0" os_prio=0 tid=0x00007fd00c1c9800 nid=0x475d runnable  

"G1 Refine#1" os_prio=0 tid=0x00007fd00c1cb800 nid=0x475e runnable  

"G1 Refine#2" os_prio=0 tid=0x00007fd00c1cd000 nid=0x475f runnable  

"G1 Refine#3" os_prio=0 tid=0x00007fd00c1cf000 nid=0x4760 runnable  

"G1 Refine#4" os_prio=0 tid=0x00007fd00c1d0800 nid=0x4761 runnable  

"G1 Refine#5" os_prio=0 tid=0x00007fd00c1d2800 nid=0x4762 runnable  

"G1 Refine#6" os_prio=0 tid=0x00007fd00c1d4000 nid=0x4763 runnable  

"G1 Refine#7" os_prio=0 tid=0x00007fd00c1d6000 nid=0x4764 runnable  

"G1 Refine#8" os_prio=0 tid=0x00007fd00c1d7800 nid=0x4765 runnable  

"G1 Refine#9" os_prio=0 tid=0x00007fd00c1d9800 nid=0x4766 runnable  

"G1 Young RemSet Sampling" os_prio=0 tid=0x00007fd00c1db000 nid=0x4767 runnable  
"VM Periodic Task Thread" os_prio=0 tid=0x00007fd00c2f5800 nid=0x4772 waiting on condition  

JNI global references: 11
```

Annother Gradle Worker:
```
[:~/work/elastic/elasticsearch] feature/31496_run_test_with_gradle ± jstack 19649
2018-06-22 16:44:16
Full thread dump OpenJDK 64-Bit Server VM (11-ea+18 mixed mode):

Threads class SMR info:
_java_thread_list=0x00007f859c001e90, length=11, elements={
0x00007f85e0014000, 0x00007f85e0270800, 0x00007f85e0274800, 0x00007f85e0287000,
0x00007f85e0289800, 0x00007f85e028b800, 0x00007f85e028d800, 0x00007f85e0353800,
0x00007f85e0368800, 0x00007f85e0812800, 0x00007f859c001000
}

"main" #1 prio=5 os_prio=0 tid=0x00007f85e0014000 nid=0x4cc3 waiting on condition  [0x00007f85ea06f000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@11-ea/Native Method)
	- parking to wait for  <0x00000000fa22c2f8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(java.base@11-ea/LockSupport.java:234)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(java.base@11-ea/AbstractQueuedSynchronizer.java:2123)
	at java.util.concurrent.ThreadPoolExecutor.awaitTermination(java.base@11-ea/ThreadPoolExecutor.java:1454)
	at org.gradle.internal.concurrent.ManagedExecutorImpl.stop(ManagedExecutorImpl.java:81)
	at org.gradle.internal.concurrent.DefaultExecutorFactory$TrackedManagedExecutor.stop(DefaultExecutorFactory.java:77)
	at org.gradle.internal.concurrent.ManagedExecutorImpl.stop(ManagedExecutorImpl.java:72)
	at org.gradle.internal.remote.internal.hub.MessageHub.stop(MessageHub.java:221)
	at org.gradle.internal.concurrent.CompositeStoppable.stop(CompositeStoppable.java:103)
	- locked <0x00000000fad8fa70> (a org.gradle.internal.concurrent.CompositeStoppable)
	at org.gradle.internal.remote.internal.hub.MessageHubBackedObjectConnection.stop(MessageHubBackedObjectConnection.java:128)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:139)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:64)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.run(GradleWorkerMain.java:62)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.main(GradleWorkerMain.java:67)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007f85e0270800 nid=0x4cca waiting on condition  [0x00007f85b9efc000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ref.Reference.waitForReferencePendingList(java.base@11-ea/Native Method)
	at java.lang.ref.Reference.processPendingReferences(java.base@11-ea/Reference.java:241)
	at java.lang.ref.Reference.access$000(java.base@11-ea/Reference.java:44)
	at java.lang.ref.Reference$ReferenceHandler.run(java.base@11-ea/Reference.java:213)

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007f85e0274800 nid=0x4ccb in Object.wait()  [0x00007f85b9dfb000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@11-ea/Native Method)
	- waiting on <0x00000000fa710900> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@11-ea/ReferenceQueue.java:155)
	- waiting to re-lock in wait() <0x00000000fa710900> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@11-ea/ReferenceQueue.java:176)
	at java.lang.ref.Finalizer$FinalizerThread.run(java.base@11-ea/Finalizer.java:170)

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007f85e0287000 nid=0x4ccc runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007f85e0289800 nid=0x4ccd waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C1 CompilerThread0" #8 daemon prio=9 os_prio=0 tid=0x00007f85e028b800 nid=0x4cce waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"Sweeper thread" #9 daemon prio=9 os_prio=0 tid=0x00007f85e028d800 nid=0x4ccf runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #10 daemon prio=9 os_prio=0 tid=0x00007f85e0353800 nid=0x4cd0 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Common-Cleaner" #11 daemon prio=8 os_prio=0 tid=0x00007f85e0368800 nid=0x4cd2 in Object.wait()  [0x00007f85b9558000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(java.base@11-ea/Native Method)
	- waiting on <0x00000000fa721370> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@11-ea/ReferenceQueue.java:155)
	- waiting to re-lock in wait() <0x00000000fa721370> (a java.lang.ref.ReferenceQueue$Lock)
	at jdk.internal.ref.CleanerImpl.run(java.base@11-ea/CleanerImpl.java:148)
	at java.lang.Thread.run(java.base@11-ea/Thread.java:832)
	at jdk.internal.misc.InnocuousThread.run(java.base@11-ea/InnocuousThread.java:134)

"/0:0:0:0:0:0:0:1:45406 to /0:0:0:0:0:0:0:1:37871 workers Thread 3" #15 prio=5 os_prio=0 tid=0x00007f85e0812800 nid=0x4cda runnable  [0x00007f85b8929000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.EPoll.wait(java.base@11-ea/Native Method)
	at sun.nio.ch.EPollSelectorImpl.doSelect(java.base@11-ea/EPollSelectorImpl.java:116)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(java.base@11-ea/SelectorImpl.java:118)
	- locked <0x00000000fa710d80> (a sun.nio.ch.Util$2)
	- locked <0x00000000fa710d28> (a sun.nio.ch.EPollSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(java.base@11-ea/SelectorImpl.java:132)
	at org.gradle.internal.remote.internal.inet.SocketConnection$SocketInputStream.read(SocketConnection.java:178)
	at com.esotericsoftware.kryo.io.Input.fill(Input.java:139)
	at com.esotericsoftware.kryo.io.Input.require(Input.java:159)
	at com.esotericsoftware.kryo.io.Input.readByte(Input.java:255)
	at org.gradle.internal.serialize.kryo.KryoBackedDecoder.readByte(KryoBackedDecoder.java:80)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:63)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:52)
	at org.gradle.internal.remote.internal.inet.SocketConnection.receive(SocketConnection.java:79)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionReceive.run(MessageHub.java:263)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@11-ea/ThreadPoolExecutor.java:1128)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@11-ea/ThreadPoolExecutor.java:628)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@11-ea/Thread.java:832)

"Attach Listener" #20 daemon prio=9 os_prio=0 tid=0x00007f859c001000 nid=0x5d11 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"VM Thread" os_prio=0 tid=0x00007f85e0268800 nid=0x4cc9 runnable  

"GC Thread#0" os_prio=0 tid=0x00007f85e003c000 nid=0x4cc4 runnable  

"GC Thread#1" os_prio=0 tid=0x00007f85b0031000 nid=0x4cdb runnable  

"GC Thread#2" os_prio=0 tid=0x00007f85b0032800 nid=0x4cdc runnable  

"GC Thread#3" os_prio=0 tid=0x00007f85b0034000 nid=0x4cdd runnable  

"GC Thread#4" os_prio=0 tid=0x00007f85b0036000 nid=0x4cde runnable  

"GC Thread#5" os_prio=0 tid=0x00007f85b0037800 nid=0x4cdf runnable  

"GC Thread#6" os_prio=0 tid=0x00007f85b0039800 nid=0x4ce0 runnable  

"GC Thread#7" os_prio=0 tid=0x00007f85b003b000 nid=0x4ce1 runnable  

"GC Thread#8" os_prio=0 tid=0x00007f85b003c800 nid=0x4ce2 runnable  

"GC Thread#9" os_prio=0 tid=0x00007f85b003e800 nid=0x4ce3 runnable  

"G1 Main Marker" os_prio=0 tid=0x00007f85e004c800 nid=0x4cc5 runnable  

"G1 Conc#0" os_prio=0 tid=0x00007f85e004e800 nid=0x4cc6 runnable  

"G1 Conc#1" os_prio=0 tid=0x00007f85bc001000 nid=0x4cec runnable  

"G1 Conc#2" os_prio=0 tid=0x00007f85bc002800 nid=0x4ced runnable  

"G1 Refine#0" os_prio=0 tid=0x00007f85e0207800 nid=0x4cc7 runnable  

"G1 Young RemSet Sampling" os_prio=0 tid=0x00007f85e0209800 nid=0x4cc8 runnable  
"VM Periodic Task Thread" os_prio=0 tid=0x00007f85e0356000 nid=0x4cd1 waiting on condition  

JNI global refs: 30, weak refs: 44
```

Gradle Daemon:
```
[:~/work/elastic/elasticsearch] feature/31496_run_test_with_gradle ± jstack 18175
2018-06-22 16:48:17
Full thread dump OpenJDK 64-Bit Server VM (10.0.1+10 mixed mode):

Threads class SMR info:
_java_thread_list=0x00007ff2bc000f30, length=50, elements={
0x00007ff414011000, 0x00007ff414284800, 0x00007ff414286800, 0x00007ff41428f800,
0x00007ff41429a000, 0x00007ff41429c000, 0x00007ff41429e000, 0x00007ff4142a0000,
0x00007ff4142a2000, 0x00007ff41431e800, 0x00007ff414337800, 0x00007ff414f7e800,
0x00007ff415002800, 0x00007ff35400c000, 0x00007ff350006800, 0x00007ff35000a800,
0x00007ff35000b800, 0x00007ff33c009800, 0x00007ff33c00c000, 0x00007ff33c11b000,
0x00007ff33c122000, 0x00007ff33c118800, 0x00007ff33ca8e000, 0x00007ff33cc4e800,
0x00007ff33cfc9000, 0x00007ff33d53e000, 0x00007ff304006800, 0x00007ff30400a800,
0x00007ff30400d800, 0x00007ff304012000, 0x00007ff30c011800, 0x00007ff30c00a800,
0x00007ff33d542000, 0x00007ff33d56a000, 0x00007ff33d56a800, 0x00007ff300002000,
0x00007ff300005000, 0x00007ff300007000, 0x00007ff33d0e7800, 0x00007ff33d61a800,
0x00007ff33d61e000, 0x00007ff33d614000, 0x00007ff33d61f800, 0x00007ff33de8d800,
0x00007ff33d763000, 0x00007ff2bc001800, 0x00007ff2c0003000, 0x00007ff2c0006000,
0x00007ff2c0007800, 0x00007ff380001000
}

"main" #1 prio=5 os_prio=0 tid=0x00007ff414011000 nid=0x4700 waiting on condition  [0x00007ff41b54b000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x0000000080269e40> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.launcher.daemon.server.DaemonStateCoordinator.awaitStop(DaemonStateCoordinator.java:95)
	at org.gradle.launcher.daemon.server.Daemon.awaitExpiration(Daemon.java:247)
	at org.gradle.launcher.daemon.server.Daemon.stopOnExpiration(Daemon.java:221)
	at org.gradle.launcher.daemon.bootstrap.DaemonMain.doAction(DaemonMain.java:131)
	at org.gradle.launcher.bootstrap.EntryPoint.run(EntryPoint.java:45)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(java.base@10.0.1/Native Method)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke(java.base@10.0.1/NativeMethodAccessorImpl.java:62)
	at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(java.base@10.0.1/DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(java.base@10.0.1/Method.java:564)
	at org.gradle.launcher.bootstrap.ProcessBootstrap.runNoExit(ProcessBootstrap.java:60)
	at org.gradle.launcher.bootstrap.ProcessBootstrap.run(ProcessBootstrap.java:37)
	at org.gradle.launcher.daemon.bootstrap.GradleDaemon.main(GradleDaemon.java:22)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007ff414284800 nid=0x471e waiting on condition  [0x00007ff3bd223000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ref.Reference.waitForReferencePendingList(java.base@10.0.1/Native Method)
	at java.lang.ref.Reference.processPendingReferences(java.base@10.0.1/Reference.java:174)
	at java.lang.ref.Reference.access$000(java.base@10.0.1/Reference.java:44)
	at java.lang.ref.Reference$ReferenceHandler.run(java.base@10.0.1/Reference.java:138)

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007ff414286800 nid=0x471f in Object.wait()  [0x00007ff3bd122000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000000802703b0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:151)
	- waiting to re-lock in wait() <0x00000000802703b0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:172)
	at java.lang.ref.Finalizer$FinalizerThread.run(java.base@10.0.1/Finalizer.java:216)

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007ff41428f800 nid=0x4720 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007ff41429a000 nid=0x4721 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C2 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007ff41429c000 nid=0x4722 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C2 CompilerThread2" #7 daemon prio=9 os_prio=0 tid=0x00007ff41429e000 nid=0x4723 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C1 CompilerThread3" #8 daemon prio=9 os_prio=0 tid=0x00007ff4142a0000 nid=0x4724 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"Sweeper thread" #9 daemon prio=9 os_prio=0 tid=0x00007ff4142a2000 nid=0x4725 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #10 daemon prio=9 os_prio=0 tid=0x00007ff41431e800 nid=0x4726 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Common-Cleaner" #11 daemon prio=8 os_prio=0 tid=0x00007ff414337800 nid=0x4728 in Object.wait()  [0x00007ff3bc67d000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000000802d5380> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:151)
	- waiting to re-lock in wait() <0x00000000802d5380> (a java.lang.ref.ReferenceQueue$Lock)
	at jdk.internal.ref.CleanerImpl.run(java.base@10.0.1/CleanerImpl.java:148)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)
	at jdk.internal.misc.InnocuousThread.run(java.base@10.0.1/InnocuousThread.java:134)

"Incoming local TCP Connector on port 33059" #14 prio=5 os_prio=0 tid=0x00007ff414f7e800 nid=0x4729 runnable  [0x00007ff3bc121000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.ServerSocketChannelImpl.accept0(java.base@10.0.1/Native Method)
	at sun.nio.ch.ServerSocketChannelImpl.accept(java.base@10.0.1/ServerSocketChannelImpl.java:424)
	at sun.nio.ch.ServerSocketChannelImpl.accept(java.base@10.0.1/ServerSocketChannelImpl.java:252)
	- locked <0x00000000802772a0> (a java.lang.Object)
	at org.gradle.internal.remote.internal.inet.TcpIncomingConnector$Receiver.run(TcpIncomingConnector.java:102)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Daemon periodic checks" #15 prio=5 os_prio=0 tid=0x00007ff415002800 nid=0x472a waiting on condition  [0x00007ff35b1ec000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x0000000080270868> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(java.base@10.0.1/LockSupport.java:234)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(java.base@10.0.1/AbstractQueuedSynchronizer.java:2117)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:1182)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:899)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Daemon" #16 prio=5 os_prio=0 tid=0x00007ff35400c000 nid=0x472b waiting on condition  [0x00007ff35b0ea000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x0000000080269e40> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.launcher.daemon.server.DaemonStateCoordinator.waitForCommandCompletion(DaemonStateCoordinator.java:313)
	at org.gradle.launcher.daemon.server.DaemonStateCoordinator.runCommand(DaemonStateCoordinator.java:302)
	at org.gradle.launcher.daemon.server.exec.StartBuildOrRespondWithBusy.doBuild(StartBuildOrRespondWithBusy.java:54)
	at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.ReturnResult.execute(ReturnResult.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.api.HandleReportStatus.execute(HandleReportStatus.java:33)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.HandleCancel.execute(HandleCancel.java:37)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.api.HandleStop.execute(HandleStop.java:45)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.DaemonCommandExecuter.executeCommand(DaemonCommandExecuter.java:48)
	at org.gradle.launcher.daemon.server.DefaultIncomingConnectionHandler$ConnectionWorker.handleCommand(DefaultIncomingConnectionHandler.java:160)
	at org.gradle.launcher.daemon.server.DefaultIncomingConnectionHandler$ConnectionWorker.receiveAndHandleCommand(DefaultIncomingConnectionHandler.java:133)
	at org.gradle.launcher.daemon.server.DefaultIncomingConnectionHandler$ConnectionWorker.run(DefaultIncomingConnectionHandler.java:121)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Handler for socket connection from /0:0:0:0:0:0:0:1:33059 to /0:0:0:0:0:0:0:1:45986" #17 prio=5 os_prio=0 tid=0x00007ff350006800 nid=0x472c runnable  [0x00007ff35afea000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.EPollArrayWrapper.epollWait(java.base@10.0.1/Native Method)
	at sun.nio.ch.EPollArrayWrapper.poll(java.base@10.0.1/EPollArrayWrapper.java:265)
	at sun.nio.ch.EPollSelectorImpl.doSelect(java.base@10.0.1/EPollSelectorImpl.java:92)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(java.base@10.0.1/SelectorImpl.java:89)
	- locked <0x00000000802697e0> (a sun.nio.ch.Util$2)
	- locked <0x00000000802697f0> (a java.util.Collections$UnmodifiableSet)
	- locked <0x0000000080269798> (a sun.nio.ch.EPollSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:100)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:104)
	at org.gradle.internal.remote.internal.inet.SocketConnection$SocketInputStream.read(SocketConnection.java:178)
	at com.esotericsoftware.kryo.io.Input.fill(Input.java:139)
	at com.esotericsoftware.kryo.io.Input.require(Input.java:159)
	at com.esotericsoftware.kryo.io.Input.readInt(Input.java:308)
	at org.gradle.internal.serialize.kryo.KryoBackedDecoder.readSmallInt(KryoBackedDecoder.java:120)
	at org.gradle.internal.serialize.DefaultSerializerRegistry$TaggedTypeSerializer.read(DefaultSerializerRegistry.java:139)
	at org.gradle.internal.serialize.Serializers$StatefulSerializerAdapter$1.read(Serializers.java:36)
	at org.gradle.internal.remote.internal.inet.SocketConnection.receive(SocketConnection.java:79)
	at org.gradle.launcher.daemon.server.SynchronizedDispatchConnection.receive(SynchronizedDispatchConnection.java:68)
	at org.gradle.launcher.daemon.server.DefaultDaemonConnection$1.run(DefaultDaemonConnection.java:63)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cancel handler" #18 prio=5 os_prio=0 tid=0x00007ff35000a800 nid=0x472d waiting on condition  [0x00007ff35aee9000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x000000008026de18> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.launcher.daemon.server.DefaultDaemonConnection$CommandQueue$1.run(DefaultDaemonConnection.java:229)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Daemon worker" #19 prio=5 os_prio=0 tid=0x00007ff35000b800 nid=0x472e waiting on condition  [0x00007ff35ade3000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000cd162008> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.dispatch.AsyncDispatch.stop(AsyncDispatch.java:181)
	at org.gradle.internal.concurrent.CompositeStoppable.stop(CompositeStoppable.java:103)
	- locked <0x00000000cd17b828> (a org.gradle.internal.concurrent.CompositeStoppable)
	at org.gradle.internal.actor.internal.DefaultActorFactory$NonBlockingActor.stop(DefaultActorFactory.java:147)
	at org.gradle.internal.concurrent.CompositeStoppable.stop(CompositeStoppable.java:103)
	- locked <0x00000000cd17b718> (a org.gradle.internal.concurrent.CompositeStoppable)
	at org.gradle.api.internal.tasks.testing.processors.MaxNParallelTestClassProcessor.stop(MaxNParallelTestClassProcessor.java:86)
	at org.gradle.api.internal.tasks.testing.processors.RunPreviousFailedFirstTestClassProcessor.stop(RunPreviousFailedFirstTestClassProcessor.java:63)
	at org.gradle.api.internal.tasks.testing.processors.PatternMatchTestClassProcessor.stop(PatternMatchTestClassProcessor.java:48)
	at org.gradle.api.internal.tasks.testing.processors.TestMainAction.run(TestMainAction.java:57)
	at org.gradle.api.internal.tasks.testing.detection.DefaultTestExecuter.execute(DefaultTestExecuter.java:116)
	at org.gradle.api.internal.tasks.testing.detection.DefaultTestExecuter.execute(DefaultTestExecuter.java:51)
	at org.gradle.api.tasks.testing.AbstractTestTask.executeTests(AbstractTestTask.java:469)
	at org.gradle.api.tasks.testing.Test.executeTests(Test.java:583)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(java.base@10.0.1/Native Method)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke(java.base@10.0.1/NativeMethodAccessorImpl.java:62)
	at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(java.base@10.0.1/DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(java.base@10.0.1/Method.java:564)
	at org.gradle.internal.reflect.JavaMethod.invoke(JavaMethod.java:73)
	at org.gradle.api.internal.project.taskfactory.StandardTaskAction.doExecute(StandardTaskAction.java:46)
	at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:39)
	at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:26)
	at org.gradle.api.internal.AbstractTask$TaskActionWrapper.execute(AbstractTask.java:794)
	at org.gradle.api.internal.AbstractTask$TaskActionWrapper.execute(AbstractTask.java:761)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter$1.run(ExecuteActionsTaskExecuter.java:124)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:317)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:309)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:185)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:97)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeAction(ExecuteActionsTaskExecuter.java:113)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:95)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.execute(ExecuteActionsTaskExecuter.java:73)
	at org.gradle.api.internal.tasks.execution.OutputDirectoryCreatingTaskExecuter.execute(OutputDirectoryCreatingTaskExecuter.java:51)
	at org.gradle.api.internal.tasks.execution.SkipUpToDateTaskExecuter.execute(SkipUpToDateTaskExecuter.java:59)
	at org.gradle.api.internal.tasks.execution.ResolveTaskOutputCachingStateExecuter.execute(ResolveTaskOutputCachingStateExecuter.java:54)
	at org.gradle.api.internal.tasks.execution.ResolveBuildCacheKeyExecuter.execute(ResolveBuildCacheKeyExecuter.java:66)
	at org.gradle.api.internal.tasks.execution.ValidatingTaskExecuter.execute(ValidatingTaskExecuter.java:59)
	at org.gradle.api.internal.tasks.execution.SkipEmptySourceFilesTaskExecuter.execute(SkipEmptySourceFilesTaskExecuter.java:101)
	at org.gradle.api.internal.tasks.execution.FinalizeInputFilePropertiesTaskExecuter.execute(FinalizeInputFilePropertiesTaskExecuter.java:44)
	at org.gradle.api.internal.tasks.execution.CleanupStaleOutputsExecuter.execute(CleanupStaleOutputsExecuter.java:91)
	at org.gradle.api.internal.tasks.execution.ResolveTaskArtifactStateTaskExecuter.execute(ResolveTaskArtifactStateTaskExecuter.java:62)
	at org.gradle.api.internal.tasks.execution.SkipTaskWithNoActionsExecuter.execute(SkipTaskWithNoActionsExecuter.java:59)
	at org.gradle.api.internal.tasks.execution.SkipOnlyIfTaskExecuter.execute(SkipOnlyIfTaskExecuter.java:54)
	at org.gradle.api.internal.tasks.execution.ExecuteAtMostOnceTaskExecuter.execute(ExecuteAtMostOnceTaskExecuter.java:43)
	at org.gradle.api.internal.tasks.execution.CatchExceptionTaskExecuter.execute(CatchExceptionTaskExecuter.java:34)
	at org.gradle.execution.taskgraph.DefaultTaskGraphExecuter$EventFiringTaskWorker$1.run(DefaultTaskGraphExecuter.java:256)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:317)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:309)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:185)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:97)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.execution.taskgraph.DefaultTaskGraphExecuter$EventFiringTaskWorker.execute(DefaultTaskGraphExecuter.java:249)
	at org.gradle.execution.taskgraph.DefaultTaskGraphExecuter$EventFiringTaskWorker.execute(DefaultTaskGraphExecuter.java:238)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$TaskExecutorWorker$1.execute(DefaultTaskPlanExecutor.java:104)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$TaskExecutorWorker$1.execute(DefaultTaskPlanExecutor.java:98)
	at org.gradle.execution.taskgraph.DefaultTaskExecutionPlan.execute(DefaultTaskExecutionPlan.java:663)
	at org.gradle.execution.taskgraph.DefaultTaskExecutionPlan.executeWithTask(DefaultTaskExecutionPlan.java:596)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$TaskExecutorWorker.run(DefaultTaskPlanExecutor.java:98)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor.process(DefaultTaskPlanExecutor.java:59)
	at org.gradle.execution.taskgraph.DefaultTaskGraphExecuter.execute(DefaultTaskGraphExecuter.java:130)
	at org.gradle.execution.SelectedTaskExecutionAction.execute(SelectedTaskExecutionAction.java:37)
	at org.gradle.execution.DefaultBuildExecuter.execute(DefaultBuildExecuter.java:37)
	at org.gradle.execution.DefaultBuildExecuter.access$000(DefaultBuildExecuter.java:23)
	at org.gradle.execution.DefaultBuildExecuter$1.proceed(DefaultBuildExecuter.java:43)
	at org.gradle.execution.DryRunBuildExecutionAction.execute(DryRunBuildExecutionAction.java:46)
	at org.gradle.execution.DefaultBuildExecuter.execute(DefaultBuildExecuter.java:37)
	at org.gradle.execution.DefaultBuildExecuter.execute(DefaultBuildExecuter.java:30)
	at org.gradle.initialization.DefaultGradleLauncher$ExecuteTasks.run(DefaultGradleLauncher.java:336)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:317)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:309)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:185)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:97)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.initialization.DefaultGradleLauncher.runTasks(DefaultGradleLauncher.java:210)
	at org.gradle.initialization.DefaultGradleLauncher.doBuildStages(DefaultGradleLauncher.java:140)
	at org.gradle.initialization.DefaultGradleLauncher.executeTasks(DefaultGradleLauncher.java:115)
	at org.gradle.internal.invocation.GradleBuildController$1.call(GradleBuildController.java:78)
	at org.gradle.internal.invocation.GradleBuildController$1.call(GradleBuildController.java:75)
	at org.gradle.internal.work.DefaultWorkerLeaseService.withLocks(DefaultWorkerLeaseService.java:152)
	at org.gradle.internal.work.StopShieldingWorkerLeaseService.withLocks(StopShieldingWorkerLeaseService.java:38)
	at org.gradle.internal.invocation.GradleBuildController.doBuild(GradleBuildController.java:100)
	at org.gradle.internal.invocation.GradleBuildController.run(GradleBuildController.java:75)
	at org.gradle.tooling.internal.provider.ExecuteBuildActionRunner.run(ExecuteBuildActionRunner.java:28)
	at org.gradle.launcher.exec.ChainingBuildActionRunner.run(ChainingBuildActionRunner.java:35)
	at org.gradle.tooling.internal.provider.ValidatingBuildActionRunner.run(ValidatingBuildActionRunner.java:32)
	at org.gradle.launcher.exec.RunAsBuildOperationBuildActionRunner$3.run(RunAsBuildOperationBuildActionRunner.java:45)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:317)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:309)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:185)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:97)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.launcher.exec.RunAsBuildOperationBuildActionRunner.run(RunAsBuildOperationBuildActionRunner.java:42)
	at org.gradle.tooling.internal.provider.SubscribableBuildActionRunner.run(SubscribableBuildActionRunner.java:51)
	at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:47)
	at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:31)
	at org.gradle.launcher.exec.BuildTreeScopeBuildActionExecuter.execute(BuildTreeScopeBuildActionExecuter.java:39)
	at org.gradle.launcher.exec.BuildTreeScopeBuildActionExecuter.execute(BuildTreeScopeBuildActionExecuter.java:25)
	at org.gradle.tooling.internal.provider.ContinuousBuildActionExecuter.execute(ContinuousBuildActionExecuter.java:80)
	at org.gradle.tooling.internal.provider.ContinuousBuildActionExecuter.execute(ContinuousBuildActionExecuter.java:53)
	at org.gradle.tooling.internal.provider.ServicesSetupBuildActionExecuter.execute(ServicesSetupBuildActionExecuter.java:61)
	at org.gradle.tooling.internal.provider.ServicesSetupBuildActionExecuter.execute(ServicesSetupBuildActionExecuter.java:34)
	at org.gradle.tooling.internal.provider.GradleThreadBuildActionExecuter.execute(GradleThreadBuildActionExecuter.java:36)
	at org.gradle.tooling.internal.provider.GradleThreadBuildActionExecuter.execute(GradleThreadBuildActionExecuter.java:25)
	at org.gradle.tooling.internal.provider.ParallelismConfigurationBuildActionExecuter.execute(ParallelismConfigurationBuildActionExecuter.java:43)
	at org.gradle.tooling.internal.provider.ParallelismConfigurationBuildActionExecuter.execute(ParallelismConfigurationBuildActionExecuter.java:29)
	at org.gradle.tooling.internal.provider.StartParamsValidatingActionExecuter.execute(StartParamsValidatingActionExecuter.java:64)
	at org.gradle.tooling.internal.provider.StartParamsValidatingActionExecuter.execute(StartParamsValidatingActionExecuter.java:29)
	at org.gradle.tooling.internal.provider.SessionFailureReportingActionExecuter.execute(SessionFailureReportingActionExecuter.java:59)
	at org.gradle.tooling.internal.provider.SessionFailureReportingActionExecuter.execute(SessionFailureReportingActionExecuter.java:44)
	at org.gradle.tooling.internal.provider.SetupLoggingActionExecuter.execute(SetupLoggingActionExecuter.java:46)
	at org.gradle.tooling.internal.provider.SetupLoggingActionExecuter.execute(SetupLoggingActionExecuter.java:30)
	at org.gradle.launcher.daemon.server.exec.ExecuteBuild.doBuild(ExecuteBuild.java:67)
	at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.WatchForDisconnection.execute(WatchForDisconnection.java:37)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.ResetDeprecationLogger.execute(ResetDeprecationLogger.java:26)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.RequestStopIfSingleUsedDaemon.execute(RequestStopIfSingleUsedDaemon.java:34)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.ForwardClientInput$2.call(ForwardClientInput.java:74)
	at org.gradle.launcher.daemon.server.exec.ForwardClientInput$2.call(ForwardClientInput.java:72)
	at org.gradle.util.Swapper.swap(Swapper.java:38)
	at org.gradle.launcher.daemon.server.exec.ForwardClientInput.execute(ForwardClientInput.java:72)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.LogAndCheckHealth.execute(LogAndCheckHealth.java:55)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.LogToClient.doBuild(LogToClient.java:62)
	at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.EstablishBuildEnvironment.doBuild(EstablishBuildEnvironment.java:82)
	at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.StartBuildOrRespondWithBusy$1.run(StartBuildOrRespondWithBusy.java:50)
	at org.gradle.launcher.daemon.server.DaemonStateCoordinator$1.run(DaemonStateCoordinator.java:295)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Asynchronous log dispatcher for DefaultDaemonConnection: socket connection from /0:0:0:0:0:0:0:1:33059 to /0:0:0:0:0:0:0:1:45986" #20 prio=5 os_prio=0 tid=0x00007ff33c009800 nid=0x4731 waiting on condition  [0x00007ff35ace7000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(java.base@10.0.1/Native Method)
	at org.gradle.launcher.daemon.server.exec.LogToClient$AsynchronousLogDispatcher.run(LogToClient.java:108)

"Stdin handler" #21 prio=5 os_prio=0 tid=0x00007ff33c00c000 nid=0x4732 waiting on condition  [0x00007ff35abe6000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x0000000080277570> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.launcher.daemon.server.DefaultDaemonConnection$CommandQueue$1.run(DefaultDaemonConnection.java:229)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for file hash cache (/home/alpar/.gradle/caches/4.7/fileHashes)" #22 prio=5 os_prio=0 tid=0x00007ff33c11b000 nid=0x4733 waiting on condition  [0x00007ff35a8e5000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x000000008026e108> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"File lock request listener" #23 prio=5 os_prio=0 tid=0x00007ff33c122000 nid=0x4734 runnable  [0x00007ff35a7e4000]
   java.lang.Thread.State: RUNNABLE
	at java.net.PlainDatagramSocketImpl.receive0(java.base@10.0.1/Native Method)
	- locked <0x00000000802758c0> (a java.net.PlainDatagramSocketImpl)
	at java.net.AbstractPlainDatagramSocketImpl.receive(java.base@10.0.1/AbstractPlainDatagramSocketImpl.java:180)
	- locked <0x00000000802758c0> (a java.net.PlainDatagramSocketImpl)
	at java.net.DatagramSocket.receive(java.base@10.0.1/DatagramSocket.java:814)
	- locked <0x0000000080275968> (a java.net.DatagramPacket)
	- locked <0x0000000080275a08> (a java.net.DatagramSocket)
	at org.gradle.cache.internal.locklistener.FileLockCommunicator.receive(FileLockCommunicator.java:79)
	at org.gradle.cache.internal.locklistener.DefaultFileLockContentionHandler$1.doRun(DefaultFileLockContentionHandler.java:108)
	at org.gradle.cache.internal.locklistener.DefaultFileLockContentionHandler$1.run(DefaultFileLockContentionHandler.java:94)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for file hash cache (/home/alpar/work/elastic/elasticsearch/.gradle/4.7/fileHashes)" #24 prio=5 os_prio=0 tid=0x00007ff33c118800 nid=0x4735 waiting on condition  [0x00007ff35a4e3000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000802777f0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Memory manager" #25 prio=5 os_prio=0 tid=0x00007ff33ca8e000 nid=0x473e waiting on condition  [0x00007ff359be2000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x0000000080d69880> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(java.base@10.0.1/LockSupport.java:234)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(java.base@10.0.1/AbstractQueuedSynchronizer.java:2117)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:1182)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:899)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for Artifact transforms cache (/home/alpar/.gradle/caches/transforms-1)" #26 prio=5 os_prio=0 tid=0x00007ff33cc4e800 nid=0x473f waiting on condition  [0x00007ff3598e1000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x0000000080dafd30> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations" #27 prio=5 os_prio=0 tid=0x00007ff33cfc9000 nid=0x4740 waiting on condition  [0x00007ff358fe0000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000802c8770> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Exec process" #32 prio=5 os_prio=0 tid=0x00007ff33d53e000 nid=0x4747 in Object.wait()  [0x00007ff313cfd000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x0000000083923350> (a java.lang.ProcessImpl)
	at java.lang.Object.wait(java.base@10.0.1/Object.java:328)
	at java.lang.ProcessImpl.waitFor(java.base@10.0.1/ProcessImpl.java:494)
	- waiting to re-lock in wait() <0x0000000083923350> (a java.lang.ProcessImpl)
	at org.gradle.process.internal.ExecHandleRunner.run(ExecHandleRunner.java:80)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"process reaper" #33 daemon prio=10 os_prio=0 tid=0x00007ff304006800 nid=0x474a runnable  [0x00007ff3bc020000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ProcessHandleImpl.waitForProcessExit0(java.base@10.0.1/Native Method)
	at java.lang.ProcessHandleImpl.access$000(java.base@10.0.1/ProcessHandleImpl.java:50)
	at java.lang.ProcessHandleImpl$1.run(java.base@10.0.1/ProcessHandleImpl.java:138)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Exec process Thread 2" #35 prio=5 os_prio=0 tid=0x00007ff30400a800 nid=0x475a in Object.wait()  [0x00007ff313afb000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000000cd0fb418> (a java.lang.ProcessImpl)
	at java.lang.Object.wait(java.base@10.0.1/Object.java:328)
	at java.lang.ProcessImpl.waitFor(java.base@10.0.1/ProcessImpl.java:494)
	- waiting to re-lock in wait() <0x00000000cd0fb418> (a java.lang.ProcessImpl)
	at org.gradle.process.internal.ExecHandleRunner.run(ExecHandleRunner.java:80)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Exec process Thread 3" #36 prio=5 os_prio=0 tid=0x00007ff30400d800 nid=0x475b runnable  [0x00007ff3139fa000]
   java.lang.Thread.State: RUNNABLE
	at java.io.FileInputStream.readBytes(java.base@10.0.1/Native Method)
	at java.io.FileInputStream.read(java.base@10.0.1/FileInputStream.java:280)
	at java.io.BufferedInputStream.fill(java.base@10.0.1/BufferedInputStream.java:252)
	at java.io.BufferedInputStream.read1(java.base@10.0.1/BufferedInputStream.java:292)
	at java.io.BufferedInputStream.read(java.base@10.0.1/BufferedInputStream.java:351)
	- locked <0x00000000839021d0> (a java.lang.ProcessImpl$ProcessPipeInputStream)
	at java.io.FilterInputStream.read(java.base@10.0.1/FilterInputStream.java:107)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.forwardContent(ExecOutputHandleRunner.java:61)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.run(ExecOutputHandleRunner.java:51)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Exec process Thread 4" #37 prio=5 os_prio=0 tid=0x00007ff304012000 nid=0x475c runnable  [0x00007ff3138f9000]
   java.lang.Thread.State: RUNNABLE
	at java.io.FileInputStream.readBytes(java.base@10.0.1/Native Method)
	at java.io.FileInputStream.read(java.base@10.0.1/FileInputStream.java:280)
	at java.io.BufferedInputStream.fill(java.base@10.0.1/BufferedInputStream.java:252)
	at java.io.BufferedInputStream.read1(java.base@10.0.1/BufferedInputStream.java:292)
	at java.io.BufferedInputStream.read(java.base@10.0.1/BufferedInputStream.java:351)
	- locked <0x0000000083948580> (a java.lang.ProcessImpl$ProcessPipeInputStream)
	at java.io.FilterInputStream.read(java.base@10.0.1/FilterInputStream.java:107)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.forwardContent(ExecOutputHandleRunner.java:61)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.run(ExecOutputHandleRunner.java:51)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:41677 to /0:0:0:0:0:0:0:1:53316 workers" #38 prio=5 os_prio=0 tid=0x00007ff30c011800 nid=0x4774 waiting on condition  [0x00007ff313bfc000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x0000000083774b78> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$Handler.run(MessageHub.java:393)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:41677 to /0:0:0:0:0:0:0:1:53316 workers Thread 2" #39 prio=5 os_prio=0 tid=0x00007ff30c00a800 nid=0x4775 waiting on condition  [0x00007ff3137f8000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x0000000083798068> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$Handler.run(MessageHub.java:393)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:41677 to /0:0:0:0:0:0:0:1:53316 workers Thread 3" #40 prio=5 os_prio=0 tid=0x00007ff33d542000 nid=0x4776 waiting on condition  [0x00007ff3136f7000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000837de2b8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$Handler.run(MessageHub.java:393)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:41677 to /0:0:0:0:0:0:0:1:53316 workers Thread 4" #41 prio=5 os_prio=0 tid=0x00007ff33d56a000 nid=0x4777 waiting on condition  [0x00007ff3135f6000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000837bb180> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionDispatch.run(MessageHub.java:314)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:41677 to /0:0:0:0:0:0:0:1:53316 workers Thread 5" #42 prio=5 os_prio=0 tid=0x00007ff33d56a800 nid=0x4778 runnable  [0x00007ff3134f5000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.EPollArrayWrapper.epollWait(java.base@10.0.1/Native Method)
	at sun.nio.ch.EPollArrayWrapper.poll(java.base@10.0.1/EPollArrayWrapper.java:265)
	at sun.nio.ch.EPollSelectorImpl.doSelect(java.base@10.0.1/EPollSelectorImpl.java:92)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(java.base@10.0.1/SelectorImpl.java:89)
	- locked <0x0000000083923748> (a sun.nio.ch.Util$2)
	- locked <0x0000000083923758> (a java.util.Collections$UnmodifiableSet)
	- locked <0x0000000083923700> (a sun.nio.ch.EPollSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:100)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:104)
	at org.gradle.internal.remote.internal.inet.SocketConnection$SocketInputStream.read(SocketConnection.java:178)
	at com.esotericsoftware.kryo.io.Input.fill(Input.java:139)
	at com.esotericsoftware.kryo.io.Input.require(Input.java:159)
	at com.esotericsoftware.kryo.io.Input.readByte(Input.java:255)
	at org.gradle.internal.serialize.kryo.KryoBackedDecoder.readByte(KryoBackedDecoder.java:80)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:63)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:52)
	at org.gradle.internal.remote.internal.inet.SocketConnection.receive(SocketConnection.java:79)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionReceive.run(MessageHub.java:263)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"process reaper" #46 daemon prio=10 os_prio=0 tid=0x00007ff300002000 nid=0x47a0 runnable  [0x00007ff312ef1000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ProcessHandleImpl.waitForProcessExit0(java.base@10.0.1/Native Method)
	at java.lang.ProcessHandleImpl.access$000(java.base@10.0.1/ProcessHandleImpl.java:50)
	at java.lang.ProcessHandleImpl$1.run(java.base@10.0.1/ProcessHandleImpl.java:138)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Exec process Thread 5" #48 prio=5 os_prio=0 tid=0x00007ff300005000 nid=0x47a2 runnable  [0x00007ff312dce000]
   java.lang.Thread.State: RUNNABLE
	at java.io.FileInputStream.readBytes(java.base@10.0.1/Native Method)
	at java.io.FileInputStream.read(java.base@10.0.1/FileInputStream.java:280)
	at java.io.BufferedInputStream.fill(java.base@10.0.1/BufferedInputStream.java:252)
	at java.io.BufferedInputStream.read1(java.base@10.0.1/BufferedInputStream.java:292)
	at java.io.BufferedInputStream.read(java.base@10.0.1/BufferedInputStream.java:351)
	- locked <0x00000000cd0fdeb8> (a java.lang.ProcessImpl$ProcessPipeInputStream)
	at java.io.FilterInputStream.read(java.base@10.0.1/FilterInputStream.java:107)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.forwardContent(ExecOutputHandleRunner.java:61)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.run(ExecOutputHandleRunner.java:51)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Exec process Thread 6" #49 prio=5 os_prio=0 tid=0x00007ff300007000 nid=0x47a3 runnable  [0x00007ff312ccd000]
   java.lang.Thread.State: RUNNABLE
	at java.io.FileInputStream.readBytes(java.base@10.0.1/Native Method)
	at java.io.FileInputStream.read(java.base@10.0.1/FileInputStream.java:280)
	at java.io.BufferedInputStream.fill(java.base@10.0.1/BufferedInputStream.java:252)
	at java.io.BufferedInputStream.read1(java.base@10.0.1/BufferedInputStream.java:292)
	at java.io.BufferedInputStream.read(java.base@10.0.1/BufferedInputStream.java:351)
	- locked <0x00000000cd0fdf48> (a java.lang.ProcessImpl$ProcessPipeInputStream)
	at java.io.FilterInputStream.read(java.base@10.0.1/FilterInputStream.java:107)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.forwardContent(ExecOutputHandleRunner.java:61)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.run(ExecOutputHandleRunner.java:51)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"build-scan-data-write-0" #55 prio=1 os_prio=0 tid=0x00007ff33d0e7800 nid=0x47d5 waiting on condition  [0x00007ff358cdd000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000829a5d20> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at com.gradle.scan.plugin.internal.d.b$2.run(SourceFile:62)

"Cache worker for file content cache (/home/alpar/work/elastic/elasticsearch/.gradle/4.7/fileContent)" #56 prio=5 os_prio=0 tid=0x00007ff33d61a800 nid=0x48e1 waiting on condition  [0x00007ff358dde000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x0000000097198c38> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for task history cache (/home/alpar/work/elastic/elasticsearch/.gradle/4.7/taskHistory)" #57 prio=5 os_prio=0 tid=0x00007ff33d61e000 nid=0x48e2 waiting on condition  [0x00007ff358edf000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x000000009714c6c8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for Build Output Cleanup Cache (/home/alpar/work/elastic/elasticsearch/.gradle/buildOutputCleanup)" #58 prio=5 os_prio=0 tid=0x00007ff33d614000 nid=0x48e3 waiting on condition  [0x00007ff3131f4000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000971001f8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for Java compile cache (/home/alpar/work/elastic/elasticsearch/.gradle/4.7/javaCompile)" #59 prio=5 os_prio=0 tid=0x00007ff33d61f800 nid=0x48e4 waiting on condition  [0x00007ff3130f3000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x000000009ad80c20> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Dispatch org.gradle.api.internal.tasks.testing.results.AttachParentTestResultProcessor@7bb678d6" #60 prio=5 os_prio=0 tid=0x00007ff33de8d800 nid=0x4cbe waiting on condition  [0x00007ff3128c9000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c9961e48> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.dispatch.AsyncDispatch.dispatchMessages(AsyncDispatch.java:115)
	at org.gradle.internal.dispatch.AsyncDispatch.access$000(AsyncDispatch.java:34)
	at org.gradle.internal.dispatch.AsyncDispatch$1.run(AsyncDispatch.java:73)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Dispatch org.gradle.api.internal.tasks.testing.processors.RestartEveryNTestClassProcessor@19657f9" #61 prio=5 os_prio=0 tid=0x00007ff33d763000 nid=0x4cbf waiting on condition  [0x00007ff3129c9000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000cd0c8198> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.process.internal.DefaultExecHandle.waitForFinish(DefaultExecHandle.java:295)
	at org.gradle.process.internal.worker.DefaultWorkerProcess.waitForStop(DefaultWorkerProcess.java:210)
	at org.gradle.process.internal.worker.DefaultWorkerProcessBuilder$MemoryRequestingWorkerProcess.waitForStop(DefaultWorkerProcessBuilder.java:228)
	at org.gradle.api.internal.tasks.testing.worker.ForkingTestClassProcessor.stop(ForkingTestClassProcessor.java:152)
	at org.gradle.api.internal.tasks.testing.processors.RestartEveryNTestClassProcessor.endBatch(RestartEveryNTestClassProcessor.java:76)
	at org.gradle.api.internal.tasks.testing.processors.RestartEveryNTestClassProcessor.stop(RestartEveryNTestClassProcessor.java:62)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(java.base@10.0.1/Native Method)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke(java.base@10.0.1/NativeMethodAccessorImpl.java:62)
	at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(java.base@10.0.1/DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(java.base@10.0.1/Method.java:564)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:35)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:24)
	at org.gradle.internal.dispatch.FailureHandlingDispatch.dispatch(FailureHandlingDispatch.java:29)
	at org.gradle.internal.dispatch.AsyncDispatch.dispatchMessages(AsyncDispatch.java:133)
	at org.gradle.internal.dispatch.AsyncDispatch.access$000(AsyncDispatch.java:34)
	at org.gradle.internal.dispatch.AsyncDispatch$1.run(AsyncDispatch.java:73)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:37871 to /0:0:0:0:0:0:0:1:45406 workers" #64 prio=5 os_prio=0 tid=0x00007ff2bc001800 nid=0x4cd4 waiting on condition  [0x00007ff312ecf000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000cd0964c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$Handler.run(MessageHub.java:393)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:37871 to /0:0:0:0:0:0:0:1:45406 workers Thread 2" #65 prio=5 os_prio=0 tid=0x00007ff2c0003000 nid=0x4cd5 waiting on condition  [0x00007ff312ff2000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000cd0c9c50> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$Handler.run(MessageHub.java:393)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:37871 to /0:0:0:0:0:0:0:1:45406 workers Thread 3" #66 prio=5 os_prio=0 tid=0x00007ff2c0006000 nid=0x4cd6 waiting on condition  [0x00007ff3101c8000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000cd0e15d8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionDispatch.run(MessageHub.java:314)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:37871 to /0:0:0:0:0:0:0:1:45406 workers Thread 4" #67 prio=5 os_prio=0 tid=0x00007ff2c0007800 nid=0x4cd7 runnable  [0x00007ff2bb7fe000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.EPollArrayWrapper.epollWait(java.base@10.0.1/Native Method)
	at sun.nio.ch.EPollArrayWrapper.poll(java.base@10.0.1/EPollArrayWrapper.java:265)
	at sun.nio.ch.EPollSelectorImpl.doSelect(java.base@10.0.1/EPollSelectorImpl.java:92)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(java.base@10.0.1/SelectorImpl.java:89)
	- locked <0x00000000cd0ceed8> (a sun.nio.ch.Util$2)
	- locked <0x00000000cd0ceec8> (a java.util.Collections$UnmodifiableSet)
	- locked <0x00000000cd0cedb0> (a sun.nio.ch.EPollSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:100)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:104)
	at org.gradle.internal.remote.internal.inet.SocketConnection$SocketInputStream.read(SocketConnection.java:178)
	at com.esotericsoftware.kryo.io.Input.fill(Input.java:139)
	at com.esotericsoftware.kryo.io.Input.require(Input.java:159)
	at com.esotericsoftware.kryo.io.Input.readByte(Input.java:255)
	at org.gradle.internal.serialize.kryo.KryoBackedDecoder.readByte(KryoBackedDecoder.java:80)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:63)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:52)
	at org.gradle.internal.remote.internal.inet.SocketConnection.receive(SocketConnection.java:79)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionReceive.run(MessageHub.java:263)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Attach Listener" #68 daemon prio=9 os_prio=0 tid=0x00007ff380001000 nid=0x6524 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"VM Thread" os_prio=0 tid=0x00007ff41427a800 nid=0x471d runnable  

"GC Thread#0" os_prio=0 tid=0x00007ff414027800 nid=0x4701 runnable  

"GC Thread#1" os_prio=0 tid=0x00007ff414029000 nid=0x4702 runnable  

"GC Thread#2" os_prio=0 tid=0x00007ff41402a800 nid=0x4703 runnable  

"GC Thread#3" os_prio=0 tid=0x00007ff41402c800 nid=0x4704 runnable  

"GC Thread#4" os_prio=0 tid=0x00007ff41402e000 nid=0x4705 runnable  

"GC Thread#5" os_prio=0 tid=0x00007ff414030000 nid=0x4706 runnable  

"GC Thread#6" os_prio=0 tid=0x00007ff414031800 nid=0x4707 runnable  

"GC Thread#7" os_prio=0 tid=0x00007ff414033000 nid=0x4708 runnable  

"GC Thread#8" os_prio=0 tid=0x00007ff414035000 nid=0x4709 runnable  

"GC Thread#9" os_prio=0 tid=0x00007ff414036800 nid=0x470a runnable  

"G1 Main Marker" os_prio=0 tid=0x00007ff414061800 nid=0x470b runnable  

"G1 Conc#0" os_prio=0 tid=0x00007ff414063000 nid=0x470c runnable  

"G1 Conc#1" os_prio=0 tid=0x00007ff414065000 nid=0x470e runnable  

"G1 Conc#2" os_prio=0 tid=0x00007ff414066800 nid=0x470f runnable  

"G1 Refine#0" os_prio=0 tid=0x00007ff414207800 nid=0x4711 runnable  

"G1 Refine#1" os_prio=0 tid=0x00007ff414209000 nid=0x4712 runnable  

"G1 Refine#2" os_prio=0 tid=0x00007ff41420a800 nid=0x4713 runnable  

"G1 Refine#3" os_prio=0 tid=0x00007ff41420c800 nid=0x4714 runnable  

"G1 Refine#4" os_prio=0 tid=0x00007ff41420e000 nid=0x4716 runnable  

"G1 Refine#5" os_prio=0 tid=0x00007ff414210000 nid=0x4717 runnable  

"G1 Refine#6" os_prio=0 tid=0x00007ff414211800 nid=0x4718 runnable  

"G1 Refine#7" os_prio=0 tid=0x00007ff414213800 nid=0x4719 runnable  

"G1 Refine#8" os_prio=0 tid=0x00007ff414215000 nid=0x471a runnable  

"G1 Refine#9" os_prio=0 tid=0x00007ff414217000 nid=0x471b runnable  

"G1 Young RemSet Sampling" os_prio=0 tid=0x00007ff414218800 nid=0x471c runnable  
"VM Periodic Task Thread" os_prio=0 tid=0x00007ff414321000 nid=0x4727 waiting on condition  

JNI global references: 21
```

Thread dumps from the reproduction repo
---------------------------------------

```
[:~/work/elastic/elasticsearch] feature/31496_run_test_with_gradle(+1/-0) ± jps
26664 GradleWrapperMain
888 Main
28137 Jps
26715 GradleDaemon
2427 RemoteMavenServer
26815 GradleWorkerMain
[:~/work/elastic/elasticsearch] feature/31496_run_test_with_gradle(+1/-0) ± jstack 26815
2018-06-22 18:37:50
Full thread dump OpenJDK 64-Bit Server VM (10.0.1+10 mixed mode):

Threads class SMR info:
_java_thread_list=0x00007f79fc000b20, length=14, elements={
0x00007f7ac4013800, 0x00007f7ac424b800, 0x00007f7ac424d800, 0x00007f7ac4257000,
0x00007f7ac4261000, 0x00007f7ac4263000, 0x00007f7ac4265000, 0x00007f7ac4267000,
0x00007f7ac4269000, 0x00007f7ac4316800, 0x00007f7ac432f000, 0x00007f7ac47d4800,
0x00007f79f427b800, 0x00007f7a24001000
}

"main" #1 prio=5 os_prio=0 tid=0x00007f7ac4013800 nid=0x68c0 waiting on condition  [0x00007f7aceb7d000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000005e5bb2a90> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(java.base@10.0.1/LockSupport.java:234)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(java.base@10.0.1/AbstractQueuedSynchronizer.java:2117)
	at java.util.concurrent.ThreadPoolExecutor.awaitTermination(java.base@10.0.1/ThreadPoolExecutor.java:1464)
	at org.gradle.internal.concurrent.ManagedExecutorImpl.stop(ManagedExecutorImpl.java:81)
	at org.gradle.internal.concurrent.DefaultExecutorFactory$TrackedManagedExecutor.stop(DefaultExecutorFactory.java:77)
	at org.gradle.internal.concurrent.ManagedExecutorImpl.stop(ManagedExecutorImpl.java:72)
	at org.gradle.internal.remote.internal.hub.MessageHub.stop(MessageHub.java:221)
	at org.gradle.internal.concurrent.CompositeStoppable.stop(CompositeStoppable.java:103)
	- locked <0x00000005e660ba68> (a org.gradle.internal.concurrent.CompositeStoppable)
	at org.gradle.internal.remote.internal.hub.MessageHubBackedObjectConnection.stop(MessageHubBackedObjectConnection.java:128)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:139)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:64)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.run(GradleWorkerMain.java:62)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.main(GradleWorkerMain.java:67)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007f7ac424b800 nid=0x68e0 waiting on condition  [0x00007f7aa833b000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ref.Reference.waitForReferencePendingList(java.base@10.0.1/Native Method)
	at java.lang.ref.Reference.processPendingReferences(java.base@10.0.1/Reference.java:174)
	at java.lang.ref.Reference.access$000(java.base@10.0.1/Reference.java:44)
	at java.lang.ref.Reference$ReferenceHandler.run(java.base@10.0.1/Reference.java:138)

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007f7ac424d800 nid=0x68e1 in Object.wait()  [0x00007f7aa823a000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000005e5e43f30> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:151)
	- waiting to re-lock in wait() <0x00000005e5e43f30> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:172)
	at java.lang.ref.Finalizer$FinalizerThread.run(java.base@10.0.1/Finalizer.java:216)

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007f7ac4257000 nid=0x68e4 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007f7ac4261000 nid=0x68e5 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C2 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007f7ac4263000 nid=0x68e6 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C2 CompilerThread2" #7 daemon prio=9 os_prio=0 tid=0x00007f7ac4265000 nid=0x68e7 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C1 CompilerThread3" #8 daemon prio=9 os_prio=0 tid=0x00007f7ac4267000 nid=0x68e8 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"Sweeper thread" #9 daemon prio=9 os_prio=0 tid=0x00007f7ac4269000 nid=0x68e9 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #10 daemon prio=9 os_prio=0 tid=0x00007f7ac4316800 nid=0x6903 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Common-Cleaner" #11 daemon prio=8 os_prio=0 tid=0x00007f7ac432f000 nid=0x6906 in Object.wait()  [0x00007f7a5dbf9000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000005e5e56770> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:151)
	- waiting to re-lock in wait() <0x00000005e5e56770> (a java.lang.ref.ReferenceQueue$Lock)
	at jdk.internal.ref.CleanerImpl.run(java.base@10.0.1/CleanerImpl.java:148)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)
	at jdk.internal.misc.InnocuousThread.run(java.base@10.0.1/InnocuousThread.java:134)

"/0:0:0:0:0:0:0:1:59640 to /0:0:0:0:0:0:0:1:41923 workers Thread 3" #15 prio=5 os_prio=0 tid=0x00007f7ac47d4800 nid=0x6949 runnable  [0x00007f7a5d0ca000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.EPollArrayWrapper.epollWait(java.base@10.0.1/Native Method)
	at sun.nio.ch.EPollArrayWrapper.poll(java.base@10.0.1/EPollArrayWrapper.java:265)
	at sun.nio.ch.EPollSelectorImpl.doSelect(java.base@10.0.1/EPollSelectorImpl.java:92)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(java.base@10.0.1/SelectorImpl.java:89)
	- locked <0x00000005e5e8c530> (a sun.nio.ch.Util$2)
	- locked <0x00000005e5e8c540> (a java.util.Collections$UnmodifiableSet)
	- locked <0x00000005e5e8c4e8> (a sun.nio.ch.EPollSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:100)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:104)
	at org.gradle.internal.remote.internal.inet.SocketConnection$SocketInputStream.read(SocketConnection.java:178)
	at com.esotericsoftware.kryo.io.Input.fill(Input.java:139)
	at com.esotericsoftware.kryo.io.Input.require(Input.java:159)
	at com.esotericsoftware.kryo.io.Input.readByte(Input.java:255)
	at org.gradle.internal.serialize.kryo.KryoBackedDecoder.readByte(KryoBackedDecoder.java:80)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:63)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:52)
	at org.gradle.internal.remote.internal.inet.SocketConnection.receive(SocketConnection.java:79)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionReceive.run(MessageHub.java:263)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"process reaper" #18 daemon prio=10 os_prio=0 tid=0x00007f79f427b800 nid=0x697e waiting on condition  [0x00007f7aa8038000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000005e5eba2d0> (a java.util.concurrent.SynchronousQueue$TransferStack)
	at java.util.concurrent.locks.LockSupport.parkNanos(java.base@10.0.1/LockSupport.java:234)
	at java.util.concurrent.SynchronousQueue$TransferStack.awaitFulfill(java.base@10.0.1/SynchronousQueue.java:462)
	at java.util.concurrent.SynchronousQueue$TransferStack.transfer(java.base@10.0.1/SynchronousQueue.java:361)
	at java.util.concurrent.SynchronousQueue.poll(java.base@10.0.1/SynchronousQueue.java:937)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1060)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Attach Listener" #20 daemon prio=9 os_prio=0 tid=0x00007f7a24001000 nid=0x6ee8 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"VM Thread" os_prio=0 tid=0x00007f7ac4241800 nid=0x68df runnable  

"GC Thread#0" os_prio=0 tid=0x00007f7ac402a000 nid=0x68c1 runnable  

"GC Thread#1" os_prio=0 tid=0x00007f7ac402b800 nid=0x68c2 runnable  

"GC Thread#2" os_prio=0 tid=0x00007f7ac402d800 nid=0x68c3 runnable  

"GC Thread#3" os_prio=0 tid=0x00007f7ac402f000 nid=0x68c4 runnable  

"GC Thread#4" os_prio=0 tid=0x00007f7ac4030800 nid=0x68c5 runnable  

"GC Thread#5" os_prio=0 tid=0x00007f7ac4032800 nid=0x68c6 runnable  

"GC Thread#6" os_prio=0 tid=0x00007f7ac4034000 nid=0x68c7 runnable  

"GC Thread#7" os_prio=0 tid=0x00007f7ac4036000 nid=0x68c8 runnable  

"GC Thread#8" os_prio=0 tid=0x00007f7ac4037800 nid=0x68c9 runnable  

"GC Thread#9" os_prio=0 tid=0x00007f7ac4039000 nid=0x68ca runnable  

"G1 Main Marker" os_prio=0 tid=0x00007f7ac4064000 nid=0x68cc runnable  

"G1 Conc#0" os_prio=0 tid=0x00007f7ac4065800 nid=0x68cd runnable  

"G1 Conc#1" os_prio=0 tid=0x00007f7ac4067800 nid=0x68cf runnable  

"G1 Conc#2" os_prio=0 tid=0x00007f7ac4069000 nid=0x68d1 runnable  

"G1 Refine#0" os_prio=0 tid=0x00007f7ac41cc000 nid=0x68d4 runnable  

"G1 Refine#1" os_prio=0 tid=0x00007f7ac41ce000 nid=0x68d5 runnable  

"G1 Refine#2" os_prio=0 tid=0x00007f7ac41cf800 nid=0x68d6 runnable  

"G1 Refine#3" os_prio=0 tid=0x00007f7ac41d1800 nid=0x68d7 runnable  

"G1 Refine#4" os_prio=0 tid=0x00007f7ac41d3000 nid=0x68d8 runnable  

"G1 Refine#5" os_prio=0 tid=0x00007f7ac41d5000 nid=0x68d9 runnable  

"G1 Refine#6" os_prio=0 tid=0x00007f7ac41d6800 nid=0x68da runnable  

"G1 Refine#7" os_prio=0 tid=0x00007f7ac41d8800 nid=0x68db runnable  

"G1 Refine#8" os_prio=0 tid=0x00007f7ac41da000 nid=0x68dc runnable  

"G1 Refine#9" os_prio=0 tid=0x00007f7ac41dc000 nid=0x68dd runnable  

"G1 Young RemSet Sampling" os_prio=0 tid=0x00007f7ac41dd800 nid=0x68de runnable  
"VM Periodic Task Thread" os_prio=0 tid=0x00007f7ac4318800 nid=0x6904 waiting on condition  

JNI global references: 70

```

Daemon:
```
[:~/work/elastic/elasticsearch] feature/31496_run_test_with_gradle(+1/-0) ± jstack 26715
2018-06-22 18:38:55
Full thread dump OpenJDK 64-Bit Server VM (10.0.1+10 mixed mode):

Threads class SMR info:
_java_thread_list=0x00007f0eb4000b20, length=51, elements={
0x00007f1014011000, 0x00007f1014261000, 0x00007f1014263000, 0x00007f101426c800,
0x00007f1014276800, 0x00007f1014278800, 0x00007f101427a800, 0x00007f101427c800,
0x00007f101427e800, 0x00007f101430b000, 0x00007f1014324000, 0x00007f1015074800,
0x00007f10150bb800, 0x00007f0f58033000, 0x00007f0f54004800, 0x00007f0f54008800,
0x00007f0f54009800, 0x00007f0f40005000, 0x00007f0f40007800, 0x00007f0f40081000,
0x00007f0f4013d800, 0x00007f0f40138800, 0x00007f0f409a2800, 0x00007f0f40c74800,
0x00007f0f40c81800, 0x00007f0f40d3b800, 0x00007f0f40d3c800, 0x00007f0f40e83800,
0x00007f0f1c085000, 0x00007f0f1c084000, 0x00007f0f1c081000, 0x00007f0f1c07e800,
0x00007f0f1c085800, 0x00007f0f1c087000, 0x00007f0f1c089000, 0x00007f0f1c08b000,
0x00007f0f1c08d000, 0x00007f0f1c08f000, 0x00007f0f1c091000, 0x00007f0f1c093000,
0x00007f0f1c0ae800, 0x00007f0f1c0b1000, 0x00007f0eb0027800, 0x00007f0ea8007000,
0x00007f0ea8014800, 0x00007f0ea8016800, 0x00007f0eb400d800, 0x00007f0eb0029800,
0x00007f0eb0052000, 0x00007f0eb0054800, 0x00007f0f84001000
}

"main" #1 prio=5 os_prio=0 tid=0x00007f1014011000 nid=0x685c waiting on condition  [0x00007f101daee000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c027b748> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.launcher.daemon.server.DaemonStateCoordinator.awaitStop(DaemonStateCoordinator.java:95)
	at org.gradle.launcher.daemon.server.Daemon.awaitExpiration(Daemon.java:247)
	at org.gradle.launcher.daemon.server.Daemon.stopOnExpiration(Daemon.java:221)
	at org.gradle.launcher.daemon.bootstrap.DaemonMain.doAction(DaemonMain.java:131)
	at org.gradle.launcher.bootstrap.EntryPoint.run(EntryPoint.java:45)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(java.base@10.0.1/Native Method)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke(java.base@10.0.1/NativeMethodAccessorImpl.java:62)
	at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(java.base@10.0.1/DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(java.base@10.0.1/Method.java:564)
	at org.gradle.launcher.bootstrap.ProcessBootstrap.runNoExit(ProcessBootstrap.java:60)
	at org.gradle.launcher.bootstrap.ProcessBootstrap.run(ProcessBootstrap.java:37)
	at org.gradle.launcher.daemon.bootstrap.GradleDaemon.main(GradleDaemon.java:22)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007f1014261000 nid=0x687a waiting on condition  [0x00007f0ff8604000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ref.Reference.waitForReferencePendingList(java.base@10.0.1/Native Method)
	at java.lang.ref.Reference.processPendingReferences(java.base@10.0.1/Reference.java:174)
	at java.lang.ref.Reference.access$000(java.base@10.0.1/Reference.java:44)
	at java.lang.ref.Reference$ReferenceHandler.run(java.base@10.0.1/Reference.java:138)

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007f1014263000 nid=0x687b in Object.wait()  [0x00007f0ff8503000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000000c0270f00> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:151)
	- waiting to re-lock in wait() <0x00000000c0270f00> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:172)
	at java.lang.ref.Finalizer$FinalizerThread.run(java.base@10.0.1/Finalizer.java:216)

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007f101426c800 nid=0x687c runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007f1014276800 nid=0x687d waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C2 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007f1014278800 nid=0x687e waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C2 CompilerThread2" #7 daemon prio=9 os_prio=0 tid=0x00007f101427a800 nid=0x687f waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C1 CompilerThread3" #8 daemon prio=9 os_prio=0 tid=0x00007f101427c800 nid=0x6880 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"Sweeper thread" #9 daemon prio=9 os_prio=0 tid=0x00007f101427e800 nid=0x6881 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #10 daemon prio=9 os_prio=0 tid=0x00007f101430b000 nid=0x6882 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Common-Cleaner" #11 daemon prio=8 os_prio=0 tid=0x00007f1014324000 nid=0x6884 in Object.wait()  [0x00007f0fc11f7000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000000c02faf88> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@10.0.1/ReferenceQueue.java:151)
	- waiting to re-lock in wait() <0x00000000c02faf88> (a java.lang.ref.ReferenceQueue$Lock)
	at jdk.internal.ref.CleanerImpl.run(java.base@10.0.1/CleanerImpl.java:148)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)
	at jdk.internal.misc.InnocuousThread.run(java.base@10.0.1/InnocuousThread.java:134)

"Incoming local TCP Connector on port 33317" #14 prio=5 os_prio=0 tid=0x00007f1015074800 nid=0x6887 runnable  [0x00007f0f5fdfc000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.ServerSocketChannelImpl.accept0(java.base@10.0.1/Native Method)
	at sun.nio.ch.ServerSocketChannelImpl.accept(java.base@10.0.1/ServerSocketChannelImpl.java:424)
	at sun.nio.ch.ServerSocketChannelImpl.accept(java.base@10.0.1/ServerSocketChannelImpl.java:252)
	- locked <0x00000000c0279268> (a java.lang.Object)
	at org.gradle.internal.remote.internal.inet.TcpIncomingConnector$Receiver.run(TcpIncomingConnector.java:102)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Daemon periodic checks" #15 prio=5 os_prio=0 tid=0x00007f10150bb800 nid=0x6888 waiting on condition  [0x00007f0f5fcfb000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02764e8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(java.base@10.0.1/LockSupport.java:234)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(java.base@10.0.1/AbstractQueuedSynchronizer.java:2117)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:1182)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:899)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Daemon" #16 prio=5 os_prio=0 tid=0x00007f0f58033000 nid=0x6889 waiting on condition  [0x00007f0f5fbf9000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c027b748> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.launcher.daemon.server.DaemonStateCoordinator.waitForCommandCompletion(DaemonStateCoordinator.java:313)
	at org.gradle.launcher.daemon.server.DaemonStateCoordinator.runCommand(DaemonStateCoordinator.java:302)
	at org.gradle.launcher.daemon.server.exec.StartBuildOrRespondWithBusy.doBuild(StartBuildOrRespondWithBusy.java:54)
	at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.ReturnResult.execute(ReturnResult.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.api.HandleReportStatus.execute(HandleReportStatus.java:33)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.HandleCancel.execute(HandleCancel.java:37)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.api.HandleStop.execute(HandleStop.java:45)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.DaemonCommandExecuter.executeCommand(DaemonCommandExecuter.java:48)
	at org.gradle.launcher.daemon.server.DefaultIncomingConnectionHandler$ConnectionWorker.handleCommand(DefaultIncomingConnectionHandler.java:160)
	at org.gradle.launcher.daemon.server.DefaultIncomingConnectionHandler$ConnectionWorker.receiveAndHandleCommand(DefaultIncomingConnectionHandler.java:133)
	at org.gradle.launcher.daemon.server.DefaultIncomingConnectionHandler$ConnectionWorker.run(DefaultIncomingConnectionHandler.java:121)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Handler for socket connection from /0:0:0:0:0:0:0:1:33317 to /0:0:0:0:0:0:0:1:49038" #17 prio=5 os_prio=0 tid=0x00007f0f54004800 nid=0x688a runnable  [0x00007f0f5faf9000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.EPollArrayWrapper.epollWait(java.base@10.0.1/Native Method)
	at sun.nio.ch.EPollArrayWrapper.poll(java.base@10.0.1/EPollArrayWrapper.java:265)
	at sun.nio.ch.EPollSelectorImpl.doSelect(java.base@10.0.1/EPollSelectorImpl.java:92)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(java.base@10.0.1/SelectorImpl.java:89)
	- locked <0x00000000c02715e0> (a sun.nio.ch.Util$2)
	- locked <0x00000000c02715f0> (a java.util.Collections$UnmodifiableSet)
	- locked <0x00000000c0271598> (a sun.nio.ch.EPollSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:100)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:104)
	at org.gradle.internal.remote.internal.inet.SocketConnection$SocketInputStream.read(SocketConnection.java:178)
	at com.esotericsoftware.kryo.io.Input.fill(Input.java:139)
	at com.esotericsoftware.kryo.io.Input.require(Input.java:159)
	at com.esotericsoftware.kryo.io.Input.readInt(Input.java:308)
	at org.gradle.internal.serialize.kryo.KryoBackedDecoder.readSmallInt(KryoBackedDecoder.java:120)
	at org.gradle.internal.serialize.DefaultSerializerRegistry$TaggedTypeSerializer.read(DefaultSerializerRegistry.java:139)
	at org.gradle.internal.serialize.Serializers$StatefulSerializerAdapter$1.read(Serializers.java:36)
	at org.gradle.internal.remote.internal.inet.SocketConnection.receive(SocketConnection.java:79)
	at org.gradle.launcher.daemon.server.SynchronizedDispatchConnection.receive(SynchronizedDispatchConnection.java:68)
	at org.gradle.launcher.daemon.server.DefaultDaemonConnection$1.run(DefaultDaemonConnection.java:64)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cancel handler" #18 prio=5 os_prio=0 tid=0x00007f0f54008800 nid=0x688b waiting on condition  [0x00007f0f5f9f8000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c0278760> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.launcher.daemon.server.DefaultDaemonConnection$CommandQueue$1.run(DefaultDaemonConnection.java:232)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Daemon worker" #19 prio=5 os_prio=0 tid=0x00007f0f54009800 nid=0x688c in Object.wait()  [0x00007f0f5f8f5000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000000c07b5fb0> (a java.lang.Object)
	at java.lang.Object.wait(java.base@10.0.1/Object.java:328)
	at org.gradle.internal.resources.DefaultResourceLockCoordinationService.withStateLock(DefaultResourceLockCoordinationService.java:51)
	- waiting to re-lock in wait() <0x00000000c07b5fb0> (a java.lang.Object)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor.awaitCompletion(DefaultTaskPlanExecutor.java:85)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor.process(DefaultTaskPlanExecutor.java:75)
	at org.gradle.execution.taskgraph.DefaultTaskExecutionGraph.execute(DefaultTaskExecutionGraph.java:131)
	at org.gradle.execution.SelectedTaskExecutionAction.execute(SelectedTaskExecutionAction.java:37)
	at org.gradle.execution.DefaultBuildExecuter.execute(DefaultBuildExecuter.java:37)
	at org.gradle.execution.DefaultBuildExecuter.access$000(DefaultBuildExecuter.java:23)
	at org.gradle.execution.DefaultBuildExecuter$1.proceed(DefaultBuildExecuter.java:43)
	at org.gradle.execution.DryRunBuildExecutionAction.execute(DryRunBuildExecutionAction.java:46)
	at org.gradle.execution.DefaultBuildExecuter.execute(DefaultBuildExecuter.java:37)
	at org.gradle.execution.DefaultBuildExecuter.execute(DefaultBuildExecuter.java:30)
	at org.gradle.initialization.DefaultGradleLauncher$ExecuteTasks.run(DefaultGradleLauncher.java:343)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:317)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:309)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:185)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:97)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.initialization.DefaultGradleLauncher.runTasks(DefaultGradleLauncher.java:212)
	at org.gradle.initialization.DefaultGradleLauncher.doBuildStages(DefaultGradleLauncher.java:140)
	at org.gradle.initialization.DefaultGradleLauncher.executeTasks(DefaultGradleLauncher.java:115)
	at org.gradle.internal.invocation.GradleBuildController$1.call(GradleBuildController.java:77)
	at org.gradle.internal.invocation.GradleBuildController$1.call(GradleBuildController.java:74)
	at org.gradle.internal.work.DefaultWorkerLeaseService.withLocks(DefaultWorkerLeaseService.java:152)
	at org.gradle.internal.work.StopShieldingWorkerLeaseService.withLocks(StopShieldingWorkerLeaseService.java:38)
	at org.gradle.internal.invocation.GradleBuildController.doBuild(GradleBuildController.java:96)
	at org.gradle.internal.invocation.GradleBuildController.run(GradleBuildController.java:74)
	at org.gradle.tooling.internal.provider.ExecuteBuildActionRunner.run(ExecuteBuildActionRunner.java:28)
	at org.gradle.launcher.exec.ChainingBuildActionRunner.run(ChainingBuildActionRunner.java:35)
	at org.gradle.tooling.internal.provider.ValidatingBuildActionRunner.run(ValidatingBuildActionRunner.java:32)
	at org.gradle.launcher.exec.RunAsBuildOperationBuildActionRunner$3.run(RunAsBuildOperationBuildActionRunner.java:47)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:317)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:309)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:185)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:97)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.launcher.exec.RunAsBuildOperationBuildActionRunner.run(RunAsBuildOperationBuildActionRunner.java:43)
	at org.gradle.tooling.internal.provider.SubscribableBuildActionRunner.run(SubscribableBuildActionRunner.java:51)
	at org.gradle.launcher.exec.InProcessBuildActionExecuter$1.transform(InProcessBuildActionExecuter.java:50)
	at org.gradle.launcher.exec.InProcessBuildActionExecuter$1.transform(InProcessBuildActionExecuter.java:46)
	at org.gradle.composite.internal.DefaultRootBuildState.run(DefaultRootBuildState.java:65)
	at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:46)
	at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:32)
	at org.gradle.launcher.exec.BuildTreeScopeBuildActionExecuter.execute(BuildTreeScopeBuildActionExecuter.java:39)
	at org.gradle.launcher.exec.BuildTreeScopeBuildActionExecuter.execute(BuildTreeScopeBuildActionExecuter.java:25)
	at org.gradle.tooling.internal.provider.ContinuousBuildActionExecuter.execute(ContinuousBuildActionExecuter.java:80)
	at org.gradle.tooling.internal.provider.ContinuousBuildActionExecuter.execute(ContinuousBuildActionExecuter.java:53)
	at org.gradle.tooling.internal.provider.ServicesSetupBuildActionExecuter.execute(ServicesSetupBuildActionExecuter.java:62)
	at org.gradle.tooling.internal.provider.ServicesSetupBuildActionExecuter.execute(ServicesSetupBuildActionExecuter.java:34)
	at org.gradle.tooling.internal.provider.GradleThreadBuildActionExecuter.execute(GradleThreadBuildActionExecuter.java:36)
	at org.gradle.tooling.internal.provider.GradleThreadBuildActionExecuter.execute(GradleThreadBuildActionExecuter.java:25)
	at org.gradle.tooling.internal.provider.ParallelismConfigurationBuildActionExecuter.execute(ParallelismConfigurationBuildActionExecuter.java:43)
	at org.gradle.tooling.internal.provider.ParallelismConfigurationBuildActionExecuter.execute(ParallelismConfigurationBuildActionExecuter.java:29)
	at org.gradle.tooling.internal.provider.StartParamsValidatingActionExecuter.execute(StartParamsValidatingActionExecuter.java:59)
	at org.gradle.tooling.internal.provider.StartParamsValidatingActionExecuter.execute(StartParamsValidatingActionExecuter.java:31)
	at org.gradle.tooling.internal.provider.SessionFailureReportingActionExecuter.execute(SessionFailureReportingActionExecuter.java:59)
	at org.gradle.tooling.internal.provider.SessionFailureReportingActionExecuter.execute(SessionFailureReportingActionExecuter.java:44)
	at org.gradle.tooling.internal.provider.SetupLoggingActionExecuter.execute(SetupLoggingActionExecuter.java:46)
	at org.gradle.tooling.internal.provider.SetupLoggingActionExecuter.execute(SetupLoggingActionExecuter.java:30)
	at org.gradle.launcher.daemon.server.exec.ExecuteBuild.doBuild(ExecuteBuild.java:67)
	at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.WatchForDisconnection.execute(WatchForDisconnection.java:37)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.ResetDeprecationLogger.execute(ResetDeprecationLogger.java:26)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.RequestStopIfSingleUsedDaemon.execute(RequestStopIfSingleUsedDaemon.java:34)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.ForwardClientInput$2.call(ForwardClientInput.java:74)
	at org.gradle.launcher.daemon.server.exec.ForwardClientInput$2.call(ForwardClientInput.java:72)
	at org.gradle.util.Swapper.swap(Swapper.java:38)
	at org.gradle.launcher.daemon.server.exec.ForwardClientInput.execute(ForwardClientInput.java:72)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.LogAndCheckHealth.execute(LogAndCheckHealth.java:55)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.LogToClient.doBuild(LogToClient.java:62)
	at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.EstablishBuildEnvironment.doBuild(EstablishBuildEnvironment.java:82)
	at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
	at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
	at org.gradle.launcher.daemon.server.exec.StartBuildOrRespondWithBusy$1.run(StartBuildOrRespondWithBusy.java:50)
	at org.gradle.launcher.daemon.server.DaemonStateCoordinator$1.run(DaemonStateCoordinator.java:295)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Asynchronous log dispatcher for DefaultDaemonConnection: socket connection from /0:0:0:0:0:0:0:1:33317 to /0:0:0:0:0:0:0:1:49038" #20 prio=5 os_prio=0 tid=0x00007f0f40005000 nid=0x688f sleeping [0x00007f0f5f7f6000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(java.base@10.0.1/Native Method)
	at org.gradle.launcher.daemon.server.exec.LogToClient$AsynchronousLogDispatcher.run(LogToClient.java:108)

"Stdin handler" #21 prio=5 os_prio=0 tid=0x00007f0f40007800 nid=0x6890 waiting on condition  [0x00007f0f5f6f5000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c0277360> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.launcher.daemon.server.DefaultDaemonConnection$CommandQueue$1.run(DefaultDaemonConnection.java:232)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for file hash cache (/home/alpar/.gradle/caches/4.8.1/fileHashes)" #22 prio=5 os_prio=0 tid=0x00007f0f40081000 nid=0x6891 waiting on condition  [0x00007f0f5f3f4000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c0291b70> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"File lock request listener" #23 prio=5 os_prio=0 tid=0x00007f0f4013d800 nid=0x6892 runnable  [0x00007f0f5f2f3000]
   java.lang.Thread.State: RUNNABLE
	at java.net.PlainDatagramSocketImpl.receive0(java.base@10.0.1/Native Method)
	- locked <0x00000000c027a5f0> (a java.net.PlainDatagramSocketImpl)
	at java.net.AbstractPlainDatagramSocketImpl.receive(java.base@10.0.1/AbstractPlainDatagramSocketImpl.java:180)
	- locked <0x00000000c027a5f0> (a java.net.PlainDatagramSocketImpl)
	at java.net.DatagramSocket.receive(java.base@10.0.1/DatagramSocket.java:814)
	- locked <0x00000000c027a698> (a java.net.DatagramPacket)
	- locked <0x00000000c027a738> (a java.net.DatagramSocket)
	at org.gradle.cache.internal.locklistener.FileLockCommunicator.receive(FileLockCommunicator.java:79)
	at org.gradle.cache.internal.locklistener.DefaultFileLockContentionHandler$1.doRun(DefaultFileLockContentionHandler.java:108)
	at org.gradle.cache.internal.locklistener.DefaultFileLockContentionHandler$1.run(DefaultFileLockContentionHandler.java:94)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for file hash cache (/home/alpar/work/elastic/gradle_test_randomizedtesting_deadlock/.gradle/4.8.1/fileHashes)" #24 prio=5 os_prio=0 tid=0x00007f0f40138800 nid=0x6893 waiting on condition  [0x00007f0f5eff2000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02940e0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for Artifact transforms cache (/home/alpar/.gradle/caches/transforms-1)" #25 prio=5 os_prio=0 tid=0x00007f0f409a2800 nid=0x6894 waiting on condition  [0x00007f0f5e2f1000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000dde30a20> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Memory manager" #26 prio=5 os_prio=0 tid=0x00007f0f40c74800 nid=0x68a0 waiting on condition  [0x00007f0f5ddf0000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000ddfd4858> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(java.base@10.0.1/LockSupport.java:234)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(java.base@10.0.1/AbstractQueuedSynchronizer.java:2117)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:1182)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(java.base@10.0.1/ScheduledThreadPoolExecutor.java:899)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for file content cache (/home/alpar/work/elastic/gradle_test_randomizedtesting_deadlock/.gradle/4.8.1/fileContent)" #27 prio=5 os_prio=0 tid=0x00007f0f40c81800 nid=0x68a1 waiting on condition  [0x00007f0f5dcef000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000dd5d41d0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for task history cache (/home/alpar/work/elastic/gradle_test_randomizedtesting_deadlock/.gradle/4.8.1/taskHistory)" #28 prio=5 os_prio=0 tid=0x00007f0f40d3b800 nid=0x68a2 waiting on condition  [0x00007f0f5dbee000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000dd3a83d8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Cache worker for Build Output Cleanup Cache (/home/alpar/work/elastic/gradle_test_randomizedtesting_deadlock/.gradle/buildOutputCleanup)" #29 prio=5 os_prio=0 tid=0x00007f0f40d3c800 nid=0x68a3 waiting on condition  [0x00007f0f5d6b3000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000dd3b7c58> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.ArrayBlockingQueue.take(java.base@10.0.1/ArrayBlockingQueue.java:417)
	at org.gradle.cache.internal.CacheAccessWorker.takeFromQueue(CacheAccessWorker.java:168)
	at org.gradle.cache.internal.CacheAccessWorker.run(CacheAccessWorker.java:132)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Task worker for ':'" #30 prio=5 os_prio=0 tid=0x00007f0f40e83800 nid=0x68a4 waiting on condition  [0x00007f0f5daeb000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000dacd6c20> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.dispatch.AsyncDispatch.stop(AsyncDispatch.java:181)
	at org.gradle.internal.concurrent.CompositeStoppable.stop(CompositeStoppable.java:103)
	- locked <0x00000000dace90d8> (a org.gradle.internal.concurrent.CompositeStoppable)
	at org.gradle.internal.actor.internal.DefaultActorFactory$NonBlockingActor.stop(DefaultActorFactory.java:147)
	at org.gradle.internal.concurrent.CompositeStoppable.stop(CompositeStoppable.java:103)
	- locked <0x00000000dace8fc8> (a org.gradle.internal.concurrent.CompositeStoppable)
	at org.gradle.api.internal.tasks.testing.processors.MaxNParallelTestClassProcessor.stop(MaxNParallelTestClassProcessor.java:86)
	at org.gradle.api.internal.tasks.testing.processors.RunPreviousFailedFirstTestClassProcessor.stop(RunPreviousFailedFirstTestClassProcessor.java:63)
	at org.gradle.api.internal.tasks.testing.processors.PatternMatchTestClassProcessor.stop(PatternMatchTestClassProcessor.java:48)
	at org.gradle.api.internal.tasks.testing.processors.TestMainAction.run(TestMainAction.java:57)
	at org.gradle.api.internal.tasks.testing.detection.DefaultTestExecuter.execute(DefaultTestExecuter.java:116)
	at org.gradle.api.internal.tasks.testing.detection.DefaultTestExecuter.execute(DefaultTestExecuter.java:51)
	at org.gradle.api.tasks.testing.AbstractTestTask.executeTests(AbstractTestTask.java:469)
	at org.gradle.api.tasks.testing.Test.executeTests(Test.java:603)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(java.base@10.0.1/Native Method)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke(java.base@10.0.1/NativeMethodAccessorImpl.java:62)
	at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(java.base@10.0.1/DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(java.base@10.0.1/Method.java:564)
	at org.gradle.internal.reflect.JavaMethod.invoke(JavaMethod.java:73)
	at org.gradle.api.internal.project.taskfactory.StandardTaskAction.doExecute(StandardTaskAction.java:46)
	at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:39)
	at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:26)
	at org.gradle.api.internal.AbstractTask$TaskActionWrapper.execute(AbstractTask.java:794)
	at org.gradle.api.internal.AbstractTask$TaskActionWrapper.execute(AbstractTask.java:761)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter$1.run(ExecuteActionsTaskExecuter.java:131)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:317)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:309)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:185)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:97)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeAction(ExecuteActionsTaskExecuter.java:120)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:99)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.execute(ExecuteActionsTaskExecuter.java:77)
	at org.gradle.api.internal.tasks.execution.OutputDirectoryCreatingTaskExecuter.execute(OutputDirectoryCreatingTaskExecuter.java:51)
	at org.gradle.api.internal.tasks.execution.SkipUpToDateTaskExecuter.execute(SkipUpToDateTaskExecuter.java:59)
	at org.gradle.api.internal.tasks.execution.ResolveTaskOutputCachingStateExecuter.execute(ResolveTaskOutputCachingStateExecuter.java:54)
	at org.gradle.api.internal.tasks.execution.ValidatingTaskExecuter.execute(ValidatingTaskExecuter.java:59)
	at org.gradle.api.internal.tasks.execution.SkipEmptySourceFilesTaskExecuter.execute(SkipEmptySourceFilesTaskExecuter.java:101)
	at org.gradle.api.internal.tasks.execution.FinalizeInputFilePropertiesTaskExecuter.execute(FinalizeInputFilePropertiesTaskExecuter.java:44)
	at org.gradle.api.internal.tasks.execution.CleanupStaleOutputsExecuter.execute(CleanupStaleOutputsExecuter.java:91)
	at org.gradle.api.internal.tasks.execution.ResolveTaskArtifactStateTaskExecuter.execute(ResolveTaskArtifactStateTaskExecuter.java:62)
	at org.gradle.api.internal.tasks.execution.SkipTaskWithNoActionsExecuter.execute(SkipTaskWithNoActionsExecuter.java:59)
	at org.gradle.api.internal.tasks.execution.SkipOnlyIfTaskExecuter.execute(SkipOnlyIfTaskExecuter.java:54)
	at org.gradle.api.internal.tasks.execution.ExecuteAtMostOnceTaskExecuter.execute(ExecuteAtMostOnceTaskExecuter.java:43)
	at org.gradle.api.internal.tasks.execution.CatchExceptionTaskExecuter.execute(CatchExceptionTaskExecuter.java:34)
	at org.gradle.api.internal.tasks.execution.EventFiringTaskExecuter$1.run(EventFiringTaskExecuter.java:51)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:317)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:309)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:185)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:97)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.api.internal.tasks.execution.EventFiringTaskExecuter.execute(EventFiringTaskExecuter.java:46)
	at org.gradle.execution.taskgraph.DefaultTaskExecutionGraph$ExecuteTaskAction.execute(DefaultTaskExecutionGraph.java:262)
	at org.gradle.execution.taskgraph.DefaultTaskExecutionGraph$ExecuteTaskAction.execute(DefaultTaskExecutionGraph.java:246)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$TaskExecutorWorker$1.execute(DefaultTaskPlanExecutor.java:136)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$TaskExecutorWorker$1.execute(DefaultTaskPlanExecutor.java:130)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$TaskExecutorWorker.execute(DefaultTaskPlanExecutor.java:201)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$TaskExecutorWorker.executeWithTask(DefaultTaskPlanExecutor.java:192)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$TaskExecutorWorker.run(DefaultTaskPlanExecutor.java:130)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations" #41 prio=5 os_prio=0 tid=0x00007f0f1c085000 nid=0x68af waiting on condition  [0x00007f0f5ccab000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 2" #42 prio=5 os_prio=0 tid=0x00007f0f1c084000 nid=0x68b0 waiting on condition  [0x00007f0f5cbaa000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 3" #43 prio=5 os_prio=0 tid=0x00007f0f1c081000 nid=0x68b1 waiting on condition  [0x00007f0f5caa9000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 4" #44 prio=5 os_prio=0 tid=0x00007f0f1c07e800 nid=0x68b2 waiting on condition  [0x00007f0f5c9a8000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 5" #45 prio=5 os_prio=0 tid=0x00007f0f1c085800 nid=0x68b3 waiting on condition  [0x00007f0f5c8a7000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 6" #46 prio=5 os_prio=0 tid=0x00007f0f1c087000 nid=0x68b4 waiting on condition  [0x00007f0f5c7a6000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 7" #47 prio=5 os_prio=0 tid=0x00007f0f1c089000 nid=0x68b5 waiting on condition  [0x00007f0f5c6a5000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 8" #48 prio=5 os_prio=0 tid=0x00007f0f1c08b000 nid=0x68b6 waiting on condition  [0x00007f0f5c5a4000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 9" #49 prio=5 os_prio=0 tid=0x00007f0f1c08d000 nid=0x68b7 waiting on condition  [0x00007f0f5c4a3000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 10" #50 prio=5 os_prio=0 tid=0x00007f0f1c08f000 nid=0x68b8 waiting on condition  [0x00007f0f5c3a2000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 11" #51 prio=5 os_prio=0 tid=0x00007f0f1c091000 nid=0x68b9 waiting on condition  [0x00007f0f5c2a1000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Build operations Thread 12" #52 prio=5 os_prio=0 tid=0x00007f0f1c093000 nid=0x68ba waiting on condition  [0x00007f0f5c1a0000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000c02e2ab0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at java.util.concurrent.LinkedBlockingQueue.take(java.base@10.0.1/LinkedBlockingQueue.java:435)
	at java.util.concurrent.ThreadPoolExecutor.getTask(java.base@10.0.1/ThreadPoolExecutor.java:1061)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1121)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Dispatch org.gradle.api.internal.tasks.testing.results.AttachParentTestResultProcessor@5796616e" #53 prio=5 os_prio=0 tid=0x00007f0f1c0ae800 nid=0x68bb waiting on condition  [0x00007f0ebfffe000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000db32f968> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.dispatch.AsyncDispatch.dispatchMessages(AsyncDispatch.java:115)
	at org.gradle.internal.dispatch.AsyncDispatch.access$000(AsyncDispatch.java:34)
	at org.gradle.internal.dispatch.AsyncDispatch$1.run(AsyncDispatch.java:73)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Dispatch org.gradle.api.internal.tasks.testing.processors.RestartEveryNTestClassProcessor@32a515c2" #54 prio=5 os_prio=0 tid=0x00007f0f1c0b1000 nid=0x68bc waiting on condition  [0x00007f0ebfefc000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000da9ab0e8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.process.internal.DefaultExecHandle.waitForFinish(DefaultExecHandle.java:300)
	at org.gradle.process.internal.worker.DefaultWorkerProcess.waitForStop(DefaultWorkerProcess.java:210)
	at org.gradle.process.internal.worker.DefaultWorkerProcessBuilder$MemoryRequestingWorkerProcess.waitForStop(DefaultWorkerProcessBuilder.java:228)
	at org.gradle.api.internal.tasks.testing.worker.ForkingTestClassProcessor.stop(ForkingTestClassProcessor.java:152)
	at org.gradle.api.internal.tasks.testing.processors.RestartEveryNTestClassProcessor.endBatch(RestartEveryNTestClassProcessor.java:77)
	at org.gradle.api.internal.tasks.testing.processors.RestartEveryNTestClassProcessor.stop(RestartEveryNTestClassProcessor.java:62)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(java.base@10.0.1/Native Method)
	at jdk.internal.reflect.NativeMethodAccessorImpl.invoke(java.base@10.0.1/NativeMethodAccessorImpl.java:62)
	at jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(java.base@10.0.1/DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(java.base@10.0.1/Method.java:564)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:35)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:24)
	at org.gradle.internal.dispatch.FailureHandlingDispatch.dispatch(FailureHandlingDispatch.java:29)
	at org.gradle.internal.dispatch.AsyncDispatch.dispatchMessages(AsyncDispatch.java:133)
	at org.gradle.internal.dispatch.AsyncDispatch.access$000(AsyncDispatch.java:34)
	at org.gradle.internal.dispatch.AsyncDispatch$1.run(AsyncDispatch.java:73)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Exec process" #56 prio=5 os_prio=0 tid=0x00007f0eb0027800 nid=0x68be in Object.wait()  [0x00007f0ebfafb000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@10.0.1/Native Method)
	- waiting on <0x00000000da8014b0> (a java.lang.ProcessImpl)
	at java.lang.Object.wait(java.base@10.0.1/Object.java:328)
	at java.lang.ProcessImpl.waitFor(java.base@10.0.1/ProcessImpl.java:494)
	- waiting to re-lock in wait() <0x00000000da8014b0> (a java.lang.ProcessImpl)
	at org.gradle.process.internal.ExecHandleRunner.run(ExecHandleRunner.java:80)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"process reaper" #57 daemon prio=10 os_prio=0 tid=0x00007f0ea8007000 nid=0x68cb runnable  [0x00007f0ff8065000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ProcessHandleImpl.waitForProcessExit0(java.base@10.0.1/Native Method)
	at java.lang.ProcessHandleImpl.access$000(java.base@10.0.1/ProcessHandleImpl.java:50)
	at java.lang.ProcessHandleImpl$1.run(java.base@10.0.1/ProcessHandleImpl.java:138)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Exec process Thread 3" #60 prio=5 os_prio=0 tid=0x00007f0ea8014800 nid=0x68d2 runnable  [0x00007f0ebf7f8000]
   java.lang.Thread.State: RUNNABLE
	at java.io.FileInputStream.readBytes(java.base@10.0.1/Native Method)
	at java.io.FileInputStream.read(java.base@10.0.1/FileInputStream.java:280)
	at java.io.BufferedInputStream.fill(java.base@10.0.1/BufferedInputStream.java:252)
	at java.io.BufferedInputStream.read1(java.base@10.0.1/BufferedInputStream.java:292)
	at java.io.BufferedInputStream.read(java.base@10.0.1/BufferedInputStream.java:351)
	- locked <0x00000000da8284b0> (a java.lang.ProcessImpl$ProcessPipeInputStream)
	at java.io.FilterInputStream.read(java.base@10.0.1/FilterInputStream.java:107)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.forwardContent(ExecOutputHandleRunner.java:61)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.run(ExecOutputHandleRunner.java:51)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Exec process Thread 4" #61 prio=5 os_prio=0 tid=0x00007f0ea8016800 nid=0x68d3 runnable  [0x00007f0ebf6f7000]
   java.lang.Thread.State: RUNNABLE
	at java.io.FileInputStream.readBytes(java.base@10.0.1/Native Method)
	at java.io.FileInputStream.read(java.base@10.0.1/FileInputStream.java:280)
	at java.io.BufferedInputStream.fill(java.base@10.0.1/BufferedInputStream.java:252)
	at java.io.BufferedInputStream.read1(java.base@10.0.1/BufferedInputStream.java:292)
	at java.io.BufferedInputStream.read(java.base@10.0.1/BufferedInputStream.java:351)
	- locked <0x00000000da826390> (a java.lang.ProcessImpl$ProcessPipeInputStream)
	at java.io.FilterInputStream.read(java.base@10.0.1/FilterInputStream.java:107)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.forwardContent(ExecOutputHandleRunner.java:61)
	at org.gradle.process.internal.streams.ExecOutputHandleRunner.run(ExecOutputHandleRunner.java:51)
	at org.gradle.internal.operations.CurrentBuildOperationPreservingRunnable.run(CurrentBuildOperationPreservingRunnable.java:42)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:41923 to /0:0:0:0:0:0:0:1:59640 workers" #62 prio=5 os_prio=0 tid=0x00007f0eb400d800 nid=0x6913 waiting on condition  [0x00007f0ebf9fa000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000da61aab8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$Handler.run(MessageHub.java:393)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:41923 to /0:0:0:0:0:0:0:1:59640 workers Thread 2" #63 prio=5 os_prio=0 tid=0x00007f0eb0029800 nid=0x691b waiting on condition  [0x00007f0ebf5f6000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000da59e958> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$Handler.run(MessageHub.java:393)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:41923 to /0:0:0:0:0:0:0:1:59640 workers Thread 3" #64 prio=5 os_prio=0 tid=0x00007f0eb0052000 nid=0x6921 waiting on condition  [0x00007f0ebf4f5000]
   java.lang.Thread.State: WAITING (parking)
	at jdk.internal.misc.Unsafe.park(java.base@10.0.1/Native Method)
	- parking to wait for  <0x00000000da446528> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(java.base@10.0.1/LockSupport.java:194)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(java.base@10.0.1/AbstractQueuedSynchronizer.java:2075)
	at org.gradle.internal.remote.internal.hub.queue.EndPointQueue.take(EndPointQueue.java:48)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionDispatch.run(MessageHub.java:314)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"/0:0:0:0:0:0:0:1:41923 to /0:0:0:0:0:0:0:1:59640 workers Thread 4" #65 prio=5 os_prio=0 tid=0x00007f0eb0054800 nid=0x6922 runnable  [0x00007f0ebf3f3000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.EPollArrayWrapper.epollWait(java.base@10.0.1/Native Method)
	at sun.nio.ch.EPollArrayWrapper.poll(java.base@10.0.1/EPollArrayWrapper.java:265)
	at sun.nio.ch.EPollSelectorImpl.doSelect(java.base@10.0.1/EPollSelectorImpl.java:92)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(java.base@10.0.1/SelectorImpl.java:89)
	- locked <0x00000000da4184e0> (a sun.nio.ch.Util$2)
	- locked <0x00000000da4184d0> (a java.util.Collections$UnmodifiableSet)
	- locked <0x00000000da4183b8> (a sun.nio.ch.EPollSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:100)
	at sun.nio.ch.SelectorImpl.select(java.base@10.0.1/SelectorImpl.java:104)
	at org.gradle.internal.remote.internal.inet.SocketConnection$SocketInputStream.read(SocketConnection.java:178)
	at com.esotericsoftware.kryo.io.Input.fill(Input.java:139)
	at com.esotericsoftware.kryo.io.Input.require(Input.java:159)
	at com.esotericsoftware.kryo.io.Input.readByte(Input.java:255)
	at org.gradle.internal.serialize.kryo.KryoBackedDecoder.readByte(KryoBackedDecoder.java:80)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:63)
	at org.gradle.internal.remote.internal.hub.InterHubMessageSerializer$MessageReader.read(InterHubMessageSerializer.java:52)
	at org.gradle.internal.remote.internal.inet.SocketConnection.receive(SocketConnection.java:79)
	at org.gradle.internal.remote.internal.hub.MessageHub$ConnectionReceive.run(MessageHub.java:263)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(java.base@10.0.1/ThreadPoolExecutor.java:1135)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(java.base@10.0.1/ThreadPoolExecutor.java:635)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(java.base@10.0.1/Thread.java:844)

"Attach Listener" #66 daemon prio=9 os_prio=0 tid=0x00007f0f84001000 nid=0x71a2 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"VM Thread" os_prio=0 tid=0x00007f1014257000 nid=0x6879 runnable  

"GC Thread#0" os_prio=0 tid=0x00007f1014027800 nid=0x685d runnable  

"GC Thread#1" os_prio=0 tid=0x00007f1014029000 nid=0x685e runnable  

"GC Thread#2" os_prio=0 tid=0x00007f101402a800 nid=0x685f runnable  

"GC Thread#3" os_prio=0 tid=0x00007f101402c800 nid=0x6860 runnable  

"GC Thread#4" os_prio=0 tid=0x00007f101402e000 nid=0x6861 runnable  

"GC Thread#5" os_prio=0 tid=0x00007f1014030000 nid=0x6862 runnable  

"GC Thread#6" os_prio=0 tid=0x00007f1014031800 nid=0x6863 runnable  

"GC Thread#7" os_prio=0 tid=0x00007f1014033000 nid=0x6864 runnable  

"GC Thread#8" os_prio=0 tid=0x00007f1014035000 nid=0x6865 runnable  

"GC Thread#9" os_prio=0 tid=0x00007f1014036800 nid=0x6866 runnable  

"G1 Main Marker" os_prio=0 tid=0x00007f101404e800 nid=0x6867 runnable  

"G1 Conc#0" os_prio=0 tid=0x00007f1014050800 nid=0x6868 runnable  

"G1 Conc#1" os_prio=0 tid=0x00007f1014052000 nid=0x6869 runnable  

"G1 Conc#2" os_prio=0 tid=0x00007f1014054000 nid=0x686a runnable  

"G1 Refine#0" os_prio=0 tid=0x00007f10141e5000 nid=0x686d runnable  

"G1 Refine#1" os_prio=0 tid=0x00007f10141e6800 nid=0x686e runnable  

"G1 Refine#2" os_prio=0 tid=0x00007f10141e8800 nid=0x686f runnable  

"G1 Refine#3" os_prio=0 tid=0x00007f10141ea000 nid=0x6871 runnable  

"G1 Refine#4" os_prio=0 tid=0x00007f10141ec000 nid=0x6872 runnable  

"G1 Refine#5" os_prio=0 tid=0x00007f10141ed800 nid=0x6873 runnable  

"G1 Refine#6" os_prio=0 tid=0x00007f10141ef800 nid=0x6874 runnable  

"G1 Refine#7" os_prio=0 tid=0x00007f10141f1000 nid=0x6875 runnable  

"G1 Refine#8" os_prio=0 tid=0x00007f10141f2800 nid=0x6876 runnable  

"G1 Refine#9" os_prio=0 tid=0x00007f10141f4800 nid=0x6877 runnable  

"G1 Young RemSet Sampling" os_prio=0 tid=0x00007f10141f6000 nid=0x6878 runnable  
"VM Periodic Task Thread" os_prio=0 tid=0x00007f101430d800 nid=0x6883 waiting on condition  

JNI global references: 20
 
```


