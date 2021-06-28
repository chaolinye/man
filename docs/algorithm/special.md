# 特殊的数据结构和算法

## 单调栈

用于解决 `Next Greater Number` 一类算法

关键点:

- 从后往前入栈，入栈前通过 pop 保证栈内元素单调递增或者递减
- 当前栈顶即是 Next Greater Number

```java
    public int[] nextGreaterNumber(int[] arr) {
        int[] result = new int[arr.length];
        Deque<Integer> stack = new LinkedList<>();
        for (int i = arr.length - 1; i >= 0; i--) {
            while(!stack.isEmpty() && stack.peek() <= arr[i]) {
                stack.poll();
            }
            result[i] = stack.isEmpty() ? -1 : stack.peek();
            stack.push(arr[i]);
        }
        return result;
    }
```

## 单调队列

用于解决 `求滑动窗口最大值` 的问题

```java
    public int[] maxSlidingWindow(int[] nums, int k) {
        Deque<Integer> deque = new LinkedList<>();
        List<Integer> result = new ArrayList<>();

        for (int i = 0; i < nums.length; i++) {
            if (i < k - 1) {
                deque.addLast(nums[i]);
            } else {
                while (!deque.isEmpty() && deque.peekLast() < nums[i]) {
                    deque.pollLast();
                }
                deque.addLast(nums[i]);
                result.add(deque.getFirst());
                if (nums[i - k + 1] == deque.getFirst()) {
                    deque.removeFirst();
                }
            }
        }
        return result.stream().mapToInt(Integer::intValue).toArray();
    }
```

## 前缀和