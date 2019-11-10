# Arts-005

## 1.Algorithm
18. [4Sum](https://leetcode.com/problems/4sum/)

    Given an array `nums` of *n* integers and an integer `target`, are there elements *a*, *b*, *c*, and *d* in `nums` such that *a* + *b* + *c* + *d* = `target`? Find all unique quadruplets in the array which gives the sum of `target`.
    
    **Note:**
    
    The solution set must not contain duplicate quadruplets.
    
    **Example:**
    
    ```
    Given array nums = [1, 0, -1, 0, -2, 2], and target = 0.
    
    A solution set is:
    [
      [-1,  0, 0, 1],
      [-2, -1, 1, 2],
      [-2,  0, 0, 2]
    ]
    ```

**My Solution1 - 使用map **

```Go
func fourSum(nums []int, target int) [][]int {
	result:= [][]int{}
	l_nums :=  len(nums)
	if l_nums < 4 {
		return result
	}

	sort.Ints(nums)

	for i, x := range nums[:l_nums-3] {
		if i >0 && x == nums[i-1] {
			continue
		}

		for j, y := range nums[i+1: l_nums -2] {
			if j >0 && y == nums[i + j] {
				continue
			}
			m := make(map[int]int)
			bFirst := true
			for k ,z := range nums[i + j + 2 :] {
				if k > 0 && z == nums[i + j + k + 1]{
					if bFirst {
						if target - x - y - z == z {
							result = append(result, []int{x,y,target-x-y-z,z})
							bFirst = false
						}
					}
					continue
				}
				if _, ok := m[z]; ok {
					result = append(result, []int{x,y,target-x-y-z,z})
				} else {
					m[target-x-y-z] = 1
				}
			}
		}
	}

 	return result
}

```

**My Solution2 **

```Go
func fourSum(nums []int, target int) [][]int {
	result:= [][]int{}
	l_nums :=  len(nums)
	var left, right int
	if l_nums < 4 {
		return result
	}

	sort.Ints(nums)

	for i, x := range nums[:l_nums-3] {
		if i >0 && x == nums[i-1] {
			continue
		}

		for j, y := range nums[i+1: l_nums -2] {
			if j >0 && y == nums[i + j] {
				continue
			}

			left = i +j + 2
			right = l_nums - 1

			for {
				if left >= right {
					break
				}
				v := target - x - y - nums[left] - nums[right]
				if v > 0 {
					left++
				} else if v < 0 {
					right--
				} else {
					result = append(result, []int{x,y,nums[left],nums[right]})
					for left < right && nums[left] == nums[left + 1] {
						left++
					}
					for left < right && nums[right] == nums[right - 1] {
						right--
					}
					left++
					right--
				}

			}
		}
	}

 	return result
}
```


## 2.Review

[InnoDB : Tablespace Space Management](https://mysqlserverteam.com/innodb-tablespace-space-management/)
本文从介绍了表空间管理，从.IBD文件物理结构谈起，一般一个表创建一个同名的表空间（file-per-table）。

Tablespace逻辑分层结构: Tablespace、Extents（连续的Pages，一般1M）、Pages （一般16LKb）。

两种Page：Header Page/XDES Page(extent description page)、Inno Page(对应Segment)。

File Segment是Pages和Extents的集合，每一个Innodb索引使用2个File Segment: Leaf Page Segment(B树叶子节点)和Non Leaf Page Segment(B树中间节点)。

最后介绍了索引创建、删除等操作引起的表空间相关操作。

全文简洁易懂，适合入门级。

![](https://mysqlserverteam.com/wp-content/uploads/2019/04/Index_File_Segment_Structure.png)




## 3.Tips
- **Go assert** 
Go test demo:
```Go
package main

import (
        "github.com/stretchr/testify/assert"
        "testing"
)

func TestCalculate(t *testing.T) {
        assert.Equal(t, Calculate(2), 4)
}

func TestCalculate2(t *testing.T) {
        assert := assert.New(t)

        var tests = []struct {
                input    int
                expected int
        }{
                {2, 4},
                {-1, 1},
                {0, 2},
                {-5, -3},
                {99999, 100001},
        }

        for _, test := range tests {
                assert.Equal(Calculate(test.input), test.expected)
        }
}

```




## 4.Share

[Turn Python Scripts into Beautiful ML Tools](https://towardsdatascience.com/coding-ml-tools-like-you-code-ml-models-ddba3357eace) 

Streamlit特点：

1. Widget当成变量处理，动态更新

2. Streamlit 每一次交互都只是自上而下重新运行脚本

3. 缓存允许Streamlit跳过冗余数据获取和计算

4. Streamlit 是免费开源库，你可以本地部署 Streamlit app

![](https://miro.medium.com/max/2232/1*l4gxFYEZnRhysQ_QWIVJgA.png)

**从零开始**

```Python
$ pip install --upgrade streamlit 
$ streamlit hello
   You can now view your Streamlit app in your browser.
   Local URL: http://localhost:8501
```

