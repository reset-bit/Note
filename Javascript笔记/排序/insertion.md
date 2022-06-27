# 插入排序 Insertionsort

## 算法步骤与思想

将第一个元素看作一个有序序列，把第二个元素到最后一个元素当成是未排序序列。从头到尾依次扫描未排序序列，将扫描到的每个元素插入有序序列的适当位置。

## 衡量指标

空间复杂度：`O(1)`

时间复杂度：平均`O(n^2)`

稳定性：稳定

## 小记

继冒泡排序的又一稳定算法。

菜鸟教程说的好，插入排序仿佛就是在打扑克。从后向前找，如果有序序列元素比当前元素大，肯定得是向后移动一个位置。如果比当前元素小，那有序序列元素的位置+1就是当前元素的位置了。

无所顾忌地向后移动一个位置的原因，是在内层循环外已经保存了当前元素了。有序序列最后一个元素的下一个元素不就是当前元素吗？:)

```js
Array.prototype.insertionSort = function() {
    let arr = this;
    let preIndex, current;
    for(let i = 1; i < arr.length; ++i) {
        preIndex = i - 1;
        current = arr[i];
        while(preIndex >= 0 && arr[preIndex] > current) {
            arr[preIndex + 1] = arr[preIndex];
            preIndex--;
        }
        arr[preIndex +]
    }
};
```

