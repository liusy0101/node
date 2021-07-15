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

给你一个未排序的整数数组 nums ，请你找出其中没有出现的最小的正整数。

请你实现时间复杂度为 O(n) 并且只使用常数级别额外空间的解决方案。


示例 1：

输入：nums = [1,2,0]
输出：3
示例 2：

输入：nums = [3,4,-1,1]
输出：2
示例 3：

输入：nums = [7,8,9,11,12]
输出：1

**提示：**

- `1 <= nums.length <= 5 * 10^5`
- `-231 <= nums[i] <= 231 - 1`



（1）布隆过滤器

既然数组长度最长是5*10^5，那么直接定义一个char数组，长度就是5\*10^5

```java
public static int firstMissingPositive(int[] nums) {
        int maxVAlue = nums[0];
        char[] blChar = new char[500000];
        for (int i = 0; i < nums.length; i++) {
            maxVAlue = Math.max(nums[i],maxVAlue);
            if (nums[i]<=0 || nums[i]>=500000) {
                continue;
            }
            blChar[nums[i]-1] = 1;
        }
        if (maxVAlue<=0) {
            return 1;
        }
        for (int i = 0; i < blChar.length; i++) {
            if (blChar[i] != 1) {
                return i+1;
            }
        }
        return maxVAlue+1;
    }
```

![image-20210705230106645](../typora-user-images/image-20210705230106645.png)



（2）原地哈希

将数组视为哈希表

- 由于题目要求我们「只能使用常数级别的空间」，而要找的数一定在 [1, N + 1] 左闭右闭（这里 N 是数组的长度）这个区间里。因此，我们可以就把原始的数组当做哈希表来使用。事实上，哈希表其实本身也是一个数组；
  我们要找的数就在 [1, N + 1] 里，最后 N + 1 这个元素我们不用找。因为在前面的 N 个元素都找不到的情况下，我们才返回 N + 1；
- 那么，我们可以采取这样的思路：就把 11 这个数放到下标为 00 的位置， 22 这个数放到下标为 11 的位置，按照这种思路整理一遍数组。然后我们再遍历一次数组，第 11 个遇到的它的值不等于下标的那个数，就是我们要找的缺失的第一个正数。
- 这个思想就相当于我们自己编写哈希函数，这个哈希函数的规则特别简单，那就是数值为 i 的数映射到下标为 i - 1 的位置。

![image-20210705225852680](../typora-user-images/image-20210705225852680.png)

```java
public class Solution {

    public int firstMissingPositive(int[] nums) {
        int len = nums.length;

        for (int i = 0; i < len; i++) {
            while (nums[i] > 0 && nums[i] <= len && nums[nums[i] - 1] != nums[i]) {
                // 满足在指定范围内、并且没有放在正确的位置上，才交换
                // 例如：数值 3 应该放在索引 2 的位置上
                swap(nums, nums[i] - 1, i);
            }
        }

        // [1, -1, 3, 4]
        for (int i = 0; i < len; i++) {
            if (nums[i] != i + 1) {
                return i + 1;
            }
        }
        // 都正确则返回数组长度 + 1
        return len + 1;
    }

    private void swap(int[] nums, int index1, int index2) {
        int temp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = temp;
    }
}
```

![image-20210705230009116](../typora-user-images/image-20210705230009116.png)



### （5）[把数组中的 0 移到末尾](https://leetcode-cn.com/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**示例:**

```
输入: [0,1,0,3,12]
输出: [1,3,12,0,0]
```

```java
public void moveZeroes(int[] nums) {
        int len = nums.length;

        for (int i = 0; i < len; i++) {
            int curNum = nums[i];
            if (curNum != 0) {
                continue;
            }
            for (int j = i+1; j < len; j++) {
                if (nums[j] != 0) {
                    int tem = nums[j];
                    nums[j] = curNum;
                    nums[i] = tem;
                    break;
                }
            }
        }   
    }
```

![image-20210705232202254](../typora-user-images/image-20210705232202254.png)





```java
public void moveZeroes(int[] nums) {

        if(nums==null) {
			return;
		}
		//第一次遍历的时候，j指针记录非0的个数，只要是非0的统统都赋给nums[j]
		int j = 0;
		for(int i=0;i<nums.length;++i) {
			if(nums[i]!=0) {
				nums[j++] = nums[i];
			}
		}
		//非0元素统计完了，剩下的都是0了
		//所以第二次遍历把末尾的元素都赋为0即可
		for(int i=j;i<nums.length;++i) {
			nums[i] = 0;
		}

    }
```

![image-20210705233647646](../typora-user-images/image-20210705233647646.png)



**一次遍历：**

这里参考了快速排序的思想，快速排序首先要确定一个待分割的元素做中间点x，然后把所有小于等于x的元素放到x的左边，大于x的元素放到其右边。
这里我们可以用0当做这个中间点，把不等于0(注意题目没说不能有负数)的放到中间点的左边，等于0的放到其右边。
这的中间点就是0本身，所以实现起来比快速排序简单很多，我们使用两个指针i和j，只要nums[i]!=0，我们就交换nums[i]和nums[j]

```java
public void moveZeroes(int[] nums) {
		if(nums==null) {
			return;
		}
		//两个指针i和j
		int j = 0;
		for(int i=0;i<nums.length;i++) {
			//当前元素!=0，就把其交换到左边，等于0的交换到右边
			if(nums[i]!=0) {
				int tmp = nums[i];
				nums[i] = nums[j];
				nums[j++] = tmp;
			}
		}
	}
```









### （6）[改变矩阵维度](https://leetcode-cn.com/problems/reshape-the-matrix/)

在MATLAB中，有一个非常有用的函数 reshape，它可以将一个矩阵重塑为另一个大小不同的新矩阵，但保留其原始数据。

给出一个由二维数组表示的矩阵，以及两个正整数r和c，分别表示想要的重构的矩阵的行数和列数。

重构后的矩阵需要将原始矩阵的所有元素以相同的行遍历顺序填充。

如果具有给定参数的reshape操作是可行且合理的，则输出新的重塑矩阵；否则，输出原始矩阵。

示例 1:

```
输入: 
nums = 
[[1,2],
 [3,4]]
r = 1, c = 4
输出: 
[[1,2,3,4]]
解释:
行遍历nums的结果是 [1,2,3,4]。新的矩阵是 1 * 4 矩阵, 用之前的元素值一行一行填充新矩阵。
```

示例 2:

```
输入: 
nums = 
[[1,2],
 [3,4]]
r = 2, c = 4
输出: 
[[1,2],
 [3,4]]
解释:
没有办法将 2 * 2 矩阵转化为 2 * 4 矩阵。 所以输出原矩阵。
```





**注意：**

​	给定矩阵的宽和高范围在 [1, 100]。
​	给定的 r 和 c 都是正数。



```java
class Solution {
    public int[][] matrixReshape(int[][] mat, int r, int c) {
        int row = mat.length;
        int len = mat[0].length;
        
        if (row*len != r*c) {
            return mat;
        }
        
        int [][]result = new int[r][c];
        int index = 0;

        for (int i = 0; i < row; i++) {
            for (int j = 0; j < len; j++) {
                int newR = index / c;
                int newL = index % c;
                result[newR][newL] = mat[i][j];
                index++;
            }
        }
        
        return result;
    }
}
```





### （7）[找出数组中最长的连续 1](https://leetcode-cn.com/problems/max-consecutive-ones/)

给定一个二进制数组， 计算其中最大连续 1 的个数。

示例：

```
输入：[1,1,0,1,1,1]
输出：3
解释：开头的两位和最后的三位都是连续 1 ，所以最大连续 1 的个数是 3.
```


提示：

输入的数组只包含 0 和 1 。
输入数组的长度是正整数，且不超过 10,000。

```java
public static int findMaxConsecutiveOnes(int[] nums) {
        int result = 0;
        int tmp = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == 1) {
                tmp++;
            } else {
                tmp = 0;
            }
            result = Math.max(tmp,result);
        }
        return result;
    }
```



**双指针解法：**

```java
public int findMaxConsecutiveOnes(int[] nums) {
        int n = nums.length, res = 0;
        for (int i = 0; i < n; i++) {
            int j = i;
            while (j < n && nums[j] == 1) j++;
            res = Math.max(res, j - i);
            i = j;
        }
        return res;
    }
```





### （8）[有序矩阵查找](https://leetcode-cn.com/problems/search-a-2d-matrix/)

编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

每行中的整数从左到右按升序排列。
每行的第一个整数大于前一行的最后一个整数。


示例 1：

```
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
输出：true
```

```java
public static boolean searchMatrix(int[][] matrix, int target) {
        int row = matrix.length;
        int len = matrix[0].length;

        int min = matrix[0][0];
        int max = matrix[row-1][len-1];

        if (target< min || target>max) {
            return false;
        }
        if (target == min || target == max) {
            return true;
        }

        int[] temRow = new int[row];
        for (int i = 0; i < row; i++) {
            temRow[i] = matrix[i][0];
        }


        int start = 0,end = row-1;

        int targetRow = 0;

        while (start<end) {
            int mid = start + (end-start)/2;
            if (target<temRow[mid]) {
                end = mid-1;
                targetRow = mid -1;
            } else if (target>temRow[mid]) {
                if (mid<row-1 && target<temRow[mid+1]) {
                    targetRow = mid;
                    break;
                } else {
                    start = mid+1;
                    targetRow = mid < row-1 ? mid+1:mid;
                }
            } else {
                return true;
            }
        }
        if (start == end) {
            targetRow = start;
        }

        int[] temLen = new int[len];
        for (int i = 0; i < len; i++) {
            temLen[i] = matrix[targetRow][i];
        }

        start = 0;
        end = len -1;
        while (start<=end) {
            int mid = start + (end-start)/2;
            if (target<temLen[mid]) {
                end = mid-1;
            } else if (target> temLen[mid]) {
                start = mid+1;
            } else {
                return true;
            }
        }

        if (start == end && temLen[start] == target) {
            return true;
        }

        return false;
    }
```

![image-20210707234259332](../typora-user-images/image-20210707234259332.png)



```java
public static boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length;
        int n = matrix[0].length;

        int l = 0;
        int r = m - 1;
        while (l < r) {
            int mid = l + r + 1 >> 1;
            if (matrix[mid][0] <= target) {
                l = mid;
            } else {
                r = mid - 1;
            }
        }

        int row = r;
        if (matrix[row][0] == target)
            return true;
        if (matrix[row][0] > target)
            return false;

        l = 0;
        r = n - 1;
        while (l < r) {
            int mid = l + r + 1 >> 1;
            if (matrix[row][mid] <= target) {
                l = mid;
            } else {
                r = mid - 1;
            }
        }
        int col = r;

        return matrix[row][col] == target;
    }
```





**一次二分查找：**

若将矩阵每一行拼接在上一行的末尾，则会得到一个升序数组，我们可以在该数组上二分找到目标元素。

代码实现时，可以二分升序数组的下标，将其映射到原矩阵的行和列上。

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length, n = matrix[0].length;
        int low = 0, high = m * n - 1;
        while (low <= high) {
            int mid = (high - low) / 2 + low;
            int x = matrix[mid / n][mid % n];
            if (x < target) {
                low = mid + 1;
            } else if (x > target) {
                high = mid - 1;
            } else {
                return true;
            }
        }
        return false;
    }
}
```



### （9）[有序矩阵的 Kth Element](https://leetcode-cn.com/problems/kth-smallest-element-in-a-sorted-matrix/)

给你一个 n x n 矩阵 matrix ，其中每行和每列元素均按升序排序，找到矩阵中第 k 小的元素。
请注意，它是 排序后 的第 k 小元素，而不是第 k 个 不同 的元素。

示例 1：

```
输入：matrix = [[1,5,9],[10,11,13],[12,13,15]], k = 8
输出：13
解释：矩阵中的元素为 [1,5,9,10,11,12,13,13,15]，第 8 小元素是 13
```


示例 2：

```
输入：matrix = [[-5]], k = 1
输出：-5
```


提示：

```
n == matrix.length
n == matrix[i].length
1 <= n <= 300
-109 <= matrix[i][j] <= 109
题目数据 保证 matrix 中的所有行和列都按 非递减顺序 排列
1 <= k <= n2
```



**二分法：**

思路非常简单：

- 找出二维矩阵中最小的数 leftleft，最大的数 right，那么第 k 小的数必定在 left~ right 之间
  mid=(left+right) / 2；
- 在二维矩阵中寻找小于等于 mid 的元素个数 count，若这个 count 小于 k，表明第 k 小的数在右半部分且不包含 mid，即 left=mid+1, right=right，又保证了第 k 小的数在 left~ right 之间
- 若这个 count大于 k，表明第 k 小的数在左半部分且可能包含 mid，即 left=left, right=mid，又保证了第 k 小的数在 left~right 之间
- 因为每次循环中都保证了第 k 小的数在 left~ right之间，当 left==right时，第 k 小的数即被找出，等于 right

```java
 public int kthSmallest(int[][] matrix, int k) {
        int row = matrix.length;
        int col = matrix[0].length;
        int left = matrix[0][0];
        int right = matrix[row - 1][col - 1];
        while (left < right) {
            // 每次循环都保证第K小的数在start~end之间，当start==end，第k小的数就是start
            int mid = (left + right) / 2;
            // 找二维矩阵中<=mid的元素总个数
            int count = findNotBiggerThanMid(matrix, mid, row, col);
            if (count < k) {
                // 第k小的数在右半部分，且不包含mid
                left = mid + 1;
            } else {
                // 第k小的数在左半部分，可能包含mid
                right = mid;
            }
        }
        return right;
    }

    private int findNotBiggerThanMid(int[][] matrix, int mid, int row, int col) {
        // 以列为单位找，找到每一列最后一个<=mid的数即知道每一列有多少个数<=mid
        int i = row - 1;
        int j = 0;
        int count = 0;
        while (i >= 0 && j < col) {
            if (matrix[i][j] <= mid) {
                // 第j列有i+1个元素<=mid
                count += i + 1;
                j++;
            } else {
                // 第j列目前的数大于mid，需要继续在当前列往上找
                i--;
            }
        }
        return count;
    }
```

![image-20210710164721513](../typora-user-images/image-20210710164721513.png)



**最小堆法：**

使用优先级队列

```java
public int kthSmallest(int[][] matrix, int k) {

        PriorityQueue<Integer> queue = new PriorityQueue<>(k);

        int n = matrix.length;

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                queue.offer(matrix[i][j]);
            }
        }
        int i = 0;
        int result = queue.peek();
        while (!queue.isEmpty()) {
            i++;
            Integer poll = queue.poll();
            if (i == k) {
                result = poll;
            }
        }

        return result;
    }
```



![image-20210710164642262](../typora-user-images/image-20210710164642262.png)









### （10）[一个数组元素在 [1, n] 之间，其中一个数被替换为另一个数，找出重复的数和丢失的数](https://leetcode-cn.com/problems/set-mismatch/description/?utm_source=LCUS&utm_medium=ip_redirect&utm_campaign=transfer2china)

### （11）[找出数组中重复的数，数组值在 [1, n] 之间](https://leetcode-cn.com/problems/find-the-duplicate-number/)

给定一个包含 n + 1 个整数的数组 nums ，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。

假设 nums 只有 一个重复的整数 ，找出 这个重复的数 。

你设计的解决方案必须不修改数组 nums 且只用常量级 O(1) 的额外空间。

 

示例 1：

```
输入：nums = [1,3,4,2,2]
输出：2
```

提示：

```
1 <= n <= 10^5
nums.length == n + 1
1 <= nums[i] <= n
nums 中 只有一个整数 出现 两次或多次 ，其余整数均只出现 一次
```

**暴力解法：**

```java
public int findDuplicate(int[] nums) {
        Set<Integer> temSet = new HashSet<>();

        for (int i = 0; i < nums.length; i++) {
            if (!temSet.add(nums[i])) {
                return nums[i];
            }
        }
        return nums[0];
    }
```

![image-20210710170115899](../typora-user-images/image-20210710170115899.png)



```java
public int findDuplicate(int[] nums) {
        char[] temChar = new char[nums.length+1];
        for (int i=0;i<nums.length;i++) {
            if (temChar[nums[i]] == 1) {
                return nums[i];
            } else {
                temChar[nums[i]] = 1;
            }
        }
        throw new RuntimeException();
    }
```

![image-20210710170530567](../typora-user-images/image-20210710170530567.png)



**快慢指针：**

快慢指针：fast 和 slow。nums[slow] 表示取慢指针对应的元素。 注意 nums 数组中的数字都是在 1 到 n 之间的(在数组中进行游走不会越界)，因为有重复数字的出现，所以这个游走必然是成环的，环的入口就是重复的元素，即按照寻找链表环入口的思路来做。

fast 指针一次走两步，slow 指针一次走一步，若 nums 数组中有重复的数，即有环存在，则 fast 和 slow 指针一定会相遇。

当 fast 和 slow 指针相遇时，令 fast 指针重新指向 nums 数组的开头，即 fast = 0，然后 fast 指针和 slow 指针均变为为一次只走一步，当 nums[slow] 和 nums[fast] 相等时，则找到了重复的整数，返回 nums[slow] 即可。

链表中：慢指针 slow = slow.next；快指针 fast = fast.next.next

数组中：慢指针 slow = nums[slow]；快指针 fast = nums[nums[fast]]，因为 nums 中的元素值都在 11 到 nn 之间（包括 11 和 nn），所以这样不会造成数组越界问题。

```java
class Solution {

    public int findDuplicate(int[] nums) {
        
        int fast = 0, slow = 0;
        while (true) {

            slow = nums[slow];
            fast = nums[nums[fast]];
            if (slow == fast) {

                fast = 0;
                while (nums[slow] != nums[fast]) {

                    slow = nums[slow];
                    fast = nums[fast];
                }
                return nums[slow];
            }
        }
    }
}
```





**二分查找解法：**



二分查找的思路是先猜一个数（有效范围 [left..right] 里位于中间的数 mid），然后统计原始数组中 小于等于 mid 的元素的个数 cnt：

如果 cnt 严格大于 mid。根据抽屉原理，重复元素就在区间 [left..mid] 里；
否则，重复元素就在区间 [mid + 1..right] 里。

```java
public int findDuplicate(int[] nums) {
     int l = 1, h = nums.length - 1;
     while (l <= h) {
         int mid = l + (h - l) / 2;
         int cnt = 0;
         for (int i = 0; i < nums.length; i++) {
             if (nums[i] <= mid) cnt++;
         }
         if (cnt > mid) h = mid - 1;
         else l = mid + 1;
     }
     return l;
}
```









### （12）[数组相邻差值的个数](github/Leetcode 题解 数组与矩阵.md#8-数组相邻差值的个数)

### （13）[数组的度](github/Leetcode 题解 数组与矩阵.md#9-数组的度)

### （14）[对角元素相等的矩阵](github/Leetcode 题解 数组与矩阵.md#10-对角元素相等的矩阵)

### （15）[嵌套数组](github/Leetcode 题解 数组与矩阵.md#11-嵌套数组)

### （16）[分隔数组](github/Leetcode 题解 数组与矩阵.md#12-分隔数组)









## 二、链表



### （1）[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

给定一个链表，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。

如果链表中存在环，则返回 true 。 否则，返回 false 。

进阶：

你能用 O(1)（即，常量）内存解决此问题吗？

示例 1：

```
输入：head = [3,2,0,-4], pos = 1
输出：true
解释：链表中有一个环，其尾部连接到第二个节点。
```



**暴力破解：** 

```java
 public boolean hasCycle(ListNode head) {
        Set<ListNode> temSet = new HashSet<>();
        
        while (head != null) {
            if (!temSet.add(head)) {
                return true;
            }
            head = head.next;
        }
        
        return false;
    }
```

![image-20210711215228710](../typora-user-images/image-20210711215228710.png)



**双指针：**

```java
public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) {
            return false;
        }
        ListNode slow = head;
        ListNode fast = head.next;
        while (slow != fast) {
            if (fast == null || fast.next == null) {
                return false;
            }
            slow = slow.next;
            fast = fast.next.next;
        }
        return true;
    }
```

![image-20210711220937390](../typora-user-images/image-20210711220937390.png)









### （2）[合并 k 个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

给你一个链表数组，每个链表都已经按升序排列。

请你将所有链表合并到一个升序链表中，返回合并后的链表。

 

示例 1：

```
输入：lists = [[1,4,5],[1,3,4],[2,6]]
输出：[1,1,2,3,4,4,5,6]
解释：链表数组如下：
[
  1->4->5,
  1->3->4,
  2->6
]
将它们合并到一个有序链表中得到。
1->1->2->3->4->4->5->6
```


示例 2：

```
输入：lists = []
输出：[]
```


示例 3：

```
输入：lists = [[]]
输出：[]
```

 

提示：

```
k == lists.length
0 <= k <= 10^4
0 <= lists[i].length <= 500
-10^4 <= lists[i][j] <= 10^4
lists[i] 按 升序 排列
lists[i].length 的总和不超过 10^4
```





**优先级队列：**

```java
public ListNode mergeKLists(ListNode[] lists) {
        if (lists == null || lists.length == 0) return null;
        PriorityQueue<ListNode> queue = new PriorityQueue<>(lists.length, new Comparator<ListNode>() {
            @Override
            public int compare(ListNode o1, ListNode o2) {
                if (o1.val < o2.val) return -1;
                else if (o1.val == o2.val) return 0;
                else return 1;
            }
        });
        ListNode dummy = new ListNode(0);
        ListNode p = dummy;
        for (ListNode node : lists) {
            if (node != null) queue.add(node);
        }
        while (!queue.isEmpty()) {
            p.next = queue.poll();
            p = p.next;
            if (p.next != null) queue.add(p.next);
        }
        return dummy.next;


    }
```





**分治法：**

```java
 public ListNode mergeKLists(ListNode[] lists) {
        if (lists == null || lists.length == 0) return null;
        return merge(lists, 0, lists.length - 1);
    }

    private ListNode merge(ListNode[] lists, int left, int right) {
        if (left == right) return lists[left];
        int mid = left + (right - left) / 2;
        ListNode l1 = merge(lists, left, mid);
        ListNode l2 = merge(lists, mid + 1, right);
        return mergeTwoLists(l1, l2);
    }

    private ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;
        if (l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1,l2.next);
            return l2;
        }
    }
```

![image-20210711230116539](../typora-user-images/image-20210711230116539.png)





### （3）[单链表反转](https://leetcode-cn.com/problems/reverse-linked-list/)

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

**示例 1：**

```
输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]
```

**示例 2：**

```
输入：head = [1,2]
输出：[2,1]
```



**提示：**

- 链表中节点的数目范围是 `[0, 5000]`
- `-5000 <= Node.val <= 5000`



**递归求解：**

一直递归，直至最后一个节点。

然后上一个节点的下下节点置为自身

下个节点置为空。

- 使用递归函数，一直递归到链表的最后一个结点，该结点就是反转后的头结点，记作 retret .
- 此后，每次函数在返回的过程中，让当前结点的下一个结点的 nextnext 指针指向当前节点。
- 同时让当前结点的 nextnext 指针指向 NULLNULL ，从而实现从链表尾部开始的局部反转
- 当递归函数全部出栈后，链表反转完成。



![img](../typora-user-images/8951bc3b8b7eb4da2a46063c1bb96932e7a69910c0a93d973bd8aa5517e59fc8.gif)

```java
public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        
        ListNode result = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return result;
    }
```

![image-20210711230614859](../typora-user-images/image-20210711230614859.png)



**非递归求解：**

```java
 public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        ListNode pre = null;
        ListNode mid = head;
        ListNode back = head.next;

        while (back != null) {
            ListNode next = back.next;
            mid.next = pre;
            back.next = mid;
            if (next == null) {
                break;
            }

            pre = mid;
            mid = back;
            back = next;
        }

        return back;
    }
```



![image-20210711232104254](../typora-user-images/image-20210711232104254.png)





- 定义两个指针： prepre 和 curcur ；prepre 在前 curcur 在后。
- 每次让 prepre 的 nextnext 指向 curcur ，实现一次局部反转
- 局部反转完成之后，prepre 和 curcur 同时往前移动一个位置
- 循环上述过程，直至 prepre 到达链表尾部

```java
public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode nextTemp = curr.next;
            curr.next = prev;
            prev = curr;
            curr = nextTemp;
        }
        return prev;
    }
```

![img](../typora-user-images/9ce26a709147ad9ce6152d604efc1cc19a33dc5d467ed2aae5bc68463fdd2888.gif)





### （4）[求链表的中间结点](https://leetcode-cn.com/problems/middle-of-the-linked-list/)

给定一个头结点为 head 的非空单链表，返回链表的中间结点。

如果有两个中间结点，则返回第二个中间结点。

 

示例 1：

```
输入：[1,2,3,4,5]
输出：此列表中的结点 3 (序列化形式：[3,4,5])
返回的结点值为 3 。 (测评系统对该结点序列化表述是 [3,4,5])。
注意，我们返回了一个 ListNode 类型的对象 ans，这样：
ans.val = 3, ans.next.val = 4, ans.next.next.val = 5, 以及 ans.next.next.next = NULL.
```


示例 2：

```
输入：[1,2,3,4,5,6]
输出：此列表中的结点 4 (序列化形式：[4,5,6])
由于该列表有两个中间结点，值分别为 3 和 4，我们返回第二个结点。
```



**双指针：**

使用两个指针变量，刚开始都位于链表的第 1 个结点，一个永远一次只走 1 步，一个永远一次只走 2 步，一个在前，一个在后，同时走。这样当快指针走完的时候，慢指针就来到了链表的中间位置。

```java
public ListNode middleNode(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        
        ListNode slow = head;
        ListNode fast = head.next;
        
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        if (fast != null && fast.next == null) { //处理偶数节点的情况，如果是节点数量是单数，那么最后fast==null
            slow = slow.next;
        }
        
        return slow;
    }
```

![image-20210711233522806](../typora-user-images/image-20210711233522806.png)





```java
public ListNode middleNode(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        
        ListNode slow = head;
        ListNode fast = head;
        
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        
        return slow;
    }
```

![image-20210711233732744](../typora-user-images/image-20210711233732744.png)

### （5）[找出两个链表的交点](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

给你两个单链表的头节点 headA 和 headB ，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 null 。

示例 1：

```
输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
输出：Intersected at '8'
解释：相交节点的值为 8 （注意，如果两个链表相交则不能为 0）。
从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。
在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点
```





**hash法：**

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        Set<ListNode> temMap = new HashSet<>();

        while (headA != null) {
            temMap.add(headA);
            headA = headA.next;
        }
        while (headB != null) {
            if (!temMap.add(headB)) {
                return headB;
            }
            headB = headB.next;
        }
        return null;
    }
```

![1626079791929](../typora-user-images/1626079791929.png)





设 A 的长度为 a + c，B 的长度为 b + c，其中 c 为尾部公共部分长度，可知 a + c + b = b + c + a。

当访问 A 链表的指针访问到链表尾部时，令它从链表 B 的头部开始访问链表 B；同样地，当访问 B 链表的指针访问到链表尾部时，令它从链表 A 的头部开始访问链表 A。这样就能控制访问 A 和 B 两个链表的指针能同时访问到交点。

如果不存在交点，那么 a + b = b + a，以下实现代码中 l1 和 l2 会同时为 null，从而退出循环。



如果只是判断是否存在交点，那么就是另一个问题，即 [编程之美 3.6]() 的问题。有两种解法：

- 把第一个链表的结尾连接到第二个链表的开头，看第二个链表是否存在环；
- 或者直接比较两个链表的最后一个节点是否相同。

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode l1 = headA, l2 = headB;
    while (l1 != l2) {
        l1 = (l1 == null) ? headB : l1.next;
        l2 = (l2 == null) ? headA : l2.next;
    }
    return l1;
}
```

![1626081289298](../typora-user-images/1626081289298.png)











### （7）[归并两个有序的链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

将两个升序链表合并为一个新的 升序 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

 

示例 1：

```
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
```


示例 2：

```
输入：l1 = [], l2 = []
输出：[]
```


示例 3：

```
输入：l1 = [], l2 = [0]
输出：[0]
```





**归并排序：**

```java
 public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode result = new ListNode();
        ListNode temRes = result;

        while (l1 != null && l2!=null) {
            if (l1.val<=l2.val) {
                temRes.next = l1;
                l1 = l1.next;
            } else {
                temRes.next = l2;
                l2 = l2.next;
            }
            temRes = temRes.next;
        }

        while (l1 != null) {
            temRes.next = l1;
            l1 = l1.next;
            temRes = temRes.next;
        }

        while (l2 != null) {
            temRes.next = l2;
            l2 = l2.next;
            temRes = temRes.next;
        }
        
        return result.next;
    }
```

![image-20210712222513283](../typora-user-images/image-20210712222513283.png)



**递归：**

思路

我们可以如下递归地定义两个链表里的 merge 操作（忽略边界情况，比如空链表等）：

![image-20210712223143313](../typora-user-images/image-20210712223143313.png)

 也就是说，两个链表头部值较小的一个节点与剩下元素的 `merge` 操作结果合并。

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        } else if (l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }
```

![image-20210712222856571](../typora-user-images/image-20210712222856571.png)







### （8）[从有序链表中删除重复节点](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

存在一个按升序排列的链表，给你这个链表的头节点 head ，请你删除所有重复的元素，使每个元素 只出现一次 。

返回同样按升序排列的结果链表。

示例 1：

```
输入：head = [1,1,2]
输出：[1,2]
```





如果当前元素跟下一个元素相等，那么删除下一个元素，直接更新指针，当前元素的下一个元素为当前元素的下下个元素。

```java
public ListNode deleteDuplicates(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        ListNode result = head;

        while (head != null && head.next != null) {
            if (head.val == head.next.val) {
                head.next = head.next.next;
            } else {
                head = head.next;
            }
        }

        return result;
    }
```

![image-20210712223947789](../typora-user-images/image-20210712223947789.png)







### （9）[删除链表的倒数第 n 个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

进阶：你能尝试使用一趟扫描实现吗？

示例 1：

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```


示例 2：

```
输入：head = [1], n = 1
输出：[]
```


示例 3：

```
输入：head = [1,2], n = 1
输出：[1]
```



**快慢指针：**

先加一个前置指针，不加的话如果n刚好等于链表的长度，会求解失败，

然后用快慢指针，

快指针先走n步，然后快慢指针一步一步走，等快指针到尾部时候，慢指针的下一个节点就是倒数第n个节点。

然后删除倒数第n个节点

```java
slow.next = slow.next.next;
```



```java
public ListNode removeNthFromEnd(ListNode head, int n) {
        if (head == null) {
            return head;
        }
		
        ListNode result = new ListNode();
        result.next = head;

        ListNode slow = result,fast = result;

        while (fast.next != null && n>0) {
            fast = fast.next;
            n--;
        }

        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next;
        }

        slow.next = slow.next.next;

        return result.next;
    }
```

![image-20210712231236840](../typora-user-images/image-20210712231236840.png)





**栈：**

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0, head);
        Deque<ListNode> stack = new LinkedList<ListNode>();
        ListNode cur = dummy;
        while (cur != null) {
            stack.push(cur);
            cur = cur.next;
        }
        for (int i = 0; i < n; ++i) {
            stack.pop();
        }
        ListNode prev = stack.peek();
        prev.next = prev.next.next;
        ListNode ans = dummy.next;
        return ans;
    }
```









### （10）[ 交换链表中的相邻结点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。

你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

 

示例 1：

```
输入：head = [1,2,3,4]
输出：[2,1,4,3]
```


示例 2：

```
输入：head = []
输出：[]
```


示例 3：

```
输入：head = [1]
输出：[1]
```




**前置指针：**

借用一个前置指针。用来指明前驱节点，否则后面置换的时候无法更新前驱节点的下一个节点。

```java
public ListNode swapPairs(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode pre = new ListNode();
        pre.next = head;
        ListNode result = pre;
        ListNode first = head;

        while (first != null && first.next != null) {
            ListNode temNode  = first.next.next;
            first.next.next = first;
            if (pre != null) {
                pre.next = first.next;
            }
            first.next = temNode;

            pre = first;
            first = first.next;

        }
        return result.next;
    }
```

![image-20210713234322327](../typora-user-images/image-20210713234322327.png)



**递归解法：**

递归写法要观察本级递归的解决过程，形成抽象模型，因为递归本质就是不断重复相同的事情。而不是去思考完整的调用栈，一级又一级，无从下手。如图所示，我们应该关注一级调用小单元的情况，也就是单个f(x)。

其中我们应该关心的主要有三点：

1. 返回值
2. 调用单元做了什么
3. 终止条件

- 返回值：交换完成的子链表
- 调用单元：设需要交换的两个点为 head 和 next，head 连接后面交换完成的子链表，next 连接 head，完成交换
- 终止条件：head 为空指针或者 next 为空指针，也就是当前无节点或者只有一个节点，无法进行交换

```java
class Solution {
    public ListNode swapPairs(ListNode head) {
        if(head == null || head.next == null){
            return head;
        }
        ListNode next = head.next;
        head.next = swapPairs(next.next);
        next.next = head;
        return next;
    }
}
```





### （11）[ 链表求和](https://leetcode-cn.com/problems/sum-lists-lcci/)

给定两个用链表表示的整数，每个节点包含一个数位。

这些数位是反向存放的，也就是个位排在链表首部。

编写函数对这两个整数求和，并用链表形式返回结果。

 

示例：

```
输入：(7 -> 1 -> 6) + (5 -> 9 -> 2)，即617 + 295
输出：2 -> 1 -> 9，即912
```



**归并法：**

类似于归并算法，并设置一个标识位，标识上一个位置的数值是否大于0，如果大于0，就在当前值上加1。

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode result = new ListNode(0);
        ListNode tem = result;

        boolean overTen = false;
        while (l1 != null && l2 != null) {
            int curVal = l1.val + l2.val + (overTen ? 1:0);
            overTen = curVal>=10;
            tem.next  = new ListNode(curVal % 10);
            if (result == null) {
                result = tem;
            }
            tem = tem.next;
            l1 = l1.next;
            l2 = l2.next;
        }

        while (l1 != null) {
            int curVal = l1.val + (overTen ? 1:0);
            overTen = curVal>=10;
            tem.next  = new ListNode(curVal % 10);
            tem = tem.next;
            l1 = l1.next;
        }
        while (l2 != null) {
            int curVal = l2.val + (overTen ? 1:0);
            overTen = curVal>=10;
            tem.next  = new ListNode(curVal % 10);
            tem = tem.next;
            l2 = l2.next;
        }
        if (overTen) {
            tem.next = new ListNode(1);
        }

        return result.next;

    }
```

![image-20210714235610722](../typora-user-images/image-20210714235610722.png)



**递归求解：**

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
       return recursion(l1,l2,false);

    }

     private ListNode recursion(ListNode l1, ListNode l2, boolean overTen) {
        if (l1 == null && l2 == null && !overTen) {
            return null;
        }
        int curVal = ((l1 == null)? 0: l1.val) + ((l2 == null)? 0: l2.val) + (overTen?1:0);
        ListNode result = new ListNode(curVal % 10);
        result.next = recursion(l1==null?null:l1.next,l2==null?null:l2.next,curVal >= 10);
        return result;
    }
```

![image-20210714235538566](../typora-user-images/image-20210714235538566.png)



进阶：思考一下，假设这些数位是正向存放的，又该如何解决呢?

示例：

```
输入：(6 -> 1 -> 7) + (2 -> 9 -> 5)，即617 + 295
输出：9 -> 1 -> 2，即912
```

**使用栈：**

```java
public ListNode addTwoNumbers2(ListNode l1, ListNode l2) {

        Stack<Integer> first = new Stack<>();
        Stack<Integer> second = new Stack<>();
        Stack<Integer> resultStack = new Stack<>();

        while (l1 != null) {
            first.push(l1.val);
            l1 = l1.next;
        }
        while (l2 != null) {
            second.push(l2.val);
            l2 = l2.next;
        }

        boolean overTen = false;
        while (!first.isEmpty() && !second.isEmpty()) {
            int curVal = first.pop() + second.pop() + (overTen ? 1:0);
            overTen = curVal>=10;
            resultStack.push(curVal%10);
        }

        while (!first.isEmpty()) {
            int curVal = first.pop() + (overTen ? 1:0);
            overTen = curVal>=10;
            resultStack.push(curVal%10);
        }
        while (!second.isEmpty()) {
            int curVal = second.pop() + (overTen ? 1:0);
            overTen = curVal>=10;
            resultStack.push(curVal%10);
        }
        if (overTen) {
            resultStack.push(1);
        }

        ListNode result = new ListNode(0);
        ListNode tem = result;

        while (!resultStack.isEmpty()) {
            tem.next = new ListNode(resultStack.pop());
            tem = tem.next;
        }
        return result.next;
    }
```















### （12）[ 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)

请判断一个链表是否为回文链表。

**示例 1:**

```
输入: 1->2
输出: false
```

**示例 2:**

```
输入: 1->2->2->1
输出: true
```

**进阶：**
你能否用 O(n) 时间复杂度和 O(1) 空间复杂度解决此题？



**双指针：**

- 遍历一遍，然后反转前半部分链表，直到快指针到链表尾部
- 如果节点数量是奇数，那么需要让慢指针往下走一步
- 然后对比前半部分反转的链表和慢指针链表

```java
public boolean isPalindrome(ListNode head) {
       if (head == null || head.next == null) {
            return true;
        }


        ListNode pre = null,resert = null;
        ListNode slow = head,fast = head;

        while (fast != null && fast.next != null) {
            pre = slow;
            slow = slow.next;
            fast = fast.next.next;

            pre.next = resert;
            resert = pre;
        }

        if (fast != null) {
            slow = slow.next;
        }

        while (resert != null && slow != null) {
            if (resert.val != slow.val) {
                return false;
            }
            resert = resert.next;
            slow = slow.next;
        }

        return true;
    }
```

![image-20210716002319478](../typora-user-images/image-20210716002319478.png)







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