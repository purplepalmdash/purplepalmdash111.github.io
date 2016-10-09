+++
categories = ["Programming"]
date = "2016-10-09T15:28:18+08:00"
description = "LeetCode Problems"
keywords = ["LeetCode"]
title = "LeetCodeTips1"

+++
又到了一年一度的跳Cao准备期间了，来刷刷LeetCode，提升一下编程技巧，准备可能的鄙视或者是
被鄙视。    

### 1. Two Sum
问题:    
给定一个整型数组，编写一函数，返回值为两个数组的下标，两个下标所在的数组元素相加的和为
给定的数值。例如:    

```
给定 nums = [2,7,11,15], targe = 9,     
因为nums[0] + nums[1] = 2 + 7 = 9,
返回的数组应为[0, 1].
```

#### C语言版
用C语言我的解决方案如下:    

```c
/* Given nums = [2, 7, 11, 15], target = 9,
 *
 * Because nums[0] + nums[1] = 2 + 7 = 9,
 * return [0, 1].
 */

#include <stdio.h>
#include <stdlib.h>

/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

int* twoSum(int* nums, int numsSize, int target) {
	int i, j;

	for (i = 0; i < numsSize; i++)
	{
		for(j = i+1; j < numsSize; j++)
		{
			if(nums[i] + nums[j] == target)
			{
				// Call malloc() for return value.
				int* returnArray = malloc(2*sizeof(int));
				returnArray[0] = i;
				returnArray[1] = j;
				return returnArray;
			}
		}
	}
	// Not found, comes here.
	return NULL;
}

int main(void)
{
	int nums[4] = {2, 7, 11, 15};

	int *result = twoSum(nums, 4, 9);
	if(result != NULL)
	{
	        printf("result is %d, %d \n", result[0], result[1]);
	}

	// Call malloc in function twoSum(), so now will call free()
	if(result != NULL)
	{
	        free(result);
	}
        return 0;
}

```

心得1: malloc()/free()的调用需要在不同函数体中进行，因而可能存在内存泄漏的风险，使用
valgrind来检测可以看到，有两次请求两次释放动作，并没有内存泄漏:    

```
$ valgrind -v --leak-check=full ./TwoSum
......
==3283== HEAP SUMMARY:
==3283==     in use at exit: 0 bytes in 0 blocks
==3283==   total heap usage: 2 allocs, 2 frees, 1,032 bytes allocated
==3283== 
==3283== All heap blocks were freed -- no leaks are possible
==3283== 
==3283== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
==3283== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

心得2: 算法复杂度为O(n^2)， 因为有嵌套的for()循环. 官方给出的有通过哈希来做的，在C语言
中内建数据类型并不包括map，因而在后面用python来实现。   

心得3: 对函数的返回值需要检测，如free()掉一个NULL的地址。StackOverFlow上关于free(NULL)
的讨论如下：    
[http://stackoverflow.com/questions/1938735/does-freeptr-where-ptr-is-null-corrupt-memory](http://stackoverflow.com/questions/1938735/does-freeptr-where-ptr-is-null-corrupt-memory)    

看起来也不会有什么严重的后果。   


