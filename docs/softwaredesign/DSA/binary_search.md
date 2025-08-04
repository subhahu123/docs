# Binary search

Classic binary search, find a number (no dupes):

```python
def search(self, nums: List[int], target: int) -> int:
    left = 0
    right = len(nums)

    while left < right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        if nums[mid] < target:
            left = mid+1
        else:
            right = mid

    # Left would be the insertion point
    return left
```

If we have dupes, we can find insertion point. This find leftmost index if dupes.

```python
def binary_search(arr, target):
    left = 0
    right = len(arr)
    while left < right:
        mid = (left + right) // 2
        if arr[mid] >= target:
            right = mid
        else:
            left = mid + 1

    return left
```

With bisect:

```python
bisect.bisect_left(a, x)
```

To find directly after:

```python
def binary_search(arr, target):
    left = 0
    right = len(arr)
    while left < right:
        mid = (left + right) // 2
        if arr[mid] > target:
            right = mid
        else:
            left = mid + 1

    return left
```

Finds insertion point to retain sorted order

```python
bisect.bisect_right(a, x)
```

Similar to bisect_left(), but returns an insertion point which comes after (to the right of) any existing entries of x in a.

