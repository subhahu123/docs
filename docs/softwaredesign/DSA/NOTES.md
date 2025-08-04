# Misc

## Subarrays

It's possible to get number of subarrays ENDING at a fixed location, starting from a non fixed location:

```
[0, 1, 2, 3]
```

Subarrays ending at `i=3`, starting `i=1`

There are `1,2,3`, `2,3`, `3` - 3 total.

Compute via `end - start + 1`
