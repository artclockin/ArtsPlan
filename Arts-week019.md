# Arts-019

## 1.Algorithm

443. [String Compression](https://leetcode.com/problems/string-compression/)


Given an array of characters, compress it [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm).

The length after compression must always be smaller than or equal to the original array.

Every element of the array should be a **character** (not int) of length 1.

After you are done **modifying the input array [in-place](https://en.wikipedia.org/wiki/In-place_algorithm)**, return the new length of the array.

**Follow up:**
Could you solve it using only O(1) extra space?

**Example 1:**

```
Input:
["a","a","b","b","c","c","c"]

Output:
Return 6, and the first 6 characters of the input array should be: ["a","2","b","2","c","3"]

Explanation:
"aa" is replaced by "a2". "bb" is replaced by "b2". "ccc" is replaced by "c3".
```



**Example 2:**

```
Input:
["a"]

Output:
Return 1, and the first 1 characters of the input array should be: ["a"]

Explanation:
Nothing is replaced.
```



**Example 3:**

```
Input:
["a","b","b","b","b","b","b","b","b","b","b","b","b"]

Output:
Return 4, and the first 4 characters of the input array should be: ["a","b","1","2"].

Explanation:
Since the character "a" does not repeat, it is not compressed. "bbbbbbbbbbbb" is replaced by "b12".
Notice each digit has it's own entry in the array.
```



**Note:**

1. All characters have an ASCII value in `[35, 126]`.

2. `1 <= len(chars) <= 1000`.

  ​     

**My Solution:**

```go
func compress(chars []byte) int {
	if len(chars) < 2 {
		return len(chars)
	}

	//curIdx current deal char
	//insertIdx insert point
	//scanIdx loop for duplicated char
	curIdx, insertIdx, scanIdx := 0, 0 ,1
	duplicatedCount := 0
	for scanIdx <= len(chars) {
		if scanIdx == len(chars)  || chars[curIdx] != chars[scanIdx] {
			chars[insertIdx] = chars[curIdx]
			insertIdx++
			duplicatedCount = scanIdx - curIdx
			if duplicatedCount > 1 {
				s := strconv.Itoa(duplicatedCount)
				for i :=0; i < len(s); i++ {
					chars[insertIdx] = s[i]
					insertIdx++
				}
			}
			curIdx = scanIdx
		}
		scanIdx++
	}
	return insertIdx
}
```



## 2.Review

[Real-time object detection with yolo](https://towardsdatascience.com/real-time-object-detection-with-yolo-9dc039a2596b)

本文简单介绍YOLO算法是什么，有什么优势，科普文。

- YOLO 提供了超快的检测，能够实时的定位目标，每秒可以达到155帧。
- Bounding box和grids，YOLO把图像切分为S*S的网格，每个网格单元预测预定数量的边界框，这些框预测对象的x、y坐标、宽度、长度以及置信得分。
- 通过IOU和非最大抑制算法，清除多余的候选框
- 使用YOLO检测的优势
  - 非常快
  - 语境感知
  - 通用网络，当应用出现意外输入或不熟悉情况时，其故障和失败情况可能性小很多。




## 3.Tips

**Mutex**

互斥锁不区分读和写，

```go
func (m *Mutex) Lock()
func (m *Mutex) Unlock()
```
**特点：**

- 非重入锁：使用 Lock() 加锁后，不能再继续对其加锁，直到利用 Unlock() 解锁后才能再加锁
- 在 Lock() 之前使用 Unlock() 会导致 panic 异常
- 不与特定的 goroutine 相关联，可用一个 goroutine 加锁，另一个 goroutine 对其解锁
- 解锁之前不能再次加锁
- 适用只有一个读或者写的场景

```go
//example 
var mutex sync.Mutex
var process int

func step1() {
	mutex.Lock()
	process += 50
	fmt.Println("step %d/100",process)
	time.Sleep(5 * time.Second)
}

func step2() {
	process += 50
	mutex.Unlock()
	fmt.Println("step %d/100",process)
}

func main() {
	go step1()
	time.Sleep(time.Second)
	go step2()
	time.Sleep(time.Second)
}
```




## 4.Share

分享一个技术文章：[What's New In Go 1.14: Test Cleanup](https://www.gopherguides.com/articles/test-cleanup-in-go-1-14)

文中介绍了Go 1.14新增的`testing.(*T).Cleanup`。并通过访问PG数据库代码为案例对比介绍`Cleanup`使用，并进一步介绍在`t.Parallel`中该Demo即使调用`Cleanup`而失败原因。

Old Test：

```go
func NewTestTaskStore(t *testing.T) (*pg.TaskStore, func()) {
	store := &pg.TaskStore{
		Config: pg.Config{
			Host:     os.Getenv("PG_HOST"),
			Port:     os.Getenv("PG_PORT"),
			Username: "postgres",
			Password: "postgres",
			DBName:   "task_test",
			TLS:      false,
		},
	}

	err := store.Open()
	if err != nil {
		t.Fatal("error opening task store: err:", err)
	}

	return store, func() {
		if err := store.Reset(); err != nil {
			t.Error("unable to truncate tasks: err:", err)
		}
	}
}

func Test_TaskStore_Count(t *testing.T) {
	store, cleanup := NewTestTaskStore(t)
	defer cleanup()

	ctx := context.Background()
	_, err := store.Create(ctx, tasks.Task{
		Name: "Do Something",
	})
	if err != nil {
		t.Fatal("error creating task: err:", err)
	}

	tasks, err := store.All(ctx)
	if err != nil {
		t.Fatal("error fetching all tasks: err:", err)
	}

	exp := 1
	got := len(tasks)

	if exp != got {
		t.Error("unexpected task count returned: got:", got, "exp:", exp)
	}
}
```



使用`Cleanup`:

```go
func NewTestTaskStore(t *testing.T) *pg.TaskStore {
	store := &pg.TaskStore{
		Config: pg.Config{
			Host:     os.Getenv("PG_HOST"),
			Port:     os.Getenv("PG_PORT"),
			Username: "postgres",
			Password: "postgres",
			DBName:   "task_test",
			TLS:      false,
		},
	}

	err = store.Open()
	if err != nil {
		t.Fatal("error opening task store: err:", err)
	}

	t.Cleanup(func() {
		if err := store.Reset(); err != nil {
			t.Error("error resetting:", err)
		}
	})

	return store
}

func Test_TaskStore_Count(t *testing.T) {
	store := NewTestTaskStore(t)

	ctx := context.Background()
	_, err := store.Create(ctx, cleanuptest.Task{
		Name: "Do Something",
	})
	if err != nil {
		t.Fatal("error creating task: err:", err)
	}

	tasks, err := store.All(ctx)
	if err != nil {
		t.Fatal("error fetching all tasks: err:", err)
	}

	exp := 1
	got := len(tasks)

	if exp != got {
		t.Error("unexpected task count returned: got:", got, "exp:", exp)
	}
}
```

