# Arts-017

## 1.Algorithm

162. [Find Peak Element](https://leetcode.com/problems/find-peak-element/)

A peak element is an element that is greater than its neighbors.

Given an input array `nums`, where `nums[i] ≠ nums[i+1]`, find a peak element and return its index.

The array may contain multiple peaks, in that case return the index to any one of the peaks is fine.

You may imagine that `nums[-1] = nums[n] = -∞`.

**Example 1:**

```
Input: nums = [1,2,3,1]
Output: 2
Explanation: 3 is a peak element and your function should return the index number 2.
```

**Example 2:**

```
Input: nums = [1,2,1,3,5,6,4]
Output: 1 or 5 
Explanation: Your function can return either index number 1 where the peak element is 2, 
or index number 5 where the peak element is 6.
```

**Note:**

Your solution should be in logarithmic complexity.



**My Solution:**

```go
func findPeakElement(nums []int) int {
	left := 0
	right := len(nums) - 1
	for left < right {
		mid := (left + right)/2
		if mid + 1 > right {
			mid = right - 1
		}
		if nums[mid] > nums[mid + 1] {
			right = mid
		} else {
			left = mid + 1
		}
	}

	return left
}
```



## 2.Review

[Functional options on steroids](https://sagikazarmark.hu/blog/functional-options-on-steroids/)

该文介绍了Go中配置实现范式-函数式选项。在开发应用中经常遇到需要大量配置参数情况，有默认配置、可选配置、必选配置等，函数式选项使得可以创建出美观、优雅的API实现。

不带函数式选项例子：

```go
type Server struct {
    addr string
}

// NewServer initializes a new Server listening on addr.
func NewServer(addr string) *Server {
    return &Server {
        addr: addr,
    }
}
```

添加超时选项后，代码如下所示：

```go
type Server struct {
    addr string

    // default: no timeout
    timeout time.Duration
}

// Timeout configures a maximum length of idle connection in Server.
func Timeout(timeout time.Duration) func(*Server) {
    return func(s *Server) {
        s.timeout = timeout
    }
}

// NewServer initializes a new Server listening on addr with optional configuration.
func NewServer(addr string, opts ...func(*Server)) *Server {
    server := &Server {
        addr: addr,
    }

    // apply the list of options to Server
    for _, opt := range opts {
        opt(server)
    }

    return server
}
```

创建server代码如下：

```go
// no optional paramters, use defaults
server := NewServer(":8080")

// configure a timeout in addition to the address
server := NewServer(":8080", Timeout(10 * time.Second))

// configure a timeout and TLS in addition to the address
server := NewServer(":8080", Timeout(10 * time.Second), TLS(&TLSConfig{}))
```



相比之下，下面构造函数变体和配置结构版本的示例：

```go
// constructor variants
server := NewServer(":8080")
server := NewServerWithTimeout(":8080", 10 * time.Second)
server := NewServerWithTimeoutAndTLS(":8080", 10 * time.Second, &TLSConfig{})


// config struct
server := NewServer(":8080", Config{})
server := NewServer(":8080", Config{ Timeout: 10 * time.Second })
server := NewServer(":8080", Config{ Timeout: 10 * time.Second, TLS: &TLSConfig{} })
```

可以看出使用函数式选项更简洁明了。构造器方法显得冗长不易为何，配置方式空结构显得不友好。



文中具体介绍了多种函数式选项实践，如list选项，With/Set命名前缀，默认值，config选项等。下面列出一段代码：

```go
// Option configures a Server.
type Option interface {
    // apply is unexported,
    // so only the current package can implement this interface.
    apply(s *Server)
}

// Timeout configures a maximum length of idle connection in Server.
type Timeout time.Duration

func (t Timeout) apply(s *Server) {
    s.timeout = time.Duration(t)
}

// Options turns a list of Option instances into an Option.
type Options []Option

func (o Options) apply(s *Server) {
    for _, opt := range o {
        o.apply(s)
    }
}

type Config struct {
    Timeout time.Duration
}

func (c Config) apply(s *Server) {
    s.config = c
}

NewServer(":8080", Timeout(time.Minute))
```








## 3.Tips

Golang中使用 mock:
```shell
GO111MODULE=on go get github.com/golang/mock/mockgen

#build and install
go vet ./...
go build ./...
go install github.com/golang/mock/mockgen
```



例子:

构建项目文件结构

```shell
mockdemo
├── client
│   ├── client.go
│   └── client_test.go
└── service
    ├── mocks
    │   └── service_mock.go
    └── service.go

```

service.go

```Go
package service

type Request struct {
   Req string
}

type Response struct {
   Resp string
}

type IService interface {
   DoWork(req *Request) (*Response, error)
}

```

service创建完后，执行mockgen,自动生成mock对象

`mockgen -destination ./mocks/service_mock.go -package mocks -source ./service.go`



client.go

```
package  client
import (
   "fmt"
   "mockdemo/service"
)

type Client struct {
   service.IService
}

func (c *Client) Call(req *service.Request) error {
   resp, err := c.DoWork(req)
   if err != nil {
      return err
   }
   fmt.Printf("req is \"%v\" and response is \"%v\"\n:", req.Req, resp.Resp)
   return nil
}
```

client_test.go
```Go
package  client
import (
   "errors"
   "github.com/golang/mock/gomock"
   "mockdemo/service"
   "mockdemo/service/mocks"
   "testing"
)

func TestDoWork(t *testing.T) {
   ctrl := gomock.NewController(t)
   defer ctrl.Finish()
   mock := mocks.NewMockIService(ctrl)

   req := &service.Request{Req: "Hello world"}
   resp := &service.Response{Resp: "Hi"}
   err := errors.New("Please input your reqest.")

   tests := []struct {
      req       *service.Request
      expectErr error
   }{
      {req: req, expectErr: nil},
      {req: nil, expectErr: err},
   }

   mock.EXPECT().DoWork(req).Return(resp, nil)
   mock.EXPECT().DoWork(nil).Return(nil, err)

   c := &Client{IService: mock}
   for i, v := range tests {
      err := c.Call(v.req)
      if err != v.expectErr {
         t.Errorf("test %v:get wrong expectErr %v and expectErr is %v", i, err, v.expectErr)
      }
   }
}
```



运行结果：

```shell
~/go/src/mockdemo/client$ go test
req is "Hello world" and response is "Hi"
:PASS
ok  	mockdemo/client	0.007s
```



## 4.Share

分享一个技术文章：[golang 结构体struct](https://juejin.im/post/5e2024e9518825364f4a44d3)

struct是一种聚合数据类型，可以包含任意类型的字段。文章从多个角度介绍结构体的特性。

- 声明结构体

  ```Go
  type Person struct {
    Name string
    Age int
    Birthday time.Time
}
  ```
结构体字段的首字母标识字段的权限访问，大写的包外可访问，小写的属于包内私有变量，仅在该结构体内部使用

- 初始化结构体

  ```Go
  ts := "2000-01-01 12:00:32"
  timeLayout := "2006-01-02 15:04:05"                             //转化所需模板
  loc, _ := time.LoadLocation("Local")                            //重要：获取时区
  theTime, _ := time.ParseInLocation(timeLayout, ts, loc) //使用模板在对应时区转化为time.time类型
  p := Person{Name:"zhanglinpeng",Age: 18, Birthday: theTime} 
  p2 := new(Person) //new()函数来创建一个「零值」结构体，所有的字段都被初始化为相应类型的零值。返回的是结构体指针
  zerostruct := struct{}{} //空结构体,字节长度为0
  ```

- 匿名结构体

  ```go
  var apr = struct {
     Name string
     Age  int
  }{
     Name: "zhanglinpeng",
     Age: 13,
  }
  fmt.Println(apr)
  ```

- 匿名字段

  ```go
      type Student struct {
        string
        int
     }
     a := Student{"zhaoll", 19}
     fmt.Println(a)
  }
  ```

- 结构体嵌套

  ```go
  var apr = struct {
     Name string
     Age  int
     Contact struct {
        Phone, Email string
     }
  }{
     Name: "zhanglinpeng",
     Age: 13,
  }
  apr.Contact.Phone = "110"
  apr.Contact.Email = "110@qq.com"
  fmt.Println(apr)
  ```

- 值传递

  结构体作为参数传递给函数时，是值传递，如果想修改原始结构体，可以使用指针

- 属性值获取

  可以使用点操作符获取属性值，点操作符还可以应用在struct指针上。

- 结构体比较

  通过其字段值比较是否完全相同

- 嵌入（“继承”）

  ```go
  type Person struct {
  	Gender int
  }
  
  type teacher struct {
  	Person
  	Name string
  	Age int
  }
  
  type student struct {
  	Person
  	Name string
  	Age int
  }
  
  t1 := teacher{Name:"mayun", Age: 44, Person: Person{Gender: 0}}
  s1 := student{Name: "zhaojj", Age: 12, Person: Person{Gender: 1}}
  t1.Name = "yangmi"
  s1.Gender = 0
  ```

  修改“继承”来的属性，可以直接点操作，而不用s1.Person.Gender = 0。如果在嵌入的结构体中存在同名的属性字段，那么在访问不同结构体中的属性字段时，需要指明。

- 结构体方法（“类方法”）

  只需要在普通函数前面加个接受者（receiver，写在函数名前面的括号里面。Go不支持函数重载，某个接收者（receiver）的某个方法只能对应一个函数。

- 结构体指针

  无论是结构体变量还是结构体指针变量，都是可以调用接受者不管是结构体还是结构体指针的方法。但是，传递给接口的时候会有所不同

  ```go
  type DisplayInfo interface {
     displayId()
     displayName()
  }
  
  type User struct {
     Id   int
     Name string
  }
  
  func (u User) displayId() {
     fmt.Println(u.Id)
  }
  
  
  func (u *User) displayName() {
     fmt.Println(u.Name)
  }
  
  
  func DisplayUserInfo(ds DisplayInfo) {
     ds.displayId()
     ds.displayName()
  }
  
  func main() {
     us := User{Id: 1, Name: "zhao"}
     us.displayId()
     us.displayName()
     us2 := &User{Id: 2, Name: "qian"}
     us2.displayId()
     us2.displayName()
     DisplayUserInfo(&User{Id:3,Name:"sun"})
     //us3 :=User{Id:3,Name:"sun"} 
     //DisplayUserInfo(us3) // cannot use us3 (type User) as type DisplayInfo in argument to DisplayUserInfo
     // User does not implement DisplayInfo (displayName method has pointer receiver)
  }
  ```

  **接受者是指针类型的时候，说明指针指向的结构体实现了接口，而接受者是值类型的时候，说明结构体本身实现了接口**.接受者是T的属于一个方法集，接受者是**T的是另一个方法集，该方法及包含接受者是*T和*T的。

- 接受者的类型决定了能否修改绑定的结构体

  ```go
  type A struct {
    Name string
  }
  type B struct {
    Name string
  }
  
  func (a A) print() {
      a.Name = "FuncA"
      fmt.Println("function A")
  }

  func (b *B) print() {
      b.Name = "FuncB"
      fmt.Println("function B")
  }
  func main() {
      a := A{}
      a.print()
      fmt.Println(a.Name)  // ""
      b := B{}
      b.print()
      fmt.Println(b.Name) // "FuncB"
  }

  ```

- 方法绑定本身只能绑定包内的类型

  包外的类型是无法绑定方法的。

- method value VS method expression

- 方法权限

  Go是以大小写来区分是公有还是私有，**但都是针对包级别的**， 所以在包内所有的都能访问，而方法绑定本身只能绑定包内的类型，所以方法可以访问接收者所有成员。
