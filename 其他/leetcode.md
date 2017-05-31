# leetcode解题思路
## 435(Non overlapping Intervals)
### Brute Force
将Intervals按start point从小到大排序，然后从左到右处理Interval，
针对每个Interval，有两种情况，属于最终的Interval或者不属于，对这
两种情况分别讨论，选择最小移除数的那种情况。处理完一个，再处理下一个。

复杂度$$O(2^n)$$，会超时。

### 贪心算法
也是先排序，从小到大处理每个Interval。针对某个Interval，和下一个Interval有如图三种关系：

![相交情况][10]

很显然，前两种情况是保留第一个Interval。主要是看最后一种情况。
蓝色的Interval对后续选择的只是相交的那一段，而绿色Interval还包
含后一段，也就是说与前者相交的Interval也必然与后者相交，这种情
况下自然要选择前者。

## 406(Queue Reconstruction by Height)
### 问题重述
现有一个未知序列$$q = \{a_1, a_2, ..., a_n\}$$，数字$$a_n$$的顺序未知，
但已知它的数值以及排在它前面比它大(或者等于)的数字的个数，通过这些已知信息求出原来
的序列q。

输入$$r = \{(x_1, c_1), (x_2, c_2), ..., (x_n, c_n)\}$$，$$x_n$$是q中的数字，$$c_n$$是在$$x_n$$前面并且比它大的数字的个数。求出q。

### 思路
最开始有n个空位置，每次从剩余的数字中挑出最小的给它安排位置。因为每个空位
都将排比它大的数，因此从左到右数够空位个数即可得到该数字在q中的原始位置。

### 代码
```cpp
class Solution {
public:
    static bool cmp(pair<int, int> &a, pair<int, int> &b) {
        if(a.first < b.first) return true;
        else if(a.first > b.first) return false;
        //当大小一样的时候，个数大的排在前面，因为这样在数个数的时候可以把
        //和它相等的数字算进去，后面的代码就简单一些
        else return a.second > b.second;
    }
public:
    vector<pair<int, int>> reconstructQueue(vector<pair<int, int>>& people) {
        sort(people.begin(), people.end(), cmp);
        vector<int> index(people.size(), -1);
        for(int i = 0; i < people.size(); i++) {
            int cnt = 0;
            for(int j = 0; j < people.size(); j++) {
                //个数够了，但不要取已占用的位置  
                if(cnt == people[i].second && index[j] < 0) {
                    index[j] = i;
                    break;
                }
                //对空位置计数
                if(index[j] < 0) cnt++;
            }
        }
        vector<pair<int, int>> ret(people.size());
        //恢复原始顺序 
        for(int i = 0;i < ret.size(); i++) {
            ret[i] = people[index[i]];
        }
        return ret;
    }
};
```

## 392(Is Subsequence)
### 问题重述
一个短字符串s(<=100)和一个长字符串t($$\approx 50,000$$)，判断s是否是t的子串，也就是说是否可以通过剔除
s中的某些字符得到t。

### 思路
将s和t从右向左对齐，对于s中最右边未对齐的字符s[i]，在t中找到右边第一个与s[i]相等的字符t[j]。
其中的逻辑在于：假设存在t[q] == s[i]且q < j使得t和s剩余的部分完全对齐，那么把q改成j仍然是
后t和s仍然是对齐的。也就是说每次找最右边的相等字符对齐是整个对齐的必要条件。

### 代码

```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int k = t.size() - 1;
        for(int i = s.size() - 1; i >= 0; i--) {
            for(;k >= 0 && t[k] != s[i];k--);
            if(k >= 0) k--;
            else return false;
        }
        return true;
    }
};
```

## 376(Wiggle Subsequence)
### 问题重述
给一个序列，问满足下面条件的最长子序列(和上面的子字符串的定义一样，通过删除某些数字得到)的长度是多少？

对子序列中的每个数字$$q_i$$，满足$$(q_i - q_{i-1}) \time (q_{i+1} - q_i) < 0$$，也就是将序列画在坐标系上呈锯齿状。

### 思路
对于每一段连续上升(或下降)的子序列，中间的数字是不用包含在最终的子序列里面的，因为假设存在一个中间点的最长子序列，
那么把这个点替换成某个端点仍然是最长子序列(如果这个点在波峰，就用连续段的最高点代替；如果在波谷，
就用连续段的最低点代替)。那么只留连续段的端点，得到的就是最长子序列。

### 代码
```cpp
class Solution {
public:
    int wiggleMaxLength(vector<int>& nums) {
         if(nums.size() < 2) return nums.size();
         if(nums.size() == 2) return 1+(nums[0] != nums[1]);
         vector<int> tmp;
         /*去除前后相等的元素，只保留一个。因为存在连续相等的子序列刚好在波峰或波谷
         的情况，如果不去除，用前面的左右导数相乘的方法就检测不出来*/
         tmp.push_back(nums[0]);
         for(int i = 1;i < nums.size();i++)
            if(nums[i] != nums[i-1]) tmp.push_back(nums[i]);
         if(tmp.size() < 3) return tmp.size();
         int cnt = 0;
         for(int i = 1; i < tmp.size()-1;i++) {
             int dl = tmp[i] - tmp[i-1];
             int dr = tmp[i+1] - tmp[i];
             if(dl*dr < 0) cnt++;
         }
         if(cnt == 0) return 1+(tmp.front() != tmp.back());
         return cnt+2;
    }
};
```

## 502(IPO)*
### 问题重述
有一系列项目，每个项目有两个属性$$P_i$$(完成项目获得的利润)和$$C_i$$(启动这个项目最低要有多少资金，
**注意**并不是需要花费这么多资金，而是一个资格，你的资金大于等于$$C_i$$就有资格启动这个项目)。
现在你的初始资金是W，问在最多选择k个项目的限制下，怎样选择项目才能使获得的最终资金最多？

### 思路
把项目按利润从大到小排序，每次选择满足初始资金要求下利润最大的项目。不过按照这个思路写的
代码超时了，需要改进。

### 代码
```cpp
class Solution {
public:
    static bool ab(pair<int,int> &a, pair<int,int> &b) {
        return a.second > b.second;
    }
    int findMaximizedCapital(int k, int W, vector<int>& Profits, 
                vector<int>& Capital) {
        vector<pair<int,int>> pc;
        for(int i = 0; i < Profits.size(); i++)
            pc.push_back(pair<int,int>(Profits[i], Capital[i]));
        sort(pc.begin(), pc.end(), ab);
        vector<bool> flag(pc.size(), true);
        int mc = W;
        for(int i = 0; i < k; i++) {
            int j = 0;
            for(; j < pc.size() && (pc[j].second > mc || !flag[j]);j++);
            if(j < pc.size()) {
                mc += pc[j].first;
                flag[j] = false;
            }
            else break;
        }
        return mc;
    }
};
```

## 55(Jump Game)
### 问题重述
有一个数组，每个元素是个非负数，表示从当前位置向前跳的最大距离，比如a[3]=2，
就可以从位置3最远跳到位置5。任意给个这样的数组，问能否从位置0跳到最后一个位置。

### 思路
对于某一位置t，在t的左边可以跳至t且离t最近的位置为n，若还存在可以跳至t且在n左边的位置x，
那么x一定可以跳至n，也就是说若存在从x跳至t的合格路径，那么也一定存在对应一条从n跳至t的路径。
所以只需要取n即可。从最后的位置，用这种方式往前跳，如果能跳到0位置，说明可以，否则不可以。

### 代码
```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int lastj = nums.size() - 1;
        while(lastj > 0) {
            bool flag = true;
            for(int j = lastj-1; j >= 0; j--){
                if(lastj - j <= nums[j]) {
                    lastj = j;
                    flag = false;
                    break;
                }
            }
            if(flag) break;
        }
        return lastj == 0;
    }
};
```

## 402(Remove K Digits)
### 问题重述
给一个10进制的数字，移除其中的K位使得剩下的数字表示的数最小。

### 思路
也就是选其中的S-K位(S是原数字的位数)使得所表示的数字最小。从最高位开始选，在保证位数够
的情况下，选择最小的数字，如果有多个最小的，选择最左边的。因为同样位数的数字，高位小的
肯定最小。有多个数字可选的话，选最左边，使得剩余位的可选择性最大。

### 代码
```cpp
class Solution {
public:
    string removeKdigits(string num, int k) {
        int len = num.size() - k;
        if(len == 0) return "0";
        string res(len, '0');
        int leftm = -1;
        for(int i = 0; i < len; i++){
            int es = k + i;
            int minv = num[es];
            int mini = es;
            for(int j = es - 1; j > leftm; j--){
                if(num[j] <= minv) {
                    minv = num[j];
                    mini = j;
                }
            }
            leftm = mini;
            res[i] = num[mini];
        }
        int s = 0;
        while(res[s] == '0') s++;
        if(s == res.size()) return "0";
        return res.substr(s);
    }
};
```

## 134(Gas Station)
### 问题重述
在一条环形的路上有N个加油站，每个加油站的储油量为gas[i]，并且从i号加油站到(i+1)%N号加油站耗油量为
cost[i]。假设你开着一辆油箱容量无限(最开始油量为0)的汽车从某个加油站出发。给定gas和cost两个加油站配置数组，
问从哪号加油站出发可保证汽车开一圈回到出发点(配置数组保证至多有一个这样的加油站，不存在返回-1)？

### 思路
基本思路很简单，就是一个个试，算法的复杂度为![O(n^2)][12]。不过超时了，后来这个基础上改进了。从i号加油站出发，
当行驶的到j号加油站发现无法继续行驶时，那么[i,j]这个区间的加油站就不用测试了，一定也不能绕一圈回到出发点，因为
从[i,j]中间任意一点出发到达j时，油箱的储油量一定不会大于从i出发到达j时的储油量。利用这个规律可以省掉很多不必要的测试点。
此时算法的复杂度为O(n)。

### 代码
```cpp
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        vector<int> avl(gas.size());
        for(int i = 0; i < avl.size(); i++)
            avl[i] = gas[i] - cost[i];
        int index = -1;
        for(int i=0; i < avl.size(); ){
            int sum = 0;
            bool flag = true;
            for(int j = 0; j < avl.size();j++) {
                int ix = (i+j)%avl.size();
                sum += avl[ix];
                if(sum < 0) {
                    flag = false;
                    if(ix+1 <= i) i = avl.size();
                    else i = ix+1;
                    break;
                }
            }
            if(flag){
                index = i;
                break;
            }
        }
        return index;
    }
};

## 53(Maximum Subarray)
### 问题重述
给一个整数数组A，找出它的连续子数组的最大和，即$$max \sum_{i=k}^n A[i], 0 \le k \le n \lt len(A)$$。

### 思路
动态规划的方法。设S[k]是以A[k]结尾的连续子数组的最大和。那么有

$$S[k] = \begin{case}
S[0] = A[0]&
S[k] = S[k-1] + A[k] if S[k-1] \gt 0
S[k] = A[k] else
\end{case}$$

最后，找出S[k]中的最大值即可。

### 代码
```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        if(nums.size() <= 1) return nums[0];
        vector<int> tmp = nums;
        for(int i = 1; i < tmp.size(); i++) {
            if(tmp[i-1] <= 0) continue;
            tmp[i] += tmp[i-1];
        }
        return *max_element(tmp.begin(), tmp.end());
    }
};
```
