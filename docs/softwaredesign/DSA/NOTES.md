# DSA

## Subarrays

It's possible to get number of subarrays ENDING at a fixed location, starting from a non fixed location:

```
[0, 1, 2, 3]
```

Subarrays ending at `i=3`, starting `i=1`

There are `1,2,3`, `2,3`, `3` - 3 total.

Compute via `end - start + 1`

## Trees

Time:
* Usually `O(n)` (visit each node)
* `O(n*k)` where k is work at each node

Space:
* `O(n)` - Straight line, all on stack
* `O(longn)` - balanced tree

Depth: `O(logn)`

### DFS

Pre-order: Operate on node on way down. (Logic before - pre)

```python
def preorder_dfs(node):
    if not node:
        return

    print(node.val)
    preorder_dfs(node.left)
    preorder_dfs(node.right)
    return
```

In an increasing tree (root < child) this gives us increasing order.

In-order: Order on the way up. (logic in middle - in)

In BST translates to in order.

Do left, then print val, then right.

```python
def inorder_dfs(node):
    if not node:
        return

    inorder_dfs(node.left)
    print(node.val)
    inorder_dfs(node.right)
    return
```

Post-order: Go to leaves before anything else. (logic after - post)

```python
def postorder_dfs(node):
    if not node:
        return

    postorder_dfs(node.left)
    postorder_dfs(node.right)
    print(node.val)
    return
```

### BFS

`O(V+E)`

(balanced tree) The final level in a perfect binary tree has n/2 nodes, so BFS uses `O(n)` space

To traverse levels, use:

```python
    def largestValues(self, root: Optional[TreeNode]) -> List[int]:
        if root is None:
            return []
        q = deque([root])
        lrg = []
        while q:
            lvl = len(q)
            for _ in range(lvl): # Traverses level
                n = q.popleft()
                if n.left is not None:
                    q.append(n.left)
                if n.right is not None:
                    q.append(n.right)

        return lrg
```

### BST

## Graphs

