## 题目描述

输入一个链表，输出该链表中倒数第k个结点。

## 题解

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode FindKthToTail(ListNode head,int k) {
        if(k<=0) return null;
        if(head==null) return null;
        ListNode kNode = null;
        int count = 1;
        ListNode index = head;
        while(index!=null){
            if(count==k) kNode = head;
            else if(count>k) kNode = kNode.next;
            count++;
            index = index.next;
        }
        return kNode;
    }
}
```

