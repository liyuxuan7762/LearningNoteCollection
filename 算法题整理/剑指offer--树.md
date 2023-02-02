### JZ55 二叉树的深度

描述

输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度，根节点的深度视为 1 。


进阶：空间复杂度 O(1)，时间复杂度 O(n)

假如输入的用例为{1,2,3,4,5,#,6,#,#,7}，那么如下图:

![img](https://uploadfiles.nowcoder.com/images/20211105/557336_1636101249194/DFDBE52B6C61F8021FC86EB0779848B1)

**解题思路1（递归方案）**

* 当传入的结点为空，则为空树，那么它的高度为0
* 当传入的结点左右结点都为空的时候，则为根结点，那么它的高度为1
* 当传入的结点只有左子树的时候，那么它的高度等于1+左子树的高度
* 同理，当传入的结点只有右子树的时候，那么它的高度等于1+右子树的高度
* 当传入结点既有左子树又有右子树的时候，那么它的高度等于1+左右子树高度中的最大值

```java
public int TreeDepth(TreeNode root) {
    // 如果这个树为空树
    if (root == null)
      return 0;
    // 如果当前结点没有左右子树
    if (root.left == null && root.right == null)
      return 1;
    // 如果只有左子树
    if (root.left != null && root.right == null)
      return 1 + TreeDepth(root.left);
    // 如果只有右子树
    if (root.left == null && root.right != null)
      return 1 + TreeDepth(root.right);
    return 1 + Math.max(TreeDepth(root.left), TreeDepth(root.right));
}
```

或者下面的方式更加简洁：

```java
public int TreeDepth(TreeNode root) {
    if (root == null)
      return 0;
    return 1 + Math.max(TreeDepth(root.left), TreeDepth(root.right));
}
```

**解题思路2（利用树的层序遍历）**：使用对列对树进行层序遍历，每遍历一层，那么深度就加1.

```java
public int TreeDepth(TreeNode root) {
    // 如果是空树
    if (root == null)
      return 0;
    // 创建结点，将根结点入队
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    int res = 0;
    // 开始循环
    while (!queue.isEmpty()) {
      // 获取当前队列长度
      int size = queue.size();
      // 出队，并将出队结点的左右子结点入队
      for (int i = 0; i < size; i++) {
        TreeNode poll = queue.poll();
        if (poll.left != null)
          queue.offer(poll.left);
        if (poll.right != null)
          queue.offer(poll.right);
      }
      res++;
    }

    return res;
}
```

> 这里需要注意的是在Java中的Queue的相关API函数。
>
> * 向队尾插入元素使用**offer()**
> * 出队使用的是**poll()**，出队的同时会将这个元素从队列中移除
> * 而**poll()**是返回队头的元素，但是不会将元素从队头删除。

### JZ27 二叉树的镜像

描述

操作给定的二叉树，将其变换为源二叉树的镜像。

要求： 空间复杂度 O(n) 。本题也有原地操作，即空间复杂度 O(1)的解法，时间复杂度 O(n)

比如：

源二叉树

![img](https://uploadfiles.nowcoder.com/images/20210922/382300087_1632302001586/420B82546CFC9760B45DD65BA9244888)

镜像二叉树

![img](https://uploadfiles.nowcoder.com/images/20210922/382300087_1632302036250/AD8C4CC119B15070FA1DBAA1EBE8FC2A)

**解题思路1（递归方案）**:

* 如果当前传入的结点为空，那么直接返回空
* 如果当前传入的结点没有左右子树，那么返回它本身
* 如果当前传入的结点只有左子树，那么就应该递归调用，求左子树的镜像。求完之后，需要将传入结点的左子树赋值给右子树，并将左子树赋值为空
* 如果当前传入的结点只有右子树，那么应该递归调用，求右子树的镜像。求完之后，需要将传入结点的右子树赋值给左子树，并将右子树赋值为空。
* 如果当前传入的结点既有左子树又有右子树，那么应该递归调用，分别求左右子树的镜像。然后利用一个中间变量，交换传入结点的左右子树。

```java
public TreeNode Mirror (TreeNode pRoot) {
    // write code here
    // 如果是空树
    if(pRoot == null)
      return null;
    // 如果传入结点没有左右子树
    if (pRoot.left == null && pRoot.right == null) 
      return pRoot;
    // 如果只有左子树
    if(pRoot.left != null && pRoot.right == null) {
      Mirror(pRoot.left);
      pRoot.right = pRoot.left;
      pRoot.left = null;
      return pRoot;
    }
    // 只有右子树
    if(pRoot.left == null && pRoot.right != null) {
      Mirror(pRoot.right);
      pRoot.left = pRoot.right;
      pRoot.right = null;
      return pRoot;
    }
    // 如果既有左子树 又有右子树
    Mirror(pRoot.left);
    Mirror(pRoot.right);
    TreeNode temp = pRoot.left;
    pRoot.left = pRoot.right;
    pRoot.right = temp;

    return pRoot;
}
```

当然，可以采用更加简洁的递归方式：

```java
public TreeNode Mirror (TreeNode pRoot) {
    // 如果是空树
    if(pRoot == null)
      return null;

    Mirror(pRoot.left);
    Mirror(pRoot.right);
    TreeNode temp = pRoot.left;
    pRoot.left = pRoot.right;
    pRoot.right = temp;

    return pRoot;
}
```

**解题思路2（遍历树中每一个结点，然后分别交换）**：利用栈对树进行深度优先遍历。

* 首先根结点入栈
* 然后开启循环，只要栈不为空，那么就从栈中取出一个元素，然后将这个元素的左右子树入栈。
* 然后交换出栈元素的左右子树。

> 实际上，栈中的元素就是等待交换左右子树的结点。而出栈后，就是已经交换完子树以后的结点了。

```java
public TreeNode Mirror(TreeNode pRoot) {
    // write code here
    if (pRoot == null)
      return null;
    Stack<TreeNode> stack = new Stack<>();
    stack.push(pRoot);

    while (!stack.isEmpty()) {
      TreeNode node = stack.pop();
      // 左右子树入栈
      if (node.right != null)
        stack.push(node.right);
      if (node.left != null)
        stack.push(node.left);
      // 交换出栈元素的左右子树
      TreeNode temp = node.left;
      node.left = node.right;
      node.right = temp;
    }

    return pRoot;
}
```

### JZ79 判断是不是平衡二叉树

描述

输入一棵节点数为 n 二叉树，判断该二叉树是否是平衡二叉树。

在这里，我们只需要考虑其平衡性，不需要考虑其是不是排序二叉树

**平衡二叉树**（Balanced Binary Tree），具有以下性质：它是一棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。

样例解释：

![img](https://uploadfiles.nowcoder.com/images/20210918/382300087_1631935149594/D55A07912354B3AB7E9F2F5EA27CB7D6)

样例二叉树如图，为一颗平衡二叉树

注：我们约定空树是平衡二叉树。

要求：空间复杂度O(1)，时间复杂度 O(n)

**输入描述：**输入一棵二叉树的根节点

**返回值描述：**输出一个布尔类型的值

**解题思路1（遍历方法）**：循环遍历树中的每一个结点，去调用判断树的深度的方法，判断每一个结点是否满足平衡二叉树的条件。如果每一个结点都满足，那么这个树就是平衡二叉树。否则就不是平衡二叉树。

```java
public boolean IsBalanced_Solution(TreeNode root) {
    // 如果是一个空树或者传入结点左右子树为空 那么是AVL树
    if (root == null)
      return true;
    // 创建一个栈 进行深度遍历
    Stack<TreeNode> stack = new Stack<>();
    stack.push(root);

    while (!stack.isEmpty()) {
      TreeNode node = stack.pop();
      if (node.right != null)
        stack.push(node.right);
      if (node.left != null)
        stack.push(node.left);
      if (Math.abs(TreeDepth(node.left) - TreeDepth(node.right)) > 1) {
        return false;
      }
    }

    return true;
}

public int TreeDepth(TreeNode root) {
    if (root == null)
      return 0;
    return 1 + Math.max(TreeDepth(root.left), TreeDepth(root.right));
}
```

**解题思路2（递归方法）**

* 判断当前结点是否满足平衡二叉树的条件，如果不满足，则直接返回false
* 如果满足，则去判断左右子树是否满足；如果都满足，则返回true，否则返回false

```java
public boolean IsBalanced_Solution(TreeNode root) {
    // 如果是一个空树 则返回true
    if (root == null)
      return true;
    // 计算左右结点的深度来判断当前结点是否满足平衡二叉树的条件
    int left = TreeDepth(root.left);
    int right = TreeDepth(root.right);
    if (Math.abs(left - right) > 1)
      return false;

    return IsBalanced_Solution(root.left) && IsBalanced_Solution(root.right);

}

public int TreeDepth(TreeNode root) {
    if (root == null)
      return 0;
    return 1 + Math.max(TreeDepth(root.left), TreeDepth(root.right));
}
```

### JZ28 对称的二叉树

描述

给定一棵二叉树，判断其是否是自身的镜像（即：是否对称）
例如：下面这棵二叉树是对称的
![img](https://uploadfiles.nowcoder.com/images/20210926/382300087_1632642756706/A22A794C036C06431E632F9D5E2E298F)
下面这棵二叉树不对称。
![img](https://uploadfiles.nowcoder.com/images/20210926/382300087_1632642770481/3304ABDD147D8E140B2CEF3201BD8372)

**解题思路1（递归）**：通过观察可以发现，对称二叉树的两种遍历方式得到结果是一样的“根左右”,"根右左"。

* 使用两个指针，同时遍历左右子树。遍历左子树的采用根左右方式。遍历右子树的采用根右左。
* 每遍历一个结点，就对比一下这两个结点是否相同；如果不同，则不满足条件。

具体思路如下：

* 如果两个指针为空，那么说明以传入结点为根的树为空树，那么满足对称条件
* 如果两个指针只有一个不为空，或者两个指针都不为空，但是值不相同，那么不满足对称条件
* 否则，那么说明遍历到当前结点为止，这棵树满足对称条件，那么继续去判断其左右子树树否满足条件。

```java
public boolean isSymmetrical(TreeNode pRoot) {
    return recursion(pRoot, pRoot);
}

public boolean recursion(TreeNode root1, TreeNode root2) {
    if (root1 == null && root2 == null)
      return true;
    if (root1 == null || root2 == null || root1.val != root2.val)
      return false;
    // 遍历子结点
    return recursion(root1.left, root1.right) && recursion(root2.right, root2.left);
}
```

**解题思路2（层序遍历）**

* 如果传入结点为空，那么为空树，则返回true
* 如果传入结点不为空，那么我们应该使用两个指针，分别层序遍历传入结点的左右子树，并且一个按照从左到右遍历，一个按照从右到左遍历。
* 每遍历一个元素就进行比较，判断是否满足条件。如果不满足则返回false
* 待完全遍历后，返回true

```java
public boolean isSymmetrical(TreeNode pRoot) {
    // 如果是空树，则满足对称条件
    if (pRoot == null)
      return true;

    // 创建两个队列
    Queue<TreeNode> q1 = new LinkedList<>();
    Queue<TreeNode> q2 = new LinkedList<>();

    // 根结点入栈
    q1.offer(pRoot);
    q2.offer(pRoot);

    // 遍历
    while (!q1.isEmpty() && q2.isEmpty()) {
      // 出队
      TreeNode left = q1.poll();
      TreeNode right = q2.poll();
      // 判断两个结点是否对称
      if (left == null && right == null)
        continue;
      if (left == null || right == null || left.val != right.val)
        return false;

      // 子结点入队
      // 左子树遍历从左到右
      q1.offer(left.left);
      q1.offer(left.right);
      // 右子树遍历从右到左
      q2.offer(right.right);
      q2.offer(right.left);

    }
    return true;

}
```

> 这个需要注意的是判断两个结点是否满足对称条件，在代码中采用的是：
>
> ```java
> if (left == null && right == null)
>   		continue;
> if (left == null || right == null || left.val != right.val)
>   		return false;
> ```
>
> 这个判断条件。这里可能会有疑问，为什么不使用left.val == right.val来判断呢？
>
> 这个是因为在循环中，我们实际上只需要判断出不满足对称的条件即可。因为如果满足条件，那我们会继续进行循环遍历。

### JZ32从上往下打印二叉树

描述

不分行从上往下打印出二叉树的每个节点，同层节点从左至右打印。例如输入{8,6,10,#,#,2,1}，如以下图中的示例二叉树，则依次打印8,6,10,2,1(空节点不打印，跳过)，请你将打印的结果存放到一个数组里面，返回。

![img](https://uploadfiles.nowcoder.com/images/20211029/557336_1635477973725/6C502E0240CAC668843969AFF396B5E4)

**解题思路1（队列层序遍历）**:利用层序遍历即可。

```java
public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
    ArrayList<Integer> list = new ArrayList<>();
    // 如果为空树，则直接返回
    if (root == null) return list;

    // 层序遍历
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
      TreeNode node = queue.poll();
      if (node != null) {
        list.add(node.val);
      }
      if (node.left != null)
        queue.offer(node.left);
      if (node.right != null)
        queue.offer(node.right);
    }
    return list;
}
```

### JZ82 二叉树中和为某一值的路径(一)

给定一个二叉树root和一个值 sum ，判断是否有从根节点到叶子节点的节点值之和等于 sum 的路径。

1.该题路径定义为从树的根结点开始往下一直到叶子结点所经过的结点

2.叶子节点是指没有子节点的节点

3.路径只能从父节点到子节点，不能从子节点到父节点

4.总节点数目为n

例如：
给出如下的二叉树，sum=22
![img](https://uploadfiles.nowcoder.com/images/20200807/999991351_1596786493913_8BFB3E9513755565DC67D86744BB6159)
返回true，因为存在一条路径25→4→11→2的节点值之和为

**解题思路1（深度优先遍历）**:使用栈实现的深度优先遍历方法。具体思路：

* 创建两个栈，第一个栈是用来实现深度优先遍历的，另一个栈记录到某个节点所进过的val之和。
* 在进行遍历的时候，从栈中取出一个元素，先判断是否是叶子结点，然后判断sum是否等于另一个栈中记录的val之和。如果满足，则返回true
* 如果循环结束还没有返回，那么返回false

![alt](https://uploadfiles.nowcoder.com/images/20211204/397721558_1638591916283/D33422A0A2A416179193EAF5EBD60154)

```java
public boolean hasPathSum(TreeNode root, int sum) {
    if (root == null)
      return false;

    // 创建两个栈 一个是深度优先遍历的栈 另一个栈记录从根到对应结点走过的val之和
    Stack<TreeNode> stack1 = new Stack<>();
    Stack<Integer> stack2 = new Stack<>();

    // 根结点入栈
    stack1.push(root);
    stack2.push(root.val);

    // 循环遍历
    while (!stack1.isEmpty()) {
      // 出栈
      TreeNode node = stack1.pop();
      Integer val = stack2.pop();

      // 判断出栈元素是否是叶子结点，且满足条件
      if (node.left == null && node.right == null && val + node.val == sum) {
        return true;
      }

      if (node.left != null) {
        stack1.push(node.left);
        stack2.push(node.left.val);
      }
      if (node.right != null) {
        stack1.push(node.right);
        stack2.push(node.right.val);
      }

    }

    return false;
}
```

**解题思路2（递归方法）**

- step 1：每次检查遍历到的节点是否为空节点，空节点就没有路径。
- step 2：再检查遍历到是否为叶子节点，且当前sum值等于节点值，说明可以刚好找到。
- step 3：检查左右子节点是否可以有完成路径的，如果任意一条路径可以都返回true，因此这里选用两个子节点递归的或。

```java
public boolean hasPathSum(TreeNode root, int sum) {
    // write code here
    if (root == null)
      return false;
    // 如果是叶子结点
    if (root.left == null && root.right == null && sum - root.val == 0)
      return true;

    // 如果当前结点不是空或者不是叶子结点
    return hasPathSum(root.left, sum - root.val) || hasPathSum(root.right, sum - root.val);
}
```

### JZ77 按之字形顺序打印二叉树

描述

给定一个二叉树，返回该二叉树的之字形层序遍历，（第一层从左向右，下一层从右向左，一直这样交替）

要求：空间复杂度：O(n)，时间复杂度：O(n)

例如：
给定的二叉树是{1,2,3,#,#,4,5}
![img](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/41FDD435F0BA63A57E274747DE377E05.png)
该二叉树之字形层序遍历的结果是

[

[1],

[3,2],

[4,5]

]

**解题思路1（层序遍历）**：使用层序遍历，遍历出来的值存储list中，然后在根据是否需要reverse进行逆序。

```java
public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
    ArrayList<ArrayList<Integer>> list = new ArrayList<>();
    // 如果为空树，则直接返回空
    if (pRoot == null) return list;

    // 根结点入队
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(pRoot);

    // 开始循环
    boolean direction = true; // true代表从左向右 false代表从右向左
    while (!queue.isEmpty()) {
      // 出队
      int size = queue.size();
      ArrayList<Integer> levelList = new ArrayList<>(size);
      for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        levelList.add(node.val);
        if (node.left != null)
          queue.offer(node.left);
        if (node.right != null)
          queue.offer(node.right);
      }

      if (!direction) {
        Collections.reverse(levelList);

      }
      direction = !direction;

      list.add(levelList);
    }
    return list;
}
```

### JZ54 二叉搜索树的第k个节点

描述

给定一棵结点数为n 二叉搜索树，请找出其中的第 k 小的TreeNode结点值。

1.返回第k小的节点值即可

2.不能查找的情况，如二叉树为空，则返回-1，或者k大于n等等，也返回-1

3.保证n个节点的值不一样

如输入{5,3,7,2,4,6,8},3时，二叉树{5,3,7,2,4,6,8}如下图所示：

![img](https://uploadfiles.nowcoder.com/images/20211117/392807_1637120852509/F732B49BA33ECC72FF97FF7BDE2ACF69)

该二叉树所有节点按结点值升序排列后可得[2,3,4,5,6,7,8]，所以第3个结点的结点值为4，故返回对应结点值为4的结点即可。

**思路**：这个题实际上考察的是二叉树的遍历方式。根据二叉排序树的性质可以知道，二叉排序树的中序遍历是有序的。因此我们可以进行中序遍历来找出这个树中第k小的结点

**解题思路1（利用非递归方式）**

```java
public int KthNode(TreeNode proot, int k) {
    if (proot == null)
      return -1;

    // 进行中序遍历
    Stack<TreeNode> stack = new Stack<>();
    int count = 0;
    while (proot != null || !stack.isEmpty()) {
      while (proot != null) {
        stack.push(proot);
        proot = proot.left;
      }
      TreeNode node = stack.pop();
      proot = proot.right;
      count++;
      if (count == k)
        return node.val;
    }
    return -1;
}
```

> 对于二叉树的前中后层四种遍历方式的递归和非递归算法需要总结和复习，会在明天的笔记中进行整理。
