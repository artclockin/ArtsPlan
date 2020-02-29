# Arts-021

## 1.Algorithm
#### [88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

给定两个有序整数数组 *nums1* 和 *nums2*，将 *nums2* 合并到 *nums1* 中*，*使得 *num1* 成为一个有序数组。

**说明:**

- 初始化 *nums1* 和 *nums2* 的元素数量分别为 *m* 和 *n*。
- 你可以假设 *nums1* 有足够的空间（空间大小大于或等于 *m + n*）来保存 *nums2* 中的元素。

**示例:**

```python
输入:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

输出: [1,2,2,3,5,6]
```



**My Solution:**


```Go
/**
1.nums1 add nums2 from m+1, sort m+n
2.compare and insert into nums1 from start
3.comprare and insert nums1 from end
4.归并数组
 */  
func merge(nums1 []int, m int, nums2 []int, n int)  {
	if m == 0 {
		for i := 0; i < n; i++ {
			nums1[i] = nums2[i]
		}
		return
	}
	//from end to compare and move
	current := m + n - 1
	i := m - 1
	j := n - 1
	for i >= 0 && j >= 0 {
		if nums1[i] > nums2[j] {
			nums1[current] = nums1[i]
			i--
		} else {
			nums1[current] = nums2[j]
			j--
		}
		current--
	}
    
	for j >=0 {
		nums1[current] = nums2[j]
		current--
		j--
	}
}
```



## 2.Review

[5 ways to write a singleton and why you shouldn’t !](https://medium.com/@sinethneranjana/5-ways-to-write-a-singleton-and-why-you-shouldnt-1cf078562376)
**本文介绍了Java实现单例的5种方法，并介绍了单例的缺点**
单例是一种常见的设计模式，它确保系统只存在一个实例，实现单例有两种选项：

- 提前加载
- 延迟加载

延迟加载会增加较大开销，所以只有当有一个非常大的对象或一个很重的构造代码，并且还拥有其他可访问的静态方法或字段时才使用它，这些方法或字段可能在需要实例之前使用。否则选择提前加载是个不错的选择。

### 常见加载模式

- 提前加载（推荐）
  ```java
  public class Singleton {
      private static final Singleton INSTANCE = new Singleton(); /* Static initialization is also doable if needed.
      //static {
      //   private static final Singleton INSTANCE = new Singleton();
      //}
  */
      private Singleton() {}

      public static Singleton getInstance() {
          return INSTANCE;
      }
  }
  ```

- 单线程单例（非线程安全）

  ```java
  public final class Singleton {
      private static Singleton singleton= null;  
      private Singleton (){}
      public static Singleton getInstance() {
         if(singleton == null) {
             singleton = new Singleton();
         }  
         return singleton;
      }
  }
  ```

  

- 严格同步,效率低

  ```java
  public final class Singleton {
      private static Singleton singleton= null;
      private Singleton (){}
      public static synchronized Singleton getInstance() {
          if(singleton != null) {
              singleton = new Singleton();
          }
      }
  }
  ```

  

- 两次检查

  ```java
  public final class Singleton {
      // Pay attention to volatile
      private volatile static Singleton uniqueInstance;
     
      private Singleton() {}
      public static Singleton getInstance(){
          if (uniqueInstance==null) {
              synchronized(Singleton.class){
                  if (uniqueInstance==null) {
                      uniqueInstance=new Singleton();
                  }
              }
          }
          return uniqueInstance;
      }
  }
  ```

  

- 静态对象按需加载

  ```java
  public class Singleton{
      private Singleton() {}
      private static class LazyHolder {
          static final Singleton INSTANCE = new Singleton();
      }
      public static Singleton getInstance() {
          return LazyHolder.INSTANCE;
      }
  }
  ```

  

- 基于Enum

  ```java
  public enum Singleton {
      SINGLETON;
      public void anotherMethodOfTheSingleton(){  
      }
  }
  ```

  

  传统单例一个问题是，一旦单例实现了序列号接口，它们就不再是单例，因为readObject()方法总返回一个新实例。

  基于枚举的单例实现简洁，并自然地提供了序列化机制。 



### 单例缺点

- 放弃了可测试性。`getInstance()`方法是全局可见的，这意味着你通常从类中访问它，而不是依赖于可以可以模拟的接口。这就是为什么当测试方法或类时不可能替换它的原因。
- 紧耦合。单例提供了全局的访问状态，单例成为所有其他代码直接或间接依赖的重心。所以在单例中做一个小小的改变就能让系统崩溃。
- 单例一直携带了状态。单例依赖于一个静态变量提供的实例，避免使用静态变量可以防止状态的转移，这是单例无法做到的。



## 3.Tips

### Go package下含多个main函数文件

```shell
#在package定义之前加上
// +build ignore

#并且后面至少一个空行
```



### gojsonq包

```go
func main() {
	const data = `{"cities":[
					{"city":"dhaka","id":1},
					{"city":"sh","id":2}],
					"type":"weekly",
					"temperatures":[30,39.9,35.4,33.5,31.6,33.2,30.7]}`
	jsonq := gojsonq.New().FromString(data)
	//支持汇总
	avg := jsonq.From("temperatures").Avg()
	fmt.Printf("Average temperature: %.2f\n", avg) 
	//支持读取数组下标
	jsonq.Reset()
	fmt.Printf("Monday temperature: %.2f\n", jsonq.Find("temperatures.[0]"))

	//支持sql
	jsonq.Reset()
	cities := jsonq.From("cities").Select("city","id").Where("id",">",1).Get()
	city, _ := json.MarshalIndent(cities, "", "  ")
	println(string(city));
}

```



## 4.Share

[整洁架构（Clean Architecture）的Go微服务: 设计原则](https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651438689&idx=3&sn=8feb96188deb104d0d42058c1eef1df4&chksm=80bb6093b7cce985297268ed454a70fcaa73400b2903070f4dd42f614de8c1e9d07708d12355&scene=21#wechat_redirect)

作者介绍了在进行编写go微服务遵循的一些原则：

- 基于接口编程：基于接口的编程的关键是将接口作为参数传递给函数，并返回接口而不是具体类型。

- 用工厂方法模式（factory method pattern）通过依赖注入（Dependency Injection）创建具体类型.

- 建立正确的依赖关系

  - 程序中的各层或组件都有自己的单独的包。接口在顶级包中定义，具体类型隐藏在子包中。
  - 不同层之间仅依赖于接口而不依赖于具体类型
  - 从顶层向下的依赖层次是：“用例”，“数据服务”和“模型”。

- 开闭原则


本文进一步介绍遵循该原则带来的好处并举案例说明，最后分析了设计的成本。

