# leetcode 最热100题

刷题是不可能刷题的，这辈子都不可能刷题的，看也看不懂，学也学不会，只能靠着题解区大佬的解答才能维持得了生活的样子，里面个个都是人才，代码写的秀，讲解说的透，我超喜欢这里的！

## 739.每日温度

![image-20210311232947223](../img/image-20210311232947223.png)

题意：找数组里面每个数字后面第一个比他大的数字距离他的距离，没有则置为0.

解法：单调栈。找数组中距离某个值左侧/右侧第一个比他大/小的值之间的距离这种，可考虑单调栈。类似lc84题。

维护一个单调递减的栈，栈中存放元素的下标：

1、栈为空，直接入栈；

2、栈不为空，比较当前值与栈顶下标对应值的大小：

若小于栈顶值则入栈当前元素的下标。

大于栈顶值，则栈顶元素出栈，下标之差即为第一个比栈顶元素大的值距离栈顶元素的距离。

java：

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        //单调栈
        Deque<Integer> stack = new LinkedList<>();
        int[] res = new int[T.length];
        //遍历数组
        for(int i=0;i<T.length;i++){
            //拿到当前元素
            int currentTemperature = T[i];
            //栈不为空且栈顶下标指向值小于当前值
            while(stack.size()!=0 && T[stack.peek()]<currentTemperature){
                //出栈并计算栈顶元素的距离差
                int temp = stack.pop();
                res[temp] = i - temp;
            }
            //其他情况则直接入栈
            stack.push(i);
        }
        return res;
    }
}
```

## 647.回文子串

![image-20210312233516206](../img/image-20210312233516206.png)

题意：计算一个子串中有多少回文子串。

解法：中心扩展法，和lc第5题 求解最长回文子串类似。

解1：串长分为奇数和偶数两种情况，中心点是不一样的，可能为1个，也可能是2个，所以直接计算两次。

```java
class Solution {
    int result = 0;

    public int countSubstrings(String s) {
        //遍历字符串，分别计算奇数、偶数两种情况，
        for (int i = 0; i < s.length(); i++) {
            //奇数中心点只有一个，也即left、right初始位置重合
            help(i, i, s);
            //偶数中心点有两个，left和right相邻
            help(i, i + 1, s);
        }
        return result;
    }

    public void help(int left, int right, String s) {
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            result++;
            left--;
            right++;
        }
    }
}
```

解2：之所以要分为奇数和偶数，就是因为串长不同，会导致左右指针的初始位置不同，如果考虑把字母之间的间隙考虑进去，如果中心点是字母之间的空格，看一下左右指针的初始位置。中心点位置为center，左指针为left，右指针为right，则

| center | left | right |
| ------ | ---- | ----- |
| 0      | 0    | 0     |
| 1      | 0    | 1     |
| 2      | 1    | 1     |
| 3      | 1    | 2     |
| 4      | 2    | 2     |
| 5      | 2    | 3     |
| 6      | 3    | 3     |

带上字母之间的间隙，center一共有2n-1个遍历位置。

很直观的结论  ： left = center/2，right = center-center/2;

Java:

```java
class Solution {
    public int countSubstrings(String s) {
        int n = s.length();
        int left = 0, right = 0, result = 0;
        //center从0~2n-1遍历
        for (int center = 0; center <= 2 * n - 1; center++) {
            //初始化left 和 right指针
            left = center / 2;
            right = center - left;
            //判断是否已到边界&&是否是回文串,更新结果
            while (left >= 0 && right < n && s.charAt(left) == s.charAt(right)) {
                result++;
                left--;
                right++;
            }
        }
        return result;
    }
}
```

