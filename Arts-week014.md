# Arts-014

## 1.Algorithm

48. [Rotate Image](https://leetcode.com/problems/rotate-image/)

You are given an *n* x *n* 2D matrix representing an image.

Rotate the image by 90 degrees (clockwise).

**Note:**

You have to rotate the image [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm), which means you have to modify the input 2D matrix directly. **DO NOT** allocate another 2D matrix and do the rotation.

**Example 1:**

```
Given input matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],

rotate the input matrix in-place such that it becomes:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```

**Example 2:**

```
Given input matrix =
[
  [ 5, 1, 9,11],
  [ 2, 4, 8,10],
  [13, 3, 6, 7],
  [15,14,12,16]
], 

rotate the input matrix in-place such that it becomes:
[
  [15,13, 2, 5],
  [14, 3, 4, 1],
  [12, 6, 8, 9],
  [16, 7,10,11]
]
```



**My Solution:**

```Go
//采用坐标位置旋转
func exchange(matrix [][]int, x1 int, ubound int, doublemid int) {
	for y1:=0; y1< ubound; y1++ {
		x2,y2 := y1, doublemid -x1
		x3,y3 := doublemid -x1,doublemid -y1
		x4,y4 := doublemid -y1,x1
		matrix[x2][y2],matrix[x3][y3],matrix[x4][y4],matrix[x1][y1] = matrix[x1][y1], matrix[x2][y2],matrix[x3][y3],matrix[x4][y4]
	}
}

func rotate(matrix [][]int)  {
	//(mid,mid) point of matrix
	mid := float32(len(matrix)-1)/2
	ubound := int(mid)
	bOdd := true
	if mid - float32(ubound) > 0.1 {
		ubound++
		bOdd = false
	}

	doublemid := int(2*mid)

	if bOdd {
		for x1:=0;x1 <= ubound;x1++{
			exchange(matrix, x1, ubound, doublemid)
		}
	} else {
		for x1:=0;x1 < ubound;x1++{
			exchange(matrix, x1, ubound, doublemid)
		}
	}
}

```



## 2.Review

[Java tuple – Working with tuples in Java](https://howtodoinjava.com/java/basics/java-tuples/)
使用第三方库javatuples，可以在java环境方便使用tuple功能。

### 什么是tuple
通常一组数据表达事物的一组属性
```Java
["Java", 1.8, "Windows"]
["Alex", 32, "New York", true]
[3, "Alexa", "howtodoinjava.com", 37000]
```

### Tuple in Java
Java未提供内建数据结构支持tuple。List/Array与Tuple对比：
- 元组是可以包含异构数据的对象。列表设计用于存储单一类型的元素。
- 元组被认为是最快的，它们消耗的内存最少。
- 数组和列表是可变的，但元组是不可变的。
- 元组的大小也是固定的。
- 如果数据集只分配一次，并且其值不应再次更改，则需要一个元组。

### javatuples maven库
```
<dependency>
    <groupId>org.javatuples</groupId>
    <artifactId>javatuples</artifactId>
    <version>1.2</version>
</dependency>
```
javatuples特性：
- 类型安全
- 不可变
- 可迭代
- 可序列化
- 可比较（需实现Comparable接口）
- 实现equals和hashCode以及toString方法
- 提供单个值（Unit）至10个值（Decade）的不同tuple，其中Pair使用最广。

### 常见使用方法
- 工厂方法如：`Pair<String, Integer> pair = Pair.with("Sajal", 12);`
- 构造器如：`Pair<String, Integer> person = new Pair<String, Integer>("Sajal", 12);`
- 通过集合或迭代器创建，如
`Quartet<String, String, String, String> quartet = Quartet.fromCollection(listOf4Names);
Pair<String, String> pair1 = Pair.fromIterable(listOf4Names, 2);`
- 获取值，如：`pair.getValue0();pair.getValue(0));`
- 设置值，如：`Pair<String, Integer> modifiedPair = pair.setAt0("Kajal")`
- 转为List/Array，如`List<Object> quartletList = quartet1.toList();`
- 其他如，`contains,containsAll,indexOf,lastIndexOf`等




## 3.Tips
**go-bindata**

Go 可以把静态资源转为二进制嵌入到执行文件中，这样可以生成一个单一的可执行文件。嵌入工具go-binddata有丰富API，可以压缩存储。

```shell
#install
$ go get -u github.com/jteeuwen/go-bindata/...

$ go-bindata --help
Usage: go-bindata [options] <input directories>

  -debug
    	Do not embed the assets, but provide the embedding API. Contents will still be loaded from disk.
  -dev
    	Similar to debug, but does not emit absolute paths. Expects a rootDir variable to already exist in the generated code's package.
  -ignore value
    	Regex pattern to ignore
  -mode uint
    	Optional file mode override for all files.
  -modtime int
    	Optional modification unix timestamp override for all files.
  -nocompress
    	Assets will *not* be GZIP compressed when this flag is specified.
  -nomemcopy
    	Use a .rodata hack to get rid of unnecessary memcopies. Refer to the documentation to see what implications this carries.
  -nometadata
    	Assets will not preserve size, mode, and modtime info.
  -o string
    	Optional name of the output file to be generated. (default "./bindata.go")
  -pkg string
    	Package name to use in the generated code. (default "main")
  -prefix string
    	Optional path prefix to strip off asset names.
  -tags string
    	Optional set of build tags to include.
  -version
    	Displays version information.

```


譬如将assets目录下文件

```shell
$ go-bindata -o assets/asset.go -pkg=assets assets/...

-o: 表示输出的文件
-pkg： 表示输出的包名
assets/...: 后面跟需要打包的文件夹，三个点表示包括所有子目录
```



## 4.Share

[100 行写一个 go 的协程池 (任务池)](https://studygolang.com/articles/25781)
本文通过构建线程池，限制goroutine数量，重用goroutine，达到更好的效果。需要解决的主要问题：
- go outine数量限制，通过创建goroutine  pool进行管理
- 任务执行。任务需要任务池来放入goroutine。
- goroutine执行完后可以用来执行其他任务。通过构建生产消费者模型解决。

### 任务的定义
```Go
type Task struct {
    Handler func(v ...interface{}) //执行函数及参数
    Params  []interface{} //参数值
}
```

### 任务池定义
```Go
type Pool struct {
    capacity       uint64
    runningWorkers uint64
    state          int64 //任务池状态
    taskC          chan *Task //任务队列
    closeC         chan bool //关闭任务池
    PanicHandler   func(interface{})
}
```
### worker启动
```Go
func (p *Pool) run() {
    p.incRunning()

    go func() {
        defer func() {
            p.decRunning()
            if r := recover(); r != nil { // 恢复 panic
                if p.PanicHandler != nil { // 如果设置了 PanicHandler, 调用
                    p.PanicHandler(r)
                } else { // 默认处理
                    log.Printf("Worker panic: %s\n", r)
                }
            }
        }()

        for {
            select {
            case task, ok := <-p.taskC:
                if !ok {
                    return
                }
                task.Handler(task.Params...)
            case <-p.closeC:
                return
            }
        }
    }()
}
```

### 生产任务
```Go
func (p *Pool) Put(task *Task) {
    if p.GetRunningWorkers() < p.GetCap() { 
        // 创建启动一个 worker
        p.run()
    }
    // 将任务推入队列, 等待消费
    p.taskC <- task
}
```

### 使用线程池Demo
```Go
func main() {
    // 创建任务池
    pool, err := NewPool(10)
    if err != nil {
        panic(err)
    }

    for i := 0; i < 20; i++ {
        // 任务放入池中
        pool.Put(&Task{
            Handler: func(v ...interface{}) {
                fmt.Println(v)
            },
            Params: []interface{}{i},
        })
    }

    time.Sleep(1e9) // 等待执行
}
```
文章通过benchmark测试代码说明使用线程池可以大幅减少内存分配并与原生goroutine性能相近。