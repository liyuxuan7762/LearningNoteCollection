### JZ7 重建二叉树

描述
给定节点数为 n 的二叉树的前序遍历和中序遍历结果，请重建出该二叉树并返回它的头结点。
例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建出如下图所示。
![img](https://uploadfiles.nowcoder.com/images/20210717/557336_1626504921458/776B0E5E0FAD11A6F15004B29DA5E628)

提示:
1.vin.length == pre.length
2.pre 和 vin 均无重复元素
3.vin出现的元素均出现在pre里

**解题思路1（使用递归方法）**：这个题就是在数据结构中根据前序遍历和中序遍历构造二叉树的经典题目。

* 如果先序遍历数组为空，或者为null，则直接返回null
* 如果先序遍历数组中只有一个元素，那么直接返回该元素构成的TreeNode即可。
* 如果先序遍历数组长度大于1，那么从先序遍历数组中获取第一个元素，这个元素为树的根。
* 然后已这个元素在中序遍历中的元素作为划分，把中序遍历数组分为两部分。
* 分别在这两部分上继续上述方法

![图片说明](https://uploadfiles.nowcoder.com/images/20220308/588579017_1646737384726/2939E21521C22C46A95A8B8DFA62CE0D)

```java
public TreeNode reConstructBinaryTree(int[] pre, int[] vin) {
    if (pre == null || pre.length == 0) return null;

    // 获取到前序遍历的第一个元素
    int root = pre[0];
    if (pre.length == 1) {
      return new TreeNode(root);
    }

    // 获取到root在中序遍历数组中的下标
    List<Integer> vinList = Arrays.stream(vin).boxed().collect(Collectors.toList());
    int i = vinList.indexOf(root);

    TreeNode treeRoot = new TreeNode(root);
    // 左
    TreeNode left = reConstructBinaryTree(Arrays.copyOfRange(pre, 1, i + 1), Arrays.copyOfRange(vin, 0, i));
    TreeNode right = reConstructBinaryTree(Arrays.copyOfRange(pre, i + 1, pre.length), Arrays.copyOfRange(vin, i + 1, vin.length));

    treeRoot.left = left;
    treeRoot.right = right;

    return treeRoot;
}
```

> 需要注意的是opyOfRange函数的两个下标是左闭右开的

### JZ26 树的子结构

描述

输入两棵二叉树A，B，判断B是不是A的子结构。（我们约定空树不是任意一个树的子结构）

假如给定A为{8,8,7,9,2,#,#,#,#,4,7}，B为{8,9,2}，2个树的结构如下，可以看出B是A的子结构

![img](https://uploadfiles.nowcoder.com/images/20211027/557336_1635320187489/B1C70B05B2BA3AAA854EE032F2A8D826)

**解题思路（利用递归）**：

* 首先判断当前结点是否和子结构的根结点相同。
* 如果相同，则继续判断左右子树是否也相同，如果相同则存在子结构。
* 如果不同，则继续以root1的左子树为一个新的树，判断这个树中是否包含子结构。

```java
public boolean HasSubtree(TreeNode root1, TreeNode root2) {
    if (root2 == null) return false;
    if (root1 == null && root2 != null) return false;
    // 判断当前传入的结点是否相同
    boolean flag = false;
    if (root1.val == root2.val) {
      // 如果当前结点相同，那么继续判断以当前结点的左右子树是否相同
      flag = isSubTree(root1, root2);
    }
    if (!flag) {
      flag = HasSubtree(root1.left, root2);
      // 在左子树上找不到子结构，那么去右子树上找子结构
      if (!flag) {
        flag = HasSubtree(root1.right, root2);
      }
    }
    return flag;
}

public boolean isSubTree(TreeNode root1, TreeNode root2) {
    if (root2 == null)
      return true;
    if (root1 == null && root2 != null)
      return false;
    if (root1.val == root2.val)
      return isSubTree(root1.left, root2.left) && isSubTree(root1.right, root2.right);
    else
      return false;
}
```

> HasSubtree是判断root1是否包含root2的子结构，如果不包含，则继续判断以root1的左右子树为根的树是否包含root2的子结构
>
> 其中isSubTree是判断以输入结点root1为根的树是否包含root2的子结构。

> 这个题一开始我认为采用先序遍历遍历两个树，然后判断root1的先序遍历是否包含root2的先序遍历序列即可。但是实际上不可行。考虑下面这种情况
>
> * root1 {1,2,3} root2 {2,3,#} root1先序遍历为123，root2先序遍历为23 序列是包含的，但是root2不是root1的子结构

> 这里学习了Java中如何判断一个字符串是另一个字符串的子串方法
>
> * 可以使用str.Contains(str1)
>
> 此外，对于ArrayList中的ContainsAll方法，判断的是list1中是否包含list2的全部元素，不是我们所说的那种子list。

### JZ33 二叉搜索树的后序遍历序列

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则返回 true ,否则返回 false 。假设输入的数组的任意两个数字都互不相同。

**解题思路1（利用递归）**:BST树后序遍历序列中最后的结点是根结点。

* 利用根结点，将序列分为左右子树。从头遍历序列，找到第一个不小于根结点的值，这个值左边就是左子树结点。从这个位置开始到序列长度-1为右子树元素。
* 在判断的时候，需要判断右子树这边元素值是否有小于根结点值的元素，如果有就不满足条件。否则就递归的去检查左右子树是否满足，直到剩下一个元素为止。

```java
public boolean VerifySequenceOfBST(int[] sequence) {
    if (sequence == null) return false;
    // 调用函数 判断输入序列是否满足BST
    return f(sequence, 0, sequence.length - 1);
}

public boolean f(int[] postorder, int i, int j) {
    // 当输入序列中是有一个元素的时候，那么满足条件
    if (i >= j) return true;
    // 从序列中获取到根结点元素
    int root = postorder[j];
    // 从头开始遍历整个序列，找到第一个不小于根结点的结点，这个结点就是以root为根的子树的右子树的第一个遍历的结点
    int p = i;
    while (postorder[p] < root) p++;
    // 此时rootIndex记录的就是以root为根的子树的右子树的第一个遍历的结点
    // 下面开始从右子树的第一个结点遍历序列，判断是否存在一个结点的值小于root的值，如果小于，则不满足BST
    for (int k = p; k < j; k++) {
      if (postorder[k] < root)
        return false;
    }
    // 递归的求左右子树是否满足条件
    return f(postorder, 0, p - 1) && f(postorder, p, j - 1);
}
```

> 总结，如何判断二叉树后序遍历的序列是否满足BST
>
> * 序列最后一个元素是根结点
> * 根据根结点，划分左右子树。
> * 判断右子树中是否有结点小于根结点，如果有就不满足
> * 否则递归求解左右子树是否满足BST

### JZ34 二叉树中和为某一值的路径(二)

描述

输入一颗二叉树的根节点root和一个整数expectNumber，找出二叉树中结点值的和为expectNumber的所有路径。

1.该题路径定义为从树的根结点开始往下一直到叶子结点所经过的结点

2.叶子节点是指没有子节点的节点

3.路径只能从父节点到子节点，不能从子节点到父节点

4.总节点数目为n

如二叉树root为{10,5,12,4,7},expectNumber为22

![img](https://uploadfiles.nowcoder.com/images/20210929/557336_1632915294911/0A4B8F161306A7054899D42C0C6937FD)

则合法路径有[[10,5,7],[10,12]]

**解题思路1（利用层序遍历）**：我们可以使用层序遍历。使用两个队列来记录，一个队列用来实现层序遍历，另一个队列用来记录经过的路径。当遍历到叶子结点的时候，根据其路径求和来判断结果是不是expectedNumber即可。

![WechatIMG165](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/WechatIMG165.jpeg)

```java
public ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int expectNumber) {
    ArrayList<ArrayList<Integer>> ret = new ArrayList<>();
    if (root == null) return ret;

    // 创建两个队列，一个队列用于中序遍历，一个用于记录路径
    Queue<ArrayList<Integer>> pathQ = new LinkedList<>();
    Queue<TreeNode> nodeQ = new LinkedList<>();
    // 根结点入队
    nodeQ.offer(root);
    pathQ.offer(new ArrayList<>(Arrays.asList(root.val)));

    // 开始遍历
    while (!nodeQ.isEmpty()) {
      // 出队
      ArrayList<Integer> currentPath = pathQ.poll();
      TreeNode node = nodeQ.poll();
      // 判断是否有左右子树
      if (node.left != null) {
        nodeQ.offer(node.left);
        ArrayList<Integer> left = new ArrayList<>(currentPath);
        left.add(node.left.val);
        pathQ.offer(left);
      }

      if (node.right != null) {
        nodeQ.offer(node.right);
        ArrayList<Integer> right = new ArrayList<>(currentPath);
        right.add(node.right.val);
        pathQ.offer(right);
      }

      if (node.left == null && node.right == null) {
        // 是叶子结点
        // 遍历经过的路径，求和看看是不是等于sum
        Integer total = 0;
        for (Integer integer : currentPath) {
          total += integer;
        }

        if (total == expectNumber)
          ret.add(currentPath);
      }
    }
    return ret;
}
```

> 这个方法还可以用来输出二叉树从根结点到叶子结点经过的路径。只需要修改当判断当前结点是叶子结点后，直接输出currentPath列表中的值即可。

**解题思路2（使用递归方法）**：利用DFS进行遍历。一直遍历到叶子结点，然后判断是否满足expectNumber。判断完当前结点后，要回退到上一个结点，然后继续遍历其他的结点。

![img](https://img2018.cnblogs.com/blog/1608161/201904/1608161-20190430151109127-936036131.png)

![img](https://img2018.cnblogs.com/blog/1608161/201904/1608161-20190430151117681-1738890995.png)

```java
public class Solution {
  
    private ArrayList<ArrayList<Integer>> ret = new ArrayList<>();
    private LinkedList<Integer> path = new LinkedList<>();

    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int expectNumber) {
        dfs(root, expectNumber);
        return ret;
    }

    private void dfs(TreeNode root, int expectNumber) {
        if (root == null) return;
        int num = expectNumber - root.val;
        path.add(root.val);
        // 判断当前结点是否是叶子
        if (root.left == null && root.right == null && num == 0) {
            ret.add(new ArrayList<>(path));
        }
        // 如果不是叶子结点，继续遍历左右子树
        dfs(root.left, num);
        dfs(root.right, num);
        // 当遍历结束后，那么当前结点就完全遍历完了，然后返回到上一个结点继续遍历
        path.removeLast();
    }
}
```

### Z84 二叉树中和为某一值的路径(三)

描述

给定一个二叉树root和一个整数值 sum ，求该树有多少路径的的节点值之和等于 sum 。

1.该题路径定义不需要从根节点开始，也不需要在叶子节点结束，但是一定是从父亲节点往下到孩子节点

2.总节点数目为n

3.保证最后返回的路径个数在整形范围内

假如二叉树root为{1,2,3,4,5,4,3,#,#,-1}，sum=6，那么总共如下所示，有3条路径符合要求

![img](https://uploadfiles.nowcoder.com/images/20211103/301499_1635923010369/C47185D4980F108BC73F790D8D2F6709)

**解题思路（利用递归方法）**：思路就是首先计算以当前结点为根结点的路径中满足条件的个树。然后求以当前结点的左右子树为根结点的满足条件的路径个数。

```java
private int count = 0;
public int FindPath(TreeNode root, int sum) {
    // write code here
    // 如果结点为空，则直接返回
    if (root == null)
      return count;

    // 计算以当前结点为根结点的满足sum条件的路径个树
    dfs(root, sum);

    // 查找以root左右子树为根的树是否有满足条件的路径
    FindPath(root.left, sum);
    // 这里是sum而不是sum-root.val是因为这是以root左右子树为根结点重新计算
    FindPath(root.right, sum);
    return count;
}

private void dfs(TreeNode root, int sum) {
    if (root == null)
      return;
    if (root.val == sum)
      count++;
    // 继续遍历左右子树
    dfs(root.left, sum - root.val);
    dfs(root.right, sum - root.val);
}
```

### JZ78 把二叉树打印成多行
描述

给定一个节点数为 n 二叉树，要求从上到下按层打印二叉树的 val 值，同一层结点从左至右输出，每一层输出一行，将输出的结果存放到一个二维数组中返回。

例如：
给定的二叉树是{1,2,3,#,#,4,5}
![img](https://uploadfiles.nowcoder.com/images/20210717/557336_1626492068888/41FDD435F0BA63A57E274747DE377E05)
该二叉树多行打印层序遍历的结果是

[

[1],

[2,3],

[4,5]

]

**解题思路1（层序遍历）**：这个题实际上考的就是二叉树的层序遍历，但是和层序遍历有一点不一样，就是需要按层输出。我们可以在出队之前获取到队的长度，然后使用循环同时将这一层的都出队，这样就可以了。

```java
public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
    ArrayList<ArrayList<Integer>> list = new ArrayList<>();
    if (pRoot == null)
      return list;
    // 创建对列
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(pRoot);

    while (!queue.isEmpty()) {
      int size = queue.size();
      ArrayList<Integer> levelList = new ArrayList<>();
      for (int i = 0; i < size; i++) {
        // 出队列
        TreeNode node = queue.poll();
        levelList.add(node.val);
        if (node.left !=null)
          queue.offer(node.left);
        if (node.right != null)
          queue.offer(node.right);
      }
      // for循环结束后，说明这一层就结束了
      list.add(levelList);
    }
    return list;
}
```

> 这个题的思路同样可以作用于非递归方法求解二叉树的层数
