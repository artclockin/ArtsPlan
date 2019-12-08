# Arts-009

## 1.Algorithm

96. [不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)
给定一个整数 n，求以 1 ... n 为节点组成的二叉搜索树有多少种？

    示例:

    输入: 3
    输出: 5
    解释:
    给定 n = 3, 一共有 5 种不同结构的二叉搜索树:

       1         3     3      2      1
        \       /     /      / \      \
         3     2     1      1   3      2
        /     /       \                 \
       2     1         2                 3



**My Solution:**

设S(n)表示n个节点可表示的二叉搜索树，可以得出S(0)=1,S(1)=1

定义F(i,n)  (1<=i<=n)表示已i为根节点，1..i-1为左节点，i+1..n为有节点的二叉搜索树

S(n) =Sum(F*(*i*,*n))=Sum(S(i-1)*S(n-i))

```Go
//递归算法
func numTrees(n int) int {
	if n < 2 {
		return 1
	}
	total := 0
	for i:= 1; i<=n; i++ {
		total += numTrees(i-1)*numTrees(n-i)
	}
	return total;
}
//动态规划
func numTrees(n int) int {
	S := make([]int, n + 1, n + 1)
	S[0],S[1] = 1,1

	for i:= 2; i<=n; i++ {
		for j:=1; j<=i; j++ {
			S[i] += S[j-1] * S[i-j]
		}
	}
	return S[n];
}
```



## 2.Review

[What's the difference between Continuous Integration and Continuous Delivery?](https://hackernoon.com/the-real-difference-between-ci-and-cd-smu363p)
**持续集成和持续交互有什么不同？**

##### 持续集成是一个团队问题
- 需避免的情况是，错误提交成为主分支，这会浪费团队的时间，导致对master分支的不信任。

- 持续集成是为了防止主分支被破坏，这样团队就不会陷入困境。它不是让所有测试一直处于绿色状态，也不是主分支在每次提交时能够部署到生产环境中。

- 持续集成使用一些自动化检查工具，至少保证：
  - 应用能够构建和启动
  - 大多数关键功能应始终正常工作（如用户注册/登录过程和关键业务功能）
  - 应用公共层应该是稳定的，并对这些做了单元测试

- 持续集成不关工具的事，是能够把任务拆分为较小任务，进行快速迭代，并将新代码集成到主分支中，并经常拉取，一般至少每天迭代一次。

- 持续集成关键点，保持构建时间短，最多3-7分钟。这是一种折衷方案，将持续集成范围内运行时间更长或几乎没有价值的测试应移至持续交互步骤，保持上下文切换到最小。

##### 持续交付和部署是工程问题
- 持续交付是指能够随时部署任何版本的代码。连续交付是达到与运行环境的交付物尽可能接近，如jar、war或exe等。准备好交互物是指可以运行所有测试，以确保代码一旦部署便可以正常工作。运行单元测试，集成测试，端到端测试，甚至性能测试。
- 它不需要非常快。30分钟或1小时是可以接受的。
- 持续部署是下一步。您将代码的最新版本和生产就绪版本部署到某个环境。
- 持续交付和持续部署不是团队问题。他们的目的是在执行时间、维护工作和测试套件之间找到适当的平衡，以便能够说“此版本按预期运行”。

##### 有什么大不同？
- 持续集成是一个水平可伸缩性问题。您希望开发人员经常合并其代码，因此集成必须快速。理想情况下，几分钟之内，这样可以让开发人员集中当前的项目上。
- 开发人员越多，越需要在所有活动分支上运行简单检查（构建和测试）。
- 持续交付和部署是垂直可伸缩性问题，需要执行一个相当复杂的操作。
- 一个常见的误解是将持续交互视为持续集成的水平可扩展性问题。
  
##### 结论
- 用于执行CI和CD的工具和原理通常非常相似。但是目标是非常不同的。

- 持续集成是给开发人员的反馈速度与执行的检查（构建和测试）的相关性之间做出的折衷。不妨碍团队进展的代码可以进入主分支。

- 持续交付部署是要进行彻底检查，以发现代码问题。检查的完整性是最重要的因素。通常以测试的代码覆盖率或功能覆盖率来衡量。尽早发现错误可以防止将损坏的代码部署到任何环境，并节省测试团队的宝贵时间。

- 精心设计持续集成持续交付以实现这些目标并保持团队生产力。没有工作流程是完美的。问题会时不时地发生。每次使用它们时，都可以将其作为学习的经验教训来加强您的工作流程。

  


## 3.Tips
##### 使用go mod 安装bee
1. 创建项目文件夹，使用go mod初始化
```bash
$ mkdir webdemo
$ cd webdemo/
$ go mod init web
```
2. 修改bee源

```bash
$ cat go.mod 
module web

replace github.com/beego/bee v1.10.0 => github.com/realguan/bee v1.12.1

go 1.13
```
3. 安装beego和bee

```bash
$ go get -u github.com/beego/bee
$ go get -u github.com/astaxie/beego
```
4. 测试

```bash
$ bee new demo
$ cd src/demo
$ go mod init demo
$ bee run

$ curl http://127.0.0.1:8080 #另启shell测试结果
```



## 4.Share

分享Go wiki：[Go 1.11 Modules](https://github.com/golang/go/wiki/Modules)

Go1.11开始引入go module机制，提供了统一的依赖包管理工具 go mod，依赖包统一收集在 $GOPATH/pkg/mod 中。go module机制实现了 import 路径与项目代码存放路径解耦，使 package 定义导入更加灵活。

Wiki介绍了最近的变化，重点介绍了go module的概念、如何使用、迁移到go module以及资源和FAQ。

国内也有介绍不错的文章，[正在或即将被使用的Go依赖包管理方法：Go Modules，Go 1.13的标准特性](https://mp.weixin.qq.com/s/SGGV3tWEg5AAJ7I_FcK0cg)

