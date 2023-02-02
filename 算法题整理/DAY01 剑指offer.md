### JZ6 从尾到头打印链表

问题描述

输入一个链表的头节点，按链表从尾到头的顺序返回每个节点的值（用数组返回）。

如输入{1,2,3}的链表如下图:

![img](https://uploadfiles.nowcoder.com/images/20210717/557336_1626506480516/103D87B58E565E87DEFA9DD0B822C55F)

返回一个数组为[3,2,1]；0 <= 链表长度 <= 10000

**解题思路1（使用栈）**：这个题因为是倒叙输出，可以考虑利用栈的特性来解决。我们遍历整个链表，然后将元素存储在栈中，最后在依次出栈即可。

```java
/**
*    public class ListNode {
*        int val;
*        ListNode next = null;
*
*        ListNode(int val) {
*            this.val = val;
*        }
*    }
*
*/
import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ListNode p = new ListNode(0);
        p.next = listNode;
        Stack<Integer> stack = new Stack<>();
        while(p.next != null) {
            p = p.next;
            stack.push(p.val);
        }
        ArrayList<Integer> res = new ArrayList<>();
        while(!stack.empty()) {
            Integer pop = stack.pop();
            System.out.println(pop);
            res.add(pop);
        }
        return res;
    }
}
```

> 这里需要注意的知识点是如何遍历链表。我们可以给链表创建一个头结点，这个头结点中不存储数据，让这个头结点指向链表中第一个元素结点。这样在遍历的时候只需要根据p.next != null这个条件就可以判断链表是否遍历完全。

**解题思路2（递归）**：使用递归和使用栈原理是一样的，使用递归实现的方法也可以使用栈实现。递归的三个条件如下：

* 结束条件：递归进入链表尾，最后一个结点为空的时候
* 返回值：返回子问题的全部输出
* 本级任务：递归到下一级子问题，等下一级的子问题输出后，将本级结点的值添加在数组末尾。

```java
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        reverseLinkList(list, listNode);
        return list;
    }
    public void reverseLinkList(ArrayList<Integer> list, ListNode head) {
        if(head != null) {
            reverseLinkList(list, head.next);
            list.add(head.val);
        }
    }
}
```

### JZ24 反转链表

问题描述

给定一个单链表的头结点pHead(该头节点是有值的，比如在下图，它的val是1)，长度为n，反转该链表后，返回新链表的表头。

数据范围： 0<=n<=1000

要求：空间复杂度 O(1) ，时间复杂度 O(n)

如当输入链表{1,2,3}时，

经反转后，原链表变为{3,2,1}，所以对应的输出为{3,2,1}。

以上转换过程如下图所示：

![img](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/4A47A0DB6E60853DEDFCFDF08A5CA249.png)

**解题思路1（使用栈）:**和上一道题一样，只不过这一次栈里面存储的不再是结点的值，而是结点对象。

```java
import java.util.Stack;
public class Solution {
    public ListNode ReverseList(ListNode head) {
        if (head == null) {
            return null;
        }
        ListNode p = new ListNode(0);
        p.next = head;
        Stack<ListNode> stack = new Stack<>();

        while (p.next != null) {
            p = p.next;
            stack.push(p);
        }
        
        ListNode reverseHead = new ListNode(0);
        p = reverseHead;
        while (!stack.empty()) {
            ListNode node = stack.pop();
            System.out.println(node.val);
            reverseHead.next = node;
            reverseHead = reverseHead.next;
        }
        reverseHead.next = null; // 这一句话需要特别注意！！！
        return p.next;
    }
}
```

> 需要指出的是，遍历完整个栈以后，一定要加上reverseHead.next = null这一句代码，否则就会出现环。因为最后一个元素是1，1的下一个结点指向了2，而经过反转后，2的下一个元素又是1，所以会产生环导致系统报错。

### JZ25合并两个排序的链表

描述

输入两个递增的链表，单个链表的长度为n，合并这两个链表并使新链表中的节点仍然是递增排序的。

数据范围： 0<=n<=1000，-1000 \le 节点值 -1000<=n<=1000
要求：空间复杂度 O(1)*O*(1)，时间复杂度 O(n)*O*(*n*)

如输入{1,3,5},{2,4,6}时，合并后的链表为{1,2,3,4,5,6}，所以对应的输出为{1,2,3,4,5,6}，转换过程如下图所示：

![img](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/09DD8C2662B96CE14928333F055C5580.png)

```java
public class Solution {
    public ListNode Merge(ListNode list1, ListNode list2) {
        if (list1 == null) {
            return list2;
        }
        if (list2 == null) {
            return list1;
        }

        ListNode L1 = list1;
        ListNode L2 = list2;
        ListNode head = new ListNode(0);
        ListNode p = head;

        while (L1 != null && L2 != null) {
            if (L1. val < L2.val) {
                p.next = L1;
                p = p.next;
                L1 = L1.next;
            } else {
                p.next = L2;
                p = p.next;
                L2 = L2.next;
            }
        }

        if (L1 != null) {
            p.next = L1;
        }

        if (L2 != null) {
            p.next = L2;
        }

        return head.next;
    }
}
```

> 这个题目需要注意的是
>
> * 如果两个链表有一个为空的情况
> * 如果while循环结束后，还有链表没有遍历完的情况

### JZ52两个链表的第一个公共结点

描述

输入两个无环的单向链表，找出它们的第一个公共结点，如果没有公共节点则返回空。（注意因为传入数据是链表，所以错误测试数据的提示是用其他方式显示的，保证传入数据是正确的）

数据范围： n≤1000*n*≤1000
要求：空间复杂度O(1)，时间复杂度 O(n)

例如，输入{1,2,3},{4,5},{6,7}时，两个无环的单向链表的结构如下图所示：

![img](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/394BB7AFD5CEA3DC64D610F62E6647A6.png)

可以看到它们的第一个公共结点的结点值为6，所以返回结点值为6的结点。

输入描述：

输入分为是3段，第一段是第一个链表的非公共部分，第二段是第二个链表的非公共部分，第三段是第一个链表和第二个链表的公共部分。 后台会将这3个参数组装为两个链表，并将这两个链表对应的头节点传入到函数FindFirstCommonNode里面，用户得到的输入只有pHead1和pHead2。

返回值描述：

返回传入的pHead1和pHead2的第一个公共结点，后台会打印以该节点为头节点的链表。

**解题思路1（使用HashSet）**：这种方法利用了HastSet的唯一性。首先遍历一个链表，将里面所有元素保存到HashSet中。然后遍历第二个链表，每一个元素来判断HashSet中是否存在，如果存在，则直接返回，否则返回空。

```java
public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
  if (pHead1 == null || pHead2 == null) {
    return null;
  }
  HashSet set = new HashSet<>();
  // 遍历其中一个链表
  while (pHead1 != null) {
    set.add(pHead1);
    pHead1 = pHead1.next;
  }

  while (pHead2 != null) {
    if (set.contains(pHead2)) {
      return pHead2;
    }
    pHead2 = pHead2.next;
  }
  return null;
}
```

### JZ22链表中倒数最后k个结点

描述

输入一个长度为 n 的链表，设链表中的元素的值为 ai ，返回该链表中倒数第k个节点。

如果该链表长度小于k，请返回一个长度为 0 的链表。

要求：空间复杂度 O(n)，时间复杂度 O(n)

进阶：空间复杂度 O(1)，时间复杂度 O(n)

例如输入{1,2,3,4,5},2时，对应的链表结构如下图所示：

![img](https://uploadfiles.nowcoder.com/images/20211105/423483716_1636084313645/5407F55227804F31F5C5D73558596F2C)

其中蓝色部分为该链表的最后2个结点，所以返回倒数第2个结点（也即结点值为4的结点）即可，系统会打印后面所有的节点来比较。

**解题思路1（使用快慢指针）**：第一个指针先移动k步，然后第二个指针再从头开始，这个时候这两个指针同时移动，当第一个指针到链表的末尾的时候，返回第二个指针即可

![img](https://uploadfiles.nowcoder.com/images/20210623/889362376_1624436560373/B27699765EEF6CA0E75AF0D1A4B9BCAC)

```java
public ListNode FindKthToTail(ListNode pHead, int k) {
  // write code here
  ListNode fast = pHead;
  ListNode slow = pHead;

  for (int i = 0; i < k; i++) {
    // 需要注意的是 如果链表长度小于k的情况
    // 通过下面的if判断判断了链表为空的情况和链表长度小于k的情况
    if (fast == null)
      return null;
    fast = fast.next;
  }
  while (fast != null) {
    slow = slow.next;
    fast = fast.next;
  }
  return slow;
}
```

> 这里需要注意的是链表长度小于k的情况

**解题思路2（使用栈）**：将链表中所有元素保存到栈中，然后再依次出栈，当出栈到第k个元素的时候返回。

```java
public ListNode FindKthToTail(ListNode pHead, int k) {
  // write code here
  Stack<ListNode> stack = new Stack<>();
  while (pHead != null) {
    stack.push(pHead);
    pHead = pHead.next;
  }
  // 判断链表是否为空，如果链表为空，那么栈为空
  // 判断链表长度是否小于k
  if (stack.isEmpty() || stack.size() < k)
    return null;
  int count = 0;
  ListNode pop = null;
  while (count != k) {
    pop = stack.pop();
    count++;
  }
  return pop;
}
```

> 需要注意的点：
>
> * 需要判断链表是否为空，可以通过栈是否为空判断。当链表为空的时候，栈也为空
> * 需要判断链表长度是否小于k，如果小于k则直接返回空。这里通过栈的大小来判断。

### JZ18删除链表的节点

给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。返回删除后的链表的头节点。

1.此题对比原题有改动

2.题目保证链表中节点的值互不相同

3.该题只会输出返回的链表和结果做对比，所以若使用 C 或 C++ 语言，你不需要 free 或 delete 被删除的节点

数据范围:

0<=链表节点值<=10000

0<=链表长度<=10000

**解题思路**：直接遍历链表，同时判断是否有满足条件的元素，如果有直接删除即可。

```java
public ListNode deleteNode (ListNode head, int val) {
  // write code here
  if (head == null)
    return null;
  ListNode h = new ListNode(0);
  h.next = head;
  ListNode p1 = h;
  ListNode p2 = h.next;

  while (p2 != null) {
    if (val == p2.val) {
      // 删除p结点
      p1.next = p2.next;
      return h.next;
    }
    p1 = p1.next;
    p2 = p2.next;
  }
  return h.next;
}
```

> 这里需要学习一下链表中如何删除元素。

### JZ76删除链表中重复的结点

描述

在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表 1->2->3->3->4->4->5 处理后为 1->2->5

进阶：空间复杂度 O(n) ，时间复杂度 O(n) 

例如输入{1,2,3,3,4,4,5}时，对应的输出为{1,2,5}，对应的输入输出链表如下图所示：

![img](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/5B9CC4C8B8AE60071D9441AB64E66772.png)

**解题思路1:直接删除**：设置一个指针依次比较链表中相邻的两个元素的值是否相等。如果相等，则开启一个新的循环

- step 1：给链表前加上表头，方便可能的话删除第一个节点。

```java
ListNode res = new ListNode(0);//在链表前加一个表头
res.next = pHead;
```

- step 2：遍历链表，每次比较相邻两个节点，如果遇到了两个相邻节点相同，则新开内循环将这一段所有的相同都遍历过去。
- step 3：在step 2中这一连串相同的节点前的节点直接连上后续第一个不相同值的节点。
- step 4：返回时去掉添加的表头。

```java
public ListNode deleteDuplication(ListNode pHead) {
    ListNode head = new ListNode(0);
    head.next = pHead;
    ListNode cur = head;

    while (cur.next != null && cur.next.next != null) {
        if (cur.next.val == cur.next.next.val) {
            int temp = cur.next.val;
            while (cur.next != null && cur.next.val == temp) {
                cur.next = cur.next.next;
            }
        } else {
            cur = cur.next;
        }
    }
    return head.next;
}
```

**解题思路2（哈希Map）**：遍历整个链表，利用HashMap记录每一个元素出现的频率。再次遍历整个链表，每遍历元素，就到HashMap中获取这个元素出现的次数，如果超过1，则直接删除。

```java
public ListNode deleteDuplication(ListNode pHead) {
  if (pHead == null)
    return null;

  // 1. 遍历链表，获取每个元素出现的频率
  ListNode head = new ListNode(0);
  head.next = pHead;
  ListNode cur = head;
  HashMap<Integer, Integer> map = new HashMap<>();

  while (cur.next != null) {
    int val = cur.next.val;
    if (map.containsKey(val)) {
      map.put(val, map.get(val) + 1);
    } else {
      map.put(val, 1);
    }
    cur = cur.next;
  }

  // 2. 重新遍历整个链表，根据Map中元素出现的频率删除元素
  cur = head;
  while (cur.next != null) {
    // 出现的次数
    Integer count = map.get(cur.next.val);
    if (count > 1) {
      // 删除
      cur.next = cur.next.next;
    } else {
      cur = cur.next;
    }
  }

  return head.next;
}
```

### JZ23链表中环的入口结点

描述

给一个长度为n链表，若其中包含环，请找出该链表的环的入口结点，否则，返回null。


要求：空间复杂度 O(1)，时间复杂度 O(n)

例如，输入{1,2},{3,4,5}时，对应的环形链表如下图所示：

![img](https://uploadfiles.nowcoder.com/images/20211025/423483716_1635154005498/DA92C945EF643F1143567935F20D6B46)

可以看到环的入口结点的结点值为3，所以返回结点值为3的结点。

**解题思路1（使用HashSet）**：遍历整个链表，每遍历一个元素就将该元素保存到HashSet中，并判断HashSet中是否已经包含该元素。如果已经包含，那么这个结点就是环的入口结点。

```java
public ListNode EntryNodeOfLoop(ListNode pHead) {
  // 循环遍历链表，将元素依次保存到HashSet中
  // 每遍历一个元素就判断这个元素是否在HashSet中已经存在
  // 如果存在，则直接返回该元素结点
  if (pHead == null)
    return null;
  HashSet<ListNode> set = new HashSet<>();
  while (pHead != null) {
    if (set.contains(pHead)) {
      return pHead;
    }
    set.add(pHead);
    pHead = pHead.next;
  }

  return null;
}
```

**解题思路2（使用快慢指针）**

![image-20230108212443675](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230108212443675.png)

![alt](https://uploadfiles.nowcoder.com/images/20220330/397721558_1648627978736/C56F671FC2966A5E586BD6E56E19D3E3)

![image-20230108213304007](https://raw.githubusercontent.com/liyuxuan7762/MyImageOSS/master/md_images/image-20230108213304007.png)

具体步骤如下：

* 通过定义slow和fast指针，slow每走一步，fast走两步
* 判断是否存在环，如果存在环在fast和slow会相遇，继续进行下一步；否则如果不存在环，则直接返回null
* 相遇后，快指针重新指向链表中第一个元素，然后快慢指针以相同的速度前进，相遇的点就是环入口

```java
public ListNode EntryNodeOfLoop(ListNode pHead) {
  if (pHead == null)
    return null;

  // 定义快慢指针
  ListNode fast = pHead;
  ListNode slow = pHead;

  while (fast != null && fast.next != null) {
    fast = fast.next.next;
    slow = slow.next;
    if (fast == slow) break; // 相遇后跳出循环
  }

  // 注意需要添加判断条件 判断是否存在环
  if (fast == null || fast.next == null) return null;

  fast = pHead;
  while (fast != slow) {
    fast = fast.next;
    slow = slow.next;
  }
  return fast;
}
```

这道题我们可以总结出如何判断一个链表中是否有环

* 设置快慢指针，一旦快慢指针相遇，那么说明有环。

```java
public boolean hasCycle(ListNode head) {
  if (head == null)
    return false;

  // 定义快慢指针
  ListNode fast = head;
  ListNode slow = head;

  while (fast != null && fast.next != null) {
    fast = fast.next.next;
    slow = slow.next;
    if (fast == slow) break; // 相遇后跳出循环
  }

  // 注意需要添加判断条件 判断是否存在环
  if (fast == null || fast.next == null) {
    return false;
  } else {
    return true;
  }
}
```

