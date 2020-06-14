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

