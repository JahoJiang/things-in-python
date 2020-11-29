## 排序

排序算法分为初级排序和高级排序。初级排序的算法复杂度为 O(N^2)，高级排序的算法复杂度为 O(NlogN)。（本文以升序排序为例）

### 1. 初级排序 <div id="elementary"/>

初级排序的算法总是两层循环。

1. 选择排序：每次从未排序的部分选出最小值，将其追加到已排序部分的末尾。

2. 冒泡排序：依次两两比较，如果这两个位置的元素不符合排序要求，则交换位置。其实相当于，每次外层循环就将一个最大值“沉底”，因此我认为也可以叫做 “沉底排序”。叫做冒泡排序，是因为每次外层循环都将较小的值逐渐上升到数组的头部，类似于冒泡。

   ```python
   def bubble_sort(nums):
       print('Nums: ', nums)
       for i in range(len(nums)):
           # [len(nums) - i:len(nums)] 为是“沉底”部分，已经有序
           for j in range(1, len(nums) - i):
               if nums[j] < nums[j-1]: nums[j-1], nums[j] = nums[j], nums[j-1]
        print('Sorted:' , nums)
   ```

3. 插入排序：将当前值插入到已排序部分的合理位置，保证已排序部分始终有序。

4. 希尔排序：插入排序仅在数组部分有序时比较高效，希尔排序先将间隔为H的元素有序，然后逐渐将 H 收敛至 1。

   ```python
   def shell_sort(nums):
       print('Nums: ', nums)
       N, h = len(nums), 1
       while h < N / 3: h = 3 * h + 1
       while h > 0:
           for i in range(h, N):
               j = i
               while j >= h and nums[j] < nums[j-h]:
                   nums[j-h], nums[j] = nums[j], nums[j-h]
                   j -= h
           h //= 3
       print('Sorted:' , nums)
   ```



### 2. 堆排序 <div id="heap-sort"/>

堆排序建立在最大堆的基础上，排序之初需要根据数组初始化一个堆，然后根据从堆顶不断拿出当前最大值，放到数组后方，从而实现排序。

堆中 i 处的节点，它的左子节点为 2 * 1，右子节点为 2 * i + 1

```python
def heap_sort(nums):
    print('Nums: ', nums)
    N = len(nums)
    # 将 nums 转为 最大堆
    def heapify():
        mid = N // 2 - 1
        for i in range(mid, -1, -1):
            sink(i, N-1)
        
    
    # 从 i 处调整一次最大堆
    def sink(i, end):
        child = 2 * i
        while child <= end:
            if child + 1 <= end and nums[child+1] > nums[child]:
                child += 1
            
            if nums[i] < nums[child]:
                nums[i], nums[child] = nums[child], nums[i]
                i = child
            else:
                break
            
    
    # 将数组排序
    def sort():
        for end in range(N-1, 0, -1):
            # 将最大值交换到数组末尾的已排序序列的对应位置
            nums[0], nums[end] = nums[end], nums[0]
            # 以新的堆的末尾对当前堆进行调整
            sink(0, end - 1)
    
    heapify()
    print('Heapify: ', nums)
    sort()
    print('Sorted:' , nums)

```





### 3. 快速排序 <div id="quick-sort"/>

快速排序是一种基于分治思想的排序，他将一个数组分为两个子数组，将两部分进行独立地排序。归并排序将数组分成两个子数组分别排序，并将有序的子数组归并以将整个数组排序；而快速排序将数组排序的方式是：当两个子数组都有序时整个数组就自然地有序了。

```python
def quick_sort(nums):
    print('Nums: ', nums)
    
    def partition(nums, lo, hi):
        i, j, pivot = lo + 1, hi, nums[lo]
        while True:
            # 向右找到一个比 pivot 大的值
            # 即让 i 之前的值都小于等于 pivot
            # i == j 也有意义，因为我们要和 pivot 做比较然后将 i 或 j 移到正确的位置
            while i <= j and nums[i] <= pivot: i += 1
            
            # 向左找到一个比 pivot 小的值
            # 即让 j 之后的值都大于 pivot
            while i <= j and nums[j] > pivot: j -= 1
            
            # 确保 i，j 有意义，进行交换
            if i >= j: break
            nums[i], nums[j] = nums[j], nums[i]
            
        # 做最后一次交换
        nums[lo], nums[j] = nums[j], nums[lo]
        return j
      
    def sort(nums, lo, hi):
        if hi <= lo: return
        pos = partition(nums, lo, hi)
        sort(nums, lo, pos - 1)
        sort(nums, pos + 1, hi)
    
    sort(nums, 0, len(nums)-1)
    print('Sorted:' , nums)
```



#### 例题

快排的精妙之处是，循环一次便找到了 前 K小的数字，第K个数字。我们可以利用这个性质解决很多问题。

##### Leetcode 973. 最接近原点的 K 个点

[题目链接](https://leetcode-cn.com/problems/k-closest-points-to-origin/)

```python
class Solution:
    # 求K个，第K个的，都可以用快排的分治思想进行处理
    def kClosest(self, points: List[List[int]], K: int) -> List[List[int]]:
        distance = lambda i: points[i][0] ** 2 + points[i][1] ** 2
        def solve(lo, hi, k):
            if hi <= lo: return
            pivot, swap_i = distance(lo), lo

            # 先将 pivot 值移动到末尾，保证其不会被打乱
            points[lo], points[hi] = points[hi], points[lo]

            # 开始从头到尾寻找比 pivot 小的，小的话就交换到 swap_i 处
            for i in range(lo, hi + 1):
                if distance(i) < pivot:
                    points[i], points[swap_i] = points[swap_i], points[i]
                    swap_i += 1
            
            # 将 pivot 值放在合适的位置
            points[swap_i], points[hi] = points[hi], points[swap_i]
            # 只需要找出前K个
            if swap_i - lo + 1 > k:
                solve(lo, swap_i - 1, k)
            elif swap_i - lo + 1 < K:
                solve(swap_i + 1, hi, k - (swap_i - lo + 1))

        solve(0, len(points) - 1, K)
        return points[:K]
```



### 4. 归并排序 <div id="merge-sort"/>

归并排序将数组分成两个子数组分别排序，并将有序的子数组归并以将整个数组排序。

```python
def merge_sort(nums):
    print('Nums:' , nums)
    # 辅助数组
    aux = [0] * len(nums)
    def merge(nums, lo, mid, hi):
        # i, j 是两个有序序列的指针
      	# 由于 mid 是向下取整，很有可能等于 lo
        i, j = lo, mid + 1
        
        # 将数字拷贝至辅助数组
        for k in range(lo, hi + 1): aux[k] = nums[k]
        
        for k in range(lo, hi + 1):
            # 前半序列已经使用完
            if i > mid:
                nums[k] = aux[j]
                j += 1
            # 后半序列已经使用完
            elif j > hi:
                nums[k] = aux[i]
                i += 1
            elif aux[i] < aux[j]:
                nums[k] = aux[i]
                i += 1
            else:
                nums[k] = aux[j]
                j += 1
      
    def sort(nums, lo, hi):
        if hi <= lo: return
        mid = (lo + hi) >> 1
        sort(nums, lo, mid)
        sort(nums, mid + 1, hi)
        merge(nums, lo, mid, hi)
    
    sort(nums, 0, len(nums) - 1)
    print('Sorted:' , nums)
```

