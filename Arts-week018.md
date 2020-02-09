# Arts-018

## 1.Algorithm

79. [Word Search](https://leetcode.com/problems/word-search/)

Given a 2D board and a word, find if the word exists in the grid.

The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.

**Example:**

```python
board =
[
['A','B','C','E'],
['S','F','C','S'],
['A','D','E','E']
]

Given word = "ABCCED", return true.
Given word = "SEE", return true.
Given word = "ABCB", return false.
```



**My Solution:**

```go
func searchFunc(board *[][]byte, word string, pos int,  x int, y int) bool {
	tmp := (*board)[x][y]

	if word[pos] != tmp {
		return false
	}
	if len(word) - 1 == pos {
		return true
	}

	(*board)[x][y] = 0;
	pos++

	if (x >0 && searchFunc(board, word, pos, x-1, y)) ||
		(y >0 && searchFunc(board, word, pos, x, y-1)) ||
		(x < len(*board)-1 && searchFunc(board, word, pos, x + 1, y)) ||
		(y < len((*board)[0])-1 && searchFunc(board, word, pos, x, y+1)) {
		return true
	}
	(*board)[x][y] = tmp
	return false
}

func exist(board [][]byte, word string) bool {
	if nil == board || word == "" {
		return  false
	}

	for x := 0; x< len(board);x++  {
		for y:= 0; y< len(board[0]); y++ {
			if searchFunc(&board, word, 0, x, y) {
				return true
			}
		}
	}
	return false
}

```



## 2.Review

[Rob Pike's 5 Rules of Programming](https://blog.codonomics.com/2017/09/rob-pikes-5-rules-of-programming.html)

1. **你无法准确说出程序运行所花费的时间。** 瓶颈出现在意外之处，因此不要去尝试猜测和加快速度直到你已证明这就是瓶颈所在。

2. **度量。** 不要调整速度直到你已进行了度量，即使这样也不要动，除非代码的一部分能够覆盖其余部分。

3. **当n小时，花哨的算法很慢，n通常很小。**花哨的算法有很大的常量。不要幻想除非你确切知道n在频繁地变的很大。(即使n确实变大，首先使用规则2）

4. **花哨算法比简单算法笨拙，并且实现更加困难。** 使用简单的算法以及简单的数据结构。

5. **数据主导。** 如果你已选择正确的数据结构且正确组织他们，那么算法几乎总是不言而喻的。数据结构而非算法，是编程的核心。



## 3.Tips

**Golang中 struct标签**

struct标签常用于json、xml等相关解析中，它提供了一种元信息，可以通过反射解析元信息进行自定义业务处理。

```go
const tagName = "check"
type User struct {
	Id  int
	Email  string `check:"[\\w+\\-.]+@[a-z\\d\\-]+(\\.[a-z]+)*\\.[a-z]+"`
}

func (user User)isValid() bool {
	v := reflect.ValueOf(user)
	for i := 0; i < v.NumField(); i++ {
		tag := v.Type().Field(i).Tag.Get(tagName)
		if tag == "" {
			continue
		}
		filedValue := fmt.Sprintf("%s",v.Field(i).Interface())
		rExpression := regexp.MustCompile(tag)
		if !rExpression.MatchString(filedValue) {
			return false
		}
	}
	return true
}


func TestUser1(t *testing.T) {
	user := User {
		Id : 10,
		Email : "abc@go.com",
	}
	actual := user.isValid()
	expected:= true
	if actual != expected {
		t.Errorf("user:%v, actual:%v, expected:%v", user, actual, expected)
	}
}

func TestUser2(t *testing.T) {
	user := User {
		Id : 10,
		Email : "abc@gocom",
	}
	actual := user.isValid()
	expected:= true
	if actual != expected {
		t.Errorf("user:%v, actual:%v, expected:%v", user, actual, expected)
	}
}
```
运行结果：

```go
=== RUN   TestUser1
--- PASS: TestUser1 (0.00s)
=== RUN   TestUser2
--- FAIL: TestUser2 (0.00s)
    structdemo_test.go:54: user:{10 abc@gocom}, actual:false, expected:true
FAIL

Process finished with exit code 1
```

 


## 4.Share

分享一个技术文章：[深度解密Go语言之unsafe](https://www.cnblogs.com/qcrao-2018/p/10964692.html)

文章从指针谈起，介绍go指针解决的问题及局限（函数传参是值拷贝），进而介绍usafe的实现原理及如何使用。

**Go指针限制：**

- 不能进行数学运算，如p++
- 不同类型指针不能互换
- 不同类型指针不能使用==或!=比较（nil除外）
- 不同类型指针不能相互赋值



**非安全类型指针：unsafe.Pointer。**通过unsafe可以绕过 Go 语言的类型系统，直接操作内存。

```go
//unsafe包定义
type ArbitraryType int 
type Pointer *ArbitraryType

func Sizeof(x ArbitraryType) uintptr //返回类型 x 所占据的字节数，但不包含 x 所指向的内容的大小。
func Offsetof(x ArbitraryType) uintptr //结构体成员在内存中的位置离结构体起始处的字节数
func Alignof(x ArbitraryType) uintptr //当类型进行内存对齐时，它分配到的内存地址能整除 m。
```

`Arbitrary` 是任意的意思，也就是说 Pointer 可以指向任意类型，实际上它类似于 C 语言里的 `void*`。

unsafe 包提供了 2 点重要的能力：

- 任何类型的指针和 unsafe.Pointer 可以相互转换。
- uintptr 类型和 unsafe.Pointer 可以相互转换。uintptr 类型可进行数学运算



使用示例：

```go
// runtime/slice.go 

type slice struct {    
	array unsafe.Pointer // 元素指针    
	len   int // 长度     
	cap   int // 容量
 }

func main() {
    s := make([]int, 9, 20)
    var Len = *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + uintptr(8)))
    fmt.Println(Len, len(s)) // 9 9

    var Cap = *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + uintptr(16)))
    fmt.Println(Cap, cap(s)) // 20 20
}
```

