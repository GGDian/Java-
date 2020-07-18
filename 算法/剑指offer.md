# 删除链表中重复的结点

```java
public ListNode deleteDuplication(ListNode pHead){
    if (pHead == null || pHead.next == null)
        return pHead;
    ListNode next = pHead.next;
    if (pHead.val == next.val){
        // next 不等于 null 才做判断，避免空指针异常
        while (next != null && pHead.val == next.val)
            next = next.next;
        // 找到第一个不重复的头节点
        return deleteDuplication(next);
    }
    // 递归，每轮定位一个数字
    pHead.next = deleteDuplication(pHead.next);
    return pHead;
}
```

# 数值的整数次方

```java
public double Power(double base, int exponent) {
    if (exponent == 0)
        return 1;
    if (base == 0)
        return 0;
    boolean isNegative = false;
    if (exponent < 0) {
        // 负的变成正的计算
        exponent = -exponent;
        // 记录下标志位
        isNegative = true;
    }
    // 递归解决， 直到 exponent 为 0；
    double power = Power(base * base, exponent / 2);
    // exponent 有可能是为单数，这样需要补一个base，因为除2是向下取整 
    if (exponent % 2 != 0)
        power = power * base;
    // 负倍数要转换
    return isNegative ? 1 / power : power;
}
```



# 冒泡思想

```java
public void reOrderArray(int[] array) {
    int N = array.length;
    // 临界问题可以在脑海里抽象一个数组
    // N-1 是最后一个数，我排序到 1 就可以了
    // j < N-1；N-1 是最后的数，j 到 N-2 就可以了的
    for (int i = N - 1; i > 0; i--) {
        for (int j = 0; j < i; j++) {
            if (isEven(array[j]) && !isEven(array[j + 1])) {
                int t = array[j];
                array[j] = array[j + 1];
                array[j + 1] = t;
            }
        }
    }
}
```





# 链表中倒数第 K 个结点

```java
public ListNode FindKthToTail(ListNode head, int k) {
    if (head == null || head.next == null)
        return head;
    ListNode p1 = head, p2 = head;
    // 注意：这里是走完 k 了的。也就是 k 为 3
    // p1 已经在头节点，此时会跑到第四个节点上
    while (p1 != null && k-- > 0) {
        p1 = p1.next;
    }
    if (k > 0){
        return  null;
    }
    // 这样的话；这里就得跑到最后面，p1 为空的位置
    // 所以判断条件为 p1 != null ；不是 p1.next != null
    // 如果想要 p1.next != null ; 前面的 k 只能跑 k-1 位置
    // 如果 k 只跑两步，那么这里跑到最后节点就可以了
    while (p1 != null){
        p1 = p1.next;
        p2 = p2.next;
    }
    return p2;
}
```



# 链表中环的入口结点

```java
public ListNode EntryNodeOfLoop(ListNode pHead) {
    if (pHead == null || pHead.next == null)
        return null;
    ListNode p1 = pHead, p2 = pHead;
    // 这里用do while
    // p1 和 p2 最开始都是 pHead 啊啊啊啊啊
    // 单纯用 while，两个点都是 pHead; 肯定不行啦
    do {
        p1 = p1.next.next;
        p2 = p2.next;
    } while (p2 != p1);
    p1 = pHead;
    while (p1 != p2) {
        p1 = p1.next;
        p2 = p2.next;
    }
    return p1;
}
```





# 反转链表（头插法）

```java
public ListNode ReverseList(ListNode head) {
    if (head == null || head.next == null ){
        return head;
    }
    // 注意这里，使用头插法
    // 新建了一个节点，但是！！我们只是用它来当工具节点
    ListNode newList = new ListNode(-1);
    while (head != null){
        ListNode next = head.next;
        // 看这里，是取newList next
        head.next = newList.next;
        // 再看这里，替代的是 newList 的 next
        newList.next = head;
        head = next;
    }
    return newList.next;
}
```

## 反转链表（递归法）

```java
public ListNode ReverseList2(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    //1->2->3->4
    ListNode next = head.next;
    // 先递归全部断掉链接
    // 1 2->3-> 4
    head.next = null;
    //递归到尾部返回 4
    ListNode newNode = ReverseList2(next);
    // 再反着全部连上
    //1  2  3 <- 4
    next.next = head;
    // 在递归的路上返回 newNode；就只是尾节点
    return newNode;
}
```



# 合并两个排序的链表

```java
public ListNode Merge(ListNode list1, ListNode list2) {
    // 归并思想
    // 先设置一个链接头节点用于返回
    ListNode head = new ListNode(-1);
    // 下标定位移动
    ListNode cur = head;
    while (list1 != null && list2 != null) {
        if (list1.val <= list2.val) {
            cur.next = list1;
            list1 = list1.next;
        } else {
            cur.next = list2;
            list2 = list2.next;
        }
        // 移动
        cur = cur.next;
    }
    // 注意！！
    // 因为是链表，所以直接绑定就可以，后面节点都会链接过来
    // 注意注意，这里跟数组的不同，数组的要慢慢遍历
    if (list1 != null)
        cur.next = list1;
    if (list2 != null)
        cur.next = list2;
    return head.next;
}
```

## 递归方法

```java
public ListNode Merge(ListNode list1, ListNode list2) {
    if (list1 == null)
        // 出口
        return list2;
    if (list2 == null)
        // 出口
        return list1;
    if (list1.val <= list2.val) {
        // 每一轮定位一个节点，一条路走到黑
        // merge 会返回下一个节点
        // 此方法保证了 next 指针不会断，一直往下
        list1.next = Merge(list1.next, list2);
        return list1;
    } else {
        // 每一轮定位一个节点，一条路走到黑
        list2.next = Merge(list1, list2.next);
        return list2;
    }
}
```



# 树的子结构

```java
public boolean HasSubtree(TreeNode root1, TreeNode root2) {
    if (root1 == null || root2 == null) {
        return false;
    }
    return isSubtreeWithRoot(root1, root2) || HasSubtree(root1.left, root2) || HasSubtree(root1.right, root2);

}
// 判断当前节点与 子树 是否匹配
private boolean isSubtreeWithRoot(TreeNode root1, TreeNode root2) {
    // 这里 root2意思为 root2 的子树，意思是子树的子树，为空就是覆盖了的
    if (root2 == null) {
        return true;
    }
    // 子树不为空，root树为空，不匹配
    if (root1 == null) {
        return false;
    }
    if (root1.val != root2.val)
        return false;
    return isSubtreeWithRoot(root1.left, root2.left) && isSubtreeWithRoot(root1.right, root2.right);
}
```





# 对称的二叉树

```java
boolean isSymmetrical(TreeNode pRoot) {
    if (pRoot == null)
        return true;
    return isSymmetrical(pRoot.left, pRoot.right);
}

boolean isSymmetrical(TreeNode lRoot, TreeNode rRoot) {
    if (lRoot == null && rRoot == null)
        return true;
  /*  if (lRoot == null && rRoot != null)
        return false;
    if (rRoot == null && lRoot != null)
        return false;*/
  // 这条语句等同于上面两条
    if (rRoot == null || lRoot == null)
        return false;
    if (lRoot.val != rRoot.val)
        return false;
    // 镜像，是镜像对称
    return isSymmetrical(lRoot.left, rRoot.right) && isSymmetrical(lRoot.right, rRoot.left);
}
```



# 顺时针打印矩阵

```java
ArrayList<Integer> ret = new ArrayList<>();
// 列，行
int c1 = 0, c2 = matrix[0].length - 1, r1 = 0, r2 = matrix.length - 1;
while (c1 <= c2 && r1 <= r2) {
    for (int i = c1; i <= c2; i++)
        ret.add(matrix[r1][i]);
    for (int i = r1 + 1; i <= r2; i++)
        ret.add(matrix[i][c2]);
    //防止同一行又跑回去加一次
    if (r1 != r2)
        for (int i = c2 - 1; i > c1; i--)
            ret.add(matrix[r2][i]);
        //防止同一列又跑回去刷一次
    if (c1 != c2)
        for (int i = r2; i > r1; i--)
            ret.add(matrix[i][c1]);
    c1++;
    c2--;
    r1++;
    r2--;
}
return ret;
```



# 从上往下打印二叉树

```java
public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
    Queue<TreeNode> queue = new LinkedList<>();
    ArrayList<Integer> ret = new ArrayList<>();
    queue.add(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        //每消耗一个减一个
        while (size-- > 0){
            //取出一个
            TreeNode node = queue.poll();
            //预防有空洞
            if (node == null)
                continue;
            ret.add(node.val);
            //加两个
            queue.add(node.left);
            queue.add(node.right);
        }
    }
    return ret;
}
```



# 最长连续序列

```java
 public int longestConsecutive(int[] nums) {
     	Set<Integer> set = new HashSet<>();
     	//遍历写入 set
        Arrays.stream(nums).forEach(set::add);
        int max = 0;
        for (int num : nums) {
            int count = 1;
            while (set.contains(num + 1)) {
                num++;
                count++;
            }
            max = Math.max(max, count);
        }
        return max;
    }
```



# 二叉搜索树的后序遍历序列

```java
public boolean VerifySquenceOfBST(int[] sequence) {
    if (sequence == null || sequence.length == 0)
        return false;
    return verify(sequence, 0, sequence.length - 1);
}

private boolean verify(int[] sequence, int first, int last) {
    // 出口；只剩下一个节点，是树
    if (last - first <= 1) {
        return true;
    }
    //最后一个数是 root 节点
    int rootVal = sequence[last];
    int cutIndex = first;
    // 找到左右子树的分界点
    while (cutIndex < last && rootVal >= sequence[cutIndex])
        cutIndex++;
    // 判断后半截是否都大于 root 节点
    for (int i = cutIndex; i < last; i++) {
        if (sequence[i] < rootVal)
            //不是，返回
            return false;
    }
    // 截掉 root 节点，左子树和右子树也要是二叉搜索树
    return verify(sequence, first, cutIndex) && verify(sequence, cutIndex + 1, last - 1);
}
```





# 二叉树中和为某一值的路径

```java
ArrayList<ArrayList<Integer>> ret = new ArrayList<>();
public ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int target) {
    backTracing(root, target, new ArrayList<>());
    return ret;
}
private void backTracing(TreeNode root, int target, ArrayList<Integer> path) {
    // 出口，null 即返回
    if (root == null)
        return;
    // 加入走过路径
    path.add(root.val);
    // 减去目标值
    target -= root.val;
    if (target == 0 && root.left == null && root.right == null) {
        ret.add(new ArrayList<>(path));
    } else {
        // 走左子树
        backTracing(root.left, target, path);
        // 走右子树
        backTracing(root.right, target, path);
    }
    // 递归回溯删除最后一个数字
    // 防止污染到其他的路径
    // 有可能路径中有重复的数字，删除也只删除一个删错了
    // 所以直接删除最后一个最为保险
    path.remove(path.size() - 1);
}
```



# 复杂链表的复制

```java
public RandomListNode Clone(RandomListNode pHead) {
    if (pHead == null)
        return null;
    // 插入新节点
    RandomListNode cur = pHead;
    while (cur != null) {
        RandomListNode clone = new RandomListNode(cur.label);
        clone.next = cur.next;
        cur.next = clone;
        cur = clone.next;
    }
    // 建立 random 链接
    cur = pHead;
    while (cur != null) {
        RandomListNode clone = cur.next;
        if (cur.random != null)
            // 这里很妙，把随机指针的复制节点引用返回
            clone.random = cur.random.next;
        cur = clone.next;
    }
    // 拆分
    cur = pHead;
    RandomListNode pCloneHead = pHead.next;
    while (cur.next != null) {
        RandomListNode next = cur.next;
        cur.next = next.next;
        cur = next;
    }
    return pCloneHead;
}
```



# 二叉搜索树与双向链表

这道题要注意的点是，二叉搜索树，即大小关系是 left < root <right

注意：在 root 后方的节点，是 right 子树中最小的节点，并不是 right 节点

所以没办法采用 

```
左子树完成链接
右子树完成链接
root 执行与左右子树链接的操作
```

这种逻辑；改用每个节点递归完成与前驱节点链接这种思维

```java
TreeNode pre, head;

public TreeNode Convert(TreeNode pRootOfTree) {
    //每个节点只需要执行自身绑定前驱节点的操作即可
    if (pRootOfTree == null)
        return null;
    // 先左边
    Convert(pRootOfTree.left);
    //然后 root 节点绑定连接到前面
    pRootOfTree.left = pre;
    if (pre != null)
        pre.right = pRootOfTree;
    //当前节点为后续节点的前驱节点
    pre = pRootOfTree;
    if(head == null)
        // 设置头结点
        head = pRootOfTree;
    //后续节点连接
    Convert(pRootOfTree.right);
    return head;
}
```





# 字符串的排列

```java
ArrayList<String> rets = new ArrayList<>();

public ArrayList<String> Permutation(String str) {
    if (str == null || str.length() == 0) {
        return null;
    }
    char[] chars = str.toCharArray();
    // 排个序先
    Arrays.sort(chars);
    // 开始递归选择元素
    backtracking(chars, new boolean[chars.length], new StringBuilder());
    return rets;
}

private void backtracking(char[] chars, boolean[] hasUsed, StringBuilder s) {
    if (chars.length == s.length()) {
        rets.add(s.toString());
        return;
    }
    for (int i = 0; i < chars.length; i++) {
        if (hasUsed[i])
            continue;
        // 当前元素与上一个元素相等，并且上一个元素没使用
        // 那么我是第二个相同的，它用过一次了，此时我再用，就会有重复的组合
        // 所以，除非我们两个一起使用；否则单个出现的机会都交给它(上一个重复元素)
        if (i != 0 && chars[i] == chars[i - 1] && !hasUsed[i - 1])
            continue;
        s.append(chars[i]);
        hasUsed[i] = true;
        // 继续选择下一个元素
        backtracking(chars, hasUsed, s);
        // 回溯删除最后一个元素
        s.deleteCharAt(s.length() - 1);
        // 回溯置为未使用
        hasUsed[i] = false;
    }
}
```



# 数组中出现次数超过一半的数字

此题还有变体 ； 即如果只刚刚好为一半

同样可以这么处理，加上两个条件；

* 我们假设尾节点就是要找的那个数，每个节点判断是否跟尾节点相等？等于一半就是
* 否则；这个数为 candidate

```java
public int MoreThanHalfNum_Solution(int[] array) {
    int cnt = 1;
    // 候选者
    int candidate = array[0];
    for (int i = 1; i < array.length; i++) {
        // 不同就两两抵消
        cnt = candidate == array[i] ? cnt + 1 : cnt - 1;
        // 为0就选择下一个候选者重新开始计数
        if (cnt == 0) {
            candidate = array[i];
            cnt = 1;
        }
    }
    // 如果有超过一半的数，那 candidate 即为那个数；再次判断
    cnt = 0;
    for (int num : array) {
        if (num == candidate)
            cnt++;
    }
    // 再次判断
    return cnt > array.length ? candidate : 0;
}
```



# 最小的K个数

用快速排序的分区法，定位到 k 这个数；此时在 k 前面的就是最小的 k 个数

用大顶堆思想，维护 k 个数的大顶堆，每挤出去一个数都比剩下的 k 个数要大；最后剩下的就是最小 k 个数

```java
public static ArrayList<Integer> GetLeastNumbers_Solution(int[] input, int k) {
    ArrayList<Integer> ret = new ArrayList<>();
    if (k >= input.length || k == 0)
        return ret;
    quickSort(input, 0, input.length - 1, k - 1);
    for (int i = 0; i < k; i++) {
        ret.add(input[i]);
    }
    return ret;
}

private static void quickSort(int[] input, int l, int h, int k) {
    while (l <= h) {
        int m = partition(input, l, h);
        if (m == k)
            break;
        if (m > k)
            h = m -1;
        else l = m + 1;
    }
}

private static int partition(int[] arr, int left, int hight) {
    int m = arr[left];
    // h+1 是为了下面在最开始的时候 --h;
    int l = left, h = hight + 1;
    while (l <= h) {
        // 这里不能l++ 的理由是，在判断 l 不满足的情况后，l依然会执行++
        // 这样逻辑就不对了；得改成 ++l，加 1 后再执行操作；
        // 还要 <= 操作，形成互补
        while (l <= h && arr[++l] <= m);
        while (l <= h && arr[--h] > m);
        if (l < h)
            swap(arr, l, h);
    }
    swap(arr,left,h);
    return h;
}

private static void swap(int[] arr, int i, int j) {
    int t = arr[i];
    arr[i] = arr[j];
    arr[j] = t;
}
```

## 维护一个大顶堆

```java
public ArrayList<Integer> GetLeastNumbers_Solution(int[] nums, int k) {
    if (k > nums.length || k <= 0)
        return new ArrayList<>();
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>((o1, o2) -> o2 - o1);
    for (int num : nums) {
        maxHeap.add(num);
        if (maxHeap.size() > k)
            maxHeap.poll();
    }
    // 这里就很厉害了
    return new ArrayList<>(maxHeap);
}	
```



# 字符流中第一个不重复的字符

用 256 int 数组计数；

每一个数计数一次，并交到 queue 队列中，while 循环确保队头元素为不重复字符

```java
 private int[] cnts = new int[256];
    private Queue<Character> queue = new LinkedList<>();

    public void Insert(char ch) {
        cnts[ch]++;
        queue.add(ch);
        // 这里有个 while 循环，每一次新加入都会找到第一个字符串
        while (!queue.isEmpty() && cnts[queue.peek()] > 1)
            queue.poll();
    }

    public char FirstAppearingOnce() {
        return queue.isEmpty() ? '#' : queue.peek();
    }
```



# 连续子数组的最大和

```java
public int FindGreatestSumOfSubArray(int[] array) {
    	// 有可能最大和为负，所以max初始化为 min_value
        int max = Integer.MIN_VALUE;
        int cnt = array[0];
        for (int i = 1; i < array.length; i++) {
            // 如果为负，那对接下来的累积没有积极作用
            // 所以重新计数
            if (cnt < 0)
                cnt = array[i];
            else
                cnt += array[i];
            max = Math.max(max, cnt);
        }
        return max;
    }
```



# 最长不含重复字符的子字符串

用 26 位数组保存字符出现下标，遍历滑动字符串，每次遇到重复字符计数一次

```java
public int longestSubStringWithoutDuplication(String str) {
    int maxlen = 0;
    int cntlen = 0;
    // 数组记录字符出现的下标
    int[] preIndexs = new int[26];
    // 初始化数组元素，都为 -1
    Arrays.fill(preIndexs,-1);
    for (int i = 0; i < str.length(); i++) {
        // 这里很妙，有点东西
        int index = str.charAt(i) - 'a';
        int preIndex = preIndexs[index];
        // 字符未出现过
        // 字符出现过，但并不在本次计数的范围内
        if (preIndex == -1 || i - preIndex > cntlen)
            cntlen++;
        else {
            // 字符重复出现，记录本次滑动的最长距离
            maxlen = Math.max(maxlen,cntlen);
            // 重新开始计数滑动窗口，从重复的那个字符开始
            cntlen = i - preIndex;
        }
        // 更新字符出现的最新下标
        preIndexs[index] = i;
    }
    // 滑动在最后是没比较直接退出循环的
    maxlen = Math.max(maxlen, cntlen);
    return maxlen;
}
```



# 丑数

关键是让数组中的每个数都跟 2，3，5相乘；每次取最小并且位置加 1

```java
 public int GetUglyNumber_Solution(int index) {
         if (index <= 0)
            return index;
        int[] dp = new int[index];
        int i2 = 0, i3 = 0, i5 = 0;
        dp[0] = 1;
        for (int i = 1; i < index; i++) {
            // 这里，让数组中的每个数都会被2，3，5相乘；
            // 假如第一次是三个数都与dp[0]相乘，2的结果比较小
            // 第二次，2* dp[1];3 * dp[0]; 5*dp[0]   3小
            // 第三次，2*dp[1];  3*dp[1];   5*dp[0]  5 小
            // 可以看到，数组中每个数都会与2 3 5相乘
            int next2 = dp[i2] * 2, next3 = dp[i3] * 3, next5 = dp[i5] * 5;
            int minTemp = Math.min(next2, Math.min(next3, next5));
            if (minTemp == next2)
                i2++;
            if (minTemp == next3)
                i3++;
            if (minTemp == next5)
                i5++;
            dp[i] = minTemp;
        }
        return dp[index - 1];
    }
```



# 数组中的逆序对

```java
	private int[] temp;
    private int cnt = 0;

    public int InversePairs(int[] array) {
        if (array == null || array.length == 0)
            return 0;
        temp = new int[array.length];
        mergeSort(array, 0, array.length - 1);
        return cnt;
    }
 
 private void mergeSort(int[] arr, int l, int p) {
        if (l < p) {
            int m = l + (p - l) >> 1;
            mergeSort(arr, l, m);
            mergeSort(arr, m + 1, p);
            merge(arr, l, p, m);
        }
    }

    private void merge(int[] arr, int l, int p, int m) {
        int left = l, right = m + 1;
        int index = left;
        while (left <= m && right <= p) {
            if (arr[left] >= arr[right])
                temp[index++] = arr[left++];
            else {
                temp[index++] = arr[right++];
                // 左右两边都有序，从小到大
                // 右边的比左边最小的还要小
                // 意味着左边剩下的数都要比右边这个数大
                // +1 是 m 也算进去了；
                cnt += m - left + 1;
            }
        }
        while (left <= m) {
            temp[index++] = arr[left++];
        }
        for (int i = l; i < index; i++) {
            arr[i] = temp[i];
        }
    }
```



# 数字在排序数组中出现的次数

```java
public static int GetNumberOfK(int[] array, int k) {
    int first = binarySearch(array, k);
    int last = binarySearch(array, k + 1);
    // 先检查长度，预防数组越界，有可能没找到，
    // first 为数组的长度；或者first 为 0，所以再判断 array[first] == k
    // 再判断大小值
    return (first < array.length && array[first] == k  ) ? last - first : 0;
}

// 下面方法要保证！找不到情况下，最大值是length；最小值是 0
private static int binarySearch(int[] arr, int k) {
    // 假设是 {1，2，3，3，3}；k + 1需要返回5；
    // p 不能等于 arr.length-1 ; 会返回 4
    // 所以，得是 length
    int le = 0, p = arr.length;
    while (le < p) {
        // 这个底是 le 在加，所以到最后求出的 mid 是 le
        // 到最后会是 le == h；两个指针会指向同一个值
        int mid = le + ((p - le) >> 1);
        // 只要有相等的嫌疑，p 就会直接等于 mid，一直向左逼近
        if (arr[mid] >= k)
            p = mid;
        // 否则，就是小于的情况，可以直接越过 mid
        else le = mid + 1;
    }
    return le;
}
```



# 二叉查找树的第 K 个结点

```java
/**
 * 递归；想象自己是在 pRoot 层中心；采用中序遍历，遍历结果就是从小到大
 * 那么 k 可能在左边，可能在右边；也有可能就是自己
 * 先找找左边的，找到我们就返回；
 * 没找到，判断我自身是否是，cnn++；
 * 自身不是，那就是在右子树，到右子树中寻找
 * 
 * 注意：不管是左子树的遍历，还是右子树的；它们都会判断自身，把 cnn++；
 * 如果是对应的 k；直接就返回了
 */
int cnn = 0;
TreeNode KthNode(TreeNode pRoot, int k) {
    if (pRoot == null)
        return pRoot;
    TreeNode left = KthNode(pRoot.left, k);
    if (left != null)
        // 在左结点中
        return left;
    cnn++;
    if (cnn == k)
        // 自己是这个结点
        return pRoot;
    // 在右结点中
    return KthNode(pRoot.right, k);
}
```



# 平衡二叉树

有两个条件：左子树和右子树都是平衡二叉树；左子树和右子树的高度差不超过1

下面这种解法，用 -1 表示子树不是平衡二叉树快速返回错误；然后返回高度判断高度差

```java
public boolean IsBalanced_Solution(TreeNode root) {
    return height(root) != -1;
}

private int height(TreeNode root) {
    if (root == null)
        return 0;
    // 左子树的高度
    int left = height(root.left);
    // 左子树已经不是平衡二叉树了
    if (left == -1) return -1;
    // 右子树的高度
    int right = height(root.right);
    // 右子树已经不是平衡二叉树了
    if (right == -1) return -1;
    // 判断左右子树的高度差，平衡二叉树的话 高度加上自己的节点
    return Math.abs(left - right) >= 1 ? -1 : Math.max(left, right) + 1;
}
```



# 翻转单词顺序列

```java
/**
 * 前后指针；每个单词先翻转，再给整条字符串作翻转
 * 用前后指针来定位到一个单词
 * 临界问题：最后一个单词，所以要 i <= n；而且翻转判断要把 n-1 放在前面；
 * 不然 chars[i] 会数组越界
 */
public String ReverseSentence(String str) {
    int n = str.length();
    int start = 0;
    char[] chars = str.toCharArray();
    for (int i = 0; i <= n; i++) {
        if (i == n - 1 || chars[i] == ' ') {
            // 当前 i 为空格
            reverse(chars, start, i - 1);
            // 越过当前空格
            start = i + 1;
        }
    }
    reverse(chars, 0, n - 1);
    return new String(chars);
}

private void reverse(char[] chars, int start, int end) {
    while (start < end)
        swap(chars, start++, end--);
}

private void swap(char[] chars, int a, int b) {
    char temp = chars[a];
    chars[a] = chars[b];
    chars[b] = temp;
}
```



# 滑动窗口的最大值

```java
public ArrayList<Integer> maxInWindows(int [] num, int size){
        int n = num.length;
        ArrayList<Integer> list = new ArrayList<>();
           if (size > n || size < 1)
       		 return list;
    	// 最大堆
        PriorityQueue<Integer> queue = new PriorityQueue<>((o1, o2) -> o2 - o1);
        for (int i = 0; i < size; i++) {
            //先把前 size 位加入到最大堆
            queue.add(num[i]);
        }
    	// 加入第一个数
        list.add(queue.peek());
        for (int i = 0, j = size; j < n; i++, j++){
            // 窗口滑动，去掉最开头的，保留最后面的
            queue.remove(num[i]);
            queue.add(num[j]);
            // 找出滑动窗口中最大的值
            list.add(queue.peek());
        }
        return list;
    }		
```

# 树中两个节点的最低公共祖先

```java
 public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
     	// 出口，如果找到 p == root || q == root
        if(root == null || p == root || q == root)
            return root;
        TreeNode leftNode = lowestCommonAncestor(root.left,p,q);
        TreeNode rightNode = lowestCommonAncestor(root.right,p,q);
        return leftNode == null ? rightNode : rightNode == null ? leftNode : root;
    }
```





# 旋转数组的最小数字

```java
public static int minNumberInRotateArray(int[] nums) {
    if (nums == null || nums.length == 0)
        return 0;
    int l = 0, p = nums.length - 1;
    while (l < p) {
        int mid = l + ((p - l) >>> 1);
        // 有特殊情况 1 1 1 1 0 1
        //  1 0 1 1 1 1
        // 判断是否为特殊情况
        if (nums[l] == nums[mid] && nums[p] == nums[mid]) {
            for (int i = l; i < p; i++) {
                if (nums[i] > nums[i + 1])
                    return nums[i + 1];
            }
            // 极限操作，还可以 1 1 1 1 1 的，此时上面的回不了
            return nums[l];
        }
        else if (nums[mid] <= nums[p])
            // 没重复情况下可以减一
            // 存在重复问题，p = mid，以l+1去推进
            p = mid;
        else l = mid + 1;
    }
    // 不管是 mid+1;还是p 跳到 mid 上，最后都会是以 l 退出
    return nums[l];
}
```