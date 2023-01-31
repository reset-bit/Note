[toc]

# 常用封装

## 判断两个对象内容相等

```javascript
// 递归实现
function isObjectValueEqual(a, b) {
    let aProps = Object.getOwnPropertyNames(a);
    let bProps = Object.getOwnPropertyNames(b);
    if(aProps.length !=== bProps.length) return false;
    for(let i = 0; i < aProps.length; ++i) {
        let propName = aProps[i];
        // 若b没有该属性值
        if(!b.hasOwnProperty(propName)) return false;
        // 若当前比较属性值是对象，并且递归之后返回false
        if(a[propName] instanceof Object && !isObjectValueEqual(a[propName], b[propName])) return false;
    }
    return true;
}
```

## 判断简单类型的数组内容相等

```javascript
// 1.
JSON.stringify(arr1.sort()) === JSON.stringify(arr2.sort())
// 2.
arr1.sort().toString() === arr2.sort().toString()
```



# 动态规划

## 1.房间移动的最少次数

存在n+1个房间，每个房间依次为房间1 2 3...i，每个房间都存在一个传送门，i房间的传送门可以把人传送到房间pi(1<=pi<=i),现在路人甲从房间1开始出发(当前房间1即第一次访问)，每次移动他有两种移动策略：
  A. 如果访问过当前房间 i 偶数次，那么下一次移动到房间i+1；
  B. 如果访问过当前房间 i 奇数次，那么移动到房间pi；
现在路人甲想知道移动到房间n+1一共需要多少次移动；

### 思路描述

动态规划的问题需要找到相邻项之间的代数关系。很容易发现，只有到达次数为偶数次的时候才能向前走；到达次数为奇数次的时候，只能向后退。所以，**到达i门，证明i门之前的所有门都走过，并且次数都是偶数次**。因为奇数次到达i门的时候向后退到p[i]门，i门一定在p[i]门之后。

设dp[i]，表示第一次到达i门需要使用的步数，dp[n+1]即为目标答案。

**dp[i] = dp[i-1] + 第二次到达i-1门需要的步数 + 1**。求第二次到达i-1门需要的步数：第一次到达i-1门会跳转到p[i-1]门。此时p[i-1]门为奇数次，其他所有门为偶数次。这和第一次到达p[i-1]门是相同的情况。而且，**dp[i-1] = dp[p[i-1]] + i-1门到p[i-1]门需要的步数**。所以第二次到达i-1门需要的步数=dp[i-1]-dp[p[i-1]]。

故，**dp[i] = dp[i-1] + dp[i-1] - dp[p[i-1]] + 1 + 1 = 2 * dp[i-1] - dp[p[i-1]] + 2**。

```javascript
let n = parseInt(readline());
let str = readline().split(' ');
let p = new Array(n + 1);// 从1到n
for(let i = 1; i <= n; ++i) {
    p[i] = parseInt(str[i - 1]);
}
let dp = new Array(n + 2).fill(0);// 从1到n+1
for(let i = 2; i <= n + 1; ++i) {
    dp[i] = mod((2 * dp[i - 1] - dp[p[i - 1]] + 2), (1e9 + 7));
}
print((dp[n + 1] < 0 ? mod(dp[n + 1], 1e9 + 7) : dp[n + 1]));// 防止范围溢出

function mod(n, m) {
    return ((n % m) + m) % m;
};
```

