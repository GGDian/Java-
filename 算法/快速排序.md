快速排序

```java
 public static int partition2(int[] arr, int l, int r) {
        int left = l + 1, right = r;
        int pivot = arr[l];
        while (left <= right) {
            while (left <= right && arr[left] <= pivot) left++;
            while (left <= right && arr[right] > pivot) right--;
            if (left < right) {
                swap(arr, left, right);
            }
        }
        swap(arr, right, l);
        return right;
    }
```

```java
 public static int partition(int[] arr, int l, int r) {
        int i = l;
        int pivot = arr[r];
        for (int j = l; j < r; j++) {
            if (arr[j] < pivot) {
                swap(arr, i, j);
                i++;
            }
        }
        swap(arr, i, r);
        return i;
    }
```

```java
protected void sort(T[] nums, int l, int h) {
        if (h <= l) {
            return;
        }
        int lt = l, i = l + 1, gt = h;
        T v = nums[l];
        while (i <= gt) {
            int cmp = nums[i].compareTo(v);
            if (cmp < 0) {
                swap(nums, lt++, i++);
            } else if (cmp > 0) {
                swap(nums, i, gt--);
            } else {
                i++;
            }
        }
        sort(nums, l, lt - 1);
        sort(nums, gt + 1, h);
}
```

