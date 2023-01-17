#### [面试题03. 数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

题面：找出数组中重复的数字。在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

+ 排序

+ 哈希表

+ 遍历数组：时间复杂度只要O(n)，空间复杂度O(1)。

  ```java
  class Solution {
      public int findRepeatNumber(int[] nums) {
          int num = -1;
          //遍历数组，由于题目条件长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内
          //也就是说如果给他们排序，没有出现重复的数字会和它的下标的值相等
          //那么我们只要对和下标不相等的值进行处理
          for(int i = 0; i < nums.length; i++){
              if (nums[i] != i){
                  if(nums[i] == nums[nums[i]]){
                      //找到重复的数字了
                      num = nums[i];
                      break;
                  }else {
                      swap(nums,nums[i], i);//这样nums[i]就可以去他对应的下标位置了
                  }
              }
          }
          return num;
      }
  
      static public void swap(int[] nums, int x, int y){
          int temp = nums[x];
          nums[x] = nums[y];
          nums[y] = temp;
      }
  }
  ```



#### [剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

题面：在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

+ 线性查找：例如从右上角开始查找，根据题干中的递增规律，每次可以排除一行或者一列的查找范围，最终找到目标值。时间复杂度O(m+n)。

  ```java
  class Solution {
      public boolean findNumberIn2DArray(int[][] matrix, int target) {
          if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
              return false;
          }
  
          int rows = matrix.length;
          int columns = matrix[0].length;
  
          //从右上角开始查找
          int row = 0;
          int column = columns - 1;
          while(row < rows && column >= 0) {
              int num = matrix[row][column];
              if (num == target) {
                  return true;
              } else if (num < target) {
                  row++;
              } else if (num > target) {
                  column--;
              }
          }
          return false;
      }
  }
  ```



#### [面试题05. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

题面：请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

+ ```java
  class Solution {
      public String replaceSpace(String s) {
          if (s == null || s.length() == 0) {
              return s;
          }
          StringBuilder res = new StringBuilder();
          for (int i = 0; i < s.length(); i++){
              char c = s.charAt(i);
              if (c == ' '){
                  res.append("%20");
              } else {
                  res.append(c);
              }
          }
          return res.toString();
      }
  }
  ```

+ ```java
  class Solution {
      public String replaceSpace(String s) {
          return  s.replaceAll(" ", "%20");
      }
  }
  ```



#### [面试题06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

题面：输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

+ 使用栈

  ```java
  class Solution {
      public int[] reversePrint(ListNode head) {
  
          Stack<Integer> stack = new Stack<>();
          ListNode node = head;
          while(node != null) {
              stack.push(node.val);
              node = node.next;
          } 
  
          int[] result = new int[stack.size()];
          int index = 0;
          while(!stack.isEmpty()) {
              result[index++] = stack.pop();
          }
  
          return result;
      }
  }
  ```

+ 递归



#### [面试题07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

题面：输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

+ 递归

  ```java
  class Solution {
      public TreeNode buildTree(int[] preorder, int[] inorder) {
          if (preorder == null || inorder == null || inorder.length == 0 || preorder.length == 0) {
              return null;
          }
          return buildTree(preorder, inorder, 0, preorder.length - 1, 0, inorder.length - 1);
      }
  
  
      public TreeNode buildTree(int[] preorder, int[] inorder, int preLeft, int preRight, int inLeft, int inRight){
  
          if (preRight < preLeft) {
              return null;
          } else if(preRight == preLeft){
              return new TreeNode(preorder[preLeft]);
          }
  
          TreeNode rootNode = new TreeNode(preorder[preLeft]);
  
          //preOder中第一个节点就是根节点，对应于inOrder中的某个节点
          for (int i = inLeft; i <= inRight; i++) {
              if (inorder[i] == preorder[preLeft]) {
                  int sonLeftLen = i - inLeft;
                  int sonRightLen = inRight - i;
                  rootNode.left = buildTree(preorder, inorder, 
                      preLeft + 1, preLeft + sonLeftLen, i - sonLeftLen, i - 1);
                  rootNode.right = buildTree(preorder, inorder, 
                      preLeft + sonLeftLen + 1, preRight, i + 1, i + sonRightLen);
              }
          }
          return rootNode;
      }
  }
  ```

+ 迭代（没有梳理）



#### [面试题09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

题面：用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

+ 一个栈主要负责入队，一个栈主要负责出队，避免每次出队都将元素倒入另一个栈中。

  ```java
  class CQueue {
      
      Stack<Integer> stackA;
      Stack<Integer> stackB;
  
      public CQueue() {
          stackA = new Stack<>();//主要负责入队列
          stackB = new Stack<>();//主要负责将stackA中进栈的元素反向出栈
      }
      
      public void appendTail(int value) {
          stackA.push(value);
      }
      
      public int deleteHead() {
          if (!stackB.isEmpty()) {
              return stackB.pop();
          } else {
              while(!stackA.isEmpty()){
                  stackB.push(stackA.pop());
              }
              return stackB.isEmpty() ? -1 : stackB.pop();
          }
      }
  }
  ```




#### [面试题10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

题面：写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项。斐波那契数列的定义如下：

F(0) = 0,   F(1) = 1，F(N) = F(N - 1) + F(N - 2)， 其中 N > 1.斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

+ 递归

  ```java
  class Solution {
      public int fib(int n) {
          if (n < 2) {
              return n;
          }
  
          int sum = fib(n - 1) + fib(n - 2);
          return sum;
      }
  }
  ```

+ 动态规划

  ```java
  class Solution {
      public int fib(int n) {
          if (n < 1) {
              return 0;
          } else {
              int[] nums = new int[n + 1];
              nums[0] = 0;
              nums[1] = 1;
              for (int i = 2; i <= n; i++) {
                  nums[i] = (nums[i -1] + nums[i - 2]) % 1000000007;
              }   
              return nums[n];
          }
      }
  }
  ```

+ 递推

  ```java
  class Solution {
      public int fib(int n) {
          if (n < 2){
              return n;
          }
  
          int temp0 = 0;
          int temp1 = 1;
          int res = temp0 + temp1;
          for (int i = 2; i <= n; i++) {
              res = temp0 + temp1;
              temp0 = temp1;
              temp1 = res;
          }
          return res;
      }
  }
  ```



#### [面试题10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

题面：一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

+ 动态规划：

  ```java
  class Solution {
      public int numWays(int n) {
          int dp[] = new int[n + 1];
          dp[0] = 1;
          if (n >= 1) {
              dp[1] = 1;
          }
          if (n >= 2) {
              dp[2] = 2;
              for (int i = 3; i <= n; i++) {
                  dp[i] = dp[i - 1]  % 1000000007 + dp[i - 2] % 1000000007;
              }
          }
          return dp[n] % 1000000007;
      }
  }
  ```

  

#### [面试题11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

题面：把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  

+ 二分查找：时间复杂度为O(logN)，但是最坏情况也会到达O(N)。

  ```java
  class Solution {
      public int minArray(int[] nums) {
          int low = 0;
          int high = nums.length - 1;
  
          //经过旋转的数组，第一个元素会比最后一个元素要大
          //同时也有可能，没有被旋转过
          while (low < high) {
              int mid = low + (high - low) / 2;
              if (nums[mid] < nums[high]) //中间值属于后部分递增序列
                  high = mid;
              else if (nums[mid] > nums[high]) //中间值属于前部分递增序列
                  low = mid + 1;
              else //这个情况没有办法判断
                  high -= 1;
          }
          return nums[low];
      }
  }
  ```



#### [剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

题面：请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

[["a","b","c","e"],
["s","f","c","s"],
["a","d","e","e"]]

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

+ 深度优先搜索

  ```java
  class Solution {
      public boolean exist(char[][] board, String word) {
          int rows = board.length;
          int columns = board[0].length;
  
          //标记这个二维数组的格子是否被访问
          int[][] boardFlag = new int[rows][columns];
  
          //编译一遍这个二维数组
          for(int i = 0; i < board.length; i++) {
              for (int j = 0; j < board[0].length; j++) {
                  if (board[i][j] == word.charAt(0) && dfs(board, word, i, j, 0, boardFlag)) {
                      return true;
                  }        
              }
          }
  
          return false;
      }
  
      private boolean dfs(char[][] board, String word, int x, int y, int index, int[][] boardFlag) {
          //先判断下范围,以及当前位置是否已经被访问过了
          if (index >= word.length()) {
              return true;
          } else if(x < 0 || y < 0 || x >= board.length || y >= board[0].length || boardFlag[x][y] == 1) {
              return false;
          } 
  
          if (boardFlag[x][y] != 1 && board[x][y] == word.charAt(index)) {
              boardFlag[x][y] = 1;
              //当前相同，继续上下左右递归
              int[] xs = {x - 1, x + 1, x, x};
              int[] ys = {y, y, y - 1, y + 1};
              for(int i = 0; i < 4; i++) {
                  if(dfs(board, word, xs[i], ys[i], index + 1, boardFlag)) {
                      return true;
                  } 
              }
              boardFlag[x][y] = 0;
          } 
  
          return false;
      }
  }
  ```

  



#### [面试题13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

题面：地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

+ 深度优先搜索

  ```java
  
  ```

  




#### [面试题14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

题面：给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m] 。请问 k[0]*k[1]*...*k[m] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

+ 动态规划：长度为n的绳子在剪第一道的时候有n-1中选择，剪出的长度可能是1、2、3…、n-1，那么可得f(n)= max(f(i) + f(n-i))，其中0<i<n。

  ```java
  class Solution {
      public int cuttingRope(int n) {
          if (n < 2) {
              return 0;
          } else if (n == 2) {
              return 1;
          } else if (n == 3) {
              return 2;
          }
  
          int[] nLen = new int[n + 1];
          nLen[0] = 0;
          nLen[1] = 1;
          nLen[2] = 2;
          nLen[3] = 3;
          //当绳子超过3以后，上面的那些长度本身就是最大值，可以不用在剪了。
          for (int i = 4; i <= n; i++) {
              int max = 0;
              for (int j = 1; j <= i / 2; j++) {
                  int mul = nLen[j] * nLen[i - j];
                  max = Math.max(max, mul);
              }
              nLen[i] = max;
          }
  
          return nLen[n];
      }
  }
  ```

+ 贪心



#### [面试题14- II. 剪绳子 II](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/)

题面：给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m] 。请问 k[0]*k[1]*...*k[m] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

+ 这题似乎无法使用动态规划。



#### [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

题面：请实现一个函数，输入一个整数，输出该数二进制表示中 1 的个数。例如，把 9 表示成二进制是 1001，有 2 位是 1。因此，如果输入 9，则该函数输出 2。

+ 假设有一个二进制数11110101101，可以想到统计1的解法有，每次消除一个1，知道这个数字变为0结束，统计消除了多少个1，想要消除一个二进制位的1，可以将**x&(x-1)**

  ```java
  public class Solution {
      // you need to treat n as an unsigned value
      public int hammingWeight(int n) {
          int count = 0;
          while(n != 0) {
              n &= n - 1;
              count++;
          }
          return count;
      }
  }
  ```



#### [剑指 Offer 16. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

题面：实现函数double Power(double base, int exponent)，求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。

+ 快速幂+递归

  ```java
  class Solution {
      public double myPow(double x, int n) {
          //注意指数为负数的情况
          return n >= 0 ? quickMul(x, n) : 1.0 / quickMul(x, -n);
      }
  
      private double quickMul(double base, int exponent) {
          if (exponent == 0) {
              return 1.0;
          }
          double num = quickMul(base, exponent / 2);
          return exponent % 2 == 0 ? num * num : num * num * base;
      }
  }
  ```

  

#### [剑指 Offer 17. 打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

题面：输入数字 `n`，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。

+ end=10^n−1

  ```java
  class Solution {
      //int[]返回int型列表默认不考虑大数情况，但是如果不是呢？
      public int[] printNumbers(int n) {
          int end = (int)Math.pow(10, n) - 1;
          int[] res = new int[end];
          for (int i = 0; i < end; i++) {
              res[i] = i + 1;
          }
          return res;
      }
  }
  ```

+ 字符串解决大数问题，使用一个标志位来记录是否可以继续进位，不能继续进位的情况就是进位会导致这个数字超出n的位数长度。

  ```java
  class Solution {
      public int[] printNumbers(int n) {
          List<String> list = new ArrayList<>();
          StringBuilder str = new StringBuilder();
          // 将str初始化为n个'0'字符组成的字符串
          for (int i = 0; i < n; i++) {
              str.append('0');
          }
          while(!increment(str)){
              // 去掉左侧的0
              int index = 0;
              while (index < str.length() && str.charAt(index) == '0'){
                  index++;
              }
              list.add(str.toString().substring(index));
          }
  
          int[] res = new int[list.size()];
          for (int i = 0; i < res.length; i++) {
              res[i] = Integer.parseInt(list.get(i));
          }
          return res;
      }
  
      public boolean increment(StringBuilder str) {
          boolean isOverflow = false;
          for (int i = str.length() - 1; i >= 0; i--) {
              //当前这位加一
              char s = (char)(str.charAt(i) + 1);
              // 如果s大于'9'则发生进位
              if (s > '9') {
                  str.replace(i, i + 1, "0");
                  if (i == 0) {
                      isOverflow = true;
                  }
              } else {
                  // 没发生进位则跳出for循环
                  str.replace(i, i + 1, String.valueOf(s));
                  break;
              }
          }
          return isOverflow;
      }
  
      public static void main(String[] args) {
          Solution solution = new Solution();
          solution.printNumbers(3);
      }
  }
  ```

+ 递归

  ```java
  class Solution {
      public int[] printNumbers(int n) {
          //从首位开始固定数字，后面每一位使用递归处理
          List<Integer> res = new ArrayList<>();
          //这里用ArrayList可以让后面查找效率更高，否则会超时
          int[] temp = new int[n];
          dfs(temp, 0, res);
          int[] nums = new int[res.size() - 1];//需要将第一个数字0去除
          for (int i = 1; i < res.size(); i++) {
              nums[i - 1] = res.get(i);
          }
          return nums;
      }
  
      void dfs(int[] temp, int index, List<Integer> res){
          if (index == temp.length) {
              res.add(arrayToNum(temp));
              return;
          }
  
          for (int i = 0; i <= 9; i++) {
              temp[index] = i;
              dfs(temp, index + 1, res);
          }
      }
  
      int arrayToNum(int[] temp) {
          int res = 0;
          for (int i = 0; i < temp.length; i++) {
              res *= 10;
              res += temp[i];
          }
          return res;
      }
  }
  ```

  

#### [剑指 Offer 18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

题面：给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。返回删除后的链表的头节点。

+ 迭代：按照常规思路编写代码

  ```java
  class Solution {
      public ListNode deleteNode(ListNode head, int val) {
          if (head == null) {
              return head;
          }
  
          if (head.val == val) {
              return head.next;
          }
  
          ListNode cur = head;
          while(cur.next != null && cur.next.val != val) {
              cur = cur.next;
          }
  
          cur.next = cur.next.next;
          return head;
      }
  }
  
  //使用到一个哑结点
  class Solution {
      public ListNode deleteNode(ListNode head, int val) {
          //作为虚拟头结点
          ListNode node = new ListNode(0);
          node.next = head;
  
          ListNode cur = head;
          ListNode pre = node;
  
          while(cur != null) {
              if (cur.val == val) {
                  pre.next = cur.next;
                  break;
              }
              pre = cur;
              cur = cur.next;
          }
  
          return node.next;
      }
  }
  ```

+ 递归！

  ```java
  class SolutionE {
      public ListNode deleteNode(ListNode head, int val) {
          if (head == null) {
              return head;
          }
  
          if (head.val == val) {
              return head.next;
          }
  
          head.next = deleteNode(head.next, val);
          return head;
      }
  }
  
  ```

  

#### [19. 正则表达式匹配](https://leetcode-cn.com/problems/zheng-ze-biao-da-shi-pi-pei-lcof/)

题面：请实现一个函数用来匹配包含'. '和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（含0次）。在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但与"aa.a"和"ab*a"均不匹配。

+ **稍后在想想吧**



#### [剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

题面：请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100"、"5e2"、"-123"、"3.1416"、"-1E-16"、"0123"都表示数值，但"12e"、"1a3.14"、"1.2.3"、"+-5"及"12e+5.4"都不是。



#### [面试题21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

题面：输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

+ 双指针

  ```java
  class Solution {
      public int[] exchange(int[] nums) {
          if (nums == null || nums.length == 0) {
              return new int[0];
          }
          int start = 0;
          int end = nums.length - 1;
          while(start < end) {
              //是奇数
              while(start < nums.length && nums[start] %2 != 0) {
                  start++;
              }
              //是偶数
              while(end >= 0 && nums[end] %2 == 0) {
                  end--;
              }
              
              if (start < end) {
                  int temp = nums[start];
                  nums[start] = nums[end];
                  nums[end] = temp;
              }
          }
  
          return nums;
      }
  }
  ```



#### [面试题22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

题面：输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。

+ 双指针

  ```java
  class Solution {
      public ListNode getKthFromEnd(ListNode head, int k) {
          if (head == null || k < 1) {
              return null;
          }
  
          ListNode node1 = head;
          ListNode node2 = head;
          int count = k - 1;
          while (node1.next != null && count > 0) {
              node1 = node1.next;
              count--;
          }
  
          if (count > 0) {
              return null;
          } else {
              while(node1.next != null) {
                  node1 = node1.next;
                  node2 = node2.next;
              }
          }
  
          return node2;
      }
  }
  ```

  

#### [面试题24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

题面：定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

+ 遍历and细心

  ```java
  class Solution {
      public ListNode reverseList(ListNode head) {
          ListNode node = head;
          ListNode preNode = null;
          while(node != null) {
              ListNode tempNode = node;//存储当前结点
              node = node.next;//指针往后走
              tempNode.next = preNode;//当前结点的后继改变方向
              preNode = tempNode;
          }
          return preNode;
      }
  }
  ```

  

#### [面试题25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

题面：输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

+ 递归：每次在两个头结点中选择值更小的那个。

  ```java
  class Solution {
      //递归，合并，每次取值更小的头结点
      public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
          if (l1 == null && l2 == null) {
              return null;
          } else if (l1 == null) {
              return l2;
          } else if (l2 == null) {
              return l1;
          }
  
          ListNode result = null;
          if(l1.val <= l2.val){
              result = l1;
              result.next = mergeTwoLists(l1.next, l2);
          } else {
              result = l2;
              result.next = mergeTwoLists(l1, l2.next);
          }
          return result;
      }
  }
  ```

+ 迭代：链表的处理更容易出错。



#### [面试题26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

题面：输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)。B是A的子结构， 即 A中有出现和B相同的结构和节点值。

+ 遍历树，每一个节点都做比较

  ```java
  class Solution {
      public boolean isSubStructure(TreeNode A, TreeNode B) {
          //B是null，是不符合要求的
          if (B == null) {
              return false;
          }
          //如果B不为null，A却为null，直接返回false
          if (A == null) {
              return false;
          }
  
          boolean isSub = isSub(A, B);//判断当前节点以及他们对应的子节点
          //否则遍历A的结点，一边对比
          return isSub || isSubStructure(A.left, B) || isSubStructure(A.right, B);
      }
  
      boolean isSub (TreeNode A, TreeNode B) {
          //B已经遍历到叶子节点以后了，可以确定A中含有B的结构
          if (B == null) {
              return true;
          }
  
          if (A == null && B != null) {
              return false;
          }
  
          if (A.val != B.val) {
              return false;
          }
  
          //继续比较对应的子节点们
          return isSub(A.left, B.left) && isSub(A.right, B.right);
      }
  }
  ```







#### [面试题27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

题面：请完成一个函数，输入一个二叉树，该函数输出它的镜像。

+ 每次交换左右子树，递归处理

  ```java
  class Solution {
      public TreeNode mirrorTree(TreeNode root) {
          if (root == null) {
              return null;
          }
  
          TreeNode temp = root.left;
          root.left = root.right;
          root.right = temp;
          
          root.left = mirrorTree(root.left);
          root.right = mirrorTree(root.right);
  
          return root;
      }
  }
  ```

  

#### [剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

题面：请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。

+ 递归：前序遍历的同时进行一个与前序遍历互为镜像的自定义遍历方式

  ```java
  class Solution {
      public boolean isSymmetric(TreeNode root) {
          return isSame(root, root);
      }
  
      private boolean isSame(TreeNode left, TreeNode right) {
          if (left == null && right == null) {
              return true;
          }
  
          if (left == null || right == null) {
              return false;
          }
  
          if (left.val != right.val) {
              return false;
          }
  
          return isSame(left.left, right.right) && isSame(left.right, right.left);
      }
  }
  
  ```

+ 迭代



#### [面试题29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

题面：输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

+ 模拟and细心：只能讲关键在于临界条件判断。

  ```java
  class Solution {
      public int[] spiralOrder(int[][] matrix) {
          if(matrix == null || matrix.length == 0) {
              return new int[0];
          }
  
          int verLen = matrix.length;
          int  horLen = matrix[0].length;
          int[] result = new int[verLen * horLen];
          int index = 0;
  
          boolean isToRight = false;//只有经历过从左到右，才可能从上到下
  
          //旋转一圈为一个单位
          for (int i = 0; i < verLen - 1 / 2; i++) {
  
              if (horLen <= 0|| verLen<=0){
                  break;
              }
  
              isToRight = false;
              //从左到右
              for (int j = i; j < horLen; j++){
                  result[index++] = matrix[i][j];
                  isToRight = true;
              }
              //从上到下
              if (isToRight) {
                  for (int j = i + 1; j < verLen - 1; j++){
                      result[index++] = matrix[j][horLen - 1];
                  }
              }
              //从右到左,需要判断有没有这个必要，也就是最后边的内容，中间是否剩下两排
              if (verLen - 1 > i) {
                  for (int j = horLen - 1; j >= i; j--){
                      result[index++] = matrix[verLen - 1][j];
                  }
              }
  
              //从下到上,需要判断有没有这个必要，也就是最后边的内容，中间是否剩下两列
              if (horLen - 1 > i) {
                  for (int j = verLen - 2; j > i; j--){
                      result[index++] = matrix[j][i];
                  }
              }
  
              verLen--;
              horLen--;
          }
          return result;
      }
  }
  ```




#### [面试题30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

题面：定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

+ 使用辅助栈，将主栈中的最小值保持在辅助栈的栈顶。

  ```java
  class MinStack {
  
      Stack<Integer> stack;
      Stack<Integer> minStack;
  
      /** initialize your data structure here. */
      public MinStack() {
          stack = new Stack<>();
          minStack = new Stack<>();
      }
      
      public void push(int x) {
          stack.push(x);
          if (minStack.isEmpty()) {
              minStack.push(x);
          } else {
              minStack.push(Math.min(x, minStack.peek()));
              //保持stack栈中最小的数在minStack栈顶
          }
      }
      
      public void pop() {
          stack.pop();
          minStack.pop();
      }
      
      public int top() {
          return stack.peek();
      }
      
      public int min() {
          return minStack.peek();
      }
  }
  
  ```

  

#### [面试题31. 栈的压入、弹出序列](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

题面：输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列 {1,2,3,4,5} 是某栈的压栈序列，序列 {4,5,3,2,1} 是该压栈序列对应的一个弹出序列，但 {4,3,5,1,2} 就不可能是该压栈序列的弹出序列。

+ 聚焦与预想的出栈序列，遍历当中元素，如果当前元素在栈顶，就是符合条件的，出栈即可；而如果不在栈顶，有两种可能：在栈中，或者还没有入栈。因此继续检查后面的元素中有没有预想当前需要出栈的元素，非目标元素入栈，只要出现目标元素，就是符合要求的，如果一直没有出现目标元素，那么预想的出栈顺序是不可能实现了。

  ```java
  class Solution {
      public boolean validateStackSequences(int[] pushed, int[] popped) {
  
          if (pushed == null || popped == null || popped.length != pushed.length) {
              return false;
          }
  
          //让压栈序列进栈
          Stack<Integer> stack = new Stack<>();
  
          Stack<Integer> stackpushed = new  Stack<>();
          for (int i = pushed.length - 1; i >= 0; i--) {
              stackpushed.push(pushed[i]);
          }//使得stackpushed的出栈顺序就是压栈序列的压栈顺序
  
          //遍历这个出栈序列，依次看当前元素在不在stack的栈顶
          //如果不在，就想办法让它在栈顶，看后面没进栈的元素里面有没有它，
          //如果没有它，那就是在栈中了，是不可能按照预想的顺序出栈了over
          for(int i = 0; i < popped.length; i++){
              //如果栈顶已经存在当前这个应该出栈的元素，就直接出栈
              if (!stack.isEmpty() && stack.peek() == popped[i]) {
                  stack.pop();
              } else {//否则，将剩下的入栈序列当中的元素依次入栈，查看后面是否有应该出栈的这个元素
                  while(!stackpushed.isEmpty() && stackpushed.peek() != popped[i]) {
                      stack.push(stackpushed.pop());
                  }
                  if(stackpushed.isEmpty()) {
                      return false;
                  } else {
                      stackpushed.pop();
                  }
              }
          }
          return true;
      }
  
  }
  ```



#### [剑指 Offer 32 - I. 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

题面：从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

+ 使用队列进行层序遍历

  ```java
  class Solution {
      public int[] levelOrder(TreeNode root) {
          if (root==null){
              return new int[0];
          }
  
          ArrayList<Integer> list = new ArrayList<>();
          LinkedList<TreeNode> queue = new LinkedList<>();
          queue.add(root);
          while(!queue.isEmpty()) {
              TreeNode treeNode = queue.removeFirst();
              list.add(treeNode.val);
              if (treeNode.left != null) {
                  queue.addLast(treeNode.left);
              }
              if (treeNode.right != null) {
                  queue.addLast(treeNode.right);
              }
          }
  
          int[] res = new int[list.size()];
          for(int i = 0; i < res.length; i++) {
              res[i] = list.get(i);
          }
          return res;
      }
  }
  ```

+ 



#### [面试题34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

题面：输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。

+ 深度优先搜索

  ```java
  class Solution {
      public List<List<Integer>> pathSum(TreeNode root, int sum) {
          List<List<Integer>> result = new ArrayList<>();
          //使用sonListTemp暂存某个路径上的值
          List<Integer> sonListTemp = new ArrayList<>();
          pathSum(root, sum, result, sonListTemp);
          return result;
      }
  
      //先序遍历，同时处理
      public void pathSum(TreeNode root, int sum, List<List<Integer>> result, List<Integer> sonListTemp) {
  
          if (root == null) { 
              return;
          }
  
          sonListTemp.add(root.val);
          int sumTarget = 0;
          //遇到叶子节点的时候再一并处理sonListTemp中的所有路径上的值
          if (root.left == null && root.right == null) {
              for (int i = 0 ; i < sonListTemp.size(); i++) {
                  sumTarget += sonListTemp.get(i);
              }
              if (sumTarget == sum) {
                  result.add(new ArrayList<>(sonListTemp));//创建一个新的List
              }
          }
          pathSum(root.left, sum, result, sonListTemp);
          pathSum(root.right, sum, result, sonListTemp);
          sonListTemp.remove(sonListTemp.size() - 1);
      }
  }
  ```




#### [面试题35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

题面：请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

+ 最直观的方式就是利用next指针复制一条新的链表，而原来random 指针指向的结点每次遍历链表去查找。时间复杂度是O(n^2)。

+ 使用Map将原节点与复制结点映射。空间换时间，时间复杂度降为O(n)。

+ 将每个复制的结点连接在原节点后面，这样random指针指向的结点就是，原节点指向结点的后一个结点，然后拆分成两个链表。不用辅助空间实现时间复杂度O(n)。

  ```java
  class Solution {
      public Node copyRandomList(Node head) {
          if(head == null){
              return head;
          }
  
          // 将克隆结点放在原结点后面
          Node node = head;
          // 1->2->3  ==>  1->1'->2->2'->3->3'
          while(node != null){
              Node clone = new Node(node.val);//克隆结点
              clone.next = node.next;
              node.next = clone;
              node = clone.next;
          }
  
          // 处理random指针
          node = head;
          while(node != null){
              node.next.random = node.random == null ? null : node.random.next;
              node = node.next.next;
          }
  
          // 还原原始链表，即分离原链表和克隆链表
          node = head;
          Node  cloneHead = node.next;
          while(node != null){
              Node temp = node.next.next;//node结点的实际下一个结点
              node.next.next = temp == null ? null : temp.next;
              node.next = temp;
              node = node.next;
          }
          return cloneHead;
      }
  }
  ```

  




#### [面试题36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

题面：输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

+ 递归

  ```java
  class Solution {
  
      //可以记录下未来的头结点，和尾结点
      Node head;
      Node pre;
  
      //递归
      public Node treeToDoublyList(Node root) {
          inOrder(root);
          if(head != null){
              head.left = pre;
              pre.right = head;
          }
          return head;
      }
  
      //中序递归
      public void inOrder(Node root) {
          if (root == null) return;
          inOrder(root.left);
          if(pre != null){
              pre.right = root;
              root.left = pre;
          } else {
              head = root;
          }
          pre = root;
          inOrder(root.right);
      }
  
  }
  ```

+ 迭代

  ```java
  class Solution {
      Node head;
      Node pre;
      public Node treeToDoublyList(Node root) {
          //迭代，使用栈来辅助遍历
          Stack<Node> stack = new Stack<>();
          Node cur = root;//指向当前遍历到的结点
          while(!stack.isEmpty() || cur != null) {
              if (cur != null){
                  stack.push(cur);
                  cur = cur.left;
              } else if(!stack.isEmpty()) {
                  Node temp = stack.pop();
                  cur = temp.right;
                  if (pre != null){
                      pre.right = temp;
                      temp.left = pre;
                  } else {
                      head = temp;
                  }
                  pre = temp;
              }
          }
          //当前结点不为空的时候，一直往左子节点的方向走，依次入栈，一直到没有左子节点
          //当栈中结点出栈的时候，就是真正进行中序遍历的时候，出栈的时候去处理它的右子树
          //右子树上也有它自己的左子树，左子节点
          if (head != null){
              head.left = pre;
              pre.right = head;
          }
          return head;
      }
  }
  ```




#### [面试题37. 序列化二叉树](https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/)

题面：请实现两个函数，分别用来序列化和反序列化二叉树。

+ 深度优先搜索：反序列化阶段使用了List。

  ```java
  public class Codec {
  
      // Encodes a tree to a single string.
      public String serialize(TreeNode root) {
          return serializeB(root, "");
      }
  
      //序列化
      public String serializeB(TreeNode root, String str) {
          if (root == null) {
              str += "null,";
          } else {
              str += String.valueOf(root.val) + ",";
              str = serializeB(root.left, str);
              str = serializeB(root.right, str);
          }
          return str;
      }
  
      // Decodes your encoded data to tree.
      public TreeNode deserialize(String data) {
          String[] datas = data.split(",");
          //List<String> list = Arrays.asList(datas);
          //这里记录一个错误，如果按照上面的写法生成List,返回的实际上是Arrays的内部类ArrayList
          //虽然这个内部类和java.util中的ArrayList都是继承自AbstractList，
          //但是它没有重写AbstractList的add和remove操作，
          //在AbstractList进行add或者remove会直接返回UnsupportedOperationException异常。
          List<String> list = new LinkedList<String>(Arrays.asList(datas));
          return deserializeB(list);
      }
  
      //反序列化
      public TreeNode deserializeB(List<String> data) {
          if (data.get(0).equals("null")) {
              data.remove(0);
              return null;
          }
  
          TreeNode root = new TreeNode(Integer.valueOf(data.get(0)));
          data.remove(0);
          root.left = deserializeB(data);
          root.right = deserializeB(data);
          return root;
      }
  }
  ```

  

#### [面试题38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

题面：输入一个字符串，打印出该字符串中字符的所有排列。你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。

+ 深度优先搜索

  ```java
  class Solution {
      public String[] permutation(String s) {
          TreeSet<String> set = new TreeSet<>();
          method(s, set, 0);
          String[] result = new String[set.size()];
          Iterator iterator = set.iterator();
          int index = 0;
          while(iterator.hasNext()){
              result[index++] = (String)iterator.next();
          }
          return result;
      }
  
      static public void method(String s, TreeSet<String> set, int index) {
          //遍历从index位置开始的后面的每个字符，index前面的内容都已经确定了，
          //每次将当前字符的位置和index位置的字符置换
          for(int i = index; i < s.length(); i++) {
              char[] sChar = s.toCharArray();
              swap(sChar, index, i);
              String sonString = new String(sChar);
              set.add(sonString);
              method(sonString, set, index + 1);
          }
      }
  
      static private void swap(char[] arr, int x, int y) {
          char temp = arr[x];
          arr[x] = arr[y];
          arr[y] = temp;
      }
  }
  ```




#### [面试题39. 数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

题面：数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。你可以假设数组是非空的，并且给定的数组总是存在多数元素。

+ 排序

+ 出现次数超过一半，也就是出现次数比其他数字出现次数的总和还要大。

  ```java
  class Solution {
      public int majorityElement(int[] nums) {
          int probTarget = nums[0];
          int count = 0;
          for (int i = 0; i < nums.length; i++) {
              if (nums[i] == probTarget) {
                  count++;
              } else if (count == 0){
                  probTarget = nums[i];
                  count++;
              } else {
                  count--;
              }
          }
          return probTarget;
      }
  }
  ```

  

#### [40.最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

题面： 输入整数数组 `arr` ，找出其中最小的 `k` 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。 

+ 先排序：至少O(nlogn)。
+ 使用堆：可以直接使用的堆结构有优先队列PriorityQueue，O(nlogk)。
+ Partition：可以用BFPRT算法，时间复杂度基本为O(n)。



#### [42. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)

题面：输入一个整型数组，数组里有正数也有负数。数组中的一个或连续多个整数组成一个子数组。求所有子数组的和的最大值。要求时间复杂度为O(n)。

+ 动态规划的思想：遍历数组一遍，如果前面的sum是负数，当前数值nums[i]相加上前面的sum，和只会变小，无论nums[i]正负，都可以直接给sum赋值；如果前面sum是非负数的话，就可以直接相加，之后比较maxProb得出可能的最大和，过程中注意判断上下溢出。时间复杂度O(n)。



#### [面试题43. 1～n整数中1出现的次数](https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/)

题面：输入一个整数 n ，求1～n这n个整数的十进制表示中1出现的次数。例如，输入12，1～12这些整数中包含1 的数字有1、10、11和12，1一共出现了5次。

+ ？？？



#### [面试题44. 数字序列中某一位的数字](https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/)

题面：数字以0123456789101112131415…的格式序列化到一个字符序列中。在这个序列中，第5位（从下标0开始计数）是5，第13位是1，第19位是4，等等。请写一个函数，求任意第n位对应的数字。

+ 将序列中设计的数字，按照位数不同分组查找，通过不同位数的一批数字所在的位数边界来确定查找范围。注意溢出的情况。

  ```java
  class Solution {
      public int findNthDigit(int n) {
          if (n < 0) {
              return -1;   
          }
  
          int digits = 1;
          while(true) {
              long numbers = countOfIntegers(digits);//得到digits位的数字总共有多少个
              if (n < numbers * digits) return (int)digitAtn(n, digits);
              n -= digits * numbers;
              digits++;
          }
  
      }
  
      //得到digits位的数字总共有多少个
      long countOfIntegers(int digits) {
          if (digits == 1) {
              return 10;
          }
          long count = (long)Math.pow(10, digits - 1);
          return 9 * count;
      }
  
      //获取m位数的第一个数字
      long beginNumber(int digits) {
          if (digits == 1) {
              return 0;
          }
          return (long)Math.pow(10, digits - 1);
      }
  
      //获取数字序列中第n位数字
      long digitAtn(int n, int digits) {
          long number = beginNumber(digits) + n / digits;
          int nFromRight = digits - n % digits;
          for (int i = 1; i < nFromRight; i++) {
              number /= 10;
          }
          //上面这个部分有点绕。。。
          return number % 10;
      }
  }
  ```

  

#### [面试题45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

题面：输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

+ 定义一个排序规则，数组根据这个规则排序之后能排成一个最小的是数字。通过比较器来实现。

  ```java
  class Solution {
      public String minNumber(int[] nums) {
          List<String> list = new ArrayList<>();
          for (int i = 0; i < nums.length; i++) {
              list.add(String.valueOf(nums[i]));
          }
  
          Collections.sort(list, (x, y) -> {
              String a = x + y;
              String b = y + x;
              return a.compareTo(b);//怎么排序会比较小
          });
  
          StringBuilder result = new StringBuilder();
          for (int i = 0; i < list.size(); i++) {
              result.append(list.get(i));
          }
          
          return result.toString();
      }
  }
  ```



#### [47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)

题面：在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

+ 动态规划：f(x,y) =max {f(x-1,y) , f(x,y-1)} + p(x,y)。创建一个temp二维数组暂存f(x,y)的值，使用两层for循环来处理，时间复杂度是O(n^2)，但是相比用递归的方式效率已经高了许多。



#### [48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

题面： 请从字符串中找出一个最长的不包含重复字符的子字符串，计算该最长子字符串的长度。 

+ 蛮力法：时间复杂度达到O(n^3)。
+ 双向队列，滑动窗口：如果队列当中已经有了这个字符，就将队列从头删除一直到重复字符已经被删掉了，相关长度的记录处理好，然后把字符添加进来，每次计较记录的长度和记录的可能长度比较保留较大的。时间复杂度应该是O(n)到O(n^2)吧。
+ 动态规划：f(i) = f(i-1) + 1（如果这个字符在f(i-1)的最长子数组中没有出现过）；如果在f(i-1)的最长子数组中出现过。



#### [面试题49. 丑数](https://leetcode-cn.com/problems/chou-shu-lcof/)

题面：我们把只包含因子 2、3 和 5 的数称作丑数（Ugly Number）。求按从小到大的顺序的第 n 个丑数。

+ 最直观的方式，但是超时。

  ```java
  class Solution {
      public int nthUglyNumber(int n) {
          if (n < 1) {
              return 0;
          }
  
          int num = 0;
          int index = 0;
          while (index < n) {
              num++;
              if (isUgly(num)) {
                  index++; 
              }
          }
          return num;
      }
  
      public boolean isUgly(int num) {
          while (num % 2 == 0) {
              num /= 2;
          }
          while (num % 3 == 0) {
              num /= 3;
          }
          while (num % 5 == 0) {
              num /= 5;
          }
          return num == 1 ? true : false;
      }  
  }
  ```

+ 动态规划：不需要在非丑数的整数上进行计算了，效率得到了提升。

  ```java
  class Solution {
      public int nthUglyNumber(int n) {
          if (n < 1) {
              return 0;
          }
          //使用一个数组来存储从小到大的丑数。
          int[] result = new int[n + 1];
          result[1] = 1;
          int mul2 = 1;
          int mul3 = 1;
          int mul5 = 1;
  
          int index = 2;
          while (index <= n) {
              int minUglyNum = Math.min(Math.min(result[mul2] * 2, result[mul3] * 3), result[mul5] * 5);
              result[index] = minUglyNum;
              //保证三个指针的位置指向的元素再乘上相应的因子，
              //得到的值只刚比当前丑数大，而不至于大太多
              //如果得到的值比当前丑数还小，这个得到的值肯定是已经在丑数排序队列当中的
              while(result[mul2] * 2 <= result[index]) {
                  mul2++;
              }
              while(result[mul3] * 3 <= result[index]) {
                  mul3++;
              }
              while(result[mul5] * 5 <= result[index]) {
                  mul5++;
              }
              index++;
          }
          return result[n];
      } 
  }
  ```

  

#### [面试题50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

题面：在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。

+ 哈希表

  ```java
  class Solution {
      public char firstUniqChar(String s) {
          HashMap<Character, Integer> hashMap = new HashMap<>();
          for (int i = 0; i < s.length(); i++) {
              char c = s.charAt(i);
              if (hashMap.containsKey(c)) {
                  hashMap.put(c, hashMap.get(c) + 1);
              } else {
                  hashMap.put(c, 1);
              }
          }
  
          char result = ' ';
          for (int i = 0; i < s.length(); i++) {
              char c = s.charAt(i);
              if (hashMap.get(c) == 1) {
                  result = c;
                  break;
              }
          }
  
          return result;
      }
  }
  ```



#### [面试题54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

题面：给定一棵二叉搜索树，请找出其中第k大的节点。

+ 递归中序

  ```java
  class Solution {
  
      public int kthLargest(TreeNode root, int k) {
          PriorityQueue<TreeNode> queue = new PriorityQueue<>(k, (x, y) -> y.val - x.val);
          findK(root, queue);
          int count = k;
          if (queue.size() < k) return -1;
          while (count > 1){
              queue.poll();
              count--;
          }
          return queue.poll().val;
      }
  
      //递归，中序遍历
      static public void findK(TreeNode root, PriorityQueue<TreeNode> queue) {
          if(root == null) {
              return;
          }
          findK(root.left, queue);
          queue.offer(root);
          findK(root.right, queue);
      }
  }
  ```

+ 迭代中序

  ```java
  class Solution {
      public int kthLargest(TreeNode root, int k) {
          //中序遍历的时候讲结点放入优先队列当中
          PriorityQueue<TreeNode> queue = new PriorityQueue<>((x, y) -> y.val - x.val);//大顶堆
          inOrderTree(root, queue);
          int count = k;
          if (queue.size() < k) return -1;
          while(!queue.isEmpty() && count > 1){
              queue.poll();
              count--;
          }
          int target = queue.poll().val;
          return target;
      }
  
      //迭代，中序遍历二叉树
      public void inOrderTree(TreeNode root, PriorityQueue<TreeNode> queue){
          Stack<TreeNode> stack = new Stack<>();
          TreeNode head = root;
          while(!stack.isEmpty() || head != null) {
              if (head != null) {
                  stack.push(head);
                  head = head.left;
              } else if (!stack.isEmpty()) {
                  TreeNode node = stack.pop();
                  queue.add(node);
                  head = node.right;
              }
          }
      }
  }
  ```

+ 中序相反的遍历方向：二叉排序树的性质来找第K小直接中序就可以，题目要求找到第k大的节点，可以定义一个与中序遍历正好相反的遍历方式。

  ```java
  class Solution {
      int count = 0;
      TreeNode target = null;
      public int kthLargest(TreeNode root, int k) {
          inOrderB(root, k);
          return target == null ? -1 : target.val;
      }
  
      //递归，使用一个与中序遍历完全相反的遍历方式
       //递归，中序遍历
       public void inOrderB(TreeNode root, int k) {
          if(root == null) {
              return;
          }
          inOrderB(root.right, k);
          count++;
          if(count == k){
              target = root;
              return;
          }
          inOrderB(root.left, k);
      }
  }
  ```




#### [面试题56 - I. 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

题面：一个整型数组 `nums` 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

+ 异或：利用异或运算结果同假异真的特点。

  ```java
  class Solution {
      public int[] singleNumbers(int[] nums) {
          int[] res = new int[2];
          //根据异或运算同假异真
          int flag = 0;
          //flag可以记录最终两个只出现一次的数字的异或结果，
          //因为相同的数字的异或结果一定是0，最终被抵消
          for (int i = 0; i < nums.length; i++) {
              flag ^= nums[i];
          }
          //那么flag中最低的不为0的那个数位，两个数字绝对不相同，
          //如果可以据此，把原来数组根据这个数位，分成两组，那么两个数字绝对在不同的组，
          //而其他出现两次的数组由于一样,所以会分在同一个组
          int toRight = findLowBit(flag);
          for (int i = 0; i < nums.length; i++) {
              //分组计算
              if (((nums[i] >> toRight) & 1) != 0) {
                  res[0] ^= nums[i];
              } else {
                  res[1] ^= nums[i];
              }
          }
          return res;
      }
  
      //找到最低的一位不为0的右移次数
      public int findLowBit(int num) {
          int toRight = 0;
          while ((num & 1) == 0) {
              num = num >> 1;
              toRight++;
          }
          return toRight;
      }
  }
  ```




#### [面试题56 - II. 数组中数字出现的次数 II](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

题面：在一个数组 `nums` 中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。

+ 排序

+ 哈希表

+ 位运算：如果一个数字出现三次，那么它的二进制表示的每一个也就都出现三次，意味着所有出现三次的数字的二进制表示的每一位都分别加起来，每一位的和都是能够被3整除的。而数组中所有数字的二进制表示每一位都加起来，如果某一位能够被3整除，那么那个只出现一次的数字的二进制表示中对应的那一位就是0，否则就是1。

  ```java
  class Solution {
      public int singleNumber(int[] nums) {
          if (nums == null || nums.length == 0) {
              return 0;
          }
  
          int[] bitSum = new int[32];
          for (int i = 0; i < nums.length; i++) {
              int bitMask = 1;
              for (int j = 31; j >= 0; j--) {
                  int bit = nums[i] & bitMask;
                  if (bit != 0) {
                      bitSum[j] += 1;
                  }
                  bitMask = bitMask << 1;
              }
          }
          int result = 0;
          for (int i = 0; i < 32; i++) {
              result = result << 1;
              result += bitSum[i] % 3;
          }
          return result;
      }
  }
  ```

  

#### [面试题57. 和为s的两个数字](https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/)

题面：输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可。

+ 首尾指针

  ```java
  class Solution {
      public int[] twoSum(int[] nums, int target) {
          if (nums == null || nums.length == 0) {
              return null;
          }
  
          int left = 0;
          int right = nums.length - 1;
          int sum = nums[left] + nums[right];
          int[] result = new int[2];
          while(sum != target && left <= right) {
              if (sum < target) {
                  left++;
              } else {
                  right--;
              }
              sum = nums[left] + nums[right];
          }
          result[0] = nums[left];
          result[1] = nums[right];
  
          return result;
      }
  }
  ```

  

#### [面试题57 - II. 和为s的连续正数序列](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)

题面：输入一个正整数 target ，输出所有和为 target 的**连续正整数序列**（至少含有两个数）。序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

+ 双指针：

  ```java
  class Solution {
      public int[][] findContinuousSequence(int target) {
          //这里返回一个二维数组就很难受了。。。
          int small = 1;
          int big = 2;
          List<int[]> result = new ArrayList<>();//这里的类型是int[]
          while(small <= target/2) {
              int sum = sumNow(small, big);
              if (sum == target){
                  int[] sonArray = new int[big - small + 1];
                  int index = 0;
                  for (int i = small; i <= big; i++){
                      sonArray[index++] = i;
                  }
                  result.add(sonArray);
                  small++;//即是这次有符合要求的结果，还是要继续找。
              } else if (sum < target){
                  big++;
              } else {
                  small++;
              }
          }
          int[][] resultArray = new int[result.size()][];
          result.toArray(resultArray);
          return resultArray;
      }
  
      //计算这段连续子序列相加得到的值
      static public int sumNow(int small, int big) {
          int count = small;
          int result = 0;
          while(count <= big){
              result += count;
              count++;
          }
          return result;
      }
  }
  ```




#### [面试题58 - II. 左旋转字符串](https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

题面：字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。

+ 



#### [59.滑动窗口的最大值](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)

题面： 给定一个数组 `nums` 和滑动窗口的大小 `k`，请找出所有滑动窗口里的最大值。 

+ 遍历：时间复杂度O(nk)。
+ 使用双端队列：可以用LinkedList，每次滑动一个数字进来如果比窗口内的值小可以先保留，之后的滑动是有可能成为窗口中的最大值的，但是如果比窗口内的值大，窗口内的较小值就可以被淘汰了，因为他们确定没有机会是窗口中的最大值，另外也要淘汰掉已经滑动出窗口外的过期值。由此看来，队列当中所保留的值一直是单调递减的。
+ 使用堆：可以直接使用的堆结构有优先队列PriorityQueue，每次添加删除的时间复杂度都是O(logk)，获取最大元素O(1)，总时间复杂度是O(nlogk)。



#### [59.队列的最大值](https://leetcode-cn.com/problems/dui-lie-de-zui-da-zhi-lcof/)

题面：请定义一个队列并实现函数 max_value 得到队列里的最大值，要求函数max_value、push_back 和 pop_front 的时间复杂度都是O(1)。

+ 使用双向队列来处理最大值的记录：由于只用一个变量记录最大值，当出队的过程由于剩余在队列中的数值可能会和出队的数重复，所以不好处理最大值记录的更新，因此可以使用一个双向队列存储单调递减值的方式来处理最大值的记录。



#### [60.n个骰子的点数](https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/)

题面：把n个骰子扔在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。

+ 递归：第1骰子和剩下（n-1）个骰子。（超时）

  ```java
  class Solution {
      int maxValue = 6;//表示骰子的面数
      public double[] twoSum(int n) {
          if (n < 1) {
              return null;
          }
          List<Double> list = new ArrayList<>();
          int maxNum = n * maxValue;//骰子们出现的最大总和
          //骰子出现的总和的范围是n - n * 6，每个下标对应一个值，数组记录每个值出现的次数
          int[] probs = new int[maxNum - n + 1];
  
          probility(n, probs);
  
          //总共会出现多少种值
          double total = Math.pow(maxValue, n);
          //一次计算，数组中对应的每一种值出现的概率
          for (int i = n; i <= maxNum; i++) {
              double ratio = probs[i - n] / total;
              if (ratio > 0) {
                  list.add(ratio);
              }
          }
  
          double[] res = new double[list.size()];
          for (int i = 0; i < list.size(); i++) {
              res[i] = list.get(i);
          }
          return res;
      }
  
      //递归处理
      private void probility(int n, int[] probs) {
          //当前骰子可能出现的各种点数
          for (int i = 1; i <= maxValue; i++) {
              probility(n, n, i, probs);
          }
      }
  
      private void probility(int n, int curn, int sum, int[] probs) {
          //当前只剩下一个骰子，直接返回
          if (curn == 1) {
              //当前出现值计数
              probs[sum - n]++;
          } else {
              for (int i = 1; i <= maxValue; i++) {
                  probility(n, n - 1, sum + i, probs);
              }
          }
      }
  
  }
  
  
  
  ```

+ 动态规划：？



#### [61.扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

题面：从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。

+ 排序：五张牌相当于数组中的五个元素，先排个序，不连续的部分可以用大小王来抵。那么不连续的条件就是大小王不够抵或者出现元素重复。

  ```java
  class Solution {
      public boolean isStraight(int[] nums) {
          
          if(nums == null || nums.length == 0) {
              return false;
          }
  
          //排序
          Arrays.sort(nums);
          int zeroCount = 0;
          int cha = 0;
  
          for (int i = 0; i < nums.length - 1; i++) {
              if (nums[i] == 0) {
                  zeroCount++;
              } else if (nums[i] == nums[i + 1]) {//出现重复且不为0的数字
                  return false;
              } else if (nums[i + 1] - nums[i] > 1) {
                  cha += (nums[i + 1] - nums[i] - 1);
              }
          }
  
          if (cha > zeroCount){
              return false;
          }
  
          return true;
      }
  }
  ```

  



#### [62.圆圈中最后剩下的数字](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)

题面：0,1,…,n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字。求出这个圆圈里剩下的最后一个数字。例如，0、1、2、3、4这5个数字组成一个圆圈，从数字0开始每次删除第3个数字，则删除的前4个数字依次是2、0、4、1，因此最后剩下的数字是3。

+ 模拟环形链表，但是时间复杂度方面可能无法在LC中AC。

  ```java
  class Solution {
      public int lastRemaining(int n, int m) {
  
          //模拟环形链表，注意下标的取值
          LinkedList<Integer> linkedList = new LinkedList<>();
          for (int i = 0; i < n; i++) {
              linkedList.add(i);
          }
          int index = 0;
          while (linkedList.size() > 1) {
              index += (m - 1);
              index %= linkedList.size();
              linkedList.remove(index);
          }
          return linkedList.getFirst();
      }
  }
  ```

+ 递归（数学方法）（需要再梳理下！）

  ```java
  class Solution {
      
      public int lastRemaining(int n, int m) {
          return method(n, m);
      }
  
      public int method(int n, int m) {
          if (n == 1) {
              return 0;
          }
          int x = method(n-1, m);
          return (m + x) % n;
      }
      
  }
  ```

+ 迭代（数学方法）



#### [63.股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)

题面： 假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？ 

+ 使用一个变量保存数组前i-1个数字中的最小值，扫描数组一次，依次和最小值作比较得出最大利润。

  ```java
  class Solution {
      public int maxProfit(int[] prices) {
          if (prices == null || prices.length == 0) {
              return 0;
          }
          int maxResult = 0;
          int minPrice = prices[0];
          for (int i = 0; i < prices.length; i++) {
              minPrice = Math.min(minPrice, prices[i]);//更新最小值
              if (prices[i] > minPrice) {
                  maxResult = Math.max(maxResult, prices[i] - minPrice);
              }
          }
          return maxResult;
      }
  }
  ```

  



#### [64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)

题面： 求 `1+2+...+n` ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。 

+ 利用&&的特点，前面的表达式为真才执行后面的表达式，就可以以此代替if判断，决定执行下一表达式。



#### [65. 不用加减乘除做加法](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

题面： 写一个函数，求两个整数之和，要求在函数体内不得使用 “+”、“-”、“*”、“/” 四则运算符号。 

+ 使用异或^处理相加，使用位与&然后<<1左移一位处理进位，不进位相加的结果加上进位的结果的和。

  ```java
  class Solution {
      public int add(int a, int b) {
          //使用异或^处理相加，使用位与&然后<<1左移一位处理进位，
          //不进位相加的结果加上进位的结果的和。
          int sum;
          int carray;
          do {
              sum = a ^ b;
              carray = (a & b) << 1;
              a = sum;
              b = carray;
          }while (carray != 0);
          return sum;
      }
  }
  ```

  



#### [66. 构建乘积数组](https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/)

题面：给定一个数组 A[0,1,…,n-1]，请构建一个数组 B[0,1,…,n-1]，其中 B 中的元素 B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]。不能使用除法。

+ 如果把数组B中的每个元素放入A中缺少的位置，那么形象的看就相当于数组B的元素就在这个矩阵的对角线上，由左半边的A数组的元素的乘积与右半边的A数组元素的乘积两部分结果的乘积形成，而左边部分不必要每次遍历一次计算结果，只要根据以前计算出的记过再乘上新加上的结果就行，自上而下计算，右边相反是自下而上的。

  

#### [67. 把字符串转换成整数](https://leetcode-cn.com/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

题面： 写一个函数 StrToInt，实现把字符串转换成整数这个功能。不能使用 atoi 或者其他类似的库函数。 

+ 细心处理：首先去掉前面空格，然后确定正负号，然后处理后面的数字（注意溢出的情况）。

  ```java
  class Solution {
      public int strToInt(String str) {
          if (str == null || str.length() == 0) {
              return 0;
          }
          int start = 0;
          int end = str.length() - 1;
          //如果遇到空格
          while(start <= end && str.charAt(start) == ' ') {
              start++;
          }
          if  (start > end) {
              return 0;
          }
  
          //正负号的判断
          boolean isNagative = false;
          if (str.charAt(start) == '+') {
              isNagative = false;
              start++;
          } else if (str.charAt(start) == '-') {
              isNagative = true;
              start++;
          } else if (str.charAt(start) > '9' || str.charAt(start) < '0') {
              //其他字符的情况
              return 0;
          }
  
          //处理数字
          int sum = 0;
          while(start <= end && str.charAt(start) <= '9' && str.charAt(start) >= '0') {
              char c = str.charAt(start);
              int number = c - '0';
              //判断是否会溢出
              if (sum > (Integer.MAX_VALUE - number) / 10) {
                  //如果溢出（目前还是绝对值）
                  return isNagative ? Integer.MIN_VALUE : Integer.MAX_VALUE;
              }
              //如果不会溢出
              sum = sum * 10 + number;
              start++;
          }
  
          //返回结果记得带上符号
          return isNagative ? -sum : sum;
      }
  }
  ```

  

#### [面试题68 - II. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

题面：给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

+ 记录到p、q的路径，找到最近公共祖先

  ```java
  class Solution {
      public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
          List<TreeNode> pathP = new ArrayList<>();
          List<TreeNode> pathQ = new ArrayList<>();
          List<List<TreeNode>> res = new ArrayList<>();
          dfs(root, p, pathP, res);
          dfs(root, q, pathQ, res);
  
          TreeNode resultNode = null;
  
          if (res.size() >= 2) {
              pathP = res.get(0);
              pathQ = res.get(1);
              if (pathP.size() < pathQ.size()) {//保证pathP对应的list更长
                  List<TreeNode> temp = pathP;
                  pathP = pathQ;
                  pathQ = temp;
              }
  
              for (int i = pathP.size() - 1; i >= 0; i--) {
                  TreeNode tempNode = pathP.get(i);
                  if (pathQ.contains(tempNode)) {
                      resultNode = tempNode;
                      break;
                  }
              }
          }
  
          return resultNode;
      }
  
      public void dfs(TreeNode root, TreeNode p, List<TreeNode> path, List<List<TreeNode>> res) {
          if (root == null) {
              return;
          }
  
          path.add(root);
          if (root.val == p.val) {
              res.add(new ArrayList<>(path));
          } else {
              dfs(root.left, p, path, res);
              dfs(root.right, p, path, res);
          }
          path.remove(path.size() - 1);
      }
  }
  ```

  