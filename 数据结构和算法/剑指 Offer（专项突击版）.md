# [剑指 Offer（专项突击版）](https://leetcode-cn.com/problem-list/e8X3pBZi/)

## 链表

### 哨兵节点

>  哨兵节点简化链表插入和删除操作，使链表无论如何也不会为空，也就不需要使用if语句来处理输入头结点为null等的情况了。

### 双指针

> 前后指针
>
> 快慢指针

#### [021. 删除链表的倒数第 n 个结点](https://leetcode.cn/problems/SLwz0R/?favorite=e8X3pBZi)

> 如果给定一个链表，请问如何删除链表中的倒数第k个节点？假设链表中节点的总数为n，那么1≤k≤n。要求只能遍历链表一次。

```java
public class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        //使用哨兵节点，避免单独处理头节点删除的情况。
        ListNode flag = new ListNode();
        flag.next = head;
        ListNode font = flag;
        ListNode back = flag; //指向被删除节点的前一个节点

        int count = n; 
        while(count > 0) {
            font = font.next;
            count--;
        }

        while(font.next != null) {
            font = font.next;
            back = back.next;
        }
        back.next = back.next.next;
        return flag.next;
    }
}
```

#### [022. 链表中环的入口节点](https://leetcode.cn/problems/c32eOV/?favorite=e8X3pBZi)

> 如果一个链表中包含环，那么应该如何找出环的入口节点？从链表的头节点开始顺着next指针方向进入环的第1个节点为环的入口节点。

需要知道环中节点数目的解法：

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode detectCycle(ListNode head) {
        //首先使用快慢指针确认下输入链表是否有环
        ListNode node = getNodeInCycle(head);
        if(node == null) {
            return null;
        } else {
            //计算环的长度
            int len = getLengthOfCycle(node);
            //使用前后指针找到环的入口，前指针先走len步
            ListNode font = head;
            ListNode back = head;
            for(int i = 0; i < len; i++) {
                font = font.next;
            }
            while(font != back) {
                font = font.next;
                back = back.next;
            }
            return font;
        }
    }

    private int getLengthOfCycle(ListNode node) {
        ListNode cur = node;
        int count = 1;
        while(cur.next != node) {
            cur = cur.next;
            count++;
        }
        return count;
    }

    private ListNode getNodeInCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while(fast != null) {
            fast = fast.next;
            if (fast != null) {
                fast = fast.next;
                slow = slow.next;
            }
            if (fast == slow) {
                return fast;
            }
        }
        return null;
    }
}
```

不需要知道环中节点数目的解法：

- 首先，可以使用快慢指针找环（可以理解为进入环中以后，走得快的fast指针其实在追赶slow）。
- 如果存在环，那么快慢指针会在环内相遇，但是相遇的点，不一定是环的起点
- 将快指针放回head，慢指针停留在前面相遇处，快慢指针再以相同速度移动，最终会在环的入口相遇。（可计算推导而出）

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        //首先使用快慢指针确认下输入链表是否有环
        ListNode node = getNodeInCycle(head);
        if(node == null) {
            return null;
        } else {
            ListNode font = node;
            ListNode back = head;
            while(font != back) {
                font = font.next;
                back = back.next;
            }
            return font;
        }
    }

    private ListNode getNodeInCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while(fast != null) {
            fast = fast.next;
            if (fast != null) {
                fast = fast.next;
                slow = slow.next;
            }
            if (fast == slow) {
                return fast;
            }
        }
        return null;
    }
}
```

#### [023. 两个链表的第一个重合节点](https://leetcode.cn/problems/3u1WK4/?favorite=e8X3pBZi)

> 输入两个单向链表，请问如何找出它们的第1个重合节点。

几种解法：

1. 在原来链表基础上构建出环形链表，利用环形链表找入口的方式，找到相交节点。
2. 利用栈，空间换时间，将两个链表的节点入栈，那么同时出栈时就是从相交部分的尾节点开始，出栈到相同的节点就是相交处。
3. 遍历两个链表分别得出长度，长链表先走若干步，之后一同前进走到相交处。

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        //计算两个链表的长度，然后长的链表先走n步，之后两条链表一起走，相交点为目标
        int lenA = getLen(headA);
        int lenB = getLen(headB);
        int delta = Math.abs(lenA - lenB); //Math.abs
        ListNode curLong = lenA > lenB ? headA : headB;
        ListNode curShort = lenA > lenB ? headB : headA; //注意lenA==lenB的情况，不要改成lenA < lenB
        while(delta > 0 && curLong != null) {
            curLong = curLong.next;
            delta--;
        }
        while(curLong != null && curShort != null && curLong != curShort) {
            curLong = curLong.next;
            curShort = curShort.next;
        }
        return curLong;
    }

    private int getLen(ListNode head) {
        int count = 0;
        while(head != null) {
            count++;
            head = head.next;
        }
        return count;
    }
}
```

+ 时间复杂度是O（m+n）。
+ 由于不需要保存链表的节点，因此这种方法的空间复杂度是O（1）。

### 反转链表

#### [024. 反转链表](https://leetcode.cn/problems/UHnkqh/?favorite=e8X3pBZi)

>  定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        while(cur != null) {
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }
}
```

+ 时间复杂度是O（n），空间复杂度是O（1）。

#### [025. 链表中的两数相加](https://leetcode.cn/problems/lMSNwu/?favorite=e8X3pBZi)

> 给定两个表示非负整数的单向链表，请问如何实现这两个整数的相加并且把它们的和仍然用单向链表表示？链表中的每个节点表示整数十进制的一位，并且头节点对应整数的最高位数而尾节点对应整数的个位数。

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        //反转链表
        //从头节点开始相加，注意进位
        ListNode head1 = reverseList(l1);
        ListNode head2 = reverseList(l2);
        ListNode result = addTwoList(head1, head2);
        return reverseList(result); //再反转一次
    }

    private ListNode reverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        while(cur != null) {
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }

    private ListNode addTwoList(ListNode l1, ListNode l2) {
        int carry = 0;
        int num = 0;
        ListNode head = new ListNode();
        ListNode cur = head;
        while(l1 != null || l2 != null) {
            int num1 = l1 == null ? 0 : l1.val;
            int num2 = l2 == null ? 0 : l2.val;
            int count = num1 + num2 + carry;
            num = count >= 10 ? count % 10 : count; //注意>=10才有进位
            carry = count >= 10 ? count / 10 : 0;
            ListNode node = new ListNode(num);

            cur.next = node;
            cur = cur.next;

            l1 = l1 != null ? l1.next : null;
            l2 = l2 != null ? l2.next : null;
        }

        //注意，如果还有一个进位不要遗漏
        if (carry > 0) {
            cur.next = new ListNode(carry);
        }
        
        return head.next;
    }
}
```

#### [026. 重排链表](https://leetcode.cn/problems/LGjMqU/?favorite=e8X3pBZi)

> 给定一个链表，链表中节点的顺序是L0→L1→L2→…→Ln-1→Ln，请问如何重排链表使节点的顺序变成L0→Ln→L1→Ln-1→L2→Ln-2→…？

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public void reorderList(ListNode head) {
        //将链表分割成两端，如果是基数节点数，需要保证前半段比后半段多一个节点
        //将后半段链表反转
        ListNode fontList = head;
        ListNode midNode = getSecondSonList(head);
        ListNode backList = reverseList(midNode);
        //两端链表节点逐个串联在一起
        mergeList(fontList, backList);
    }

    private ListNode mergeList(ListNode head1, ListNode head2) {
        ListNode pre = head1;
        while(head2 != null) {
            ListNode next1 = pre.next;
            ListNode next2 = head2.next;
            pre.next = head2;
            head2.next = next1;
            head2 = next2;
            pre = next1;
        }
        return head1;
    }

    private ListNode reverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        while(cur != null) {
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }

    private ListNode getSecondSonList(ListNode head) {
        //使用一个假节点，可以减少判断逻辑
        ListNode flag = new ListNode();
        flag.next = head;
        ListNode fast = flag;
        ListNode slow = flag;
        while(fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        ListNode temp = slow.next;
        slow.next = null;
        return temp;
    }
}
```

#### [027. 回文链表](https://leetcode.cn/problems/aMhZSa/?favorite=e8X3pBZi)

> 如何判断一个链表是不是回文？要求解法的时间复杂度是O（n），并且不得使用超过O（1）的辅助空间。如果一个链表是回文，那么链表的节点序列从前往后看和从后往前看是相同的。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public boolean isPalindrome(ListNode head) {
        //将链表切割成两段，如果是奇数个节点，中间的那个节点不用比较了
        ListNode head1 = head;
        ListNode head2 = getSecondList(head);
        //第二段链表反转
        head2 = reverseList(head2);
        //将两段链表逐个比较节点是否相同
        return isSame(head1, head2);
    }

    private boolean isSame(ListNode head1, ListNode head2) {
        while(head1 != null && head2 != null) {
            if (head1.val != head2.val) {
                return false;
            }
            head1 = head1.next;
            head2 = head2.next;
        }
        return true;
    }

    private ListNode reverseList(ListNode head) {
        ListNode pre = null;
        while(head != null) {
            ListNode temp = head.next;
            head.next = pre;
            pre = head;
            head = temp;
        }
        return pre;
    }

    private ListNode getSecondList(ListNode head) {
        //快慢指针分割链表
        ListNode dummy = new ListNode();
        dummy.next = head;
        ListNode fast = dummy;
        ListNode slow = dummy;
        while(fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        ListNode temp = slow.next;
        slow.next = null;
        return temp;
    }
}
```



### 双向链表和循环链表

#### [028. 展平多级双向链表](https://leetcode.cn/problems/Qv1Da2/?favorite=e8X3pBZi)

> 在一个多级双向链表中，节点除了有两个指针分别指向前后两个节点，还有一个指针指向它的子链表，并且子链表也是一个双向链表，它的节点也有指向子链表的指针。请将这样的多级双向链表展平成普通的双向链表，即所有节点都没有子链表。

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node prev;
    public Node next;
    public Node child;
};
*/

class Solution {
    public Node flatten(Node head) {
        //使用递归的处理方式
        flattenAndGetTail(head);
        return head;
    }

    //处理好扁平化，并且返回尾节点
    private Node flattenAndGetTail(Node head) {
        Node cur = head;
        Node tail = null;
        while(cur != null) {
            Node next = cur.next;
            if (cur.child != null) {
                Node child = cur.child;
                Node childTail = flattenAndGetTail(child);

                //子链表头部衔接
                cur.child = null;
                cur.next = child;
                child.prev = cur;

                //子链表尾部衔接
                childTail.next = next;
                if (next != null) {
                    next.prev = childTail;
                }

                //tail为子链表的尾节点
                tail = childTail;
            } else {
                tail = cur;
            }
            cur = next;
        }
        return tail;
    }
}
```

#### [029. 排序的循环链表](https://leetcode.cn/problems/4ueAj6/?favorite=e8X3pBZi)

> 在一个循环链表中节点的值递增排序，请设计一个算法在该循环链表中插入节点，并保证插入节点之后的循环链表仍然是排序的。

```java
class Solution {
    public Node insert(Node head, int insertVal) {
        Node insertNode = new Node(insertVal);
        Node cur = head;
        while(cur != null) {
            Node next = cur.next;
            //正常递增情况下的可插入条件：
            //1. next.val > insertVal && cur.val <= insertVal 前一个数字更小，后一个数字更大
            //2. next.val < cur.val && cur.val <= insertVal insertVal比链表中所有数字都更大
            //3. next.val < cur.val && cur.val > insertVal && next.val > insertVal insertVal比链表中所有数字都更小
            //所有元素大小相同的循环链表，最多绕一圈：cur.next == head 时直接插入（！！！避免死循环）
            if (((next.val > insertVal || next.val < cur.val) && cur.val <= insertVal) 
                || (next.val < cur.val && cur.val > insertVal && next.val > insertVal) 
                || cur.next == head) { 
                cur.next = insertNode;
                insertNode.next = next;
                break;
            } else {
                cur = next;
            }
        }

        if(head == null) {
            //输入链表为空的情况下，插入的节点需要自己形成循环链表
            insertNode.next = insertNode;
            return insertNode;
        } else {
            return head;
        }
    }
}
```



## 栈

> 函数pop和peek都能返回位于栈顶的元素，但函数pop会将位于栈顶的元素出栈，而函数peek不会。Stack的函数push、pop和peek的时间复杂度都是O（1）。

#### [036. 后缀表达式](https://leetcode.cn/problems/8Zf90G/?favorite=e8X3pBZi)

> 后缀表达式是一种算术表达式，它的操作符在操作数的后面。输入一个用字符串数组表示的后缀表达式，请输出该后缀表达式的计算结果。假设输入的一定是有效的后缀表达式。例如，后缀表达式["2"，"1"，"3"，"*"，"+"]对应的算术表达式是“2+1*3”，因此输出它的计算结果5。
>
> 后缀表达式又叫逆波兰式（Reverse Polish Notation，RPN），是一种将操作符放在操作数后面的算术表达式。通常用的是中缀表达式，即操作符位于两个操作数的中间，如“2+1*3”。使用后缀表达式的好处是不需要使用括号。例如，中缀表达式的“2+1*3”和“（2+1）*3”不相同。它们的后缀表达式分别为“213*+”和“21+3*”。后缀表达式不使用括号也能无歧义地表达这两个不同的算术表达式。

```java
class Solution {
    public int evalRPN(String[] tokens) {
        //使用栈来保存操作数，遇到操作符的时候，出栈两个数计算，计算结果再入栈、
        Stack<Integer> stack = new Stack<>();
        for(String t: tokens) {
            switch(t){
                case "+": 
                case "-": 
                case "*": 
                case "/": {
                    int num2 = stack.pop(); //先出栈的数作为第二个操作数，例如除数
                    int num1 = stack.pop();
                    int result = caculateResult(t, num1, num2);
                    stack.push(result);
                    break; //不要忘记Java的break
                }
                default: stack.push(Integer.parseInt(t));
            };
        }
        //无操作符时出栈最后结果返回
        return stack.peek();
    }

    private int caculateResult(String operator, int num1, int num2) {
        switch(operator) {
            case "+": return num1 + num2;
            case "-": return num1 - num2;
            case "*": return num1 * num2;
            case "/": return num1 / num2; //题目条件给出 除数不为0                
            default: return 0;
        }
    }
}
```

+ 如果输入数组的长度是n，那么对其中的每个字符串都有一次push操作；如果是操作符，那么还需要进行数学计算和两次push操作。由于每个push操作、pop操作和数学计算都是O（1），因此总体时间复杂度是O（n）。由于栈中可能有O（n）个操作数，因此这种解法的空间复杂度也是O（n）。

#### [037. 小行星碰撞](https://leetcode.cn/problems/XagZNi/?favorite=e8X3pBZi)

> 输入一个表示小行星的数组，数组中每个数字的绝对值表示小行星的大小，数字的正负号表示小行星运动的方向，正号表示向右飞行，负号表示向左飞行。如果两颗小行星相撞，那么体积较小的小行星将会爆炸最终消失，体积较大的小行星不受影响。如果相撞的两颗小行星大小相同，那么它们都会爆炸消失。飞行方向相同的小行星永远不会相撞。求最终剩下的小行星。

```java
class Solution {
    /*
    直观理解上，只要画出这些星星的顺序平面图,
    例如[-2,-1,1,2]， [<--(-2),<--(-1),(1)-->,(2)-->],所以它们不会存在相撞的情况。

    每次栈顶元素和当前元素比较，没有产生相撞则入栈；否则未消失星的入栈。
    1. 如果当前星是正数，不会产生碰撞
    2. 当前星为负数，如果栈顶为正数则会产生碰撞
        （1）都消失
        （2）栈顶消失，当前星入栈
        （3）栈顶保留，当前星消失
     */
    public int[] asteroidCollision(int[] asteroids) {
        Stack<Integer> stack = new Stack();
        for(int asteroid: asteroids) {
            if (stack.isEmpty() || asteroid > 0 || (asteroid < 0 && stack.peek() < 0)) {
                stack.push(asteroid);
            } else {
                int temp = 0; //asteroids[i] != 0
                while(!stack.isEmpty() && stack.peek() > 0){
                    if (stack.peek() + asteroid == 0) {
                        stack.pop();
                        temp = 0;
                        break;
                    } else if (stack.peek() + asteroid < 0) {
                        stack.pop();
                        //当前星只要没有消失，都需要继续判断是否出栈
                        temp = asteroid;
                    } else {
                        temp = 0;
                        break;
                    }
                }
                if (temp != 0) {
                    stack.push(temp);
                }
            }
        }

        int[] result = new int[stack.size()];
        for(int i = result.length - 1; i >= 0; i--) {
            result[i] = stack.pop();
        }
        return result;
    }
}
```

精简代码如下：

```java
class Solution {
    public int[] asteroidCollision(int[] asteroids) {
        // asteroid > 0 不会发生碰撞，直接入栈
        // asteroid < 0 && peek < 0 不会发生碰撞，直接入栈
        // asteroid < 0 && peek > 0 会发生碰撞
        // （1）asteroid + peek == 0 都消失
        // （2）asteroid + peek < 0 peek消失
        // （3）asteroid + peek > 0 asteroid消失

        Stack<Integer> stack = new Stack();
        for(int asteroid: asteroids) {
            //先处理必须出栈的情况，之后入栈asteroid
            while(!stack.isEmpty() && stack.peek() > 0 && stack.peek() < -asteroid) {
                stack.pop();
            }
            //peek和asteroid一同消失的情况
            if (!stack.isEmpty() && stack.peek() > 0 && stack.peek() == -asteroid) {
                stack.pop();
            } else if(stack.isEmpty() || asteroid > 0 || (asteroid < 0 && stack.peek() < 0)){
                stack.push(asteroid);
            }
            //其他就是asteroid因碰撞消失的情况
        }

        return stack.stream().mapToInt(i -> i).toArray();
    }
}
```

+ 假设有n颗小行星。上述代码中有一个嵌套的二重循环，它的时间复杂度是不是O（n2）？由于每颗小行星只可能入栈、出栈一次，因此时间复杂度是O（n），空间复杂度也是O（n）。

#### [038. 每日温度](https://leetcode.cn/problems/iIQa4I/?favorite=e8X3pBZi)

> 输入一个数组，它的每个数字是某天的温度。请计算每天需要等几天才会出现更高的温度。例如，如果输入数组[35，31，33，36，34]，那么输出为[3，1，1，0，0]。由于第1天的温度是35℃，要等3天才会出现更高的温度36℃，因此对应的输出为3。第4天的温度是36℃，后面没有更高的温度，它对应的输出是0。其他的以此类推。

保存在栈中的温度（通过数组的下标可以得到温度）是递减排序的。这是因为如果当前温度比位于栈顶的温度高，位于栈顶的温度将出栈，所以每次入栈时当前温度一定比位于栈顶的温度低或相同。

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        //依次入栈每日的温度，入栈之前和栈顶的温度比较，
        //不断和栈顶温度比较（如果比栈顶温度高，栈顶就找到结果可以出栈了），至不比栈顶温度高就可以入栈了。
        //最后停留在栈内的就是没有温度更高的日子了
        //为了便于比较日期的差异，入栈的应该是温度的数组下标
        int[] result = new int[temperatures.length];
        Stack<Integer> stack = new Stack();
        for(int i = 0; i < temperatures.length; i++) {
            //先处理出栈的内容
            while(!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
                result[stack.peek()] = i - stack.peek();
                stack.pop();
            }
            stack.push(i);
        }
        return result;
    }
}
```

+ 假设输入数组的长度为n。虽然上述代码中有一个嵌套的二重循环，但它的时间复杂度是O（n），这是因为数组中每个温度入栈、出栈各1次。
+ 这种解法的空间复杂度也是O（n）。

#### [039. 直方图最大矩形面积](https://leetcode.cn/problems/0ynMMM/?favorite=e8X3pBZi)

> 直方图是由排列在同一基线上的相邻柱子组成的图形。输入一个由非负数组成的数组，数组中的数字是直方图中柱子的高。求直方图中最大矩形面积。假设直方图中柱子的宽都为1。

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int maxArea = 0;
        //维持一个单调递增栈，用于记录一个柱子前面比它矮的柱子的下标（会制约当前柱子往两遍延伸形成的矩形面积）
        Stack<Integer> stack = new Stack();
        for (int i = 0; i < heights.length; i++) {
            //先处理出栈
            while(!stack.isEmpty() && heights[stack.peek()] >= heights[i]) {
                int height = heights[stack.pop()];
                int leftIndex = !stack.isEmpty() ? stack.peek() : -1;
                int width = i - leftIndex - 1;
                maxArea = Math.max(maxArea, height * width);
            }
            stack.push(i);
        }
        //最后还没出栈的柱子，意味着右边没有比自己矮的柱子
        while(!stack.isEmpty()) {
            int height = heights[stack.pop()];
            int leftIndex = !stack.isEmpty() ? stack.peek() : -1;
            int width = heights.length - 1 - leftIndex;
            maxArea = Math.max(maxArea, height * width);
        }
        return maxArea;
    }
}
```

+ 假设输入数组的长度为n。直方图的每根柱子都入栈、出栈一次，并且在每根柱子的下标出栈时计算以它为顶的最大矩形面积，这些操作对每根柱子而言时间复杂度是O（1），因此这种单调栈法的时间复杂度是O（n）。
+ 这种解法需要一个辅助栈，栈中可能有O（n）根柱子在数组中的下标，因此空间复杂度是O（n）。

#### [040. 矩阵中最大的矩形](https://leetcode.cn/problems/PLYXKQ/?favorite=e8X3pBZi)

> 请在一个由0、1组成的矩阵中找出最大的只包含1的矩形并输出它的面积。

转换成求直方图最大矩形面积。

```java
class Solution {
    public int maximalRectangle(String[] matrix) {
        if(matrix == null || matrix.length == 0 || matrix[0] == null || matrix[0].length() == 0) {
            return 0;
        }
        //转换成直方图求最大矩形面积的形式
        //每次扩宽纵向高度，求当前高度范围下，1形成的最大矩形面积
        int maxResult = 0;
        int[] heights = new int[matrix[0].length()];
        for(int i = 0; i < matrix.length; i++) { //确定横轴的起始位置
            for(int j = 0; j < matrix[0].length(); j++) { //确定纵向的连续1形成柱子的高度
                if (matrix[i].charAt(j) == '1') {
                    heights[j]++;
                } else {
                    heights[j] = 0;
                }
            }
            //以第i层为横轴的连续1高度就形成了
            maxResult = Math.max(largestRectangleArea(heights), maxResult);
        }

        return maxResult;
    }

    private int largestRectangleArea(int[] heights) {
        int maxArea = 0;
        //维持一个单调递增栈，用于记录一个柱子前面比它矮的柱子的下标（会制约当前柱子往两遍延伸形成的矩形面积）
        Stack<Integer> stack = new Stack();
        for (int i = 0; i < heights.length; i++) {
            //先处理出栈
            while(!stack.isEmpty() && heights[stack.peek()] >= heights[i]) {
                int height = heights[stack.pop()];
                int leftIndex = !stack.isEmpty() ? stack.peek() : -1;
                int width = i - leftIndex - 1;
                maxArea = Math.max(maxArea, height * width);
            }
            stack.push(i);
        }
        //最后还没出栈的柱子，意味着右边没有比自己矮的柱子
        while(!stack.isEmpty()) {
            int height = heights[stack.pop()];
            int leftIndex = !stack.isEmpty() ? stack.peek() : -1;
            int width = heights.length - 1 - leftIndex;
            maxArea = Math.max(maxArea, height * width);
        }
        return maxArea;
    }
}
```

+ 假设输入的矩阵的大小为m×n。该矩阵可以转换成m个直方图。如果采用单调栈法，那么求每个直方图的最大矩形面积需要O（n）的时间，因此这种解法的时间复杂度是O（mn）。
+ 用单调栈法计算直方图中最大矩阵的面积需要O（n）的空间，同时还需要一个长度为n的数组heights，用于记录直方图中柱子的高度，因此这种解法的空间复杂度是O（n）。

## 队列

### 队列的应用

#### [041. 滑动窗口的平均值](https://leetcode.cn/problems/qIsx9U/?favorite=e8X3pBZi)

> 请实现如下类型MovingAverage，计算滑动窗口中所有数字的平均值，该类型构造函数的参数确定滑动窗口的大小，每次调用成员函数next时都会在滑动窗口中添加一个整数，并返回滑动窗口中所有数字的平均值。

```java
class MovingAverage {

    private Queue<Integer> queue = new LinkedList();
    private int capacity;
    private int sum;

    /** Initialize your data structure here. */
    public MovingAverage(int size) {
        capacity = size;
    }
    
    public double next(int val) {
        queue.offer(val);
        sum += val;
        if (queue.size() > capacity) {
            sum -= queue.poll();
        }
        return (double)sum / queue.size(); //题目条件size >= 1
    }
}

/**
 * Your MovingAverage object will be instantiated and called as such:
 * MovingAverage obj = new MovingAverage(size);
 * double param_1 = obj.next(val);
 */
```

#### [042. 最近请求次数](https://leetcode.cn/problems/H8086Q/description/?favorite=e8X3pBZi)

> 请实现如下类型RecentCounter，它是统计过去3000ms内的请求次数的计数器。该类型的构造函数RecentCounter初始化计数器，请求数初始化为0；函数ping（int t）在时间t添加一个新请求（t表示以毫秒为单位的时间），并返回过去3000ms内（时间范围为[t-3000，t]）发生的所有请求数。假设每次调用函数ping的参数t都比之前调用的参数值大。

```java
class RecentCounter {

    private Queue<Integer> queue;
    private int windowSize;

    public RecentCounter() {
        queue = new LinkedList();
        windowSize = 3000;
    }
    
    public int ping(int t) {
        queue.offer(t);
        while (queue.peek() + windowSize < t) {
            queue.poll();
        }
        return queue.size();
    }
}

/**
 * Your RecentCounter object will be instantiated and called as such:
 * RecentCounter obj = new RecentCounter();
 * int param_1 = obj.ping(t);
 */
```

+ 假设计数器时间窗口的大小是w毫秒，其中记录的时间是递增的，那么时间窗口中记录的时间的数目是O（w），因此空间复杂度是O（w）。
+ 每当收到一个新的请求ping时，由于可能需要删除O（w）个已经滑出时间窗口的请求，因此时间复杂度也是O（w）。但是由于这个题目中时间窗口的大小为3000毫秒，w是一个常数，因此也可以认为时间复杂度和空间复杂度都是O（1）

### 二叉树的广度优先搜索

如果关于二叉树的面试题提到层这个概念，就可以尝试运用广度优先搜索来解决这个问题。

#### [043. 往完全二叉树添加节点](https://leetcode.cn/problems/NaqhDT/?favorite=e8X3pBZi)

> 在完全二叉树中，除最后一层之外其他层的节点都是满的（第n层有2n-1个节点）。最后一层的节点可能不满，该层所有的节点尽可能向左边靠拢。
>
> 实现数据结构CBTInserter有如下3种方法。
>
> + 构造函数CBTInserter（TreeNode root），用一棵完全二叉树的根节点初始化该数据结构。
> + 函数insert（int v）在完全二叉树中添加一个值为v的节点，并返回被插入节点的父节点。
> + 函数get_root()返回完全二叉树的根节点。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class CBTInserter {

    private Queue<TreeNode> queue = new LinkedList();
    private TreeNode root;

    /**
    root是给定树
     */
    public CBTInserter(TreeNode root) {
        this.root = root;
        queue.offer(root);
    }
    
    public int insert(int v) {
        //使用队列进行层序遍历， 并保持队头TreeNode可以衔接子节点
        //出队的TreeNode，才将它的叶子节点入队
        while(queue.peek() != null && queue.peek().left != null && queue.peek().right != null) {
            TreeNode parent = queue.poll();
            TreeNode left = parent.left;
            TreeNode right = parent.right;
            if (left != null) {
                queue.offer(left);
            }
            if (right != null) {
                queue.offer(right);
            }
        }

        //新节点插入二叉树
        TreeNode node = new TreeNode(v);
        TreeNode parentNode = queue.peek();
        if (parentNode.left == null) {
            parentNode.left = node;
        } else {
            parentNode.right = node;
        }
        //此处不需要着急入队的queue.offer(node)，只要处理好二叉树的父子关系就行，否则会破坏队列中的层序
        return parentNode.val; 
    }
    
    public TreeNode get_root() {
        return this.root;
    }
}

/**
 * Your CBTInserter object will be instantiated and called as such:
 * CBTInserter obj = new CBTInserter(root);
 * int param_1 = obj.insert(v);
 * TreeNode param_2 = obj.get_root();
 */
```

#### [044. 二叉树每层的最大值](https://leetcode.cn/problems/hPov7L/?favorite=e8X3pBZi)

> 输入一棵二叉树，请找出二叉树中每层的最大值。

使用一个队列完成层序遍历，并且找出每一层中的最大值，变量记录每一层的节点数量。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> largestValues(TreeNode root) {
        List<Integer> result = new LinkedList();
        if (root == null) {
            return result;
        }
        //层序遍历二叉树
        //每一层单独比较大小得出最大值
        Queue<TreeNode> queue = new LinkedList();
        queue.offer(root);

        int currentSize = 1; //当前这层的节点数量
        int next = 0; //下一层的节点数量
        int maxNum = Integer.MIN_VALUE;
        while(!queue.isEmpty()) {
            TreeNode parent = queue.poll();
            currentSize--;
            maxNum = Math.max(maxNum, parent.val);

            if (parent.left != null) {
                queue.offer(parent.left);
                next++;
            }
            if (parent.right != null) {
                queue.offer(parent.right);
                next++;
            }

            if(currentSize == 0) {
                currentSize = next;
                result.add(maxNum);
                maxNum = Integer.MIN_VALUE;
                next = 0;
            }
        }

        return result;
    }
}
```

使用两个队列实现二叉树的广度优先遍历。逻辑更简单一些。如果用广度优先的顺序遍历二叉树时需要区分二叉树的每层的情况就可以采用两个队列来实现。

```java
class Solution {
    public List<Integer> largestValues(TreeNode root) {
        List<Integer> result = new LinkedList();
        if (root == null) {
            return result;
        }
        //队列辅助层序遍历，queue1存储当前层，queue2存储下一层
        Queue<TreeNode> queue1 = new LinkedList();
        Queue<TreeNode> queue2 = new LinkedList();
        queue1.offer(root);

        int maxNum = Integer.MIN_VALUE;
        while(!queue1.isEmpty()) {
            TreeNode parent = queue1.poll();
            maxNum = Math.max(maxNum, parent.val);

            if (parent.left != null) {
                queue2.offer(parent.left);
            }
            if (parent.right != null) {
                queue2.offer(parent.right);
            }

            //一层遍历完了，记录结果，并重置两个queue
            if (queue1.isEmpty()) {
                result.add(maxNum);
                maxNum = Integer.MIN_VALUE;

                Queue<TreeNode> temp = queue1;
                queue1 = queue2;
                queue2 = temp;
            }
        }

        return result;
    }
}
```

#### [045. 二叉树最底层最左边的](https://leetcode.cn/problems/LwUNpT/?favorite=e8X3pBZi)

> 如何在一棵二叉树中找出它最低层最左边节点的值？假设二叉树中最少有一个节点。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    //输入的根节点不一定是一个完全二叉树
    public int findBottomLeftValue(TreeNode root) {
        //使用两个队列完成层序遍历，
        //每一层遍历的方向从右到左!!!
        Queue<TreeNode> queue1 = new LinkedList();
        Queue<TreeNode> queue2 = new LinkedList();
        queue1.offer(root);
        while(!queue1.isEmpty() ) {
            TreeNode parent = queue1.poll();
            if (parent.right != null) {
                queue2.offer(parent.right);
            }
            if (parent.left != null) {
                queue2.offer(parent.left);
            }

            if (queue1.isEmpty()) {
                if (!queue2.isEmpty()) {
                    Queue<TreeNode> temp = queue1;
                    queue1 = queue2;
                    queue2 = temp;
                } else {
                    //已经是最后一层，且当前出队的是最左边的节点
                    return parent.val;
                }
            }
        }
        return 0;
    }
}
```

#### [046. 二叉树的右侧视图](https://leetcode.cn/problems/WNC0Lk/?favorite=e8X3pBZi)

> 给定一棵二叉树，如果站在该二叉树的右侧，那么从上到下看到的节点构成二叉树的右侧视图。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> result = new LinkedList();
        if (root == null) {
            return result;
        }

        //两个队列完成层序遍历，每次队尾就是要收集的值
        Queue<TreeNode> queue1 = new LinkedList();
        Queue<TreeNode> queue2 = new LinkedList();
        queue1.offer(root);
        while(!queue1.isEmpty()) {
            TreeNode parent = queue1.poll();
            if (parent.left != null) {
                queue2.offer(parent.left);
            }
            if (parent.right != null) {
                queue2.offer(parent.right);
            }
            if(queue1.isEmpty()){
                result.add(parent.val);
                Queue<TreeNode> temp = queue1; 
                queue1 = queue2;
                queue2 = temp;
            }
        }

        return result;
    }
}
```

## 树

### 树的基础知识

> 其中二叉树中每个节点最多只有两个子节点，可以分别把它们称为左子节点和右子节点。且二叉树本身就是递归的数据结构。
>
> 1. 深度优先搜索更加适合解决与路径相关的面试题。

### 二叉树的深度优先搜索

#### [047. 二叉树剪枝](https://leetcode.cn/problems/pOCWxh/?favorite=e8X3pBZi)

> 一棵二叉树的所有节点的值要么是0要么是1，请剪除该二叉树中所有节点的值全都是0的子树。

```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}
class Solution {
    /**
     * 后续遍历途中，将叶子节点逐个减掉:
     * 如果用后序遍历的顺序遍历到某个节点，那么它的左右子树的节点一定已经遍历过了。
     */
    public TreeNode pruneTree(TreeNode root) {
        if (root == null) {
            return root;
        }
        root.left = pruneTree(root.left);
        root.right = pruneTree(root.right);
        if(root.left == null && root.right == null && root.val == 0) {
            return null;
        }
        return root;
    }
}
```

#### [048. 序列化与反序列化二叉树](https://leetcode.cn/problems/h54YBf/?favorite=e8X3pBZi)

> 请设计一个算法将二叉树序列化成一个字符串，并能将该字符串反序列化出原来二叉树的算法。

层序遍历：

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Codec {

    //使用层序遍历记录下二叉树的完整结构
    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        StringBuilder treeContents = new StringBuilder();
        if (root == null) {
            return treeContents.toString();
        }

        //层序遍历
        Queue<TreeNode> queue = new LinkedList();
        queue.offer(root);
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if (node != null) {
                queue.offer(node.left);
                queue.offer(node.right);
                treeContents.append(node.val + ",");
            } else {
                treeContents.append("null,");
            }
        }
        return treeContents.toString();
    }


    /**
    恢复二叉树时，由于题目并没有说输入的二叉树是完全二叉树，因此不能通过左右子节点不为空来判断一个节点的配置是否完备。
    1. 使用一个标记位来记录每个节点的子节点配置次数
     */
    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        TreeNode root = null;

        //如果输入二叉树为null,直接返回
        if (data == null || data.isEmpty()) {
            return root;
        }

        Queue<TreeNode> queue = new LinkedList();
        String[] nodeContents = data.split(",");
        int operateRecord = 0;
        for(String nodeContent: nodeContents) {
            //创建当前节点
            TreeNode node;
            if (nodeContent.equals("null")) {
                node = null;
            } else {
                node = new TreeNode(Integer.parseInt(nodeContent));
            }
            operateRecord++;

            /**
            找父节点, 并完成父子配置
            1. 队内不为null的元素才需要配置子节点，让其出队
            2. 如果有父节点，则将当前节点配置为父节点的左/右子节点
            */
            while(!queue.isEmpty() && queue.element() == null) {
                queue.poll();
            }
            if(!queue.isEmpty()) {
                TreeNode parentNode = queue.element();
                if (operateRecord == 1) {
                    parentNode.left = node;
                } else if (operateRecord == 2) {
                    parentNode.right = node;
                    //同时，配置完备的节点就需要出队了
                    queue.poll();
                    operateRecord = 0;
                }
            }

            //当前节点入队，无论是否null
            queue.offer(node);

            //如果是根节点则需要被记录下来
            if(root == null) {
                root = node;
                //operateRecord是为父节点配置左右子节点服务的，
                //如果当前节点已经是根节点了，则operateRecord则没有必要记录
                operateRecord = 0;
            }
        }
        return root;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec ser = new Codec();
// Codec deser = new Codec();
// TreeNode ans = deser.deserialize(ser.serialize(root));
```

前序遍历递归方式：

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Codec {

    private int index = 0;//也可以创建一个数组记录这个值的变化

    // 前序递归遍历记录二叉树的结构
    public String serialize(TreeNode root) {
        if(root == null) {
            return "#";
        }

        String left = serialize(root.left);
        String right = serialize(root.right);
        return root.val + "," + left + "," + right;
    }

    public TreeNode deserialize(String data) {
        String[] nodeContents = data.split(",");
        return deserialize(nodeContents);
    }

    public TreeNode deserialize(String[] nodeContents) {
        String content = nodeContents[index];
        index++;
    
        if(content.equals("#")) {
            return null;
        }

        TreeNode node = new TreeNode(Integer.parseInt(content));
        node.left = deserialize(nodeContents);
        node.right = deserialize(nodeContents);
        return node;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec ser = new Codec();
// Codec deser = new Codec();
// TreeNode ans = deser.deserialize(ser.serialize(root));
```

#### [049. 从根节点到叶节点的路径数字之和](https://leetcode.cn/problems/3Etpl5/?favorite=e8X3pBZi)

> 在一棵二叉树中所有节点都在0～9的范围之内，从根节点到叶节点的路径表示一个数字。求二叉树中所有路径表示的数字之和。

```java
class Solution {
    public int sumNumbers(TreeNode root) {
        List<Integer> nums = new LinkedList();
        sumNumbers(root, "", nums);
        return addNums(nums);
    }

    private int addNums(List<Integer> nums) {
        int count = 0;
        for(int i = 0; i < nums.size(); i++) {
            count += nums.get(i);
        }
        return count;
    }

    //递归前序遍历，使用字符串记录每一个路径下对应的数字
    private void sumNumbers(TreeNode root, String pathContent, List<Integer> nums) {
        if (root == null) {
            return;
        }

        String currentContent = pathContent + root.val;
        if (root.left == null && root.right == null) {
            nums.add(Integer.parseInt(currentContent));
            return;
        }

        sumNumbers(root.left, currentContent, nums);
        sumNumbers(root.right, currentContent, nums);
    }
}
```

精简代码如下：

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int sumNumbers(TreeNode root) {
        return sumNumbers(root, 0);
    }

    //递归前序遍历, 并在遍历过程中处理计算
    private int sumNumbers(TreeNode root, int pathNum) {
        if (root == null) {
            return 0;
        }

        int currentNum = pathNum * 10 + root.val;
        if (root.left == null && root.right == null) {
            return currentNum;
        }

        return sumNumbers(root.left, currentNum) + sumNumbers(root.right, currentNum);
    }
}
```

#### [050. 向下的路径节点之和](https://leetcode.cn/problems/6eUYwP/?favorite=e8X3pBZi)

> 给定一棵二叉树和一个值sum，求二叉树中节点值之和等于sum的路径的数目。路径的定义为二叉树中顺着指向子节点的指针向下移动所经过的节点，但**不一定从根节点开始，也不一定到叶节点结束**。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    /**
    即根节点到叶子节点形成的路径中的子路径可能为targetSum，
    1. 前序遍历、递归、根节点到当前节点的路径之和sum
    2. num = sum - targetSum， 如果num为根节点到”当前节点之前的节点“的路径之和，则存在子路径之和为targetSum
    3. 使用Map记录路径之和出现的次数
     */
    public int pathSum(TreeNode root, long targetSum) {
        HashMap<Long, Integer> map = new HashMap<>();
        map.put(0L, 1);
        return pathSum(root, targetSum, map, 0L);
    }

    private int pathSum(TreeNode root, long targetSum, HashMap<Long, Integer> map, long sum) {
        if (root == null) {
            return 0;
        }
        long curSum = sum + root.val;
        long num = curSum - targetSum; //如果为0也是符合预期的
        int count = map.getOrDefault(num, 0);
        map.put(curSum, map.getOrDefault(curSum, 0) + 1);//map.getOrDefault
        
        count += pathSum(root.left, targetSum, map, curSum);
        count += pathSum(root.right, targetSum, map, curSum);
        map.put(curSum, map.get(curSum) - 1));
        //该函数结束时，程序将回到节点的父节点，也就是说，在函数结束之前需要将当前节点从路径中删除，从根节点到当前节点累加的节点值之和也要从哈希表map中删除。
        return count;
    }
}
```

#### [051. 节点之和最大的路径](https://leetcode.cn/problems/jC7MId/?favorite=e8X3pBZi)

> 在二叉树中将路径定义为顺着节点之间的连接从**任意一个节点开始到达任意一个节点所经过的所有节点**。路径中至少包含一个节点，不一定经过二叉树的根节点，也不一定经过叶节点。给定非空的一棵二叉树，请求出二叉树所有路径上节点值之和的最大值。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int maxPathSum(TreeNode root) {
        int[] maxNum = {Integer.MIN_VALUE};
        dfs(root, maxNum);
        return maxNum[0];
    }

    /**
    一条符合条件的路径：
    1. 左子节点 - 父节点 - 右子节点 
    2. 子节点 - 父节点 - 父节点的父节点
    当路径到达某个节点时，该路径既可以前往它的左子树，也可以前往它的右子树。
    但如果路径同时经过它的左右子树，那么就不能经过它的父节点。

    后续遍历：
    先求出左右子树中路径节点值之和的最大值（左右子树中的路径不经过当前节点），
    再求出经过根节点的路径节点值之和的最大值，
    最后对三者进行比较得到最大值。
    
    数组maxPathSum记录下全局的最大路径和，
    maxPathSum方法返回根节点到一端子树的最大路径和。
     */
    private int dfs(TreeNode root, int[] maxNum) {
        if (root == null) {
            return 0;
        }

        //分别求子树中”经过/不经过“根节点的最大路径和
        int[] maxLeft = {Integer.MIN_VALUE};
        int left = Math.max(0, dfs(root.left, maxLeft));
        int[] maxRight = {Integer.MIN_VALUE};
        int right = Math.max(0, dfs(root.right, maxRight));

        //求出左右子树中路径节点值之和的最大值（左右子树中的路径不经过当前节点）
        maxNum[0] = Math.max(maxLeft[0], maxRight[0]);
        //求出经过根节点的路径节点值之和的最大值
        maxNum[0] = Math.max(maxNum[0], left + root.val + right);

        //返回根到一端子树的路径和最大值
        return root.val + Math.max(left, right);
    }
}
```

### ==二叉搜索树==

> 二叉搜索树是一类特殊的二叉树，它的左子节点总是小于或等于根节点，而右子节点总是大于或等于根节点。
>
> 中序遍历是解决二叉搜索树相关面试题最常用的思路，这是因为中序遍历按照节点值递增的顺序遍历二叉搜索树的每个节点。

#### [052. 展平二叉搜索树](https://leetcode.cn/problems/NYBBNL/?favorite=e8X3pBZi)

> 给定一棵二叉搜索树，请调整节点的指针使每个节点都没有左子节点。调整之后的树看起来像一个链表，但仍然是二叉搜索树。

```java
//递归
class Solution {
    public TreeNode increasingBST(TreeNode root) {
        if (root == null) {
            return null;
        }

        TreeNode left = increasingBST(root.left);
        if (left != null) {
            //找到最右叶子节点
            TreeNode cur = left;
            while(cur.right != null) {
                cur = cur.right;
            }
            cur.right = root;
        } else {
            left = root;
        }
        TreeNode right = increasingBST(root.right);
        root.left = null;
        root.right = right;
        
        return left;
    }
}
```

```java
class Solution {

    private TreeNode resNode;

    public TreeNode increasingBST(TreeNode root) {
        TreeNode dummyNode = new TreeNode();
        resNode = dummyNode;
        inOrderTraversal(root);
        return dummyNode.right;
    }

    public void inOrderTraversal(TreeNode root) {
        if (root == null) {
            return;
        }

        inOrderTraversal(root.left);
        resNode.right = root;
        root.left = null;
        resNode = root;
        inOrderTraversal(root.right);
    }
}
```

迭代遍历：

```java
class Solution {
    public TreeNode increasingBST(TreeNode root) {
        TreeNode dummyNode = new TreeNode();
        TreeNode curDummy = dummyNode;
        //中序遍历、迭代，每一个节点重新组装右/左子树
        TreeNode cur = root;
        Stack<TreeNode> stack = new Stack();
        while(cur != null || !stack.isEmpty()) {
            while(cur != null) {
                stack.push(cur);
                cur = cur.left;
            }

            TreeNode node = stack.pop();
            curDummy.right = node;
            node.left = null;
            curDummy = node;
            cur = node.right;
        }
        return dummyNode.right;
    }
}
```

#### [053. 二叉搜索树中的中序后继](https://leetcode.cn/problems/P5rCT8/?favorite=e8X3pBZi)

> 给定一棵二叉搜索树和它的一个节点p，请找出按中序遍历的顺序该节点p的下一个节点。假设二叉搜索树中节点的值都是唯一的。

```java
```



## 二分查找

### 排序数组中二分查找

#### [068. 查找插入位置](https://leetcode.cn/problems/N6YdxV/?favorite=e8X3pBZi)

> 给定一个排序的整数数组 nums 和一个整数目标值 target ，请在数组中找到 target ，并返回其下标。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。
>
> 请必须使用时间复杂度为 O(log n) 的算法。

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        while(left <= right) {
            int mid = (left + right) / 2;
            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] < target) {
                left = mid + 1;
                if (left < nums.length && nums[left] > target) {
                    return left;
                } 
            } else {
                right = mid - 1;
                if (right >= 0 && nums[right] < target) {
                    return mid;
                }
            }
        }

        return (right < 0) ? 0 : left;
    }
}
```

更简洁

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        while(left <= right) {
            int mid = (left + right) / 2;
            if (nums[mid] >= target) { //往左边找
                if(mid == 0 || nums[mid - 1] < target) {
                    return mid;
                }
                right = mid - 1;
            } else if (nums[mid] < target) { //往又边找
                left = mid + 1;
            } 
        }

        return nums.length;
    }
}
```

+ 时间复杂度：O(logn)，其中 n 为数组的长度。二分查找所需的时间复杂度为 O(logn)。

+ 空间复杂度：O(1)。我们只需要常数空间存放若干变量。

#### [069. 山峰数组的顶部](https://leetcode.cn/problems/B1IidL/?favorite=e8X3pBZi)

> 在一个长度大于或等于3的数组中，任意相邻的两个数字都不相等。该数组的前若干数字是递增的，之后的数字是递减的，因此它的值看起来像一座山峰。请找出山峰顶部，即数组中最大值的位置。例如，在数组[1，3，5，4，2]中，最大值是5，输出它在数组中的下标2。

```java
class Solution {
    public int peakIndexInMountainArray(int[] arr) {
        //根据题目条件，首尾两个元素肯定不是目标，查找范围可缩小为1~n-2，也可以防止下标越界
        int left = 1;
        int right = arr.length - 2;
        while(left <= right) {
            int mid = (left + right) / 2;
            if (arr[mid] > arr[mid - 1] && arr[mid] > arr[mid + 1]) { //山
                return mid;
            } else if (arr[mid] > arr[mid - 1]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        } 
        return -1;
    }
}
```

#### [070. 排序数组中只出现一次的数字](https://leetcode.cn/problems/skFtm2/?favorite=e8X3pBZi)

> 给定一个只包含整数的==有序==数组 `nums` ，每个元素都会出现==两次==，唯有一个数只会出现一次，请找出这个唯一的数字。你设计的解决方案必须满足 `O(log n)` 时间复杂度和 `O(1)` 空间复杂度。

```java
class Solution {
    public int singleNonDuplicate(int[] nums) {
        //有序、两次，只有一个数只出现一次
        //将数组元素按序，每“两个”分为一组，那么从只出现一次的数字开始，每组数字都将不同
        int left = 0;
        int right = nums.length / 2;
        while(left <= right) {
            int mid = (left + right) / 2;
            int i = mid * 2; //小组第一个数字
            if (i < nums.length - 1 && nums[i] != nums[i + 1]) {
                if (mid == 0 || nums[i - 2] == nums[i - 1]) {
                    return nums[i];
                } 
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        //如果上述还没有找到答案，则是最后一个元素
        return nums[nums.length - 1];
    }
}
```

#### [071. 按权重生成随机数](https://leetcode.cn/problems/cuyjEf/?favorite=e8X3pBZi)（没看懂题目，后续）

> 给定一个正整数数组 `w` ，其中 `w[i]` 代表下标 `i` 的权重（下标从 `0` 开始），请写一个函数 `pickIndex` ，它可以随机地获取下标 `i`，选取下标 `i` 的概率与 `w[i]` 成正比。
>
> 例如，对于 `w = [1, 3]`，挑选下标 `0` 的概率为 `1 / (1 + 3) = 0.25` （即，25%），而选取下标 `1` 的概率为 `3 / (1 + 3) = 0.75`（即，75%）。
>
> 也就是说，选取下标 `i` 的概率为 `w[i] / sum(w)` 。

```java
```

### 在数值范围内二分查找

#### [072. 求平方根](https://leetcode.cn/problems/jJ0w9p/?favorite=e8X3pBZi)

> 输入一个非负整数，请计算它的平方根。正数的平方根有两个，只输出其中的==正数平方根==。如果平方根不是整数，那么只需要输出它的==整数部分==。例如，如果输入4则输出2；如果输入18则输出4。

```java
class Solution {
    //下面通过强制类型转换防止了产生溢出时可能造成的程序执行超出时间限制。
    public int mySqrt(int x) {
        int min = 0;
        int max = x;
        while(min <= max) {
            int mid = (min + max) / 2;
            if ((long)mid * mid == x) {
                return mid;
            } else if ((long)mid * mid < x) {
                if ((long)((mid + 1) * (mid + 1)) > x) {
                    return mid;
                }
                min = mid + 1;
            } else {
                max = mid - 1;
            }
        }  
        return 0;  
    }
}
```

防止溢出的代码优化如下：

```java
class Solution {
    public int mySqrt(int x) {
        int min = 1; //0的平方根是0，1以上开始平方根至少是1；而且下面运算会使用备选数作为除数，不能使用0
        int max = x;
        while(min <= max) {
            int mid = (min + max) / 2;
            if (mid == (x / mid)) {
                return mid;
            } else if (mid < x / mid) {
                //(mid < x / mid 当然等价于 mid * mid < x （但是这可能会产生溢出）
                if ((mid + 1) > x / (mid + 1)) {
                    return mid;
                }
                min = mid + 1;
            } else {
                max = mid - 1;
            }
        }  
        return 0;  
    }
}
```

#### [073. 狒狒吃香蕉](https://leetcode.cn/problems/nZZqjQ/?favorite=e8X3pBZi)

> 狒狒很喜欢吃香蕉。一天它发现了n堆香蕉，第i堆有piles[i]根香蕉。门卫刚好走开，H小时后才会回来。狒狒吃香蕉喜欢细嚼慢咽，但又想在门卫回来之前吃完所有的香蕉。请问狒狒每小时至少吃多少根香蕉？如果狒狒决定每小时吃k根香蕉，而它在吃的某一堆剩余的香蕉的数目少于k，那么它只会将这一堆的香蕉吃完，下一个小时才会开始吃另一堆的香蕉。

```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int min = 1; //piles中最小元素不一定就是最小的吃速
        int max = Integer.MIN_VALUE;
        for(int pile: piles) {
            max = Math.max(max, pile);
        }

        //二分查找
        while(min <= max) {
            int mid = (min + max) / 2;
            int hours = getHours(piles, mid);

            if (hours <= h) { //吃太快
                //==h时，也许还可以找到更小的速度；
                //<h时，也有可能，速度稍微再小点，就>h不满足要求了；
                //因此需要试探下
                if(mid == 1 || getHours(piles, mid - 1) > h) {
                    return mid;
                } 
                max = mid - 1;
            } else {
                min = mid + 1;
            }
        }
        return -1;
    }

    private int getHours(int[] piles, int speed) {
        int hours = 0;
        for(int pile: piles) {
            hours += (pile + speed - 1) / speed; //这样就不需要单独计算余数了
        }
        return hours;
    }
}
```

+ 时间复杂度：如果总共有m堆香蕉，最大一堆香蕉的数目为n，函数minEatingSpeed在1到n的范围内做二分查找，需要尝试O（logn）次，每尝试一次需要遍历整个数组求出按某一速度吃完所有香蕉需要的时间，因此总的时间复杂度是O（mlogn）。

## 排序

#### [074. 合并区间](https://leetcode.cn/problems/SsGoHC/)

![image-20221102130014276](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221102130014276-7462150.png)

先根据每个区间的起点排序；然后遍历这一批排好序的区间，以此合并。

```java
class Solution074 {
    public int[][] merge(int[][] intervals) {
        Arrays.sort(intervals, (i, j) -> i[0] - j[0]);
        List<int[]> mergedResult = new LinkedList<>();
        for(int i = 0; i < intervals.length; ) {
            int[] mergedZone = {intervals[i][0], intervals[i][1]}; //记录已经合并的区间
            int j = i + 1;
            while(j < intervals.length && intervals[j][0] <= mergedZone[1]) {
                mergedZone[1] = Math.max(mergedZone[1], intervals[j][1]);
                j++;
            }
            mergedResult.add(mergedZone);
            i = j;
        }
        int[][] result = new int[mergedResult.size()][];
        return mergedResult.toArray(result); //比mergedResult.toArray()更安全
    }

    public static void main(String[] args) {
        int[][] array = {{1, 3},{4, 5},{2, 6},{8, 10},{9, 12},{15, 18}};
        for (int[] ints : new Solution074().merge(array)) {
            System.out.println(ints[0] + "  " + ints[1]);
        }
    }
}
```

+ 时间复杂度：O(nlogn)，其中 n 为区间的数量。除去排序的开销，我们只需要一次线性扫描，所以主要的时间开销是排序的 O(nlogn)。
+ 空间复杂度：O(logn)，其中 n 为区间的数量。这里计算的是存储答案之外，使用的额外空间。O(logn) 即为排序所需要的空间复杂度。

> Java Lambda 表达式

```kotlin
class Solution {
    fun merge(intervals: Array<IntArray>): Array<IntArray> {
        Arrays.sort(intervals) { i: IntArray, j: IntArray ->
            i[0] - j[0]
        }
        val mergedResult: MutableList<IntArray> = LinkedList()
        var i = 0;
        while(i < intervals.size) {
            val mergedZone = intArrayOf(intervals[i][0], intervals[i][1])
            var j = i + 1
            while(j < intervals.size && intervals[j][0] <= mergedZone[1]) {
                mergedZone[1] = Math.max(intervals[j][1], mergedZone[1])
                j++;
            }
            mergedResult.add(mergedZone)
            i = j
        }
        return mergedResult.toTypedArray() //使用到了泛型实化
    }
}
```

#### [075. 数组相对排序](https://leetcode.cn/problems/0H97ZC/)

![image-20221102135745667](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221102135745667-7462150.png)

使用计数排序的思想。

```java
public class Solution075 {
    public int[] relativeSortArray(int[] arr1, int[] arr2) {
        int[] count = new int[1001]; //根据题目条件给出的数字区间
        for(int num: arr1) {
            count[num]++;
        }
        //根据arr2中数字的顺序将arr1排序
        int index = 0;
        for(int num: arr2) {
            while(count[num] > 0) {
                arr1[index++] = num;
                count[num]--;
            }
        }
        //剩余的数字按照升序填入数组
        for(int num = 0; num < count.length; num++) {
            while(count[num] > 0) {
                arr1[index++] = num;
                count[num]--;
            }
        }
        return arr1;
    }
}
```

+ 如果数组arr1的长度为m，数组arr2的长度为n，那么时间复杂度是O（m+n）。
+ 用来统计每个数字出现次数的辅助数组counts的长度为1001，是一个常数，因此空间复杂度可以认为是O（1）。

```kotlin
class Solution075_ {
    fun relativeSortArray(arr1: IntArray, arr2: IntArray): IntArray {
        val count = IntArray(1001)
        for(num in arr1) {
            count[num]++
        }
        var index = 0
        for(num in arr2) {
            while(count[num] > 0) {
                arr1[index++] = num
                count[num]--
            }
        }
        for(i in count.indices) { //indices
            while(count[i] > 0) {
                arr1[index++] = i
                count[i]--
            }
        }
        return arr1
    }
}
```

#### [076. 数组中的第 k 大的数字](https://leetcode.cn/problems/xx4gT2/)

![image-20221102145431779](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221102145431779-7462150.png)

使用快速排序的思想，利用partition子步骤。

```java
public class Solution076 {
    public int findKthLargest(int[] nums, int k) {
        int targetIndex = nums.length - k;
        int partitionIndex = -1;
        int start = 0;
        int end = nums.length - 1;
        while (partitionIndex != targetIndex) {
            if (partitionIndex < targetIndex) { //往右边部分继续寻找
                start = partitionIndex + 1;
            } else {  //往左边部分继续寻找
                end = partitionIndex - 1;
            }
            partitionIndex = partition(nums, start, end);
        }
        return nums[targetIndex];
    }

    private int partition(int[] nums, int start, int end) {
      	//概率随机性，但是删掉这一步程序也是正确的。
       	int randomIndex = new Random().nextInt(end - start + 1) + start;
        swap(nums, randomIndex, end);
      
        int midIndex = start - 1;
        for (int i = start; i < end; i++) {
            if (nums[i] < nums[end]) {
                midIndex++;
                swap(nums, midIndex, i);
            }
        }
        swap(nums, ++midIndex, end);
        return midIndex;
    }

    private void swap(int[] nums, int x, int y) {
        int temp = nums[x];
        nums[x] = nums[y];
        nums[y] = temp;
    }


    public static void main(String[] args) {
        int[] nums = {3,2,3,1,2,4,5,5,6};
        System.out.println(new Solution076().findKthLargest(nums, 4));
    }
}

```

+ 基于函数partition找出数组中第k大的数字的时间复杂度是O（n），空间复杂度是O（1）。
+ 假设函数partition每次选择的中间值都位于分区后的数组的中间位置，那么第1次函数partition需要扫描长度为n的数组，第2次需要扫描长度为n/2的子数组，第3次需要扫描长度为n/4的数组，重复这个过程，直到子数组的长度为1。由于n+n/2+n/4+…+1=2n，因此总的时间复杂度是O（n）。

#### [ 077. 链表排序](https://leetcode.cn/problems/7WHec2/)

![image-20221102203114119](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221102203114119-7462150.png)

归并排序思想，递归方式。

```java
class ListNode {
    int val;
    ListNode next;

    ListNode() {
    }

    ListNode(int val) {
        this.val = val;
    }

    ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }
}

public class Solution {
    public ListNode sortList(ListNode head) {
        if (head == null || head.next == null) { //先做一个判空，节省操作，同时也避免递归栈溢出。
            return head;
        }
        //归并排序 递归方式
        ListNode head1 = head;
        ListNode head2 = splitList(head);

        head1 = sortList(head1);
        head2 = sortList(head2);

        return merge(head1, head2);
    }

    private ListNode merge(ListNode head1, ListNode head2) {
        ListNode flag = new ListNode();
        ListNode cur = flag;
        while (head1 != null && head2 != null) {
            if (head1.val < head2.val) {
                cur.next = head1;
                head1 = head1.next;
            } else {
                cur.next = head2;
                head2 = head2.next;
            }
            cur = cur.next;
        }
        cur.next = (head1 == null) ? head2 : head1;
        return flag.next;
    }

    private ListNode splitList(ListNode head) {
        ListNode slow = head;
        ListNode fast = head.next;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        ListNode next = slow.next;
        slow.next = null; //切断链表

        return next; //返回的是基数中间位置下一个节点，或者偶数后半部分的第一个节点
    }

    public static void main(String[] args) {
        ListNode head = new ListNode(4);
        head.next = new ListNode(2);
        head.next.next = new ListNode(1);
        head.next.next.next = new ListNode(3);


        ListNode cur = new Solution().sortList(head);
        while (cur != null) {
            System.out.println(cur.val);
            cur = cur.next;
        }
    }
}

```

+ 时间复杂度是O（nlogn）。
+ 由于对链表进行归并排序不需要创建另外一个相同大小的链表来保存合并之后的节点，因此对链表进行归并排序的空间效率更高。由于代码存在递归调用，递归调用栈的深度为O（logn），因此空间复杂度为O（logn）。

```kotlin
class ListNode(var `val`: Int) {
    var next: ListNode? = null
}

class Solution {
    fun sortList(head: ListNode?): ListNode? {
        if (head?.next == null) {
            return head
        }
        //归并排序，递归方式
        var head1 = head
        var head2 = splitList(head)
        head1 = sortList(head1)
        head2 = sortList(head2)
        return mergeList(head1, head2)
    }

    private fun splitList(head: ListNode): ListNode? {
        var slow: ListNode? = head
        var fast: ListNode? = head.next
        while(fast?.next != null) {
            slow = slow?.next
            fast = fast.next?.next
        }
        val next = slow?.next
        slow?.next = null //切断链表
        return next
    }

    private fun mergeList(head1: ListNode?, head2: ListNode?): ListNode? {
        val flag = ListNode(0)
        var cur: ListNode? = flag
        var head1 = head1
        var head2 = head2
        while(head1 != null && head2 != null) {
            if (head1.`val` < head2.`val`) {
                cur?.next = head1
                head1 = head1.next
            } else {
                cur?.next = head2
                head2 = head2.next
            }
            cur = cur?.next
        }
        cur?.next = head1 ?: head2
        return flag.next
    }
}
```

归并排序迭代方式。

```java
package offerbook;

import java.util.Random;
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

class Solution {

    public ListNode sortList(ListNode head) {
        if (head == null) {
            return head;
        }
        /**
         * 归并排序思想，迭代方式
         */
        //首先获取链表长度
        int length = getLength(head);
        //引用已经排序好的链表
        ListNode flag = new ListNode(0, head);
        //从子长度1开始处理，下一遍处理整个链表时，字长度翻倍
        for(int subLength = 1; subLength < length; subLength <<= 1) {
            //辅助切割链表，并且记录剩余链表的起点, 注意不能指向方法入参head!因为每次循环都要对上一次处理好的链表继续处理
            ListNode cur = flag.next;
            ListNode flagCur = flag; //记录已经排序好的链表的最后一个节点
            while(cur != null) {
                //按子长度分割出两个链表
                ListNode head1 = cur;
                for (int i = 1; i < subLength && cur.next != null; i++) {
                    cur = cur.next;
                }
                ListNode head2 = cur.next;
                cur.next = null;

                cur = head2;
                for (int i = 1; i < subLength && cur != null && cur.next != null; i++) { //对cur本身判空
                    cur = cur.next;
                }

                ListNode remain = null;
                if (cur != null) {
                    remain = cur.next;
                    cur.next = null;
                }

                ListNode mergedList = mergeList(head1, head2);
                flagCur.next = mergedList;
                while(flagCur.next != null) {
                    flagCur = flagCur.next;
                }

                //记录下剩余未处理链表，下一次循环的时候处理
                cur = remain;
            }
        }
        return flag.next;
    }

    private int getLength(ListNode head) {
        int len = 0;
        while(head != null) {
            len++;
            head = head.next;
        }
        return len;
    }

    private ListNode mergeList(ListNode head1, ListNode head2) {
        ListNode flag = new ListNode();
        ListNode cur = flag;
        while(head1 != null && head2 != null) {
            if (head1.val < head2.val) {
                cur.next = head1;
                head1 = head1.next;
            } else {
                cur.next = head2;
                head2 = head2.next;
            }
            cur = cur.next;
        }
        cur.next = (head1 == null) ? head2 : head1;
        return flag.next;
    }


    public static void main(String[] args) {
        ListNode head = new ListNode(4);
        head.next = new ListNode(2);
        head.next.next = new ListNode(1);
        head.next.next.next = new ListNode(3);

        ListNode node = new Solution().sortList(head);
        while(node != null) {
            System.out.println(node.val);
            node = node.next;
        }
    }
}
```

+ 时间复杂度：O*(*n*log*n)，其中 n 是链表的长度。
+ 空间复杂度：O(1)。



#### [078. 合并排序链表](https://leetcode.cn/problems/vvXgSW/?favorite=e8X3pBZi)

![image-20221103155246206](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221103155246206-7462150.png)

![image-20221103155701939](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221103155701939.png)

利用小顶堆。

```java
class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        ListNode flag = new ListNode();
        ListNode flagCur = flag;

        //堆排序
        PriorityQueue<ListNode> priorityQueue = new PriorityQueue<>((n1, n2) -> n1.val - n2.val);
        for (ListNode list : lists) {
            if (list != null) {
                priorityQueue.offer(list);
            }
        }

        //按照升序取出节点
        while (!priorityQueue.isEmpty()) {
            ListNode minNode = priorityQueue.poll();
            flagCur.next = minNode;
            flagCur = minNode;

            if (minNode.next != null) {
                priorityQueue.offer(minNode.next);
            }
        }
        return flag.next;
    }

    public static void main(String[] args) {
        ListNode head = new ListNode(1);
        head.next = new ListNode(4);
        head.next.next = new ListNode(5);

        ListNode head1 = new ListNode(1);
        head1.next = new ListNode(3);
        head1.next.next = new ListNode(4);

        ListNode head2 = new ListNode(2);
        head2.next = new ListNode(6);

        ListNode[] listNodes = new ListNode[]{head, head1, head2};

        ListNode node = new Solution().mergeKLists(listNodes);
        ListNode cur = node;
        while(cur != null) {
            System.out.println(cur.val);
            cur = cur.next;
        }
    }
}
```

+ 假设k个排序链表总共有n个节点。如果堆的大小为k，那么空间复杂度就是O（k）。
+ 每次用最小堆处理一个节点需要O（logk）的时间，因此这种解法的时间复杂度是O（nlogk）。

```kotlin
class ListNode(var `val`: Int) {
    var next: ListNode? = null
}

class Solution {
    //一个public的函数，返回值类型不能是一个internal类，internal仅模块内可见，所以ListNode不能声明为internal
    fun mergeKLists(lists: Array<ListNode?>): ListNode? {
        val flag = ListNode(0)
        var flagCur: ListNode = flag

        //堆排序
        val priorityQueue = PriorityQueue { n1: ListNode, n2: ListNode -> n1.`val` - n2.`val` }
        for (node in lists) {
            node?.let {
                priorityQueue.offer(it)
            }
        }

        //按照升序取出节点
        while (!priorityQueue.isEmpty()) {
            val minNode = priorityQueue.poll()
            flagCur.next = minNode
            flagCur = minNode
            minNode.next?.let {
                priorityQueue.offer(it)
            }
        }
        return flag.next
    }
}
```

归并排序。

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists == null || lists.length == 0) {
            return null;
        }
        //归并排序
        return mergeSort(lists, 0, lists.length - 1);
    }
    private ListNode mergeSort(ListNode[] lists, int start, int end) {
        if (start == end) {
            return lists[start];
        }
        int mid = start + (end - start) / 2;
        ListNode head1 = mergeSort(lists, start, mid);
        ListNode head2 = mergeSort(lists, mid + 1, end);
        return mergeList(head1, head2);
    }

    private ListNode mergeList(ListNode head1, ListNode head2) {
        ListNode flag = new ListNode();
        ListNode cur = flag;
        while(head1 != null && head2 != null) {
            if (head1.val < head2.val) {
                cur.next = head1;
                head1 = head1.next;
            } else {
                cur.next = head2;
                head2 = head2.next;
            }
            cur = cur.next;
        }
        cur.next = (head1 == null) ? head2 : head1;
        return flag.next;
    }
}
```

+ 递归调用的深度是O（logk），每次需要合并n个节点，因此时间复杂度是O（nlogk）。
+ 它的空间复杂度是O（logk），用来维护递归调用栈。



## 回溯

### 集合的组合、排列

#### [079. 所有子集](https://leetcode.cn/problems/TVdhkn/?favorite=e8X3pBZi)

![image-20221103222024366](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221103222024366.png)

> **排列和组合的区别：排列与元素的顺序相关，子集（组合）和元素顺序无关。**

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        LinkedList<Integer> subList = new LinkedList<>();
        List<List<Integer>> resultList = new ArrayList<>();
        if (nums.length == 0) {
            return resultList;
        }
        findSets(nums, subList, resultList, 0);
        return resultList;
    }

    private void findSets(int[] nums, LinkedList<Integer> subList, List<List<Integer>> resultList, int start) {
        if (start == nums.length) {
            resultList.add(new ArrayList<>(subList));
            return;
        }
        //不加进子集
        findSets(nums, subList, resultList, start + 1);
        //加进子集
        subList.add(nums[start]);
        findSets(nums, subList, resultList, start + 1);

        //清除状态，便于回溯父节点
        subList.removeLast();
    }

    public static void main(String[] args) {
        int[] nums = {};
        List<List<Integer>> subsets = new Solution().subsets(nums);
        for (List<Integer> subset : subsets) {
            for (Integer integer : subset) {
                System.out.println(integer);
            }
            System.out.println("--------------------");
        }

    }
}
```

+ 如果输入的集合中有n个元素，由于每个元素都有2个选项，因此总的时间复杂度是O（2^n）。
+ 空间复杂度：O(n)。临时数组 t的空间代价是 O(n)，递归时栈空间的代价为 O(n)。

```kotlin
internal class Solution {
    fun subsets(nums: IntArray): List<List<Int>> {
        val resultList: MutableList<List<Int>> = LinkedList()
        if (nums.isEmpty()) {
            return resultList
        }
        findSets(nums, LinkedList<Int>(), resultList, 0)
        return resultList
    }

    private fun findSets(nums: IntArray, subList: LinkedList<Int>, resultList: MutableList<List<Int>>, start: Int) {
        if (start == nums.size) {
            resultList.add(LinkedList(subList))
            return
        }
        //不加进子集
        findSets(nums, subList, resultList, start + 1)
        //加进子集
        subList.add(nums[start])
        findSets(nums, subList, resultList, start + 1)

        //清除状态，便于回溯父节点
        subList.removeLast()
    }
}
```



####  [080. 含有 k 个元素的组合](https://leetcode.cn/problems/uUsW3B/?favorite=e8X3pBZi)

![image-20221103231857251](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221103231857251.png)

同样是回溯递归，注意边界就行。

```java
class Solution {
    public List<List<Integer>> combine(int n, int k) {
        List<List<Integer>> result = new LinkedList<>();
        findCombineList(k, new LinkedList<>(), result, n, 1);
        return result;
    }

    private void findCombineList(
            int k,
            LinkedList<Integer> subList,
            List<List<Integer>> result,
            int n,
            int start
    ) {
        if (start > n || subList.size() == k) {
            if (subList.size() == k) {
                result.add(new LinkedList<>(subList));
            }
        } else {
            findCombineList(k, subList, result, n, start + 1);

            subList.add(start);
            findCombineList(k, subList, result, n, start + 1);
            subList.removeLast();
        }
    }

    public static void main(String[] args) {
        List<List<Integer>> subsets = new Solution().combine(1, 1);
        for (List<Integer> subset : subsets) {
            for (Integer integer : subset) {
                System.out.println(integer);
            }
            System.out.println("--------------------");
        }

    }
}
```



#### [081. 允许重复选择元素的组合](https://leetcode.cn/problems/Ygoe9J/?favorite=e8X3pBZi)

![image-20221103234448664](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221103234448664.png)

![image-20221103234507693](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221103234507693.png)

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> result = new LinkedList();
        findCombinations(candidates, result, new LinkedList<Integer>(), target, 0);
        return result;
    }

    private void findCombinations(
        int[] candidates, 
        List<List<Integer>> result, 
        LinkedList<Integer> subCombination, 
        int target, 
        int index
    ) {
        if(target == 0) {
            result.add(new LinkedList(subCombination));
        } else if (target > 0 && index < candidates.length) {
            //选择不添加index当前元素
            findCombinations(candidates, result, subCombination, target, index + 1);

            //如果添加index当前元素，意味着此后也可以重复添加
            int num = candidates[index];
            subCombination.add(num);
            findCombinations(candidates, result, subCombination, target - num, index);
            subCombination.removeLast();
        }
    }
}
```

+ 空间复杂度：O(target)。除答案数组外，空间复杂度取决于递归的栈深度，在最差情况下需要递归 O(target) 层。

#### [082. 含有重复元素集合的组合](https://leetcode.cn/problems/4sjJUc/?favorite=e8X3pBZi)

![image-20221104143746721](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221104143746721.png)

![image-20221104143759880](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221104143759880.png)

```java
class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        //首先排序，便于跳过相同数字，避免重复组合出现，
        Arrays.sort(candidates);
        List<List<Integer>> result = new LinkedList<>();
        findCombinations(candidates, result, new LinkedList<Integer>(), target, 0);
        return result;
    }

    private void findCombinations(int[] candidates, List<List<Integer>> result, LinkedList<Integer> subCombination, int target, int index) {
        if (target == 0) {
            result.add(new LinkedList(subCombination));
        } else if(target > 0 && index < candidates.length){
            int num = candidates[index];

            //如果决定不添加这个数字，那么后续也要避开这个数字
            int nextIndex = getNextNumIndex(num, index, candidates);
            findCombinations(candidates, result, subCombination, target, nextIndex);

            subCombination.add(num);
            findCombinations(candidates, result, subCombination, target - num, index + 1);
            subCombination.removeLast();
        }
    }

    private int getNextNumIndex(int num, int index, int[] candidates) {
        int i = index  + 1;
        while(i < candidates.length && candidates[i] == num) {
            i++;
        }
        return i;
    }
}
```

#### [083. 没有重复元素集合的全排列](https://leetcode.cn/problems/VvJkup/?favorite=e8X3pBZi)

![image-20221104182129016](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221104182129016.png)

```java
//解法比较直接，且用了一个数组用于记录元素的选中状态。
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        LinkedList<List<Integer>> result = new LinkedList();
        LinkedList<Integer> subSequence = new LinkedList();
        //记录对应元素是否被选择过
        int[] chooseFlag = new int[nums.length];
        findPermute(nums, result, subSequence, chooseFlag);
        return result;
    }

    private void findPermute(int[] nums, LinkedList<List<Integer>> result, LinkedList<Integer> subSequence, int[] chooseFlag) {
        if(subSequence.size() == nums.length) {
            result.add(new LinkedList(subSequence));
        } else {
            for(int i = 0; i < nums.length; i++) {
                if (chooseFlag[i] == 0) {
                    subSequence.add(nums[i]);
                    chooseFlag[i] = 1;
                    findPermute(nums, result, subSequence, chooseFlag);
                    chooseFlag[i] = 0;
                    subSequence.removeLast();
                }
            }
        }
    }
}
```

上面的解法还可以优化。在原数组上进行操作，不需要chooseFlag了，选中的元素直接交换到前面处理完的区间中。

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        LinkedList<List<Integer>> result = new LinkedList();
        findPermute(nums, result, 0);
        return result;
    }

    private void findPermute(int[] nums, LinkedList<List<Integer>> result, int index) {
        if(index == nums.length) {
            LinkedList<Integer> subSequence = new LinkedList();
            for(int i = 0; i < nums.length; i++) {
                subSequence.add(nums[i]);
            }
            result.add(subSequence);
        } else {
            for(int i = index; i < nums.length; i++) {
                swap(nums, i, index);
                findPermute(nums, result, index + 1);
                swap(nums, i, index);
            }
        }
    }

    private void swap(int[] nums, int i, int j) {
        if(i != j) {
            int temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
        }
    }
}
```

+ 假设数组nums的长度为n，当i等于0时递归函数helper中的for循环执行n次，当i等于1时for循环执行n-1次，以此类推，当i等于n-1时，for循环执行1次。因此，全排列的时间复杂度是O（n！）。

#### [084. 含有重复元素集合的全排列](https://leetcode.cn/problems/7p8L0Z/?favorite=e8X3pBZi) 

![image-20221104191352913](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221104191352913.png)

```java
class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        List<List<Integer>> result = new LinkedList();
        helper(nums, result, 0);
        return result;
    }

    private void helper(int[] nums, List<List<Integer>> result, int index) {
        if(index == nums.length) {
            LinkedList<Integer> subList = new LinkedList();
            for(int num: nums) {
                subList.add(num);
            }
            result.add(subList);
        } else {
            Set<Integer> set = new HashSet<>(); ///此次循环中相同的数字，不重复处理。
            for (int i = index; i < nums.length; i++) {
                int num = nums[i];
                if (!set.contains(num)) {
                    set.add(num);
                    swap(nums, i, index);
                    helper(nums, result, index + 1);
                    swap(nums, i, index);
                }
            }
        }
    }

    private void swap(int[] nums, int i, int j) {
        if(i != j) {
            int temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
        }
    }
}
```

```kotlin
class Solution {
    fun permuteUnique(nums: IntArray): List<List<Int>> {
        val result: MutableList<List<Int>> = LinkedList()
        helper(nums, result, 0)
        return result
    }

    private fun helper(nums: IntArray, result: MutableList<List<Int>>, index: Int) {
        if (index == nums.size) {
            val subList = LinkedList<Int>()
            for(num in nums) {
                subList.add(num)
            }
            result.add(subList)
        } else {
            val set = HashSet<Int>()
            for(i in index until nums.size) {
                val num = nums[i]
                if (!set.contains(num)) {
                    set.add(num)
                    swap(nums, i, index)
                    helper(nums, result, index + 1)
                    swap(nums, i, index)
                }
            }
        }
    }

    private fun swap(nums: IntArray, i: Int, j: Int) {
        if(i != j) {
            val temp = nums[i]
            nums[i] = nums[j]
            nums[j] = temp
        }
    }
}
```



### 使用回溯法解决其他类型的问题

#### [085. 生成匹配的括号](https://leetcode.cn/problems/IDBivT/?favorite=e8X3pBZi)

![image-20221104211346491](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221104211346491.png)

```java
//每次添加括号都要保证左括号数量不能少于又括号数量，否则就是错误的匹配。
class Solution {
    public List<String> generateParenthesis(int n) {
        LinkedList<String> result = new LinkedList();
        LinkedList<Character> sub = new LinkedList();
        helper(result, sub, n, 0, 0);
        return result;
    }

    private void helper(LinkedList<String> result, LinkedList<Character> sub, int n, int left, int right) {
        if(sub.size() == n * 2) {
            StringBuilder subResult = new StringBuilder();
            for(Character c: sub) {
                subResult.append(c);
            }
            result.add(subResult.toString());
        } else {
            if (left < n) {
                sub.add('(');
                helper(result, sub, n, left + 1, right);
                sub.removeLast();
            }
            if(left > right) {
                sub.add(')');
                helper(result, sub, n, left, right + 1);
                sub.removeLast();
            }
        }
    }
}
```

上述代码还能精简下。

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        LinkedList<String> result = new LinkedList();
        helper(result, "", n, n);
        return result;
    }

    private void helper(LinkedList<String> result, String sub, int left, int right) {
        if(left == 0 && right == 0) {
            result.add(sub);
        } else {
            if (left <= right && left > 0) {
                helper(result, sub + "(", left - 1, right);
            }

            if(left < right) {
                helper(result, sub + ")", left, right - 1);
            }
        }
    }
}
```

#### [086. 分割回文子字符串](https://leetcode.cn/problems/M99OJA/?favorite=e8X3pBZi)

![image-20221104222954775](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221104222954775.png)

```java
class Solution {
    public String[][] partition(String s) {
        LinkedList<List<String>> result = new LinkedList();
        LinkedList<String> sub = new LinkedList();
        help(result, sub, s, 0);

        String[][] resultArray = new String[result.size()][];
        for (int i = 0; i < result.size(); i++) {
            int subLen = result.get(i).size();
            resultArray[i] = new String[subLen];
            for (int j = 0; j < subLen; j++) {
                resultArray[i][j] = result.get(i).get(j);
            }
        }
        return resultArray;
    }

    private void help(LinkedList<List<String>> result, LinkedList<String> sub, String s, int index) {
        if (index == s.length()) {
            result.add(new LinkedList(sub));
        } else {
            for(int i = index; i < s.length(); i++) {
                if(check(s, index, i)) {
                    sub.add(s.substring(index, i + 1)); //方法名substring没有大写字母
                    help(result, sub, s, i + 1);
                    sub.removeLast();
                }
            }
        }
    }

    private boolean check(String s, int start, int end) {
        while(start < end) {
            if (s.charAt(start++) != s.charAt(end--)) {
                return false;
            }
        }
        return true;
    }
}
```

#### [087. 复原 IP](https://leetcode.cn/problems/0on3uN/?favorite=e8X3pBZi) 

![image-20221104232346609](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221104232346609.png)

![image-20221104232444090](/Users/chenying/Documents/学习笔记/TyporaImages/image-20221104232444090.png)

```java
class Solution {
    public List<String> restoreIpAddresses(String s) {
        List<String> result = new LinkedList();
        help(s, result, "", 0, 0);
        return result;
    }

    private void help(String s, List<String> result, String ip, int seqCount, int index) {
        if (index == s.length() && seqCount == 4) {
            result.add(ip.substring(0, ip.length() - 1));
        } else {
            for (int i = index; i < s.length() && i < index + 3; i++) {
                //前导0不符合；>255不符合
                String sub = s.substring(index, i + 1);
                if (isValid(sub)) {
                    help(s, result, ip + sub + ".", seqCount + 1, i + 1);
                }
            }
        }
    }

    public boolean isValid(String seq) {
        //注意必须使用equals
        return (Integer.valueOf(seq) <= 255 && (seq.equals("0") || !seq.startsWith("0"))); 
    }
}
```

