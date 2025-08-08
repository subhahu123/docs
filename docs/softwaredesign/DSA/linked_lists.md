# Linked Lists

## Slow and Fast Pointers

Can be used to find the middle or a cycle. If a cycle occurs the fast pointer will eventually meet the slow.

Find middle:

```python
class Node:
    def __init__(self, val):
        self.val = val
        self.next = None

    def __str__(self):
        return str(self.val)

print("Creating even linked list (length 4)")
# [1, 2, 3, 4]
head = Node(1)
n = head
for i in range(2, 5):
    n.next = Node(i)
    n = n.next

n = head
while n:
    print(n.val)
    n = n.next

fast = head
slow = head
while fast and fast.next:
    fast = fast.next.next
    slow = slow.next

print("Middle: ", slow.val)

print("Creating odd linked list (length 5)")
# [1, 2, 3, 4, 5]
head = Node(1)
n = head
for i in range(2, 6):
    n.next = Node(i)
    n = n.next

n = head
while n:
    print(n.val)
    n = n.next

fast = head
slow = head
while fast and fast.next:
    fast = fast.next.next
    slow = slow.next

print("Middle: ", slow.val)
```

## Reversal

```python
# Reverse
n = head
prev = None
while n:
    # Create temp for next
    nextemp = n.next
    # Set next to prev
    n.next = prev
    # Update prev
    prev = n
    # Go to next node
    n = nextemp

print("\nReversed linked list:")
# Last ends up as prev
n = prev
while n:
    print(n.val)
    n = n.next
```
