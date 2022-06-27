# 快速排序 Quick sort

## 算法步骤与思想

选择pivot作为衡量基准（基准的选择有多种方式）。重新排序数组，所有比pivot小的放到左边，所有比pivot大的放到右边。对pivot左右两边分别进行相同的操作。

## 衡量指标

时间复杂度：平均`O(nlogn)`，最差`O(n^2)`

稳定性：不稳定

## 小记

终于到了最最著名的快排，左拖右拖终于还是来了。

总是记住选择基准pivot，牢记快之一字，判断的逻辑越少越好。所以：比我小的向左走，比我大的向右走。这时候左右两边仍然是无序的。正因为此，在一次“指挥交通”完成之后，pivot可以毫无顾忌地和最后一个比它小的元素交换位置。反正左边是无序的，都比pivot小，交换到第一个又不会咋样。

这里并没有使用变量直接保存数组值，而是保存了数组下标。所以swap后元素变了，下标却还是原来的值。所以partition()返回值还是`smaller- 1`。`smaller-pivot`表示的是比pivot小的元素个数+1（因为凡进入if，一定代表找到了一个比pivot小的元素）。如果当前值比pivot小，自然是要到左边去的，和【上一个已经确定比povit小的元素的右邻居】交换（毕竟前文提到差=个数+1），`smaller++`。如果当前值比pivot大，`smaller`不变，可是代表当前值下标的i还是在变大。一旦后面还有比pivot小的值，还是会根据`smaller`精确地找到该去的位置。完美。

```js
function quickSort(arr, left, right) {
    left = typeof left === 'number' ? left : 0;
    right = typeof right === 'number' ? right : arr.length - 1;
    let partitionIndex = 0;
    if(left < right) {
        partitionIndex = partition(arr, left, right);
        quickSort(arr, left, partitionIndex - 1);
        quickSort(arr, partition + 1, right);
    }
};
function partition(arr, left, right) {
    let pivot = left;
    let smaller = pivot + 1;
    for(let i = index; i < right; ++i) {
        if(arr[i] < arr[pivot]) {
            swap(arr, i, smaller);
            smaller++;
        }
    }
    swap(arr, pivot, smaller - 1);
    return smaller - 1;
};
function swap(arr, i, j) {
    let temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
};
```

