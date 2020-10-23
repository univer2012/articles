#### [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

示例:

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```



答案：

Swift版:

```swift
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    var res = [0,0] // 要返回的2个元素的下标
    guard nums.count > 1 else  {
        return res
    }
    var map = [Int: Int]() //第一个Int为num的下标，第二个Int为num的值
    for (index,num) in nums.enumerated() {
        let val = target - num
        if map.keys.contains(val) {
            res[0] = index
            if let valIndex = map[val] {
                res[1] = valIndex
            }
            return res.reversed()
        } else {
            map[num] = index
        }
    }
    return res
}

//例子：
let nums = [1,2,9,7,5]//[2,7,11,15]
print("jieguo=\(twoSum(nums, 9))")
```



