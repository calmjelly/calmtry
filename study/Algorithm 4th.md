# 基础数据结构

## 背包

背包是一种不支持从中删除元素的集合数据类型。

## 队列

先进先出（FIFO），例如二叉树的广度优先遍历可以用队列实现。

## 栈

后进先出（LIFO），例如二叉树的深度优先遍历可以用栈实现。

### 经典应用：

leetcode.150 逆波兰表达式求值（后缀表示法）

![image-20200614225003828](../img/image-20200614225003828.png)

解答：

```java
        public int evalRPN(String[] tokens) {
            int num1, num2;
            Stack<Integer> stack = new Stack<>();
            for (int i = 0; i < tokens.length; i++) {
                switch (tokens[i]) {
                    case "+":
                        num2 = stack.pop();
                        num1 = stack.pop();
                        stack.push(num1 + num2);
                        break;
                    case "-":
                        num2 = stack.pop();
                        num1 = stack.pop();
                        stack.push(num1 - num2);
                        break;
                    case "*":
                        num2 = stack.pop();
                        num1 = stack.pop();
                        stack.push(num1 * num2);
                        break;
                    case "/":
                        num2 = stack.pop();
                        num1 = stack.pop();
                        stack.push(num1 / num2);
                        break;
                    default:
                        stack.push(Integer.parseInt(tokens[i]));
                }
            }
            return stack.pop();
        }
```

## 链表

一个节点由该节点存储的数据和指向下一个节点的引用组成。

```java
public class Node{
    Item item;
    Node next;
}
```

典型操作：

1、插入、删除节点

在头部插入节点，只需要新建一个节点，将节点的next指向当前链表的头结点，然后将头结点的引用改为新节点即可。

```java
newNode = new Node();//新建一个节点
newNode.next = head;//next指向当前链表的头结点
head = newNode;//更新头结点为新建的节点

```

在链表中间插入节点，需要先将待插入节点的next指向待插入位置后面的节点，然后将前面节点的next指向自己即可。

```java
newNode = new Node();
newNode.next = pre.next;
pre.next = newNode;
```

在链表中间删除节点，直接让节点的next指向next的next即可。

```
node.next = node.next.next;
```

2、链表的遍历

链表遍历用节点是否为null来判断，每次循环更新next即可。

```java
while(node!=null){
    //do something
    node=node.next;
}

for(Node x = first;x!=null;x=x.next){
    //do something
}
```

3、链表模拟栈

4、链表模拟队列

5、环形链表

6、反转链表