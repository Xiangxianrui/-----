# 二分查找
# 二分查找
二分查找在生活中运用的很多，理解起来也很简单，但是在真正做起来并不简单。
### 基本思想
二分查找的基本思想是，
1.对比当前数组的中位数和目标值
2.如果当前中位数大于目标值，说明目标值在数组中的位置（如果存在）在中间位置的右边，此时进行迭代，把左边界更改成上一次的中间值位置
3.再次对比左边界和右边界的中间值处的数值和目标值，
如果不相等，，直到最终（left==right），
如果相等，就返回目前的下标mid
4.直到循环结束，如果此时都匹配不上，就说明不存在，此时跳出循环，return -1

```c++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size()-1;
        int mid = (left+right)/2;
        while(left <= right)
        {
            if(nums[mid] == target){
                return mid;
            }
            else if(nums[mid] > target){//目标值在左边
                right = mid-1;
                mid = (left+right)/2;
            }
            else if(nums[mid] < target){//目标值在左边
                left = mid + 1;
                mid = (left+right)/2;
            }
        }
        return -1;
        
    }
};
```
### 主要注意的点：
* 右边界 right 应该控制为数组的长度-1，因为数组的下标是从 0 开始的，那么边界也就是 n-1
* 控制循环的条件应该设置成 left <= right，不能去掉=。因为有可能多次迭代后 left == right,此时 target 的值就是 (left+right)/2，也就是 mid的值，如果不加等于的话会直接跳出，错过这种情况
* 迭代的时候，如果已经判断了nums[mid] < target，那么 left 就应该更新成mid - 1 而不是 mid,因为这个时候其实已经判断出了 mid 是不等于 target 的，可以排除 mid。



#移除元素
# 移除元素
## 使用迭代器删除元素的方法
```c++
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        vector<int>::iterator it = nums.begin();
        int size = 0;
        while(it!=nums.end())
        {
            if(*it == val){
                it = nums.erase(it);
            }
            else{
                ++it;
            }
        }       
        return nums.size();    
    }
};
```
看到题目的一瞬间就想到了用迭代器删除元素的方法，这个方法跟移除数组中的奇数很相似，但会更简单。只需要一个一个访问 vector 中的元素就行，如果这个位置的值等于 val,就把他移除。     
但是这里有一个大家经常会犯的的错误，直接 nums.erase(it)。这样会导致迭代器失效（删除一个后，数组的大小在改变，后面的元素本来就会向前移动一位，也就是索引-1，这时候又++it,就会跳过一个元素）。解决方法就是每次都用 it 来接收 erase 的返回值（只想下一个迭代器位置）；       
这种方法还有一个有点，就是 erase会改变数字的长度，所以最后只需要返回 nums.size()就可以；

## 双指针法
双指针法的主要思想是，将后面所有不等于 val 的值都移动到数组的前面。            
想法倒是不难，可以用一个指针指向数组的开头，然后一直向后移动，判断是否等于 val，如果等于 val 就继续向后移动，如果不等于就移到前面去。这时候就遇到了一个问题，该怎么往前放呢，这也就是另一个指针的用处了。         
我们把用来检测元素的指针叫做快指针，记录目前已经添加的元素的位置（下一次添加元素的位置）的指针叫做慢指针。   
	在运行的时候，fast 在前面检测，如果相等，就说明这个元素是不需要的，此时 fast 应该++去探测下一个元素。如果不想等，那就应该把目前 fast 位置的值 nums[fast]赋给nums[slow],之后 fast 正常++,slow 位置在得到值后也应该++指向下一个位置。这个动作一直持续到探测完 nums 里 所有的元素。不难发现，在整个过程中，遇到一个非 val 的元素就会把它放到前面（也就是 slow的位置），所以 slow 也就是整个数组中非 val 的元素的个数。

	
	class Solution { public:int removeElement(vector<int>& nums, int val)
	{ 	int slow = 0;  //慢指针指向下一个有效位置         
	  	  int fast = 0;  //快指针遍历整个数组         
		  int n = nums.size();                  
		  while (fast < n)
		  {
		  if (nums[fast] != val)
		  	{//当前元素不需要移除，复制到慢指针位置                 
		  	nums[slow] = nums[fast];                 
		  	slow += 1;
		  	}             
		fast += 1;  // 无论是否匹配，快指针都前进         
			}
		return slow;              } 
	};
	
	



# 有序数组的平方

## 思路一：先全都平方，再 sort 一遍

一开始看到题目没多想，就想着把所有元素都平方一下，然后排序不就行了？代码超级简单，直接看：
```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        for(int i = 0; i < nums.size(); i++){
            nums[i] = nums[i] * nums[i];
        }
        sort(nums.begin(), nums.end());
        return nums;
    }
};

缺点也很明显：用了 sort()，时间复杂度是 O(nlogn)，不是最优解。

⸻

## 方法二：双指针，从两边夹

再一看题目，说是“原来数组有序”，那就不能浪费这信息了。

这个时候就想到双指针法。左边一个指针、右边一个指针，比较两边的绝对值谁大，谁的平方放到结果数组的末尾，然后指针向中间靠。

核心逻辑：
	1.	两个指针 left 和 right；
	2.	创建一个 res 数组，从最后一个位置往前放；
	3.	谁的平方大，就放谁的，指针相应地往中间移动。

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        int left = 0, right = nums.size() - 1;
        int pos = nums.size() - 1;
        vector<int> res(nums.size());

        while(left <= right){
            int l = nums[left] * nums[left];
            int r = nums[right] * nums[right];

            if(l > r){
                res[pos--] = l;
                left++;
            } else {
                res[pos--] = r;
                right--;
            }
        }

        return res;
    }
};









