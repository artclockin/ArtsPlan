# Arts-003

## 1.Algorithm
15. 3Sum(https://leetcode.com/problems/3sum/)

Given an array `nums` of *n* integers, are there elements *a*, *b*, *c* in `nums` such that *a* + *b* + *c* = 0? Find all unique triplets in the array which gives the sum of zero.

**Note:**

The solution set must not contain duplicate triplets.

**Example:**

```
Given array nums = [-1, 0, 1, 2, -1, -4],

A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```



**My Solution:**
Go并发版（leetcode提交会报异常）

```Go
type Job struct {
	pos int
	results chan<- Result
}

type Result struct {
	triples [][]int
}

var worker = runtime.NumCPU()

func addJob(nums []int, jobs chan<- Job, results chan<- Result) {
	var pos int
	for i:=0; i< len(nums) -2; i++ {
		if i > 0 && nums[i] == nums[i-1] {
			continue
		}
		pos = i + 1
		jobs <- Job {pos, results}
	}
	close(jobs)
}

func doJob(jobs <-chan Job, nums []int, dones chan<- struct{}) {
	for job := range jobs {
		job.Do(nums)
	}
	dones <- struct{}{}
}

func awaitForCloseResult(dones <-chan struct{}, results chan Result) (totalTripes [][]int){
	working := worker
	done := false
	for {
		select {
		case result := <-results:
			for _, item := range result.triples {
				totalTripes = append(totalTripes, item)
			}
		case <-dones:
			working -= 1
			if working <= 0 {
				done = true
			}
		default:
			if done {
				return
			}
		}
	}
}

func (j *Job) Do(nums []int) {
	lpos := j.pos
	rpos := len(nums) - 1
	numi := nums[lpos - 1]

	var v int
	result:= [][]int{}
	for {
		if lpos >= rpos {
			break
		}

		v = numi + nums[lpos] + nums[rpos]
		if v < 0 {
			lpos++
		} else if v > 0 {
			rpos--
		} else {
			result = append(result, []int{numi, nums[lpos], nums[rpos]})
			for lpos < rpos && nums[lpos] == nums[lpos + 1] {
				lpos++
			}
			for lpos < rpos && nums[rpos] == nums[rpos - 1]{
				rpos--
			}
			lpos++
			rpos--
		}
	}

	j.results <- Result {result}
}

func min(a, b int) int {
	if a > b {
		return b
	}
	return a
}

func fixOne(nums []int, lpos int, rpos, numi int) [][]int {
	var v int
	result:= [][]int{}
	for {
		if lpos >= rpos {
			break
		}

		v = numi + nums[lpos] + nums[rpos]
		if v < 0 {
			lpos++
		} else if v > 0 {
			rpos--
		} else {
			result = append(result, []int{numi, nums[lpos], nums[rpos]})
			for lpos < rpos && nums[lpos] == nums[lpos + 1] {
				lpos++
			}
			for lpos < rpos && nums[rpos] == nums[rpos - 1]{
				rpos--
			}
			lpos++
			rpos--
		}
	}
	return result
}

func threeSum(nums []int) [][]int {
	sort.Ints(nums)
	total_result:= [][]int{}

	total_pos := len(nums) -1
	if total_pos < 2 {
		return total_result
	}

	jobs := make(chan Job, worker)
	results := make(chan Result, min(1000, len(nums)))
	dones := make(chan struct{}, worker)
	go addJob(nums, jobs, results)
	for i:=0; i<worker; i++ {
		go doJob(jobs, nums, dones)
	}
	total_result = awaitForCloseResult(dones, results)

	return total_result
}
```



## 2.Review

[Composite command pattern](https://gunnarpeipman.com/composite-command-pattern/?utm_source=tuicool&utm_medium=referral)
**组合命令模式**
顾名思义就是把组合模式和命令模式结合起来的一种设计模式。作者给出使用这样模式的一个应用场景（Web上传照片）：

![Composite pattern: Upload photo](https://static.gunnarpeipman.com/wp-content/uploads/2019/10/composite-command-upload-photo.png)

如果使用组合命令模式就可以简单写成

```Java
var command = new UploadPhotoCommand(model);
command.Execute();
```

作者提到，如果执行的业务逻辑复杂，不要使用组合命令模式，建议使用工作流引擎。

下面看下组合命令模式：

![Composite command](https://static.gunnarpeipman.com/wp-content/uploads/2019/10/composite-command.png)

示例代码：

```Java
//定义接口：
public interface ICommand
{
    void Execute();
}

//组合命令基类
public abstract class CompositeCommand : ICommand
{
    private IList<ICommand> _children = new List<ICommand>();
 
    protected void AddChild(ICommand command)
    {
        _children.Add(command);
    }
 
    public void Execute()
    {
        foreach(var child in _children)
        {
            child.Execute();
        }
    }
}

```
使用组合命令模式
```Java
public class ImportInvoicesCommand : CompositeCommand
{
    public ImportInvoicesCommand()
    {
        AddChild(new DownloadInvoicesCommand());
        AddChild(new ImportInvoicesToDatabaseCommand());
        AddChild(new ArchiveInvoicesCommand());
    }
}

public void ImportInvoices()
{
    var command = new ImportInvoicesCommand();
        
    command.Execute();
}
```

作者也提出了本文使用组合命令模式尚未涉及的问题：

- Transactional commands

- Parameterized commands

- Validation before execute

- Reporting errors

  


## 3.Tips
- **Function Currying in Go example**:

```Go
func multiply(base int) func(int) int {
	return func(num int) int {
		return base * num
	}
}
func subtract(base int) func(int) int {
	return func(num int) int {
		return num - base
	}
}

func TestCurry(t *testing.T) {
	input := 20
	muti := multiply(2)  // base data
	sub := subtract(5) // base data

	curry := func (num int ) int {return muti(sub(num))}
	fmt.Printf("mutisub:%v submuti%v\n", muti(sub(input)), curry(input))
}

```

- **Mac中复制路径快捷键。**

快捷键为： option + command + C

选中一个文件后，按下快捷键 option + command + C，然后按下 command + V 或者 `粘贴` 粘贴就可以了。




## 4.Share

[谷歌量子计算登上Nature封面，首次实现量子优越性，里程碑式突破](https://www.jiqizhixin.com/articles/2019-10-24) 

量子理论看过薛定谔的猫后感觉越来越玄乎，我思故我在。
但量子计算机看起来正逐步从0向1构建未来