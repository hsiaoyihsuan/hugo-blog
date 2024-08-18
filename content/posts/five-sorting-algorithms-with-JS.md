---
title: "Implement 5 common sorting algorithms with JavaScript"
date: 2024-08-18
categories:
  - JavaScript
tags:
  - JavaScript
  - DSA
thumbnailImagePosition: left
thumbnailImage: images/javascript.svg
---
We'll explore five of the most common sorting algorithms: **Bubble Sort**, **Selection Sort**, **Insertion Sort**, **Merge Sort** and **Quick Sort**. Each will be implemented in JavaScript, with an analysis of their time and space complexities.

## 1. Bubble Sort

![bubble sort](/images/bubble-sort.svg)

```javascript
function bubbleSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = 1; j < arr.length; j++) {
      if (arr[j - 1] > arr[j]) {
        [arr[j - 1], arr[j]] = [arr[j], arr[j - 1]];
      }
    }
  }
  return arr;
}
```

This code can be optimized by ruducing the numbers of iteration in the inner loop and exiting early if the array is already sorted.

Refined Version:
```javascript
function bubbleSort(arr) {
  let isSorted = true; // Assume array is already sorted
  for (let i = 0; i < arr.length; i++) {
    // reduce push bubble times by minus i
    for (let j = 1; j < arr.length - i; j++) {
      if (arr[j - 1] > arr[j]) {
        isSorted = false; // When we know that arr is not sorted
        [arr[j - 1], arr[j]] = [arr[j], arr[j - 1]];
      }
    }
    // If arr is sorted, arr will be return when i = 0
    if (isSorted) {
      return arr;
    }
  }
  return arr;
}
```

- Time Complexity: *O(n2)*
- Space Complexity: *O(1)*

## 2. Selection Sort

![selection sort](/images/selection-sort.svg)

```javascript
function selectionSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    let minIndex = i;
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }
    [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
  }
  return arr;
}
```

- Time Complexity: *O(n2)*
- Space Complexity: *O(1)*

## 3. Insertion Sort

![insertion sort](/images/insertion-sort.svg)

```javascript
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    let current = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > current) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = current;
  }
  return arr;
}
```

- Time Complexity:
  -  Best Case: *O(n)*
  -  Worst Case: *O(n2)*
  -  Average Case: *O(n2)*
- Space Complexity: *O(1)*

Bubble sort, selection sort and insertion sort all sort the array in place and are straightforward to implement. They have the time complexity of *O(n2)*, but insertion sort is generally more efficient. It performs better on the nearly sorted arrays, making it a more popular choice for real-world application.

## 4. Merge Sort

![merge sort](/images/merge-sort.svg)

```javascript
function mergeSort(arr) {
  if (arr.length <= 1) {
    return arr;
  }

  const mid = Math.floor(arr.length / 2);
  const left = arr.slice(0, mid);
  const right = arr.slice(mid);
  return merge(mergeSort(left), mergeSort(right));
}

function merge(left, right) {
  const result = [];
  let leftIndex = 0;
  let rightIndex = 0;

  while (leftIndex < left.length && rightIndex < right.length) {
    if (left[leftIndex] < right[rightIndex]) {
      result.push(left[leftIndex]);
      leftIndex++;
    } else {
      result.push(right[rightIndex]);
      rightIndex++;
    }
  }
  return result.concat(left.slice(leftIndex)).concat(right.slice(rightIndex));
}
```
- Time Complexity: *O(n log n)*
- Space Complexity: *O(n)*

In comparison to the algorithms mentioned above, merge sort offers a better time complexity *O(n log n)* and is stable across differenct cases, making it suitable for sorting large, unsorted arrays.

## 5. Quick Sort

![quick sort](/images/quick-sort.svg)

```javascript
function quickSort(arr, start = 0, end = arr.length - 1) {
  if (start >= end) {
    return arr;
  }

  const pivot = partition(arr, start, end);
  quickSort(arr, start, pivot - 1);
  quickSort(arr, pivot + 1, end);

  return arr;
}

function partition(arr, start, end) {
  const pivot = arr[end];
  let i = start;

  for (let j = start; j < end; j++) {
    if (arr[j] < pivot) {
      [arr[i], arr[j]] = [arr[j], arr[i]];
      i++;
    }
  }
  [arr[i], arr[end]] = [arr[end], arr[i]];
  return i;
}
```
- Time Complexity:
  -  Best Case: *O(n log n)*
  -  Worst Case: *O(n2)*
  -  Average Case: *O(n log n)*
- Space Complexity:
  -  Best Case: *O(log n)*
  -  Worst Case: *O(n)* (When array is nearly sorted)
  -  Average Case: *O(log n)*

Quick sort is the fastest among the all sorting algorithms, but it must be optimized to avoid the worst-case scenario, especially when the list is nearly sorted. Unlike merge sort, quick sort use less memory which helps with cache efficiency. However, it's more challenging to implement and is untable across cases.

---

## References

- https://www.youtube.com/@BroCodez
- https://en.wikipedia.org/wiki/Sorting_algorithm#:~:text=Together%20with%20its%20modest%20O,in%20many%20standard%20programming%20libraries.
