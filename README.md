# codepratice-day2
代码随想录第二天
Day2 滑动窗口、区间定义（类似二分法中的开闭区间）、前缀和

[长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)
```CPP
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int res=INT_MAX,i=0,sum=0;
        for(int j=0;j<nums.size();j++)
        {
            sum+=nums[j];
            while(sum>=target)
            {
                int subl=j-i+1;
                res=min(res,subl);
                sum-=nums[i];
                i++;
            }
        }
        return res==INT_MAX?0:res;
    }
};
```

[螺旋矩阵 Ⅱ](https://leetcode.cn/problems/spiral-matrix-ii/)
```CPP
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> res(n,vector<int>(n,0));
        int startx=0,starty=0;
        int loop=n/2;
        int mid=n/2;
        int offset=1;
        int count=1;
        int i,j;
        while(loop--)
        {
            i=startx;
            j=starty;
            for(;j<n-offset;j++)
            {
                res[i][j]=count++;
            }
            for(;i<n-offset;i++)
            {
                res[i][j]=count++;
            }
            for(;j>starty;j--)
            {
                res[i][j]=count++;
            }
            for(;i>startx;i--)
            {
                res[i][j]=count++;
            }

            startx++;
            starty++;

            offset++;
        }
        if(n%2)
        {
            res[mid][mid]=count;
        }
        return res;
    }
};
```

螺旋矩阵的难点，在于应该每次循环时，计算的量是一样的，即头结点包含，但是尾节点不包含，在下一次计算时包含，即在每次遍历时都赋值的数组数量相同。

区间和内容：
即前缀和，之前学过，一维前缀和，不难。

开发商购买土地：
这段代码中有类似sum - horizontalCut - horizontalCut这样的计算，这里原因是答案需要两个开发商之间的差，差为两部分相减，一部分为horizontalCut(水平前缀和)
减去总和减去前缀和horizontalCut，就成了sum-两次的前缀和。计算思路是计算每行前缀和，用总的减去前缀和，即为当前行或列划分的结果。
其实用二维前缀和也好理解一些，看着更简洁。
多种代码解答如下：
```CPP   二维前缀和
#include <iostream>
#include <vector>
#include <cmath>
#include <climits>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;
    vector<vector<int>> grid(n, vector<int>(m));
    vector<vector<int>> prefix(n+1, vector<int>(m+1, 0));

    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < m; ++j) {
            cin >> grid[i][j];
            prefix[i+1][j+1] = grid[i][j] + prefix[i][j+1] + prefix[i+1][j] - prefix[i][j];
        }
    }

    int minDiff = INT_MAX;
    // 横向切割
    for (int i = 1; i < n; ++i) {
        int top = prefix[i][m];
        int bottom = prefix[n][m] - top;
        minDiff = min(minDiff, abs(top - bottom));
    }

    // 纵向切割
    for (int j = 1; j < m; ++j) {
        int left = prefix[n][j];
        int right = prefix[n][m] - left;
        minDiff = min(minDiff, abs(left - right));
    }

    cout << minDiff << endl;
    return 0;
}
```

```CPP 一维前缀和
#include <iostream>
#include <vector>
#include <climits>

using namespace std;
int main () {
    int n, m;
    cin >> n >> m;
    int sum = 0;
    vector<vector<int>> vec(n, vector<int>(m, 0)) ;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> vec[i][j];
            sum += vec[i][j];
        }
    }
    // 统计横向
    vector<int> horizontal(n, 0);
    for (int i = 0; i < n; i++) {
        for (int j = 0 ; j < m; j++) {
            horizontal[i] += vec[i][j];
        }
    }
    // 统计纵向
    vector<int> vertical(m , 0);
    for (int j = 0; j < m; j++) {
        for (int i = 0 ; i < n; i++) {
            vertical[j] += vec[i][j];
        }
    }
    int result = INT_MAX;
    int horizontalCut = 0;
    for (int i = 0 ; i < n; i++) {
        horizontalCut += horizontal[i];
        result = min(result, abs(sum - horizontalCut - horizontalCut));
    }
    int verticalCut = 0;
    for (int j = 0; j < m; j++) {
        verticalCut += vertical[j];
        result = min(result, abs(sum - verticalCut - verticalCut));
    }
    cout << result << endl;
}
```
优化后，不使用一维前缀和也可以计算出来。
```CPP 
#include <iostream>
#include <vector>
#include <climits>

using namespace std;
int main () {
    int n, m;
    cin >> n >> m;
    int sum = 0;
    vector<vector<int>> vec(n, vector<int>(m, 0)) ;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> vec[i][j];
            sum += vec[i][j];
        }
    }

    int result = INT_MAX;
    int count = 0; // 统计遍历过的行
    for (int i = 0; i < n; i++) {
        for (int j = 0 ; j < m; j++) {
            count += vec[i][j];
            // 遍历到行末尾时候开始统计
            if (j == m - 1) result = min (result, abs(sum - count - count));

        }
    }

    count = 0; // 统计遍历过的列
    for (int j = 0; j < m; j++) {
        for (int i = 0 ; i < n; i++) {
            count += vec[i][j];
            // 遍历到列末尾的时候开始统计
            if (i == n - 1) result = min (result, abs(sum - count - count));
        }
    }
    cout << result << endl;
}
```
