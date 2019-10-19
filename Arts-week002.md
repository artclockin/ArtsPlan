# Arts-002

## 1.Algorithm
4. [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)

There are two sorted arrays **nums1** and **nums2** of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

You may assume **nums1** and **nums2** cannot be both empty.

**Example 1:**

```
nums1 = [1, 3]
nums2 = [2]

The median is 2.0

```

**Example 2:**

```
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```

**My Solution:**


```Go
func updatePrevLastValue(arr []int, pos int, prevLast []int) {
	if pos >= 2 {
		if len(arr) > pos -2 && arr[pos-2] > prevLast[0] {
			prevLast[0] = arr[pos-2]
		}
		if len(arr) > pos -1 && arr[pos -1] > prevLast[1] {
			if prevLast[0] < prevLast[1] {
				prevLast[0] = prevLast[1]
			}
			prevLast[1] = arr[pos -1]
		}
	} else if pos >=1 {
		if len(arr) > pos -1 && arr[pos -1] > prevLast[1] {
			if prevLast[0] >= prevLast[1] {
				prevLast[1] = arr[pos-1]
				prevLast[0] = prevLast[1]
			} else {
				prevLast[0] = prevLast[1]
				prevLast[1] = arr[pos-1]
			}
		}
	} else {
		prevLast[0] = prevLast[1]
	}
}

func updateLastValueCur(prevLast [] int, curValue int) {
	if curValue > prevLast[1] {
		prevLast[0] = prevLast[1]
		prevLast[1] = curValue
	} else if curValue > prevLast[0] {
		prevLast[0] = curValue
	}
}

func uptdateNexValue(prev_last []int, arr []int, pos int) {
	if len(arr) > pos && pos >= 0{
		updateLastValueCur(prev_last, arr[pos])
	}
}

func binarySearchPos(arr []int, maxPos int, val int) int{
	arr = arr[:maxPos]
	valPos := 0
	low := 1
	mid := 0
	high := len(arr)
	for {
		if low > high {
			break
		}

		mid = (low + high)/2
		guess := arr[mid-1]
		if guess < val {
			valPos += mid
			arr=arr[mid:]
			high = len(arr)
		} else if guess > val {
			high = high / 2
		} else {
			valPos += mid
			break
		}
		if len(arr) == 0 {
			break
		} else if arr[0] > val {
			break
		}
	}

	repeatNums := 0
	if len(arr) > mid {
		for _, nextValue := range (arr[mid:]) {
			if nextValue == val {
				repeatNums += 1
			} else {
				break
			}
		}
	}
	valPos += repeatNums
	return valPos
}

func judgePos(findPos int, finalPos int, arr []int) int {
	if len(arr) == 0 {
		return 0
	}
	l1 := int((len(arr) + 1)/2)
	step := (finalPos - findPos)/2
	if l1 >= step {
		l1 = step
	}
	if l1 < 1 {
		l1 = 1
	}
	return  l1
}

func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
	const INT_MAX = int(^uint(0) >> 1)
	const INT_MIN = ^(INT_MAX)
	finalPos := int((len(nums1) + len(nums2) + 1)/2)//last position
	prevLast := []int{INT_MIN,INT_MIN} //prev last max values
	nextValue := INT_MAX//tmp value
	findPos := 0 //current pos
	nextPos := 0 //next arr remove pos
	diffPos := findPos - finalPos

	twoNum := false
	if (len(nums1) + len(nums2)) % 2 == 0 {
		twoNum = true
	}

	//define l1 position and l2 postion
	l1 := 0
	l2 := 0
	for {
		if len(nums2) == 0 {
			updatePrevLastValue(nums1, finalPos-findPos, prevLast)
			uptdateNexValue(prevLast, nums1, finalPos-findPos)
			break
		} else if len(nums1) == 0 {
			updatePrevLastValue(nums2, finalPos-findPos, prevLast)
			uptdateNexValue(prevLast, nums2, finalPos-findPos)
			break
		}
		l1 = judgePos(findPos, finalPos, nums1)
		l2 = judgePos(findPos, finalPos, nums2)

		//exchange arr if value1 > value2
		if nums1[l1 - 1] > nums2[l2 - 1] {
			nums1, nums2 = nums2, nums1
			l1, l2 = l2,l1
		}

		diffPos = findPos + l1 + l2 -1 - finalPos
		if diffPos == 0 {
			prevLast[0] = nums1[l1-1]
			if len(nums1) > l1 {
				nextValue = nums1[l1]
			}
			if nextValue > nums2[l2-1] {
				nextValue = nums2[l2 -1]
			}
			prevLast[1] = nextValue
			break
		} else if diffPos == 1 {
			prevLast[0] = prevLast[1]
			prevLast[1] = nums1[l1-1]
			break
		} else if diffPos > 1 {
			panic("should not execute here")
		} else if diffPos < 0 {
			findPos += l1
			updatePrevLastValue(nums1, l1 ,prevLast)
			nextPos = binarySearchPos(nums2, l2, nums1[l1-1])
			if nextPos >0 {
				findPos += nextPos
				updatePrevLastValue(nums2, nextPos, prevLast)
				nums2 = nums2[nextPos:]
			}
			nums1 = nums1[l1:]
		}
	}
	if !twoNum {
		prevLast[1] = prevLast[0]
	}

	return float64(prevLast[0]+prevLast[1])/2.0
}
```



## 2.Review

[Machine Translation: A Short Overview](https://towardsdatascience.com/machine-translation-a-short-overview-91343ff39c9f)
**本文列举了机器翻译的3个阶段：**

- 基于规则的机器翻译 (RBMT): 1970s-1990s
  - 优点：
    - 不需要双语文本
    - 领域无关
    - 完全控制（每种情况下都可能有新规则对应）
    - 可重用性（当与新语言配对时，语言的现有规则可以被转移）
  - 缺点：
    - 需要好的字典
    - 手动设置规则（需要专业知识）
    - 规则越多，系统处理就越困难
- 基于统计学的机器翻译(SMT): 1990s-2010s
  - 优点：
    - 语言专家手工工作较少
    - 适合许多语言词对场景(基于Bayes统计)
    - 减少字典外翻译，使用合适语言模型可使翻译更流畅
  - 缺点：
    - 需要双语语料库
    - 特定错误很难修复
    - 不太适合单词顺序差异较大的语言对需要好的字典
- 基于深度网络机器翻译(NMT): 2014-至今
  - 优点：
    - 端到端模型（没有特定任务的流程）
  - 缺点：
    - 需要双语语料库
    - 稀有字问题

   文中分别介绍了各个阶段一些代表性软件和优缺点。如Apertium是RBMT开源软件的代表，Moses是SMT开源软件，基于神经网络就非OpenNMT莫属了。
   本文是机器翻译的入门导读，给了不少参考链接，有兴趣的可以了解一下。



## 3.Tips

interface类型定义了一组方法，如果某个对象实现了某个接口的所有方法，则此对象就实现了此接口。
所以任意的类型都实现了空interface(interface{})，也就是包含0个method的interface。那如何根据变量确定变量类型呢？主要有两种：

- 断言类型变量 value, ok = ele.(T) 或 i.(type) //只能在switch中使用
- 反射 t := reflect.TypeOf(var) 

再看实现了String方法的接口：
fmt/print.go

```Go
// Stringer is implemented by any value that has a String method,
// which defines the ``native'' format for that value.
// The String method is used to print values passed as an operand
// to any format that accepts a string or to an unformatted printer
// such as Print.
type Stringer interface {
	String() string
}
```
String()相当于Java的toString方法，写个简单的例子
```Go
package main
import (
	"fmt"
	"reflect"
)

type Animal interface {
}

type Dog struct {
    name string
    color string
}

type Cat struct {
    name string
    color string
}

//实现fmt.Stringer相当于Java的toString方法
func (d Dog) String() string {
    return "[" + d.name + ": " + d.color + "]"
}

func main() {
	dog1 := Dog{"Dog Army", "Brown"}
	cat1 := Cat{"Cat Bob", "Red"}
	fmt.Println(dog1)
	fmt.Println(cat1)

	type List [] Animal
	animals := make(List, 2)
	animals[0] = cat1
	animals[1] = dog1

	for _, animal := range animals{
		switch value := animal.(type) {
		case Cat:
			fmt.Println(value, " is a cat")
		case Dog:
			fmt.Println(value, " is a Dog")
		}
	}

	if value,ok := animals[0].(Cat);ok {
		fmt.Println(value, " is a cat")
	}
	v := reflect.ValueOf(dog1)
	fmt.Println("type:", v.Type())
}
```
运行结果：
```
[Dog Army: Brown]
{Cat Bob Red}
{Cat Bob Red}  is a cat
[Dog Army: Brown]  is a Dog
{Cat Bob Red}  is a cat
type: main.Dog
```




## 4.Share

[How to Be a Good Senior Developer](https://levelup.gitconnected.com/how-to-be-a-good-senior-developer-958948e02ada) - It’s not what you do—it’s how you do it

**Commit mistake, correct mistake, learn from mistake, share mistake—repeat.**
犯错误，改正错误，从错误中学习，分享错误，重复。

**A senior developer is a person who can bring in 10x more value to the company. These are developers who know how “good stuff” works and can deliver value to the customer.**
高级开发人员是指能够为公司带来10倍以上价值的人。这些开发人员知道“好东西”是如何工作的，能够为客户提供价值。

**高级开发人员特质：**

- They’re Curious 好奇心
- They Learn Their Chosen Platform Inside Out 从内而外地学习
- They’re Great Mentors 他们是伟大的导师
- They Don’t Have “Shiny-Toy” Syndrome 他们没有“发光玩具”综合症（他们确切地知道什么时候不做某事）
- They Admit What They Don’t Know 承认自己不知道的
- They Can Detect the Smell of Bad Code 他们可以检测到错误代码的气味




