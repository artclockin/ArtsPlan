# Arts-016

## 1.Algorithm

720. [Longest Word in Dictionary](https://leetcode.com/problems/longest-word-in-dictionary/)

    Given a list of strings `words` representing an English Dictionary, find the longest word in `words` that can be built one character at a time by other words in `words`. If there is more than one possible answer, return the longest word with the smallest lexicographical order.

If there is no answer, return the empty string.

**Example 1:**

```
Input: 
words = ["w","wo","wor","worl", "world"]
Output: "world"
Explanation: 
The word "world" can be built one character at a time by "w", "wo", "wor", and "worl".
```



**Example 2:**

```
Input: 
words = ["a", "banana", "app", "appl", "ap", "apply", "apple"]
Output: "apple"
Explanation: 
Both "apply" and "apple" can be built from other words in the dictionary. However, "apple" is lexicographically smaller than "apply".
```



**Note:**

All the strings in the input will only contain lowercase letters.

The length of `words` will be in the range `[1, 1000]`.

The length of `words[i]` will be in the range `[1, 30]`.



**My Solution:**

```Go
//Trie + DFS
const MAXCAP = 26

type Trie struct {
	next map[rune]*Trie
	lastIndex int
	words []string
}

type Stack [] *Trie

func (stack *Stack) Push(value *Trie)  {
	*stack = append(*stack, value)
}

func (stack *Stack) IsEmpty() bool {
	if len(*stack) == 0 {
		return true
	}
	return false
}

func (stack *Stack) Peek() *Trie {
	return (*stack)[len(*stack) - 1]
}

func (stack *Stack) Pop() *Trie {
	len_stack := len(*stack) -1
	value := (*stack)[len_stack]
	*stack = (*stack)[:len_stack]
	return value
}


func CreateTrie() *Trie {
	root := new(Trie)
	root.next = make(map[rune]*Trie, MAXCAP)
	root.lastIndex = -1
	return root
}


func (this *Trie) Insert(word string, index int ) {
	cur := this
	for _, v := range word {
		if cur.next[v] == nil {
			node := new(Trie)
			node.next = make(map[rune]*Trie, MAXCAP)
			node.lastIndex = -1
			cur.next[v] = node
		}
		cur = cur.next[v]
	}

	cur.lastIndex = index
}


func (this *Trie)searchLongestWord() string {
	result := ""
	stack := &Stack{}
	stack.Push(this)
	for !stack.IsEmpty() {
		node := stack.Pop()
		if node.lastIndex > -1 || node == this {
			if node != this {
				word := this.words[node.lastIndex]
				if len(word) > len(result) || len(word) == len(result)   && strings.Compare(word, result) < 0 {
					result = word
				}
			}
			for _, subNode := range (node.next) {
				stack.Push(subNode)
			}
		}
	}
	return result
}

func longestWord(words []string) string {
	root := CreateTrie()
	for index, word := range words {
		root.Insert(word, index)
	}
	root.words = words
	return root.searchLongestWord()
}

```



## 2.Review

[Go Pointers: Why I Use Interfaces (in Go)](https://medium.com/@kent.rancourt/go-pointers-why-i-use-interfaces-in-go-338ae0bdc9e4

本文介绍了Go实现构造器的方案。在Java中可以通过构造函数初始化一些数据，在Go中没有提供构造函数，如何做呢？举个例子：
```Java
package io.krancour.widget;

import java.util.UUID;

public class Widget {

	private String id;

	// A constructor that performs some initialization!
	public Widget() {
		id = UUID.randomUUID().toString();
	}

	public String getId() {
		return id;
	}

}
class App {

		public static void main( String[] args ){
			Widget w = new Widget();
			System.out.println(w.getId());
		}

}
```

如果使用Go，可以写如下代码：
```Go
package widgets

type Widget struct {
	id string
}

func (w Widget) ID() string {
	return w.id
}
package main

import (
	"fmt"

	"github.com/krancour/widgets"
)

func main() {
	w := widgets.Widget{}
	fmt.Println(w.ID())
}
```
因为没有构造函数，获取ID为零值
```Go
func NewWidget() Widget {
	return Widget{
		id: uuid.NewV4().String(),
	}
}
```
一般可以通过增加一个函数`NewWidget`来解决，但这样无法强制让使用者使用使用类似构造器方法`NewWidget`

令一种方案是通过减少可见性来解决，如：
```Go
package widgets

import uuid "github.com/satori/go.uuid"

type widget struct {
	id string
}

func NewWidget() widget {
	return widget{
		id: uuid.NewV4().String(),
	}
}

func (w widget) ID() string {
	return w.id
}
```
这个方案，把内部widget对外暴露，godoc也不会生成未暴露的函数类型等文档（Go中，包是基本的重用单元）。那如何处理呢？

```Go
package widgets

import uuid "github.com/satori/go.uuid"

// Widget is a ...
type Widget interface {
	// ID returns a widget's unique identifier
	ID() string
}

type widget struct {
	id string
}

// NewWidget() returns a new Widget
func NewWidget() Widget {
	return widget{
		id: uuid.NewV4().String(),
	}
}

func (w widget) ID() string {
	return w.id
}
```
通过接口，可以看到这个方案实现较为优雅。



## 3.Tips
### GoLang’s go build command

- 执行编译时go build会忽略“_test.go”文件
- 查看帮助 `go help build`
- 常见go build命令

```shell
go build -x sourcefile.go
```
编译时，打印执行的所有命令。



```shell
go build -a sourcefile.go
```

重新构建包即使已经存在。



```shell
go build -n sourcefile.go
```

显示编译执行的命令，但不执行。



```shell
go build -n sourcefile.go
```

打印编译文件时使用的临时目录，不删除。



```shell
go build -ldflags=”-flag” sourcefile.go
```

通过flag可以在编译时插入动态进行到二进制文件中，如传递编译变量。如`go build -ldflags=”-X ‘main.variable=value1’ `





## 4.Share

分享一个技术网站：[FastAPI](https://fastapi.tiangolo.com/)

FastAPI是一个现代的、高性能的构建API的web框架，它基于Python3.6+。熟悉Flask的可以快速上手，相当方便。

- 安装
```shell
pip install fastapi
#Production 环境建议安装
pip install uvicorn
```

- example:`main.py`
```pyhton
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

- 执行

```
uvicorn main:app --reload
```

- 交互式的API文档 ` http://127.0.0.1:8000/docs.`
  ![swaggerui](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)



- 性能测试对比
https://www.techempower.com/benchmarks/#section=test&runid=7464e520-0dc2-473d-bd34-dbdfd7e85911&hw=ph&test=query&l=zijzen-7