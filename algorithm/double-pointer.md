## 双指针

双指针算法的内核为，维护两个指针，然后根据这两个指针所指向的值判断该如何移动指针。以在数组上移动为例，根据两个指针的初始位置和移动状态等，可以有以下这些变种：

1. 快慢指针。两个指针从数组同一端开始，同向移动，一个移动的快，一个移动的慢。
2. 滑动窗口。快慢指针的变种，两个指针之间形成的窗口内的值也是我们需要的。
3. 其它变种：
   * 相撞指针。两个指针从数组两端开始，互相靠近。
   * 分散指针。两个指针从数组某一处开始，向两端扩展。



### 1. 快慢指针 <div id="slow-fast"/>

快慢指针在对单向链表的问题上相当有效。

#### 例题

##### Leetcode 141. 环形链表

[题目链接](https://leetcode-cn.com/problems/linked-list-cycle/)

```python
class Solution:
    def hasCycle(self, head: ListNode) -> bool:
        if not head: return False
        slow, fast = head, head.next
        while fast and fast.next:
            slow, fast = slow.next, fast.next.next
            if fast is slow: return True
        return False
```



##### 剑指 Offer 22. 链表中倒数第k个节点

[题目链接](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

```python
class Solution:
    def getKthFromEnd(self, head: ListNode, k: int) -> ListNode:
        slow, fast, steps = head, head, 1
        while fast and fast.next:
            fast = fast.next
            steps += 1
            if steps > k: slow = slow.next
        return slow
```



### 2. 滑动窗口 <div id="slide-window"/>



#### 例题

##### Leetcode 438. 找到字符串中所有字母异位词

[题目链接](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/)



### 3. 其它变种 <div id="other"/>

#### 3.1 相撞指针



#### 3.2 分散指针