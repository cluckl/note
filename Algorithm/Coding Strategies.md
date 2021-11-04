# 战略
* Guess what the possible knowledges could have relations with this problem,and think how to use them.
* 反向分析，输出推导输入
* 注意数据范围，特别是范围比较小的数据，这种数据可能可以进行**==全局举例==**、**==状态压缩==**等等。
* 对于多个变量相互关联的情况，可以尝试分类讨论，分析出变量在不同情况下对结果产生的作用
* 相同复杂度的算法通过**剪枝**可能会有很大的时间差异
* 根据例子进行**<u>可视化分析</u>**，尝试排除重复操作的计算
* 多维度考虑
  * 考虑限定条件及目标和不同的考虑顺序
  * 尝试放宽和收缩限定条件
  * 考虑重复计算的原因以及解决方式
* ==复盘==：Review right thinking process after solving the problem, and compare it with wrong thinking process . Know how to avoid wrong thinking process and come up to right thinking process faster next time.
# Step

1. 确保正确理解题意。
2. 建模
3. 可视化：尽量将过程显示表达出来，同时表达出所有可能出现的情况。
4. 分析：分析可能涉及到的方法模型。
5. 赋能：实践上一步分析出的方法模型。
6. 检查：检查边界条件。

# 战术
* DP
	* 触发条件：在brute force中出现多次相同的计算过程，考虑使用DP存储计算结果。
	* memories the results of calculation and use memory to decrease calculate steps.
	* 根据状态转移方程的特点，可能调整相关可迭代对象的遍历次序（如从后往前遍历、第一维从后往前遍历，第二维从前往后遍历等等）。
	* 函数的输入或者输出都可能成为DP的参考对象
* Greedy
	
	* 触发条件：按某种顺序遍历集合时，第一次访问到的特定元素必定成为构成最终答案的一部分。
	* 每个step的结果不影响最终结果
* Double pointer(slide window) and Priority Queue(Heap)
  * 触发条件：**Priority Queue** usually related at **extreme values ** and **<u>continuous tables</u>**, and it become double pointer when there are only two objects to consider，
  * 最小堆用于求前k大的元素或第k大的元素；最大堆用于求前k小的元素或第k小的元素
* Key is **<u>extreme/top values</u>** 
  
* Slide window
  * 触发条件：；将满足一定条件下的顺序表的子连续序列形成窗口
  * 使用方法：窗口的右边不断移动，左边在不满足条件下移动直到满足条件
  * 注意事项：注意满足窗口条件的判定

* Monotony queue/stack
	* 触发条件：
	  * 在一个顺序表中找pop出的元素的左右两边第一个大于、小于、等于该元素的元素（也可能是其他可比较的关系），以pop出的元素的视角
	  * 淘汰不可能符合输出相关的对象，以push进的元素为视角
	* 使用方法：以**栈顶**的元素为聚焦点，遍历数组以寻找能让栈顶元素pop的条件
	* 注意事项：strict monotony or no?
* Divide and conquer
	
	* 触发条件：当把一个有序集合divide成两部分，使这两个部分**分别有序**之后，可以更高效地解决问题。
* Recursion
	* condition to stop recursion
	* 当递归函数在输入参数相同的情况下被多次调用，尝试使用**<u>记忆化递归</u>**（Python3中使用`@cache`修饰递归函数）。
	* 将问题分解成相同的小问题，在一定条件下可以转为iteration。
	* 当相同的小问题出现多次时，应该考虑使用DP
	* 当每次递归时自身的递归函数只调用一次时，考虑转换成迭代减少空间复杂度
* DFS/BFS/backtracking
	* 触发条件：搜索潜在答案
	* 使用：
	  * 当需要获取最短答案时使用BFS
	  * when limitation become less, may could be transform to DP 
* Binary Search
	
	* 触发条件：used to search sth. 使用某一**判断函数**（在数组的目标数值查找中，判断函数明显为两个数的大小比较，但有时候判断函数并不明显）可以把潜在答案减少一半，可以作用于array, **int** , etc.
	
	* know the difference between search and find: search may need to **judge** 
	* 注意边界（left和right）的更新，以及终止二分的条件（left是否可以等于right）
* **Partion**
	
	* 源于快排，能在O(N)时间复杂度内将一组元素根据规则（如大小规则，奇偶规则等）**分成两类**，并能够获取这两类元素的**分界点**。
* Coloring
	
	* be used for binary graph，用于对二分图的节点进行分类
* UnionSet
  * 题目有连通，等价的关系
  * 注意使用路径压缩

# 模板

使用方式：**<u>首先构造出模板，再处理业务。</u>**

## Partion

```python
def partion(arr, left, right):
    if left >= right:
        return -1
    
	# random pivot if wanting to avoid worse situation
	# pivotIndex = random.randint(left, right - 1)
    # arr[right - 1], arr[pivotIndex] = arr[pivotIndex], arr[right - 1]
    
    storeIndex = left
    for i in range(left, right - 1):
        if arr[i] < arr[right - 1]:
            arr[i], arr[storeIndex] = arr[storeIndex], arr[i] # switch arr[i] and arr[storeIndex]
            storeIndex += 1
    
    arr[storeIndex], arr[right - 1] = arr[right - 1], arr[storeIndex]
    return storeIndex
```

该思想可以将数组分成两个部分，使左边的部分和右边的部分具体不同的性质，但内部性质相同。

## 单调栈

```python
stack = []
i = 0
size = len(arr)

while stack or i < size: 
    while stack and (i == size or arr[stack[-1]] >= arr[i]): # 根据栈的单调情况取不同的符号，这里是严格单调递增
        preIndex = stack.pop()
        left = preIndex - stack[-1] - 1 if stack else preIndex
        right = i - preIndex - 1

        # 执行题目逻辑

    if i < size:
        stack.append(i)
        i += 1
```

## 并查集

```python
class Union:
    def __init__(self, size):
        self.size = [1 for i in range(size)]
        self.parent = [i for i in range(size)]

    def find(self, index):
        while self.parent[index] != index:
            index = self.parent[index]
        return index
    
    def isConnected(self, i, j):
        return self.find(i) == self.find(j)
    
    def connect(self, i, j):
        if self.isConnected(i, j):
            return 

        if self.size[i] < self.size[j]:
            self.parent[self.find(i)] = j
            self.size[i] += self.size[j]
        else:
            self.parent[self.find(j)] = i
            self.size[j] += self.size[i]    
```

## 二叉树非递归遍历

非递归遍历本质是**模拟栈的调用**。

### 前序遍历

```python
def preorderTraversal(root: TreeNode):
    stack = []
    cur = root

    while stack or cur:
        while cur:
            # visit
            stack.append(cur.right)
            cur = cur.left

        cur = stack.pop()
```

### 中序遍历

```python
def inorderTraversal(root: TreeNode):
    stack = []
    cur = root

    while stack or cur:
        while cur:
            stack.append(cur)
            cur = cur.left

        cur = stack.pop()
        #visit
        cur = cur.right
```

### 后序遍历

```python
def postorderTraversal(root: TreeNode):
    stack = []
    cur = root
    before = None
    
    while stack or cur:
        while cur:
            stack.append(cur)
            cur = cur.left
        
        cur = stack[-1]

        if not cur.right or before == cur.right:
            # visit
            cur = None
            before = stack.pop()
        else:
            cur = cur.right
```

## 前缀树

前缀树（Trie）用于检索字符串数据集中的键。可应用于自动补全、拼写检查及[IP](../Net/%E7%BD%91%E7%BB%9C%E5%B1%82.md)的最长前缀匹配等。

```python
class Trie:

    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.root = Node();

    def insert(self, word: str) -> None:
        """
        Inserts a word into the trie.
        """
        node = self.root
        size = len(word)
        for i in range(size):
            if word[i] not in node.next:
                node.next[word[i]] = Node(word[i])
            node = node.next[word[i]]
            if i == size - 1:
                node.leave = True


    def search(self, word: str) -> bool:
        """
        Returns if the word is in the trie.
        """
        node = self.root
        size = len(word)
        for i in range(size):
            if word[i] not in node.next:
                return False
            node = node.next[word[i]]
            if i == size - 1:
                return node.leave
        return True

    def startsWith(self, prefix: str) -> bool:
        """
        Returns if there is any word in the trie that starts with the given prefix.
        """
        node = self.root
        for char in prefix:
            if char not in node.next:
                return False
            node = node.next[char]
        return True        

class Node:
    def __init__(self, value=None):
        self.next = dict()
        self.val = value
        self.leave = False
```

## 堆



# 容器选择

|          | 数组    | 链表 | hash表 | 红黑树  |
| -------- | ------- | ---- | ------ | ------- |
| 插入     | O(N)    | O(1) | O(1)   | O(LogN) |
| 精确查找 | O(1)    | O(N) | O(1)   | O(LogN) |
| 范围查找 | O(LogN) | O(N) | O(N)   | O(LogN) |
| 删除     | O(N)    | O(1) | O(1)   | O(LogN) |



