# Arts-015

## 1.Algorithm

26. [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

Given a sorted array *nums*, remove the duplicates [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm) such that each element appear only *once* and return the new length.

Do not allocate extra space for another array, you must do this by **modifying the input array [in-place](https://en.wikipedia.org/wiki/In-place_algorithm)** with O(1) extra memory.

**Example 1:**

```
Given nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively.

It doesn't matter what you leave beyond the returned length.
```

**Example 2:**

```
Given nums = [0,0,1,1,1,2,2,3,3,4],

Your function should return length = 5, with the first five elements of nums being modified to 0, 1, 2, 3, and 4 respectively.

It doesn't matter what values are set beyond the returned length.
```

**Clarification:**

Confused why the returned value is an integer but your answer is an array?

Note that the input array is passed in by **reference**, which means modification to the input array will be known to the caller as well.

Internally you can think of this:

```
// nums is passed in by reference. (i.e., without making a copy)
int len = removeDuplicates(nums);

// any modification to nums in your function would be known by the caller.
// using the length returned by your function, it prints the first len elements.
for (int i = 0; i < len; i++) {
print(nums[i]);
}
```

​    

**My Solution:**

```Go
func removeDuplicates(nums []int) int {
	if len(nums) == 0 {
		return 0
	}
	//double pointer method
	i := 0
	for j:= 1; j < len(nums); j++ {
		if nums[i] != nums[j] {
			i++
			nums[i] = nums[j]
		}
	}
	return i + 1
}

```



## 2.Review

[Switching from OOP to Functional Programming](https://medium.com/@olxc/switching-from-oop-to-functional-programming-4187698d4d3)

为什么函数式编程如此难？本篇分享了学习FP容易困扰问题，并用很多示例并通过问答进行解释。

- 没有类
程序有各种函数组成，并相互调用。很多语言通过Type来提供多态。譬如Scala语言`case class Person(name: String, age: Int)`，使用FP术语，它们是类型，构造函数称为“类型构造函数”。以下是构造类型并从中获取值的方法：`val person = Person("Bob", 42);val personsAge = person.age`。如果要修改值，则需要copy数据，如`val olderBob = bob.copy(age = 43)`。类型分为Product和sum。sum类型常用于模式匹配。

- 你只需要函数。函数的3大特性：
	- 无副作用
	- 完备的，对所有的输入有返回值。常见并不总是如此，譬如函数发生被0整除时，会抛出异常。
	- 确定性，相同输入产生相同输出
如果函数满足着三大特性，就可以获得“引用透明性”。可以任意重构代码而不担心风险。

- 你不能改变一个变量
变量不可变，仅用于别名或标签值。

- 不能使用for来进行循环，一般采用模式匹配和高阶函数如Map、fold等。

- 代码不再是一些指令列表
命令式语句一般有副作用，函数式编程是直到最后一刻才开始执行，一个函数输出是另一个函数输入。

- 关于空值和异常
空值通常用于表示不存在或某种内部故障，从而阻止函数返回正确的值。可以通过Sum类型如Option解决。同样，可以用普通值表示缺失、失败和异常。使用throw e抛出异常会使函数破坏引用透明性并产生问题。

- Functors（函子）, Monads（单子）, Applicatives（应用函子）?
这些是范畴论的一般模式，熟悉函数组合、高阶函数和多态函数，然后阅读类型类。在那之后，函子和单子理解是自然的了。



## 3.Tips

Go中两个随机数生成器：

### Math/rand 伪随机数生成器

- 随机数由源生成。
- 使用默认的共享源，该源在每次运行程序时都会产生确定的值序列。
- 如果每次运行需要不同的行为，需使用种子函数初始化默认的源。
- 默认的Source可安全地供多个goroutine并发使用，但不是由NewSource创建的Source。

### Crypto/rand 加密安全的随机数生成器






## 4.Share

分享一个技术文章：[Golang IO Cookbook](https://github.com/jesseduffield/notes/wiki/Golang-IO-Cookbook)

作者介绍了开发Horcrux程序（把文件切割加密存储与恢复）时对io.Reader、io.Writer接口的理解。
- 第一版
```Go
func main() {
  // plaintext: "this is my file's content"
  content, _ := ioutil.ReadFile("myfile")

  encryptedContent := encrypt(content)

  // ciphertext: "uijt!jt!nz!gjmf(t!dpoufou"
  ioutil.WriteFile("myfile.encryped", encryptedContent, 0644)
}
```
对小文件很快，但对大文件（1GB）很慢，因为把源文件全部读入内存。

- 使用io.Reader和io.Writer
![示意图](https://camo.githubusercontent.com/0a71972e5e461df800bd3297bf023433bb10fbae/68747470733a2f2f692e696d6775722e636f6d2f6b7049464e69722e706e67)
通过传输缓冲区，无需将整个资源加载到内存
无需等待读取整个文件就可以进行加密，通过缓冲区写入目标文件。

- io.Copy 可以将读与写连接起来，不需要手工处理缓冲区
```Go
type augmentedWriter struct {
	innerWriter io.Writer
	augmentFunc func([]byte) []byte
}

// replaces ' ' with '!'
func bangify(buf []byte) []byte {
	return bytes.Replace(buf, []byte(" "), []byte("!"), -1)
}

func (w *augmentedWriter) Write(buf []byte) (int, error) {
	return w.innerWriter.Write(w.augmentFunc(buf))
}

func BangWriter(w io.Writer) io.Writer {
	return &augmentedWriter{innerWriter: w, augmentFunc: bangify}
}

func UpcaseWriter(w io.Writer) io.Writer {
	return &augmentedWriter{innerWriter: w, augmentFunc: bytes.ToUpper}
}


func main() {
	originalReader := strings.NewReader("this is the stuff I'm reading\n")
	writer := os.Stdout

	augmentedReader := UpcaseReader(BangReader(originalReader))
	_, err := io.Copy(writer, augmentedReader)
	if err != nil {
		log.Fatal(err)
	}
}
```

- io.Pipe 管道可以从writer传递给reader
```Go
package augment

func encrypt(s []byte) []byte {
	result := make([]byte, len(s))
	for i, c := range s {
		result[i] = c + 28 // state-of-the-art encryption ladies and gentlemen
	}
	return result
}

func EncryptReader(r io.Reader) io.Reader {
	return &augmentedReader{innerReader: r, augmentFunc: encrypt}
}

...

package main

func main() {
	originalReader := strings.NewReader("this is the stuff I'm reading\n")
	originalWriter := os.Stdout

	pipeReader, pipeWriter := io.Pipe()

	go func() {
		defer pipeWriter.Close()
		_, err := io.Copy(UpcaseWriter(pipeWriter), originalReader)
		if err != nil {
			log.Fatal(err)
		}
	}()

	defer pipeReader.close()
	_, err := io.Copy(originalWriter, EncryptReader(pipeReader))
	if err != nil {
		log.Fatal(err)
	}
	// output: 'pdeo<eo<pda<opqbb<eCi<na]`ejc&' (notably not uppercased)
}
```

- TeeReader可以解决reader只能读取一次的问题
```Go
func main() {
	reader := strings.NewReader("look at me\n")

	file, err := os.OpenFile("file.encrypted", os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		log.Fatal(err)
	}

	teeReader := io.TeeReader(reader, EncryptWriter(file))

	_, err = io.Copy(os.Stdout, teeReader)
	if err != nil {
		log.Fatal(err)
	}
	// output: this is the stuff I'm reading\n
	// file.encrypted's contents:hkkg<]p<ia&
}
```

- 组合起来流程图
![示意图](https://camo.githubusercontent.com/d84384675c7f2606157de318eb440f19e01e2922/68747470733a2f2f692e696d6775722e636f6d2f4d477636625a6a2e706e67)



文中也介绍一些常见IO函数如bufio.Scanner，io.WriteString，文件操作函数ioutil.ReadFile、ioutil.WriteFile等。