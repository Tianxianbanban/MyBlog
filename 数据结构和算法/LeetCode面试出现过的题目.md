## LeetCode面试出现过的题目

#### [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

题面：给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那两个整数，并返回他们的数组下标。你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

+ 蛮力：时间复杂度O(n^2)。
+ 哈希表：哈希表保存元素和下标的映射，遍历一遍，只要target-nums[i]的结果存在于数组，就符合条件。两边哈希（先遍历依次保存映射在哈希表，之后再遍历将结果去哈希表中验证）或者一遍哈希（遍历一遍，每个元素直接得出对应结果就回头去哈希表中验证），时间复杂度O(n)，空间复杂度O(n)。



#### [面试题 08.06. 汉诺塔问题](https://leetcode-cn.com/problems/hanota-lcci/)

题面：在经典汉诺塔问题中，有 3 根柱子及 N 个不同大小的穿孔圆盘，盘子可以滑入任意一根柱子。一开始，所有盘子自上而下按升序依次套在第一根柱子上(即每一个盘子只能放在更大的盘子上面)。移动圆盘时受到以下限制:
(1) 每次只能移动一个盘子;
(2) 盘子只能从柱子顶端滑出移到下一根柱子;
(3) 盘子只能叠在比它大的盘子上。

请编写程序，用栈将所有盘子从第一根柱子移到最后一根柱子。

+ 递归：当只有一个盘子的时候，直接从A移动到C就可以了；只有两个盘子的时候，A中的上面小盘子移动到辅助B上，然后A中的大盘移动到C，B上的小盘再移动到C就完成了；而当盘子大于两个以后，可以以递归的思路，将这一叠盘子，分成两个部分，也就是最底下的大盘子和剩余n-1个盘子，接下来的处理方法和只有两个盘子时是一样的，将n-1个盘子移动到辅助B上，将最大的盘子移动到C上，再将n-1个盘子移动到C上面，其中n-1个盘子移动到C这个过程以A为辅助，整个过程可以递归完成。时间复杂度O(2^n - 1)，空间复杂度O(1)。



#### [103. 二叉树的锯齿形层次遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

题面：给定一个二叉树，返回其节点值的锯齿形层次遍历。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

+ ```java
  class Solution {
      public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
          List<List<Integer>> res = new LinkedList<>();
          if (root == null) {
              return res;
          }
          Queue<TreeNode> queue = new LinkedList<>();
          boolean leftToRight = true;
          int countNow = 1;
          int countNext = 0;
          queue.add(root);
          TreeNode cur = root;
          while (!queue.isEmpty()) {
              LinkedList<Integer> linkedList = new LinkedList<>();
              for (int i = 0; i < countNow; i++) {
                  cur = queue.poll();
                  if (leftToRight) {//从左到右
                      linkedList.add(cur.val);
                  } else {
                      linkedList.addFirst(cur.val);
                  }
  
                  if (cur.left != null) {
                      queue.add(cur.left);
                      countNext += 1;
                  }
                  if (cur.right != null) {
                      queue.add(cur.right);
                      countNext += 1;
                  }
  
              }
              res.add(linkedList);
              leftToRight = !leftToRight;
              countNow = countNext;
              countNext = 0;
          }
  
          return res;
      }
  }
  ```

  



#### [128. 最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

题面：给定一个未排序的整数数组，找出最长连续序列的长度。要求算法的时间复杂度为 *O(n)*。

+ 使用HashSet：时间复杂度要求O(n)，那么就不能通过排序的方式去找答案了，开辟一个辅助空间HashSet（HashSet可以存放不重复元素，不保证元素顺序），数组遍历一遍放入HashSet，之后每次遍历遇到nums[i]-1的元素不存在与HashSet当中说明遇到了一个子序列的开头元素，这个时候尝试从nums[i]递增并且验证后面的元素是否在HashSet当中，并且同时统计长度，最终求解。时间复杂度O(n)，空间复杂度O(n)。



#### [701. 二叉搜索树中的插入操作](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)

题面：给定二叉搜索树（BST）的根节点和要插入树中的值，将值插入二叉搜索树。 返回插入后二叉搜索树的根节点。 保证原始二叉搜索树中不存在新值。注意，可能存在多种有效的插入方式，只要树在插入后仍保持为二叉搜索树即可。 你可以返回任意有效的结果。

+ 递归：时间复杂度平均O(logN)，最坏O(N)，

  ```java
  class Solution {
      public TreeNode insertIntoBST(TreeNode root, int val) {
          if (root == null) {
              return new TreeNode(val);
          }
  
          if (val >= root.val) {
              //插入右子树
              root.right = insertIntoBST(root.right, val);
          } else {
              root.left =  insertIntoBST(root.left, val);
          }
  
          return root;
      }
  }
  ```

+ 迭代：



#### [887. 鸡蛋掉落](https://leetcode-cn.com/problems/super-egg-drop/)

题面：你将获得 K 个鸡蛋，并可以使用一栋从 1 到 N  共有 N 层楼的建筑。每个蛋的功能都是一样的，如果一个蛋碎了，你就不能再把它掉下去。你知道存在楼层 F ，满足 0 <= F <= N 任何从高于 F 的楼层落下的鸡蛋都会碎，从 F 楼层或比它低的楼层落下的鸡蛋都不会破。每次移动，你可以取一个鸡蛋（如果你有完整的鸡蛋）并把它从任一楼层 X 扔下（满足 1 <= X <= N）。你的目标是确切地知道 F 的值是多少。无论 F 的初始值如何，你确定 F 的值的最小移动次数是多少？

+ 

