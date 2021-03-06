## 并发

### 线程安全

java线程安全的实现手段包括

+ 互斥同步(mutual exclusion & synchronization)

  是指保证一个共享变量在一个时刻只能被一天线程使用。实现互斥同步分方式包括：

  + 临界区(critical section)
  + 互斥量(mutex)
  + 信号量(semaphore)

  synchronized关键字，juc包里面的Lock都能实现互斥同步。

+ 非阻塞同步(non-blocking synchronization)

  互斥同步是一种悲观的策略，而非阻塞同步则是一种乐观的策略。简单的说，这个策略就是不管冲突的风险，先进行操作，如果没有别的线程竞争，那就直接成功，如果产生了冲突，就采取别的补偿措施。CAS是其中的一种手段。但是CAS无法解决"ABA问题"。

+ 无同步方案

  在程序设计的是，避免出现贡献变量的情况，这样就不需要进行同步。

### 锁优化

jdk5升级到jdk6以后，有了很多锁优化

+ 适应性自旋(adaptive spinning)

  一般情况来说，一个线程获取了锁以后，很快会释放，如果另外的线程每次获取失败都挂起的话，会浪费很多资源，自旋锁就是为了提高这样场景的性能。使用自旋锁的时候，线程获取锁失败不会马上挂起，而会执行一个忙循环(自旋)，到失败一定次数以后，还是会照样挂起。这样就能避免线程切换的开销，但是它会占用处理器。

  适应性自旋意味着，自旋的时间和次数不是固定的，会根据同一个锁上的自旋时间以及锁的拥有者的状态来动态决定。

  所谓自适应就意味着自旋的次数**不再是固定**的，**它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定**。它怎么做呢？

  - 线程如果自旋成功了，那么下次自旋的次数会更加多，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。
  - 反之，如果对于某个锁，很少有自旋能够成功的，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源。

  有了自适应自旋锁，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测会越来越准确，虚拟机会变得越来越聪明。

+ 锁消除(lock elimimation)

  锁消除是指在运行过程中，对一些在代码层面做了同步限制，但是运行时检测到时不会出现共享数据的锁进行消除。这个判断的依据是逃逸分析的结果，如果检测到一个堆对象是不会逃逸到别的线程，那么这个对象可以当做栈上数据对待。

  ~~~java
  public void vectorTest(){
      Vector<String> vector = new Vector<String>();
      for (int i = 0 ; i < 10 ; i++){
      	vector.add(i + "");
      }
      System.out.println(vector);
  }
  ~~~

  在运行这段代码时，JVM 可以明显检测到变量 `vector` 没有逃逸出方法 `#vectorTest()` 之外，所以 JVM 可以大胆地将 `vector` 内部的加锁操作消除。

+ 锁膨胀(lock coarsening)

+ 锁粗化

**由来**

  我们知道在使用同步锁的时候，需要让同步块的作用范围尽可能小：仅在共享数据的实际作用域中才进行同步。这样做的目的，是为了使需要同步的操作数量尽可能缩小，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。  在大多数的情况下，上述观点是正确的，LZ 也一直坚持着这个观点。但是如果一系列的连续加锁解锁操作，可能会导致不必要的性能损耗，所以引入**锁粗化**的概念。

  **定义**

  锁粗话概念比较好理解，就是将**多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁**。

  如上面实例：`vector` 每次 add 的时候都需要加锁操作，JVM 检测到对同一个对象（`vector`）连续加锁、解锁操作，会合并一个更大范围的加锁、解锁操作，即加锁解锁操作会移到 `for` 循环之外。

+ 轻量级锁(lightweight locking)

  轻量级锁和下面的偏向锁的实现，都是基于对象在内存布局中的对象头对象中的Mark word。Mark word会存放对象的哈希码、GC分代年龄、锁状态等。

  当线程进入到同步的代码块中，如果发现同步对象的锁没被锁定，就会在当前线程的栈帧中创建锁记录(lock record)来存放锁对象mark world的拷贝。然后虚拟机会使用CAS尝试把锁对象的mark word设置为指向lock record的指针并且修改mark word锁标记位。更新成功的话，那这个线程就获得了轻量级锁。

  如果设置失败，这就意味着有另外的线程也在争抢这个锁。当有两个线程同时争抢一个锁的时候，就回膨胀成为重量级锁(互斥量)。膨胀为重量级锁以后，会让线程释放资源，提高cpu的利用率。

  由此可见，轻量级适用于低冲突的情况，如果高冲突的情况下性能会比重量级锁要差。

+ 偏向锁(biased locking)

  偏向锁适用在无竞争的场景下。

  偏向锁会偏向于第一个获得它的线程。对于一个锁对象，第一次被一条线程获取到的时候，这个锁对象会进入到偏向锁状态，同时也通过CAS把这条线程的ID记录在锁对象的mark word中。往后，这线程进入到这个锁相关的同步块的时候，都不需要进行任何同步操作。当另外一条线程尝试去获取这个锁的时候，这个锁机会撤销偏向状态。

