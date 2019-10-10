# Arts-001

## 1.Algorithm

43. [Multiply Strings](https://leetcode.com/problems/multiply-strings/)

Given two non-negative integers `num1` and `num2` represented as strings, return the product of `num1` and `num2`, also represented as a string.

**Example 1:**

```
Input: num1 = "2", num2 = "3"
Output: "6"
```

**Example 2:**

```
Input: num1 = "123", num2 = "456"
Output: "56088"
```

**Note:**

1. The length of both `num1` and `num2` is < 110.
2. Both `num1` and `num2` contain only digits `0-9`.
3. Both `num1` and `num2` do not contain any leading zero, except the number 0 itself.
4. You **must not use any built-in BigInteger library** or **convert the inputs to integer** directly.

**My Solution**

```
func multiply(num1 string, num2 string) string {
	len_1 := len(num1)
	len_2 := len(num2)
	//lookuptable
	var vtable [10][10]int
	for i := 0; i < 10; i++ {
		for j := 0; j < 10; j++ {
			vtable[i][j] = i * j
		}
	}

	//calc each number of num2 * num1
	caltable := make([]int, len_1+len_2, len_1+len_2)
	for j, n2 := range num2 {
		roundi := caltable[j+1 : len_1+j+1]
		for i, n1 := range num1 {
			roundi[i] = roundi[i] + vtable[n1-48][n2-48]
		}
	}

	//carry bit
	ret := make([]rune, len_1+len_2)
	for i := len_1 + len_2 - 1; i > 0; i-- {
		caltable[i-1] = caltable[i-1] + int(caltable[i]/10)
		caltable[i] = caltable[i] % 10
		ret[i] = rune(caltable[i] + 48)
	}
	ret[0] = rune(caltable[0] + 48)

	//remove leading 0
	start := 0
	for i, chr := range ret {
		if chr != 48 {
			start = i
			break
		} else {
			start++
		}
	}

	if start == len_1+len_2 {
		start = len_1 + len_2 - 1
	}

	return string(ret[start:])
}
```



## 2.Review

[BPF Performance Tools: Linux System and Application Observability (book)](http://www.brendangregg.com/blog/2019-07-15/bpf-performance-tools-book.html)

![封面](http://www.brendangregg.com/blog/images/2019/bpfperftools_bookcover_500.png)

- 推荐即将发行(2019-11-4)的新书，作者Brendan Gregg是Netflix的高级性能工程师，也是bpf（ebpf）的主要贡献者。

- BPF是动态跟踪的代表，排查问题的利器。

- 本书主要讲解使用BPF进行各种场景的性能分析，覆盖CPU、memory、disks、file system、networking、
  不同开发语言、应用如mysql、容器、虚拟化等。

- BPF开发很繁琐，本书重点推荐高级语言前端工具BCC和bpftrace，bpftrace已经在Netflix、Facebook和Shopify等公司用于生产分析。

- 这本书号称是Linux可观察性新时代的开始，在生产中拥有轻松查看任何东西的能力。


## 3.Tips
Golang中 int转string，调研一下主要有三种方法
a. strconv.Itoa, str := strconv.Itoa(i)
b. strconv.FormatInt, str := strconv.FormatInt(i, 10)
c. fmt.Sprintf,  str := fmt.Sprintf("%d", i)

a和b是等效的，c方法性能略低，建议使用a
看下简化后源码：
```
Itoa和FormatInt方法位于strconv/itoa.go

const digits = "0123456789abcdefghijklmnopqrstuvwxyz"
const smallsString = "00010203040506070809" +
    "10111213141516171819" +
    "20212223242526272829" +
    "30313233343536373839" +
    "40414243444546474849" +
    "50515253545556575859" +
    "60616263646566676869" +
    "70717273747576777879" +
    "80818283848586878889" +
    "90919293949596979899"

func Itoa(i int) string {
    return FormatInt(int64(i), 10)
}

func FormatInt(i int64, base int) string {
    //小于100直接查表优化处理
    if 0 <= i && i < nSmalls && base == 10 {
        return small(int(i))
    }
    _, s := formatBits(nil, uint64(i), base, i < 0, false)
    return s
}

// small returns the string for an i with 0 <= i < nSmalls.
func small(i int) string {
    if i < 10 {
        return digits[i : i+1]
    }
    return smallsString[i*2 : i*2+2]
}

func formatBits(dst []byte, u uint64, base int, neg, append_ bool) (d []byte, s string) {
    var a [64 + 1]byte // +1 for sign of 64bit value in base 2
    i := len(a)

    if neg {
        u = -u
    }

    // u guaranteed to fit into a uint
    us := uint(u)
    //从尾向前每两位查表，把对应str值赋值给a数组
    for us >= 100 {
        is := us % 100 * 2
        us /= 100
        i -= 2
        a[i+1] = smallsString[is+1]
        a[i+0] = smallsString[is+0]
    }

    // us < 100 最前1-2位处理
    is := us * 2
    i--
    a[i] = smallsString[is+1]
    if us >= 10 {
        i--
        a[i] = smallsString[is]
    }

    // add sign, if any
    if neg {
        i--
        a[i] = '-'
    }

    //反甲字符串
    s = string(a[i:])
    return
}
```
fmt.Sprintf转int核心方法位于fmt/format.go
```
func (f *fmt) fmtInteger(u uint64, base int, isSigned bool, verb rune, digits string) 

buf := f.intbuf[0:]
i := len(buf)
for u >= 10 {
    i--
    next := u / 10
    buf[i] = byte('0' + u - next*10)
    u = next
}
```

如果自己实现一个可参考一下。


## 4.Share

分享一个技术文章：[微信在Github开源了Hardcoder，对Android开发者有什么影响？](https://juejin.im/post/5d9e935f51882558b77d3200)

Hardcoder提供了一个APP和系统之间的通讯框架，可以告诉系统需要哪些资源。扩展开来，如果各个系统都能够动态地交换资源需求和使用情况，系统调度算法就能够更好优化，从而达到更好的系统利用率。期待PC端也有类似框架。

