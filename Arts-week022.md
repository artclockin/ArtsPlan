# Arts-022

## 1.Algorithm

20. [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:

1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.

Note that an empty string is also considered valid.

**Example 1:**

```python
Input: "()"
Output: true
```

**Example 2:**

```python
Input: "()[]{}"
Output: true
```

**Example 3:**

```python
Input: "(]"
Output: false
```

**Example 4:**

```python
Input: "([)]"
Output: false
```

**Example 5:**

```python
Input: "{[]}"
Output: true
```



**My Solution:**

```go
func isValid(s string) bool {
	//1.stack ([{ push )]} check and pop
	//2.recursive replace ()[]{} to "" O(n^2)
	stack := make([]rune, 0, len(s))
	for _, item := range s {
		if item == '[' {
			stack = append(stack, item)
			stack = append(stack, ']')
		} else if item == '(' {
			stack = append(stack, item)
			stack = append(stack, ')')
		} else if item == '{' {
			stack = append(stack, item)
			stack = append(stack, '}')
		} else {
			if len(stack) == 0 {
				return false
			} else {
				if stack[len(stack) -1] == item {
					stack = stack[:len(stack) -2]
				} else {
					return  false
				}
			}
		}
	}
	if len(stack) == 0 {
		return true
	}
	return false
}

```



## 2.Review

[Using Service Objects in Go](https://itnext.io/using-service-objects-in-go-d899dc599335)

本篇介绍了如何在Go中如何应用服务对象和设计模式使得代码更简洁、易维护。

服务对象在Ruby on rails中是一种高度使用的模式，它提供了保持控制器精简、模型干净，移除领域逻辑。是使用单一职责原则和依赖注入的典范。

Robert Martin在他的《代码简洁之道》中展现了一种架构，包括了四个层次职责：实体、用例、接口、适配器、框架和驱动。这个架构引入用例原因与服务对象封装业务逻辑原因相同。

使用接口和依赖注入可以使代码独立于UI、框架和驱动程序，同时提供了通过Mock的测试能力。

下面用例子解释

```go
// Repository is a data access layer.
type Repository interface {
	Exists(email string) (bool, error)
	Create(*Form) (*User, error)
}

// RegistrationHandler for handling registration requests.
type RegistrationHandler struct {
	Validator *validator.Validate
	Repository
}

// ServerHTTP implements http.Handler.
func (h *RegistrationHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	var f Form
	if err := json.NewDecoder(r.Body).Decode(&f); err != nil {
		w.WriteHeader(http.StatusBadRequest)
		return
	}

	validations := make(map[string]string)

	err := h.Validator.Struct(f)
	if err != nil {
		if vs, ok := err.(validator.ValidationErrors); ok {
			for _, v := range vs {
				validations[v.Tag()] = fmt.Sprintf("%s is invalid", v.Tag())
			}
		}
	}

	if f.Password != f.PasswordConfirmation {
		validations["password"] = passwordMismatch
	}

	exists, err := h.Exists(f.Email)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		return
	}
	if exists {
		validations["email"] = emailExists
	}

	if len(validations) > 0 {
		w.WriteHeader(http.StatusUnprocessableEntity)
		json.NewEncoder(w).Encode(validations)
		return
	}

	u, err := h.Create(&f)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		return
	}

	json.NewEncoder(w).Encode(&u)
}
```



这里处理程序除了对请求进行响应，还包括用户注册的业务逻辑，代码僵化，脆弱。一旦有需求变更，很容易引起bug。譬如需要增加向注册用户发送通知验证邮件，则代码会变得更加难以理解和测试。



下面进行重构，第一步将注册过程逻辑封装到服务对象中。

```go
// Repository is a data access layer.
type Repository interface {
	Unique(email string) error
	Create(*Form) (*User, error)
}

// Validater validation abstraction.
type Validater interface {
	Validate(*Form) error
}

// ValidationErrors holds validation errors.
type ValidationErrors map[string]string

// Error implements error interface.
func (v ValidationErrors) Error() string {
	return validationMsg
}

// Service holds data required for registration.
type Service struct {
	Validater
	Repository
}

// Registrate holds registration domain logic.
func (s *Service) Registrate(f *Form) (*User, error) {
	if err := s.Validater.Validate(f); err != nil {
		return nil, errors.Wrap(err, "validater validate")
	}

	user, err := s.Repository.Create(f)
	if err != nil {
		return nil, errors.Wrap(err, "repository create")
	}

	return user, nil
}
```

Registrate方法包含了用户注册的两个步骤：

- 对表单进行验证
- 模型写入数据库



```go
// Registrater abstraction for registration service.
type Registrater interface {
	Registrate(*Form) (*User, error)
}

// RegistrationHandler for regisration requests.
type RegistrationHandler struct {
	Registrater
}

// ServerHTTP implements http.Handler.
func (h *RegistrationHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	var f Form
	if err := json.NewDecoder(r.Body).Decode(&f); err != nil {
		w.WriteHeader(http.StatusBadRequest)
		return
	}

	u, err := h.Registrate(&f)
	if err != nil {
		switch v := errors.Cause(err).(type) {
		case ValidationErrors:
			w.WriteHeader(http.StatusUnprocessableEntity)
			json.NewEncoder(w).Encode(v)
		default:
			w.WriteHeader(http.StatusInternalServerError)
		}
		return
	}

	json.NewEncoder(w).Encode(&u)
}
```

通过使用服务对象，代码变得更易理解和维护。

现代Web发展对系统提出新的要求，重要一点就是可观测性，包括日志记录、度量和跟踪。要达到这目标，给定代码应该通过上下文传递到服务对象中。

```go
// Service holds data required for registration.
type Service struct {
	Validater
	Repository
}

// Registrate holds registration domain logic.
func (s *Service) Registrate(ctx context.Context, f *Form) (*User, error) {
	if err := s.Validater.Validate(ctx, f); err != nil {
		return nil, errors.Wrap(err, "validater validate")
	}

	user, err := s.Repository.Create(ctx, f)
	if err != nil {
		return nil, errors.Wrap(err, "repository create")
	}

	return user, nil
}
```

可以使用装饰模式扩展服务对象的能力，如增加日志、度量等。






## 3.Tips
Golang中 使用emoji
```shell
go get github.com/enescakir/emoji
```
测试：

```go
	fmt.Printf("Hello %v\n", emoji.WavingHand)
	fmt.Printf("I am %v from %v\n",
		emoji.ManTechnologist,
		emoji.FlagForTurkey,
	)
	fmt.Printf("Different skin tones.\n  default: %v light: %v dark: %v\n",
		emoji.ThumbsUp,
		emoji.OkHand.Tone(emoji.Light),
		emoji.CallMeHand.Tone(emoji.Dark),
	)
	fmt.Printf("Emojis with multiple skin tones.\n  both medium: %v light and dark: %v\n",
		emoji.PeopleHoldingHands.Tone(emoji.Medium),
		emoji.PeopleHoldingHands.Tone(emoji.Light, emoji.Dark),
	)

Hello 👋
I am 👨‍💻 from 🇹🇷
Different skin tones.s
  default: 👍 light: 👌🏻 dark: 🤙🏿
Emojis with multiple skin tones.
  both medium: 🧑🏽‍🤝‍🧑🏽 light and dark: 🧑🏻‍🤝‍🧑🏿
```






## 4.Share

[Golang : cobra 包简介](https://www.cnblogs.com/sparkdev/p/10856077.html)

Cobra 是一个 Golang 包，它提供了简单的接口来创建命令行程序。同时，Cobra 也是一个应用程序，用来生成应用框架，从而开发以 Cobra 为基础的应用。

cobra 的主要功能如下：

- 简易的子命令行模式，如 app server， app fetch 等等
- 完全兼容 posix 命令行模式
- 嵌套子命令 subcommand
- 支持全局，局部，串联 flags
- 使用 cobra 很容易的生成应用程序和命令，使用 cobra create appname 和 cobra add cmdname
- 如果命令输入错误，将提供智能建议，如 app srver，将提示 srver 没有，是不是 app server
- 自动生成 commands 和 flags 的帮助信息
- 自动生成详细的 help 信息，如 app help
- 自动识别帮助 flag -h，--help
- 自动生成应用程序在 bash 下命令自动完成功能
- 自动生成应用程序的 man 手册
- 命令行别名
- 自定义 help 和 usage 信息
- 可选的与 viper apps 的紧密集成



cobra 中有个重要的概念，分别是 commands、arguments 和 flags。其中 commands 代表行为，arguments 就是命令行参数(或者称为位置参数)，flags 代表对行为的改变(也就是我们常说的命令行选项)。

cobra 是一个非常实用(流行)的包，很多优秀的开源应用都在使用它，包括 Docker 和 Kubernetes 等等。