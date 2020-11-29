##  回溯算法

回溯算法实际上一个类似枚举的搜索尝试过程，主要是在搜索尝试过程中寻找问题的解，当发现已不满足求解条件时，就 “回溯” 返回，尝试别的路径。回溯法是一种选优搜索法，按选优条件向前搜索，以达到目标。但当探索到某一步时，发现原先选择并不优或达不到目标，就退回一步重新选择，这种走不通就退回再走的技术为回溯法，而满足回溯条件的某个状态的点称为 “回溯点”。许多复杂的，规模较大的问题都可以使用回溯法，有“通用解题方法”的美称。

回溯算法的基本思想是：从一条路往前走，能进则进，不能进则退回来，换一条路再试。



### 例题

#### Leetcode 46. 全排列

[题目链接](https://leetcode-cn.com/problems/permutations/)

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        ans, cands = [], nums[:]
        def backtrack(seq, cands):
            if not cands:
                ans.append(seq[:])
                return
            
            for i, cand in enumerate(cands):
                seq.append(cand)
                backtrack(seq, cands[:i] + cands[i+1:])
                # 回溯做出选择时对数据做出的修改
                seq.pop()
        
        backtrack([], cands)
        return ans
```

