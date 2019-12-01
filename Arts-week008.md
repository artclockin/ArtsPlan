# Arts-008

## 1.Algorithm
98.[Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)

Given a binary tree, determine if it is a valid binary search tree (BST).

Assume a BST is defined as follows:

- The left subtree of a node contains only nodes with keys **less than** the node's key.
- The right subtree of a node contains only nodes with keys **greater than** the node's key.
- Both the left and right subtrees must also be binary search trees.

 

**Example 1:**

```
    2
   / \
  1   3

Input: [2,1,3]
Output: true
```

**Example 2:**

```
    5
   / \
  1   4
     / \
    3   6

Input: [5,1,4,null,null,3,6]
Output: false
Explanation: The root node's value is 5 but its right child's value is 4.    
```



**My Solution**

```Go
func isValidValue(node *TreeNode, min int, max int) bool {
	if node == nil {
		return true
	}
	if node.Val <= min || node.Val >= max {
		return false
	}
	return isValidValue(node.Left, min, node.Val) && isValidValue(node.Right, node.Val, max)
}

func isValidBST(root *TreeNode) bool {
	const INT_MAX = int(^uint(0) >> 1)
	const INT_MIN = ^INT_MAX
	return isValidValue(root, INT_MIN, INT_MAX)
}
```



## 2.Review

[DISCOVERING LESS-KNOWN POSTGRESQL V12 FEATURES](https://www.cybertec-postgresql.com/en/discovering-less-known-postgresql-12-features/)
**PostgreSQL v12 一些鲜为人知特性**

PosqgreSQL v12.1已经发布，v12带来了许多改变，作者从DB优化、语言功能、运维等方面介绍了v12带来的新特性。

- “自动的”性能提升
  - 自动内联通用表表达式（CTE），支持将 WITH 语句中的查询条件下推到外层SQL中，从而提升 CTE 语句性能。如果希望不这样做（极少情况下如执行计划判断异常等）可以使用“MATERIALIZED”关键词强制先物化。
>  WITH w AS MATERIALIZED ( 
> SELECT * FROM pgbench_accounts 
> ) 
> SELECT * FROM w WHERE aid = 1; 


  - 在SERIALIZABLE隔离模式下时允许并行查询
  - 默认情况下启用即时（JIT）编译，v11的杀手级Feature，当时作为可选项引入，v12改为默认。在非常复杂查询可能会变慢，可以在session、transaction级别上进行禁用，或者调整jitting的阀值(jit_*_cost)。

- 开发者相关特性
	- 支持 SQL/JSON path 特性
	- 分区表支持外键引用，被引用的列必须是分区键一部分，因此可能需要自定义验证触发器，或者定期运行一些FK检查脚本
	- 添加分区内省函数，引入pg_partition_root(), pg_partition_ancestors() and pg_partition_tree()
	- 添加连接参数tcp_user_timeout以控制libpq的tcp超时
	- 在psql的“\h[elp]”输出中显示SQL命令的页面URL
	- VACUUM 新增 INDEX_CLEANUP 选项控制是否回收索引
	- EXPLAIN 新增 SETTINGS 选项显示非默认优化器参数
	- log_statement_sample_rate 参数控制数据库日志中慢SQL百分比

- DBA方面
	-  异常恢复使用最新时间线
	-  并行自动索引重建，REINDEX 命令新增 CONCURRENTLY 选项，操作简单安全
	-  “pg_checksums”实用程序可以启用/禁用脱机集群的页面校验和
	-  pgbench 新增 \gset 命令支持将SQL结果存入变量

		> \set bid random(1, 1 * :scale) 
    > \set tid random(1, 10 * :scale) 
    > \set delta random(-5000, 5000) 
    > BEGIN; 
    > select aid from pgbench_accounts where abalance = 0 order by aid limit 1 \gset 
    > UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid; 
    > SELECT abalance FROM pgbench_accounts WHERE aid = :aid; 
    > UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid; 
    > UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid; 
    > INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta,  CURRENT_TIMESTAMP); 
    > END; 

	-  允许小数方式输入服务器参数，如允许SET work_mem = ‘1.5GB’ 
	-  允许vacuumdb基于wraparound horizon选择表进行vacuum，可采取手动措施以避免有效的停机时间。
	-  recovery.conf文件参数合并到 postgresql.conf/ostgresql.auto.conf

PS.具体可以参考(https://www.postgresql.org/docs/12/release-12.html#id-1.11.6.6.5)



## 3.Tips
- **go module镜像**

    go get 下载文件报错可以使用go module镜像解决下载源问题
    
    ```go
		//七牛云
		go env -w GO111MODULE=on
		go env -w GOPROXY=https://goproxy.cn,direct
		//or 阿里云
		go env -w GO111MODULE=on
		go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
    ```
    
- **Delve:Golang debug tools** 

  - **Install**
  ```Go
  go get -v github.com/go-delve/delve/cmd/dlv
  ```
  
  - **Debug**
  ```Go
  $ dlv debug main.go 
  Type 'help' for list of commands.
  (dlv) break runtime.main
  Breakpoint 1 set at 0x1035683 for runtime.main() /usr/local/go/src/runtime/proc.go:113
  (dlv) c
      > runtime.main() /usr/local/go/src/runtime/proc.go:113 (hits goroutine(1):1 total:1) (PC: 0x1035683)
      Warning: debugging optimized function
      108: 
      109: // Value to use for signal mask for newly created M's.
      110: var initSigmask sigset
      111: 
      112: // The main goroutine.
      => 113: func main() {
      114:         g := getg()
      115: 
      116:         // Racectx of m0->g0 is used only as the parent of the main goroutine.
      117:         // It must not be used for anything else.
      118:         g.m.g0.racectx = 0
  
  (dlv) b main.handle
  Breakpoint 2 set at 0x1357018 for main.handle() ./main.go:19
  (dlv) c
  runing at: 8080
  
  //curl http://127.0.0.1:8080
      > main.handle() ./main.go:19 (hits goroutine(50):1 total:1) (PC: 0x1357018)
          14: 
          15:         fmt.Println("runing at: " + port)
          16:         log.Fatal(http.ListenAndServe(":" + port, nil))
          17: }
          18: 
      =>  19: func handle(w http.ResponseWriter, r *http.Request) {
          20:         hostName, _ := os.Hostname()
          21:         fmt.Fprintf(w, "Host: %s", hostName)
          22: }
  (dlv) p hostName
  "192.168.1.5"
  (dlv) locals
  hostName = "192.168.1.5"
  (dlv) args
  w = net/http.ResponseWriter(*net/http.response) 0xc000089850
  r = ("*net/http.Request")(0xc000132000)
  ```
  也可以attach到go项目进程上
  
  ```Go
  $ dlv attach 40455
  Type 'help' for list of commands.
  (dlv) b main.handle
  Breakpoint 1 set at 0x125ecc3 for main.handle() ./main.go:19
  ```

	delve常见命令：
    - b func：下断点
    - bp：显示断点
    - n：执行到下一行
    - s ：单步执行
    - p：输出变量信息
    - args ：所有的方法参数信息
    - locals ：所有的本地变量


## 4.Share

[Go语言之指针常见问题](https://mp.weixin.qq.com/s/1n_s5EDm5Sl75J4GKVWKOQ) 

在Go语言中，指针其实有下面几种表现形式，第一：指针；第二：接口；第三：slice；第四：map。文中列举不少例子。

- 指针变量作为参数使用，会被复制一份指针变量。接口，slice和map作为参数传递的时候，其实跟使用指针变量是一致的。
- 指针变量最好不要在go语句中使用，协程的执行顺序是不可预期的。
  - 场景一：只是读指针中的数据的话，可以使用指针变量做参数，不建议使用，除非参数的结构体很大，复制一份数据结构的话，非常耗时。
  - 场景二：指针中的数据更改与goroutinue的执行顺序无关的话，可以使用指针变量做参数。
  - 场景三：指针中的数据变化与执行顺序有关系，不能使用指针变量做参数。
- 循环使用slice的时候，更改其中数据的影响：
  - range对元素是复制操作，不会改变原有值
  - 如果range使用下标直接操作原始列表，相当于更改原变量的数值
  - 如果slice元素为指针，则修改会改变原有值
  - 在对range指向的数组nodes做删除操作之后，range中的索引不会更改，从而导致遍历错误。
- 循环使用map的时候，map更新了之后，key的值会跟着更新。

