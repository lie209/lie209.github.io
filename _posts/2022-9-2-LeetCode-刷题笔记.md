---
layout: post
title: LeetCode 刷题笔记
date: 2023-6-13
catalog: true
tags: [Java]
subtitle: 持续更新，持续进步...
---

# `LeetCode` 刷题笔记

这篇笔记主要记录学习`labuladong`算法秘籍的一些心得体会

## 前缀和数组

以第[`303`](https://leetcode.cn/problems/range-sum-query-immutable/)题为例，一开始尝试以`前缀和`的思想减少时间开销，代码如下

```java
public NumArray(int[] nums) 
    {
        this.preNums = new int[nums.length+1];
        this.preNums[0]=nums[0];
        for(int i=1;i<nums.length;i++)
        {
            int tem=0;
            for(int j=0;j<=i;j++)
            {
                tem=tem + nums[j];
            }
            this.preNums[i] = tem;
        }
    }
```

最终运行结果及时间为

```bash
Accepted
15/15 cases passed (52 ms)
Your runtime beats 8.77 % of java submissions
Your memory usage beats 10.82 % of java submissions (44.6 MB)
```

然后发现初始化`前缀和`的时候仍然有优化空间，修改代码如下

```java
public NumArray(int[] nums) 
    {
        this.preNums = new int[nums.length+1];
        this.preNums[0]=nums[0];
        for(int i=1;i<nums.length;i++)
        {
            this.preNums[i] = preNums[i-1]+nums[i];
        }
    }
```

运行结果及时间为

```bash
Accepted
15/15 cases passed (7 ms)
Your runtime beats 100 % of java submissions
Your memory usage beats 44.73 % of java submissions (44.3 MB)
```

优化效果很明显！

## 差分数组

### 差分数组的构造

所谓的差分数组就是构造一个数组，其中第0个元素不定，可以为0或者为原数组第0个元素，剩余元素为原数组对应位置减去前一个位置的差

![image-20220923101642435](https://s2.loli.net/2023/03/10/6UstIOj3nR2MNJh.png)

### 差分数组的使用

- 利用差分数组可以很方便的在一个循环里对一个数组做**多个范围内的加减**操作

  > 或者乘除操作也可以，只需要在构造差分数组的时候变换一下构造方式

  1. 先建立一个差分数组

     ```java
     //给数组i到j的元素加上val
     public int[] increment(int i,int j,int val,int[] diff)
     {   
         diff[i]=diff[i]+val;
         if(j+1<diff.length)
         {
             diff[j+1]=diff[j+1]-val;
         }
         return diff;
     }
     ```

     以上就是利用差分数组对原数组第`i`到`j`的元素加`val`的操作，只需变换差分数组两个位置的元素即可，后面的`-val`原因是，前面`i`处将`i`以及之后的元素都加上了`val`，在`j`之后的元素加上了多余的`val`，就要减掉

  2. 使用差分数组，在一个循环里对原数组进行操作

     ```java
     public int[] getResult(int[] diff,int n)
     {
         //建立一个长度为n的结果数组
         int[] result = new int[n];
         result[0]=diff[0];
         //数据操作
         for(int i=1;i<n;i++)
         {
             result[i]=result[i-1]+diff[i];
         }
         return result;
     }
     ```

- 以力扣[1094题](https://leetcode.cn/problems/car-pooling)为例

  1. 先建立差分数组

     ```java
     public int[] difference(int[][] trips)
     {
         int[] diff=new int[1001];
         for(int i=0;i<trips.length;i++)
         {
             diff[trips[i][1]]=diff[trips[i][1]]+trips[i][0];
             if(trips[i][2]+1<=1000)
             {
                 diff[trips[i][2]]=diff[trips[i][2]]-trips[i][0];
             }   
         }
         return diff;
     } 
     ```

  2. 利用差分数组进行区间的加减法

     ```java
     public int[] incremnet(int[] diff)
     {
         int[] result=new int[1001];
         result[0]=diff[0];
         for(int i=1;i<result.length;i++)
         {
             result[i]=result[i-1]+diff[i];
         }
         return result;
     }
     ```

     这样就在一个循环里，对一个数组进行了多次的加减法操作，大大降低了时间复杂度

## 使用优先级队列进行排序

> 最小堆指的是根节点的值小于所有其他堆节点的值

> 在`Java`中可以使用`PriorityQueue`来实现和使用最小堆

`Java`中的优先级队列基于堆排序实现，具体底层实现可见[此处](https://mp.weixin.qq.com/s/o7tdyLiYm668dpUWd-x7Lg)

建立最小优先级队列：

```java
//建立一个优先级队列，传入优先级队列长度以及比较的接口
PriorityQueue<ListNode> list=new PriorityQueue<>(length,(a,b)->(a.val-b.val));
```

后面的`(a,b)->(a.val-b.val)`是一个`lambada`表达式形式的接口，**注意`lambada`表达式中的相减顺序，这将影响队列优先级**

例如，创建一个返回最大值的优先级队列：

```java
//创建一个从大到小的优先级队列
PriorityQueue<int[]> maxNums2=new PriorityQueue<>((int[] pairs1,int[] pairs2)->
{
    return pairs2[1]-pairs1[1];
});
```



以`Java`中最小优先级队列为例，每次使用`poll()`方法，可以返回队列中的最小元素：

```java
//获得优先级队列中最小的元素
p.next=list.poll();
```

使用`add()`方法往堆中添加元素，元素会自动在堆中排好序，或者也可以使用`offer()`方法来添加

```java
list.add(node);
```

具体使用实例：力扣[23题](https://leetcode.cn/problems/merge-k-sorted-lists/)

## 使用双指针解决某些问题

比如力扣[26题](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)，题目要求删除有序数组中的重复项

![image-20220929160157458](https://s2.loli.net/2023/03/10/F3txT6RZLaphvij.png)

具体思路是：

- 使用两个指针，一个`p`指向前面，一个`q`指向后面，`p`用于指向最终结果的最后一个，`q`用于向后探测
- 因为是**有序数组**，所以只要`q`指向的元素不等于`p`指向的元素，便可以将这个元素放到`p`位置，实现了在一个循环里删除掉了一个数组中的重复元素

力扣[83题](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/)也是一样的操作，实现在**一个循环**里查找和删除重复元素

对于力扣[283题](https://leetcode.cn/problems/move-zeroes/)以及力扣[27题](https://leetcode.cn/problems/remove-element/)，可以使用双指针实现原地删除或移动元素，即不需要使用额外的空间，以`283题`为例，实现原地删除或移动元素的代码模板大致如下：

```java
int p=0,q=0;
//将非零元素移动到前面
while(q<nums.length) 
{
    if(nums[q]!=0)
    {
         nums[p]=nums[q];
         p++;
         q++;
     }
     else
     {
          q++;
     }
}    
```

主要是使用两个指针，一个“守家”，一个“探路”，进而实现要求的操作

## 滑动窗口的使用

一个右指针`right`，一个左指针`left`

一个哈希表`HashMap`

右指针向右滑动，同时维护哈希表中的元素，达到相关条件后，左指针开始收缩，获得想要的结果，以力扣[3题](https://leetcode.cn/problems/longest-substring-without-repeating-characters/submissions/)为例

```java
    public int run(String s)
    {
        if(s.length()==0)
        {
            return 0;
        }
        HashMap<Character,Integer> map=new HashMap<>();
        int left = 0,right = 0;
        int res=Integer.MIN_VALUE;
        while (right <s.length())
        {
            char current=s.charAt(right);
            if(!map.containsKey(current))
            {
                map.put(current, 1);
            }
            else
            {
                map.replace(current, map.get(current)+1);
            }
            //临界条件，出现重复元素，左边开始收缩
            while (map.get(current)>1)
            {
                char currentLeft=s.charAt(left);
                map.replace(currentLeft, map.get(currentLeft)-1 );
                left++;
            }
            res=res<right-left+1?right-left+1:res;
            right++;
        }
        return res;
    }
```

## 二分法

通常使用二分法来解决边界问题，或者查找问题

比如说，要在一个范围里，查找一个满足条件的数，就需要使用二分法，来逐步缩小范围，一个一个试

以力扣[1011题](https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/)为例

设定好了上下界，然后使用二分法寻找满足条件的下界

```java
    public int run(int[] weights,int days)
    {
        int maxWeight=0,sum = 0;
        for (int i = 0; i < weights.length; i++)
        {
            maxWeight=weights[i]>maxWeight?weights[i]:maxWeight;
            sum=sum+weights[i];
        }
        int left = Math.max(maxWeight, sum/days);
        int right = sum;
        int result = right;
        while (left<right)
        {
            int weight=(left+right)/2;
            int day=1;
            int index=0;
            int currentWeight=0;
            while (index < weights.length)
            {
                if(currentWeight+weights[index]<=weight)
                {
                    currentWeight=currentWeight+weights[index];
                    index++;
                }
                else
                {
                    currentWeight=weights[index];
                    day++;
                    index++;
                }
            }
            if(day<=days)
            {
                right=weight;
                result=weight;
            }
            else if (day>days)
            {
                left=left+1;
            }
        }
        return result;
    }
```

## 单调栈

单调栈一般用来解决求**下一个最大**，或者**下一个最小值**的问题。

以求**下一个最大值**为例，一般步骤为：

- 判断栈顶元素和当前元素的大小
  - 若 当前元素＜栈顶元素：当前元素入栈
  - 若 当前元素＞栈顶元素：弹出栈顶元素，并记录当前元素作为栈顶元素的下一个更大元素

例如力扣[503题](https://leetcode.cn/problems/next-greater-element-ii/)：

与传统的单调栈不同的是，加入了循环数组，对于循环数组问题，可以简单地将数组延伸为两倍即可，代码实现如下

```java
	//具体实现时，可以使用取余而不是创建一个两倍长度数组，来实现循环数组
    public int[] run(int[] nums)
    {
        int n=nums.length;
        int[] results=new int[nums.length];
        //默认值设置为-1
        Arrays.fill(results, -1);
        Stack<Integer> stack=new Stack<>();
        stack.push(0);
        for (int i = 1; i < n*2; i++)
        {
            int num=nums[i%n];
            //判断当前元素与栈顶元素的大小关系
            if(num>nums[stack.peek()%n])
            {
                while (!stack.isEmpty()&&num>nums[stack.peek()])
                {
                    results[stack.peek()%n]=num;
                    stack.pop();
                }
                stack.push(i%n);
            }
            else
            {
                stack.push(i%n);
            }

        }
        return results;
    }
```

其次，单调栈还可用用来维护一个单调递增或者递减的序列，如力扣[316题](https://leetcode.cn/problems/remove-duplicate-letters/)：

![image-20230602172855405](https://s2.loli.net/2023/06/02/pVJsnYgGDQk5wAz.png)

小字母越靠前字典序越小

- 单调递增栈stack维护递增的最小字典序 + 哈希表hash维护所有字母最后出现的位置
- 如果按普通单调栈去重，无法一定得到最小的字典序排列，比如abacb，在单调栈处理时会将第一个b直接弹出，使得最终结果为acb，但是最小字典序排列应该是abc，问题在于当我们前两位的a、b已符合递增入栈，那么后续遇到第二个a就不应该再次操作栈，因为栈中相同的a一定处于第二个a前方，按照最小字典序排布规律，较小的应该尽量放前方，**所以遇到栈中存在的字母应当跳过**

- 当第一次遇到某个小于栈顶且栈中不存在的字母 i 时，如何判断是否弹出栈顶将当前i放入？按最小字典序（单调递增栈）原则会选择弹出，但是这里要注意res的完整性，不能缺少任何字母，所以在遇到这种情况时需要判断i后方是否还有栈顶字母，如果i后方还有栈顶字母，那么就可以弹出栈顶将i入栈；如果i后方没有栈顶字母了，那么为了保证完整性只能选择局部破坏单调性，跳过弹出并将小于原栈顶 i 压入

```java
    public String run(String s)
    {
        int len=s.length();
        //记录该字母最后出现的位置
        HashMap<Character,Integer> hashMap=new HashMap<>();
        char[] chars=s.toCharArray();

        for (int i = 0; i < len; i++)
        {
            if(hashMap.containsKey(chars[i]))
            {
                hashMap.replace(chars[i],i);
            }
            else
            {
                hashMap.put(chars[i],i);
            }
        }
        Stack<Character> stack=new Stack<>();
        stack.push(chars[0]);
        for (int i = 1; i < len; i++)
        {
            if(stack.contains(chars[i]))
            {
                continue;
            }
            while (!stack.isEmpty()&&chars[i]<stack.peek()&&hashMap.get(stack.peek())>i)
            {
                stack.pop();
            }
            stack.push(chars[i]);
        }
        char[] result=new char[stack.size()];
        int i=stack.size()-1;
        while (!stack.isEmpty())
        {
            result[i]=stack.pop();
            i--;
        }
        return String.valueOf(result);
    }
```

## 单调队列

单调队列用来解决维护一个窗口，可以获得窗口中的最大值或者最小值

> 在这种情况下，使用优先级队列会很耗费时间，并且在某些场景下不满足条件

单调队列就像一个互联网公司：

当你刚刚毕业进入公司，可恶的资本家就会裁掉那些不仅年纪比你大，能力还不如你的程序员（删掉队尾）； 当你努力工作终于在40岁变为全公司最强的程序员时，公司招进一个20岁但能力远不如你的毕业生，并因为你年龄太大而把你裁掉（删掉队头）

![image-20230530161952547](https://s2.loli.net/2023/05/30/CceWgwdQNK8ly6S.png)

具体实现如下，`Java`使用`Deque`来实现

```java
class MonotonicQueue
    {
        //队列最大值
        Deque<Integer> deque=new LinkedList<>();
        int front;
        void pop(int a)
        {
            if(!deque.isEmpty()&&deque.getFirst()==a)
            {
                deque.pollFirst();

            }
            front=deque.getFirst();

        }
        void push(int a)
        {
            while (!deque.isEmpty()&&deque.getLast()<a)
            {
                deque.pollLast();
            }
            deque.offerLast(a);
            front=deque.getFirst();
        }
        int getMax()
        {
            return front;
        }
        MonotonicQueue()
        {
        }
    }
```

值得注意的是，对于一个`Deque`，具体为`[1, 2]`时，`last`元素是2，`first`元素是1。

具体应用，例如力扣[239题](https://leetcode.cn/problems/sliding-window-maximum/)：

```java
    public int[] run(int[] nums,int k)
    {
        if(k==1)
        {
            return nums;
        }
        int[] results=new int[nums.length-k+1];
        MonotonicQueue monotonicQueue=new MonotonicQueue();
        //初始化第一个队列
        for (int i = 0; i < k; i++)
        {
            monotonicQueue.push(nums[i]);
        }
        results[0]=monotonicQueue.getMax();
        for (int i = 1; i <= nums.length-k; i++)
        {
            monotonicQueue.pop(nums[i-1]);
            monotonicQueue.push(nums[i+k-1]);
            results[i]=monotonicQueue.getMax();
        }
        return results;
    }
```

## 数据结构设计

### LRU算法

日常很多时候，有些题目不能使用已有的数据结构来解决，这种情况下就需要自己来设计适合这种题目的数据结构，例如力扣[146题](https://leetcode.cn/problems/lru-cache/)：

根据题意，要求`put`以及`get`操作能够在`O(1)`的时间复杂度下完成，这样就最好不要采用暴力破解法来解题了

根据题意：

- 要求能够很快根据`key`来访问到`value`值，这种情况下我们就可以想到使用`哈希表`来实现
- 缓存队列能够很快的进行更新操作，这样我们就会想到链表
- 题目还要求根据优先级进行链表的更新，为了满足时间复杂度的要求，我们就可以想到使用双向链表来实现

![数据结构图](https://s2.loli.net/2023/06/06/OzJTQ5adfPugx7o.png)

还需要注意的一点是，链表中的节点优先级需要在以下几个情况下进行更新：

- 节点被访问过
- 节点被修改过

根据节点优先级，在容量满了之后，剔除优先级最低的节点，**为了方便对`哈希表`中的元素进行`remove`操作，我们还需要在链表中存储`key`**，如上图所示

![image-20230606160826211](https://s2.loli.net/2023/06/06/ZH16bXkBmuTSCeV.png)

最终我的代码实现如下：

```java
class LRUCache
    {
        class DoublyListNode
        {
            int key;
            int val;
            DoublyListNode pre;
            DoublyListNode next;

            public DoublyListNode(int key,int val, DoublyListNode pre, DoublyListNode next) {
                this.key=key;
                this.val = val;
                this.pre = pre;
                this.next = next;
            }
        }
        HashMap<Integer, DoublyListNode> hashMap;
        DoublyListNode head;
        int length;
        int maxLength;
        public LRUCache(int capacity)
        {
            this.hashMap=new HashMap<>();
            maxLength=capacity;
            head=new DoublyListNode(-1,-1,null,null);

            length=0;
        }

        public int get(int key)
        {
            if(hashMap.containsKey(key))
            {
                DoublyListNode node=hashMap.get(key);
                if(node.next!=null&&node.pre!=null)
                {
                    node.next.pre= node.pre;
                    node.pre.next=node.next;
                    node.next=head.next;
                    head.next.pre=node;
                    head.next=node;
                    node.pre=head;

                    return node.val;
                }
                if(node.next==null&&node.pre!=null)
                {
                    if(node.pre==head)
                    {
                        return node.val;
                    }
                    node.pre.next=null;

                    node.next=head.next;
                    head.next.pre=node;
                    head.next=node;
                    node.pre=head;
                    return node.val;
                }
                if(node.pre==head)
                {
                    return node.val;
                }


            }
            return -1;
        }

        public void put(int key, int value)
        {
            if(hashMap.containsKey(key))
            {
                hashMap.get(key).val=value;
                DoublyListNode node=hashMap.get(key);
                if(node.next!=null&&node.pre!=null)
                {
                    node.next.pre= node.pre;
                    node.pre.next=node.next;
                    node.next=head.next;
                    head.next.pre=node;
                    head.next=node;
                    node.pre=head;
                }
                if(node.next==null&&node.pre!=null)
                {
                    if(node.pre!=head)
                    {
                        node.pre.next=null;

                        node.next=head.next;
                        head.next.pre=node;
                        head.next=node;
                        node.pre=head;
                    }
                }
            }
            else
            {
                DoublyListNode node=new DoublyListNode(key,value,null, null);
                if(length<maxLength)
                {
                    if(length==0)
                    {
                        head.next=node;
                        node.next=null;
                        node.pre=head;
                        length++;
                    }
                    else
                    {
                        node.next=head.next;
                        head.next.pre=node;
                        head.next=node;
                        node.pre=head;
                        length++;
                    }
                    hashMap.put(key, node);
                }
                else
                {
                    //移除最后一个节点
                    DoublyListNode p=head;
                    while (p.next.next!=null)
                    {
                        p=p.next;
                    }
                    hashMap.remove(p.next.key);
                    p.next=null;
                    if(head.next==null)
                    {
                        head.next=node;
                        node.pre=head;
                        hashMap.put(key, node);
                    }
                    else
                    {
                        node.next=head.next;
                        head.next.pre=node;
                        head.next=node;
                        node.pre=head;
                        hashMap.put(key, node);
                    }
                }
            }
        }
    }
```

当然了，我们还可以不手搓链表，使用`Java`自带的`LinkedHashMap`来实现`LRU`算法

```java
class LRUCache 
{
    int cap;
    LinkedHashMap<Integer, Integer> cache = new LinkedHashMap<>();
    public LRUCache(int capacity)
    {
        this.cap = capacity;
    }

    public int get(int key) 
    {
        if (!cache.containsKey(key)) 
        {
            return -1;
        }
        // 将 key 变为最近使⽤
        makeRecently(key);
        return cache.get(key);
    }

    public void put(int key, int val) 
    {
        if (cache.containsKey(key)) 
        {
            // 修改 key 的值
            cache.put(key, val);
            // 将 key 变为最近使⽤
            makeRecently(key);
            return;
        }

        if (cache.size() >= this.cap) 
        {
            // 链表头部就是最久未使⽤的 key
            int oldestKey = cache.keySet().iterator().next();
            cache.remove(oldestKey);
        }
        // 将新的 key 添加链表尾部
        cache.put(key, val);
    }

    private void makeRecently(int key) 
    {
        int val = cache.get(key);
        // 删除 key，重新插⼊到队尾
        cache.remove(key);
        cache.put(key, val);
    }
}
```

### LFU缓存

基于力扣[460题](https://leetcode.cn/problems/lfu-cache/)，这个同样是一个数据结构设计问题

根据题意，需要满足以下几点：

- 需要以`O(1)`的时间复杂度运行，这就想到了哈希表

  ```java
  LinkedHashMap<Integer,Integer> caches=new LinkedHashMap<>();
  ```

- 需要记录每个元素的访问时间，这也想到了哈希表

  ```java
  HashMap<Integer,Integer> keysCounts=new HashMap<>();
  ```

- 第三个，也是最关键的一个，需要获得最久未使用的键，可能有多个键的访问次数是一样的，而又需要获得最早的那个

  而且还需要在`O(1)`的时间复杂度下

  这就可以想到用`LinkedHashSet`，与`LinkedHashMap`类似，`LinkedHashSet`是有序的，**最后放入的元素在集合末尾，第一个元素在最前面**，因此，我们可以利用这个特性，获得同一访问次数下最久未被访问的键（因为每个键被访问的时候，访问次数都会加一）

  ```java
  HashMap<Integer, LinkedHashSet<Integer>> count2keys =new HashMap<>();
  ```

最终代码如下：

```java
class LFUCache {

        LinkedHashMap<Integer,Integer> caches=new LinkedHashMap<>();
        HashMap<Integer,Integer> keysCounts=new HashMap<>();
        HashMap<Integer, LinkedHashSet<Integer>> count2keys =new HashMap<>();
        int capacity;
        public LFUCache(int capacity) {
            this.capacity=capacity;
        }

        public int get(int key)
        {
            if(caches.containsKey(key))
            {
                int count=keysCounts.get(key);
                keysCounts.replace(key,count+1);
                count2keys.get(count).remove(key);
                if(count2keys.containsKey(count+1))
                {
                    count2keys.get(count+1).add(key);
                }
                else
                {
                    count2keys.put(count+1, new LinkedHashSet<>());
                    count2keys.get(count+1).add(key);
                }
                return caches.get(key);
            }
            else
            {
                return -1;
            }

        }

        public void put(int key, int value)
        {
            if(caches.containsKey(key))
            {
                caches.replace(key,value);
                //访问次数加一
                int count=keysCounts.get(key);
                keysCounts.replace(key, count+1);
                count2keys.get(count).remove(key);
                if(count2keys.containsKey(count+1))
                {
                    count2keys.get(count+1).add(key);
                }
                else
                {
                    count2keys.put(count+1, new LinkedHashSet<>());
                    count2keys.get(count+1).add(key);
                }
            }
            else
            {
                //没有满
                if(caches.size()<this.capacity)
                {
                    caches.put(key, value);
                    keysCounts.put(key, 1);
                    if(count2keys.containsKey(1))
                    {
                        count2keys.get(1).add(key);
                    }
                    else
                    {
                        count2keys.put(1, new LinkedHashSet<>());
                        count2keys.get(1).add(key);
                    }
                }
                //满了
                else
                {
                    //最小的访问次数
                    int min=0;
                    for(int i:count2keys.keySet())
                    {
                        if(count2keys.get(i).size()!=0)
                        {
                            min=i;
                            break;
                        }
                    }
                    int minKey=count2keys.get(min).iterator().next();
                    caches.remove(minKey);
                    count2keys.get(min).remove(minKey);
                    keysCounts.remove(minKey);
                    caches.put(key, value);
                    keysCounts.put(key, 1);
                    count2keys.get(1).add(key);
                }
            }
        }
    }
```

### 使用HashMap去重

力扣[380题](https://leetcode.cn/problems/insert-delete-getrandom-o1/)

看到题目可以有以下想法：

- 需要是一个集合，不能有重复元素，这就可以想到使用`HashMap`进行去重，因为`HashMap`的键是不能重复的
- 需要访问随机索引的值，那就可以想到用`List`
- 需要删除元素，那么我们就可以**将要删除的元素的值与最后一个值进行交换，删除最后一个元素**就行了，这样也可以避免移动元素增加时间开销

综上，代码如下：

`HashMap`存储元素与其在`list`中索引的映射，（ value --> index ）`length`存储最后一个元素的索引位置

```java
class RandomizedSet {
        HashMap<Integer,Integer> hashMap;
        List<Integer> list;
        int length;

        public RandomizedSet() {
            this.hashMap=new HashMap<>();
            this.list=new ArrayList<>();
            this.length=0;
        }

        public boolean insert(int val)
        {
            if(!hashMap.containsKey(val))
            {
                list.add(val);
                hashMap.put(val, length);
                length++;
                return true;
            }
            return false;
        }

        public boolean remove(int val)
        {
            if(hashMap.containsKey(val))
            {
                //目标值
                int index=hashMap.get(val);
                list.set(index,list.get(length-1));
                hashMap.replace(list.get(index), index);
                list.remove(length-1);
                length--;
                hashMap.remove(val);
                return true;
            }
            return false;
        }

        public int getRandom()
        {
            int index=new Random().nextInt(length);
            return this.list.get(index);
        }
    }
```

