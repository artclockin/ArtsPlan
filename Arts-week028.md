# Arts-028

## 1.Algorithm

#### [466. 统计重复个数](https://leetcode-cn.com/problems/count-the-repetitions/)

由 n 个连接的字符串 s 组成字符串 S，记作 `S = [s,n]`。例如，`["abc",3]`=“abcabcabc”。

如果我们可以从 s2 中删除某些字符使其变为 s1，则称字符串 s1 可以从字符串 s2 获得。例如，根据定义，"abc" 可以从 “abdbec” 获得，但不能从 “acbbe” 获得。

现在给你两个非空字符串 s1 和 s2（每个最多 100 个字符长）和两个整数 0 ≤ n1 ≤ 106 和 1 ≤ n2 ≤ 106。现在考虑字符串 S1 和 S2，其中 `S1=[s1,n1]` 、`S2=[s2,n2]` 。

请你找出一个可以满足使`[S2,M]` 从 `S1` 获得的最大整数 M 。

 

**示例：**

```python
输入：
s1 ="acb",n1 = 4
s2 ="ab",n2 = 2

返回：
2
```



**My Solution:**

```go
 type Pair struct {
	c1 int
	c2 int
}

func getMaxRepetitions(s1 string, n1 int, s2 string, n2 int) int {
	if n1 <= 0 {
		return 0;
	}

	lenS2 := len(s2)

	var s1cnt, s2cnt, index int;
	recall := make(map[int]Pair)
	var preLoop, inLoop Pair

	// find cycle struct
	for {
		s1cnt++
		for i:=0; i < len(s1); i++ {
			if s1[i] == s2[index] {
				index++
				if index == lenS2 {
					s2cnt++
					index = 0
				}
			}
		}

		// s1 loop done
		if s1cnt == n1 {
			return s2cnt / n2;
		}

		if _preLoop, ok := recall[index]; ok {
			preLoop = _preLoop
			inLoop = Pair{s1cnt - preLoop.c1, s2cnt - preLoop.c2}
			break
		} else {
			recall[index] = Pair{s1cnt, s2cnt}
		}
	}

	ans := preLoop.c2 + (n1 - preLoop.c1) / inLoop.c1 * inLoop.c2;
	rest := (n1 - preLoop.c1) % inLoop.c1

	for c :=0; c < rest; c++ {
		for i:=0; i < len(s1); i++ {
			if s1[i] == s2[index] {
				index++
				if index == lenS2 {
					ans++
					index = 0
				}
			}
		}
	}
	return ans / n2;
}
```



## 2.Review

[Best Practices for Versioning REST APIs](https://medium.com/better-programming/best-practices-for-versioning-an-api-for-rest-apis-530a9398f311)

API版本控制不应该事后才想到这样做，而应该是API设计的首要部分。

基本原则：为了管理复杂度，往往需要对API进行版本管理。API版本控制使更改时更快地迭代。

如何进行版本化：

- 路由版本控制 (如. `/v1/:foo/:bar/:baz`)

  - 清晰、简洁

  - 增加了API路由长度

- Header头 (如. `Accept-Version: v2`)
  
- 容易实现
  - 必须使用自定义表头，服务器也需要解析该header
  
- 查询字符串 (如. `/:foo/:bar/:baz?version=v2`)

  - 最差实践



## 3.Tips

#### Fuser 使用

- 安装
    ```shell
    // install
    yum install psmisc
    ```
- 检查文件夹使用情况

  ```shell
  fuser -cu /data/ 或者
  fuser -mu /data/
  
  显示结果中如 123rce(root)，进程号+访问类型+User
  访问类型：
  c 当前目录。
  e 正在运行的可执行文件。
  f 打开的文件。
  F 打开的写入文件。
  r 根目录。
  m mmap的文件或共享库
  ```

- 查看文件被使用

  ```shell
  fuser -f /tmp/abc.log
  ```

- 检查端口使用

  ```shell
  fuser -vn tcp 80
  ```

 

## 4.Share

[Go语言的GPM调度器是什么？](https://mp.weixin.qq.com/s/NFfhKQgcM3qrwAD5yYy-XQ)

非常形象化介绍了CSP模型在Go中实现-GPM调度模型

GPM代表了三个角色，分别是Goroutine、Processor、Machine。

- Goroutine：go关键字创建的执行体，它对应一个结构体g，结构体里保存了goroutine的堆栈信息。Goroutine由一个名为`runtime.go`的结构体表示。Goroutine调度相关的数据存储在sched，在协程切换、恢复上下文的时候用到。

- Machine：表示操作系统的线程

  M里面存了两个比较重要的东西，一个是g0，一个是curg。

  - g0：会深度参与运行时的调度过程，比如goroutine的创建、内存分配等
  - curg：代表当前正在线程上执行的goroutine。

- Processor：表示处理器，有了它才能建立G、M的联系。结构体P中存储了性能追踪、垃圾回收、计时器等相关的字段外，还存储了处理器的待运行队列，队列中存储的是待执行的Goroutine列表。

三者关系：

- 默认启动四个线程四个处理器，然后互相绑定。
- 一个Goroutine结构体被创建，在进行函数体地址、参数起始地址、参数长度等信息以及调度相关属性更新之后，它就要进到一个处理器的队列等待发车。轮流往其他P里面放。假如有很多G，都塞满了怎么办呢？那就不把G塞到处理器的私有队列里了，而是把它塞到全局队列里（候车大厅）。
- M首先去处理器的私有队列里取G执行，如果取完的话就去全局队列取，如果全局队列里也没有的话，就去其他处理器队列里偷，如果哪里都没找到要执行的G呢？那M就会因为太失望和P断开关系，然后去睡觉（idle）了。
- 如果G进行了系统调用syscall，M也会跟着进入系统调用状态，P不会傻傻的等待G和M系统调用完成，而会去找其他比较闲的M执行其他的G。当G完成了系统调用，因为要继续往下执行，所以必须要再找一个空闲的处理器发车。如果没有空闲的处理器了，那就只能把G放回全局队列当中等待分配。