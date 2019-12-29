# Arts-012

## 1.Algorithm

215. [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

Find the **k**th largest element in an unsorted array. Note that it is the kth largest element in the sorted order, not the kth distinct element.

**Example 1:**

```
Input: [3,2,1,5,6,4] and k = 2
Output: 5
```

**Example 2:**

```
Input: [3,2,3,1,2,4,5,5,6] and k = 4
Output: 4
```

**Note:**
You may assume k is always valid, 1 ≤ k ≤ array's length.

 

**My Solution:**

```Go
type NHeap struct{
	maxSize int
	heap  []int
	size   int
}

func CreateHeap (nums []int,k int) *NHeap{
	nHeap  := &NHeap{k,[]int{},0}
	return nHeap
}

func (n *NHeap) add (val int) {
	if n.size < n.maxSize{
		n.heap = append(n.heap,val)
		n.size++
		n.heapUp(n.size-1)
	}else{
		if val > n.heap[0]{
			n.heap[0] = val
			n.heapDown(0)
		}
	}
}
func (n *NHeap) swap(x,y int){
	// swap heap node value
	n.heap[x],n.heap[y] = n.heap[y],n.heap[x]
}


func (n *NHeap) heapUp(index int){
	for index > 0 && n.heap[(index-1)/2] > n.heap[index] {
		n.swap(index,(index-1)/2)
		index = (index-1)/2
	}
}

func (n *NHeap) heapDown(index int){
	for index < n.size-1{
		left := 2*index +1
		right := left +1

		min := index
		if left <n.size && n.heap[left ] < n.heap[index]{
			min = left
		}
		if right <n.size && n.heap[right]< n.heap[min]{
			min = right
		}
		if min != index{
			n.swap(min,index)
			index = min
		}else{
			break
		}
	}

}

func findKthLargest(nums []int, k int) int {
	m:=CreateHeap(nums,k)
	for _, item := range nums{
		m.add(item)
	}
	return m.heap[0]
}
```



## 2.Review

[Code duplication is not always evil](https://schneide.blog/2019/12/27/code-duplication-is-not-always-evil/)

作者对“代码重复”概念进一步思考，认为这个词描述太模糊。为了避免重复而重用代码可能得不偿失。作者举例：

```Java
public class Price {
    public final BigDecimal amount;
    public final Currency currency;
 
    public Price(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }
 
    // more methods like add(Price)
}
 
public class Discount {
    public final BigDecimal amount;
    public final Currency currency;
 
    public Discount(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }
 
    // more methods like add(Discount)
}
```
价格和折扣的初始领域实体可以以完全相同的方式实现，但它们是完全不同的抽象，在后期行为中可能完全不同。如果设计初期使用一个对象描述，后期再拆开代价会很高。

作者进而得出结论：相同抽象的重复代码是软件设计中万恶之源。

如果通过消除代码重复来引入独立概念的耦合，往往比真正的代码复制更难发现和解决。
“复制允许代码独立发展”这个思维也很重要。



## 3.Tips
brew update卡死，换科大源
```Shell
#替换 homebrew-core源
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin git://mirrors.ustc.edu.cn/homebrew-core.git

#替换 Homebrew Bottles源
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile

brew update
```


## 4.Share

[Go：协程，操作系统线程和 CPU 管理](https://studygolang.com/articles/25292)

本文介绍了协程和线程管理。

- 每个协程（G）运行在与一个逻辑 CPU（P）相关联的 OS 线程（M）上。
-  Go 根据机器逻辑 CPU 的个数来创建不同的 P，并且把它们保存在一个空闲 P 的 list 里。
- 新创建的已经准备好的协程会唤醒一个 P，这个 P 通过与之相关联的 OS 线程来创建一个 M。
- 在程序启动阶段，Go 就已经创建了一些 OS 线程并与 M 相关联了。
- 在系统调用中，Go 不会限制可阻塞的 OS 线程数。GOMAXPROCS 变量表示可同时运行用户级 Go 代码的操作系统线程的最大数量。
- Go 优化了线程使用，所以当协程阻塞时，它仍可复用。

