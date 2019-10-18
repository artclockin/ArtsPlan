# Arts-002

## 1.Algorithm
4. [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)

There are two sorted arrays **nums1** and **nums2** of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

You may assume **nums1** and **nums2** cannot be both empty.

**Example 1:**

```
nums1 = [1, 3]
nums2 = [2]

The median is 2.0

```

**Example 2:**

```
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```

**My Solution:**


```Go
func updatePrevLastValue(arr []int, pos int, prevLast []int) {
	if pos >= 2 {
		if len(arr) > pos -2 && arr[pos-2] > prevLast[0] {
			prevLast[0] = arr[pos-2]
		}
		if len(arr) > pos -1 && arr[pos -1] > prevLast[1] {
			if prevLast[0] < prevLast[1] {
				prevLast[0] = prevLast[1]
			}
			prevLast[1] = arr[pos -1]
		}
	} else if pos >=1 {
		if len(arr) > pos -1 && arr[pos -1] > prevLast[1] {
			if prevLast[0] >= prevLast[1] {
				prevLast[1] = arr[pos-1]
				prevLast[0] = prevLast[1]
			} else {
				prevLast[0] = prevLast[1]
				prevLast[1] = arr[pos-1]
			}
		}
	} else {
		prevLast[0] = prevLast[1]
	}
}

func updateLastValueCur(prevLast [] int, curValue int) {
	if curValue > prevLast[1] {
		prevLast[0] = prevLast[1]
		prevLast[1] = curValue
	} else if curValue > prevLast[0] {
		prevLast[0] = curValue
	}
}

func uptdateNexValue(prev_last []int, arr []int, pos int) {
	if len(arr) > pos && pos >= 0{
		updateLastValueCur(prev_last, arr[pos])
	}
}

func binarySearchPos(arr []int, maxPos int, val int) int{
	arr = arr[:maxPos]
	valPos := 0
	low := 1
	mid := 0
	high := len(arr)
	for {
		if low > high {
			break
		}

		mid = (low + high)/2
		guess := arr[mid-1]
		if guess < val {
			valPos += mid
			arr=arr[mid:]
			high = len(arr)
		} else if guess > val {
			high = high / 2
		} else {
			valPos += mid
			break
		}
		if len(arr) == 0 {
			break
		} else if arr[0] > val {
			break
		}
	}

	repeatNums := 0
	if len(arr) > mid {
		for _, nextValue := range (arr[mid:]) {
			if nextValue == val {
				repeatNums += 1
			} else {
				break
			}
		}
	}
	valPos += repeatNums
	return valPos
}

func judgePos(findPos int, finalPos int, arr []int) int {
	if len(arr) == 0 {
		return 0
	}
	l1 := int((len(arr) + 1)/2)
	step := (finalPos - findPos)/2
	if l1 >= step {
		l1 = step
	}
	if l1 < 1 {
		l1 = 1
	}
	return  l1
}

func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
	const INT_MAX = int(^uint(0) >> 1)
	const INT_MIN = ^(INT_MAX)
	finalPos := int((len(nums1) + len(nums2) + 1)/2)//last position
	prevLast := []int{INT_MIN,INT_MIN} //prev last max values
	nextValue := INT_MAX//tmp value
	findPos := 0 //current pos
	nextPos := 0 //next arr remove pos
	diffPos := findPos - finalPos

	twoNum := false
	if (len(nums1) + len(nums2)) % 2 == 0 {
		twoNum = true
	}

	//define l1 position and l2 postion
	l1 := 0
	l2 := 0
	for {
		if len(nums2) == 0 {
			updatePrevLastValue(nums1, finalPos-findPos, prevLast)
			uptdateNexValue(prevLast, nums1, finalPos-findPos)
			break
		} else if len(nums1) == 0 {
			updatePrevLastValue(nums2, finalPos-findPos, prevLast)
			uptdateNexValue(prevLast, nums2, finalPos-findPos)
			break
		}
		l1 = judgePos(findPos, finalPos, nums1)
		l2 = judgePos(findPos, finalPos, nums2)

		//exchange arr if value1 > value2
		if nums1[l1 - 1] > nums2[l2 - 1] {
			nums1, nums2 = nums2, nums1
			l1, l2 = l2,l1
		}

		diffPos = findPos + l1 + l2 -1 - finalPos
		if diffPos == 0 {
			prevLast[0] = nums1[l1-1]
			if len(nums1) > l1 {
				nextValue = nums1[l1]
			}
			if nextValue > nums2[l2-1] {
				nextValue = nums2[l2 -1]
			}
			prevLast[1] = nextValue
			break
		} else if diffPos == 1 {
			prevLast[0] = prevLast[1]
			prevLast[1] = nums1[l1-1]
			break
		} else if diffPos > 1 {
			panic("should not execute here")
		} else if diffPos < 0 {
			findPos += l1
			updatePrevLastValue(nums1, l1 ,prevLast)
			nextPos = binarySearchPos(nums2, l2, nums1[l1-1])
			if nextPos >0 {
				findPos += nextPos
				updatePrevLastValue(nums2, nextPos, prevLast)
				nums2 = nums2[nextPos:]
			}
			nums1 = nums1[l1:]
		}
	}
	if !twoNum {
		prevLast[1] = prevLast[0]
	}

	return float64(prevLast[0]+prevLast[1])/2.0
}
```



## 2.Review




## 3.Tips



## 4.Share


