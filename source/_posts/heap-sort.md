---
title: "堆排序简单实现"
date: 2018-12-29
tags: 数据结构与算法
---

```java
public class HeapSort {

    public static void heapSort(int[] a) {
        int len = a.length - 1;
        buildHeap(a, len);
        int k = len;
        while (k > 0) {
            swap(a, 0, k);
            --k;
            heapify(a, k, 0);
        }
    }

    private static void buildHeap(int[] a, int n) {
        for (int i = (n - 1) / 2; i >= 0; --i) {
            heapify(a, n, i);
        }
    }

    private static void heapify(int[] a, int n, int i) {
        while (true) {
            int maxPos = i;
            if (i * 2 + 1 <= n && a[i] < a[i * 2 + 1]) maxPos = i * 2 + 1;
            if (i * 2 + 2 <= n && a[maxPos] < a[i * 2 + 2]) maxPos = i * 2 + 2;
            if (maxPos == i) break;
            swap(a, i, maxPos);
            i = maxPos;
        }
    }

    private static void swap(int[] a, int i, int maxPos) {
        int t = a[i];
        a[i] = a[maxPos];
        a[maxPos] = t;
    }
}
```
