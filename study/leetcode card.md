# 栈和队列

## 设计循环队列

rear指向队尾元素的下一个位置，front指向队首元素。

```java
class MyCircularQueue {
        private int front;
        private int rear;
        private int capaticy;
        private int[] arr;

    /** Initialize your data structure here. Set the size of the queue to be k. */
    public MyCircularQueue(int k) {
        capaticy = k+1;
        arr = new int[capaticy];
        front=0;
        rear=0;
    }
    
    /** Insert an element into the circular queue. Return true if the operation is successful. */
    public boolean enQueue(int value) {
        if(isFull()){
            return false;
        }
        arr[rear] = value;
        rear = (rear+1)%capaticy;
        return true;

    }
    
    /** Delete an element from the circular queue. Return true if the operation is successful. */
    public boolean deQueue() {
        if(isEmpty()){
            return false;
        }
        front = (front+1)%capaticy;
        return true;

    }
    
    /** Get the front item from the queue. */
    public int Front() {
        if(isEmpty()){
            return -1;
        }
        return arr[front];

    }
    
    /** Get the last item from the queue. */
    public int Rear() {
        if(isEmpty()){
            return -1;
        }
        return arr[(rear-1+capaticy)%capaticy];

    }
    
    /** Checks whether the circular queue is empty or not. */
    public boolean isEmpty() {
        return front==rear;

    }
    
    /** Checks whether the circular queue is full or not. */
    public boolean isFull() {
        return (rear+1)%capaticy == front;

    }
}

/**
 * Your MyCircularQueue object will be instantiated and called as such:
 * MyCircularQueue obj = new MyCircularQueue(k);
 * boolean param_1 = obj.enQueue(value);
 * boolean param_2 = obj.deQueue();
 * int param_3 = obj.Front();
 * int param_4 = obj.Rear();
 * boolean param_5 = obj.isEmpty();
 * boolean param_6 = obj.isFull();
 */
```

## 数据流中的移动平均值

## 题目描述:

给定一个整数数据流和一个窗口大小，根据该滑动窗口的大小，计算其所有整数的移动平均值。

## 示例:

```text
MovingAverage m = new MovingAverage(3);
m.next(1) = 1
m.next(10) = (1 + 10) / 2
m.next(3) = (1 + 10 + 3) / 3
m.next(5) = (10 + 3 + 5) / 3
```

使用一个队列来保存数据，并且让队列的长度最大不超过初始化时候传入的值。

```java
public class MovingAverage {
    private int sum;
    private int size;
    private Queue<Integer> queue;

    public MovingAverage(int size) {
        //用于控制队列中元素的数量，最大不超过窗口大小
        this.size =size ;
        sum = 0;
        queue = new ArrayDeque<>(size);
    }

    public double next(int num) {
        //当队列中元素达到窗口大小时候，先出队，再入队，保持队列大小不变
        if (queue.size() == size) {
            sum -= queue.remove();
        }
        queue.add(num);
        sum += num;
        return sum * 1.0 / queue.size();
    }

}
```

