[toc]







# Leetcode题解记录



















## 一、数组

### （1）[两数之和](https://leetcode-cn.com/problems/two-sum/)

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。



```java
public int[] twoSum(int[] nums, int target) {
        int[] result = null;

        Map<Integer, Integer> value2Index = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            if (value2Index.get(target-nums[i]) != null) {
                result = new int[]{i,value2Index.get(target-nums[i])};
                break;
            }
            value2Index.put(nums[i],i);
        }
        return result;
    }
```

![image-20210704222307795](../typora-user-images/image-20210704222307795.png)





### （2）[三数之和 ](https://leetcode-cn.com/problems/3sum/)

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有和为 0 且不重复的三元组。

注意：答案中不可以包含重复的三元组。

示例 1：

输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
示例 2：

输入：nums = []
输出：[]
示例 3：

输入：nums = [0]
输出：[]



```java
public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);

        List<List<Integer>> result = new ArrayList<>();

        int len = nums.length;

        if (len < 3) {
            return result;
        }

        for (int i = 0; i < len; i++) {
            if (nums[i]>0) {
                continue;
            }
            if(i > 0 && nums[i] == nums[i-1]) continue;
            int start = i+1;
            int end = len - 1;
            while (start<end) {
                int tem = nums[start] + nums[end];
                if (tem > -nums[i]) {
                    end--;
                } else if (tem < -nums[i]) {
                    start++;
                } else {
                    while (start<len-1 && nums[start]==nums[start+1]) start++;
                    while (end> start && nums[end]==nums[end-1]) end--;
                    ArrayList<Integer> subResult = new ArrayList<>();
                    subResult.add(nums[i]);
                    subResult.add(nums[start++]);
                    subResult.add(nums[end--]);
                        result.add(subResult);
                }
            }
        }

        return result;
    }
```











### （3）[多数元素](https://leetcode-cn.com/problems/majority-element/)

给定一个大小为 n 的数组，找到其中的多数元素。多数元素是指在数组中出现次数 大于 ⌊ n/2 ⌋ 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

**示例 1：**

```
输入：[3,2,3]
输出：3
```



1、哈希法

```java
public int majorityElement(int[] nums) {

        int len = nums.length;
        int moreLen = len / 2 ;
        int result = nums[0];

        Map<Integer, Integer> tmpMap = new HashMap<>();

        for (int i = 0; i < len; i++) {
            if (tmpMap.get(nums[i]) != null) {
                tmpMap.put(nums[i], tmpMap.get(nums[i])+ 1 );
            } else {
                tmpMap.put(nums[i],1);
            }
            if (tmpMap.get(nums[i])>moreLen) {
                result = nums[i];
                break;
            }

        }

        return result;
    }
```



2、排序法

如果将数组 nums 中的所有元素按照单调递增或单调递减的顺序排序，那么下标为n/2 的元素（下标从 0 开始）一定是众数。

```java
public int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length / 2];
    }
```



3、摩尔投票法

候选人(cand_num)初始化为nums[0]，票数count初始化为1。
当遇到与cand_num相同的数，则票数count = count + 1，否则票数count = count - 1。
当票数count为0时，更换候选人，并将票数count重置为1。
遍历完数组后，cand_num即为最终答案。

为何这行得通呢？
投票法是遇到相同的则票数 + 1，遇到不同的则票数 - 1。
且“多数元素”的个数> ⌊ n/2 ⌋，其余元素的个数总和<= ⌊ n/2 ⌋。
因此“多数元素”的个数 - 其余元素的个数总和 的结果 肯定 >= 1。
这就相当于每个“多数元素”和其他元素 两两相互抵消，抵消到最后肯定还剩余至少1个“多数元素”。

无论数组是1 2 1 2 1，亦或是1 2 2 1 1，总能得到正确的候选人。

```java
class Solution {
    public int majorityElement(int[] nums) {
        int cand_num = nums[0], count = 1;
        for (int i = 1; i < nums.length; ++i) {
            if (cand_num == nums[i])
                ++count;
            else if (--count == 0) {
                cand_num = nums[i];
                count = 1;
            }
        }
        return cand_num;
    }
}
```













### （4）[求缺失的第一个正数](https://leetcode-cn.com/problems/first-missing-positive/)

### （5）[把数组中的 0 移到末尾](github/Leetcode 题解 数组与矩阵.md#1-把数组中的-0-移到末尾)

### （6）[改变矩阵维度](github/Leetcode 题解 数组与矩阵.md#2-改变矩阵维度)

### （7）[找出数组中最长的连续 1](github/Leetcode 题解 数组与矩阵.md#3-找出数组中最长的连续-1)

### （8）[有序矩阵查找](./github/Leetcode 题解 数组与矩阵.md#4-有序矩阵查找)

### （9）[有序矩阵的 Kth Element](github/Leetcode 题解 数组与矩阵.md#5-有序矩阵的-kth-element)

### （10）[一个数组元素在 [1, n\] 之间，其中一个数被替换为另一个数，找出重复的数和丢失的数](github/Leetcode 题解 数组与矩阵.md#6-一个数组元素在-[1-n]-之间，其中一个数被替换为另一个数，找出重复的数和丢失的数)

### （11）[找出数组中重复的数，数组值在 [1, n\] 之间](github/Leetcode 题解 数组与矩阵.md#7-找出数组中重复的数，数组值在-[1-n]-之间)

### （12）[数组相邻差值的个数](github/Leetcode 题解 数组与矩阵.md#8-数组相邻差值的个数)

### （13）[数组的度](github/Leetcode 题解 数组与矩阵.md#9-数组的度)

### （14）[对角元素相等的矩阵](github/Leetcode 题解 数组与矩阵.md#10-对角元素相等的矩阵)

### （15）[嵌套数组](github/Leetcode 题解 数组与矩阵.md#11-嵌套数组)

### （16）[分隔数组](github/Leetcode 题解 数组与矩阵.md#12-分隔数组)









## 二、链表



### （1）[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

### （2）[合并 k 个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

### （3）[单链表反转]()

### （4）[求链表的中间结点]()

### （5）[找出两个链表的交点](github/Leetcode 题解 链表.md#1-找出两个链表的交点)

### （6）[链表反转](github/Leetcode 题解 链表.md#2-链表反转)

### （7）[归并两个有序的链表](github/Leetcode 题解 链表.md#3-归并两个有序的链表)

### （8）[从有序链表中删除重复节点](github/Leetcode 题解 链表.md#4-从有序链表中删除重复节点)

### （9）[删除链表的倒数第 n 个节点](github/Leetcode 题解 链表.md#5-删除链表的倒数第-n-个节点)

### （10）[ 交换链表中的相邻结点](github/Leetcode 题解 链表.md#6-交换链表中的相邻结点)

### （11）[ 链表求和](github/Leetcode 题解 链表.md#7-链表求和)

### （12）[ 回文链表](github/Leetcode 题解 链表.md#8-回文链表)

### （13）[ 分隔链表](github/Leetcode 题解 链表.md#9-分隔链表)

### （14）[ 链表元素按奇偶聚集](github/Leetcode 题解 链表.md#10-链表元素按奇偶聚集)









## 三、栈

### （1）[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

### （2）[最长有效的括号](https://leetcode-cn.com/problems/longest-valid-parentheses/)

### （3）[逆波兰表达式求值](https://leetcode-cn.com/problems/evaluate-reverse-polish-notation/)

### （4）[用栈实现队列](github/Leetcode 题解 栈和队列.md#1-用栈实现队列)

### （5）[用队列实现栈](github/Leetcode 题解 栈和队列.md#2-用队列实现栈)

### （6）[最小值栈](github/Leetcode 题解 栈和队列.md#3-最小值栈)

### （7）[用栈实现括号匹配](github/Leetcode 题解 栈和队列.md#4-用栈实现括号匹配)

### （8）[数组中元素与下一个比它大的元素之间的距离](github/Leetcode 题解 栈和队列.md#5-数组中元素与下一个比它大的元素之间的距离)

### （9）[循环数组中比当前元素大的下一个元素](github/Leetcode 题解 栈和队列.md#6-循环数组中比当前元素大的下一个元素)





## 四、队列



### （1）[设计一个双端队列](https://leetcode-cn.com/problems/design-circular-deque/)

### （2）[滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)





## 五、哈希表

### （1）[数组中两个数的和为给定值](github/Leetcode 题解 哈希表.md#1-数组中两个数的和为给定值)

### （2）[判断数组是否含有重复元素](github/Leetcode 题解 哈希表.md#2-判断数组是否含有重复元素)

### （3）[最长和谐序列](github/Leetcode 题解 哈希表.md#3-最长和谐序列)

### （4）[最长连续序列](github/Leetcode 题解 哈希表.md#4-最长连续序列)



## 六、递归

### （1）[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)





## 七、排序

### （1）归并排序

### （2）快速排序

### （3）插入排序

### （4）冒泡排序

### （5）选择排序

### （6）O(n)时间复杂度内找到一组数据的第K大元素

### （7）堆排序

	-  [Kth Element](github/Leetcode 题解 排序.md#1-kth-element)

### （8）桶排序

-  [出现频率最多的 k 个元素](github/Leetcode 题解 排序.md#1-出现频率最多的-k-个元素)
- [按照字符出现次数对字符串排序](github/Leetcode 题解 排序.md#2-按照字符出现次数对字符串排序)

### （9） [ 按颜色进行排序](github/Leetcode 题解 排序.md#1-按颜色进行排序)







## 八、二分查找



### （1）实现一个有序数组的二分查找算法

### （2）模糊二分查找算法（比如大于等于给定值的第一个元素）

### （3）[x的平方根](https://leetcode-cn.com/problems/sqrtx/)

### （4）[大于给定元素的最小元素](github/Leetcode 题解 二分查找.md#2-大于给定元素的最小元素)

### （5）[有序数组的 Single Element](github/Leetcode 题解 二分查找.md#3-有序数组的-single-element)

### （6）[第一个错误的版本](github/Leetcode 题解 二分查找.md#4-第一个错误的版本)

### （7）[旋转数组的最小数字](github/Leetcode 题解 二分查找.md#5-旋转数组的最小数字)

### （8）[查找区间](github/Leetcode 题解 二分查找.md#6-查找区间)



## 九、散列表

### （1）实现一个基于链表法解决冲突问题的散列表

### （2）LRU 缓存淘汰算法







## 十、字符串

### （1）实现一个字符集，只包含 a～z 这 26 个英文字母的 Trie 树

### （2）实现朴素的字符串匹配算法

### （3）[反转字符串](https://leetcode-cn.com/problems/reverse-string/)

### （4）[翻转字符串里的单词](https://leetcode-cn.com/problems/reverse-words-in-a-string/)

### （5）[字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi/)

### （6）[字符串循环移位包含](github/Leetcode 题解 字符串.md#1-字符串循环移位包含)

### （7）[字符串循环移位](github/Leetcode 题解 字符串.md#2-字符串循环移位)

### （8）[字符串中单词的翻转](github/Leetcode 题解 字符串.md#3-字符串中单词的翻转)

### （9）[两个字符串包含的字符是否完全相同](github/Leetcode 题解 字符串.md#4-两个字符串包含的字符是否完全相同)

### （10）[计算一组字符集合可以组成的回文字符串的最大长度](github/Leetcode 题解 字符串.md#5-计算一组字符集合可以组成的回文字符串的最大长度)

### （11）[字符串同构](github/Leetcode 题解 字符串.md#6-字符串同构)

### （12）[回文子字符串个数](github/Leetcode 题解 字符串.md#7-回文子字符串个数)

### （13）[判断一个整数是否是回文数](github/Leetcode 题解 字符串.md#8-判断一个整数是否是回文数)

### （14）[统计二进制字符串中连续 1 和连续 0 数量相同的子字符串个数](github/Leetcode 题解 字符串.md#9-统计二进制字符串中连续-1-和连续-0-数量相同的子字符串个数)





## 十一、位运算

### （1）[统计两个数的二进制表示有多少位不同](github/Leetcode 题解 位运算.md#1-统计两个数的二进制表示有多少位不同)

### （2）[数组中唯一一个不重复的元素](github/Leetcode 题解 位运算.md#2-数组中唯一一个不重复的元素)

### （3）[找出数组中缺失的那个数](github/Leetcode 题解 位运算.md#3-找出数组中缺失的那个数)

### （4）[数组中不重复的两个元素](github/Leetcode 题解 位运算.md#4-数组中不重复的两个元素)

### （5）[翻转一个数的比特位](github/Leetcode 题解 位运算.md#5-翻转一个数的比特位)

### （6）[不用额外变量交换两个整数](github/Leetcode 题解 位运算.md#6-不用额外变量交换两个整数)

### （7）[判断一个数是不是 2 的 n 次方](github/Leetcode 题解 位运算.md#7-判断一个数是不是-2-的-n-次方)

### （8）[判断一个数是不是 4 的 n 次方](github/Leetcode 题解 位运算.md#8--判断一个数是不是-4-的-n-次方)

### （9）[判断一个数的位级表示是否不会出现连续的 0 和 1](github/Leetcode 题解 位运算.md#9-判断一个数的位级表示是否不会出现连续的-0-和-1)

### （10）[求一个数的补码](github/Leetcode 题解 位运算.md#10-求一个数的补码)

### （11）[实现整数的加法](github/Leetcode 题解 位运算.md#11-实现整数的加法)

### （12）[字符串数组最大乘积](github/Leetcode 题解 位运算.md#12-字符串数组最大乘积)

### （13）[统计从 0 ~ n 每个数的二进制表示中 1 的个数](github/Leetcode 题解 位运算.md#13-统计从-0-~-n-每个数的二进制表示中-1-的个数)





## 十二、双指针

### （1）[有序数组的 Two Sum](github/Leetcode 题解 双指针.md#1-有序数组的-two-sum)

### （2）[两数平方和](github/Leetcode 题解 双指针.md#2-两数平方和)

### （3）[反转字符串中的元音字符](github/Leetcode 题解 双指针.md#3-反转字符串中的元音字符)

### （4）[回文字符串](github/Leetcode 题解 双指针.md#4-回文字符串)

### （5）[归并两个有序数组](github/Leetcode 题解 双指针.md#5-归并两个有序数组)

### （6）[判断链表是否存在环](github/Leetcode 题解 双指针.md#6-判断链表是否存在环)

### （7）[最长子序列](github/Leetcode 题解 双指针.md#7-最长子序列)



## 十三、树

### （1）实现一个二叉查找树，并且支持插入、删除、查找操作

### （2）实现查找二叉查找树中某个节点的后继、前驱节点

### （3）实现二叉树前、中、后序以及按层遍历

### （4）[翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

### （5）[二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

### （6）[验证二叉查找树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

### （7）[路径总和](https://leetcode-cn.com/problems/path-sum/)

### （8）递归

  [1. 树的高度](github/Leetcode 题解 树.md#1-树的高度)
  [2. 平衡树](github/Leetcode 题解 树.md#2-平衡树)
  [3. 两节点的最长路径](github/Leetcode 题解 树.md#3-两节点的最长路径)
  [4. 翻转树](github/Leetcode 题解 树.md#4-翻转树)
  [5. 归并两棵树](github/Leetcode 题解 树.md#5-归并两棵树)
  [6. 判断路径和是否等于一个数](github/Leetcode 题解 树.md#6-判断路径和是否等于一个数)
  [7. 统计路径和等于一个数的路径数量](github/Leetcode 题解 树.md#7-统计路径和等于一个数的路径数量)
  [8. 子树](github/Leetcode 题解 树.md#8-子树)
  [9. 树的对称](github/Leetcode 题解 树.md#9-树的对称)
  [10. 最小路径](github/Leetcode 题解 树.md#10-最小路径)
  [11. 统计左叶子节点的和](github/Leetcode 题解 树.md#11-统计左叶子节点的和)
  [12. 相同节点值的最大路径长度](github/Leetcode 题解 树.md#12-相同节点值的最大路径长度)
  [13. 间隔遍历](github/Leetcode 题解 树.md#13-间隔遍历)
  [14. 找出二叉树中第二小的节点](github/Leetcode 题解 树.md#14-找出二叉树中第二小的节点)

### （9）层次遍历

  [1. 一棵树每层节点的平均数](github/Leetcode 题解 树.md#1-一棵树每层节点的平均数)
  [2. 得到左下角的节点](github/Leetcode 题解 树.md#2-得到左下角的节点)

### （10）前中后序遍历

  [1. 非递归实现二叉树的前序遍历](github/Leetcode 题解 树.md#1-非递归实现二叉树的前序遍历)
  [2. 非递归实现二叉树的后序遍历](github/Leetcode 题解 树.md#2-非递归实现二叉树的后序遍历)
  [3. 非递归实现二叉树的中序遍历](github/Leetcode 题解 树.md#3-非递归实现二叉树的中序遍历)

### （11）BST

  [1. 修剪二叉查找树](github/Leetcode 题解 树.md#1-修剪二叉查找树)
  [2. 寻找二叉查找树的第 k 个元素](github/Leetcode 题解 树.md#2-寻找二叉查找树的第-k-个元素)
  [3. 把二叉查找树每个节点的值都加上比它大的节点的值](github/Leetcode 题解 树.md#3-把二叉查找树每个节点的值都加上比它大的节点的值)
  [4. 二叉查找树的最近公共祖先](github/Leetcode 题解 树.md#4-二叉查找树的最近公共祖先)
  [5. 二叉树的最近公共祖先](github/Leetcode 题解 树.md#5-二叉树的最近公共祖先)
  [6. 从有序数组中构造二叉查找树](github/Leetcode 题解 树.md#6-从有序数组中构造二叉查找树)
  [7. 根据有序链表构造平衡的二叉查找树](github/Leetcode 题解 树.md#7-根据有序链表构造平衡的二叉查找树)
  [8. 在二叉查找树中寻找两个节点，使它们的和为一个给定值](github/Leetcode 题解 树.md#8-在二叉查找树中寻找两个节点，使它们的和为一个给定值)
  [9. 在二叉查找树中查找两个节点之差的最小绝对值](github/Leetcode 题解 树.md#9-在二叉查找树中查找两个节点之差的最小绝对值)
  [10. 寻找二叉查找树中出现次数最多的值](github/Leetcode 题解 树.md#10-寻找二叉查找树中出现次数最多的值)

### （12）Trie

  [1. 实现一个 Trie](github/Leetcode 题解 树.md#1-实现一个-trie)
  [2. 实现一个 Trie，用来求前缀和](github/Leetcode 题解 树.md#2-实现一个-trie，用来求前缀和)



## 十四、堆

### （1）实现一个小顶堆、大顶堆、优先级队列

### （2）实现堆排序

### （3）利用优先级队列合并 K 个有序数组

### （4）求一组动态数据集合的最大 Top K



## 十五、图

### （1）实现有向图、无向图、有权图、无权图的邻接矩阵和邻接表表示方法

### （2）实现图的深度优先搜索、广度优先搜索

### （3）实现 Dijkstra 算法、A* 算法

### （4）实现拓扑排序的 Kahn 算法、DFS 算法

### （5）[岛屿的个数](https://leetcode-cn.com/problems/number-of-islands/description/)

### （6）[有效的数独](https://leetcode-cn.com/problems/valid-sudoku/)

### （7）二分图

  [1. 判断是否为二分图](github/Leetcode 题解 图.md#1-判断是否为二分图)

### （8）拓扑排序

  [1. 课程安排的合法性](github/Leetcode 题解 图.md#1-课程安排的合法性)
  [2. 课程安排的顺序](github/Leetcode 题解 图.md#2-课程安排的顺序)

### （9）并查集

  [1. 冗余连接](github/Leetcode 题解 图.md#1-冗余连接)





## 十六、贪心

### （1）[分配饼干](github/Leetcode 题解 贪心思想.md#1-分配饼干)

### （2）[不重叠的区间个数](github/Leetcode 题解 贪心思想.md#2-不重叠的区间个数)

### （3）[投飞镖刺破气球](github/Leetcode 题解 贪心思想.md#3-投飞镖刺破气球)

### （4）[根据身高和序号重组队列](github/Leetcode 题解 贪心思想.md#4-根据身高和序号重组队列)

### （5）[买卖股票最大的收益](github/Leetcode 题解 贪心思想.md#5-买卖股票最大的收益)

### （6）[买卖股票的最大收益 II](github/Leetcode 题解 贪心思想.md#6-买卖股票的最大收益-ii)

### （7）[种植花朵](github/Leetcode 题解 贪心思想.md#7-种植花朵)

### （8）[判断是否为子序列](github/Leetcode 题解 贪心思想.md#8-判断是否为子序列)

### （9）[修改一个数成为非递减数组](github/Leetcode 题解 贪心思想.md#9-修改一个数成为非递减数组)

### （10）[子数组最大的和](github/Leetcode 题解 贪心思想.md#10-子数组最大的和)

### （11）[分隔字符串使同种字符出现在一起](github/Leetcode 题解 贪心思想.md#11-分隔字符串使同种字符出现在一起)



## 十七、分治

### （1）利用分治算法求一组数据的逆序对个数

### （2）[给表达式加括号](github/Leetcode 题解 分治.md#1-给表达式加括号)

### （3）[不同的二叉搜索树](github/Leetcode 题解 分治.md#2-不同的二叉搜索树)



## 十八、回溯

### （1）利用回溯算法求解八皇后问题

### （2）利用回溯算法求解 0-1 背包问题

### （3）[正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)





## 十九、动态规划

### （1）斐波那契数列

  [1. 爬楼梯](github/Leetcode 题解 动态规划.md#1-爬楼梯)
  [2. 强盗抢劫](github/Leetcode 题解 动态规划.md#2-强盗抢劫)
  [3. 强盗在环形街区抢劫](github/Leetcode 题解 动态规划.md#3-强盗在环形街区抢劫)
  [4. 信件错排](github/Leetcode 题解 动态规划.md#4-信件错排)
  [5. 母牛生产](github/Leetcode 题解 动态规划.md#5-母牛生产)

### （2）矩阵路径

  [1. 矩阵的最小路径和](github/Leetcode 题解 动态规划.md#1-矩阵的最小路径和)
  [2. 矩阵的总路径数](github/Leetcode 题解 动态规划.md#2-矩阵的总路径数)

### （3）数组区间

  [1. 数组区间和](github/Leetcode 题解 动态规划.md#1-数组区间和)
  [2. 数组中等差递增子区间的个数](github/Leetcode 题解 动态规划.md#2-数组中等差递增子区间的个数)

### （4）分割整数

  [1. 分割整数的最大乘积](github/Leetcode 题解 动态规划.md#1-分割整数的最大乘积)
  [2. 按平方数来分割整数](github/Leetcode 题解 动态规划.md#2-按平方数来分割整数)
  [3. 分割整数构成字母字符串](github/Leetcode 题解 动态规划.md#3-分割整数构成字母字符串)

### （5）最长递增子序列

  [1. 最长递增子序列](github/Leetcode 题解 动态规划.md#1-最长递增子序列)
  [2. 一组整数对能够构成的最长链](github/Leetcode 题解 动态规划.md#2-一组整数对能够构成的最长链)
  [3. 最长摆动子序列](github/Leetcode 题解 动态规划.md#3-最长摆动子序列)

### （6）最长公共子序列

  [1. 最长公共子序列](github/Leetcode 题解 动态规划.md#1-最长公共子序列)

### （7）0-1 背包

  [1. 划分数组为和相等的两部分](github/Leetcode 题解 动态规划.md#1-划分数组为和相等的两部分)
  [2. 改变一组数的正负号使得它们的和为一给定数](github/Leetcode 题解 动态规划.md#2-改变一组数的正负号使得它们的和为一给定数)
  [3. 01 字符构成最多的字符串](github/Leetcode 题解 动态规划.md#3-01-字符构成最多的字符串)
  [4. 找零钱的最少硬币数](github/Leetcode 题解 动态规划.md#4-找零钱的最少硬币数)
  [5. 找零钱的硬币数组合](github/Leetcode 题解 动态规划.md#5-找零钱的硬币数组合)
  [6. 字符串按单词列表分割](github/Leetcode 题解 动态规划.md#6-字符串按单词列表分割)
  [7. 组合总和](github/Leetcode 题解 动态规划.md#7-组合总和)

### （8）股票交易

  [1. 需要冷却期的股票交易](github/Leetcode 题解 动态规划.md#1-需要冷却期的股票交易)
  [2. 需要交易费用的股票交易](github/Leetcode 题解 动态规划.md#2-需要交易费用的股票交易)
  [3. 只能进行两次的股票交易](github/Leetcode 题解 动态规划.md#3-只能进行两次的股票交易)
  [4. 只能进行 k 次的股票交易](github/Leetcode 题解 动态规划.md#4-只能进行-k-次的股票交易)

### （9）字符串编辑

  [1. 删除两个字符串的字符使它们相等](github/Leetcode 题解 动态规划.md#1-删除两个字符串的字符使它们相等)
  [2. 编辑距离](github/Leetcode 题解 动态规划.md#2-编辑距离)
  [3. 复制粘贴字符](github/Leetcode 题解 动态规划.md#3-复制粘贴字符)



## 二十、搜索

### （1）BFS

  [1. 计算在网格中从原点到特定点的最短路径长度](github/Leetcode 题解 搜索.md#1-计算在网格中从原点到特定点的最短路径长度)
  [2. 组成整数的最小平方数数量](github/Leetcode 题解 搜索.md#2-组成整数的最小平方数数量)
  [3. 最短单词路径](github/Leetcode 题解 搜索.md#3-最短单词路径)

### （2）DFS

  [1. 查找最大的连通面积](github/Leetcode 题解 搜索.md#1-查找最大的连通面积)
  [2. 矩阵中的连通分量数目](github/Leetcode 题解 搜索.md#2-矩阵中的连通分量数目)
  [3. 好友关系的连通分量数目](github/Leetcode 题解 搜索.md#3-好友关系的连通分量数目)
  [4. 填充封闭区域](github/Leetcode 题解 搜索.md#4-填充封闭区域)
  [5. 能到达的太平洋和大西洋的区域](github/Leetcode 题解 搜索.md#5-能到达的太平洋和大西洋的区域)

### （3）Backtracking

  [1. 数字键盘组合](github/Leetcode 题解 搜索.md#1-数字键盘组合)
  [2. IP 地址划分](github/Leetcode 题解 搜索.md#2-ip-地址划分)
  [3. 在矩阵中寻找字符串](github/Leetcode 题解 搜索.md#3-在矩阵中寻找字符串)
  [4. 输出二叉树中所有从根到叶子的路径](github/Leetcode 题解 搜索.md#4-输出二叉树中所有从根到叶子的路径)
  [5. 排列](github/Leetcode 题解 搜索.md#5-排列)
  [6. 含有相同元素求排列](github/Leetcode 题解 搜索.md#6-含有相同元素求排列)
  [7. 组合](github/Leetcode 题解 搜索.md#7-组合)
  [8. 组合求和](github/Leetcode 题解 搜索.md#8-组合求和)
  [9. 含有相同元素的组合求和](github/Leetcode 题解 搜索.md#9-含有相同元素的组合求和)
  [10. 1-9 数字的组合求和](github/Leetcode 题解 搜索.md#10-1-9-数字的组合求和)
  [11. 子集](github/Leetcode 题解 搜索.md#11-子集)
  [12. 含有相同元素求子集](github/Leetcode 题解 搜索.md#12-含有相同元素求子集)
  [13. 分割字符串使得每个部分都是回文数](github/Leetcode 题解 搜索.md#13-分割字符串使得每个部分都是回文数)
  [14. 数独](github/Leetcode 题解 搜索.md#14-数独)
  [15. N 皇后](github/Leetcode 题解 搜索.md#15-n-皇后)





## 二一、数学

### （1）[素数分解](github/Leetcode 题解 数学.md#素数分解)

### （2）[整除](github/Leetcode 题解 数学.md#整除)

### （3）最大公约数最小公倍数

  [1. 生成素数序列](github/Leetcode 题解 数学.md#1-生成素数序列)
  [2. 最大公约数](github/Leetcode 题解 数学.md#2-最大公约数)
  [3. 使用位操作和减法求解最大公约数](github/Leetcode 题解 数学.md#3-使用位操作和减法求解最大公约数)

### （4）进制转换

  [1. 7 进制](github/Leetcode 题解 数学.md#1-7-进制)
  [2. 16 进制](github/Leetcode 题解 数学.md#2-16-进制)
  [3. 26 进制](github/Leetcode 题解 数学.md#3-26-进制)

### （5）阶乘

  [1. 统计阶乘尾部有多少个 0](github/Leetcode 题解 数学.md#1-统计阶乘尾部有多少个-0)

### （6）字符串加法减法

  [1. 二进制加法](github/Leetcode 题解 数学.md#1-二进制加法)
  [2. 字符串加法](github/Leetcode 题解 数学.md#2-字符串加法)

### （7）相遇问题

  [1. 改变数组元素使所有的数组元素都相等](github/Leetcode 题解 数学.md#1-改变数组元素使所有的数组元素都相等)

### （8）多数投票问题

  [1. 数组中出现次数多于 n / 2 的元素](github/Leetcode 题解 数学.md#1-数组中出现次数多于-n--2-的元素)

### （9）其它

  [1. 平方数](github/Leetcode 题解 数学.md#1-平方数)
  [2. 3 的 n 次方](github/Leetcode 题解 数学.md#2-3-的-n-次方)
  [3. 乘积数组](github/Leetcode 题解 数学.md#3-乘积数组)
  [4. 找出数组中的乘积最大的三个数](github/Leetcode 题解 数学.md#4-找出数组中的乘积最大的三个数)