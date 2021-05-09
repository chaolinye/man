# 链表

## 反转链表

### 迭代反转

```java
    ListNode reverse(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        while (cur != null) {
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }
```

### 递归反转

```java
    ListNode reverse(ListNode head) {
        if (head.next == null) {
            return head;
        }
        ListNode reverseHead = reverse(head.next);
        head.next.next = head;
        head.next = null;
        return reverseHead;
    }
```

### 反转前 n 个节点

```java
    private ListNode next = null;

    ListNode reverseN(ListNode head, int n) {
        if (n == 1) {
            next = head.next;
            return head;
        }
        ListNode reverseHead = reverseN(head.next, n - 1);
        head.next.next = head;
        head.next = next;
        return reverseHead;
    }
```

### 反转一部分节点

```java
    ListNode reverseBetween(ListNode head, int m, int n) {
        if (m == 1) {
            return reverseN(head, n);
        }
        head.next = reverseBetween(head, m - 1, n - 1);
        return head;
    }
```