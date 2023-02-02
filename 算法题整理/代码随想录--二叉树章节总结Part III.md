## 代码随想录--二叉树章节总结Part III

### 1.Leetcode106 从中序与后序遍历序列构造二叉树

给定两个整数数组 inorder 和 postorder ，其中 inorder 是二叉树的中序遍历， postorder 是同一棵树的后序遍历，请你构造并返回这颗 二叉树 。

**解题思路1:** 递归方法。利用后序遍历解决。

* 首先判断如果数组没没有元素，则返回空
* 如果数组中有元素，则取后序遍历数组中的最后一个元素，创建根结点。
* 然后根据根结点的值，找到这个值在中序遍历中的下标。根据下标，将中序遍历分为左右子树。
* 然后递归的根据分开后的遍历序列，构建左右子树

> 注意，在后序遍历中，无法根据根结点来判断哪些元素属于左子树，哪些属于右子树。但是左右子树划分的长度和中序遍历是一致的。
>
> ![image-20230128114451284](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230128114451284.png)

```java
public TreeNode buildTree(int[] inorder, int[] postorder) {
  	return f(inorder, postorder);
}
private TreeNode f(int[] inorder, int[] postorder) {
    if(postorder.length == 0) return null;
    // 处理根结点 从后序遍历中取出最后的元素，创建结点
    TreeNode root = new TreeNode(postorder[postorder.length - 1]);
    int index;
    for (index = 0; index < inorder.length; index++) {
      	if (inorder[index] == root.val) break;
    }
    // 左
    root.left = f(Arrays.copyOfRange(inorder, 0, index), Arrays.copyOfRange(postorder, 0, index));
    // 右
    root.right = f(Arrays.copyOfRange(inorder, index + 1, inorder.length), Arrays.copyOfRange(postorder, index, postorder.length - 1));
    // 根
    return root;
}
```

> 此外需要注意的是，`Arrays.copyOfRange()`的区间是左闭右开的

### 2.Leetcode105 从中序与前序遍历序列构造二叉树

给定两个整数数组 preorder 和 inorder ，其中 preorder 是二叉树的先序遍历， inorder 是同一棵树的中序遍历，请构造二叉树并返回其根节点。

**解题思路：** 和上面的方法一样，采用递归的后序遍历方法。

* 首先如果两个数组任意一个为空，则直接返回空
* 然后从先序遍历数组中取出第一个元素，用来创建根结点
* 然后找到根结点在中序遍历中的位置，将遍历序列进行划分
* 然后递归的构建根结点的左右子树
* 最后返回根结点

```java
public TreeNode buildTree(int[] preorder, int[] inorder) {
  	return f(inorder, preorder);
}

private TreeNode f(int[] inorder, int[] preorder) {
    if (preorder.length == 0 || inorder.length == 0) return null;
    // 从前序遍历序列中取出第一个元素作为根结点
    int root_val = preorder[0];
    // 构建根结点
    TreeNode root = new TreeNode(root_val);

    // 查找根结点在中序遍历中的位置
    int index;
    for (index = 0; index < inorder.length; index++) {
      	if (inorder[index] == root_val) break;
    }

    TreeNode left = f(Arrays.copyOfRange(inorder, 0, index), Arrays.copyOfRange(preorder, 1, index + 1));
    TreeNode right = f(Arrays.copyOfRange(inorder, index + 1, inorder.length), Arrays.copyOfRange(preorder, index + 1, preorder.length));

    root.left =left;
    root.right = right;
    return root;
}
```

> 前序遍历和中序遍历的划分：
>
> 注意这里的区间我用的是左闭右开区间，为了和`Arrays.copyOfRange()`保持一致
>
> ![image-20230128120545098](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230128120545098.png)

### 3.Leetcode654 最大二叉树

给定一个不重复的整数数组 nums 。 最大二叉树 可以用下面的算法从 nums 递归地构建:

创建一个根节点，其值为 nums 中的最大值。
递归地在最大值 左边 的 子数组前缀上 构建左子树。
递归地在最大值 右边 的 子数组后缀上 构建右子树。
返回 nums 构建的 最大二叉树 。

示例：

![654.最大二叉树](https://img-blog.csdnimg.cn/20210204154534796.png)

**解题思路：** 利用递归，具体的思路和上面两道题是一样的，只是划分数组的方式不一样而已

* 首先判断数组长度是为0，如果为0，则直接返回null
* 否则将获取到数组中的最大值，根据最大值构建根结点
* 并且根据最大值的下标将数组划分为两部分，利用这两部分分别构建左右子树
* 最后返回根结点即可。

> 需要注意的是边界条件的处理
>
> * 最大值在数组的最后一个元素时，那么构建的右子树应该为空
> * 最大值在数组的第一个元素时，那么构建的左子树为空。
>
> 在代码中需要进行判断，确保不会越界的问题
>
> ![image-20230128132011999](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230128132011999.png)

```java
public TreeNode constructMaximumBinaryTree(int[] nums) {
  	return f(nums);
}

/**
 * 构建二叉树
 * @param nums
 * @return
*/
private TreeNode f(int[] nums) {
    // 如果数组中没有元素，则返回空
    if (nums.length == 0) return null;
    // 获取数组中最大元素下标
    int index = maxNums(nums);
    // 创建根结点
    TreeNode root = new TreeNode(nums[index]);
    // 递归创建左右子树
    TreeNode left = null;
    TreeNode right = null;
    if (index > 0) {
      	left = f(Arrays.copyOfRange(nums, 0, index));
    }
    if (index + 1 < nums.length) {
      	right = f(Arrays.copyOfRange(nums, index + 1, nums.length));
    }

    // 处理根
    root.left = left;
    root.right = right;
    return root;
}

/**
 * 返回数组中最大值的下标
 * @param nums
 * @return
*/
private int maxNums(int[] nums) {
    if (nums.length == 1) return 0;
    int max = nums[0];
    int index = 0;
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] > max) {
            max = nums[i];
            index = i;
        }
    }
    return index;
}
```

### 4.Leetcode617 合并二叉树

给定两个二叉树，想象当你将它们中的一个覆盖到另一个上时，两个二叉树的一些节点便会重叠。

你需要将他们合并为一个新的二叉树。合并的规则是如果两个节点重叠，那么将他们的值相加作为节点合并后的新值，否则不为 NULL 的节点将直接作为新二叉树的节点。

示例如下：

![img](https://assets.leetcode.com/uploads/2021/02/05/merge.jpg)

**解题思路：** 利用递归求解。使用前序遍历，当遍历到结点为空的时候，则结束递归。否则继续递归合并左右子树。

```java
public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
    if (root1 == null) return root2;
    if (root2 == null) return root1;

    // root1 root2 都不为空
    root1.val += root2.val;

    // 递归处理左右子树
    TreeNode left = mergeTrees(root1.left, root2.left);
    TreeNode right = mergeTrees(root1.right, root2.right);
    root1.left = left;
    root1.right = right;
    return root1;
}
```

### 5.Leetcode700 二叉搜索树中的搜索

给定二叉搜索树（BST）的根节点 root 和一个整数值 val。

你需要在 BST 中找到节点值等于 val 的节点。 返回以该节点为根的子树。 如果节点不存在，则返回 null 。

**解题思路：** 直接利用递归前序遍历即可。

* 终止条件是结点为空。
* 根结点处理：如果根结点的值等于val，直接返回根结点。否则根据val的大小去递归搜索左子树或者右子树即可。

```java
public TreeNode searchBST(TreeNode root, int val) {
    // 递归终止条件
    if (root == null) return null;
    // 处理根结点
    if (root.val == val) return root;
    if (root.val > val) return searchBST(root.left, val);
    if (root.val < val) return searchBST(root.right, val);
    return null;
}
```

**解题思路2**：迭代法。从根结点开始遍历，直到遍历到空结点为止。在循环中，根据val和root的值判断搜索方向。

```java
public TreeNode searchBST(TreeNode root, int val) {
    if (root == null) return null;
    TreeNode cur = root;
    while (cur != null) {
        if (cur.val == val) return cur;
        if (cur.val > val) 
            cur = cur.left;
        else if (cur.val < val)
            cur = cur.right;
    }
    return null;
}
```

### 6.Leetcode98 验证平衡二叉树

给你一个二叉树的根节点root，判断其是否是一个有效的二叉搜索树。

有效二叉搜索树定义如下：

* 节点的左子树只包含小于当前节点的数
* 节点的右子树只包含大于当前节点的数
* 所有左子树和右子树自身必须也是二叉搜索树

**错误思路：** 利用递归进行前序遍历，先判断根结点是不是满足上面的条件。如果满足，则继续判断左右子树是不是满足条件。

```java
public boolean isValidBST(TreeNode root) {
    // 如果为空 则返回true
    if (root == null) return true;
    // 如果是叶子结点，则返回true
    if (root.left == null && root.right == null) return true;

    // 如果根只有左子树
    if (root.left != null && root.right == null) {
        boolean flag = root.val > root.left.val;
        return flag && isValidBST(root.left);
    }
    // 如果只有右子树
    if (root.left == null && root.right != null) {
        boolean flag = root.val < root.right.val;
        return flag && isValidBST(root.right);
    }

    boolean flag = (root.val > root.left.val) && (root.val < root.right.val);
    return flag && isValidBST(root.left) && isValidBST(root.right);
}
```

> 上面的代码看似是对的，但是针对下面这种情况
>
> * 二叉树为[5,4,6,null,null,3,7]
> * 3这个结点在就不满足条件
>
> 因为只判断了当前结点和左右孩子。

> 正确思路是：二叉搜索树的中序遍历是严格单调递增的。应该利用这一条性质进行解题

**解题思路1**：利用中序遍历和数组。将中序遍历序列保存到数组中，然后判断这个数组中的元素是不是严格单调递增的即可。

```java
public boolean isValidBST(TreeNode root) {
    ArrayList<Integer> list = new ArrayList<>();
    inorder(root, list);
    System.out.print(list);
    // 注意这里i从1开始
    for (int i = 1; i < list.size(); i++) {
        if (list.get(i-1) >= list.get(i))
          	return false;
    }
    return true;
}
private void inorder(TreeNode root, ArrayList<Integer> list) {
    if (root == null) return;
    inorder(root.left, list);
    list.add(root.val);
    inorder(root.right, list);
}
```

**解题思路2:** 优化思路1，不采用数据进行保存，而是定一个全局遍历用来记录每一次遍历结点的前一个结点，然后这两个结点的val进行比较即可。

```java
private TreeNode preNode = null;
public boolean isValidBST(TreeNode root) {
    if (root == null) return true;
    // 遍历左边
    boolean left = isValidBST(root.left);
    if (left == false) return false;

    // preNode != null 主要是在根结点进入递归的时候preNode还是空 if应该不成立 否则就直接返回false了
    if (preNode != null && preNode.val >= root.val)
        return false;
    preNode = root;
    // 实际上应该是左&&中&&右 但是执行到这一步 左中一定为true 所以省略
    return isValidBST(root.right);
}
```

### 7.Leetcode530 二叉搜索树的最小绝对差

给你一个二叉搜索树的根节点root，返回**树中任意两不同节点值之间的最小差值** 。差值是一个正数，其数值等于两值之差的绝对值。

问题分析：二叉搜索树的中序遍历是单调递增的，那么就说明了最小绝对差只能存与中序遍历两两相邻的两个元素之间，因此只需要获取到中序遍历，然后在两两比较即可。

**解题思路1:** 递归求中序遍历，将遍历序列保存到数组中。然后两个指针遍历数组，每次计算两个指针指向的元素的val差的绝对值，最终找到最小值返回。

```java
public int getMinimumDifference(TreeNode root) {
    ArrayList<Integer> list = new ArrayList<>();
    inorder(root, list);
    int min = Integer.MAX_VALUE;
    for (int i = 1; i < list.size(); i++) {
        int diff = Math.abs(list.get(i-1) - list.get(i));
        if (diff < min)
          	min = diff;
    }
    return min;
}
private void inorder(TreeNode root, ArrayList<Integer> list) {
    if (root == null) return;
    inorder(root.left, list);
    list.add(root.val);
    inorder(root.right, list);
}
```

**解题思路2:** 和上面的题一样，可以使用指针记录上一个元素的地址，从而不使用数组即可。

```java
private int min = Integer.MAX_VALUE;
private TreeNode pre = null;
// 定义全局变量
public int getMinimumDifference(TreeNode root) {
    if (root == null) return 0;
    inorder(root);
    return min;
}
private void inorder(TreeNode root) {
    if (root == null) return;
    inorder(root.left);
    if (pre != null) {
        int diff = Math.abs(pre.val - root.val);
        min = Math.min(diff, min);
    }
    pre = root;
    inorder(root.right);
}
```

> 这里我们发现，二叉树的递归函数有一些是有返回值的，有一些又是没有返回值的。
>
> * 通常情况下，如果需要遍历所有结点，一般是没有返回值的，如这个题就需要遍历所有结点
> * 对于遍历部分结点就可以得到结果的题，大部分递归函数是有返回值的
> * 此外，如果要求返回满足条件的结点或者子树，那么也是需要有返回值的