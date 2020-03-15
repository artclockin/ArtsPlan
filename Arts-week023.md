# Arts-023

## 1.Algorithm

179. [Largest Number](https://leetcode.com/problems/largest-number/)


   Given a list of non negative integers, arrange them such that they form the largest number.

   **Example 1:**

   ```
   Input: [10,2]
   Output: "210"
   ```

   **Example 2:**

   ```python
   Input: [3,30,34,5,9]
   Output: "9534330"
   ```

   **Note:** The result may be very large, so you need to return a string instead of an integer.

   

**My Solution:**

```go
func largestNumber(nums []int) string {
	sort.Slice(nums, func(i, j int) bool {
		s1 := strconv.Itoa(nums[i])
		s2 := strconv.Itoa(nums[j])
		return s1 + s2 > s2 + s1;
	})
	if nums[0] == 0 {
		return "0"
	}
	return strings.Replace(strings.Trim(fmt.Sprint(nums), "[]"), " ", "", -1)
}

//优化版
func largestNumber(nums []int) string {
	strNums := make([]string, len(nums))
	for i, num := range nums {
		strNums[i] = strconv.Itoa(num)
	}
	sort.Slice(strNums, func(i, j int) bool {
		return strNums[i] + strNums[j] > strNums[j] + strNums[i];
	})
	if strNums[0] == "0" {
		return "0"
	}

	return strings.Join(strNums, "")
}
```



## 2.Review

[The Monad Design Pattern in Java](https://medium.com/thg-tech-blog/monad-design-pattern-in-java-3391d4095b3f)

本文将monad理解为一种设计模式，首先将其引入函数式编程范式，然后在Java特定的上下文中对其进行探索。

- monad应用于函数式编程，允许纯函数“产生副作用”。如Haskell的I/O系统通过IO Monad的特性来保持函数纯度。

- 将monad设计模式集成到代码中的一种方法是使用修饰类型包装不纯净或棘手的代码。用修饰类型包装代码块有助于抽象，通过一些工具来将它们组合在一起，并相互连接。

案例：

```java
public Integer divide(Integer a, Integer b) {
    return a/b;
}
public Integer addTen(Integer a) {
    return a+10;
}
```

函数式编程常用组合方法。譬如`addTen(divide(8,2)) = addTen(4) = 14`。这里divide第二个参数如果为0，将会抛异常。因此首选需要把这些函数变成纯函数。

```java
public Optional<Integer> divide(Integer a, Integer b) {
    return (b == 0) ? Optional.empty() : Optional.of(a/b);//Optional 是monad
}
ublic Optional<Integer> addTen(Integer a) {
    return Optional.of(a + 10);
}
```

这两个方法还无法进行组合，需要进一步处理，譬如

```java
public Optional<Integer> divideAndAddTen(Integer a, Integer b) {
    final Optional<Integer> divisionResult = divide(a, b);
    if (divisionResult.isPresent()){
        return addTen(divisionResult.get());
    }
    return Optional.empty();
}
```

这个方法虽然解决问题，但没有使用monad，不是我们寻找的“特殊功能组合”。那monad是什么？

定义：Monad是一个泛型类型构造函数（即JavaType->JavaType），包含两个映射：

- `unit` **mapping: 将给定的类型构造函数应用于Java类型或函数的映射.**
- `flattener`**: 如果一个类型构造函数至少应用了两次，则展开JavaType.**

来看Optional：

```java
unit:: T -> Optional<T>   //Optional.of()
unit:: (X -> Y) -> (Optional<X> -> Optional<Y>)
flattener:: Optional<Optional<T>> -> Optional<T>
```

再看函数调整：

```java
Optional<Integer> divideAndAddTenWithMonad(Integer a, Integer b) {
    return divide(a,b)
          .flatMap(divisionResult -> addTen(divisionResult));
}
```

这里flatMap与我们的一元类型构造函数附带的flatten函数不同，flatten是执行的最后一步。

monad使我们能够在命令式程序中使用纯函数，它将程序的所有副作用推到边缘，并尽可能使用纯函数。这将反过来使代码不易受我们所生活的现实世界的不确定性的影响。






## 3.Tips
#### mac中查看坐标:

command+shift+4  



#### 图片中查看图像某个位置坐标：

打开图片，预览状态下，点击左键，拖动鼠标时显示拖动的右下角位置。




## 4.Share

[Caffeine Cache 进程缓存之王](https://cloud.tencent.com/developer/article/1462616)

Caffeine是使用Java 8对Guava缓存的重写版本，基于LRU算法实现，支持多种缓存过期策略。高性能和多种过期、更新策略是其优点。



#### Example：

```java
 public static void main(String[] args) {
        LoadingCache<String, String> build = CacheBuilder.newBuilder().initialCapacity(1).maximumSize(100).expireAfterWrite(1, TimeUnit.DAYS)
                .build(new CacheLoader<String, String>() {
                 //默认的数据加载实现，当调用get取值的时候，如果key没有对应的值，就调用这个方法进行加载
                    @Override
                    public String load(String key)  {
                        return "";
                    }
                });
    }
 }
```

#### **参数方法**

- initialCapacity(1) 初始缓存长度为1

- maximumSize(100) 最大长度为100

- expireAfterWrite(1, TimeUnit.DAYS) 设置缓存策略在1天未写入过期缓存

#### **过期策略** 

分为两种缓存，一个是有界缓存，一个是无界缓存，无界缓存不需要过期并且没有界限。在有界缓存中提供了三个过期API:

- expireAfterWrite：代表着写了之后多久过期
- expireAfterAccess: 代表着最后一次访问了之后多久过期
- expireAfter:在expireAfter中需要自己实现Expire接口，这个接口支持create,update,以及access了之后多久过期。

#### **更新策略** 

Caffeine提供了refreshAfterWrite()方法来确定写入后多久更新缓存。例如

   ```java
LoadingCache<String, String> build = CacheBuilder.newBuilder().refreshAfterWrite(1, TimeUnit.DAYS)
              .build(new CacheLoader<String, String>() {
                   @Override
                   public String load(String key)  {
                       return "";
                   }
               });
   ```

上面的代码建立一个CacheLodaer来进行刷新,这里是同步进行的, 可以通过buildAsync方法进行异步构建。这里的一天后刷新是指缓存一天后再次访问才会刷新。

#### **填充策略（**Population**）** 

Caffeine 提供了三种填充策略：手动、同步和异步，通过builder模式实现。

#### **驱逐策略（**eviction**）** 

Caffeine提供三类驱逐策略：基于大小（size-based），基于时间（time-based）和基于引用（reference-based）。

### **移除监听器**（**Removal**）

- 驱逐（eviction）：由于满足了某种驱逐策略，后台自动进行的删除操作
- 无效（invalidation）：表示由调用方手动删除缓存
- 移除（removal）：监听驱逐或无效操作的监听器removalListener

### **统计**（**Statistics**）

- Caffeine.recordStats()，您可以打开统计信息收集。Cache.stats() 方法返回提供统计信息的CacheStats。
- hitRate()：返回命中与请求的比率
- hitCount(): 返回命中缓存的总数
- evictionCount()：缓存逐出的数量
- averageLoadPenalty()：加载新值所花费的平均时间

