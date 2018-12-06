---
title: "二分查找算法简单实现"
date: 2018-10-27
tags: 数据结构与算法
---

二分查找是基于已排序数组来查询的，所以是一个对目标数据有一定要求的算法，而且因为需要满足随机访问的特性，底层必须是数组，因此需要使用连续的内存空间，有一定的局限性。但是这种算法很适合那些修改较少而查询较多的场景，并且在时间复杂度O(logn)和空间复杂度O(1)上都有很大的优势。

## 最简单的二分查找
```java
    /**
     * Simple binary search
     *
     * @param a   sorted array
     * @param val target value
     * @return index
     */
    public static int bsearch(int[] a, int val) {
        int len = a.length;
        int low = 0, high = len - 1;
        while (low <= high) {
            int mid = (low + high) / 2;// 可以替换为 low + (high - low) >> 1 以避免溢出
            if (val < a[mid]) {
                high = mid - 1;
            } else if (val > a[mid]) {
                low = mid + 1;
            } else {
                return mid;
            }
        }
        return -1;
    }
```

## 查找第一个值等于给定值的元素
```java
    /**
     * Find the first equal value
     *
     * @param a   sorted array
     * @param val target value
     * @return index
     */
    public static int fbsearch(int[] a, int val) {
        int len = a.length;
        int low = 0, high = len - 1;
        while (low <= high) {
            int mid = low + (high - low) >> 1;// avoid overflow
            if (val < a[mid]) {
                high = mid - 1;
            } else if (val > a[mid]) {
                low = mid + 1;
            } else {
                if (mid == 0 || a[mid - 1] != val) {
                    return mid;
                } else {
                    high = mid - 1;
                }
            }
        }
        return -1;
    }
```

## 查找最后一个值等于给定值的元素
```java
    /**
     * Find the last equal value
     *
     * @param a   sorted array
     * @param val target value
     * @return index
     */
    public static int lbsearch(int[] a, int val) {
        int len = a.length;
        int low = 0, high = len - 1;
        while (low <= high) {
            int mid = low + (high - low) >> 1;// avoid overflow
            if (val < a[mid]) {
                high = mid - 1;
            } else if (val > a[mid]) {
                low = mid + 1;
            } else {
                if (mid == len - 1 || a[mid + 1] != val) {
                    return mid;
                } else {
                    return mid;
                }
            }
        }
        return -1;
    }
```

## 查找第一个大于等于等于给定值的元素
```java
    /**
     * Find the first greater or equal value
     *
     * @param a   sorted array
     * @param val target value
     * @return index
     */
    public static int fgebsearch(int[] a, int val) {
        int len = a.length;
        int low = 0, high = len - 1;
        while (low <= high) {
            int mid = low + (high - low) >> 1;// avoid overflow
            if (a[mid] >= val) {
                if (mid == 0 || (a[mid - 1] < val)) {
                    return mid;
                } else {
                    high = mid - 1;
                }
            } else {
                low = mid + 1;
            }
        }
        return -1;
    }
```

## 查找最后一个小于等于给定值的元素
```java
    /**
     * Find the last lower or equal value
     *
     * @param a   sorted array
     * @param val target value
     * @return index
     */
    public static int llebsearch(int[] a, int val) {
        int len = a.length;
        int low = 0, high = len - 1;
        while (low <= high) {
            int mid = low + (high - low) >> 1;// avoid overflow
            if (a[mid] <= val) {
                if (mid == len - 1 || (a[mid + 1] > val)) {
                    return mid;
                } else {
                    low = mid + 1;
                }
            } else {
                high = mid - 1;
            }
        }
        return -1;
    }
```