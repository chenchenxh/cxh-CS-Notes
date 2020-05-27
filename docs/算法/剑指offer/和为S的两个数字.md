## 题目描述

输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。

## 输出描述:

```
对应每个测试案例，输出两个数，小的先输出。
```

## 题解

```java
import java.util.*;
public class Solution {
    public ArrayList<Integer> FindNumbersWithSum(int [] array,int sum) {
        if(array.length==0) return new ArrayList<>();
        int index1 = 0, index2 = 0;
        while(array[index1]+array[index2]<sum) index2++;
        if(!(array[index1]+array[index2]==sum)) {
            index1++;index2--;
            while(index2>index1){
                while(index2>index1 && array[index1]+array[index2]<sum) index1++;
                if(index2>index1 && array[index1]+array[index2]==sum) break;
                while(index2>index1 && array[index2]+array[index1]>sum) index2--;
                if(index2>index1 && array[index1]+array[index2]==sum) break;
            }
        }
        if(array[index1]+array[index2]==sum){
            ArrayList<Integer> list = new ArrayList<>();
            list.add(array[index1]);list.add(array[index2]);
            return list;
        }
        return new ArrayList<>();
    }
}
```

