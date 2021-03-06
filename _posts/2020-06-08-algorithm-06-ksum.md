---
layout: post
title: leecode 详解 06-ksum 求符合条件的 k 个数
date:  2020-6-8 15:13:08 +0800
categories: [Algorithm]
tags: [Algorithm, data-struct, leetcode, sf]
published: true
---

## 两数之和

### 题目

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

示例:

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

### 思路

最简单粗暴的一个双层循环：

（1）遍历第一层数组 nums[i]

（2）遍历第二层数组 nums[j]，如果 nums[i] + nums[j] == t，符合。

### 基础解法

java 实现：

```java
public int[] twoSumBasic(int[] nums, int target) {
    for(int i = 0; i < nums.length; i++) {
        for(int j = 0; j < nums.length; j++) {
            // 每个元素只使用一次
            if(i == j) {
                continue;
            }
            if(nums[i] + nums[j] == target) {
                return new int[]{i, j};
            }
        }
    }
    // 实际每个都有答案，不应该都到这里。
    return null;
}
```

性能分析：

可谓惨不忍睹，为什么呢？

是否有比较好的优化思路？

```
Runtime: 167 ms, faster than 5.01% of Java online submissions for Two Sum.
Memory Usage: 41.3 MB, less than 10.92% of Java online submissions for Two Sum.
```

### 优化解法

优化思路：

借助 HashMap 数据结构，将原本 O(n) 的遍历，降低为 O(1) 的查询。

java 实现如下：

实现时注意 HashMap 的扩容问题，此处直接指定为和数组一样大。

```java
public int[] twoSum(int[] nums, int target) {
    int[] result = new int[2];
    final int length  = nums.length;
    Map<Integer, Integer> map = new HashMap<>(length);
    for(int i = 0; i < length; i++) {
        int num = nums[i];
        int targetKey = target - num;
        if (map.containsKey(targetKey)) {
            result[1] = i;
            result[0] = map.get(targetKey);
            return result;
        }
        map.put(num, i);
    }
    return result;
}
```

效果：

```
Runtime: 1 ms, faster than 99.93% of Java online submissions for Two Sum.
Memory Usage: 39.5 MB, less than 69.35% of Java online submissions for Two Sum.
```

## 三数之和

结束了第一道开胃菜之后，我们来看看第二道菜。

### 题目

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。

示例：

```
给定数组 nums = [-1, 0, 1, 2, -1, -4]，

满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```

### 粗暴解法

最简单的思路，我们直接来一个三层循环：

```java
public List<List<Integer>> threeSum(int[] nums) {
    if(nums.length < 3) {
        return Collections.emptyList();
    }
    List<List<Integer>> results = new ArrayList<>();
    Set<String> stringSet = new HashSet<>();
    for(int i = 0; i < nums.length-2; i+=3) {
        for(int j = i+1; j < nums.length-1; j++) {
            for(int k = j+1; k < nums.length; k++) {
                if((nums[i] + nums[j]+nums[k]) == 0) {
                    List<Integer> list = Arrays.asList(nums[i], nums[j], nums[k]);
                    // 排序保证结果不重复
                    Collections.sort(list);
                    String string = list.toString();
                    if(stringSet.contains(string)) {
                        continue;
                    }
                    stringSet.add(string);
                    results.add(list);
                }
            }
        }
    }
    return results;
}
```

- 执行结果

很不幸，这个是不通过的。会执行超时，因为执行的比较慢。

### 优化解法

优化思路：

（1）对原始数组进行排序，保证可以使用双指针法

（2）固定 1 个元素。剩余的两个元素采用双指针的方式。初始化时，一个在最左边，一个在最右边。然后不断调整位置，直到符合条件为止。

（3）不可以包含重复的三元组，要同时跳过重复的信息。

java 实现：

```java
public List<List<Integer>> threeSum(int[] nums) {
    //1. 排序
    Arrays.sort(nums);
    List<List<Integer>> results = new ArrayList<>(nums.length);
    //2. 双指针
    for(int i = 0; i < nums.length; i++) {
        int num = nums[i];
        if(num > 0) {
            return results;
        }
        if(i > 0 && nums[i] == nums[i-1]) {
            continue;
        }
        int l = i+1;
        int r = nums.length-1;
        while (l < r) {
            int sum = num + nums[l] + nums[r];
            if(sum < 0) {
                l++;
            } else if(sum > 0) {
                r--;
            } else {
                List<Integer> integers = new ArrayList<>(3);
                integers.add(num);
                integers.add(nums[l]);
                integers.add(nums[r]);
                results.add(integers);
                // 跳过重复的元素
                while(l < r && nums[l+1] == nums[l]) {
                    l++;
                }
                while (l < r && nums[r-1] == nums[r]) {
                    r--;
                }
                l++;
                r--;
            }
        }
    }
    return results;
}
```

性能：

速度超过 99.87 的用户提交，还不错。

## 四数之和

常言道，有二有三必须有四。

这道题当然还有四个数之和的版本。

### 题目

给定一个包含 n 个整数的数组 nums 和一个目标值 target，判断 nums 中是否存在四个元素 a，b，c 和 d ，使得 a + b + c + d 的值与 target 相等？找出所有满足条件且不重复的四元组。

注意：

答案中不可以包含重复的四元组。

示例：

```
给定数组 nums = [1, 0, -1, 0, -2, 2]，和 target = 0。

满足要求的四元组集合为：
[
  [-1,  0, 0, 1],
  [-2, -1, 1, 2],
  [-2,  0, 0, 2]
]
```

### 解法

解题思路类似上述的 3 个数之和，区别是这次我们固定左边 2 个元素。

java 实现如下：

```java
public List<List<Integer>> fourSum(int[] nums, int target) {
    // 大小可以优化
    List<List<Integer>> resultList = new ArrayList<>(nums.length);
    //1. 排序
    Arrays.sort(nums);
    //2. 类似双指针，固定左边2个元素。
    for(int i = 0; i < nums.length -3; i++) {
        // 跳过 i 的重复元素
        if(i > 0 && nums[i] == nums[i-1]) {
            continue;
        }
        for(int j = i+1; j < nums.length-2; j++) {
            // 确保跳过 j 的重复元素
            if(j > i+1 && nums[j] == nums[j-1])  {
                continue;
            }
            // 双指针法则
            int l = j+1;
            int r = nums.length-1;
            while (l < r) {
                int sum = nums[i]+nums[j]+nums[l]+nums[r];
                // 遍历完所有符合的信息
                if(sum < target) {
                    l++;
                } else if(sum > target) {
                    r--;
                } else {
                    List<Integer> result = Arrays.asList(nums[i], nums[j], nums[l], nums[r]);
                    resultList.add(result);
                    // 跳过重复的元素
                    while (l < r && nums[l] == nums[l+1]) {
                        l++;
                    }
                    while(l < r && nums[r] == nums[r-1]) {
                        r--;
                    }
                    l++;
                    r--;
                }
            }
        }
    }
    return resultList;
}
```

经过 3sum 之后，你对自己的实现信心十足。

效果对比打败了 49.10% 的用户。WTF! 

其实答案也很简单，因为大家和你一样已经学会了这种解法。

那么，我们还能优化吗？

### 优化方法

思路：

主要是可以快速返回的一些优化

（1）对于长度小于 4 的数组，直接返回

（2）如果在当前循环中 max 小于 target，或者 min 大于 target 其实也可以快速跳过。

java 实现：

```java
public List<List<Integer>> fourSum(int[] nums, int target) {
    //1.1 快速返回
    if(nums.length < 4) {
        return Collections.emptyList();
    }
    // 大小可以优化
    List<List<Integer>> resultList = new ArrayList<>(nums.length);
    //2. 排序
    Arrays.sort(nums);
    //1.2 范围判断
    final int length = nums.length;
    int min = nums[0] + nums[1] + nums[2] + nums[3];
    int max = nums[length-1] + nums[length-2] + nums[length-3] + nums[length-4];
    if(min > target || max < target) {
        return resultList;
    }
    //3. 类似双指针，固定左边2个元素。
    for(int i = 0; i < length -3; i++) {
        // 跳过 i 的重复元素
        if(i > 0 && nums[i] == nums[i-1]) {
            continue;
        }
        for(int j = i+1; j < length-2; j++) {
            // 确保跳过 j 的重复元素
            if(j > i+1 && nums[j] == nums[j-1])  {
                continue;
            }
            // 双指针法则
            int l = j+1;
            int r = length-1;
            // 快速跳过
            int minInner = nums[i] + nums[j] + nums[j+1] + nums[j+2];
            int maxInner = nums[i] + nums[j] + nums[r-1] + nums[r];
            if(minInner > target || maxInner < target) {
                continue;
            }
            while (l < r) {
                int sum = nums[i]+nums[j]+nums[l]+nums[r];
                // 遍历完所有符合的信息
                if(sum < target) {
                    l++;
                } else if(sum > target) {
                    r--;
                } else {
                    List<Integer> result = Arrays.asList(nums[i], nums[j], nums[l], nums[r]);
                    resultList.add(result);
                    // 跳过重复的元素
                    while (l < r && nums[l] == nums[l+1]) {
                        l++;
                    }
                    while(l < r && nums[r] == nums[r-1]) {
                        r--;
                    }
                    l++;
                    r--;
                }
            }
        }
    }
    return resultList;
}
```

效果：

超过了 92.32% 的提交，勉强过关。

```
Runtime: 5 ms, faster than 92.21% of Java online submissions for 4Sum.
Memory Usage: 39.9 MB, less than 56.17% of Java online submissions for 4Sum.
```

## ksum

### 题目

经过了上面的 2sum/3sum/4sum 的洗礼，我们现在将这道题做下推广。

如何求 ksum？

### 思路

其实所有的这种解法都可以转换为如下的两个问题：

（1）sum 问题

（2）将 k sum 问题转换为 k-1 sum 问题

### 示例代码

```java
/**
 * 对 k 个数进行求和
 * @param nums 数组
 * @param target 目标值
 * @param k k
 * @param index 下标
 * @return 结果类表
 * @since v1
 */
public List<List<Integer>> kSum(int[] nums, int target, int k, int index) {
    int len = nums.length;
    List<List<Integer>> resultList = new ArrayList<>();
    if (index >= len) {
        return resultList;
    }
    if (k == 2) {
        int i = index, j = len - 1;
        while (i < j) {
            //find a pair
            if (target - nums[i] == nums[j]) {
                List<Integer> temp = new ArrayList<>();
                temp.add(nums[i]);
                temp.add(target - nums[i]);
                resultList.add(temp);
                //skip duplication
                while (i < j && nums[i] == nums[i + 1]) {
                    i++;
                }
                while (i < j && nums[j - 1] == nums[j]) {
                    j--;
                }
                i++;
                j--;
                //move left bound
            } else if (target - nums[i] > nums[j]) {
                i++;
                //move right bound
            } else {
                j--;
            }
        }
    } else {
        for (int i = index; i < len - k + 1; i++) {
            //use current number to reduce ksum into k-1 sum
            List<List<Integer>> temp = kSum(nums, target - nums[i], k - 1, i + 1);
            if (temp != null) {
                //add previous results
                for (List<Integer> t : temp) {
                    t.add(0, nums[i]);
                }
                resultList.addAll(temp);
            }
            while (i < len - 1 && nums[i] == nums[i + 1]) {
                //skip duplicated numbers
                i++;
            }
        }
    }
    return resultList;
}
```

## 参考资料

https://leetcode-cn.com/problems/two-sum

https://leetcode.com/problems/two-sum/discuss/3/Accepted-Java-O(n)-Solution

https://leetcode-cn.com/problems/3sum

https://leetcode-cn.com/problems/4sum

https://leetcode.com/problems/4sum/discuss/8609/My-solution-generalized-for-kSums-in-JAVA

* any list
{:toc}