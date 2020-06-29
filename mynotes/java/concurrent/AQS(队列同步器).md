# AQS(队列同步器)

全称AbstracQueueSynchronizer，是一个用来构建锁和同步器的框架。JUC包中基本所有的并发工具都使用到AQS。

## 1. 简介

AQS ，AbstractQueuedSynchronizer ，即**队列同步器**。它是构建锁或者其他同步组件的基础框架（如 ReentrantLock、ReentrantReadWriteLock、Semaphore 等）

## 2. 优势

AQS 解决了在实现同步器时涉及当的大量细节问题，例如获取同步状态、FIFO 同步队列。基于 AQS 来构建同步器可以带来**很多好处**。它不仅能够极大地减少实现工作，而且也不必处理在多个位置上发生的竞争问题。

在基于 AQS 构建的同步器中，只能在一个时刻发生阻塞，从而降低上下文切换的开销，提高了吞吐量。同时在设计 AQS 时充分考虑了可伸缩性，因此 J.U.C 中，所有基于 AQS 构建的同步器均可以获得这个优势。

## 3. 同步状态

AQS 的主要使用方式是**继承**，子类通过继承同步器，并实现它的**抽象方法**来管理同步状态。

AQS 使用一个 `int` 类型的成员变量 `state` 来**表示同步状态**：

- 当 `state > 0` 时，表示已经获取了锁。
- 当 `state = 0` 时，表示释放了锁。

它提供了三个方法，来对同步状态 `state` 进行操作，并且 AQS 可以确保对 `state` 的操作是**安全**的：

- `#getState()`
- `#setState(int newState)`
- `#compareAndSetState(int expect, int update)`

## 4. 同步队列

AQS 通过内置的 FIFO 同步队列来完成资源获取线程的**排队工作**：

- 如果当前线程获取同步状态失败（锁）时，AQS 则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列，同时会阻塞当前线程
- 当同步状态**释放**时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。

## 5. 主要内置方法

AQS 主要提供了如下**方法**：

- `#getState()`：返回同步状态的当前值。
- `#setState(int newState)`：设置当前同步状态。
- `#compareAndSetState(int expect, int update)`：使用 CAS 设置当前状态，该方法能够保证状态设置的原子性。
- 【可重写】`#tryAcquire(int arg)`：独占式获取同步状态，获取同步状态成功后，其他线程需要等待该线程释放同步状态才能获取同步状态。
- 【可重写】`#tryRelease(int arg)`：独占式释放同步状态。
- 【可重写】`#tryAcquireShared(int arg)`：共享式获取同步状态，返回值大于等于 0 ，则表示获取成功；否则，获取失败。
- 【可重写】`#tryReleaseShared(int arg)`：共享式释放同步状态。
- 【可重写】`#isHeldExclusively()`：当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占。
- `acquire(int arg)`：独占式获取同步状态。如果当前线程获取同步状态成功，则由该方法返回；否则，将会进入同步队列等待。该方法将会调用**可重写**的 `#tryAcquire(int arg)` 方法；
- `#acquireInterruptibly(int arg)`：与 `#acquire(int arg)` 相同，但是该方法响应中断。当前线程为获取到同步状态而进入到同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException 异常并返回。
- `#tryAcquireNanos(int arg, long nanos)`：超时获取同步状态。如果当前线程在 nanos 时间内没有获取到同步状态，那么将会返回 false ，已经获取则返回 true 。
- `#acquireShared(int arg)`：共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式的主要区别是在同一时刻可以有多个线程获取到同步状态；
- `#acquireSharedInterruptibly(int arg)`：共享式获取同步状态，响应中断。
- `#tryAcquireSharedNanos(int arg, long nanosTimeout)`：共享式获取同步状态，增加超时限制。
- `#release(int arg)`：独占式释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒。
- `#releaseShared(int arg)`：共享式释放同步状态。

从上面的方法看下来，基本上可以分成 **3** 类：

- **独占式**获取与释放同步状态
- **共享式**获取与释放同步状态
- 查询**同步队列**中的等待线程情况