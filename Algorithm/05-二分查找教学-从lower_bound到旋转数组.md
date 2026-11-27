# 二分查找：从 lower_bound 到旋转数组

> 我是小翊，19 岁，全干 AI 应用开发工程师。  
> 二分查找是「简单到人人会写、难到人人写错」的经典。核心就一句：**每次排除一半**。  
> 这一篇讲清二分的本质、`lower_bound` 万能模板、边界处理，再配两道经典题（查首尾位置、搜索旋转数组）。  
> Let's use AI to create a better world。

---

## 目录

- [一、二分查找的本质](#一二分查找的本质)
- [二、lower_bound 万能模板（重点）](#二lower_bound-万能模板重点)
- [三、题 1：在排序数组中查找首尾位置](#三题-1在排序数组中查找首尾位置)
- [四、题 2：搜索旋转排序数组](#四题-2搜索旋转排序数组)
- [五、二分边界速查表](#五二分边界速查表)

---

## 一、二分查找的本质

**二分查找 = 在能判断「目标在中间点哪一侧」的前提下，每次排除一半的搜索策略。**

使用前提（重点）：
- 整体符合**单调性**（有序或满足某种单调关系）。
- 存在一个**临界值**：小于它满足某条件，大于它不满足（或反过来）时，就能用二分定位到这个临界数字。

> 一句话：**只要「能判断答案在中间点左边还是右边」，就能二分。不只是有序数组——「单调的答案区间」也能二分。**

---

## 二、lower_bound 万能模板（重点）

`lower_bound(x)`：返回**第一个 ≥ x 的位置**。这是二分最核心的模板，很多题都靠它。

```python
def lower_bound(nums, x):
    left, right = 0, len(nums)  # 注意 right 是 len(nums)，不是 len(nums)-1

    while left < right:
        mid = (left + right) // 2

        if nums[mid] < x:
            left = mid + 1
        else:
            right = mid

    return left
```

**模板关键（必背，易错）**：

| 要点 | 说明 |
| --- | --- |
| `right = len(nums)` | 半开区间 `[left, right)`，答案可能等于 `len(nums)`（都小于 x） |
| 循环条件 `left < right` | 不是 `<=`，因为半开区间 |
| `nums[mid] < x` | 严格小于，用 `<` 不是 `<=` |
| `left = mid + 1` | mid 不可能是答案时，跳过它 |
| `right = mid` | mid 可能是答案时，保留它 |

> 记忆：**找「第一个 ≥ x」，条件用 `< x`，满足就 `left = mid + 1`，否则 `right = mid`。** 这个模板背熟，二分边界题全都不怕。

---

## 三、题 1：在排序数组中查找首尾位置

**题目**：在升序数组里找 `target` 第一次和最后一次出现的位置，找不到返回 `[-1, -1]`。

**思路**：两次 `lower_bound` —— 第一次找 `target`，第二次找 `target+1` 再减一。

```python
from typing import List

class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        def lower_bound(x: int) -> int:
            left, right = 0, len(nums)

            while left < right:
                mid = (left + right) // 2

                if nums[mid] < x:
                    left = mid + 1
                else:
                    right = mid

            return left

        left = lower_bound(target)           # 第一个 >= target 的位置
        right = lower_bound(target + 1) - 1  # 第一个 >= target+1 的位置 - 1 = 最后一个 target

        if left <= right:
            return [left, right]

        return [-1, -1]
```

**核心理解（易错）**：
- `lower_bound(target)` = 第一个 `target` 的下标。
- `lower_bound(target + 1)` = 第一个**大于** `target` 的下标，减一就是**最后一个** `target`。
- 用 `left <= right` 判断是否存在（不存在时 `left` 会跑到 `right+1`）。

> 一句话：**首位置 = lower_bound(target)，尾位置 = lower_bound(target+1) - 1。** 一个模板搞定区间查找。

---

## 四、题 2：搜索旋转排序数组

**题目**：一个升序数组在某个点旋转了（如 `[4,5,6,7,0,1,2]`），在其中找 `target`。

**思路**：二分，但先判断「哪一半是有序的」，再决定往哪边搜。

```python
from typing import List

class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1

        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] == target:
                return mid

            if nums[left] <= nums[mid]:
                # 左半边有序
                if nums[left] <= target < nums[mid]:
                    right = mid - 1
                else:
                    left = mid + 1
            else:
                # 右半边有序
                if nums[mid] < target <= nums[right]:
                    left = mid + 1
                else:
                    right = mid - 1

        return -1
```

**核心理解（易错）**：
- 旋转后数组「一半有序、一半可能无序」。
- 每次先判断 `nums[left] <= nums[mid]` 判断**左半边是否有序**。
- 若 `target` 落在有序的那半边范围内，就在那边搜；否则去另一边。
- `mid = left + (right - left) // 2` 是为了**防止 left + right 溢出**（虽然 Python 不会溢出，但这是好习惯，也利于迁移到 Java/C++）。

> 一句话：**旋转数组的二分 = 普通二分 + 「先判断哪半边有序」这一步。**

---

## 五、二分边界速查表

| 情况 | 写法 |
| --- | --- |
| 找第一个 ≥ x（lower_bound） | 半开区间 `[left, right)`，条件 `< x` |
| 找第一个 > x | `lower_bound(x + 1)` |
| 找最后一个 < x | `lower_bound(x) - 1` |
| 普通精确查找 | 闭区间 `[left, right]`，条件 `<=` |
| 防止 mid 溢出 | `left + (right - left) // 2` |

---

**一句话总结**：二分核心是「**每次排除一半**」。背熟 `lower_bound` 模板（半开区间 + `< x`），首尾位置、旋转数组这些变体都能拆解成「换个判断条件」。

**文末预告**：下一篇「数论」——gcd、逆元、筛质数、快速幂、矩阵快速幂，把「大数取模」那套一次讲透。

Let's use AI to create a better world。
