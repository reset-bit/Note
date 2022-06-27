# 选择排序 Selection sort

## 算法步骤与思想
简单直观的排序算法。在【未排序序列】中找到最小/大元素，存放到排序序列的起始位置。重复上述过程，直到所有元素排序完毕。

## 衡量指标
数据量越小越好

时间复杂度：`O(n^2)`

稳定性：不稳定

## 小记

在外层循环依旧要注意，遍历`length-1`次即可，最后一个自动确定顺序。对寻找【未排序序列中最小元素的下标】循环，下标从外层`i`基础上+1开始，到最后一个元素结束。结束内层循环后，将找到的最小元素与当前元素进行交换。

选择排序其实和冒泡排序有相似之处，但冒泡排序是和邻近的元素进行比较，选择排序是在未排序的序列范围内进行比较。

```js
Array.prototype.selectionSort = function() {
	let tmp = 0, minIndex = 0;
	for(let i = 0; i < this.length - 1; ++i) {
		// 找到未排序序列中最小元素下标
		minIndex = i;
		for(let j = i + 1; j < this.length; ++j) {
			if(this[j] < this[minIndex]) {
				minIndex = j;
			}
		}
		tmp = this[i];
		this[i] = this[minIndex];
		this[minIndex] = tmp;
	}
};
```

