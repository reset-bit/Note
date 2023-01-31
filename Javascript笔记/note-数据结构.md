[toc]

# 数据结构

## Map(es6)
键值对**数组**
```javascript
let map = new Map([
    [1, 'one'],
    [2, 'two'],
    [3, 'three']
]);
map.set('a', 'aa');
map.set('b', 'bb');
map.get('a');// 'aa'
map.delete('a');
map.has('a');// false
map.clear();// null
map.forEach(function(item, key, obj) {
    console.log(item.toString());
});
// 按值排序
let arr = Array.from(map);// 将map转为数组， 也可[...map]
arr.sort(function(a, b) { return a[1] - b[1]; });// a[1]=>a.value
console.log(map);
// 获取keys并转为数组
let keys = [...map.keys()];
// 获取values并转为数组
let values = [...map.values()];
```

## Set(es6)

元素不重复的键值对数组，支持存放NaN、undefined

```javascript
let set = new Set([1, 2, 3, 3, 4]);//[1,2,3,4]
// let set = new Set('Hello');//['H','e','l','l','o']
set.add('some');//[1,2,3,4,"some"]
set.has(3);
set.size;//5
set.delete('some');
set.forEach(function(value) {
    console.log(value);
});
for(let i of set.keys()) {
    console.log(i);
}
for(let [key, value] of set.entries()) {
    console.log(key);
}
// 求交
let intersection = new Set([...set1].filter(x => set2.has(x)));
// 求差
let difference = new Set([...set1].filter(x => !set2.has(x)));
// 数组去重
let numbers = [1,2,1,2,1,4,3];
console.log([...new Set(numbers)]);//[1,2,3,4]
```

## 二叉树

```js
// 递归遍历
function traverse(root) {
    if(root !== null) {
        // 前序：根左右
        console.log(root.val);
       	traverse(root.left);
        traverse(root.right);
        // 中序：左根右
       	traverse(root.left);
        console.log(root.val);
        traverse(root.right);
        // 后序：左右根
       	traverse(root.left);
        traverse(root.right);
        console.log(root.val);
    }
};
// 非递归遍历
function traverse(root) {
    const stack = [], res = [];
    // 前序
    root && stack.push(root);
    while(stack.length > 0) {
        let cur = stack.pop();
        res.push(cur.val);
        cur.right && stack.push(cur.right);// 先入栈的后出栈，所以先入右子树
        cur.left && stack.push(cur.left);
    }
    // 中序
    while(root || stack.length) {
        if(root.left) {// 迭代左子树，令左子树依次入栈
            stack.push(root);
            root = root.left;
        } else if(root.right) {// 若左子树为空，（第一次）遍历到第一个叶子结点。输出当前节点val，迭代右子树
            res.push(root.val);
            root = root.right;
        } else if(!root.left && !root.right) {// 若左右子树都为空，则输出当前节点，弹出栈顶，返回上一层。注意root.left置空，防止再次进入循环
            res.push(root.val);
            root = stack.pop();
            root && (root.left = null);
        }
    }
    // 后序
    while(root || stack.length) {
        if(root.left) {// 迭代左子树，令左子树依次入栈
            stack.push(root);
            root = root.left;
        } else if(root.right) {// 若左子树为空，遍历到叶子节点，迭代右子树
            stack.push(root);
            root = root.right;
        } else {// 若左右子树都为空，输出当前节点，弹出栈顶，返回上一层。注意root.left、root.right置空
            res.push(root.val);
            root = stack.pop();
            if(root && root.left) {
                root.left = null;
            } else if(root && root.right) {
                root.right = null;
            }
        }
    }
    return res;
};
```

