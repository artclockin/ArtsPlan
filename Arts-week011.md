# Arts-011

## 1.Algorithm

43. [Employee Importance](https://leetcode.com/problems/employee-importance/)

You are given a data structure of employee information, which includes the employee's **unique id**, his **importance value** and his **direct** subordinates' id.

For example, employee 1 is the leader of employee 2, and employee 2 is the leader of employee 3. They have importance value 15, 10 and 5, respectively. Then employee 1 has a data structure like [1, 15, [2]], and employee 2 has [2, 10, [3]], and employee 3 has [3, 5, []]. Note that although employee 3 is also a subordinate of employee 1, the relationship is **not direct**.

Now given the employee information of a company, and an employee id, you need to return the total importance value of this employee and all his subordinates.

**Example 1:**

```
Input: [[1, 5, [2, 3]], [2, 3, []], [3, 3, []]], 1
Output: 11
Explanation:
Employee 1 has importance value 5, and he has two direct subordinates: employee 2 and employee 3. They both have importance value 3. So the total importance value of employee 1 is 5 + 3 + 3 = 11.
```

 

**Note:**

1. One employee has at most one **direct** leader and may have several subordinates.
2. The maximum number of employees won't exceed 2000.

 

**My Solution:**

```Python
#递归遍历
class Solution(object):
    def getImportance(self, employees, id):
        """
        :type employees: List[Employee]
        :type id: int
        :rtype: int
        """
        self.employees_map = {}
        for employee in employees:
            self.employees_map[employee.id] = employee
        return self.travel_deepth(self.employees_map.get(id))
    
    def travel_deepth(self, employee):
        sum = employee.importance
        for sub_employee_id in employee.subordinates:
            sum += self.travel_deepth(self.employees_map[sub_employee_id])
        return sum

#广度遍历
class Solution(object):
    def getImportance(self, employees, id):
        """
        :type employees: List[Employee]
        :type id: int
        :rtype: int
        """
        queue = []
        employees_map = {}
        for employee in employees:
            employees_map[employee.id] = employee
            if employee.id == id:
                queue.append(employee)
        
        sum = 0
        while queue:
            employee =  queue.pop(0)
            sum += employee.importance
            for sub_employee in employee.subordinates:
                queue.append(employees_map[sub_employee])
                
        return sum
      
```



## 2.Review

[11 Top VueJS Developer Tools for 2020](https://blog.bitsrc.io/top-10-vuejs-developer-tools-becd61375447)

VueJs越来越受欢迎，它简单易学，本文介绍了一些优秀的工具，这将帮助您快速构建优秀的应用程序。

- Vue CLI。提供交互式快速开发的项目脚手架，快速构建零配置原型开发，一个运行时依赖，丰富官方插件，一套完全图形化的创建和管理 Vue.js 项目的用户界面。(https://cli.vuejs.org/zh/guide/)

- Nuxt JS。Nuxt JS提供了多种应用创建，如Server-Side Rendered (SSR), Progressive Web Applications (PWA), Single Page Applications (SPA), and Static Sites，并提供了大量模块如Google分析。 

- Bit for Vue。它解决了跨存储库共享和协作单个Vue组件的问题。

- Vue-router。提供丰富的路由功能。(https://router.vuejs.org/zh/guide/#javascript)

- Vuex。专为 Vue.js 应用程序开发的状态管理模式，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。

- Axios。创建和管理Ajax请求的第三方库。

- Vuetify。Vu e + Beautify，提供80多种设计组建，可以快速构建漂亮应用。

- Vue Apollo。以申明式方式使用GraphSQL。

- Mocha。通过Nodejs允许前后端，方便进行异步测试。

- Vue.js DevTools for browsers。方便的调试工具，支持Firefox和Chrome

- Official Vue.js Guide。





## 3.Tips
获取运行程序位置:
```Go
//方法一	
dir, _ := os.Getwd()
println(dir)

//方法二
dir, _ := os.Executable()
path := filepath.Dir(dir)
println(path)
```
如果在IDE环境下，两者输出不同，方法一是go项目文件路径，方法二会是如/private/var/folders/mh/krrg51957cqgl0rhgnwyylvc0000gn/T这样的路径。


## 4.Share

分享一个技术文章：[由浅入深剖析 go channel](https://www.jianshu.com/p/24ede9e90490)

channel 是 goroutine 之间通信的一种方式，可以从一个go routine向另一个go routine发消息。

### channel为引用类型，channel的读写操作：
```Go
//一定要初始化后才能进行读写操作，否则会永久阻塞。
ch := make(chan int)
// write to channel
ch <- x
// read from channel
x <- ch
// another way to read
x = <- ch

//关闭
close(ch)
```

### channel关闭注意事项：
- 关闭一个未初始化(nil) 的 channel 会产生 panic

- 重复关闭同一个 channel 会产生 panic

- 向一个已关闭的 channel 中发送消息会产生 panic

- 从已关闭的 channel 读取消息不会产生 panic，且能读出 channel 中还未被读取的消息，若消息均已读出，则会读到类型的零值。从一个已关闭的 channel 中读取消息永远不会阻塞，并且会返回一个为 false 的 ok-idiom，可以用它来判断 channel 是否关闭

- 关闭 channel 会产生一个广播机制，所有向 channel 读取消息的 goroutine 都会收到消息

### channel 的类型
- 无缓存的 channel。通过无缓存的 channel 进行通信时，接收者收到数据 happens before 发送者 goroutine 唤醒

- 有缓存的 channel。make 函数的第二个参数，该参数为 channel 缓存的容量。有缓存的 channel 类似一个阻塞队列(采用环形数组实现)。

### channel使用
- goroutine 通信
```Go
c := make(chan int)  // Allocate a channel.

// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()

doSomethingForAWhile()
<-c
```
主 goroutine 会阻塞，直到执行 sort 的 goroutine 完成。

- range遍历
channel 会一直从 channel 中读取数据，直到有 goroutine 对 channel 执行 close 操作，循环才会结束。
```Go
// consumer worker
ch := make(chan int, 10)
for x := range ch{
    fmt.Println(x)
}

等价于
for {
    x, ok := <- ch
    if !ok {
        break
    }
    
    fmt.Println(x)
}
```

- 配合 select 
```Go
select {
    case <- ch1:
    ...
    case <- ch2:
    ...
    case ch3 <- 10;
    ...
    default:
    ...
}
```
  - select 可以同时监听多个 channel 的写入或读取
  
  - 执行 select 时，若只有一个 case 通过(不阻塞)，则执行这个 case 块，若有多个 case 通过，则随机挑选一个 case 执行
  - 若所有 case 均阻塞，且定义了 default 模块，则执行 default 模块。若未定义 default 模块，则 select 语句阻塞，直到有 case 被唤醒。
  - 使用 break 会跳出 select 块。

**例：**
**设置超时时间**

```Go
ch := make(chan struct{})

// finish task while send msg to ch
go doTask(ch)

timeout := time.After(5 * time.Second)
select {
    case <- ch:
        fmt.Println("task finished.")
    case <- timeout:
        fmt.Println("task timeout.")
}
```
**quite channel**

```Go
msgCh := make(chan struct{})
quitCh := make(chan struct{})
for {
    select {
    case <- msgCh:
        doWork()
    case <- quitCh:
        finish()
        return
}
```

本文还从源码角度对channel的类结构、make、send、receive以及close过程进行了分析。