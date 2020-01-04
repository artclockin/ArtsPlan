# Arts-013

## 1.Algorithm

31. [Next Permutation](https://leetcode.com/problems/next-permutation/)

Implement **next permutation**, which rearranges numbers into the lexicographically next greater permutation of numbers.

If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).

The replacement must be **[in-place](http://en.wikipedia.org/wiki/In-place_algorithm)** and use only constant extra memory.

Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.

```
1,2,3` → `1,3,2`
`3,2,1` → `1,2,3`
`1,1,5` → `1,5,1
```



**My Solution:**

```Go
func reverse(nums []int, start int) {
	end := len(nums) - 1
	for start < end {
		nums[start],nums[end] = nums[end],nums[start]
		start++
		end--
	}
}

func nextPermutation(nums []int)  {
	start := len(nums) - 2
	for start >= 0 && nums[start] >= nums[start + 1] {
		start--
	}
	if start >= 0 {
		end := len(nums) - 1
		for end >=0  && nums[end] <= nums[start] {
			end--
		}
		nums[start],nums[end] = nums[end],nums[start]
	}
	reverse(nums, start + 1)
}
```



## 2.Review

[Go: Should I Use a Pointer instead of a Copy of my Struct?](https://medium.com/a-journey-with-go/go-should-i-use-a-pointer-instead-of-a-copy-of-my-struct-44b43b104963)

赋值和指针sample：

- 在大量数据分配情况下
```Go
func byCopy() S {
   return S{
      a: 1, b: 1, c: 1,
      e: "foo", f: "foo",
      g: 1.0, h: 1.0, i: 1.0,
   }
}

func byPointer() *S {
   return &S{
      a: 1, b: 1, c: 1,
      e: "foo", f: "foo",
      g: 1.0, h: 1.0, i: 1.0,
   }
}

func BenchmarkMemoryStack(b *testing.B) {
   var s S

   f, err := os.Create("stack.out")
   if err != nil {
      panic(err)
   }
   defer f.Close()

   err = trace.Start(f)
   if err != nil {
      panic(err)
   }

   for i := 0; i < b.N; i++ {
      s = byCopy()
   }

   trace.Stop()

   b.StopTimer()

   _ = fmt.Sprintf("%v", s.a)
}

func BenchmarkMemoryHeap(b *testing.B) {
   var s *S

   f, err := os.Create("heap.out")
   if err != nil {
      panic(err)
   }
   defer f.Close()

   err = trace.Start(f)
   if err != nil {
      panic(err)
   }

   for i := 0; i < b.N; i++ {
      s = byPointer()
   }

   trace.Stop()

   b.StopTimer()

   _ = fmt.Sprintf("%v", s.a)
}

name          time/op
MemoryHeap-4  75.0ns ± 5%
name          alloc/op
MemoryHeap-4   96.0B ± 0%
name          allocs/op
MemoryHeap-4    1.00 ± 0%
------------------
name           time/op
MemoryStack-4  8.93ns ± 4%
name           alloc/op
MemoryStack-4   0.00B
name           allocs/op
MemoryStack-4    0.00
```

通过benchmark测试，可以得出使用副本比指针快8倍。使用指针迫使go编译器将变量定义到堆中，垃圾收集器占据了进程的重要部分。



- 密集型函数调用

```Go
func (s S) stack(s1 S) {}

func (s *S) heap(s1 *S) {}

func BenchmarkMemoryStack(b *testing.B) {
   var s S
   var s1 S

   s = byCopy()
   s1 = byCopy()
   for i := 0; i < b.N; i++ {
      for i := 0; i < 1000000; i++  {
         s.stack(s1)
      }
   }
}

func BenchmarkMemoryHeap(b *testing.B) {
   var s *S
   var s1 *S

   s = byPointer()
   s1 = byPointer()
   for i := 0; i < b.N; i++ {
      for i := 0; i < 1000000; i++ {
         s.heap(s1)
      }
   }
}

name          time/op
MemoryHeap-4  301µs ± 4%
name          alloc/op
MemoryHeap-4  0.00B
name          allocs/op
MemoryHeap-4   0.00
------------------
name           time/op
MemoryStack-4  595µs ± 2%
name           alloc/op
MemoryStack-4  0.00B
name           allocs/op
MemoryStack-4   0.00
```
通过benchmark测试，可以得出堆比栈快点。

可以得出Go使用指针而不是结构副本并不一定总是最佳，使用内存分析有助于了解分配和堆的使用。



## 3.Tips
Golang中init函数，实现包级别的初始化操作，在main函数之前执行。

init函数特点：

- 单个包可以有多个init函数，这些init函数都会执行
- init函数不能有输入、输出参数，也不能被引用
- init函数只初始化一次
- 初始化单进程执行，按包依赖关系顺序执行，递归执行init函数。



init函数用途：

- 初始化变量、资源
- 检查系统状态
- 登记注册
- 实现sync.Once功能

引入包只为执行其init函数，可以使用“import _ 包名”进行导入。




## 4.Share

分享一个技术文章：[golang context机制](https://studygolang.com/articles/25784)

本文介绍了什么是Context，介绍了使用Context常见三个场景（多个Goroutine之间的数据交互、超时控制、上下文控制）并列举一些示例。

Context结构
```Go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```
- Deadline 返回一个time.Time,表示当前Context应该结束的时间，ok则表示有结束时间。

- Done 当Context被取消或者超时的时候返回一个close的channel，告诉给context相关的函数要停止当前工作，然后返回了

- Value context实现共享数据的存储，协程安全

库提供了4个Context实现：

- emptyCtx：空接口，不可取消，没有值，没有deadline。包内定义两个变量background、todo
- cancelCtx：继承Context并实现canceler接口
- timerCtx：继承cancelCtx，并增加timeout机制 
- valueCtx：存储键值对的数据