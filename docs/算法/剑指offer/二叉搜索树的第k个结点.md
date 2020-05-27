## 题目描述

给定一棵二叉搜索树，请找出其中的第k小的结点。例如， （5，3，7，2，4，6，8）  中，按结点数值大小顺序第三小结点的值为4。

## 题解

```java
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    private int index;
    TreeNode KthNode(TreeNode pRoot, int k){
        index = k;
        TreeNode result = find(pRoot);
        return result;
    }
    private TreeNode find(TreeNode pRoot){
        if(pRoot==null) return null;
        TreeNode left = find(pRoot.left);
        if(left!=null) return left;
        index--;
        if(index==0) return pRoot;
        TreeNode right = find(pRoot.right);
        return right;
    }
}
```

