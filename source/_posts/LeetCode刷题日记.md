---
title: LeetCode刷题日记
date: 2022-07-01 22:43:11
tags: 算法
categories: 学习笔记-算法
---

## 字符串部分

#### [415. 字符串相加](https://leetcode.cn/problems/add-strings/)

给定两个字符串形式的非负整数 num1 和num2 ，计算它们的和并同样以字符串形式返回。

你不能使用任何內建的用于处理大整数的库（比如 BigInteger）， 也不能直接将输入的字符串转换为整数形式。

 

示例 1：

输入：num1 = "11", num2 = "123"
输出："134"

```java
class Solution {
    public static void main(String[] args) {
        Solution s = new Solution();
        String result = s.addStrings("6913259244","71103343");
         System.out.println(result);
    }
    public String addStrings(String num1, String num2) {
        int i = num1.length()-1;
        int j = num2.length()-1;
        int add = 0;
        StringBuffer result = new StringBuffer();
        while(i>=0||j>=0||add!=0){
            int x = i>=0 ? num1.charAt(i)-'0':0;
            int y = j>=0 ? num2.charAt(j)-'0':0;
            int sum = x+y+add;
            result.append(sum%10);
            add = sum/10;
            i--;j--;
        }
        result.reverse();
        return result.toString();
    }
}

```

#### KMP算法

```java
/**
 * KMP算法
 */
public class kmp {
    public static void main(String[] args) {
        char[] pattern = {'a','b','c','a','b','c','d'};
        String str = "fghabcabca abcabcd efg";
        int[] next = new int[pattern.length];
        kmpSearch(str.toCharArray(), pattern, next);
    }

    public static void kmpSearch(char[] str,char[] pattern,int[] next){
        getNext(pattern, next);
        int i=0,j=0;
        while(i<str.length){
            if(j==-1){
                i++;
                j++;
            }else if(str[i]==pattern[j]){
                i++;
                j++;
            }else{
                j=next[j];
            }
            if(j==pattern.length){
                System.out.println(i-j);
                break;
            }
        }
    }

    public static void getNext(char[] pattern,int[] next){
        next[0]=-1;
        int i=0,j=-1;
        while(i<pattern.length){
            if(j==-1){
                i++;
                j++;
            }else if(pattern[i]==pattern[j]){
                i++;
                j++;
                next[i] = j;
            }else{
                //next[i]=0;
                j=next[j];
            }
        }
    }
}

```

#### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

给你一个字符串 `s`，找到 `s` 中最长的回文子串。

 

**示例 1：**

```
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。
```

**示例 2：**

```
输入：s = "cbbd"
输出："bb"
```

 

```java
class Solution {
     public String longestPalindrome(String s) {
        int len = s.length();
        if(len==1) return s;
        if(len==2){
            if(s.charAt(0)==s.charAt(1)) return s;
            return String.valueOf(s.charAt(0));
        }
        while(len>1){
            for(int i=0;i<=s.length()-len;i++){
                int j_tmp=i;int k_tmp=len-1+i;
                for(int j=i,k=len-1+i;j<=k;j++,k--){
                    if(j==k) return (String) s.subSequence(j_tmp, k_tmp+1);
                    if(s.charAt(j)==s.charAt(k)){
                        if(k-j==1) return s.substring(j_tmp, k_tmp+1);
                        //continue;
                    }else{
                        break;
                    }
                    
                }
            }
            len--;
        }
        return String.valueOf(s.charAt(0)) ;

    }
}
```

#### [剑指 Offer 05. 替换空格](https://leetcode.cn/problems/ti-huan-kong-ge-lcof/)

请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

**示例 1：**

```
输入：s = "We are happy."
输出："We%20are%20happy."
```

```java
	// 用遍历来解
    public static String replaceSpace(String s) {
        StringBuffer res = new StringBuffer();
        char[] res1 = s.toCharArray();
        for (char c : res1) {
            if (c == ' ') {
                res.append("%20");
            } else {
                res.append(c);
            }
        }
        return res.toString();
    }

    // 用JavaApi解
    public static String replaceSpace2(String s) {
        StringBuffer str = new StringBuffer(s);
        return str.toString().replaceAll("\\s", "%20");
    }
```

#### [125. 验证回文串](https://leetcode.cn/problems/valid-palindrome/)

给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。

**说明：**本题中，我们将空字符串定义为有效的回文串。

 

**示例 1:**

```
输入: "A man, a plan, a canal: Panama"
输出: true
解释："amanaplanacanalpanama" 是回文串
```

**示例 2:**

```
输入: "race a car"
输出: false
解释："raceacar" 不是回文串
```

第一种方法：笨方法，没有使用Character方法判断

```java
public static void main(String[] args) {
        String s =  "ab_a";
        System.out.println(isPalindrome2(s)); 
 
    }
    public static boolean isPalindrome(String s) {
        int len = s.length();
        s=s.toLowerCase();//把大写字母转小写
        char[] c = s.toCharArray();//转成字符数组
        //先把字符串中的非字母去除
        StringBuilder builder = new StringBuilder();
        for(int i=0;i<len;i++){
            if((c[i]>='a'&&c[i]<='z')||(c[i]>='0'&&c[i]<='9')){
                builder.append(c[i]);
            }
        }
        s=builder.toString();
        //判断是否为回文串
        if(builder.length()<=1){
            return true;
        }
        for(int i=0,j=builder.length()-1;j>=i;){
            if(s.charAt(i)==s.charAt(j)){
                i++;j--;
            }else if(i<j){
                return false;
            }
            if(i==j||i>j) return true;
        }
        return false;
    }
```

第二种，使用Character方法判断字符，并使用StringBuffer的reverse方法判断

```java
public static boolean isPalindrome(String s) {
        StringBuffer sBuffer = new StringBuffer();
        int len = s.length();
        char[] c = s.toCharArray();
        for(int i=0;i<len;i++){
            if(Character.isLetterOrDigit(c[i])){
                sBuffer.append(Character.toLowerCase(c[i]));
            }
        }
        StringBuffer reverseBuffer = new StringBuffer(sBuffer).reverse();
        return reverseBuffer.toString().equals(sBuffer.toString()) ;     
    }
```

#### [14. 最长公共前缀](https://leetcode.cn/problems/longest-common-prefix/)

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 `""`。

**示例 1：**

```
输入：strs = ["flower","flow","flight"]
输出："fl"
```

**示例 2：**

```
输入：strs = ["dog","racecar","car"]
输出：""
解释：输入不存在公共前缀。
```

思路：将字符串数组排序(Arrays.sort方法)，再比较第一个与最后一个字符串

```java
class Solution {
    public static String longestCommonPrefix(String[] strs) {
        //将字符串排序，然后比较第一个与最后一个字符串
        Arrays.sort(strs);
        //比较第一个与最后一个字符串
        return compareString(strs[0], strs[strs.length-1]);
    }
    public static String compareString(String s1,String s2){
        int s1_len = s1.length();
        int s2_len = s2.length();
        StringBuffer result = new StringBuffer();
        for(int i=0,j=0;i<s1_len&&j<s2_len;i++,j++){
            if(s1.charAt(i)==s2.charAt(j)){
                result.append(s1.charAt(i));
            }else{
                break;
            }
        }
        return result.toString();
    }
}
```



## 贪心

#### [198. 打家劫舍-贪心](https://leetcode.cn/problems/house-robber/)

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

示例 1：

输入：[1,2,3,1]
输出：4
解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。

```java
public class DBDemo1 {
    public static void main(String[] args) {
        int[] nums = new int[]{2,3,9,3,1};
        System.out.println(maxMoney(nums));
    }

    /**
     * 递归解决办法
     * @param nums
     * @param index
     * @return
     */
    public static int maxMoney(int[] nums,int index){
        if(nums == null||index<0){
            return 0;
        }
        if(index==0){
            return nums[0];
        }
        return Math.max(maxMoney(nums, index-1), maxMoney(nums, index-2)+nums[index]);
    }
    /**
     * 循环解决方法
     * @param nums
     * @return
     */
    public static int maxMoney(int[] nums){
        int db[] = new int[nums.length];
        int len = nums.length;
        if(nums == null||len==0){
            return 0;
        }
        if(len == 1){
            return nums[0];
        }
        db[0] = nums[0];
        db[1] = Math.max(nums[0], nums[1]);
        for(int i = 2;i<len;i++){
            db[i] = Math.max(db[i-1], db[i-2]+nums[i]);
        }
        return db[len-1];
    }
}

```

#### [213. 打家劫舍 II](https://leetcode.cn/problems/house-robber-ii/)

你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。

给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，今晚能够偷窃到的最高金额。

> 这道题其实和前一道题思路一样，只是在求解时要分别求两次最大值，一次去掉头，一次去掉尾，这样就能保证他们不会首尾相连，而且也可以求出所有情况的最优解

```java
package javaTest.test;

public class DBDemo2 {
    public static void main(String[] args) {
        int[] nums = new int[]{2,7,9,3,1};
        System.out.println(rob(nums));
    }
    public static int rob(int[] nums) {
        if(nums.length==1){
            return nums[0];
        }
        if(nums.length==2){
            return Math.max(nums[0], nums[1]);
        }
        int result = Math.max(maxMoney(nums, 0, nums.length-2), maxMoney(nums, 1, nums.length-1));
        return result;
        
    }
    public static int maxMoney(int[] nums,int start,int end){
        int first = nums[start];
        int second = Math.max(nums[start], nums[start+1]);
        int tmp=second;
        for(int i = start+2;i<=end;i++){
            tmp=second;
            second = Math.max(first+nums[i], second);
            first=tmp;
        }
        return second;
    }
}

```



#### [53. 最大子数组和-动态规划](https://leetcode.cn/problems/maximum-subarray/)

#### 

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组** 是数组中的一个连续部分。

**示例 1：**

```
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```

**示例 2：**

```
输入：nums = [1]
输出：1
```

**示例 3：**

```
输入：nums = [5,4,-1,7,8]
输出：23
```

```java
public int maxSubArray(int[] nums) {
        int[] res = new int[nums.length];
        res[0] = nums[0];
        int max = res[0];
        for(int i=1;i<nums.length;i++){
            res[i]=Math.max(res[i-1]+nums[i], nums[i]);
            if(res[i]>max) max = res[i];
        }
        return max;
    }
```

#### [435. 无重叠区间-贪心](https://leetcode.cn/problems/non-overlapping-intervals/)

给定一个区间的集合 `intervals` ，其中 `intervals[i] = [starti, endi]` 。返回 *需要移除区间的最小数量，使剩余区间互不重叠* 。

**示例 1:**

```
输入: intervals = [[1,2],[2,3],[3,4],[1,3]]
输出: 1
解释: 移除 [1,3] 后，剩下的区间没有重叠。
```

**示例 2:**

```
输入: intervals = [ [1,2], [1,2], [1,2] ]
输出: 2
解释: 你需要移除两个 [1,2] 来使剩下的区间没有重叠。
```

**示例 3:**

```
输入: intervals = [ [1,2], [2,3] ]
输出: 0
解释: 你不需要移除任何区间，因为它们已经是无重叠的了。
```

**贪心算法或贪心思想**采用贪心的策略，保证每次操作都是局部最优的，从而使最 后得到的结果是全局最优的。

```java
/**
*贪心策略：优先保留结尾小且不相交的区间。
*/
    public static int eraseOverlapIntervals(int[][] intervals) {
        //二维数组排序，按照第二个元素排序
        Arrays.sort(intervals,new Comparator<int[]>() {
            @Override
            public int compare(int[] o1, int[] o2) {
                return o1[1]-o2[1];
            }
        });
        int result=0;
        int pre=intervals[0][1];
        for(int i=1;i<intervals.length;i++){
            if(intervals[i][0]<pre){
                result++;
            }else{
                pre=intervals[i][1];
            }
        }
        return result;
    }
```

#### [452. 用最少数量的箭引爆气球-贪心](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)

有一些球形气球贴在一堵用 XY 平面表示的墙面上。墙面上的气球记录在整数数组 `points` ，其中`points[i] = [xstart, xend]` 表示水平直径在 `xstart` 和 `xend`之间的气球。你不知道气球的确切 y 坐标。

一支弓箭可以沿着 x 轴从不同点 **完全垂直** 地射出。在坐标 `x` 处射出一支箭，若有一个气球的直径的开始和结束坐标为 `x``start`，`x``end`， 且满足  `xstart ≤ x ≤ x``end`，则该气球会被 **引爆** 。可以射出的弓箭的数量 **没有限制** 。 弓箭一旦被射出之后，可以无限地前进。

给你一个数组 `points` ，*返回引爆所有气球所必须射出的 **最小** 弓箭数* 。

**示例 1：**

```
输入：points = [[10,16],[2,8],[1,6],[7,12]]
输出：2
解释：气球可以用2支箭来爆破:
-在x = 6处射出箭，击破气球[2,8]和[1,6]。
-在x = 11处发射箭，击破气球[10,16]和[7,12]。
```

**示例 2：**

```
输入：points = [[1,2],[3,4],[5,6],[7,8]]
输出：4
解释：每个气球需要射出一支箭，总共需要4支箭。
```

**示例 3：**

```
输入：points = [[1,2],[2,3],[3,4],[4,5]]
输出：2
解释：气球可以用2支箭来爆破:
- 在x = 2处发射箭，击破气球[1,2]和[2,3]。
- 在x = 4处射出箭，击破气球[3,4]和[4,5]。
```

```java
//贪心思想
class Solution {
    public static int findMinArrowShots(int[][] points) {
        Arrays.sort(points,new Comparator<int[]>() {
            public int compare(int[] o1, int[] o2) {
                if(o1[0]>o2[0]){
                   return 1; 
                } else {
                    return -1;
                }
            }    
        });
        int pre=points[0][0];
        int rear=points[0][1];
        int sum=1;//最终结果
        for (int i = 1; i < points.length; i++) {
            if(points[i][0]>rear){
                sum++;
                pre=points[i][0];
                rear=points[i][1];
            }else{
                pre=Math.max(pre, points[i][0]);
                rear=Math.min(rear, points[i][1]);
            }
        }
        return sum;
    }
}
```

#### [406. 根据身高重建队列-贪心](https://leetcode.cn/problems/queue-reconstruction-by-height/)

假设有打乱顺序的一群人站成一个队列，数组 `people` 表示队列中一些人的属性（不一定按顺序）。每个 `people[i] = [hi, ki]` 表示第 `i` 个人的身高为 `hi` ，前面 **正好** 有 `ki` 个身高大于或等于 `hi` 的人。

请你重新构造并返回输入数组 `people` 所表示的队列。返回的队列应该格式化为数组 `queue` ，其中 `queue[j] = [hj, kj]` 是队列中第 `j` 个人的属性（`queue[0]` 是排在队列前面的人）。

**示例 1：**

```
输入：people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]
输出：[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]
```

**示例 2：**

```
输入：people = [[6,0],[5,0],[4,0],[3,2],[2,2],[1,4]]
输出：[[4,0],[5,0],[2,2],[3,2],[1,4],[6,0]]
```

```java
public static int[][] reconstructQueue(int[][] people) {
        Arrays.sort(people,new Comparator<int[]>() {
            public int compare(int[] o1, int[] o2) {
                if(o1[0]<o2[0]) return 1;
                if(o1[0]==o2[0]) return o1[1]-o2[1];
                return -1;
            };
        });

        List<int[]> list = new LinkedList<>();
        for (int i = 0; i < people.length; i++) {
            list.add(people[i][1], people[i]);
        }
        list.toArray(people);
        return people;
    }
```

#### [665. 非递减数列-贪心](https://leetcode.cn/problems/non-decreasing-array/)

给你一个长度为 `n` 的整数数组 `nums` ，请你判断在 **最多** 改变 `1` 个元素的情况下，该数组能否变成一个非递减数列。

我们是这样定义一个非递减数列的： 对于数组中任意的 `i` `(0 <= i <= n-2)`，总满足 `nums[i] <= nums[i + 1]`。

 

**示例 1:**

```
输入: nums = [4,2,3]
输出: true
解释: 你可以通过把第一个 4 变成 1 来使得它成为一个非递减数列。
```

**示例 2:**

```
输入: nums = [4,2,1]
输出: false
解释: 你不能在只改变一个元素的情况下将其变为非递减数列。
```

```java
public static boolean checkPossibility(int[] nums) {
        int[] tmp = new int[nums.length-1];
        for (int i = 1; i < nums.length; i++) {
            tmp[i-1]=nums[i-1]-nums[i];
        }
        IntPredicate predicate = new IntPredicate(){
            @Override
            public boolean test(int value) {
                return value>0;
            }   
        };
        int[] sub =Arrays.stream(tmp).filter(predicate).toArray();
        if(sub.length==0) return true;
        if(sub.length>1) return false;
        if(tmp[0]>0||tmp[tmp.length-1]>0) return true;
        
        return isSorted(tmp);
    }
    public static boolean isSorted(int[] tmp){
        for (int i = 1; i < tmp.length; i++) {
            if(tmp[i]>0){
                if(Math.abs(tmp[i-1])>Math.abs(tmp[i])||Math.abs(tmp[i+1])>Math.abs(tmp[i])){
                    return true;
                }
            }
        }
        return false;
    }
```





# 双指针

#### [35. 搜索插入位置-双指针](https://leetcode.cn/problems/search-insert-position/)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 O(log n) 的算法。

利用二分查找思想

 ```
示例 1:
输入: nums = [1,3,5,6], target = 5
输出: 2

示例 2:
输入: nums = [1,3,5,6], target = 2
输出: 1

示例 3:
输入: nums = [1,3,5,6], target = 7
输出: 4
 ```

```java
class Solution {
    public static int searchInsert(int[] nums, int target) {
        int result = midSearch(nums,0, nums.length-1,target);
        return result;
    }
    private static int midSearch(int[] nums, int left,int right,int target) {
        if(right<left){
            return left;
        }
        if(nums[right]==target){
            return right;
        }
        if(nums[left]==target){
            return left;
        }
        int mid = (right+left)/2;
        if(nums[mid]==target){
            return mid;
        }
        if(nums[mid]<target){
            mid++;
            return midSearch(nums, mid, right, target);
        }else{
            mid--;
            return midSearch(nums, left, mid, target);
        }
    }
}
```

#### 

#### [11. 盛最多水的容器-双指针](https://leetcode.cn/problems/container-with-most-water/)

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

```
输入：[1,8,6,2,5,4,8,3,7]
输出：49 
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。

示例 2：
输入：height = [1,1]
输出：1
```

```java
//双指针法
public static int maxArea(int[] height) {
        int len = height.length;
        int max=0;
        for(int i=0,j=len-1;j>i&&i>=0;){
            int area=(j-i)*(Math.min(height[i], height[j]));
            if(area>max) max=area;
            if(height[i]>height[j]){
                while(height[j-1]<height[j]&&j>i) j--;
                j=j-1;
            }else{
                while(height[i+1]<height[i]&&j>i) i++;
                i=i+1;
            }
        }
        return max;
    }
```

#### [142. 环形链表 II-双指针/哈希](https://leetcode.cn/problems/linked-list-cycle-ii/)

给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改** 链表。

```java
//双指针-快慢指针，一个快指针和一个慢指针，如果存在环形链表，则他们一定会相遇
public class Solution {
    public ListNode detectCycle(ListNode head) {
        if (head == null) {
            return null;
        }
        ListNode slow = head, fast = head;
        while (fast != null) {
            slow = slow.next;
            if (fast.next != null) {
                fast = fast.next.next;
            } else {
                return null;
            }
            if (fast == slow) {
                ListNode ptr = head;
                while (ptr != slow) {
                    ptr = ptr.next;
                    slow = slow.next;
                }
                return ptr;
            }
        }
        return null;
    }
}
```

```java
//哈希set方法
    public ListNode detectCycle(ListNode head) {
        HashSet<ListNode> set = new HashSet<>();
        ListNode tmp = head;
        while(tmp!=null){
            if(set.contains(tmp)){
                return tmp;
            }else {
                set.add(tmp);
                tmp=tmp.next;
            }
        }
        return null;
    }
```



#### [76. 最小覆盖子串- 滑动窗口](https://leetcode.cn/problems/minimum-window-substring/)

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。

**注意：**

- 对于 `t` 中重复字符，我们寻找的子字符串中该字符数量必须不少于 `t` 中该字符数量。
- 如果 `s` 中存在这样的子串，我们保证它是唯一的答案。

 

**示例 1：**

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
```

**示例 2：**

```
输入：s = "a", t = "a"
输出："a"
```

**示例 3:**

```
输入: s = "a", t = "aa"
输出: ""
```

```java
class Solution {
    public static String minWindow(String s, String t) {
        if (s == null || s == "" || t == null || t == "" || s.length() < t.length()) {
            return "";
        }
        //Ascll码的长度为128
        int[] need = new int[128];
        int[] have = new int[128];

        for (int i = 0; i < t.length(); i++) {
            need[t.charAt(i)]++;
        }
        int left=0,right=0,min=s.length()+1,count=0,start=0;
        while(right<s.length()){
            char r = s.charAt(right);
            if(need[r]==0){
                //说明该字符不是需要的
                right++;
                continue;
            }
            if(have[r]<need[r]){
                count++;//
            }
            have[r]++;
            right++;
            while(count == t.length()){
                if(right-left<min){
                    min = right-left;
                    start = left;
                }
                char l = s.charAt(left);
                if(need[l]==0){
                    left++;
                    continue;
                }
                if(have[l] == need[l]){
                    count--;
                }
                have[l]--;
                left++;
            }
        }
        if(min==s.length()+1) return "";
        return s.substring(start, start+min);
    }
}
```



# 树

#### [199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)

给定一个二叉树的 **根节点** `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

![](https://assets.leetcode.com/uploads/2021/02/14/tree.jpg)

```
输入: [1,2,3,null,5,null,4]
输出: [1,3,4]
```

```java
/**
*广度优先算法
**/
class Solution {
   public List<Integer> rightSideView(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if(root==null){
            return result;
        }
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()){
            int size = queue.size();
            for(int i=0;i<size;i++){
                TreeNode node = queue.poll();
                if(node.left != null){
                    queue.offer(node.left);
                }
                if(node.right != null){
                    queue.offer(node.right);
                }
                if(i==size-1){
                    result.add(node.val);
                }
            }
        }
        return result;
    }
}
```

深度优先搜索解答

```java
/**
 * 深度优先搜索
  */
class Solution {
   public List<Integer> rightSideView(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if(root==null){
            return result;
        }
        DFS(root,result,0);
        return result;
    }

    private void DFS(TreeNode root,List<Integer> result, int dep) {
        if(root == null){
            return;
        }
        if(result.size()==dep){
            result.add(root.val);
        }
        dep++;
        if(root.right!=null){
            DFS(root.right, result, dep);
        }
        if(root.left!=null){
            DFS(root.left, result, dep);
        }
    }
}
```

两种方法的时间复杂度和空间复杂度都是O(n)

# 二分查找

#### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题。

**示例 1：**

```
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]
```

**示例 2：**

```
输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]
```

**示例 3：**

```
输入：nums = [], target = 0
输出：[-1,-1]
```



```java
public static int[] searchRange(int[] nums, int target) {
        int[] res = {-1,-1};
        int tmp = getTmp(nums, target);
        if(tmp == -1) return res;
        int i = tmp;int j = tmp;
        while (i<nums.length&&nums[i]==nums[tmp]){
            res[1]=i;
            i++;
        }
        while (j>=0&&nums[j]==nums[tmp]){
            res[0]=j;
            j--;
        }
        System.out.println(Arrays.toString(res));
        return res;
    }

    private static int getTmp(int[] nums, int target) {
        int front=0,back= nums.length-1;
        if(back<0) return -1;
        int mid = (front+back)/2;
        int tmp=-1;
        do {
            mid = (front+back)/2;
            if(nums[mid]< target){
                front = mid+1;//mid元素可以舍去，因为这一层if已经判断过
            } else if(nums[mid]> target) {
                back = mid-1;//
            } else {
                tmp = mid;
            }
        } while (front<=back&&tmp==-1);
        return tmp;
    }
```



#### [81. 搜索旋转排序数组 II](https://leetcode.cn/problems/search-in-rotated-sorted-array-ii/)

已知存在一个按非降序排列的整数数组 `nums` ，数组中的值不必互不相同。

在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了 **旋转** ，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 **从 0 开始** 计数）。例如， `[0,1,2,4,4,4,5,6,6,7]` 在下标 `5` 处经旋转后可能变为 `[4,5,6,6,7,0,1,2,4,4]` 。

给你 **旋转后** 的数组 `nums` 和一个整数 `target` ，请你编写一个函数来判断给定的目标值是否存在于数组中。如果 `nums` 中存在这个目标值 `target` ，则返回 `true` ，否则返回 `false` 。

你必须尽可能减少整个操作步骤。

 

**示例 1：**

```
输入：nums = [2,5,6,0,0,1,2], target = 0
输出：true
```

**示例 2：**

```
输入：nums = [2,5,6,0,0,1,2], target = 3
输出：false
```

```java
class Solution {
     public static boolean search(int[] nums, int target) {
        int left=0,right=nums.length-1;
        int mid = (left+right)/2;
        while (left<=right) {
            mid = (right+left)/2;
            if(nums[mid]==target) return true;
            if(nums[left]==nums[mid]) {//排除中间节点和首节点相同的情况
                left++;
                continue;
            }
            if(nums[mid]<nums[left]) {//说明后半部分有序
                if (nums[right]>=target&&nums[mid]<target) {//应该从右半部分找
                    left = mid+1;
                }else { //应该从左半部分找
                    right = mid-1;
                }
            } else { //说明前半部分有序
                if(nums[left]<=target&&nums[mid]>target) {
                    right = mid-1;
                }else {
                    left = mid+1;
                }
            }
        }
        return false;
    }
}
```

#### [540. 有序数组中的单一元素](https://leetcode.cn/problems/single-element-in-a-sorted-array/)

给你一个仅由整数组成的有序数组，其中每个元素都会出现两次，唯有一个数只会出现一次。

请你找出并返回只出现一次的那个数。

你设计的解决方案必须满足 `O(log n)` 时间复杂度和 `O(1)` 空间复杂度。

 

**示例 1:**

```
输入: nums = [1,1,2,3,3,4,4,8,8]
输出: 2
```

**示例 2:**

```
输入: nums =  [3,3,7,7,10,11,11]
输出: 10
```

```java
public static int singleNonDuplicate(int[] nums) {
        int left=0,right=nums.length-1;
        int mid=0;
        while (left<=right&&right<nums.length) {
            mid = (left+right)/2;
            if(mid==0||mid==nums.length-1||(nums[mid]<nums[mid+1]&&nums[mid]>nums[mid-1])) return nums[mid];
            if(mid%2==0) {//mid是偶数，如果和左边相等，则结果在左边，否则在右边
                if(nums[mid]==nums[mid-1]) {//结果在左边
                    right = mid-2;
                }else if(nums[mid]==nums[mid+1]) {//结果在右边
                    left=mid+2;
                }
            }else { //mid是奇数，如果和左边相等，则结果在右边，否则，结果在左边
                if(nums[mid]==nums[mid-1]) {//结果在右边
                    left=mid+1;
                }else if(nums[mid]==nums[mid+1]) {//结果在左边
                    right=mid-1;
                }
            }
        }
        return 0;
    }
```



#### [4. 寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)

给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数** 。

算法的时间复杂度应该为 `O(log (m+n))` 。

 

**示例 1：**

```
输入：nums1 = [1,3], nums2 = [2]
输出：2.00000
解释：合并数组 = [1,2,3] ，中位数 2
```

**示例 2：**

```
输入：nums1 = [1,2], nums2 = [3,4]
输出：2.50000
解释：合并数组 = [1,2,3,4] ，中位数 (2 + 3) / 2 = 2.5
```

 

 

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        //先把较小的数组放到第一个
        if(nums1.length>nums2.length) {
            int[] temp = nums1;
            nums1 = nums2;
            nums2 = temp;
        }
        int m = nums1.length;
        int n = nums2.length;
        int totalLeft = (m+n+1)/2;//左边一共的个数，也就是数组1与数组2左边数字个数之和
        //寻找第一个数组合适的分割位置
        int left = 0,right=m;
        while (left<right) {
            int i = left + (right-left+1)/2;//第一个数组分割线
            int j = totalLeft-i;//第二个数组的分割线
            if(nums1[i-1]>nums2[j]) {//说明第一个数组的分割线靠右，应该向左移 下一个区间 [left,i-1]
                right= i-1;
            } else {
                left = i;
            }
        }
        int i = left;
        int j = totalLeft - i;
        int maxLeft1 = i==0?Integer.MIN_VALUE:nums1[i-1];
        int maxLeft2 = j==0?Integer.MIN_VALUE:nums2[j-1];
        int minRight1 = i==m?Integer.MAX_VALUE:nums1[i];
        int minRight2 = j==n?Integer.MAX_VALUE:nums2[j];
        return (m+n)%2==1?Math.max(maxLeft1, maxLeft2):(double)(Math.min(minRight1, minRight2)+Math.max(maxLeft1, maxLeft2))/2;
    }
}
```

# 排序

#### 快速排序

```java
public class 快速排序 {
    public static void main(String[] args) {
        int[] nums = new int[]{5,2,7,9,3,5,6,7,2,3};
        // quick_sort(nums, 0, nums.length-1);
        findKthLargest(nums,2);
        
    }

    public static int findKthLargest(int[] nums, int k) {
        quick_sort(nums, 0, nums.length-1);
        System.out.println(nums[k-1]);
        return nums[k];
    }

    //挖坑法
    public static void quick_sort(int[] nums,int left,int right) {
        if(right<=left) return;
        int first = left,last = right;
        int key = nums[left];
        while(first<last) {
            //先从右边找比key小的值
            while(first<last&&nums[last]<=key) last--;
            nums[first] = nums[last];
            //再从左边找比key大的值
            while(first<last&&nums[first]>key) first++;
            nums[last] = nums[first];
        }
        nums[first] = key;
        quick_sort(nums, left, first-1);
        quick_sort(nums, first+1, right);
    }
}
```

#### 归并排序

```java
public class 归并排序 {
    public static void main(String[] args) {
        int[] a =new int[]{3,2,5,6,4,7,8};
        归并排序 排序 =  new 归并排序();
        排序.mergesort(a, 0, a.length-1, new int[a.length]);
        System.out.println(Arrays.toString(a));
    }
    
    void mergesort(int[] a, int first, int last, int[] temp) {
        if(first<last) {
            int mid = (first+last)/2;
            //使前半部分有序
            mergesort(a, first, mid, temp);
            //使后半部分有序
            mergesort(a, mid+1, last, temp);
            //归并
            merge(a,first,mid,last,temp);
        }
    }
    private void merge(int[] a, int first, int mid, int last, int[] temp) {
        if(first==mid&&last==mid) return;
        //合并两个有序数组
        int i = first,j=mid+1;
        int k = 0;//temp数组的索引
        while(i<=mid && j<=last) {
            if(a[i]<a[j]) {
                temp[k++] = a[i++];
            }else {
                temp[k++] = a[j++];
            }
        }
        //补全剩余数组
        while(i<=mid) {
            temp[k++] = a[i++];
        }
        while(j<=last) {
            temp[k++] = a[j++];
        }

        //将temp数组写回原数组
        for(i=0;i<k;i++) {
            a[first+i] = temp[i];
        }
    }
    
}
```

#### [347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)

给你一个整数数组 `nums` 和一个整数 `k` ，请你返回其中出现频率前 `k` 高的元素。你可以按 **任意顺序** 返回答案。

**示例 1:**

```
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

**示例 2:**

```
输入: nums = [1], k = 1
输出: [1]
```

```java
//基于桶排序求解「前 K 个高频元素」
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        List<Integer> tmp = new ArrayList();
        // 使用字典，统计每个元素出现的次数，元素为键，元素出现的次数为值
        HashMap<Integer,Integer> map = new HashMap();
        for(int num : nums){
            if (map.containsKey(num)) {
               map.put(num, map.get(num) + 1);
             } else {
                map.put(num, 1);
             }
        }
        
        //桶排序
        //将频率作为数组下标，对于出现频率不同的数字集合，存入对应的数组下标
        List<Integer>[] list = new List[nums.length+1];
        for(int key : map.keySet()){
            // 获取出现的次数作为下标
            int i = map.get(key);
            if(list[i] == null){
               list[i] = new ArrayList();
            } 
            list[i].add(key);
        }
        
        // 倒序遍历数组获取出现顺序从大到小的排列
        for(int i = list.length - 1;i >= 0 && tmp.size() < k;i--){
            if(list[i] == null) continue;
            tmp.addAll(list[i]);
        }
        int[] res = tmp.stream().mapToInt(Integer::valueOf).toArray();
        return res;
    }
}

```

#### [451. 根据字符出现频率排序](https://leetcode.cn/problems/sort-characters-by-frequency/)

```java
class Solution {
    public static String frequencySort(String s) {
        //用于存放字符以及出现次数
        Map<Character,Integer> map = new HashMap<>();
        List<Character>[] list = new List[s.length()+1];
        StringBuilder stringBuilder = new StringBuilder();
        int i=0;
        for (i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            if(map.get(ch)==null) {
                map.put(ch, 1);
            }else {
                int num = map.get(s.charAt(i));
                map.put(s.charAt(i), ++num);
            }  
        }
       
        for(char ch : map.keySet()) {
            int num = map.get(ch);
            if(list[num]==null) list[num] = new ArrayList<>();
            list[num].add(ch);
        }
        for (i = s.length(); i >=0 ; i--) {
            if(list[i]==null) continue;
            int tmp = i;
            list[i].forEach(item->{
                for(int j=0;j<tmp;j++)
                stringBuilder.append(item);
            });
        }
        System.out.println(stringBuilder);
        return stringBuilder.toString();
    }
}
```

