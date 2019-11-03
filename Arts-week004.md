# Arts-004

## 1.Algorithm
94. Binary Tree Inorder Traversal(https://leetcode.com/problems/binary-tree-inorder-traversal/)

Medium

Given a binary tree, return the *inorder* traversal of its nodes' values.

**Example:**

```
Input: [1,null,2,3]
   1
    \
     2
    /
   3

Output: [1,3,2]
```

**Follow up:** Recursive solution is trivial, could you do it iteratively?



**My Solution:**

```Go
type Stack [] *TreeNode

func (stack *Stack) Push(value *TreeNode)  {
	*stack = append(*stack, value)
}

func (stack *Stack) IsEmpty() bool {
	if len(*stack) == 0 {
		return true
	}
	return false
}

func (stack *Stack) Peek() *TreeNode {
	return (*stack)[len(*stack) - 1]
}

func (stack *Stack) Pop() *TreeNode {
	len_stack := len(*stack) -1
	value := (*stack)[len_stack]
	*stack = (*stack)[:len_stack]
	return value
}

/*
* 如果栈顶元素非空，则读取栈顶元素的左节点写入栈，继续左遍历查找，若为空则退出。
* pop栈顶元素，输出val，如果存在右节点压栈，并继续左遍历流程，若不存在继续pop
*/
func inorderTraversal(root *TreeNode) []int {
	result := []int {}
	if root == nil {
		return result
	}
	
	var node *TreeNode
	stack := &Stack{}
	stack.Push(root)

	for {
		if stack.IsEmpty() {
			break
		}

		for {
			if stack.Peek().Left != nil{
				stack.Push(stack.Peek().Left )
			} else {
				break
			}
		}

		for {
			if stack.IsEmpty() {
				break
			}

			node = stack.Pop()
			result = append(result, node.Val)
			if node.Right != nil {
				stack.Push(node.Right)
				break
			}
		}
	}

	return result
}

```



## 2.Review

[Why Generics?](https://blog.golang.org/why-generics)
**GO的范型设计**
Ian Lance Taylor在Gophercon 2019上介绍了关于向Go添加泛型的意义、为什么应该这样做，并介绍Go添加泛型的设计可能的改变。

本文从最简单的切片反转谈起，介绍Go当前的通用编程的缺陷，提出范型解决应用场景，列出设计遵循的原则。

- 尽量减少新概念
- 复杂性落在通用代码的编写者身上，而不是用户身上
- 库开发者和使用者可以独立工作（解耦）
- 构建时间短，执行时间短
- 保持Go的清晰度和简洁性

本文也提出设计草案，包括contract设计，挺有趣。



## 3.Tips
- **Python deque** :
deque遵循先进先出（First-In-First-Out，FIFO）的原则，关键特性是保持队列长度一直不变
```Python
from collections import deque
my_queue = deque(maxlen=10)

for i in range(10):
    my_queue.append(i+1)
print(my_queue)
#deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], maxlen=10)
for i in range(10, 15):
    my_queue.append(i+1)
print(my_queue)    
#deque([6, 7, 8, 9, 10, 11, 12, 13, 14, 15], maxlen=10)

```
-   **Python namedtuple** :
namedtuple() 可以返回一个 tuple，该 tuple 中的每个位置都有固定名称，有点像Scala的Case class
```Python
from collections import namedtuple

Person = namedtuple('Person', 'name age job')
Once the template is created, you can use it to create namedtuple objects. Let’s create 2 namedtuple’s for 2 Persons and print out their representation.
Person = namedtuple('Person', 'name age job')

Kate = Person(name="Kate", age=18, job='Project Manager')
print(Kate) #Person(name='Mike', age=30, job='Data Scientist')
print(Kate.name) #Mike
```




## 4.Share

[Linux 环境写文件如何稳定跑满磁盘 I/O 带宽?](https://lexburner.github.io/linux-io-benchmark/) 

文中做了四个实验：

1. Buffer IO 写入

2. 4K 单次 Direct IO 写入

3. mmap 写入

4. 改进的 mmap 写入

结论是mmap还是很靠谱方案，建议可以玩一玩。