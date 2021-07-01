[toc]

## 54 | 算法实战（三）：剖析高性能队列Disruptor背后的数据结构和算法

1. Disruptor 简介

	> Disruptor 是线程之间用于消息传递的队列
	>
	> 它的性能表现非常优秀，可以算得上是最快的内存消息队列了

2. 开篇题

	- Disruptor 是如何做到如此高性能的？其底层依赖了哪些数据结构和算法？

### 基于**循环队列**的 “生产者 - 消费者模型”

1. 队列的两种实现思路

	1. 基于**链表**，案例　“无界队列”
	2. 基于**数组**，案例　“有界队列”

2. **循环队列**，一种特殊的顺序队列。

3. 代码实现，“当队列满了之后，生产者就轮训等待；当队列空了之后，消费者就轮训等待”

	- ```java
		
		public class Queue {
		  private Long[] data;
		  private int size = 0, head = 0, tail = 0;
		  public Queue(int size) {
		    this.data = new Long[size];
		    this.size = size;
		  }
		
		  public boolean add(Long element) {
		    if ((tail + 1) % size == head) return false;
		    data[tail] = element;
		    tail = (tail + 1) % size;
		    return true;
		  }
		
		  public Long poll() {
		    if (head == tail) return null;
		    long ret = data[head];
		    head = (head + 1) % size;
		    return ret;
		  }
		}
		
		public class Producer {
		  private Queue queue;
		  public Producer(Queue queue) {
		    this.queue = queue;
		  }
		
		  public void produce(Long data) throws InterruptedException {
		    while (!queue.add(data)) {
		      Thread.sleep(100);
		    }
		  }
		}
		
		public class Consumer {
		  private Queue queue;
		  public Consumer(Queue queue) {
		    this.queue = queue;
		  }
		
		  public void comsume() throws InterruptedException {
		    while (true) {
		      Long data = queue.poll();
		      if (data == null) {
		        Thread.sleep(100);
		      } else {
		        // TODO:...消费数据的业务逻辑...
		      }
		    }
		  }
		}
		```

### 基于**加锁**的并发“生产者 - 消费者模型”

1. 刚刚的“生产者 - 消费者模型”实现代码，会存在多线程**并发问题**。
	- ![img](imgs/4f88bec40128dbc8c1b700b4cf38b63d.jpg)
2. 如何解决这种线程并发问题呢？
	- 加锁
3. 加锁带来的影响
	- 效率下降

### 基于**无锁**的并发“生产者 - 消费者模型”

1. Disruptor 基本思想（“两阶段写入”）

	1. **生产者**，它往队列中添加数据之前，**先申请**可用空闲存储单元，并且是批量地申请连续的 n 个（n≥1）存储单元。
	2. 当申请到这组连续的存储单元之后，后续往队列中**添加**元素，就可以**不用加锁**了，因为这组存储单元是这个**线程独享**的。
	3. 不过，申请存储单元**的过程是**需要加锁**的。
	4. **消费者**，处理的过程跟生产者是类似的。

2. Disruptor 实现思路的一个弊端

	> 如果生产者 A 申请到了一组连续的存储单元，假设是下标为 3 到 6 的存储单元
	>
	> 生产者 B 紧跟着申请到了下标是 7 到 9 的存储单元
	>
	> 在 3 到 6 没有完全**写入**数据之前，7 到 9 的数据是无法**读取**的。