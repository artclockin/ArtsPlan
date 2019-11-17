# Arts-006

## 1.Algorithm
257. [Binary Tree Paths](https://leetcode.com/problems/binary-tree-paths/)
    Given a binary tree, return all root-to-leaf paths.
    
    **Note:** A leaf is a node with no children.
    
    **Example:**
    
    ```Python
    Input:
    
       1
     /   \
    2     3
     \
      5
    
    Output: ["1->2->5", "1->3"]
    
    Explanation: All root-to-leaf paths are: 1->2->5, 1->3
    ```
    



**My Solution1 - 使用map **

```Go
func travelPath(root *TreeNode, path string) []string {
	result := []string{};
	if root == nil {
		return result
	}
	if path == "" {
		path = strconv.Itoa(root.Val)
	} else {
		path = path + "->" + strconv.Itoa(root.Val)
	}
	if root.Left == nil && root.Right == nil {
		result = append(result, path)
	}
	if root.Left != nil {
		for _, val := range travelPath(root.Left, path) {
			result = append(result, val)
		}
	}
	if root.Right != nil {
		for _, val := range travelPath(root.Right, path) {
			result = append(result, val)
		}
	}
	return result
}

func binaryTreePaths(root *TreeNode) []string {
	return travelPath(root, "")
}
```



## 2.Review

[Inside the Language: Sealed Types](https://blogs.oracle.com/javamagazine/inside-the-language-sealed-types)
**How Java is moving toward pattern matching, improved enums, and better switch statements**

本文探讨了enums

```Java
enum Pet {
    CAT,
    DOG
}

$ javap –c Pet.class
Compiled from "Pet.java"
final class Pet extends java.lang.Enum<Pet> {
  public static final Pet CAT;

  public static final Pet DOG;

  …
  // Private constructor
}
```

从而引出Sealed typles。Enums是表示有限数量的实例，而Sealed typles则描述有限数量的类型。本文给出了一个例子

```Java
public abstract sealed class SealedPet permits Cat, Dog {
    protected final String name;
    public abstract void speak();
    public SealedPet(String name) {
        this.name = name;
    }
}

public final class Cat extends SealedPet {
    public Cat(String name) {
        super(name);
    }
    
    public void speak() {
        System.out.println(name +" says Meow");
    }
    
    public void huntMouse() {
        System.out.println(name +" caught a mouse");
    }
}

public final class Dog extends SealedPet {
    public Dog(String name) {
        super(name);
    }
    
    public void speak() {
        System.out.println(name +" says Woof");
    }        

    public void pullSled() {
        System.out.println(name +" pulled the sled");
    }        
}
```

目前sealed，permits还不是keywords。permits表示允许可能的继承类。

sealed思想借鉴了其他语言如C#、Scala等，做了一定扩展。

作者论述了Sealed types与switch expression结合，认为switch expression、sealed type和其他一些正在开发的JVM技术的结合进一步扩展了Java对函数式和面向对象编程风格的支持。

个人感觉越像Scala了，语言特性有了很大进步，看起来给函数式编程范式催生了更多的需求。

目前要试用Sealed types特性需要自己编译jdk源代码。



## 3.Tips
- **执行函数，限定超时时间** 
通过channal实现go对超时函数的限制。
```Go
package main

import (
	"fmt"
	"time"
)

func main() {

	c1 := make(chan string, 1)

	go func() {
		result := LongTask()
		c1 <- result
	}()

	select {
	case res := <-c1:
		fmt.Println(res)
	case <-time.After(2 * time.Second):
		fmt.Println("timeout")
	}

}

func LongTask() string {
	time.Sleep(3 * time.Second)
	return "long task done"
}
```




## 4.Share

[在Python中制作个性化词云图](https://www.cnblogs.com/feffery/p/11842798.html) 

文章介绍了采用wordcloud包如何生成词云，并自定义地图蒙版，调整参数效果。同时介绍了stylecloud包绘制词云方法。

![](img/2019-11-17-22-38-58.png)
