---
layout: post
title: LeetCode 刷题笔记
date: 2023-8-4
catalog: true
tags: [Java]
---


# LeetCode 刷题笔记

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

> 在`Java`中可以使用`PriorityQueue`来实现和使用最小堆，默认堆顶是最小值

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

### 索引值映射

力扣[710题](https://leetcode.cn/problems/random-pick-with-blacklist/)

乍一看，这题可以使用一个数组，然后把其中在黑名单内的元素去除，然后使用随即索引访问即可，我尝试了，但这样的话就`OOM`了

第二种方法是使用哈希表来映射，主要思想如下：

- 最后使用随机索引查找的范围是`n-blacklist.length`，设这个值为边界值`bound`
- 我们只需要将`bound`内部的`blacklist`中的值，映射到`bound`外面的非`blacklist`值即可

我的第一种代码如下：

```java
class Solution
    {
        HashMap<Integer,Integer> hashMap;
        int bound;
        public Solution(int n, int[] blacklist)
        {
            bound=n-blacklist.length;
            hashMap=new HashMap<>();
            HashSet<Integer> set=new HashSet<>();
            for (int i :blacklist)
            {
                set.add(i);
            }
            int i=0;
            int last=n-1;
            while (i<bound)
            {
                if(set.contains(i))
                {
                    while (set.contains(last))
                    {
                        last--;
                    }
                    hashMap.put(i, last);
                    last--;
                }
                i++;
            }
        }
        public int pick()
        {
            Random random=new Random();
            int index=random.nextInt(bound);
            return hashMap.getOrDefault(index, index);
        }
    }
```

可惜最后提交超时了，原因在于在构造映射哈希表的时候，查找范围定的太大了，在所有的`bound`里面查找，但是可以不需要这样，只需要**在`bound`内部的`blacklist`元素中查找**就行了

```java
class Solution
    {
        HashMap<Integer,Integer> hashMap;
        int bound;
        public Solution(int n, int[] blacklist)
        {
            bound=n-blacklist.length;
            hashMap=new HashMap<>();
            HashSet<Integer> set=new HashSet<>();
            for (int i :blacklist)
            {
                set.add(i);
            }
            //指向边界值外部不在blacklist中的值
            int outBound=bound;
            //只在黑名单内部查找，就少了很多遍历和判断的步骤
            for(int i :blacklist)
            {
                if(i<bound)
                {
                    while (set.contains(outBound))
                    {
                        outBound++;
                    }
                    hashMap.put(i,outBound);
                    outBound++;
                }
            }
        }
        public int pick()
        {
            Random random=new Random();
            int index=random.nextInt(bound);
            return hashMap.getOrDefault(index, index);
        }
    }
```

### 数据流的中位数

力扣[295题](https://leetcode.cn/problems/find-median-from-data-stream/)，题目要求很快获得一个数据流的中位数，而且需要不断往数据流中动态添加数据，有一种很巧妙的做法，使用两个优先级队列，两个队列可以自动排序，将序列分成两堆，就很容易获得中位数了

使用一个大顶堆，一个小顶堆：

- 大顶堆的每个节点的值大于等于左右孩子节点的值，堆顶为最大值
- 小顶堆的每个节点的值小于等于左右孩子节点的值，堆顶为最小值

![41-1.png](https://pic.leetcode-cn.com/1638802694-YTYpGU-41-1.png)

我们要保证每次插入元素后，两堆维持相对长度。让minHeap为长度较大的堆，每次插入元素时进行判断：

- 当两堆总长度为偶数时，即两堆长度相等，将新元素插入到minHeap，插入后minHeap比maxHeap长度大1；
- 当两堆总长度为奇数时，即两堆长度不等，将新元素插入到maxHeap，插入后两堆长度相等；

还要保证插入元素后，两堆仍是保证从下往上递增的顺序性。如上面的偶数情况，将新元素x直接插入到minHeap，是有可能破坏两堆的顺序性的，例如：（minHeap是存储“较大一半”的值）

若x的值恰好为“较大一半”，直接将插入到“较大一半”的minHeap中，是不会破坏顺序的；
若x的值为“较小一半”，而此时却插入到“较大一半”的minHeap中，是会破坏顺序的。
那么，每次新元素插入时，都需要先插入到另一个堆，进行重新排序后，再将最值拿出来插入正确的堆中。因此，最终得出的结论为：

- 当两堆总大小为偶数时，即两堆大小相等，先将新元素插入maxHeap，重新排序后将新的最值拿出并插入到minHeap；
- 当两堆总大小为奇数时，即两堆大小不等，先将新元素插入minHeap，重新排序后将新的最值拿出并插入到maxHeap；

代码实现如下：

```java
	class MedianFinder 
    {
        PriorityQueue<Integer> queue1;
        PriorityQueue<Integer> queue2;
        int mid;
        public MedianFinder() 
        {
			//小顶堆，放比中位数大的数
            this.queue1=new PriorityQueue<>((a,b)->(a-b));
			//大顶堆，放比中位数小的数
            this.queue2=new PriorityQueue<>((a,b)->(b-a));
        }
        public void addNum(int num)
        {
            //如果两个堆长度相等
            if(queue1.size()==queue2.size())
            {
                //先放到下面这个堆中排序，然后放到上面
                queue2.offer(num);
                queue1.offer(queue2.poll());
                return;
            }
            //如果长度不等
            if(queue1.size()!=queue2.size())
            {
                //先放到上面这个堆排序，然后加入下面这个堆
                queue1.offer(num);
                queue2.offer(queue1.poll());
                return;
            }
        }

        public double findMedian()
        {
            //题目中说至少有一个元素
            if(queue1.size()==queue2.size())
            {
                return (queue1.peek()+queue2.peek())/2.0;
            }
            else
            {
                return queue1.peek();
            }
        }
    }
```

## 二叉树

### 概念

- 种类

  - 满二叉树
  - 完全二叉树
  - 二叉搜索树
  - 平衡二叉搜索树

- 存储方式

  - 链式存储
  - 数组存储

- 遍历方式

  - 深度优先搜索

    搜到底回去

  - 广度优先搜索

    一层一层遍历（在图里面是一圈一圈遍历）

### 构造虚拟化结构辅助解题

如力扣[116题](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node/)

要求求每个节点下一个右侧节点的指针

![img](https://assets.leetcode.com/uploads/2019/02/14/116_sample.png)

需要注意的是，每个节点默认的`next`是`null`

然后就可以在**左右节点中间**构造出一个虚拟的节点，遍历的时候，每次遍历只做一个操作

![image-20230710152747378](https://s2.loli.net/2023/07/10/pL1VWaXlrDUQzxf.png)

代码如下：

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;
    public Node next;

    public Node() {}
    
    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, Node _left, Node _right, Node _next) {
        val = _val;
        left = _left;
        right = _right;
        next = _next;
    }
};
*/

class Solution {
    
        public Node connect(Node root)
    {
        if (root==null)
        {
            return null;
        }
        traverse(root.left,root.right);
        return root;
    }

    public void traverse(Node left,Node right)
    {
        if (left == null || right == null) {
            return;
        }
            left.next = right;
            traverse(left.left, left.right);
            //连接虚拟节点的左右子节点
            traverse(left.right, right.left);
            traverse(right.left, right.right);
            return;
    }
}
```

或者也可以不使用递归，使用两个队列交替使用，对**完美二叉树**进行层次遍历：

```java
public Node run(Node root)
    {
        if(root==null)
        {
            return null;
        }
        Queue<Node> queue1=new LinkedList<>();
        queue1.offer(root);
        Queue<Node> queue2=new LinkedList<>();
        while (!queue1.isEmpty()||!queue2.isEmpty())
        {
            while (!queue1.isEmpty())
            {
                //完美二叉树所以这么处理
                Node node=queue1.poll();
                node.next=queue1.peek();
                //完美二叉树，有left必有right
                if (node.left!=null)
                {
                    queue2.offer(node.left);
                    queue2.offer(node.right);
                }
            }
            while (!queue2.isEmpty())
            {
                Node node=queue2.poll();
                node.next=queue2.peek();
                if(node.left!=null)
                {
                    queue1.offer(node.left);
                    queue1.offer(node.right);
                }
            }
        }
        return root;
    }
```

### 二叉树的构造

关于二叉树的很多都是使用递归来解决问题，利用递归一般主要采用两种思维方式

- **是否可以通过遍历⼀遍⼆叉树得到答案？**则使用一个 traverse 函数配合外部变量来解决问题，这叫「遍历」的思维模式
- **是否可以定义⼀个递归函数，通过子问题（子树）的答案推导出原问题的答案？**如果可以，写出这个递归函数的定义，并充分利用这个函数的返回值，这叫「分解问题」的思维模式

利用递归来求解，需要思考：
**如果单独抽出⼀个二叉树节点，它需要做什么事情？需要在什么时候（前/中/后序位置）做？其他的节点不用管，递归函数会帮你在所有节点上执行相同的操作（仅一次操作，不要想多了，想多了会绕进去）**

如力扣[105题](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

该题的主要思维方式也是递归，每次做相同的操作，层层深入，层层求解

这题搞清楚坐标，然后递归深入便可求解

> 将大问题简化为递归过程中的小问题，已本题为例，一开始的这个根节点，左右子树和递归过程中小问题里面的根节点，左右子树，差异并不大，也就是简化思维

![image-20230711143916462](https://s2.loli.net/2023/07/11/dnKX4RbUPCwV38k.png)

```java
    public TreeNode buildTree(int[] preorder, int[] inorder)
    {
        return  buildTree(preorder, 0, preorder.length - 1, inorder, 0, inorder.length - 1);
    }

    public TreeNode buildTree(int[] preorder,int preLeft,int preRight,int[] inorder,int inLeft,int inRight)
    {
        if(preLeft>preRight||inLeft>inRight)
        {
            return null;
        }
        int rootVal=preorder[preLeft];
        //这一步实现了本次递归的操作，可以通过空间换时间的哈希表进行优化
        int rootIndex= Arrays.stream(inorder).boxed().toList().indexOf(rootVal);
        TreeNode root=new TreeNode(rootVal);
        root.left=buildTree(preorder,preLeft+1,preLeft+rootIndex-inLeft,inorder,inLeft,rootIndex-1);
        root.right=buildTree(preorder,preLeft+rootIndex-inLeft+1,preRight,inorder,rootIndex+1,inRight);
        return root;
    }
```

再如力扣[889题](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)

同样也是二叉树构造的题目，通过⼆叉树的构造问题⼀般都是使用「分解问题」的思路：

**构造整棵树 = 根节点 + 构造左子树+构造右子树**。先找出根节点，然后根据根节点的值找到左右⼦树的元素，进而递归构建出左右⼦树。

```java
    public TreeNode constructFromPrePost(int[] preorder, int[] postorder)
    {
        return buildTree(preorder,postorder,0,preorder.length-1,0,postorder.length-1);
    }

    public TreeNode buildTree(int[] preorder, int[] inorder,int preLeft,int preRight, int inLeft, int inRight)
    {
        if(preLeft > preRight||inLeft>preRight)
        {
            return null;
        }
        if(preLeft==preRight)
        {
            return new TreeNode(preorder[preLeft]);
        }
        if(inLeft==inRight)
        {
            return new TreeNode(inorder[inLeft]);
        }
        int rootVal = preorder[preLeft];
        TreeNode root=new TreeNode(rootVal);
        int index= Arrays.stream(inorder).boxed().toList().indexOf(preorder[preLeft+1]);
        int size=index-inLeft+1;
        root.left= buildTree(preorder,inorder,preLeft+1,preLeft+size,inLeft,index);
        root.right=buildTree(preorder,inorder,preLeft+size+1,preRight,index+1,inRight-1);
        return root;
    }
```

### 去除重复二叉树

如力扣[652题](https://leetcode.cn/problems/find-duplicate-subtrees/)

![image-20230714160202185](https://s2.loli.net/2023/07/14/ntWBucqQ97oEAJs.png)

由题，可以想到：

- 去重，可以使用**哈希表**
- 对比，一开始想到使用`List<Integer>`这种方式来记录遍历的序列，其实使用**字符串**是最方便的
- 为了降低开销，可以边生成字符串边对比

最后代码如下：

```java
HashMap<String,Integer> map = new HashMap<>();
    List<TreeNode> result=new ArrayList<>();
    public List<TreeNode> findDuplicateSubtrees(TreeNode root)
    {
        traversal(root);
        return result;
    }
    public String traversal(TreeNode root)
    {
        if(root==null) {
            return "#";
        }
        //加空格防止1 11 #和11 1 #这种情况
        String str=root.val+" "+traversal(root.left)+" "+traversal(root.right);
        //题目要求返回一颗子树即可
        if(map.containsKey(str)&&map.get(str)==1)
        {
            result.add(root);
            map.replace(str,map.get(str)+1);
        }
        if(!map.containsKey(str))
        {
            map.put(str,1);
        }
        return str;
    }
```

### 删除二叉搜索树中的节点

力扣[450题](https://leetcode.cn/problems/delete-node-in-a-bst/)

通过交换节点值的方法来删除太过于繁琐，所以通过一个简单的规则来删除节点：

根据二叉搜索树的性质：

- 如果目标节点大于当前节点值，则去右子树中删除；
- 如果目标节点小于当前节点值，则去左子树中删除；
- 如果目标节点就是当前节点，分为以下三种情况：
  - 其无左子：其右子顶替其位置，删除了该节点；
  - 其无右子：其左子顶替其位置，删除了该节点；
  - 其左右子节点都有：其左子树转移到其右子树的最左节点的左子树上，然后右子树顶替其位置，由此删除了该节点。

第三种情况如图：

![450.jpg](https://pic.leetcode-cn.com/1611932922-MelojG-450.jpg)

代码如下：

```java
public TreeNode deleteNode(TreeNode root, int key)
    {
        if(root==null)
        {
            return null;
        }
        if(key==root.val)
        {
            if(root.left==null && root.right==null)
            {
                return null;
            }
            if(root.right==null)
            {
                return root.left;
            }
            if(root.left==null)
            {
                return root.right;
            }
            TreeNode right=root.right;
            TreeNode left=root.left;
            TreeNode p=right;
            while (p.left!= null)
            {
                p=p.left;
            }
            p.left=left;
            root=root.right;
        }
        if(key>root.val)
        {
            //需要注意递归的思想，从一个更高的角度来思考就可以写出递归
            root.right=deleteNode(root.right, key);
        }
        if(key<root.val)
        {
            root.left=deleteNode(root.left, key);
        }
        return root;
    }
```

## 回溯法

### 基础框架

例如力扣[46题](https://leetcode.cn/problems/permutations/)

![image-20230724174642491](https://s2.loli.net/2023/07/24/cUw8ELgDeNK1HTx.png)

利用递归的思想，代码如下：

```java
    List<List<Integer>> result=new ArrayList<>();
    public List<List<Integer>> permute(int[] nums)
    {
        int[] used=new int[nums.length];
        for(int i:used)
        {
            i=0;
        }
        List<Integer> temp=new ArrayList<>();
        backTrack(nums, used, temp);
        return result;
    }

    public void backTrack(int[] nums,int[] used,List<Integer> track)
    {
        if(track.size()==nums.length)
        {
            result.add(new ArrayList<>(track));
            return;
        }
        for(int i=0;i<nums.length;i++)
        {
            if(used[i]==0)
            {
                //本次使用第i个数
                used[i]=1;
                track.add(nums[i]);
                backTrack(nums,used,track);
                //第i个数用过了，恢复成原样，不然会导致他始终是使用的状态
                track.remove(track.size()-1);
                used[i]=0;
            }
        }
    }
```

需要注意的是，直接做:

```
result.add(track);
```

会有问题。因为track是一个引用,每次backtrack递归的时候,track都在原地被修改。如果直接把track添加到result,会导致result里的所有permutation列表都是一模一样的,都引用了同一个track。

所以需要创建一个新的ArrayList,并传入track,来copy track的内容

### 回溯法的终极奥义

回溯法归根结底就是在一个树上搜索，每一个for循环就代表，搜索树上同一层的节点

这样的话，剪枝就好理解了

例如力扣[40题](https://leetcode.cn/problems/combination-sum-ii/description/)

![image-20230725193823905](https://s2.loli.net/2023/07/25/ebD267JlFPgTW3V.png)

- 这种题目，一般可以先排个序，这样后面或许可以避免潜在的问题
- **要求集合内元素的和等于target，那么就需要一个List不断保存搜索过程中路径上的值，以及方法可能需要的其他的中间值**，这就是确定方法参数的方法
- 要求不能包含重复的，那么就可以在搜索的过程中限制**树的起点**，也就是**for循环的起点**
- 同时，还要求搜索的过程中不会又搜索到前面的节点，比如说1，1，2之类的情况，就需要在第一个1搜索结束后，将树的搜索起点放到非1的元素那里

综上，代码如下：

```java
    List<List<Integer>> result=new ArrayList<>();
    public List<List<Integer>> combinationSum2(int[] candidates, int target)
    {
        Arrays.sort(candidates);
        backTrack(candidates,target,new ArrayList<>(),0,0);
        return result;
    }

    public void  backTrack(int[] nums, int target,List<Integer> list,int cur,int begin)
    {
        if(cur==target)
        {
            result.add(new ArrayList<>(list));
            return;
        }
        for (int i=begin;i<nums.length;i++)
        {
            if(cur+nums[i]<=target)
            {
                list.add(nums[i]);
                backTrack(nums,target,list,cur+nums[i],i+1);
                list.remove(list.size()-1);
                while (i+1<nums.length&&nums[i+1]==nums[i])
                {
                    i++;
                }
            }
        }
    }
```

### 岛屿问题

**一般这种情况都是把访问过的岛屿用水淹了**

力扣[200题](https://leetcode.cn/problems/number-of-islands/)

![image-20230726144421500](https://s2.loli.net/2023/07/26/sg3VHtFxoTLSOdz.png)

这道题具体思路是：

遍历数组，遇到一个陆地，结果加一，然后把相邻的陆地给淹了，代码如下：

```java
    int result;
    int m,n;
    int[][] visited;
    //把访问过的陆地以及旁边的陆地都淹掉
    public int numIslands(char[][] grid)
    {
        m=grid.length;
        n= grid[0].length;
        visited=new int[m][n];
        for (int i=0;i<m;i++)
        {
            for (int j=0;j<n;j++)
            {
                if (grid[i][j]=='1' && visited[i][j]==0)
                {
                    result++;
                    backTrack(grid,i,j);
                }
            }
        }
        return result;
    }

    public void backTrack(char[][] grid, int i,int j)
    {
        if(i<0 || i>=m || j<0 || j>=n)
            return;
        if(visited[i][j]==1)
            return;
        visited[i][j]=1;
        if(grid[i][j]=='0')
        {
            return;
        }
        else
        {
            grid[i][j]='0';
            backTrack(grid, i-1, j);
            backTrack(grid, i, j-1);
            backTrack(grid, i, j+1);
            backTrack(grid, i+1, j);
        }

    }
```

再如力扣[1254题](https://leetcode.cn/problems/number-of-closed-islands/)

![image-20230726145028271](https://s2.loli.net/2023/07/26/lKqw7E4yJ21uLOh.png)

统计封闭岛屿数量，由题意可知，数组边缘的岛屿都是不符合要求的，只需要这些岛屿淹掉，然后再次使用上面的方法即可，代码如下：

```java
    int m,n;
    int[][] visited;
    public int closedIsland(int[][] grid)
    {
        m=grid.length;
        n=grid[0].length;
        visited=new int[m][n];
        int result=0;
        //把边缘的岛屿淹了
        for (int i=0; i<m; i++)
        {
            if(i==0)
            {
                for (int j=0;j<n;j++)
                {
                    if(grid[0][j]==0)
                    {
                        backTrack(grid,0,j);
                    }
                }
            }
            if(i!=0&&grid[i][0]==0)
            {
                backTrack(grid,i,0);
            }
            if(i!=0&&grid[i][n-1]==0)
            {
                backTrack(grid,i,n-1);
            }
            if(i==m-1)
            {
                for (int j=0;j<n;j++)
                {
                    if(grid[m-1][j]==0)
                    {
                        backTrack(grid,m-1,j);
                    }
                }
            }
        }
        //复用第200题的方法
        for (int i = 0; i < m; i++) {
            for (int j=0;j<n;j++)
            {
                if(grid[i][j]==0&&visited[i][j]==0)
                {
                    result++;
                    backTrack(grid,i,j);
                }
            }
        }
        return result;
    }

    public void backTrack(int[][] grid, int i, int j)
    {

        if (i<0 || i>=m || j<0 || j>=n)
            return;
        if(visited[i][j]==1)
        {
            return;
        }
        visited[i][j]=1;
        if(grid[i][j]==1)
        {
            return;
        }
        grid[i][j]=1;
        backTrack(grid,i-1,j);
        backTrack(grid, i+1, j);
        backTrack(grid, i, j+1);
        backTrack(grid, i, j-1);
    }
```

再如力扣[1905题](https://leetcode.cn/problems/count-sub-islands)

![image-20230726171104099](https://s2.loli.net/2023/07/26/U3O8spdjLmQSTx7.png)



第一种思路是，只遍历岛屿2，遍历的过程中检查该块区域在不在岛屿1中

代码如下：

```java
    int m,n;
    int[][] visited1,visited2;
    int[][] grid3;
    int count=0;
    public int countSubIslands(int[][] grid1, int[][] grid2)
    {
        m=grid1.length;
        n=grid1[0].length;
        grid3=grid1;
        visited1=new int[m][n];
        visited2=new int[m][n];
        int result=0;
        for(int i=0;i<m;i++) {
            for (int j = 0; j < n; j++)
            {
                if(grid2[i][j]==1&&visited2[i][j]==0)
                {
                    count=0;
                    List<Integer> list=backTrack(grid2,visited2,i,j,new ArrayList<>());
                    if(count==list.size())
                    {
                        result++;
                    }
                }
            }
        }
        return result;
    }

    public List<Integer> backTrack(int[][] grid,int[][] visited,int i,int j,List<Integer> list)
    {
        if(i<0||j<0||i>=m||j>=n||visited[i][j]==1||grid[i][j]==0)
        {
            return list;
        }
        visited[i][j]=1;
        if(grid[i][j]==grid3[i][j])
        {
            count++;
        }
        grid[i][j]=0;
        list.add(grid[i][j]);
        backTrack(grid,visited,i,j-1, list);
        backTrack(grid,visited,i,j+1,list);
        backTrack(grid,visited,i+1, j,list);
        backTrack(grid,visited,i-1,j,list);
        return list;
    }
```

第二种方法是，先排除岛屿2中不在一中的岛屿，再查找岛屿2中岛屿数量

代码如下：

```java
    int m,n;
    int[][] visited1,visited2;
    int[][] grid3;
    public int countSubIslands(int[][] grid1, int[][] grid2)
    {
        m=grid1.length;
        n=grid1[0].length;
        grid3=grid1;
        visited1=new int[m][n];
        visited2=new int[m][n];
        int result=0;
        for(int i=0;i<m;i++) {
            for (int j = 0; j < n; j++)
            {
                if(grid2[i][j]==1&&grid1[i][j]==0)
                {
                    backTrack(grid2,visited2,i,j,new ArrayList<>());
                }
            }
        }
        for(int i=0;i<m;i++) {
            for (int j = 0; j < n; j++)
            {
                if(grid2[i][j]==1&&visited2[i][j]==0)
                {
                    result++;
                    backTrack(grid2,visited2,i,j,new ArrayList<>());
                }
            }
        }
        return result;
    }
    public List<Integer> backTrack(int[][] grid,int[][] visited,int i,int j,List<Integer> list)
    {
        if(i<0||j<0||i>=m||j>=n||visited[i][j]==1||grid[i][j]==0)
        {
            return list;
        }
        visited[i][j]=1;
        if(grid[i][j]==grid3[i][j])

        grid[i][j]=0;
        list.add(grid[i][j]);
        backTrack(grid,visited,i,j-1, list);
        backTrack(grid,visited,i,j+1,list);
        backTrack(grid,visited,i+1, j,list);
        backTrack(grid,visited,i-1,j,list);
        return list;
    }
```

## BFS

### 基本框架

广度优先搜索（层次优先搜索），基本上都是使用队列来实现一层一层的搜索

例如力扣[752题](https://leetcode.cn/problems/open-the-lock/)

![image-20230727121147402](https://s2.loli.net/2023/07/27/oMHzVe2c3LQEUtw.png)

这个题目就类似于图的层次搜索，每一次搜索后，该结果就会生成8种其他的结果，再在下一层的结果上继续搜索，代码如下，

具体还使用了几种优化方法：

- 使用了Set来判断是否包含在内
- 使用Visited进行剪枝操作

```java
    public int openLock(String[] deadends, String target)
    {
        Queue<String> queue = new LinkedList<>();
        Set<String> deadSet=new HashSet<>();
        for (String s : deadends)
            deadSet.add(s);
        queue.offer("0000");
        //剪枝，避免回头路
        HashSet<String> visited=new HashSet<>();
        visited.add("0000");
        int depth=0;
        while (!queue.isEmpty())
        {
            int size=queue.size();
            for (int i = 0; i < size; i++)
            {
                String cur=queue.poll();
                //向周围扩散============================================
                //使用Set快速判断包含关系
                if(deadSet.contains(cur))
                {
                    continue;
                }
                if(cur.equals(target))
                {
                    return depth;
                }
                for (int j = 0; j < 4; j++)
                {
                    String plusOne=plusOne(cur,j);
                    String minusOne=minusOne(cur,j);
                    if(!visited.contains(plusOne))
                    {
                        visited.add(plusOne);
                        queue.offer(plusOne);
                    }
                    if(!visited.contains(minusOne))
                    {
                        visited.add(minusOne);
                        queue.offer(minusOne);
                    }
                }
                //==================================================
            }
            depth++;
        }
        return -1;
    }
	
    public String plusOne(String s,int i)
    {
        char[] sc=s.toCharArray();
        if(sc[i]=='9')
            sc[i]='0';
        else
            sc[i]=(char)(sc[i]+1);
        return new String(sc);
    }
    public String minusOne(String s,int i)
    {
        char[] sc=s.toCharArray();
        if(sc[i]=='0')
            sc[i]='9';
        else
            sc[i]=(char)(sc[i]-1);
        return new String(sc);
    }
```

### 根本思想

BFS 算法并不只是⼀个寻路算法，⽽**是⼀种暴⼒搜索算法，只要涉及暴⼒穷举的问题，BFS 就可以⽤，⽽且可以最快地找到答案**

例如力扣[773题](https://leetcode.cn/problems/sliding-puzzle/)

![image-20230727153046127](https://s2.loli.net/2023/07/27/qz72TBrdQVAKvas.png)

同样也是搜索法，想象成一个结果不断向外扩散，广度优先搜索，求最早的获得满足条件结果的搜索层数

本题要点如下：

- 我们需要达到什么样情况,就说明已经完成,找出最后跳出条件
- 因为board是二维数组,对计算机不是很友好,如果能转化成一维的数组,那就好了（使用String判断）
- 要记录已经访问过的情况,不能重复访问,造成死循环（使用Visited Set 存储访问过的节点）

代码如下：

```java
class Solution {
    public int slidingPuzzle(int[][] board)
    {
        int[][] dirs = {{1, 3}, {0, 2, 4}, {1, 5}, {0, 4}, {1, 3, 5}, {2, 4}};
        StringBuilder s=new StringBuilder();
        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[0].length; j++) {
                s.append(board[i][j]);
            }
        }
        Queue<String> queue=new LinkedList<>();
        queue.add(s.toString());
        HashSet<String> set=new HashSet<>();
        String result="123450";
        set.add(result);
        int step=0;
        while (!queue.isEmpty()) {

            int size = queue.size();

            for (int i = 0; i < size; i++)
            {
                String sq=queue.poll();
                if(sq.equals(result))
                {
                    return step;
                }
                if(set.contains(sq))
                {
                    continue;
                }
                set.add(sq);
                int index0=getIndex(sq);
                for (int j:dirs[index0])
                {
                    char[] sc=sq.toCharArray();
                    char temp=sc[j];
                    sc[j]=sc[index0];
                    sc[index0]=temp;
                    String st=new String(sc);
                    queue.add(st);
                }
            }
            step++;
        }
        return -1;
    }
	//获得0在字符串中的位置
    public int getIndex(String s)
    {
        char[] sc=s.toCharArray();
        for (int i = 0; i < sc.length; i++)
        {
            if(sc[i]=='0')
            {
                return i;
            }

        }
        return -1;
    }
}
```

## 动态规划

**动态规划本质上就是一种穷举**，只不过是递进的穷举，当前状态的值是上一个状态所有可能的最优解“加上1”（或者其他处理方式）

**动态规划就是一步一步穷举出最优的状态（或者说情况），堆叠成了全局最优解**

### 一维的穷举

如力扣[300题](https://leetcode.cn/problems/longest-increasing-subsequence/)

![image-20230730004514415](https://s2.loli.net/2023/07/30/yM8bRnIsPBYUm9S.png)

本题要求求最长的递增子序列，就可以有如下思路：

- 序列长度为0时，长度为0
- 序列长度大于0时，当前最长的递增子序列长度，就是把当前元素加到前面的最长的递增子序列中时的长度（就是找当前状态前面的最优解再加1）

代码如下：

```java
    public int lengthOfLIS(int[] nums)
    {
        int[] dp=new int[nums.length];
        dp[0]=1;
        int max=1;
        for(int i=1;i<nums.length;i++)
        {
            //初始化为1
            dp[i]=1;
            //在前面找最优的
            for(int j=0;j<i;j++)
            {
                if(nums[i]>nums[j])
                {
                    dp[i]=Math.max(dp[i],dp[j]+1);
                }
            }
            max=Math.max(max,dp[i]);
        }
        return max;
    }
```

### 求最长递增子序列的优化（二分法）

主要思想：

- 建立一个一维数组cell，用于存放最长递增子序列的内容

- 对于原序列中的每个元素，如果当前cell序列中没有比当前元素大，则将当前元素添加到cell序列中（cell末尾）

- 如果有元素比当前元素大，则将当前元素替换cell中第一个比它大的元素**（尽可能压缩最长子序列的范围）**

  > 使用Arrays.*binarySearch*实现

代码如下：

```java
	public int getMaxLength(int[] nums)
    {
        int[] cell=new int[nums.length];
        cell[0]=nums[0];
        //要插入的点的索引
        int res=1;
        for (int i=1;i<nums.length;i++)
        {
            if(nums[i]>cell[res-1])
            {
                cell[res]=nums[i];
                res++;
            }
            else
            {
                //将当前元素与cell中比当前元素大的第一个元素进行交换
                int index=Arrays.binarySearch(cell,0,res,nums[i]);
                if(index<0)
                {
                    int index1=-index-1;
                    cell[index1]=nums[i];
                }
            }
        }
        return res;
    }
```



### 二维的穷举

上一题的情况是一维的，意味着当前状态的上一种状态的最有情况从一种状态集合中就能找到

但是还有二维的情况，例如力扣[1143题](https://leetcode.cn/problems/longest-common-subsequence/)

![image-20230730005135617](https://s2.loli.net/2023/07/30/VMQ8H7D5qIJ16Lj.png)

本题在求解的过程中，获取最优的上一种状态时，需要同时考虑两种情况，也就是标题所说的二维的穷举

具体思路如下：

- 在两个字符串其中一个字符串为空字符串时，最长公共子序列的长度为0

- 两个字符串非空时，根据当前搜索的字符串是否相等

  - 如果相等，当前状态就是两个字符串加上当前字符之前的情况加1（加上本字符之前的最长公共子序列长度加1）

  - 如果不等，当前状态就是两个字符串任意不加上当前字符，两个字符串之间最长的公共子序列的长度（继承上一种状态中的最优的情况）

    > 若text1[i] != text2[j]，也就是说两个字符串的最后一位不相等，那么字符串text1的[1,i]区间和字符串text2的[1,j]区间的最长公共子序列长度无法延长，因此f[i][j]就会继承f[i-1][j]与f[i][j-1]中的较大值，即f[i][j] = max(f[i - 1][j],f[i][j - 1])

![image-20230730010220349](https://s2.loli.net/2023/07/30/qQDaePbE8SNkgRB.png)代码如下：

```java
    public int longestCommonSubsequence(String text1, String text2)
    {
        int m=text1.length();
        int n=text2.length();
        int[][] dp=new int[m+1][n+1];
        //空字符串之间的公共子序列的长度为0
        for (int i = 0; i <=m; i++)
        {
            dp[i][0]=0;
        }
        for (int i = 0; i <=n; i++)
        {
            dp[0][i]=0;
        }
        for (int i = 1; i <=m; i++)
        {
            for (int j = 1; j <= n; j++)
            {
                //如果两个字符串的当前字符相等，则公共子序列的长度等于两个字符串的前一个字符的公共子序列的长度+1
                //上一个状态+1
                if(text1.charAt(i-1)==text2.charAt(j-1))
                {
                    dp[i][j]=dp[i-1][j-1]+1;
                }
                //如果不相等，取两个字符串上一个状态的最大值，因为需要求最长的公共子序列
                else
                {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[m][n];
    }
```

再如力扣[354题](https://leetcode.cn/problems/russian-doll-envelopes/)

![image-20230730142832024](https://s2.loli.net/2023/07/30/Z95R4dSHApQz8mN.png)

这题有点不太好想，具体思路如下：

- 先对序列中的第0个元素从小到大排列
- 对序列中每个成员的第1个元素进行从大到小排列（后续要求最长递增子序列，这样就可以保证最长递增子序列中不会取到宽度相同的信封）
- 最终求得的最长递增子序列大小就是信封个数

![image-20230730142941000](https://s2.loli.net/2023/07/30/lbJTfkQcAgFNEae.png)

代码如下：

```java
    public int maxEnvelopes(int[][] envelopes)
    {
        //先按照第0个元素进行升序排序，遇到第0个元素相同的情况下，按照降序排列
        if(envelopes.length==1)
        {
            return 1;
        }
        Arrays.sort(envelopes,(a,b)->a[0]==b[0]?b[1]-a[1]:a[0]-b[0]);
        int[] dp=new int[envelopes.length];
        for (int i = 0; i < envelopes.length; i++)
        {
            dp[i]=envelopes[i][1];
        }
        return getMaxLength(dp);
    }

	//使用二分的查找最长递增子序列方法，不然会超时
    public int getMaxLength(int[] nums)
    {
        int[] cell=new int[nums.length];
        cell[0]=nums[0];
        //要插入的点的索引
        int res=1;
        for (int i=1;i<nums.length;i++)
        {
            if(nums[i]>cell[res-1])
            {
                cell[res]=nums[i];
                res++;
            }
            else
            {
                //将当前元素与cell中比当前元素大的第一个元素进行交换
                int index=Arrays.binarySearch(cell,0,res,nums[i]);
                if(index<0)
                {
                    int index1=-index-1;
                    cell[index1]=nums[i];
                }
            }
        }
        return res;
    }
```

