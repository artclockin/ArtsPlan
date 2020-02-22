# Arts-020

## 1.Algorithm
[189. 旋转数组](https://leetcode-cn.com/problems/rotate-array/)

给定一个数组，将数组中的元素向右移动 *k* 个位置，其中 *k* 是非负数。

**示例 1:**

```python
输入: [1,2,3,4,5,6,7] 和 k = 3s
输出: [5,6,7,1,2,3,4]
解释:
向右旋转 1 步: [7,1,2,3,4,5,6]
向右旋转 2 步: [6,7,1,2,3,4,5]
向右旋转 3 步: [5,6,7,1,2,3,4]
```

**示例 2:**

```python
输入: [-1,-100,3,99] 和 k = 2
输出: [3,99,-1,-100]
解释: 
向右旋转 1 步: [99,-1,-100,3]
向右旋转 2 步: [3,99,-1,-100]
```

**说明:**

- 尽可能想出更多的解决方案，至少有三种不同的方法可以解决这个问题。

- 要求使用空间复杂度为 O(1) 的 **原地** 算法。



**My Solution:**


```Go
  //method 1: rotate k times
func rotateKTimes(nums []int, k int)  {
	var current, previous int
	lenNums := len(nums);
	for i:= 0; i < k; i++ {
		previous = nums[lenNums - 1]
		for j := 0; j < lenNums; j++ {
			current = nums[j]
			nums[j] = previous
			previous = current
		}
	}
}

// method 2: find rotate position and rotate
func rotatePos(nums []int, k int)  {
	k = k % len(nums)
	var current, prev, next, tmp int
	lenNums := len(nums);
	times := 0
	for start := 0; times < lenNums; start++ {
		current = start
		prev = nums[current]
		for {
			next = (current + k) % lenNums
			tmp = nums[next]
			nums[next] =prev
			prev = tmp
			current = next
			times++
			if start == current {
				break
			}
		}
	}
}

// method 3: reverse and rotate
func reverse(nums []int, start int, end int) {
	for start < end {
		nums[start], nums[end] = nums[end], nums[start]
		start++
		end--
	}
}

func rotate(nums []int, k int)  {
	k = k % len(nums)
	//reverse all
	reverse(nums,0 , len(nums)-1)
	//reverse k
	reverse(nums,0,k-1)
	//reverse n-k
	reverse(nums,k, len(nums) -1)
}
```



## 2.Review

[Challenges with distributed systems](https://aws.amazon.com/builders-library/challenges-with-distributed-systems/)
本文介绍AWS如何处理分布式系统产生的复杂的开发和运营问题。重点介绍了硬实时分布式系统遇到的挑战。

### 分布式系统类型
根据分布式系统实施的难易，分为离线分布式系统、软实时分布系统和硬实时分布系统。
- 离线分布式系统。包括批处理系统、大数据分布式集群等。它几乎包括了分布式计算的所有优点（可扩展性和容错能力），几乎没有缺点（复杂故障模式和不确定性）

- 软实时系统。有相对充裕时间窗口执行操作，不断产生或更新数据的系统，如索引生成器，EC2的角色等，操作时间可能数小时，但不会对客户造成不当影响。

- 硬实时系统。通常称为请求/应答服务。难以实现原因是无法预计请求的到达但又必须作出响应，如Web服务器、订单管道、交易、API等。

### 硬实时分布系统
硬实时分布系统允许从一个容错域发送到另一个容错域，发送消息使得情况复杂。通过网络进行请求/回复消息消息收发时，情况正常情况分为8个步骤：
1. POST REQUEST：CLIENT 将请求 MESSAGE 放到 NETWORK 上。
2. DELIVER REQUEST：NETWORK 将 MESSAGE 传送到 SERVER。
3. VALIDATE REQUEST：SERVER 验证 MESSAGE。
4. UPDATE SERVER STATE：如有必要，SERVER 根据 MESSAGE 更新其状态。
5. POST REPLY：SERVER 将回复 REPLY 放到 NETWORK 上。
6. DELIVER REPLY：NETWORK 将 REPLY 传送到 CLIENT。
7. VALIDATE REPLY：CLIENT 验证 REPLY。
8. UPDATE CLIENT STATE：如有必要，CLIENT 根据 REPLY 更新其状态。

CLIENT、SERVER 和 NETWORK 可能会发生故障。代码必须处理提及的任何步骤发生的故障。在常见的工程实现中，这些类型的故障发生在一台计算机上（一个容错域），它们共担命运，这可以大大减少必须处理的不同故障模式。

### 硬实时系统的故障处理模式
必须假定上述每个步骤都可能发生故障，必须确保客户端和服务器上的代码始终能否针对这些故障运行。
1.POST REQUEST 失败：NETWORK 无法传送消息（例如，中间路由器恰好不合时宜地崩溃），或者 SERVER 明确拒绝了该消息。
2.DELIVER REQUEST 失败：NETWORK 已成功将 MESSAGE 传送到 SERVER，但是 SERVER 收到 MESSAGE 后立即崩溃。
3. VALIDATE REQUEST 失败：SERVER 判定 MESSAGE 无效。原因可能各种情况都有。例如，数据包损坏、软件版本不兼容，或者客户端或服务器出现错误。
4.UPDATE SERVER STATE 失败：SERVER 尝试更新其状态，但无法更新。
5.POST REPLY 失败：无论尝试回复成功还是失败，SERVER 都可能无法发布回复。例如，它的网卡可能恰好不合时宜地过热。
6.DELIVER REPLY 失败：即使 NETWORK 在上文的步骤中可以正常运行，NETWORK 仍可能无法像上文描述的那样将 REPLY 传送给 CLIENT。
7.VALIDATE REPLY 失败：CLIENT 判定 REPLY 无效。
8.UPDATE CLIENT STATE 失败：CLIENT 可能会收到消息 REPLY，但是无法更新其自身的状态、无法理解消息（由于不兼容）或由于其他原因而失败。

这些故障模式使得分布式计算变得困难重重，称之为天灾8种模式。普通代码在硬实时分布式系统中会编程15中额外的步骤，发生这种扩展的原因是客户端和服务器之间的每次往返通信在八个不同的点都有可能发生故障。

### 测试硬实时分布式系统
测试状态空间的增加，测试复杂性也成爆炸式增长。

### 处理未知情况
在分布式工程设计中，要想知道为什么事情并非如表现的那样，就需要弄清楚如何处理 UNKNOWN 错误类型。

### 硬实时分布式系统群
可以在多个抽象层进行查看：
1. 个别计算机
2. 计算机组
3. 计算机组群
4. 其他（潜在）

组与组之间8种方式测试均可能失败，组内两服务器8种方式测试也可能失败，由于存在大量的边缘情况，因此测试颇具挑战性。

### 分布式错误通常是潜在的

### 分布式错误的病毒式传播
1. 分布式错误必定涉及网络的使用。
2. 分布式错误更有可能传播到其他计算机（或计算机组）。


### 分布式系统中的问题总结
1. 无法对错误状况进行组合。
2. 任何网络操作的结果都可能是 UNKNOWN。
3. 分布式问题发生在分布式系统的所有逻辑层级，而不仅仅是低层级的物理计算机。
4. 分布式错误通常会在部署到系统后很长时间才出现。
5. 分布式错误可能会蔓延到整个系统。
6. 许多问题都源自联网的物理定律，无法更改。




## 3.Tips

项目在Ubuntu上创建git仓库，在mac中clone后，发现文件权限改变

```shell
$ git diff  src/main/java/com/XXX/CacheKey.java
diff --git a/src/main/java/com/XXX/CacheKey.java b/src/main/java/com/XXX/CacheKey.java
old mode 100644
new mode 100755
```



这是由于unix文件权限模式不同，对git进行设置，忽略文件模式

```shell
#本仓库
git config core.filemode false
#设置全局模式
git config --global core.filemode false
#全局模式 增加到 ~/.gitconfig 
[core]
     filemode = false
```



Python override实现，通过构建一个Namspace，存储函数名称和参数个数构成dict，函数调用时获取具体的函数执行。参考:https://arpitbhayani.me/blogs/function-overloading

```python
# python3
# override_func
from inspect import getfullargspec

class Function(object):
  """Function is a wrap over standard python function.
  """
  def __init__(self, fn):
    self.fn = fn

  def __call__(self, *args, **kwargs):
    """Overriding the __call__ function which makes the
    instance callable.
    """
    # fetching the function to be invoked from the virtual namespace
    # through the arguments.
    fn = Namespace.get_instance().function_map.get(self.key(args))
    if not fn:
      raise Exception("no matching function found.")

    # invoking the wrapped function and returning the value.
    return fn(*args, **kwargs)

  def key(self, args=None):
    """Returns the key that will uniquely identify
    a function (even when it is overloaded).
    """
    # if args not specified, extract the arguments from the
    # function definition
    if args is None:
      args = getfullargspec(self.fn).args

    return tuple([
      self.fn.__module__,
      self.fn.__class__,
      self.fn.__name__,
      len(args or []),
    ])

class Namespace(object):
  """Namespace is the singleton class that is responsible
  for holding all the functions.
  """
  __instance = None

  def __init__(self):
    if self.__instance is None:
      self.function_map = dict()
      Namespace.__instance = self
    else:
      raise Exception("cannot instantiate a virtual Namespace again")

  @staticmethod
  def get_instance():
    if Namespace.__instance is None:
      Namespace()
    return Namespace.__instance

  def register(self, fn):
    """registers the function in the virtual namespace and returns
    an instance of callable Function that wraps the
    function fn.
    """
    func = Function(fn)
    self.function_map[func.key()] = fn
    return func

def overload(fn):
  """overload is the decorator that wraps the function
  and returns a callable object of type Function.
  """
  return Namespace.get_instance().register(fn)



# test.py
from override_func import  overload

@overload
def area(width, height):
  return width * height

@overload
def area(r):
  import math
  return round(math.pi * r ** 2,2)

print(area(3,2))
print(area(5))
```





## 4.Share

[Go - atomic包使用及atomic.Value源码分析](https://studygolang.com/articles/26288)

原子操作指执行过程中不能被中断，由底层硬件支持，锁则由操作系统提供的API实现，通常原子操作性能更高。

Go语言提供原子操作都是非侵入式的，由sync/atomic包提供

atomic包支持6种类型:int32、uint32、int64、uint64、uintptr、unsafe.Pointer

对每一种类型，提供5类原子操作：

- LoadXXX(addr)：原子性的获取`*addr`的值，等价于`return *addr`

- StoreXXX(addr, val)：原子性的将`val`的值保存到`*addr`，等价于`addr = val`

- AddXXX(addr, delta)：原子性将`delta`的值添加到`*addr`并返回新值（`unsafe.Pointer`不支持）,等价于

  ```go
  *addr += delta
  return *addr
  ```

  

- SwapXXX(addr, delta)：原子性将`new`的值保存到`*addr`并返回旧值，等价于：

  ```go
  old = *addr
  *addr = new
  return old
  ```

  

- CompareAndSwapXXX(addr, old, new) bool：原子性比较`*addr`和`old`，如果相同则将`new`赋值给`*addr`，并返回true

  ```go
  if *addr == old {
  	*addr = new
  	return true
  }
  return false
  ```

  

譬如`count++`不是原子操作，它执行三个操作，因此如果多个goroutine同时操作，就会造成安全问题。

```
从内存读取count
cpu更新count=count + 1
写count到内存
```

可修改为

```go
atomic.AddInt32(&count, 1)
// or
for {
		if atomic.CompareAndSwapInt32(&count, count, count + 1) {
			break
		}
}
```

Go1.4引入新类型`Value`,相当于一个容器，用来原子存储任意类型的值。

- func(v *Value) Load() (x interface{}): 读操作，从线程安全的v中读取上一步存放的内容
- func(v *Value) Store(x interface{}): 写操作，将原始的变量x存放在`atomic.Value`类型的v中



文中进一步对atomic.Value源码进行分析，建议看原文。