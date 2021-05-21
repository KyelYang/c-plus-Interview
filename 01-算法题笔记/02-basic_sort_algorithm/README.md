# 基础排序算法

- 稳定：如果a原本在b前面，而a=b，排序之后a仍然在b的前面。
- 不稳定：如果a原本在b的前面，而a=b，排序之后 a 可能会出现在 b 的后面。  

- 插入型排序
  - 插入排序
  - 希尔排序
- 交换型排序
  - 冒泡排序
  - 快速排序
- 选择型排序
  - 简单选择排序
  - 堆排序
- 其他排序
  - 归并排序
  - 基数排序
  - 桶排序
  
  
## 1. 插入排序
### 平均时复O(N^2)，最坏时复O(N^2)，最好时复O(N)，空复O(1)，稳定

插入排序（Insertion-Sort）的算法描述是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。  

插入排序在几乎排列好的队列中效率最高，因为在有序的队列中，其完全不用赋值，只需要进行n−1次比较即完成排序。  

### 算法描述
一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：  
- 从第一个元素开始，该元素可以认为已经被排序；
- 取出下一个元素，在已经排序的元素序列中从后向前扫描；
- 如果该元素（已排序）大于新元素，将该元素移到下一位置；
- 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置；
- 将新元素插入到该位置后；
- 重复步骤2~5。

```CPP
void insert_sort(vector<int>& nums){
    int n = nums.size();
    if(n < 2) return;
    for(int i = 1;i < n;++i){  //进行n - 1趟排序
        int tmp = nums[i];
        int j = i; //j从i开始向前枚举，也即向前查找合适位置进行插入
        while(j > 0 && tmp < nums[j - 1]) nums[j--] = nums[j - 1];  //只要tmp小于nums[j - 1]，就把j-1上的元素后移一位
        nums[j] = tmp;  //找到插入位置，将nums[j]上的元素插入
    }
}
```   

## 2. 希尔(shell)排序
### 平均时复O(NlogN)，最坏时复O(O(N^s) 1<s<2,增量区间影响s值)，最好时复(不详)，空复O(1)，不稳定

1959年Shell发明，第一个突破O(n2)的排序算法，是简单插入排序的改进版。它与插入排序的不同之处在于，它会优先比较距离较远的元素。希尔排序又叫缩小增量排序。  


Shell排序通过将数据分成不同的组，先对每一组进行排序，然后再对所有的元素进行一次插入排序，以减少数据交换和移动的次数。平均效率是O(nlogn)。其中分组的合理性会对算法产生重要的影响。  

### 算法描述
先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，具体算法描述：

- 选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
- 按增量序列个数k，对序列进行k 趟排序；
- 每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。  


```CPP
void shell_sort(vector<int>& nums){
    int n = nums.size();
    for(int group = n / 2;group > 0;group /= 2){    //根据增量序列进行遍历，增量区间长度每次减半
        for(int i = group;i < n;i++){
            //下面就是插入排序的过程
            int tmp = nums[i];
            int j = i;
            for(;j - group >= 0 && tmp < nums[j - group];j -= group) nums[j] = nums[j - group];
            nums[j] = tmp;
        }
    }
}
```

## 3. 冒泡排序
### 平均时复O(N^2)，最坏时复O(N^2)，最好时复O(N)，空复O(1)，稳定  

从后往前分别两两对比，遇到前一个比后一个小，就进行交换，这样每一轮下来就会有一个最小数被交换到前面，以此类推。  

冒泡排序是最慢的排序算法。在实际运用中它是效率最低的算法。只要数据一多，排序效率将会大打折扣。  

### 算法描述
- 比较相邻的元素。如果第一个比第二个大，就交换它们两个；
- 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对，这样在最后的元素应该会是最大的数；
- 针对所有的元素重复以上的步骤，除了最后一个；
- 重复步骤1~3，直到排序完成。

```CPP
void bubble_sort(vector<int>& nums){
    int n = nums.size();
    if(n < 2) return;
    bool flag;
    for(int i = 1;i < n;++i){   //每次冒泡的长度
        flag = false;   //初始化标志位
       for(int j = 0;j < n - i;++j){    //第i趟排序从nums[0]到nums[n - i - 1]
           if(nums[j] > nums[j + 1]){
               swap(nums[j],nums[j + 1]);
               flag = true; 
           }
       }
       if(!flag) return;    //如果之前的循环中不存在元素交换的话，说明当前序列已经有序，直接退出
    }
}
```

## 4. 快速排序
### 平均时复O(NlogN)，最坏时复O(NlogN)，最好时复O(N^2)，空复O(logN)，不稳定  

快速排序的基本思想：通过一趟排序将待排记录分隔成独立的两部分，其中一部分记录的关键字均比另一部分的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序。  

快速排序是一个就地排序，分而治之，大规模递归的算法。从本质上来说，它是归并排序的就地版本。  

快速排序比大部分排序算法都要快。尽管我们可以在某些特殊的情况下写出比快速排序快的算法，但是就通常情况而言，没有比它更快的了。快速排序是递归的，因此需要使用一个小的辅助栈来实现递归（O(logN)），对于内存非常有限的机器来说，它不是一个好的选择。  

### 算法描述
快速排序使用分治法来把一个串（list）分为两个子串（sub-lists）。具体算法描述如下：

- 从数列中挑出一个元素，称为 “基准”（pivot）；
- 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；
- 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

```CPP
void quick_sort(vector<int>& nums,int left, int right){
    if(left < right){   //当前区间长度大于1
        int pos = _partition(nums,left,right);
        quick_sort(nums,left,pos - 1);
        quick_sort(nums,pos + 1,right);
    }
}

int _partition(vector<int>& nums,int left, int right){
    int tmp = nums[left];
    while(left < right){
        while(left < right && nums[right] > tmp) --right;
        nums[left] = nums[right];
        while(left < right && nums[left] <= tmp) ++left;
        nums[right] = nums[left];
    }
    nums[left] = tmp;
    return left;
}
```

## 5. 简单选择排序
### 平均时复O(N^2)，最坏时复O(N^2)，最好时复O(N^2)，空复O(1)，不稳定  

选择排序(Selection-sort)是一种简单直观的排序算法。它的工作原理：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。   

简单选择排序与冒泡排序都是基于交换的排序算法，效率都是 O(n2)。在实际应用中处于和冒泡排序基本相同的地位。它们只是排序算法发展的初级阶段，在实际中使用较少。  

### 算法描述
n个记录的直接选择排序可经过n-1趟直接选择排序得到有序结果。具体算法描述如下：  

- 初始状态：无序区为R\[1..n]，有序区为空；
- 第i趟排序(i=1,2,3…n-1)开始时，当前有序区和无序区分别为R\[1..i-1]和R\[i..n)。该趟排序从当前无序区中-选出关键字最小的记录 R\[k]，将它与无序区的第1个记录R交换，使R\[1..i]和R\[i+1..n)分别变为记录个数增加1个的新有序区和记录个数减少1个的新无序区；
- n-1趟结束，数组有序化了。

```CPP
void selectSort(vector<int>& nums){
    int n = nums.size();
    if(n < 2) return;
    for(int i = 0;i < n - 1;++i){   //找第i小的数
        int k = i;
        for(int j = i;j < n;++j){
            if(nums[j] < nums[k]) k = j;    //从第i个位置向后查找，找到小于nums[k]的数，用k记录该数的下标
        }
        swap(nums[i],nums[k]);  //将nums[k]上第i小的数与nums[i]进行交换
    }
}
```

## 6. 堆排序(大顶堆)  
### 平均时复O(NlogN)，最坏时复O(NlogN)，最好时复O(NlogN)，空复O(1)，不稳定  

堆排序（Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。  

堆排序适合于数据量非常大的场合（百万数据）。  

堆排序不需要大量的递归或者多维的暂存数组。这对于数据量非常巨大的序列是合适的。比如超过数百万条记录，因为快速排序，归并排序都使用递归来设计算法，在数据量非常大的时候，可能会发生堆栈溢出错误。  

### 算法描述
- 将初始待排序关键字序列(R1,R2….Rn)构建成大顶堆，此堆为初始的无序区；
- 将堆顶元素R\[1]与最后一个元素R\[n]交换，此时得到新的无序区(R1,R2,……Rn-1)和新的有序区(Rn),且满足R\[1,2…n-1]<=R\[n]；
由于交换后新的堆顶R\[1]可能违反堆的性质，因此需要对当前无序区(R1,R2,……Rn-1)调整为新堆，然后再次将R\[1]与无序区最后一个元素交换，- - 得到新的无序区(R1,R2….Rn-2)和新的有序区(Rn-1,Rn)。不断重复此过程直到有序区的元素个数为n-1，则整个排序过程完成。  

```CPP
void heap_sort(vector<int>& nums){  //堆排序
    int n = nums.size();
    if(n < 2) return;
    create_heap(nums);  //创建大顶堆
    for(int i = n - 1;0 <= i;--i){  //从后往前，将堆顶节点和数组最右边的节点交换
        swap(nums[i],nums[0]); 
        down_adjust(nums,0,i - 1);  //然后在0~(i-1)范围内向下调整堆
    }
}

void create_heap(vector<int>& nums){    //建堆时间复杂度为O(n)
    int n = nums.size();
    for(int i = n / 2 - 1;i >= 0;--i) down_adjust(nums,i,n - 1);    //从第一个非叶子节点开始向下调整堆
}

void down_adjust(vector<int>& nums,int left, int right){    //向下更新堆，时间复杂度为O(logn)
    int i = left * 2 + 1;   //节点left的左孩子下标为left * 2 + 1
    while(i <= right){
        if(i + 1 <= right && nums[i + 1] > nums[i]) i = i + 1;  //若右子节点存在，且右子节点大于左子节点，则将i的下标更新为右子节点下标
        if(nums[i] > nums[left]){   //若i中存储的孩子节点值大于父节点值，则将其交换，并进行下一轮比较；否则更新结束
            swap(nums[i],nums[left]);
            left = i;
            i = left * 2 + 1;
        }else break;
    }
}

//堆的其他操作

void delete_top(vector<int>& nums){ //删除堆顶元素
    int n = nums.size();
    swap(nums[0],nums[n - 1]);  //将堆顶元素和数组最后一个元素交换
    nums.erase(nums.end() - 1); //删除数组最后一个元素
    down_adjust(nums,0,n - 2);  //从堆顶开始更新堆
}

void insert(vector<int>& nums,int x){   //插入元素
    nums.push_back(x);  //将元素加到数组末尾
    up_adjust(nums,0,nums.size());  //从下往上更新堆
}

void up_adjust(vector<int>& nums,int left, int right){  //向上更新堆
    int i = right, j = (right - 1) / 2; //j为父节点
    while(left <= j){
        if(nums[i] > nums[j]){  //如果父节点值小于i节点的值，则将值互换
            swap(nums[i],nums[j]);
            i = j;  //保持i为预调整节点，j为父节点
            j = (j - 1) / 2;
        }else break;
    }
}
```

## 7. 归并排序
### 平均时复O(NlogN)，最坏时复O(NlogN)，最好时复O(NlogN)，空复O(N)，稳定
归并排序是建立在归并操作上的一种有效的排序算法。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为2-路归并。   

归并排序比堆排序稍微快一点，但是需要比堆排序多一倍的内存空间，因为它需要一个额外的数组。  

### 算法描述
- 把长度为n的输入序列分成两个长度为n/2的子序列；
- 对这两个子序列分别采用归并排序；
- 将两个排序好的子序列合并成一个最终的排序序列。  

```CPP
void merge_sort(vector<int>& nums,int left, int right){
    //递归写法：推荐
    if(left < right){
        int mid = (left + right) / 2;   //取中点
        merge_sort(nums,left,mid);  //递归，将左子区间[left,mid]归并排序
        merge_sort(nums,mid + 1,right); //递归，将右子区间[mid + 1，right]归并排序
        merge(nums,left,mid,mid + 1,right); //将左子区间与右子区间合并
    }
    
    //递推写法
    int n = nums.size();
    if(n < 2) return;
    int step = 1;
    while(step < n){
        step *= 2;  //每次合并的宽度
        for(int i = 0;i < n;i += step){
            int mid = i + step/2 - 1;   //求左半边区域的右边界，一定要减一，不然出错
            if(mid + 1 < n) merge(nums,i,mid,mid + 1,min(i + step - 1,n - 1));  //将左子区间与右子区间合并
        }
    }
}

void merge(vector<int>& nums,int l1,int r1,int l2,int r2){  //合并两个有序数组，与合并两个有序链表思路一样
    vector<int> vec;
    int i = l1,j = l2;
    while(i <= r1 && j <= r2){
        if(nums[i] < nums[j]) vec.push_back(nums[i++]);
        else vec.push_back(nums[j++]);
    }
    while(i <= r1) vec.push_back(nums[i++]);
    while(j <= r2) vec.push_back(nums[j++]);

    copy(vec.begin(),vec.end(),nums.begin() + l1);
}
```

## 8. 计数排序（用空间换时间）
### 平均时复O(N + K)，最坏时复O(N + K)，最好时复O(N + K)，空复O(N + K)，稳定
### 排序思路
- [计数排序（Counting Sort）](https://zhuanlan.zhihu.com/p/100158406)
- [Counting Sort Algorithm](https://www.programiz.com/dsa/counting-sort)
### 具体实现
上述思路是用数组下标来实现的，但如果数组中包含负数，用下标就不可行，因此改用unordered_map替代计数数组
```CPP
void countSort(vector<int>& nums){
    //数组中的元素必须都大于零，否则的话
    int n = nums.size();
    int max_val = nums[0];
    //找出数组的最大值
    for(auto num : nums) max_val = max(max_val,num);
    //定义计数数组
    unordered_map<int,int> count;
    //进行元素计数
    for(auto num : nums) count[num]++;
    //将元素计数结果累加
    for(int i = 1;i <= max_val;i++) count[i] += count[i - 1];
    //将结果保存在结果数组中
    vector<int> res(n,0);
    for(int i = 0;i < n;i++){
        res[count[nums[i]] - 1] = nums[i];
        count[nums[i]]--;
    }
    //将最终结果复制到nums数组中
    nums = res;
}
```

## 9. 桶排序（以后遇到再总结）
