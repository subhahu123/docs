# DP

## Kadane's Algorithm

Can be used to find the maximum sum of a subarray.

Basically store in a variable, the best sum ending at an index. This will either be the best sum at the last index plus the current value, or it will be the current value. So if the previous best does not improve by adding the current, we will essentially reset.


