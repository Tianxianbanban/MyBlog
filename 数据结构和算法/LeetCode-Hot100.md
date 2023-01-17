## LeetCode-Hot100



#### [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

题面：给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

+ 暴力
+ 哈希表：一遍哈希，优化下两遍哈希



#### [2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

题面：给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

+ 数学方法：注意进位。时间复杂度O(n)；

  ```java
  class Solution {
      public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
          ListNode cur = new ListNode(0);
          ListNode result = cur;
          ListNode p = l1;
          ListNode q = l2;
          int carry = 0;
          while(p != null || q != null) {
              int pNum = (p != null) ? p.val : 0;
              int qNum = (q != null) ? q.val : 0;
              int num = pNum + qNum + carry;
              carry = num/10;
              cur.next = new ListNode(num % 10);
              cur = cur.next;
              if (p != null) p = p.next;
              if (q != null) q = q.next;
          }
          if (carry > 0) {
              cur.next = new ListNode(carry);
          }
          return result.next;
      }
  }
  ```



#### [4. 寻找两个有序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

题面：给定两个大小为 m 和 n 的有序数组 nums1 和 nums2。请你找出这两个有序数组的中位数，并且要求算法的时间复杂度为 **O(log(m + n))**。你可以假设 nums1 和 nums2 不会同时为空。

+ 递归处理：借鉴二分查找的思想

  ```java
  public class Solution4 {
      public double findMedianSortedArrays(int[] nums1, int[] nums2) {
  
          int m = nums1.length;
          int n = nums2.length;
          if (m > n) {
              //需要保证数组nums1比nums2长度小些，
              // 以保证在求nums2划分下标的时候不会得出一个负数
              int[] temp = nums1;
              nums1 = nums2;
              nums2 = temp;
              //需要记住重新得出数组长度
              m = nums1.length;
              n = nums2.length;
          }
          int iMin = 0;
          int iMax = m;
          int halfLen = (m + n + 1) / 2;
          while (iMin <= iMax) {
              int i = (iMin + iMax) / 2;
              int j = halfLen - i;
              if (i > iMin && nums1[i - 1] > nums2[j]) {//i的位置太大
                  iMax = i - 1;
              } else if (i < iMax && nums1[i] < nums2[j - 1]) {//i的位置太小
                  iMin = i + 1;
              } else {//i的位置刚刚好
                  //可以处理中位数的结果
                  int maxLeft = 0;
                  if (i == 0) {
                      maxLeft = nums2[j - 1];
                  } else if (j == 0) {
                      maxLeft = nums1[i - 1];
                  }else {
                      maxLeft = Math.max(nums1[i - 1], nums2[j - 1]);
                  }
                  //如果两数组总长度是奇数，那么中位数只有一个
                  if ((m + n) % 2 != 0) {
                      return maxLeft;
                  }
  
                  //如果是偶数长度，中位数需要进行计算得出
                  int minRight = 0;
                  if (i == m) {
                      minRight = nums2[j];
                  } else if (j == n) {
                      minRight = nums1[i];
                  } else {
                      minRight = Math.min(nums1[i], nums2[j]);
                  }
                  return (double) (maxLeft + minRight) / 2;
              }
          }
  
          return 0.0;
      }
  }
  ```

  



#### [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

题面： 给定一个字符串 `s`，找到 `s` 中最长的回文子串。你可以假设 `s` 的最大长度为 1000。 

+ 暴力法：由于使用暴力法的过程当中，字符串是奇数长度或者偶数长度对于判断会有一定的影响，所以可以选择在每个字符中间插入一个其他字符，这样找出最长的回文半径以后除以2就是真正的半径了。
+ 公共子串：但是如果有具有公共子串但是不是回文的情况需要注意。





#### [11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

题面：给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。说明：你不能倾斜容器，且 n 的值至少为 2。

头脑清醒点，两条线可以不是相邻的！而且也不一定构成容纳最多水的这个水缸的两个ax和ay是最大的两个值。

+ 暴力法：一个个尝试，计算((y-x)^2)*Math.max(ax,ay)，得出最大的那个结果。时间复杂度O（n2），空间复杂度O(1)。

  ```java
  class Solution {
      public int maxArea(int[] height) {
          int maxArea = 0;
          for (int i = 0; i < height.length - 1; i++){
              for (int j = i + 1; j < height.length; j++){
                  maxArea = Math.max(maxArea, Math.min(height[i], height[j]) * (j - i));
              }
          }
          return maxArea;
      }
  }
  ```

+ 双指针：首尾分别设置一个指针，如果希望购成的水缸矩形面积尽量大，由于水缸受限于较短的一个边界，那么较长的一个边界如果往内侧移动只会减小面积，而较短的一个边界往里移动却有可能获得一个较长的边界，反而可以让水缸的矩形面积增大。时间复杂度恒定扫描O(n)，空间复杂度O(1)。

  ```jav
  class Solution {
      public int maxArea(int[] height) {
          int maxArea = 0;
          int left = 0;
          int right = height.length - 1;
          while (left < right) {
              maxArea = Math.max(maxArea, Math.min(height[left], height[right]) * (right - left));
              if (height[left] < height[right]) {
                  left++;
              } else {
                  right--;
              }
          }
          return maxArea;
      }
  }
  ```

  

#### [15. 三数之和](https://leetcode-cn.com/problems/3sum/)

题面：给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。**注意：**答案中不可以包含重复的三元组。

+ 定位一个数，另外的两个数使用首尾指针找出，注意去重。

  ```java
  class Solution {
      public List<List<Integer>> threeSum(int[] nums) {
      
          List<List<Integer>> list = new ArrayList<>();
          if (nums == null || nums.length < 3) {
              return list;
          }
          //先进行一个排序
          Arrays.sort(nums);
          //后面的操作根据排序进行
          int len = nums.length;
          int left;
          int right;
          int count = 0;
          //每次固定一个数，后面使用首尾指针，三个数得出是否是0，注意去重
          for (int i = 0; i < len; i++) {
              //如果第一个固定的数都已经大于0了，后面的数都不用看了
              if (nums[i] > 0) {
                  break;
              }
              if(i > 0 && nums[i] == nums[i-1]) continue; // 去重,这里往前面的数比较去重
              left = i + 1;
              right = len - 1;
              while (left < right) {
                  count = nums[i] + nums[left] + nums[right];
                  if (count == 0) {
                      list.add(Arrays.asList(nums[i], nums[left], nums[right]));
                      //去重
                      while (left < right && nums[left] == nums[left + 1]) {
                          //这个地方注意限定left < right
                          left++;
                      }
                      while (left < right && nums[right] == nums[right - 1]) {
                          right--;
                      }
                      left++;
                      right--;
                  } else if (count > 0) {
                      right--;
                  }else if(count < 0){
                      left++;
                  }
              }
          }
  
          return list;
      }
  }
  ```

  

#### [19. 删除链表的倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

题面：给定一个链表，删除链表的倒数第 *n* 个节点，并且返回链表的头结点。

+ 双指针：只需要两个指针一齐扫描一遍链表，第一个指针先走n步。时间复杂度O(n)，空间复杂度O(1)。

  ```java
  class Solution {
      public ListNode removeNthFromEnd(ListNode head, int n) {
          //这个头结点使用的方法可以的呢
          ListNode root = new ListNode(0);
          root.next = head;
          ListNode first = root;
          ListNode second = root;
          //先走n+1步
          for (int i = 0; i <= n; i++){
              first = first.next;
          }
          while(first != null){
              first = first.next;
              second = second.next;
          }
          second.next = second.next.next;
          return root.next;
      }
  }
  ```



#### [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

题面：将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

+ 迭代：时间复杂度O(m+n)，空间复杂度O(1)。

  ```java
  class Solution {
      public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
          //这个头结点的使用真的使后面的判断少了好多
          ListNode head = new ListNode(0);
          ListNode pre = head;
          while(l1 != null && l2 != null) {
              if(l1.val <= l2.val) {
                  pre.next = l1;
                  l1 = l1.next;
              } else {
                  pre.next = l2;
                  l2 = l2.next;
              }
              pre = pre.next; 
          }
          pre.next = l1 != null ? l1 : l2;
          return head.next;
      }
  }
  ```




#### [22. 括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

题面：数字 *n* 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

+ 递归：对于所有可能产生的序列一一检验，而对于一个序列的检验方法可以使用一个变量，对于左括号或者右括号计数。时间复杂度O(N）= (2 ^ 2n) * n ；空间复杂度O(N) = n。

  ```java
  class Solution {
      public List<String> generateParenthesis(int n) {
          List<String> combinations = new ArrayList<String>();
          generateAllString(new char[2 * n], 0, combinations);
          return combinations;
      }
  
      private void generateAllString(char[] cur, int pos, List<String> combinations) {
          if (pos == cur.length) {
              if (valid(cur)) {
                  combinations.add(new String(cur));
              }
          } else {
              cur[pos] = '(';
              generateAllString(cur, pos + 1, combinations);
              cur[pos] = ')';
              generateAllString(cur, pos + 1, combinations);
          }
      }
  
      private boolean valid(char[] cur) {
          int balance = 0;
          for (int i = 0; i < cur.length; i++) {
              if (cur[i] == '(') {
                  balance++;
              } else {
                  balance--;
              }
              //左括号都不可能比右括号少！
              if (balance < 0) {
                  return false;
              }
          }
          return balance == 0;
      }
  }
  ```

+ 回溯法：是递归方法的改进，跟踪左括号和右括号的数目在当前序列有效时添加左或者右括号，而不是每次都添加，当左括号不大于n就加左括号，当左括号大于右括号数目就加右括号。

  ```java
  class Solution {
      public List<String> generateParenthesis(int n) {
          List<String> result = new ArrayList<>();
          backTrack(result, new StringBuilder(), 0, 0, n);
          return result;
      }
  
      private void backTrack(List<String> result, StringBuilder cur, int open, int close, int max) {
          if (cur.length() == 2 * max) {
              result.add(cur.toString());
              return;
          }
  
          if (open < max) {
              //添加左括号
              cur.append('(');
              backTrack(result, cur, open + 1, close, max);
              cur.deleteCharAt(cur.length() - 1);
          }
  
          if (close < open) {
              //添加右括号
              cur.append(')');
              backTrack(result, cur, open, close + 1, max);
              cur.deleteCharAt(cur.length() - 1);
          }
      }
  }
  ```

  


#### [23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

题面：给你一个链表数组，每个链表都已经按升序排列。请你将所有链表合并到一个升序链表中，返回合并后的链表。

+ 逐一两两合并链表：时间复杂度O(kn)，空间复杂度O(1)。

  ```java
  
  ```

+ 分治法合并链表：时间复杂度O(nlogk) ，其中k 是链表的数目。可以在O(n) 的时间内合并两个有序链表，其中 n 是两个链表中的总节点数，将所有的合并进程加起来。空间复杂度O(1)。
  



#### [31. 下一个排列](https://leetcode-cn.com/problems/next-permutation/)

题面：实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中**下一个更大的排列**。如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。必须原地修改，只允许使用额外常数空间。以下是一些例子，输入位于左侧列，其相应输出位于右侧列。1,2,3 → 1,3,2	3,2,1 → 1,2,3	1,1,5 → 1,5,1

+ 可以从后往前找，如果遇到第i个数，第i-1个数比它小，就说明这个情况是可以得到一个比当前排列更大的排列的。只需要从第i个数以及它的后面的元素中找到一个比第i-1个数大的数当中的最小数和第i-1个数交换位置，交换位置以后还需要保持第i-1个数后面的数是递减的状态，这才是下一个更大的序列。

  ```java
  class Solution {
      public void nextPermutation(int[] nums) {
          int len = nums.length;
          boolean  flag = false;
          for(int i = len -1; i >= 0; i--) {
              if(i > 0 && nums[i] > nums[i - 1]) {
                  //就需要取一个i-1位置之后最小的一个比nums[i - 1]大的数交换位置
                  for (int j = len - 1; j >= i; j--) {
                      if(nums[j] > nums[i - 1]) {
                          int temp = nums[i - 1];
                          nums[i - 1] = nums[j];
                          nums[j] = temp;
                          flag = true;
                          Arrays.sort(nums,i,len);
                          break;
                      }
                  }
                  if(flag) break;
                  //这里真的要注意，不要遗漏了,否则可能在前面继续产生交换
              }
          }
          if(!flag) {
              Arrays.sort(nums);
          }
      }
  }
  ```






#### [39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

题面：给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。candidates 中的数字可以无限制重复被选取。说明：所有数字（包括 target）都是正整数。
解集不能包含重复的组合。 

+ 深度优先搜索：通过查找数组中是否还有元素能够组成target-candidates[i]，所以是一个递归的过程，为了避免重复元素的不同集合的出现，可以先将数组进行一个排序，遍历的过程中逐渐减小剩余的搜索范围。

  ```java
  class Solution {
      public List<List<Integer>> combinationSum(int[] candidates, int target) {
          List<List<Integer>> list = new ArrayList<>();
          //先对数组排个序，之后能够按序查找，减少重复
          Arrays.sort(candidates);
  
          Deque<Integer> path = new ArrayDeque<>();
          dfs(candidates, 0, target, path, list);
          return list;
      }
  
      public void dfs (int[] candidates, 
                       int beginIndex, 
                       int remainNum, 
                       Deque<Integer> path, 
                       List<List<Integer>> list) {
          //如果剩余的数为0，就是找到了可以组成target的所有数字了
          if (remainNum == 0) {
              list.add(new ArrayList<>(path));
              return;
          } 
  
          for (int i = beginIndex; i < candidates.length; i++) {
              if (remainNum - candidates[i] < 0) {
                  break;
              } 
              path.add(candidates[i]);
              dfs(candidates, i, remainNum - candidates[i], path, list);
              path.removeLast();
          }
      }
      
  }
  ```

  

#### [42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

题面：给定 *n* 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

+ 暴力法：遍历整个数组，依次找出当前元素的前面元素的最大值和后面元素的最大值，前面和后面的最大值当中的最小值就限制了当前位置能够储水的量。这样遍历一遍，将每个位置的最大储水量累计就是最终结果。时间复杂度O(n^2)，空间复杂度O(1)。

  ```java
  class Solution {
      public int trap(int[] height) {
      
          int result = 0;
          int len = height.length;
          //第一个位置和最后一个位置是不可能储水的
          for (int i = 1; i < len - 1; i++) {
              int maxLeft = 0;
              int maxRight = 0;
              //遍历到的位置，分别找出该位置前面部分和后面部分的最大值
              for (int j = i; j >= 0; j--) {
                  maxLeft = Math.max(maxLeft, height[j]);
              }
              for (int j = i; j < len; j++) {
                  maxRight = Math.max(maxRight, height[j]);
              }
              //那么当前位置的储水量就是
              int mount = Math.min(maxLeft, maxRight) - height[i];
              result += mount;
          }
  
          return result;
      }
  }
  ```

+ 双指针：也就两端向中间收缩的过程中，将两端的最大值保存了下来。时间复杂度O(n)，空间复杂度O(1)。

  ```java
  class Solution {
      public int trap(int[] height) {
          int len = height.length;
          int left = 0;
          int right = len - 1;
          int leftMax = 0;
          int rightMax = 0;
          int result = 0;
          while (left < right) {
              if (height[left] < height[right]) {
                  if (height[left] > leftMax) {
                      leftMax = height[left++];
                  } else {
                      result += (leftMax - height[left++]);
                  }
              } else {
                  if (height[right] > rightMax) {
                      rightMax = height[right--];
                  } else {
                      result += (rightMax - height[right--]);
                  }
              }
          }
  
          return result;
      }
  }
  ```

  

#### [46. 全排列](https://leetcode-cn.com/problems/permutations/)

题面：给定一个 **没有重复** 数字的序列，返回其所有可能的全排列。

+ 递归

  ```java
  class Solution {
      public List<List<Integer>> permute(int[] nums) {
          List<List<Integer>> list = new ArrayList<>();
          int[] isVisited = new int[nums.length];
          Deque<Integer> queue = new ArrayDeque<>();
          process(list,queue,nums,isVisited);
          return list;
      }
  
      public void process(
          List<List<Integer>> list, 
          Deque<Integer> queue, 
          int[] nums, 
          int[] isVisited) {
  
          if (queue.size() == nums.length) {
              list.add(new ArrayList<>(queue));
              return;
          }
          for (int i = 0; i < nums.length; i++) {
              if (isVisited[i] == 1) {
                  continue;
              }
              //添加到队列中，并改标记位
              queue.add(nums[i]);
              isVisited[i] = 1;
              //递归
              process(list,queue,nums,isVisited);
              //从队列中删除，并恢复标记位
              queue.removeLast();
              isVisited[i] = 0;
          }
      }
  }
  ```

  



#### [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

题面：给定一个非负整数数组，你最初位于数组的第一个位置。数组中的每个元素代表你在该位置可以跳跃的**最大长度**。判断你是否能够到达最后一个位置。

+ 回溯法：每一次按照当前位置可以跳出的最大距离跳到下一位置，如果可以到达最后了就返回true；如果没有到达最后一个位置，就通过回溯，将上一次跳跃的距离减小，查看最大距离之内的每一次跳跃能不能促成最后的成功。

  ```java
  //不能在LeetCode上AC，超出时间限制
  class Solution {
      public boolean canJump(int[] nums) {
          if (nums == null || nums.length == 0) {
              return false;
          }
          return canJumpFromPosition (0, nums);
      }
  
      public boolean canJumpFromPosition (int position, int[] nums) {
          if (position + nums[position] >= nums.length - 1) {
              return true;
          }
          //如果说没有直接跳到最后一个位置的话
          //接着就枚举回溯吧
          int maxDistance = Math.min(position + nums[position], nums.length - 1);
          for (int nextPosition = maxDistance; nextPosition > position; nextPosition--) {
              if(canJumpFromPosition(nextPosition, nums)) return true;
          }
          return false;
      }
  }
  ```

+ 自底向上的动态规划：从最后的位置开始往前推，依次标记每个位置是否是可达最后位置的元素。每一个前面的元素如果跳到后面的不可达元素，就是永远不可达的，如果能够跳到后面的可达元素，那也就是可达的。时间复杂度O(n^2)，空间复杂度O(n)。

  ```java
  class Solution {
      public static final int IS_ARRIVED = 1;
  
      public boolean canJump(int[] nums) {
          int[] flag = new int[nums.length];
          int des = nums.length - 1;
          flag[des] = IS_ARRIVED;
          for (int i = des - 1; i >= 0; i--) {
              int maxDistance = Math.min(nums[i] + i, des);
              for(int j = maxDistance; j > i; j--) {
                  if(flag[j] == IS_ARRIVED) {
                      flag[i] = IS_ARRIVED;
                      break;               
                  }
              }
          }
          return flag[0] == IS_ARRIVED ? true : false;
      }
  }
  ```




#### [62. 不同路径](https://leetcode-cn.com/problems/unique-paths/)

题面：一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。问总共有多少条不同的路径？

+ 动态规划：`dp[i][j]` 是到达 `i, j` 最多路径，动态方程：`dp[i][j] = dp[i-1][j] + dp[i][j-1]`。

  ```java
  class Solution {
      public int uniquePaths(int m, int n) {
          int[][] result = new int[m][n];
          //初始化边界部分的值
          for (int i = 0; i < m; i++) {
              result[i][0] = 1;
          }
          for (int i = 0; i < n; i++) {
              result[0][i] = 1;
          }
          for (int i = 1; i < m; i++) {
              for (int j = 1; j < n; j++) {
                  result[i][j] = result[i - 1][j] + result[i][j - 1];
              }
          }
          return result[m - 1][n - 1];
      }
  }
  
  //优化1:由于d[i][j]至于当前行左边的状态有关或者和前面一行的状态有关，所以可以优化成两个一维数组。
  
  ```





#### [70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

题面：假设你正在爬楼梯。需要 n 阶你才能到达楼顶。每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？给定 n 是一个正整数。

+ 动态规划：到第n阶的楼梯，可以通过第n-1阶走一步，或者通过第n-2阶走两步的两种方式到达，而对于第n-1阶和第n-2阶也是同样，所以可以得出动态方程dp[n] = dp[n-1] + dp[n-2]。时间复杂度O(n)，空间复杂度O(n)。

  ```java
  class Solution {
      public int climbStairs(int n) {
          if(n < 1) {
              return 0;
          }
  
          int[] dp = new int[n + 1];
          dp[1] = 1;
          if (n >= 2){
              dp[2] = 2;
              for(int i = 3; i <= n; i++) {
                  dp[i] = dp[i - 1] + dp[i - 2];
              }
          }
          return dp[n];
      }
  }
  ```

  




#### [84. 柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

题面：给定 *n* 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。求在该柱状图中，能够勾勒出来的矩形的最大面积。

+ 暴力：两层循环，每两个圆柱作为边界都可以得到一个矩形，而面积则是受其中最短圆柱限制。时间复杂度O(n^2)。

  ```java
  class Solution {
      public int largestRectangleArea(int[] heights) {
          if(heights == null || heights.length == 0) {
              return 0;
          }
          int maxArea = 0;
          for(int i = 0; i < heights.length; i++) {
              int minHeight = Integer.MAX_VALUE;
              for(int j = i; j < heights.length; j++) {
                  minHeight = Math.min(minHeight, heights[j]);
                  int area = (j - i + 1) * minHeight;
                  maxArea = Math.max(area, maxArea);
              }
          }
          return maxArea;
      }
  }
  ```

+ 使用栈：保持栈中下标对应的圆柱高度成单调增的状态。时间复杂度为O(n)，空间复杂度O(n)。

  ```java
  class Solution {
      public int largestRectangleArea(int[] heights) {
          if(heights == null || heights.length == 0){
              return 0;
          }
  
          Stack<Integer> stack = new Stack<>();
          stack.push(-1); //stack入栈的内容为下标,初始化为-1
          int maxArea = 0;
  
          for (int i = 0; i < heights.length; i++) {
              while(stack.peek() != -1 && heights[i] <= heights[stack.peek()]){
                  maxArea = Math.max(maxArea, heights[stack.pop()] * (i - stack.peek() - 1));
              }
              stack.push(i);
          }
  
          //收尾工作，最后可能栈中会存在没有处理完的内容
          while(stack.peek() != -1){
              maxArea = Math.max(maxArea, heights[stack.pop()] * (heights.length - 1 - stack.peek()));
          }
  
          return maxArea;
      }
  }
  ```

  



#### [85. 最大矩形](https://leetcode-cn.com/problems/maximal-rectangle/)

题面：给定一个仅包含 0 和 1 的二维二进制矩阵，找出只包含 1 的最大矩形，并返回其面积。

+ 暴力：找两个对角点，再遍历对角点形成的矩形内部。时间复杂度为O((N^3)*(M^3))。

+ 动态规划：使用辅助二维数据，记录每一行每一个位置可达到的最大连续'1'的长度，然后纵向往上延伸，匹配出矩形，不断比较得出最大矩形面积。时间复杂度为O((N^2)*M)。

  ```java
  class Solution {
      public int maximalRectangle(char[][] matrix) {
          if(matrix == null || matrix.length == 0) {
              return 0;
          }
  
          //使用一个辅助二维数组，记录每个位置对应的横向可达最大宽度
          int[][] dp = new int[matrix.length][matrix[0].length];
          int maxArea = 0;
  
          for(int i = 0; i < matrix.length; i++){
              for(int j = 0; j< matrix[0].length; j++){
                  if (matrix[i][j] == '1'){
                      dp[i][j] = j == 0 ? 1 : dp[i][j - 1] + 1;
                      //往上适配矩形，计算可达到的最大面积
                      int width = dp[i][j];
                      for(int k = i; k >= 0; k--) {
                          //可能被限制的矩形的宽度
                          width = Math.min(width, dp[k][j]);
                          int area = width*(i - k + 1);
                          maxArea = Math.max(maxArea, area);
                      }
                  }
              }
          }
          return maxArea;
      }
  }
  ```

+ 使用栈：（没整理完）



#### [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

题面：给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。如果你最多只允许完成一笔交易（即买入和卖出一支股票一次），设计一个算法来计算你所能获取的最大利润。注意：你不能在买入股票前卖出股票。

+ 一次遍历：利用一个哨兵记录最小值。

  ```java
  class Solution {
      public int maxProfit(int[] prices) {
          if (prices == null || prices.length < 2){
              return 0;
          }
  
          int maxValue = 0;
          int minPrice = Integer.MAX_VALUE;
          for (int i  = 0; i < prices.length; i++){
              if (prices[i] < minPrice) {
                  minPrice = prices[i];
              } else {
                  maxValue = Math.max(maxValue, prices[i] - minPrice);
              }
          }
          return maxValue;
      }
  }
  ```

  



#### [221. 最大正方形](https://leetcode-cn.com/problems/maximal-square/)

题面：在一个由 0 和 1 组成的二维矩阵内，找到只包含 1 的最大正方形，并返回其面积。

+ 动态规划：使用一个辅助矩阵，记录到达每个位置可形成的最大正方形边长，动态方程为 dp[i][j] = Math.min(Math.min(dp[i - 1][j],dp[i][j - 1]), dp[i - 1])，画图更好理解。时间复杂度O(nm)，空间复杂度O(nm)。

  ```java
  class Solution {
      public int maximalSquare(char[][] matrix) {
  
          if(matrix == null || matrix.length == 0){
              return 0;
          }
  
          int maxLength = 0;
          int[][] dp = new int[matrix.length+1][matrix[0].length+1];
          for(int i = 1; i <= matrix.length; i++){
              for (int j = 1; j <= matrix[0].length; j++){
                  if (matrix[i-1][j-1] == '1') {
                      dp[i][j] = Math.min(Math.min(dp[i - 1][j],dp[i][j - 1]), dp[i - 1][j - 1]) + 1;
                      maxLength = Math.max(maxLength, dp[i][j]);
                  }
              }
          }
          return maxLength * maxLength;
      }
  }
  
  
  //动态规划空间复杂度优化，可以改成一维数组。
  ```



#### [300. 最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

题面：给定一个无序的整数数组，找到其中最长上升子序列的长度。

+ 动态规划：首先这里自序列可以是不连续的。时间复杂度O(n^2)，空间复杂度O(n)。

  ```java
  public class Solution300 {
      public int lengthOfLIS(int[] nums) {
  
          if (nums == null || nums.length == 0) {
              return 0;
          }
  
          //辅助数组，记录到某一个位置能形成的最大子序列的长度
          int[] dp = new int[nums.length];
          dp[0] = 1;
          int maxResult = 1; //子序列长度至少也是个1
          for (int i = 1; i < nums.length; i++) {
              int maxLength = 0;
              for (int j = 0; j < i; j++) {
                  //只要前面有比nums[i]小的数字，就能形成一个递增子序列
                  //记录下这个最大长度
                  if (nums[j] < nums[i]){
                      maxLength = Math.max(maxLength, dp[j]);
                  }
              }
              dp[i] = maxLength + 1;//更新位置i的最大长度
              maxResult = Math.max(maxResult, dp[i]);
          }
          return maxResult;
      }
  }
  
  ```

  

#### [322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)

题面：给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。（可以认为每种硬币的数量是无限的。）

+ 



#### [647. 回文子串](https://leetcode-cn.com/problems/palindromic-substrings/)

题面：给定一个字符串，你的任务是计算这个字符串中有多少个回文子串。具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被计为是不同的子串。

+ 中心往两侧延伸：避免奇数或者偶数个字符的判断麻烦，可以假设字符之间都插入了一个其他字符，这样就永远为奇数个字符了，接着将所有字符遍历一遍共len*2 - 1次，每次往两边延伸的时候左右字符的计算方式可以是left = i/2以及right = left + i%2，left和right可能相等。时间复杂度是O(n^2)。

  ```java
  class Solution {
      public int countSubstrings(String s) {
          int len = s.length();
          int count = 0;
          for (int i = 0; i < len*2 - 1; i++) {
              int left = i/2;
              int right = left + i%2;
              while (left >= 0 && right < len && s.charAt(left) == s.charAt(right)) {
                  count++;
                  left--;
                  right++;
              }
          }
          return count;
      }
  }
  ```





-----

**hot100出现频率列表**

| 完成 | 序号 | 题目                                                         | 题解                                                         | 通过率 | 难度 |      |
| ---- | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------ | ---- | ---- |
|      | 1    | [两数之和](https://leetcode-cn.com/problems/two-sum)         | [2925](https://leetcode-cn.com/problems/two-sum/solution)    | 48.1%  | 简单 |      |
|      | 4    | [寻找两个有序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays) | [866](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/solution) | 37.3%  | 困难 |      |
|      | 206  | [反转链表](https://leetcode-cn.com/problems/reverse-linked-list) | [4777](https://leetcode-cn.com/problems/reverse-linked-list/solution) | 68.4%  | 简单 |      |
|      | 5    | [最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring) | [918](https://leetcode-cn.com/problems/longest-palindromic-substring/solution) | 29.3%  | 中等 |      |
|      | 3    | [无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters) | [1760](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/solution) | 33.7%  | 中等 |      |
|      | 2    | [两数相加](https://leetcode-cn.com/problems/add-two-numbers) | [1881](https://leetcode-cn.com/problems/add-two-numbers/solution) | 37.1%  | 中等 |      |
|      | 146  | [LRU缓存机制](https://leetcode-cn.com/problems/lru-cache)    | [225](https://leetcode-cn.com/problems/lru-cache/solution)   | 47.0%  | 中等 |      |
|      | 21   | [合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists) | [800](https://leetcode-cn.com/problems/merge-two-sorted-lists/solution) | 61.0%  | 简单 |      |
|      | 85   | [最大矩形](https://leetcode-cn.com/problems/maximal-rectangle) | [107](https://leetcode-cn.com/problems/maximal-rectangle/solution) | 45.3%  | 困难 |      |
|      | 15   | [三数之和](https://leetcode-cn.com/problems/3sum)            | [714](https://leetcode-cn.com/problems/3sum/solution)        | 26.4%  | 中等 |      |
|      | 221  | [最大正方形](https://leetcode-cn.com/problems/maximal-square) | [157](https://leetcode-cn.com/problems/maximal-square/solution) | 39.4%  | 中等 |      |
|      | 53   | [最大子序和](https://leetcode-cn.com/problems/maximum-subarray) | [811](https://leetcode-cn.com/problems/maximum-subarray/solution) | 50.1%  | 简单 |      |
|      | 46   | [全排列](https://leetcode-cn.com/problems/permutations)      | [562](https://leetcode-cn.com/problems/permutations/solution) | 74.7%  | 中等 |      |
|      | 23   | [合并K个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists) | [533](https://leetcode-cn.com/problems/merge-k-sorted-lists/solution) | 49.9%  | 困难 |      |
|      | 10   | [正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching) | [346](https://leetcode-cn.com/problems/regular-expression-matching/solution) | 27.2%  | 困难 |      |
|      | 394  | [字符串解码](https://leetcode-cn.com/problems/decode-string) | [212](https://leetcode-cn.com/problems/decode-string/solution) | 49.6%  | 中等 |      |
|      | 148  | [排序链表](https://leetcode-cn.com/problems/sort-list)       | [248](https://leetcode-cn.com/problems/sort-list/solution)   | 65.0%  | 中等 |      |
|      | 70   | [爬楼梯](https://leetcode-cn.com/problems/climbing-stairs)   | [804](https://leetcode-cn.com/problems/climbing-stairs/solution) | 48.3%  | 简单 |      |
|      | 253  | [会议室 II](https://leetcode-cn.com/problems/meeting-rooms-ii) | [88](https://leetcode-cn.com/problems/meeting-rooms-ii/solution) | 42.7%  | 中等 |      |
|      | 42   | [接雨水](https://leetcode-cn.com/problems/trapping-rain-water) | [841](https://leetcode-cn.com/problems/trapping-rain-water/solution) | 50.6%  | 困难 |      |
|      | 301  | [删除无效的括号](https://leetcode-cn.com/problems/remove-invalid-parentheses) | [53](https://leetcode-cn.com/problems/remove-invalid-parentheses/solution) | 45.7%  | 困难 |      |
|      | 11   | [盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water) | [705](https://leetcode-cn.com/problems/container-with-most-water/solution) | 62.4%  | 中等 |      |
|      | 32   | [最长有效括号](https://leetcode-cn.com/problems/longest-valid-parentheses) | [299](https://leetcode-cn.com/problems/longest-valid-parentheses/solution) | 30.1%  | 困难 |      |
|      | 56   | [合并区间](https://leetcode-cn.com/problems/merge-intervals) | [443](https://leetcode-cn.com/problems/merge-intervals/solution) | 41.0%  | 中等 |      |
|      | 543  | [二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree) | [573](https://leetcode-cn.com/problems/diameter-of-binary-tree/solution) | 49.4%  | 简单 |      |
|      | 20   | [有效的括号](https://leetcode-cn.com/problems/valid-parentheses) | [1382](https://leetcode-cn.com/problems/valid-parentheses/solution) | 41.3%  | 简单 |      |
|      | 124  | [二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum) | [205](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/solution) | 39.8%  | 困难 |      |
|      | 76   | [最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring) | [163](https://leetcode-cn.com/problems/minimum-window-substring/solution) | 35.6%  | 困难 |      |
|      | 406  | [根据身高重建队列](https://leetcode-cn.com/problems/queue-reconstruction-by-height) | [95](https://leetcode-cn.com/problems/queue-reconstruction-by-height/solution) | 64.1%  | 中等 |      |
|      | 33   | [搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array) | [499](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/solution) | 36.7%  | 中等 |      |
|      | 215  | [数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array) | [443](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/solution) | 61.9%  | 中等 |      |
|      | 31   | [下一个排列](https://leetcode-cn.com/problems/next-permutation) | [384](https://leetcode-cn.com/problems/next-permutation/solution) | 33.2%  | 中等 |      |
|      | 121  | [买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock) | [1114](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/solution) | 53.8%  | 简单 |      |
|      | 322  | [零钱兑换](https://leetcode-cn.com/problems/coin-change)     | [608](https://leetcode-cn.com/problems/coin-change/solution) | 38.9%  | 中等 |      |
|      | 19   | [删除链表的倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list) | [862](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/solution) | 38.3%  | 中等 |      |
|      | 200  | [岛屿数量](https://leetcode-cn.com/problems/number-of-islands) | [380](https://leetcode-cn.com/problems/number-of-islands/solution) | 48.0%  | 中等 |      |
|      | 312  | [戳气球](https://leetcode-cn.com/problems/burst-balloons)    | [53](https://leetcode-cn.com/problems/burst-balloons/solution) | 58.7%  | 困难 |      |
|      | 239  | [滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum) | [286](https://leetcode-cn.com/problems/sliding-window-maximum/solution) | 46.2%  | 困难 |      |
|      | 128  | [最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence) | [170](https://leetcode-cn.com/problems/longest-consecutive-sequence/solution) | 48.5%  | 困难 |      |
|      | 226  | [翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree) | [370](https://leetcode-cn.com/problems/invert-binary-tree/solution) | 74.8%  | 简单 |      |
|      | 169  | [多数元素](https://leetcode-cn.com/problems/majority-element) | [886](https://leetcode-cn.com/problems/majority-element/solution) | 62.9%  | 简单 |      |
|      | 94   | [二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal) | [402](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/solution) | 71.0%  | 中等 |      |
|      | 72   | [编辑距离](https://leetcode-cn.com/problems/edit-distance)   | [392](https://leetcode-cn.com/problems/edit-distance/solution) | 59.2%  | 困难 |      |
|      | 48   | [旋转图像](https://leetcode-cn.com/problems/rotate-image)    | [425](https://leetcode-cn.com/problems/rotate-image/solution) | 67.7%  | 中等 |      |
|      | 240  | [搜索二维矩阵 II](https://leetcode-cn.com/problems/search-a-2d-matrix-ii) | [135](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/solution) | 40.0%  | 中等 |      |
|      | 78   | [子集](https://leetcode-cn.com/problems/subsets)             | [415](https://leetcode-cn.com/problems/subsets/solution)     | 77.1%  | 中等 |      |
|      | 105  | [从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal) | [275](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/solution) | 64.8%  | 中等 |      |
|      | 64   | [最小路径和](https://leetcode-cn.com/problems/minimum-path-sum) | [439](https://leetcode-cn.com/problems/minimum-path-sum/solution) | 65.3%  | 中等 |      |
|      | 102  | [二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal) | [510](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/solution) | 61.4%  | 中等 |      |
|      | 309  | [最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown) | [125](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/solution) | 53.1%  | 中等 |      |
|      | 104  | [二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree) | [574](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/solution) | 72.8%  | 简单 |      |
|      | 155  | [最小栈](https://leetcode-cn.com/problems/min-stack)         | [379](https://leetcode-cn.com/problems/min-stack/solution)   | 52.5%  | 简单 |      |
|      | 739  | [每日温度](https://leetcode-cn.com/problems/daily-temperatures) | [264](https://leetcode-cn.com/problems/daily-temperatures/solution) | 60.1%  | 中等 |      |
|      | 621  | [任务调度器](https://leetcode-cn.com/problems/task-scheduler) | [94](https://leetcode-cn.com/problems/task-scheduler/solution) | 48.3%  | 中等 |      |
|      | 234  | [回文链表](https://leetcode-cn.com/problems/palindrome-linked-list) | [413](https://leetcode-cn.com/problems/palindrome-linked-list/solution) | 41.4%  | 简单 |      |
|      | 141  | [环形链表](https://leetcode-cn.com/problems/linked-list-cycle) | [538](https://leetcode-cn.com/problems/linked-list-cycle/solution) | 47.4%  | 简单 |      |
|      | 448  | [找到所有数组中消失的数字](https://leetcode-cn.com/problems/find-all-numbers-disappeared-in-an-array) | [193](https://leetcode-cn.com/problems/find-all-numbers-disappeared-in-an-array/solution) | 57.6%  | 简单 |      |
|      | 101  | [对称二叉树](https://leetcode-cn.com/problems/symmetric-tree) | [506](https://leetcode-cn.com/problems/symmetric-tree/solution) | 50.8%  | 简单 |      |
|      | 560  | [和为K的子数组](https://leetcode-cn.com/problems/subarray-sum-equals-k) | [113](https://leetcode-cn.com/problems/subarray-sum-equals-k/solution) | 44.6%  | 中等 |      |
|      | 279  | [完全平方数](https://leetcode-cn.com/problems/perfect-squares) | [241](https://leetcode-cn.com/problems/perfect-squares/solution) | 55.4%  | 中等 |      |
|      | 136  | [只出现一次的数字](https://leetcode-cn.com/problems/single-number) | [472](https://leetcode-cn.com/problems/single-number/solution) | 66.3%  | 简单 |      |
|      | 287  | [寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number) | [193](https://leetcode-cn.com/problems/find-the-duplicate-number/solution) | 63.7%  | 中等 |      |
|      | 581  | [最短无序连续子数组](https://leetcode-cn.com/problems/shortest-unsorted-continuous-subarray) | [121](https://leetcode-cn.com/problems/shortest-unsorted-continuous-subarray/solution) | 34.5%  | 简单 |      |
|      | 55   | [跳跃游戏](https://leetcode-cn.com/problems/jump-game)       | [483](https://leetcode-cn.com/problems/jump-game/solution)   | 38.6%  | 中等 |      |
|      | 79   | [单词搜索](https://leetcode-cn.com/problems/word-search)     | [267](https://leetcode-cn.com/problems/word-search/solution) | 41.2%  | 中等 |      |
|      | 283  | [移动零](https://leetcode-cn.com/problems/move-zeroes)       | [652](https://leetcode-cn.com/problems/move-zeroes/solution) | 60.4%  | 简单 |      |
|      | 236  | [二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree) | [221](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/solution) | 61.3%  | 中等 |      |
|      | 617  | [合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees) | [226](https://leetcode-cn.com/problems/merge-two-binary-trees/solution) | 75.8%  | 简单 |      |
|      | 207  | [课程表](https://leetcode-cn.com/problems/course-schedule)   | [197](https://leetcode-cn.com/problems/course-schedule/solution) | 49.9%  | 中等 |      |
|      | 22   | [括号生成](https://leetcode-cn.com/problems/generate-parentheses) | [811](https://leetcode-cn.com/problems/generate-parentheses/solution) | 75.2%  | 中等 |      |
|      | 17   | [电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number) | [712](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/solution) | 53.2%  | 中等 |      |
|      | 198  | [打家劫舍](https://leetcode-cn.com/problems/house-robber)    | [513](https://leetcode-cn.com/problems/house-robber/solution) | 44.2%  | 简单 |      |
|      | 160  | [相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists) | [420](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/solution) | 54.0%  | 简单 |      |
|      | 152  | [乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray) | [228](https://leetcode-cn.com/problems/maximum-product-subarray/solution) | 37.8%  | 中等 |      |
|      | 39   | [组合总和](https://leetcode-cn.com/problems/combination-sum) | [396](https://leetcode-cn.com/problems/combination-sum/solution) | 68.8%  | 中等 |      |
|      | 34   | [在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array) | [601](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/solution) | 39.3%  | 中等 |      |
|      | 62   | [不同路径](https://leetcode-cn.com/problems/unique-paths)    | [513](https://leetcode-cn.com/problems/unique-paths/solution) | 59.9%  | 中等 |      |
|      | 96   | [不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees) | [233](https://leetcode-cn.com/problems/unique-binary-search-trees/solution) | 65.2%  | 中等 |      |
|      | 139  | [单词拆分](https://leetcode-cn.com/problems/word-break)      | [215](https://leetcode-cn.com/problems/word-break/solution)  | 44.4%  | 中等 |      |
|      | 438  | [找到字符串中所有字母异位词](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string) | [127](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/solution) | 42.5%  | 中等 |      |
|      | 114  | [二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list) | [245](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/solution) | 68.3%  | 中等 |      |
|      | 238  | [除自身以外数组的乘积](https://leetcode-cn.com/problems/product-of-array-except-self) | [127](https://leetcode-cn.com/problems/product-of-array-except-self/solution) | 67.5%  | 中等 |      |
|      | 84   | [柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram) | [233](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/solution) | 39.3%  | 困难 |      |
|      | 300  | [最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence) | [553](https://leetcode-cn.com/problems/longest-increasing-subsequence/solution) | 44.2%  | 中等 |      |
|      | 142  | [环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii) | [320](https://leetcode-cn.com/problems/linked-list-cycle-ii/solution) | 49.7%  | 中等 |      |
|      | 647  | [回文子串](https://leetcode-cn.com/problems/palindromic-substrings) | [129](https://leetcode-cn.com/problems/palindromic-substrings/solution) | 61.3%  | 中等 |      |
|      | 75   | [颜色分类](https://leetcode-cn.com/problems/sort-colors)     | [380](https://leetcode-cn.com/problems/sort-colors/solution) | 54.7%  | 中等 |      |
|      | 297  | [二叉树的序列化与反序列化](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree) | [119](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/solution) | 46.2%  | 困难 |      |
|      | 347  | [前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements) | [242](https://leetcode-cn.com/problems/top-k-frequent-elements/solution) | 61.0%  | 中等 |      |
|      | 437  | [路径总和 III](https://leetcode-cn.com/problems/path-sum-iii) | [219](https://leetcode-cn.com/problems/path-sum-iii/solution) | 54.7%  | 简单 |      |
|      | 494  | [目标和](https://leetcode-cn.com/problems/target-sum)        | [111](https://leetcode-cn.com/problems/target-sum/solution)  | 44.0%  | 中等 |      |
|      | 208  | [实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree) | [186](https://leetcode-cn.com/problems/implement-trie-prefix-tree/solution) | 66.3%  | 中等 |      |
|      | 49   | [字母异位词分组](https://leetcode-cn.com/problems/group-anagrams) | [243](https://leetcode-cn.com/problems/group-anagrams/solution) | 61.1%  | 中等 |      |
|      | 98   | [验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree) | [406](https://leetcode-cn.com/problems/validate-binary-search-tree/solution) | 29.7%  | 中等 |      |
|      | 337  | [打家劫舍 III](https://leetcode-cn.com/problems/house-robber-iii) | [139](https://leetcode-cn.com/problems/house-robber-iii/solution) | 56.6%  | 中等 |      |
|      | 416  | [分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum) | [134](https://leetcode-cn.com/problems/partition-equal-subset-sum/solution) | 46.5%  | 中等 |      |
|      | 538  | [把二叉搜索树转换为累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree) | [142](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/solution) | 60.2%  | 简单 |      |
|      | 338  | [比特位计数](https://leetcode-cn.com/problems/counting-bits) | [212](https://leetcode-cn.com/problems/counting-bits/solution) | 75.0%  | 中等 |      |
|      | 461  | [汉明距离](https://leetcode-cn.com/problems/hamming-distance) | [266](https://leetcode-cn.com/problems/hamming-distance/solution) | 75.6%  | 简单 |      |
|      | 399  | [除法求值](https://leetcode-cn.com/problems/evaluate-division) | [91](https://leetcode-cn.com/problems/evaluate-division/solution) | 53.7%  | 中等 |      |