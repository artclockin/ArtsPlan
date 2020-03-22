# Arts-024

## 1.Algorithm

945. [使数组唯一的最小增量](https://leetcode-cn.com/problems/minimum-increment-to-make-array-unique/)

给定整数数组 A，每次 *move* 操作将会选择任意 `A[i]`，并将其递增 `1`。

返回使 `A` 中的每个值都是唯一的最少操作次数。

**示例 1:**

```
输入：[1,2,2]
输出：1
解释：经过一次 move 操作，数组将变为 [1, 2, 3]。
```

**示例 2:**

```
输入：[3,2,1,2,1,7]
输出：6
解释：经过 6 次 move 操作，数组将变为 [3, 4, 1, 2, 5, 7]。
可以看出 5 次或 5 次以下的 move 操作是不能让数组的每个值唯一的。
```

**提示：**

1. `0 <= A.length <= 40000`
2. `0 <= A[i] < 40000`



**My Solution:**

```go
func minIncrementForUnique(A []int) int {
	if len(A) == 0 {
		return 0
	}

  //排序后逐项计算差值
	sort.Ints(A) 
	result, curMin := 0, A[0]

	for i := 1; i < len(A); i++ {
		if A[i] <= curMin {
			curMin++
			result += curMin - A[i]
		} else {
			curMin = A[i]
		}
	}
	return result
}
```



## 2.Review

[Ready for changes with Hexagonal Architecture](https://netflixtechblog.com/ready-for-changes-with-hexagonal-architecture-b315ec967749)

本文介绍构建项目初期就要有长远考虑，并从六边形架构设计角度为调整做好准备。

- 高度集成。项目刚开始就需要高度集成，数据可能通过不同协议来自不同系统。
- 数据源可交换。刚开始采用单体应用，这样可以快速开发，随着应用演化，需要对不同领域设置界限，并使专门团队能独立针对领域的服务，要为拆分成不同微服务和数据源做好准备。
- 利用六边形架构思想。 将输入和输出置于设计的边缘。业务逻辑不应该依赖公开的接口，也不应该依赖于从何处获取数据，如数据库、文件等。六边形架构能够将程序核心逻辑和对外专注点分开，这样就可以轻松更改数据源而不会造成重大影响。使用清晰边界的另一个优势是测试策略，大多数测试可以不依赖于易变的协议就可以验证业务逻辑。
- 定义核心概念。
  - 业务逻辑三个主要概念是实体、存储和交互其。
    - 实体是领域对象（例如电影或拍摄地点），它们对其存储位置一无所知。
    - 存储库。获取实体以及创建、修改实体的接口，通过数据源交互返回单个或列表实体。
    - 交互器（Serivice）。用于编排和执行领域操作的类，执行领域操作的复杂业务逻辑和验证逻辑。
  - 业务之外是数据源和传输层
    - 数据源是不同存储实现的适配器，数据源实现存储上定义的方法。
    - 传输层触发交互器执行业务逻辑。微服务常见使用HTTP API层和一组处理请求的控制器，将业务逻辑提取到交互器。交互器不仅可以由控制器触发，还可以由事件，cron作业或命令行触发。
  - 在六角形架构中，所有依赖都指向内部。核心业务层对传输层和数据源一无所知。传输层知道如何使用交互器，数据源知道如何符合存储库接口。
- 交换数据源。通过实现存储库接口就可以快速进行切换。这个体系结构优势能够封装数据源实现细节。
- 测试策略。
  - 要能够快速开发必须具有可靠和快速的测试套件
  - 利用依赖注入并模拟任何类型的存储库交互，进行业务逻辑详细测试。
  - 测试数据源，确定是否可以与其他服务正确集成，是否符合存储库接口，检查出现错误的行为。
  - 集成测试，从传输层/API到交互器（Service）到存储库到数据源等，每个操作有成功方案和失败方案。
  - 不测试存储库，因为它们是数据源实现的简单接口，很少测试实体，因为它们是定义属性的普通对象，测试实体是否具有其他方法。
  - 不去ping依赖的服务，而是依赖合同测试。

- 延迟决策。在数据源交换到不同的微服务方面，关键是延迟一些决定，如如何存储应用程序内部数据，数据存储的类型等。
- 在项目开始时，关于我们正在构建的系统的信息量最少。我们不应该将自己锁定在一个不知情的决定而导致[项目悖论](https://twitter.com/tofo/status/512666251055742977)的体系结构中。



## 3.Tips

整型溢出问题:
```java
public class TestLong {
    private static final DateTimeFormatter timeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
  
    //想当然以为返回类型为long就没问题
    public static long extractTimestamp(int timestamp) {
        return timestamp*1000;
    }
  
    public static void main(String[] args) {
        int start = 1577808100;
        System.out.println(Instant.ofEpochMilli(extractTimestamp(start)).atZone(ZoneId.systemDefault()).format(timeFormatter));
    }
}

//1970-01-19 07:58:22
```
修正：

```java
public static long extractTimestamp(int timestamp) {
        return timestamp*1000L;
}
// 2020-01-01 00:01:40    
```




## 4.Share

[Go项目简单接入travis ci](https://juejin.im/post/5e7592c0518825494a3fadd9)

本文介绍了travis应用与单元测试和github release，如何编写Makefile实现go语言项目的自动化测试。

### 单元测试

只需要在`tracis`中执行`go test -v`即可。

```yml
# https://github.com/flytam/filenamify/blob/master/.travis.yml
language: go

go:
    - 1.13.x
env:
    - GO111MODULE=on
script: go test -v

```

然后给在项目中加上构建状态图标。



### 发布github release

1. 新建`.travis.yml`文件，填入基本的 Go 配置环境

   ```yaml
   language: go
   
   go:
       - 1.13.x
   env:
       - GO111MODULE=on # 启用Go mod
   install:
       - go get -v
   
   ```

   

2. 编写Makefile，例：

   ```makefile
   GOCMD=go
   GOBUILD=$(GOCMD) build
   BINARY_NAME=bin
   NAME=blog-sync
   
   #mac
   build:
   	CGO_ENABLED=0 $(GOBUILD) -o $(BINARY_NAME)/$(NAME)-mac
   # windows
   build-linux:
   	CGO_ENABLED=0 GOOS=windows GOARCH=amd64 $(GOBUILD) -o $(BINARY_NAME)/$(NAME)-linux
   # linux
   build-win:
   	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 $(GOBUILD) -o $(BINARY_NAME)/$(NAME)-win.exe
   # 全平台
   build-all:
   	make build
   	make build-win
   	make build-linux
   
   ```

   执行`make build-all`即可在`bin`目录下生成 3 个平台的可执行文件。

   ```yaml
   # https://github.com/flytam/blog-sync/blob/master/.travis.yml
   language: go
   
   go:
       - 1.13.x
   env:
       - GO111MODULE=on # 启用Go mod
   install:
       - go get -v
   before_deploy: make build-all # 发布前执行生成二进制文件的命令
   deploy:
       provider: releases
       api_key:
           secure: xxxx
       # 使用glob匹配发布bin目录下的文件
       file_glob: true
       file: bin/*
       skip_cleanup: true
       on:
           repo: flytam/blog-sync
           # tag才触发发布
           tags: true
   
   ```

   

3. 使用`setup`初始化配置

   ```shell
   # 已经安装travis cli
   travis setup releases
   # 按需填写，输入github账号密码，加密key，发布文件等
   
   ```

   

4. 发布

   每次打tag推送到仓库，就会触发自动发布可执行文件到github release

   ```shell
   git tag 1.0.0
   git push --tags
   ```

   

