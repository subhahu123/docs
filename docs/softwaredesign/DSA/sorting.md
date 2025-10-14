# Sorting

## Mergesort


Ref:
* [Mergesort](https://youtu.be/mB5HXBb_HY8?si=qfJIGZxus6SuCOD5)
* [Mergesort](https://www.youtube.com/watch?v=4VqmGXwpLqc)

Recursively break original array into multiple arrays cutting in half every time. Return if we hit a base case of array is empty or len=1.

After calling merge sort recursively on the two different sides, we have two sorted regions and we call merge on them.

```python
def sortArray(self, nums: List[int]) -> List[int]:
    def merge(a1, a2):
        n, m = len(a1), len(a2)
        i, j = 0, 0

        res = []

        while i < n or j < m:
            if i == n:
                res.append(a2[j])
                j+=1
            elif j == m:
                res.append(a1[i])
                i+=1
            else:
                if a1[i] < a2[j]:
                    res.append(a1[i])
                    i+=1
                else:
                    res.append(a2[j])
                    j+=1

        return res


    def mergeSort(arr):
        n = len(arr)
        # Stop when len 1
        if n <= 1:
            return arr

        m = n // 2

        # Recursively sort left and right

        a1 = mergeSort(arr[:m])
        a2 = mergeSort(arr[m:])

        # Two sides now sorted, lets merge
        return merge(a1, a2)

    return mergeSort(nums)
```

We do end up using extra space for the temporary arrays, but it ends up being linear space because the maximum amount of space we are keeping is the full array in temp variables
