## 查找

### 1. BFS <div id="bfs"/>

广度优先查找，先遍历当前可选项，查找是否存在目标值，如果不存在，然后再遍历当前这些可选项的下一层可选项。例如在二叉树中，可用来做层序遍历。

#### 例题

##### Leetcode 102. 二叉树的层序遍历

[题目链接](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

```python
class Solution:
    def levelOrder(self, root: TreeNode) -> List[List[int]]:
        ans, stack = [], [root] if root else []
        
        while stack:
            curr_level, next_stack = [], []
            for node in stack:
                curr_level.append(node.val)
                if node.left: next_stack.append(node.left)
                if node.right: next_stack.append(node.right)
            ans.append(curr_level)
            stack = next_stack
        
        return ans
```



### 2. DFS <div id="dfs"/>

深度优先搜索，保持一个方向持续向下搜索目标值（优先向更深处搜索），没有可选项时则退回到上一步。例如二叉树的先序遍历就可视为一个深度优先搜索过程，它递归地持续向左搜索，然后逐渐回退。深度优先搜索也可用于例如迷宫寻找出口等问题。



#### 例题

##### Leetcode 101. 对称二叉树

[题目链接](https://leetcode-cn.com/problems/symmetric-tree/)

此题使用 BFS 很简单直接，尝试使用 DFS 更为巧妙。

```python
class Solution:
    def isSymmetric(self, root: TreeNode) -> bool:
        
        # 只关注两个节点 left 和 right 本身是否对称，那么：
        def sym(left, right):
            # left 和 right 同时为 None，对称
            if not left and not right: return True
            
            # left 和 right 只有一个为 None，不对称
            if not left or not right: return False
            
            # left 和 right 自身的值不想等，不对称
            if left.val != right.val: return False
            
            # left 的左子节点和 right 的右子节点应该对称
            # 同时 left 的右子节点 和 right 的左子节点应该对称
            return sym(left.left, right.right) and sym(left.right, right.left)
        
        if not root: return True
        return sym(root.left, root.right)
```



### 3. 二分查找 <div id="binary-search"/>

二分查找也叫折半查找，涉及三个指针：left，right 和 mid。我们每次在 left 和 right 形成的区间内计算出 mid 的位置，然后对比 mid 所指向的值，根据状态选择应该结束查找还是向左或向右查找。

#### 二分查找的边界问题

二分查找涉及三个指针：left，right 和 mid，涉及的边界问题有：

1. left 和 right 形成的区间是闭区间？开区间？还是左开右闭，左闭右开。区间的类型应该取决于我们期望查找结束时的状态以及left，right的初始值。
2. left 和 right 移动时 是否需要 加1或减1？
3. 查找循环终止条件是 left <= right 还是 left < right ？
4. mid 如何取值？(left + right) >> 1 还是 ((left + right) >> 1) + 1？还是？



#### 例题

##### 举例说明边界的变化情况

区间 [left, right)，为左闭右开区间，每次取 mid  = (left + right) >> 1时，我们都在向下取整，那么更新 right 时就应该为 right = mid，因为此次我们已经遍历了 mid，下一次我们不需要再次遍历 mid 指向的位置。循环终止条件，也应该为 left < right，因为对于区间 [2, 2) 是没有意义的。



##### 以单调不减序列为例

**查找第一次出现的位置**

```python
def index_first(nums, target):
    # [l, r]，闭区间
    l, r = 0, len(nums) - 1
    
    # 当循环结束， l = r，l 和 r 指向的位置就是我们要找的位置
    # 由于要找第一次出现的位置，那么区间应该向前收缩
    # 则我们应该尽量将右指针尽量向左收缩
    while l < r:
        mid = (l + r) >> 1
        # 如果 nums[mid] > target，那么 mid 处一定不是我们想要的位置
        if nums[mid] > target:
            r = mid - 1
        # 如果 nums[mid] = target，那么 mid 有可能是我们想要的位置，将 mid 留在下一次查找的区间内
        elif nums[mid] == target:
            r = mid
        # 如果nums[mid] < target，那么 mid 处一定不是我们想要的位置
        else:
            l = mid + 1
    return l if nums[l] == target else -1
```



**查找最后一次出现的位置**

```python
def index_last(nums, target):
    # [l, r]，闭区间
    l, r = 0, len(nums) - 1
    
    # 当循环结束， l = r，l 和 r 指向的位置就是我们要找的位置
    # 由于要找最后一次出现的位置，那么区间应该向后收缩
    # 则我们应该尽量将左指针尽量向右收缩
    while l < r:
        # **很关键**
        # 由于除数总是向下取整
        # 而我们会在 nums[l] == target 时移动左边界
        # 那么如果仍为 (l + r) >> 1 就会陷入死循环（想象一个数组为[target, target]）
        
        mid = ((l + r) >> 1) + 1
        # 如果 nums[mid] > target，那么 mid 处一定不是我们想要的位置
        if nums[mid] > target:
            r = mid - 1
        # 如果 nums[mid] = target，那么 mid 有可能是我们想要的位置，将 mid 留在下一次查找的区间内
        elif nums[mid] == target:
            l = mid
        # 如果nums[mid] < target，那么 mid 处一定不是我们想要的位置
        else:
            l = mid + 1
    return l if nums[l] == target else -1
```



##### 剑指 Offer 53 - II. 0～n-1中缺失的数字

[题目链接](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/)

```python
class Solution:
    def missingNumber(self, nums: List[int]) -> int:
        # 实际上是在找未缺数的序列的右边界
        l, r = 0, len(nums) - 1

        # 我们希望最后 l = r 指向的是未缺数的序列的右边界
        # 例如输入：[0,1,2,3,4,5,6,7,9]
        # 最后 l 和 r 应该指向 7
        while l < r:
            # 考虑 [0, 2]，下边界 0 总是满足要求，因此 mid 应该向上取
            mid = ((l + r) >> 1) + 1
            # 未失序，左边界移动
            target = mid
            if nums[mid] == target:
                l = mid
            else:
                r = mid - 1
        # 右边界的下一个值就是缺失的值
        # 因此要检查 l 确实是未缺数的序列的右边界
        return l + 1 if nums[l] == l else l
```



