## 题目描述

输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

## 题解

```java
import java.util.*;
public class Solution {
    public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
        if(k>input.length || k<=0) return new ArrayList<>();
        PriorityQueue<Integer> priorityQueue = new PriorityQueue<>(k, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o2.compareTo(o1);
            }
        });
        int i = 0;
        for(; i < k; i++) priorityQueue.offer(input[i]);
        for(; i < input.length; i++){
            if(priorityQueue.peek() > input[i]){
                priorityQueue.poll();
                priorityQueue.offer(input[i]);
            }
        }
        ArrayList<Integer> arrayList = new ArrayList();
        while(!priorityQueue.isEmpty()) arrayList.add(priorityQueue.poll());
        Collections.sort(arrayList);
        return arrayList;
    }
}
```

