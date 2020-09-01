# 树

#### [116. 填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

给定一个完美二叉树，其所有叶子节点都在同一层，每个父节点都有两个子节点。二叉树定义如下：

struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}
填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 NULL。

初始状态下，所有 next 指针都被设置为 NULL。

示例：

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/15/116_sample.png)

```Java
//此题解采用了深度优先遍历给每个节点添加next结点主要使对右节点的判断要判断它的父节点是否有next结点如果没有那么右节点的next结点为null，如果有那么为父节点的next结点的左节点
class Solution {
    public Node connect(Node root) {
        dfs(root,null);
        return root;
    }
    public void dfs(Node root,Node next){
        if(root!=null){
            root.next = next;
            dfs(root.left,root.right);
            dfs(root.right,root.next!=null?root.next.left:null);
        }
    }
}
```

```java
//此题解使用的是层序遍历的方法来添加每一层节点然后根据每一层的节点个数来遍历添加next结点，注意的是在每一层最后一个结点时和前面的结点不同此时的最后一个结点的next为null。
class Solution {
    public Node connect(Node root) {
        Queue<Node> queue = new LinkedList<Node>();
        if(root==null) return root;
        queue.add(root);
        while(!queue.isEmpty()){
            int j = queue.size();
            for(int i=0;i<j;i++){
                Node Q = queue.poll();
                
                if(queue.peek()!=null&&i<j-1){
                    Q.next = queue.peek();
                }
            if(Q.left!=null) queue.add(Q.left);
            if(Q.right!=null) queue.add(Q.right);
            }
        }
        return root;
    }
}
```



#### [113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

给定一个二叉树和一个目标和，找到所有从根节点到叶子节点路径总和等于给定目标和的路径。

说明: 叶子节点是指没有子节点的节点。

示例:
给定如下二叉树，以及目标和 sum = 22，

              5
             / \
            4   8
           /   / \
          11  13  4
         /  \    / \
        7    2  5   1
返回:

```
[
   [5,4,11,2],
   [5,8,4,5]
]
```

```java
/*此题解使用的是深度优先遍历回溯的方法来解决的。
本题使用递归+回溯的思想 DFS 遍历整个二叉树求出每条目标路径

首先创建 lists 结果集用来接收找到的目标路径，然后定义递归方法 dfs 寻找每条路径上满足题意的路径

考虑特殊情况：只有根节点或者遍历到了叶子结点的时候，就将该临时路径数组 list 放入结果集中（每次都初始路径 list）

其他情况就按照递归的方法依次遍历左右子树各个结点

其中利用回溯思想将每次目标路径的数组元素回退到上一个结点，之后遍历另一条边的结点寻找更多的可能性

所有遍历结束返回结果集 lists 即可*/
class Solution {
    
    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        List<List<Integer>> lists = new ArrayList<>();
        List<Integer> list = new ArrayList<>();
        dfs(root,sum,list,lists);
        return lists;
    }
    public void dfs(TreeNode node,int sum,List<Integer> list,List<List<Integer>> lists){
        if(node==null) return;
        list.add(node.val);
        if(sum==node.val&&node.right==null&&node.left==null){
            lists.add(new ArrayList<>(list));
        }
        sum-=node.val;
        if(node.left!=null) dfs(node.left,sum,list,lists);
        if(node.right!=null) dfs(node.right,sum,list,lists);
        list.remove(list.size()-1);
    }
}
```

#### [1379. 找出克隆二叉树中的相同节点](https://leetcode-cn.com/problems/find-a-corresponding-node-of-a-binary-tree-in-a-clone-of-that-tree/)

给你两棵二叉树，原始树 original 和克隆树 cloned，以及一个位于原始树 original 中的目标节点 target。

其中，克隆树 cloned 是原始树 original 的一个 副本 。

请找出在树 cloned 中，与 target 相同 的节点，并返回对该节点的引用（在 C/C++ 等有指针的语言中返回 节点指针，其他语言返回节点本身）。

注意：

你 不能 对两棵二叉树，以及 target 节点进行更改。
只能 返回对克隆树 cloned 中已有的节点的引用。
进阶：如果树中允许出现值相同的节点，你将如何解答？

示例 1:

![img](https://assets.leetcode.com/uploads/2020/02/21/e1.png)

输入: tree = [7,4,3,null,null,6,19], target = 3
输出: 3
解释: 上图画出了树 original 和 cloned。target 节点在树 original 中，用绿色标记。答案是树 cloned 中的黄颜色的节点（其他示例类似）。

```java
//先根遍历,后序遍历，中序遍历都可以，通过判断original和target是否相等将在同样位置的cloned节点输出
class Solution {
    public final TreeNode getTargetCopy(final TreeNode original, final TreeNode cloned, final TreeNode target) {
        if (original == null){
            return null;
        }
        if (original == target){
            return cloned;
        }
        // 递归左子树
        TreeNode res = getTargetCopy(original.left,cloned.left,target);
        if (res != null){         //如果不加判断是否不为null才返回，可能直接在遍历完一边的树后就返回结点。
            return res;
        }
        // 递归右子树
        res = getTargetCopy(original.right,cloned.right,target);
        if (res != null){
            return res;
        }
        return null;
    }
}
```

```java
//此题题解时使用了迭代先序遍历的方法，在压入结点的时候，根节点向左节点遍历，当弹出结点的时候，根节点向右遍历，就会形成先序遍历
class Solution {
    public final TreeNode getTargetCopy(final TreeNode original, final TreeNode cloned, final TreeNode target) {
        Deque<TreeNode> stack = new LinkedList<>();
        Deque<TreeNode> clonedStack = new LinkedList<>();
        TreeNode node = original;
        TreeNode cloneTarget = cloned;
        while (node != null || !stack.isEmpty()){
            if (node != null){
                if (node == target){
                    return cloneTarget;// 找到目标，返回
                }
                // 节点不空，走左子树
                stack.push(node);
                node = node.left;
                clonedStack.push(cloneTarget);
                cloneTarget = cloneTarget.left;
            }else {
                // 节点空了，进入右子树
                node = stack.pop();
                node = node.right;
                cloneTarget = clonedStack.pop();
                cloneTarget = cloneTarget.right;
            }
        }
        return null;
    }
}
```

#### [70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

假设你正在爬楼梯。需要 n 阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

注意：给定 n 是一个正整数。

示例 1：

输入： 2
输出： 2
解释： 有两种方法可以爬到楼顶。
1.  1 阶 + 1 阶
2.  2 阶
示例 2：

输入： 3
输出： 3
解释： 有三种方法可以爬到楼顶。
1.  1 阶 + 1 阶 + 1 阶
2.  1 阶 + 2 阶
3.  2 阶 + 1 阶

```java
//尾递归可以减少堆栈的使用，如果使用正常的递归会导致溢出，可参考https://www.cnblogs.com/Anker/archive/2013/03/04/2943498.html
//尾递归主要是将生成的结果存储到参数中继续进行递归
class Solution {
        public static int climbStairs(int n) {
        return Fibonacci(n, 1, 1);
    }

    public static int Fibonacci(int n, int a, int b) {
        if (n <= 1)
            return b;
        return Fibonacci(n - 1, b, a + b);
    }
}
```

```java
//使用动态规划的方法解决，还可以继续将数组优化为变量进行存储
class Solution {
    public static int climbStairs(int n) {
        if(n<=1) return 1;
        int dp[] = new int[n];
        dp[0] = 1;
        dp[1] = 2;
        for(int i=2;i<n;i++){
            dp[i] = dp[i-1] + dp[i-2];
        }
        return dp[n-1];
    }
}
```

#### [面试题 04.03. 特定深度节点链表](https://leetcode-cn.com/problems/list-of-depth-lcci/)

给定一棵二叉树，设计一个算法，创建含有某一深度上所有节点的链表（比如，若一棵树的深度为 D，则会创建出 D 个链表）。返回一个包含所有深度的链表的数组。

 

示例：

输入：[1,2,3,4,5,null,7,8]

        1
       /  \ 
      2    3
     / \    \ 
    4   5    7
   /
  8

输出：[[1],[2,3],[4,5,7],[8]]

```java
//特定深度节点链表,层序遍历然后一层一层的将栈中的元素写入链表中，本题主要收获就是生成链表添加链表信息
	public static ListNode[] listOfDepth(TreeNode tree){
		if(tree == null) return null;
		List<ListNode> resArr = new ArrayList<>();
		Queue<TreeNode> queue = new LinkedList<>();
		queue.add(tree);
		while(!queue.isEmpty()){
			int size = queue.size();
			ListNode node = new ListNode(0);//头节点
			ListNode head = node;
			for(int i=0;i<size;i++){
				TreeNode p = queue.poll();
				ListNode n = new ListNode(p.val);
				node.next = n;
				node = n;
				if(p.left != null) queue.add(p.left);
				if(p.right != null) queue.add(p.right);
			}
			resArr.add(head.next);
		}
		return resArr.toArray(new ListNode[resArr.size()]);
	}
```

#### [654. 最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)

给定一个不含重复元素的整数数组。一个以此数组构建的最大二叉树定义如下：

二叉树的根是数组中的最大元素。
左子树是通过数组中最大值左边部分构造出的最大二叉树。
右子树是通过数组中最大值右边部分构造出的最大二叉树。
通过给定的数组构建最大二叉树，并且输出这个树的根节点。

 

示例 ：

输入：[3,2,1,6,0,5]
输出：返回下面这棵树的根节点：

      6
    /   \
   3     5
    \    / 
     2  0   
       \
        1

```java
//本题解使用的是创建中序二叉树的算法，找到最大的然后分左右递归建立二叉树
class Solution {

    int[] nums;

    public TreeNode constructMaximumBinaryTree(int[] nums) {
        this.nums = nums;
        return helper(0, nums.length);
    }

    public TreeNode helper(int start, int end) {
        if (start >= end) {
            return null;
        }
        int max = nums[start];
        int index = start;
        for (int i = start + 1; i < end; i++) {
            if (max < nums[i]) {
                max = nums[i];
                index = i;
            }
        }
        TreeNode root = new TreeNode(max);
        root.left = helper(start, index);
        root.right = helper(index + 1, end);
        return root;
    }
}
```

#### [1300. 转变数组后最接近目标值的数组和](https://leetcode-cn.com/problems/sum-of-mutated-array-closest-to-target/)

给你一个整数数组 arr 和一个目标值 target ，请你返回一个整数 value ，使得将数组中所有大于 value 的值变成 value 后，数组的和最接近  target （最接近表示两者之差的绝对值最小）。

如果有多种使得和最接近 target 的方案，请你返回这些整数中的最小值。

请注意，答案不一定是 arr 中的数字。

示例 1：

输入：arr = [4,9,3], target = 10
输出：3
解释：当选择 value 为 3 时，数组会变成 [3, 3, 3]，和为 9 ，这是最接近 target 的方案。
示例 2：

输入：arr = [2,3,5], target = 10
输出：5
示例 3：

输入：arr = [60864,25176,27249,21296,20204], target = 56803
输出：11361

```java
class Solution {
    public int findBestValue(int[] arr, int target) {
        Arrays.sort(arr);
        int n = arr.length;
        int prefix[] = new int[n+1];//计算每一个前缀数组的和
        for(int i=1;i<=n;i++){
            prefix[i] = prefix[i-1]+arr[i-1];
        }
        int r = arr[n-1];
        int ans = 0,diff = target;
        for(int i=1;i<=r;i++){
            int index = Arrays.binarySearch(arr,i);
            if(index<0){
                index = -index-1;/*搜索键的索引，如果它包含在数组中; 否则， (-(insertion point) - 1) 。 如果阵列中的所有元素都小于指定键的第一元件比所述键时，或a.length的索引： 插入点被定义为将键插入到阵列的点。 请注意，这确保当且仅当找到该键时返回值将为> = 0。 */
            }
            int cur = prefix[index] + (n-index)*i;
            if(Math.abs(cur-target)<diff){
                ans = i;
                diff = Math.abs(cur-target);
            }
        }
        return ans;
    }
}
```

