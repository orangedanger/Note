# 面试LeetCode热题100道

## 1.[两数之和](https://leetcode.cn/problems/two-sum/)(HashMap)

[1. 两数之和](https://leetcode.cn/problems/two-sum/)

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。



哈希表，通过将每个数据存入map中每次获得新数据时查看map中是否有可以匹配的答案
`hashMap.find(target - nums[i])`

因为存入数据时将这个数值当作key即可快速查找

最后通过获得对应的value即可查找

`int val = hashMap[target - nums[i]];`

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) 
    {
        unordered_map<int, int> hashMap;
        hashMap.insert({ nums[0], 0 });
        for (int i = 1; i < nums.size(); ++i)
        {
	
            if (hashMap.find(target - nums[i]) == hashMap.end())
            {
		        //没有在哈希表中找到,则将该元素插入哈希表
                hashMap.insert({ nums[i], i });
            }
            else
            {
                //在哈希表中找到即输出
                int val = hashMap[target - nums[i]];
                return { val, i };
            }       
        }
        return {};
    }
};
```

## [49.字母异位词分组](https://leetcode.cn/problems/group-anagrams/)(HashMap)

给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。





需要先对string进行排序后通过HashMap将key值相同的放入一个Hash表中即可得到答案

```c++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) 
	{
		//先将strs中的string进行排序则自然相同的在一块
		vector<vector<string>> res;
		unordered_map<string, vector<string>> hashMap;

		for (string& s : strs)
		{
			string key = s;
			sort(key.begin(), key.end());
			hashMap[key].push_back(s);
		}
		for(auto& p : hashMap)
		{
			res.push_back(p.second);
		}

		return res;
    }
};
```

## [128.最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。



先排序我加入了Set中去除相同，但似乎Set的inert操作过于消耗时间

然后对Set进行遍历一遍即可

```c++
class Solution {
public:
	int longestConsecutive(vector<int>& nums) 
	{	
		set<int> numSet;
		numSet.insert(nums.begin(), nums.end());

        if (numSet.size() == 0)
	        return 0;

		int MaxLen = 1;
		int currentLen = 1;

		for (auto it = numSet.begin(); it != --numSet.end();)
		{
			if ((*it) + 1 == *(++it))
			{
				MaxLen = max(MaxLen, ++currentLen);
			}
			else
			{
				currentLen = 1;
			}

		}
		return MaxLen;
	}
};
```

## [283.移动零](https://leetcode.cn/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。



第一次写

遍历有0删除，最后一起加

```c++
class Solution {
public:
	void moveZeroes(vector<int>& nums) 
	{
		//记有多少个0
		int count = 0;
		for (int i = 0; i < nums.size(); ++i)
		{
			if (nums[i] == 0)
			{
				nums.erase(nums.begin() + i);
				--i;
				++count;
			}
		}

		for (int i = 0; i < count; i++)
		{
			nums.push_back(0);
		}
	}
};
```

看完题解

双指针

```c++
class Solution {
public:
	void moveZeroes(vector<int>& nums) 
	{
		int slow = 0;
		for (int fast = 0; fast < nums.size(); fast++)
		{
			if (nums[fast] != 0)
			{
				swap(nums[fast], nums[slow]);
				++slow;

			}
		}
	}
};
```

## [11.盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)(双指针)

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明：**你不能倾斜容器。



双指针

要先分析题目开始在左右2边，开始变化每次只变一边情况下，变更小的那边是更有利的因为变长的那边无论下一个是加长还是减少体积都会变小所以

 `height[l] > height[r] ? r-- : l++;`

每次变化取最大体积即可

```c++
class Solution {
public:
    int maxArea(vector<int>& height) {
        int l, r;

        l = 0;
		r = height.size() - 1;

		int res = r * min(height[l], height[r]);
        int cur = 0;
        while (l < r)
        {
            height[l] > height[r] ? r-- : l++;
            cur = (r - l) * min(height[l], height[r]);
			res = max(res, cur);
        }
        return res;
    }
};

```

## [15. 三数之和](https://leetcode.cn/problems/3sum/)(排序+双指针)

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。



排序+双指针

```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        //思路 先用2个的和，变成 2数求和
		//双指针，先对数组排序，外层循环依次遍历整个数组，内层循环通过双指针左边2个数之和与最右边的数加一起看是否为0，如果大于零说明数值偏大右指针左移，如果小于零说明数值偏小左指针右移，一次循环直到左右指针相遇

		sort(nums.begin(), nums.end());
        vector<vector<int>> res;

        for (int i = 0; i < nums.size()-2; i++)
        {
            //避免重复
            if (i&&nums[i] == nums[i - 1])
            {
                continue;
            }
            int l = i + 1;
            int r = nums.size() - 1;
            while (l < r)
            {
                if (nums[i] + nums[l] + nums[r] > 0)
                {
                    r--;
                }
                else if (nums[i] + nums[l] + nums[r] < 0)
                {
                    l++;
                }
                else if (nums[i] + nums[l] + nums[r] == 0)
                {
                    res.push_back({ nums[i],nums[l],nums[r] });
                    //去除重复三元组
                    //为什么从外面移动到内部 因为前2个if时就算重复也不需要计入res中无关结果,执行次数相同
                    for (l++; l < r && nums[l] == nums[l - 1]; ++l);
                    for (r--; l < r && nums[r] == nums[r + 1]; --r);
                }   
            }
        }
        return res;
    }
};
```

## [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/)

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

 

自己想的

双指针逐层寻找

```c++
class Solution {
public:
	int trap(vector<int>& height)
	{
        //思路逐层寻找
        //获取每层面积减去总面积即可
		int l, r;
		int res = 0;
		int sum = 0;
		//获取最大高度
		int maxHeight = 0;
		for (int i = 0; i < height.size(); ++i)
		{
			maxHeight = max(height[i], maxHeight);
			sum += height[i];
		}

		for (int i = 0; i < maxHeight; ++i)
		{
			l = 0;
			r = height.size() - 1;

			//左右指针找边界
			for (; l < r && (height[l]<= i); ++l);
			for (; l < r && (height[r]<= i); --r);

			res += (r - l + 1);
		}

		return  res -= sum;
	}
};
```



思路竖行来看

此处i接水等于遍历此处左边的max_right和右边的max_left

`min(max_right,max_left)-height[i]`

从左到右进行遍历相加即是答案

`ans+=min(max_right,max_left)-height[i]`

动态规划

通过提前获得每个位置的`max_right`,`max_left`即可一次遍历完成

```c++
class Solution {
public:
	int trap(vector<int>& height)
	{
		int ans = 0;
		vector<int> right_arr;

		right_arr.resize(height.size());

		int temp = 0;
		for (int i = height.size() - 1; i >= 0; i--)
		{
			temp = max(temp, height[i]);
			right_arr[i] = temp;
		}
		temp = 0;
        
		for (int i = 0; i < height.size(); i++)
		{
            temp = max(temp, height[i]);
			ans += min(right_arr[i], temp) - height[i];
		}
	
		return ans;
	}
};

```



单调栈

通过判断i所在的高度和栈顶高度进行判断是否可以储存水
如果i所在高度高则一定能储存水

因为top的前一个栈元素一定比top大(因为比top小的都pop出去进入ans的计算了)

所以pop栈顶元素，与新的top计算高度差和宽度得到储水量

```c++
class Solution {
public:
	int trap(vector<int>& height)
	{
		stack<int> stack;
		int ans = 0;;

		for (int i = 0; i < height.size(); i++)
		{
			while (!stack.empty()&& height[stack.top()]< height[i])
			{
				int temp = stack.top();
				stack.pop();
				if (stack.empty())
					break;
				ans += (min(height[i], height[stack.top()]) - height[temp]) * (i - stack.top() - 1);
			}		
			stack.push(i);
		}

		return ans;
	}
};
```

## [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长 子串** 的长度。



第一次思路

HashMap

通过在HashMap中查找来判断

如果出现则

`i = hashMap[s[i]];`

来实现回到上个重复字母前来重新寻找

```c++
class Solution {
public:
	int lengthOfLongestSubstring(string s) {
		//不能出现重复的
		//怎么判断是否重复
		//存入hashmap
		int ans = 0;
		int temp = 0;
		unordered_map<char, int> hashMap;
		for (int i = 0; i < s.size(); ++i)
		{
			if (!hashMap.count(s[i]))
			{
				temp++;
				hashMap.insert({ s[i],i });
			}
			else
			{
				ans = max(ans, temp);
				//重新初始化
				i = hashMap[s[i]];
				hashMap.clear();
				temp = 0;
			}
		}
		//最后没有重复的需要再进行一次最大值比较
		ans = max(ans, temp);
		return ans;
	}
};
```



看题解新思路

由于HashMap的删除是O(n)导致时间复杂度是O(n^2)

所以需要不变动HashMap

`hashMap[c] >= left`即找到需要判断是否在更新后坐标后

`hashMap[c] = right;`更新新坐标

滑动窗口

```c++
class Solution {
public:
	int lengthOfLongestSubstring(string s) {
		//不能出现重复的
		//怎么判断是否重复
		//存入hashmap

		//新思路 由于HashMap的删除为O(n) 所以采用左指针来更新重复的字符位置
		int ans = 0;
		int temp = 0;
		unordered_map<char, int> hashMap;
		int left = 0;
		for (int right = 0; right < s.size(); ++right)
		{
			char c = s[right];
			//找到了
			if (hashMap.find(c) != hashMap.end() && hashMap[c] >= left)
			{
				//改变左指针到新位置
				left = hashMap[c] + 1;
			}
			//更新坐标
			hashMap[c] = right;
			ans = max(ans, right - left + 1);
		}
		return ans;
	}
};
```

## [438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)



给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。



想到了滑动窗口

```c++

class Solution {
public:
	vector<int> findAnagrams(string s, string p) {
		//看到思路 滑动窗口
		//1.排序再比较 O(n^2 logn)
		//2.计数
		vector<int> res;
		vector<int> scount(26);
		vector<int> pcount(26);

		//当p的长度小于s时
		if (p.size() > s.size())
		{
			return res;
		}

		//统计p中的字母数
		for (int i =0; i < p.size(); i++)
		{
			++pcount[p[i] - 'a'];
			++scount[s[i] - 'a'];
		}

		if (pcount == scount)
		{
			res.push_back(0);
		}

		for (int i = p.size() ; i < s.size(); i++)
		{
			//他的后面n个需要满足
			//怎么删除第一个

			--scount[s[i - p.size()] - 'a'];
			++scount[s[i] - 'a'];

			if (pcount == scount)
			{
				res.push_back(i - p.size() + 1);
			}
		}
		return res;
	}
};
```

## [560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 *该数组中和为 `k` 的子数组的个数* 。

子数组是数组中元素的连续非空序列。





刚开始想到滑动窗口，但是滑动窗口只能在当窗口变化时结果呈现单调变化，这题出现负数变化不单调故不可滑动

通过暴力解O(n^2)

```c++
class Solution {
public:
	int subarraySum(vector<int>& nums, int k) {
		
		int ans = 0;

		//硬算
		for (int i = 0; i < nums.size(); i++)
		{
			int sum = 0;
			for (int j = i; j < nums.size(); j++)
			{
				sum += nums[j];
				if (sum == k)
				{
					ans++;
				}
			}
		}
		return ans;
	}
};
```

时间复杂度过高不行



看了题解因为i~j的这段数的和可以通过

`sum_arr[j]-sum_arr[i]`来求出

则有公式当 `k == sum_arr[j]-sum_arr[i]`时满足条件

简单变化一下 `sum_arr[i] == sum_arr[j] - k`

因为 i < j

所以我们可以在之前就将`sum_arr[i]`存储起来,当求到`sum_arr[j]`时再寻找`sum_arr[i]`的数量

所以用到HashMap

```c++
class Solution {
public:
	int subarraySum(vector<int>& nums, int k) {
		
		int ans = 0;
		unordered_map<int, int> hashMap;
		hashMap[0]++;
		int sum = 0;
		for (int i = 0; i < nums.size(); i++)
		{
			sum += nums[i];
			if (hashMap.find(sum-k) != hashMap.end())
			{
				ans += hashMap[sum - k];
			}
			hashMap[sum]++;
		}
		
		return ans;
	}
};
```

## [239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。



通过使用最大堆和最小堆的排序为O(logn)

来不断添加数据和删除

有一点技巧是在删除数据时可以先将当前数据加入，如果队列中最大的数据不在窗口中再进行删除可以节省一点时间

优先队列

```c++
class Solution {
public:
	vector<int> maxSlidingWindow(vector<int>& nums, int k) {
		priority_queue<pair<int,int>> q;
		vector<int> res;
		for (int i = 0; i < k; ++i)
		{
			q.push({ nums[i],i });
		}
		//加入最大的数
		res.push_back(q.top().first);
		for (int i = k; i < nums.size(); ++i)
		{
            //加入新来的数
			q.push({ nums[i],i });
			//移除最大的数直到最大的数是当前窗口中存在的
			while(q.top().second < i - k +1)
			{
				q.pop();
			}
			//加入最大的数
			res.push_back(q.top().first);
		}
		
		return res;
	}
};
```



单调队列

使用了双向队列deque

当需要存储数组的数值时，数组不发生变化时可以考虑存储数组的下标

我们应该从后往前考虑删除比当前数据小的值

```c++
class Solution {
public:
	vector<int> maxSlidingWindow(vector<int>& nums, int k) {
		vector<int> res;
		deque <int> q;

		for (int i = 0; i < k; i++)
		{
			while (!q.empty()&&nums[i] > nums[q.back()])
			{
				q.pop_back();
			}
			q.push_back(i);
		}

		res.push_back(nums[q.front()]);

		for (int i = k; i < nums.size(); i++)
		{
			while (!q.empty() && nums[i] > nums[q.back()])
			{
				q.pop_back();
			}
			q.push_back(i);
			while (q.front() < i - k + 1)
			{
				q.pop_front();
			}

			res.push_back(nums[q.front()]);
		}
		return res;
	}
};
```



## [76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。



用滑动窗口当窗口内不满足条件时r向右滑动
直到满足条件时

l指针向左滑动并不断判断条件是否满足



自己写的时候发现仍然超时最后把map的定义放在外面，减少了调用函数时变量的反复赋值时间

```c++
class Solution {
public:
	unordered_map<char, int> s_Map, t_Map;

	bool check()
	{
		for (auto& p : t_Map)
		{
			if (s_Map[p.first] < p.second)
			{
				return false;
			}
		}
		return true;
	}

	string minWindow(string s, string t) {

		if (s.size() < t.size())
		{
			return "";
		}

		string res = s;

		//统计t中字符出现数量
		for (const auto& c:t)
		{
			++t_Map[c];
		}

		int l = 0, r = -1;
		int len = INT_MAX;
		int ansL = 0;

		while (r < (int)s.size())
		{
			if (t_Map.find(s[++r]) != t_Map.end())
            {
	            ++s_Map[s[r]];
            }
			while(check()&&l<=r)
			{
				//当前窗口含有t的所有字符
				if (r - l +1< len)
				{
					//更新长度
					len = r - l +1;
					ansL = l;
				}
               if (t_Map.find(s[l]) != t_Map.end())
               {
                    --s_Map[s[l]];
               }
				++l;
			}	
		}
		return len>s.size()?string():s.substr(ansL, len);
	}
};
```

发给ai时ai告诉我由于每次检测都要完整检测map过于耗时，所以换成了

match和required

match是当 t_Map[s[r]]&&s_Map[s[r]] == t_Map[s[r]] 就完成一个匹配

即当前字符在t_Map中出现且s_Map中出现的字符数和t_Map中出现的数相等

就完成了一个字母的匹配

当所有字母都完成匹配即match == required 说明可以进行l的计算和左窗口进行左移了

```c++
class Solution {
public:
    string minWindow(string s, string t) {
        if (s.size() < t.size()) {
            return "";
        }

        unordered_map<char, int> t_Map, window;
        
        // 统计t中字符出现数量
        for (const auto& c : t) {
            ++t_Map[c];
        }

        int left = 0, right = 0;
        int minLen = INT_MAX;
        int start = 0;
        int matched = 0; // 记录已经匹配的字符种类数
        int required = t_Map.size(); // 需要匹配的字符种类数

        while (right < s.size()) {
            char c = s[right];
            window[c]++;
            
            // 如果当前字符在t中，且出现次数达到要求
            if (t_Map.count(c) && window[c] == t_Map[c]) {
                matched++;
            }

            // 当窗口包含t的所有字符时，尝试收缩左边界
            while (left <= right && matched == required) {
                // 更新最小窗口
                if (right - left + 1 < minLen) {
                    minLen = right - left + 1;
                    start = left;
                }

                // 收缩左边界
                char leftChar = s[left];
                window[leftChar]--;
                
                // 如果移除了t中的字符，且数量不再满足要求
                if (t_Map.count(leftChar) && window[leftChar] < t_Map[leftChar]) {
                    matched--;
                }
                
                left++;
            }
            
            right++;
        }

        return minLen == INT_MAX ? "" : s.substr(start, minLen);
    }
};
```

## [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组**是数组中的一个连续部分。



贪心算法

每次移动到下一个考虑上一次的和如果小于0则丢弃从这开始计算

```c++
class Solution {
public:
	int maxSubArray(vector<int>& nums) {
		int sum = nums[0];
		int ans = nums[0];
		for (int i = 1; i < nums.size(); ++i)
		{
			if (sum < 0)
			{
				sum = nums[i];
			}
			else
			{
				sum += nums[i];
			}
			ans = max(ans, sum);
		}
		return ans;
	}
};
```

动态规划

 每次考虑

```c++
class Solution {
public:
	int maxSubArray(vector<int>& nums) {
		for (int i = 1;i<nums.size();++i)
		{
            //dp公式
			nums[i] = max(nums[i], nums[i] + nums[i-1]);
		}
        //计算范围最大值
		return ranges::max(nums);
	}
};
```



前缀和+贪心

通过计算前缀和将问题转换为[121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)通过贪心解决即可

```c++
class Solution {
public:
	int maxSubArray(vector<int>& nums) {
		vector<int> arr(nums.size() + 1);
		int minVal = INT_MAX;
		arr[0] = 0;
		int ans = INT_MIN;
		for (int i = 0; i < nums.size(); ++i)
		{
			arr[i+1] = arr[i] + nums[i];
			minVal = min(minVal, arr[i]);
			ans = max(ans, arr[i + 1] - minVal);
		}
		return ans;
	}
};
```





## [56. 合并区间](https://leetcode.cn/problems/merge-intervals/)

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。



排序

```c++
class Solution {
public:
	vector<vector<int>> merge(vector<vector<int>>& intervals) {
		//不是有序的可以先对first进行排序
		//判断[i].first 和 max_second 比较
		//如果重合
		//登记最初的.first 和 Max_second当作新的起点
		//如果不重合,res记录这个first和Max_second
		//将下一个的first和second 记录并遍历到结尾
		vector<vector<int>> res;

		sort(intervals.begin(), intervals.end());

		int l = intervals[0][0];
		int max_second = intervals[0][1];
		for (int i = 1; i < intervals.size(); i++)
		{
			if (intervals[i][0] <= max_second)
			{
				//说明在范围内
				//更新max_second
				max_second = max(max_second, intervals[i][1]);
			}
			else
			{
				//不在范围内
				//结果加入res
				res.push_back({l,max_second});
				//更新l和max_second
				l = intervals[i][0];
				max_second = intervals[i][1];
			}
		}
		res.push_back({ l,max_second });
		return res;
	}
};
```

## [189. 轮转数组](https://leetcode.cn/problems/rotate-array/)

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

 

单调队列双向插入

原本的时间复杂度O(n+k)

在Ai提醒下当k大于n时他会回到原位
所有我们可以对k进行取模
 `k = k % nums.size();`

这极大减少了k的循环

```c++
class Solution {
public:
	void rotate(vector<int>& nums, int k) {
		deque<int> q;
		for (const int& num : nums)
		{
			q.push_back(num);
		}

		int temp = 0;
		for (int i = 0; i < k; ++i)
		{
			temp = q.back();
			q.pop_back();
			q.push_front(temp);
		}
		return nums.assign(q.begin(), q.end());
	}
};

```

所以我们可以在前k次从前插入后面 k~n从后向前插入
时间复杂度O(n)

```c++
class Solution {
public:
	void rotate(vector<int>& nums, int k) {
		deque<int> q;

		int n = nums.size();
		k = k % nums.size();
		for (int i = 0; i < n; ++i)
		{
			if (i < n - k)
			{
				q.push_back(nums[i]);
			}
			else
			{
				q.push_front(nums[2 * n - k - i - 1]);
			}
		}
		nums.assign(q.begin(), q.end());
	}
};
```

翻转数组

```c++
class Solution {
public:
	void rotate(vector<int>& nums, int k) {
		k = k % nums.size();
		if (k == 0) return;

		//翻转数组
		reverse(nums.begin(), nums.end()); // 7 6 5 4 3 2 1
		reverse(nums.begin(), nums.begin() + k);//翻转前k个 5 6 7 4 3 2 1
		reverse(nums.end() + k - nums.size(), nums.end());//翻转后面 n - k 个 5 6 7 1 2 3 4
	}
};
```

## [238. 除自身以外数组的乘积](https://leetcode.cn/problems/product-of-array-except-self/)

给你一个整数数组 `nums`，返回 数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积 。

题目数据 **保证** 数组 `nums`之中任意元素的全部前缀元素和后缀的乘积都在 **32 位** 整数范围内。

请 **不要使用除法，**且在 `O(n)` 时间复杂度内完成此题。



双向前缀积

```c++
class Solution {
public:
	vector<int> productExceptSelf(vector<int>& nums) {
		//索引 左侧乘积*右侧乘积

		int n = nums.size();
		vector<int> left(n), right(n);
		vector<int> res(n);

		left[0] = 1;
		for (int i = 1; i < nums.size(); i++)
		{
			left[i] = left[i - 1] * nums[i - 1];
		}
		right[n - 1] = 1;
		for (int i = n - 2; i >= 0; i--)
		{
			right[i] = right[i + 1] * nums[i + 1];
		}

		for (int i = 0; i < nums.size(); i++)
		{
			res[i] = left[i] * right[i];
		}

		return res;
	}
};
```

## [41. 缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/)

给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。

请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。



将下标作为答案,根据负号来判断是否满足条件

```c++
class Solution {
public:
	int firstMissingPositive(vector<int>& nums) {
		int minAns = 1;
		int count = 0;
		int n = nums.size();
		for (int i = 0; i < nums.size(); ++i)
		{
			if (nums[i] <= 0)
			{
				nums[i] = n + 1;
			}
		}

		for (int i = 0; i < nums.size(); ++i)
		{
			if (abs(nums[i]) <= n && nums[abs(nums[i]) - 1] > 0)
			{
				nums[abs(nums[i])-1] *= -1;
			}
		}

		int i = 0;
		for (; i < nums.size(); ++i)
		{
			if (nums[i] > 0 )
			{
				return i + 1;
			}
		}
		return i + 1;
	}
};
```



桶排序

一个萝卜一个坑，你是什么位置的萝卜就去什么位置

注意换坑之后的新萝卜可能也不是这个坑的需要进行循环直到不满足交换条件为止

```c++
class Solution {
public:
	int firstMissingPositive(vector<int>& nums) {
		int n = nums.size();
		for (int i = 0; i < nums.size(); ++i)
		{
			//出现无限循环,添加限制 交换的两数不能相等
			while (nums[i] > 0 && nums[i] <= n && nums[i]!= i + 1 && nums[i]!= nums[nums[i] - 1])
			{
				swap(nums[i], nums[nums[i] - 1]);
			}
		}
		
		int ans = 1;
		for (int i = 0; i < nums.size(); ++i)
		{
			if (nums[i] == i + 1)
			{
				ans++;
			}
			else
			{
				return ans;
			}
		}
		return ans;
	}
};
```

## [73. 矩阵置零](https://leetcode.cn/problems/set-matrix-zeroes/)

给定一个 `m* x *n` 的矩阵，如果一个元素为 **0** ，则将其所在行和列的所有元素都设为 **0** 。请使用 **[原地](http://baike.baidu.com/item/原地算法)** 算法**。**



没有空间就创造空间  借用矩阵的首行和首列

```c++
class Solution {
public:
	void setZeroes(vector<vector<int>>& matrix) {
		//标记
		//拿什么标记，
		//确实空间，我就将matrix的空间当自己的空间用
		//将matrix的首行和首列用来记录是否满足条件

		int n = matrix.size();
		int m = matrix[0].size();
		bool f_col = false, f_row = false;

		//判断首列有没有0
		for (int i = 0; i < m; i++)
		{
			if (matrix[0][i] == 0)
			{
				f_row = true;
				break;
			}
		}
		//判断首行有没有0
		for (int i = 0; i < n; i++)
		{
			if (matrix[i][0] == 0)
			{
				f_col = true;
				break;
			}
		}

		for (int i = 1; i < n; i++)
		{
			for (int j = 1; j < m; j++)
			{
				if (matrix[i][j] == 0)
				{
					matrix[i][0] = 0;
					matrix[0][j] = 0;
				}
			}
		}

		for (int i = 1; i < n; i++)
		{
			for (int j = 1; j < m; j++)
			{
				if (matrix[i][0] == 0 || matrix[0][j] == 0)
				{
					matrix[i][j] = 0;
				}
			}
		}

		if (f_col)
		{
			for (int i = 0; i < n; i++)
			{
				matrix[i][0] = 0;
			}
		}

		if (f_row)
		{
			//判断首行有没有0
			for (int i = 0; i < m; i++)
			{
				matrix[0][i] = 0;
			}
		}
	}
};
```

## [54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。



```c++
class Solution {
public:
	vector<int> spiralOrder(vector<vector<int>>& matrix) {
		//一共有4步		i++				j++				i--			j--
		//减少/增加到	n-count+1		m-count+1		count-1		count
		//				1 2 3           6 9				8 7			4
		//				5    

		int m = matrix.size();
		int n = matrix[0].size();
		//循环次数
		int count = 1;
		vector<int> res;
		int i = 0, j = 0;
		while (1)
		{
			for (; i < n - count + 1; i++)
			{
				res.push_back(matrix[j][i]);
			}
			if (res.size() == m * n)
			{
				break;
			}
			i--;
			j++;

			for (; j < m - count + 1; j++)
			{
				res.push_back(matrix[j][i]);
			}
			if (res.size() == m * n)
			{
				break;
			}
			j--;
			i--;
			for (; i >= count -1; i--)
			{
				res.push_back(matrix[j][i]);
			}
			if (res.size() == m * n)
			{
				break;
			}
			i++;
			j--;
			for (; j >= count; j--)
			{
				res.push_back(matrix[j][i]);
			}
			if (res.size() == m * n)
			{
				break;
			}
			j++;
			i++;
			count++;
		}
		return res;
	}
};
```



模拟方向，设置一个表示方向的变量每次判断下次前进方向如果数组越界或者该位置被访问就调整方向一直循环

```c++
class Solution {
public:
	vector<int> spiralOrder(vector<vector<int>>& matrix) {
		int m = matrix.size();
		int n = matrix[0].size();


		int direction = 0; //右: 0 ,下: 1 ,左: 2,上: 3
		vector<vector<int>> visits(m, vector<int>(n , 0)); //开辟一个空间存放是否访问
		vector<int> res;
		int i = 0, j = 0;
		while (res.size() < m * n)
		{
			//标记访问
			visits[i][j] = 1;
			res.push_back(matrix[i][j]);


			switch (direction)
			{
			case 0:
				if (j == n - 1 || visits[i][j + 1] == 1)
				{
					direction = (direction + 1) % 4;
				}
				break;
			case 1:
				if (i == m - 1 || visits[i + 1][j] == 1)
				{
					direction = (direction + 1) % 4;
				}
				break;
			case 2:
				if (j == 0 || visits[i][j - 1] == 1)
				{
					direction = (direction + 1) % 4;
				}
				break;
			case 3:
				if (i == 0 || visits[i - 1][j] == 1)
				{
					direction = (direction + 1) % 4;
				}
				break;
			}

			switch (direction)
			{
			case 0:
				j++;
				break;
			case 1:
				i++;
				break;
			case 2:
				j--;
				break;
			case 3:
				i--;
				break;
			}
		}
		return res;
	}
};

```

## [48. 旋转图像](https://leetcode.cn/problems/rotate-image/)

给定一个 *n* × *n* 的二维矩阵 `matrix` 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在**[ 原地](https://baike.baidu.com/item/原地算法)** 旋转图像，这意味着你需要直接修改输入的二维矩阵。**请不要** 使用另一个矩阵来旋转图像。





```c++
class Solution {
public:
	void rotate(vector<vector<int>>& matrix) {
		//matrix[x][y]->nums[y][n-x];
		//分块

		/*x y		y n-x
		y n-x	n-x n-y
		n-x n-y	n-y	x
		n-y x	x y*/

		int n = matrix.size();

		for (int i = 0; i < (n + 1)/2; i++)
		{
			for (int j = 0; j < n/2; j++)
			{
				int temp = matrix[i][j];
				matrix[i][j] = matrix[n - j - 1][i];
				matrix[n - j - 1][i] = matrix[n - i - 1][n - j - 1];
				matrix[n - i - 1][n - j - 1] = matrix[j][n - i - 1];
				matrix[j][n - i - 1] = temp;
			}
		}
	}
};
```





## [240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)

编写一个高效的算法来搜索 `*m* x *n*` 矩阵 `matrix` 中的一个目标值 `target` 。该矩阵具有以下特性：

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。



```c++
class Solution {
public:
	bool searchMatrix(vector<vector<int>>& matrix, int target) 
	{
		int m = matrix.size();
		int n = matrix[0].size();

		int j = 0;
		for (; j < n; ++j)
		{
			//横向遍历
			//碰到大于的数或者边界时向下遍历
			if (matrix[0][j] >= target || j == n - 1)
			{
				break;
			}
		}

		//向下遍历
		//当 j ==0时还不满足 return false
		for (int i = 0; i < m; ++i)
		{
			//每次向下移动都需要进行判断
			if (matrix[i][j] == target)
			{
				return true;
			}

			if (i + 1 == m)
			{
				return false;
			}

			//碰到大于的数 j-- 同时while(直到 matrix[i+1][j]小于)
			while (matrix[i + 1][j] > target )
			{
				j--;
				if (j == -1)
				{
					return false;
				}
			}
		}
        return false;
    }
};
```



二分查找

由于每行都排了序，所以可以每行都执行二分查找
lower_bound() O(logn)

遍历每行O(mlogn)

```c++
class Solution {
public:
	bool searchMatrix(vector<vector<int>>& matrix, int target)
	{
		int m = matrix.size();
		int n = matrix[0].size();

		for (const auto& arr : matrix)
		{
			//返回第一个大于等于target的迭代器
			auto it = lower_bound(arr.begin(), arr.end(), target);
			if (it != arr.end()&&*it == target)
			{
				return true;
			}
		}
		return false;
	}
};
```



## [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

给你两个单链表的头节点 `headA` 和 `headB` ，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 `null` 。

图示两个链表在节点 `c1` 开始相交**：**



```c++
class Solution 
{
public:
	ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) 
    {
		unordered_map<ListNode*,int> HashMap;

		for (ListNode* it = headA; it!= nullptr; it = it->next)
		{
			HashMap.insert({ it,it->val });
		}

		for (ListNode* it = headB; it != nullptr; it = it->next)
		{
			if (HashMap.find(it) != HashMap.end())
			{
				return it;
			}
		}
        return nullptr;
	}
};
```

## [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。





递归

```c++
class Solution {
public:

    ListNode* temp;

    ListNode* reverseList(ListNode* head) 
    {
        if(head == nullptr || head->next == nullptr)
        {
            return head;
        }
        temp = head->next;
        if (head->next->next != nullptr)
        {
            temp = head->next->next;
            //先递归到最后
            reverseList(head->next);       
        }
        //此时的head在最后
        head->next->next = head;
        head->next = nullptr;
        return temp;
    }
};
```

迭代

```c++
class Solution {
public:

   
    ListNode* reverseList(ListNode* head)
    {
        ListNode* prev = nullptr;
        ListNode* curr = head;

       while(curr)
        {
           ListNode* next = curr->next; // 2                            3                   nullptr
            curr->next = prev;          // 1 -> nullptr                 2 -> 1              3 -> 2
            prev = curr;                // nullptr - - 1                1 - - 2             2 - - 3
            curr = next;                // 1 - - 2                      2 - - 3             3 - - nullptr
        }
       return prev;
    }
};
```

## [234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/)

给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false` 。



数组记录

```c++
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        vector<int> arr;
        ListNode* temp = head;
        while(temp)
        {
            arr.push_back(temp->val);
            temp = temp->next;
        }

        temp = head;
        for (int i = 0; i < arr.size() / 2 ; ++i)
        {
            if (temp->val != arr[arr.size() - 1 - i])
            {
                return false;
            }
            temp = temp->next;
        }
        return true;
    }
};
```

## [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。

*如果链表中存在环* ，则返回 `true` 。 否则，返回 `false` 。

```c++
class Solution {
public:
    bool hasCycle(ListNode* head) {
        unordered_set<ListNode*> hashSet;
        
        while (head)
        {
            if (hashSet.find(head) == hashSet.end())
            {
                hashSet.insert({head});
            }
            else
            {
                return true;
            }
            head = head->next;
        }
        return false;
    }
};
```

## [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)



```c++
class Solution {
public:
    ListNode* detectCycle(ListNode* head) {
        unordered_set<ListNode*> hashSet;

        while (head)
        {
            if (hashSet.find(head) == hashSet.end())
            {
                hashSet.insert(head);
            }
            else
            {
                return head;
            }
        }
        return nullptr;
    }
};

```



## [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 





```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        //合并链表
        //使用优先队列排序
        //返还链表

        priority_queue<int,vector<int>,greater<int>> queue;

        if(!list1 && !list2) return nullptr;
        if(!list1 )return list2;
        if(!list2 )return list1;
        
        //遍历list1
        ListNode* temp = list1;
        queue.push(temp->val);
        while (temp->next)
        {
            temp = temp->next;
            queue.push(temp->val);
        }

        temp->next = list2;
        //遍历list2
        while (list2)
        {
            queue.push(list2->val);
            list2 = list2->next;
        }

        //排数数值输入到链表中
        temp = list1;
        while (temp)
        {
            temp->val = queue.top();
            queue.pop();
            temp = temp->next;
        }
        return list1;
    }
};
```

## [2. 两数相加](https://leetcode.cn/problems/add-two-numbers/)

给你两个 **非空** 的链表，表示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。



```c++
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        //当前位
        int num = 0;
        //进一位
        int count = 0;

        //需要知道更长的一个链表作为存储计算结果的位置
        //如果我不像遍历2个链表怎么获取谁长?
        //自己创建链表遍历即可

        ListNode* head = nullptr;
        ListNode* tail = nullptr;
        //当不存在进一且l1,l2都指向空时说明完成
        while (l1 || l2|| count)
        {

            //判断l1,l2是否有效 有效就加入
            num += l1? l1->val : 0;
            num += l2? l2->val : 0;
            
            num += count;
            //判断是否进一
            count = num>9 ? 1:0;
            
            if (!head)
            {
                tail = new ListNode(num % 10);
                head = tail;
            }
            else
            {
                tail->next = new ListNode(num % 10);
                tail = tail->next;
            }
            num = 0;
            if (l1)
            {
                l1 = l1->next;
            }
            if (l2)
            {
                l2 = l2->next;
            }
        }
        return head;
    }
};
```

## [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

```c++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        //遍历整个链表获取链表长度

        //再次遍历删除第n个节点

        int size = 0;
        ListNode* temp = head;
        while (temp)
        {
            size++;
            temp = temp->next;
        }

        if(size == 1) return nullptr;
        if(size == n) return head->next;

        temp = head;
        for (int i = 0; i < size - n  -1; ++i)
        {
            temp = temp->next;
        }

        if (temp->next!= nullptr)
        {
            temp->next = temp->next->next;
        }
        return head;
    }
};
```

## [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。





```c++
class Solution {
public:
    ListNode* swapPairs(ListNode* head) 
    {
        ListNode* l = head;

        if(head == nullptr) return nullptr;
        
        ListNode* r = head->next;
        ListNode* prev = head;

        bool bFirst = true;

        while (l && r)
        {
            l->next = r->next;
            r->next = l;

            if (!bFirst)
            {
                prev->next = r;
            }
           
            if (bFirst)
            {
                bFirst = false;
                head = r;
            }

            l = l->next;
            prev = r->next;
            if (l)
            {
                r = l->next;
            }
        }
        return head;
    }
};
```

## [25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)

给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。





```c++
class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        ListNode* temp = head;
        int size = 0;
        while (temp)
        {
            size++;
            temp = temp->next;
        }
        int count = size / k;


        ListNode* newhead = new ListNode();
        newhead->next = head;
          
        //记录上一个链的结尾
        ListNode* prev = newhead;
        //记录当前链的开头(head)
        //记录当前链的结尾(tail)
        ListNode* tail = head;
        //记录下一个链的开头(next)
        ListNode* next = head;

        //改变链表顺序
        ListNode* curr = newhead;

        while (count--)
        {
            //更新当前链的开头结尾，
            curr = next;
            head = next;

            for (int i = 0; i < k; i++)
            {
                tail = next;
                next = tail->next;
                if (tail != curr)
                {
                    tail->next = curr;
                }
                curr = tail;
            }

            //这是已经翻转的当前链如 0 1<-2<-3 4  tail指针在3 head指针在1上  next指针在4上 prev指针在0上
            // 我需要变为0->3->2->1->4       prev->next = tail;   head->next = next;
            // 
            // 0 1<-2 3
            // 0->2->1->3->4
            // 0->2->1 3<-4  5<-6  nullptr
            //当前链的尾指向下个链的开头
            head->next = next;
            //上个链的结尾连接当前链的开头
            prev->next = tail;
            //更新上个链的结尾变为当前链的结尾
            prev = head;
        }
        return newhead->next;
    }
};
```



题解 重新创建一个函数完成链表翻转,每次将新链表的头和尾输入给翻转链表输出新链表的头和尾，

将新加入了链表头和尾重新加入原链表中

## [138. 随机链表的复制](https://leetcode.cn/problems/copy-list-with-random-pointer/)

给你一个长度为 `n` 的链表，每个节点包含一个额外增加的随机指针 `random` ，该指针可以指向链表中的任何节点或空节点。

构造这个链表的 **[深拷贝](https://baike.baidu.com/item/深拷贝/22785317?fr=aladdin)**。 深拷贝应该正好由 `n` 个 **全新** 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 `next` 指针和 `random` 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。**复制链表中的指针都不应指向原链表中的节点** 。

例如，如果原链表中有 `X` 和 `Y` 两个节点，其中 `X.random --> Y` 。那么在复制链表中对应的两个节点 `x` 和 `y` ，同样有 `x.random --> y` 。

返回复制链表的头节点。

用一个由 `n` 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 `[val, random_index]` 表示：

- `val`：一个表示 `Node.val` 的整数。
- `random_index`：随机指针指向的节点索引（范围从 `0` 到 `n-1`）；如果不指向任何节点，则为 `null` 。

你的代码 **只** 接受原链表的头节点 `head` 作为传入参数。

```c++
class Solution {
public:
    Node* copyRandomList(Node* head) 
    {
        Node* temp = head;
        Node* newHead = nullptr;
        Node* node = nullptr;

        unordered_map<Node* ,int> hashMap;
        unordered_map<int, Node*> nodeMap;
        int index = 0;

        //第一遍遍历创造链表，不管随机节点
        while (temp)
        {
            if (!newHead)
            {
                newHead = new Node(temp->val);
                node = newHead;
            }
            else
            {
                Node* nodeNew = new Node(temp->val);
                node->next = nodeNew;
                node = node->next;
            }

            //存储每个Node对应的index
            hashMap[temp] = index;
            index++;
            temp = temp->next;
        }

        temp = head;
        index = 0;
        Node* newTemp = newHead;
        //此时链表已经创建完毕，可以添加random节点，
        //从哪里获取random节点,需要提前存储
        while (temp)
        {
            //每个Node对应的random
            if (hashMap.find(temp->random) != hashMap.end())
            {
                //hashMap[temp->random]对应了random对应的下标
                //hashMap变为 新Node 与其对应的random下标
                hashMap[newTemp] = hashMap[temp->random];
            }
            //nodeMap存储每个Node对应的index
            nodeMap[index] = newTemp;
            index++;
            temp = temp->next;
            newTemp = newTemp->next;
        }

        newTemp = newHead;
        while (newTemp)
        {
            if (hashMap.find(newTemp) != hashMap.end())
            {
                newTemp->random = nodeMap[hashMap[newTemp]];
            }
            else
            {
                newTemp->random = nullptr;
            }
            newTemp = newTemp->next;
        }
        return newHead;
    }
};
```

## [148. 排序链表](https://leetcode.cn/problems/sort-list/)

给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。



归并排序

原理先分割链表
无限细分直到单个链表为1个节点再进行排序

难点 1 从中间分割链表 使用 快慢指针即可

难点 2 进行排序 由于是有序链表则正常递归排序，当一个链表为空时，将剩余的链表加入尾部

```c++
n->next = left == nullptr ? right : left;
```





```c++
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        //归并排序
        //将链表一份为二
        //

        if (head == nullptr || head->next == nullptr)
        {
            return head;
        }

        ListNode* slow = head;
        ListNode* fast = head->next;

        //寻找中点,
        while (fast && fast->next)
        {
            slow = slow->next;
            fast = fast->next->next;
        }

        //记录下个链表的头
        ListNode* temp = slow->next;
        //分割链表
        slow->next = nullptr;

        //继续对左边的链表进行拆分
        ListNode* left = sortList(head);
        //对右边链表拆分
        ListNode* right = sortList(temp);
        

        ListNode* res = new ListNode();
        ListNode* n = res;
        //最后拆成单个节点了
        //需要对新的两个链表进行排序
        while (left && right)
        {
            if (left->val < right->val)
            {
                n->next = left;
                left = left->next;
            }
            else
            {
                n->next = right;
                right = right->next;
            }
            n = n->next;
        }
        //如果其中一个仍有剩余需要补齐
        n->next = left == nullptr ? right : left;
        return res->next;
    }
};
```

## [23. 合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)

给你一个链表数组，每个链表都已经按升序排列。

请你将所有链表合并到一个升序链表中，返回合并后的链表。

```c++
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {

        ListNode* res = new ListNode();
        ListNode* temp = res;
        ListNode* left = nullptr;
        ListNode* right = nullptr;
        //我的想法
        //每次都使用2个链表 由于是有序的 正常排序即可
        for (int i = 0; i < lists.size(); i++)
        {
            temp = res;
            right = lists[i];
            left = temp->next;
            while (right && temp->next)
            {
                if (left->val < right->val)
                {
                    temp->next = left;
                    left = left->next;
                }
                else
                {
                    temp->next = right;
                    right = right->next;
                }
                temp = temp->next;
            }
            temp->next = right == nullptr ? left : right;
        }
        return res->next;
    }
};
```

