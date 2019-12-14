# Arts-010

## 1.Algorithm

95. [不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

给定一个整数 n，生成所有由 1 ... n 为节点所组成的二叉搜索树。

示例:

输入: 3
  输出:
  [
    [1,null,3,2],
    [3,2,null,1],
  [3,1,null,null,2],
    [2,1,3],
    [1,null,2,null,3]
  ]
  解释:
  以上的输出对应以下 5 种不同结构的二叉搜索树：

  ```Python
   1         3     3      2      1
      \       /     /      / \      \
       3     2     1      1   3      2
      /     /       \                 \
     2     1         2                 3
  ```

**My Solution:**

```Go
func generateTrees(n int) []*TreeNode {
	if n == 0 {
		return []*TreeNode{}
	}

	return createTrees(1,n)
}

func createTrees(start int, end int) []*TreeNode {
	ret := []*TreeNode{}
	if start > end {
		ret = append(ret, nil)
		return ret
	}

	for i := start; i <= end; i++ {
		lt := createTrees(start, i-1)
		rt := createTrees(i+1, end)
		for _, l := range lt {
			for _, r := range rt {
				node := &TreeNode{
					Val: i,
					Left: l,
					Right:r,
				}
				ret = append(ret, node)
			}
		}
	}
	return ret
}
```



## 2.Review

[The Complete Guide to JavaScript Classes](https://dmitripavlutin.com/javascript-classes-complete-guide)

本文介绍了JavaScript的类语法，并简单对比了类语法和原型继承语法。类语法对熟悉Java等面向对象语言来说非常熟悉，JavaScript在ES2015中引入类语法，包括了构造器、字段、类字段、私有、公有、静态属性和方法以及继承等。文中给了大量例子。

##### 类定义
```javascript
const UserClass = class {
  // The body of class
};

//a named export:
export class User {
  // The body of class
}
```

##### 构造器
```javascript
class User {
  constructor(name) {
    name; // => 'Jon Snow'
    this.name = name;
  }
}
const user = new User('Jon Snow');
```

##### 字段和方法

私有用#开头，静态用static开头

```javascript
class User {
  static #takenNames = [];

  static isNameTaken(name) {
    return User.#takenNames.includes(name);
  }
  
 #nameValue;

  get name() {
    return this.#nameValue;
  }

  set name(name) {
    if (name === '') {
      throw new Error(`name field of User cannot be empty`);
    }
    this.#nameValue = name;
  }

  constructor(name) {
    this.name = name;
    User.#takenNames.push(name);
  }
}

const user = new User('Jon Snow');
User.isNameTaken('Jon Snow');   // => true
User.isNameTaken('Arya Stark'); // => false
user.name; // The getter is invoked, => 'Jon Snow'
user.name = 'Jon White'; // The setter is invoked

user.name = ''; // The setter throws an Error
```

##### 继承
```javascript
class User {
  name;

  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class ContentWriter extends User {
  posts = [];

  constructor(name, posts) {
    super(name);
    this.posts = posts;
  }

  getName() {
    const name = super.getName();
    if (name === '') {
      return 'Unknwon';
    }
    return name;
  }
}

const writer = new ContentWriter('', ['Why I like JS']);
writer.getName(); // => 'Unknwon'
```

##### 类型检查
```javascript
class User {
  name;

  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class ContentWriter extends User {
  posts = [];

  constructor(name, posts) {
    super(name);
    this.posts = posts;
  }
}

const writer = new ContentWriter('John Smith', ['Why I like JS']);
writer instanceof ContentWriter; // => true
writer.constructor === ContentWriter; // => true
writer.constructor === User;          // => false
```
可以看出Javascript可以用熟悉的类语法写程序，对熟悉Java、C#的人来说更容易上手了，也少了很多黑魔法。



  


## 3.Tips
##### Go Printf
```Go
	fmt.Printf("%t\n", true) //true
	n := 17
	fmt.Printf("%T\n", n) //int
	fmt.Printf("%v\n", n) //17
	fmt.Printf("%b\n", n) //10001
	fmt.Printf("%d\n", n) //17
	fmt.Printf("%x\n", n) //11
	fmt.Printf("%#x\n", n) //0x11
	fmt.Printf("%#v\n", n) //17
	fmt.Printf("%05d\n", n) //00017

	s := "test,测试"
	fmt.Printf("%s\n", s) //test,测试
	fmt.Printf("%v\n", s) //test,测试
	fmt.Printf("%#v\n", s) //"test,测试"
	fmt.Printf("%010s\n", s) //000test,测试

	st  := struct {
		name string
		age int
	}{"张三",10}
	fmt.Printf("%v\n", st) //{张三 10}
	fmt.Printf("%+v\n", st) //{name:张三 age:10}
	fmt.Printf("%#v\n", st) //struct { name string; age int }{name:"张三", age:10}

	fmt.Printf("%p\n", &st) //例如：0xc00009a000
	fmt.Printf("%#p\n", &st) //例如：c00009a000
```



## 4.Share

[通过 sync.Once 学习到 Go 的内存模型](https://juejin.im/post/5df0a5866fb9a0160823723c)

文章从Go单例模式采用sync.Once实现谈起，深入其实现源码
```Go
package sync

import (
	"sync/atomic"
)

// Once is an object that will perform exactly one action.
type Once struct {
	// done indicates whether the action has been performed.
	// It is first in the struct because it is used in the hot path.
	// The hot path is inlined at every call site.
	// Placing done first allows more compact instructions on some architectures (amd64/x86),
	// and fewer instructions (to calculate offset) on other architectures.
	done uint32
	m    Mutex
}

// Do calls the function f if and only if Do is being called for the
// first time for this instance of Once. In other words, given
// 	var once Once
// if once.Do(f) is called multiple times, only the first call will invoke f,
// even if f has a different value in each invocation. A new instance of
// Once is required for each function to execute.
//
// Do is intended for initialization that must be run exactly once. Since f
// is niladic, it may be necessary to use a function literal to capture the
// arguments to a function to be invoked by Do:
// 	config.once.Do(func() { config.init(filename) })
//
// Because no call to Do returns until the one call to f returns, if f causes
// Do to be called, it will deadlock.
//
// If f panics, Do considers it to have returned; future calls of Do return
// without calling f.
//
func (o *Once) Do(f func()) {
	// Note: Here is an incorrect implementation of Do:
	//
	//	if atomic.CompareAndSwapUint32(&o.done, 0, 1) {
	//		f()
	//	}
	//
	// Do guarantees that when it returns, f has finished.
	// This implementation would not implement that guarantee:
	// given two simultaneous calls, the winner of the cas would
	// call f, and the second would return immediately, without
	// waiting for the first's call to f to complete.
	// This is why the slow path falls back to a mutex, and why
	// the atomic.StoreUint32 must be delayed until after f returns.

	if atomic.LoadUint32(&o.done) == 0 {
		// Outlined slow-path to allow inlining of the fast-path.
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}

```

作者对实现关键点进行解读如为什么需要doSlow，为什么已经使用lock还需要atomic.StoreUnit32等，很不错。
