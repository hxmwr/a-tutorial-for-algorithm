## A Tutorial To Algorithms (part 1) ##
### Introduction ###
These stuffs are not organized purpersely, nor it's a note of the book ***Introduction Algorithms***. Mostly it is like a sequence of my personal footprints when I learned Algorithm, which record my ideas, inspirations and deep thoughts. I hope to review them often and make these knowlegement more firm in my brain. That's a good learning method I think, also a good excersise for brain which would make the brain more sharp and effcient on normal problems of the computer world.
### 1. Insert Sort ###
	void insert_sort(int *arr, int len)
	{
		int key, i, j;
		for(i = 1; i < len; i++){
			key = arr[i];
			j = i;
			while(--j>-1){
				if(key < arr[j]){
					arr[j+1] = arr[j];
					arr[j] = key;
				}else{
					arr[j+1] = key;
					break;
				}
			}
		}
	}

### 2. Quick Sort ###
	//single loop
	void quick_sort(int *arr, int start, int end)
	{
		if(start == end){
			return;
		}
	
		int i = start, j = end, key = arr[start], mid;
	
		while(i!=j){
			if(arr[i] >= key){
				if(arr[j] < key){
					swap(arr[j], arr[i]);
					if(( j-i) > 2){
						if(arr[i+1] < key){
							i++;
						}
						j--;
					}else if((j-i) == 1){
						j--;
					}else{
						if(arr[i+1] < key){
							i++;
							j--;
						}else{
							j -= 2;
						}
					}
				}else{
					j--;
				}				
			}else{
				if(arr[j] < key){
					i++;
				}else{
					if((j - i) > 2){
						if(arr[i+1] < key){
							i++;
						}
						j--;
					}else if((j-i) == 2){
						if(arr[i+1] < key){
							i++;
							j--;
						}else{
							j -= 2;
						}
					}else{
						j--;
					}
				}
			}
		}
	
		mid = i;
		quick_sort(arr, start, mid);
		quick_sort(arr, mid + 1, end);
	}
######
	//double loop
	void quick_sort(int* x,int* y)
	{
	    int temp,w,* u,*v;
	    u=x;v=y;w=*y;
	    if(x==y)return;
	    while(u!=v)//开始调换元素位置，大于等于w的放在右边，其余放在左边
	    {
	        if(*u>w)//检查当前u所指向的元素是否大于末尾元素的值，是则执行下面语句块
	        {
	            //向左查找可以交换的元素
	            while(u!=v)
	            {
	                if(*v<=w){temp=*u;*u=*v;*v=temp;break;}//找到这样一个元素，则立即进行交换
	                v--;
	            }
	        }
	        if(u==v)break;
	        u++;
	    }
	    quick_sort(x,u-1);
	    quick_sort(u,y);
	}
>NOTE<br>
>This qsort algorithm's core idea is quite simple, before you rearrange the array, you take one element of it, then you just seperate the input array into two parts, one of which is less or equal than the standard value, while another part is greater than the standard value, you process these seperated parts with the same manner aforementioned recursively until there are no seperated part consist of more than one element and you'll get a sorted array. But the actual codes of this algorithm are something confusing, you have to keep two pointers, one of which moves from the most left to right, another moves from the most right to left, in every loop you dereference these two pointers and base on these two values you decide whether or not swap the target elements and  move these pointers. At the same time you should deal with some side effects carefully which are not so straightforward.

### 3.Merge Sort ###
	void merge(int *s1, int *e1, int *s2, int *e2, int *temp)
	{
		int len = (e1 - s1) + (e2 - s2) + 2;
		int idx;
		for(int i=0,k=0,idx=0;idx<len;idx++){
			if((s1 + i) > e1){
				temp[idx] = *(s2 + k);
				k++;
				continue;
			}
	
			if((s2 + k) > e2){
				temp[idx] = *(s1 + i);
				i++;
				continue;
			}
	
			if(*(s1 + i) <= *(s2 + k)){
				temp[idx] = *(s1 + i);
				i++;
			}else{
				temp[idx] = *(s2 + k);
				k++;
			}
		}
	
		for(int j=0;j<len;j++){
			s1[j] = temp[j];
		}
	}
	void merge_sort(int *start, int *end, int *temp)
	{
		if(start == end)return;
		int *arr1_start = start, *arr1_end, *arr2_start, *arr2_end = end;
		int distance = end - start;
		int mid = distance/2;
		arr1_end = arr1_start + mid;
		arr2_start = arr1_end +1;
	
		merge_sort(arr1_start, arr1_end, temp);
		merge_sort(arr2_start, arr2_end, temp);
		merge(arr1_start, arr1_end, arr2_start, arr2_end, temp);
	}
>NOTE<br>
The idea behind this algorithm is clear which is seperating the original array into two parts and sort them seperately by ***merge sort*** itself also known as recursive invoke, then merge them in another block of empty memory space which is the same size with the original array. The key point for this is ***merge***, in each loop you sequencely pick a smaller element out from these two seperated arraies and put it into the newly applied memory one by one until there is no elements left in these two arraies. The below gives another ***merge*** method which is slightly different from the above one.

	void Merge(int sourceArr[],int tempArr[], int startIndex, int midIndex, int endIndex)
	{
	    int i = startIndex, j=midIndex+1, k = startIndex;
	    while(i!=midIndex+1 && j!=endIndex+1)
	    {
	        if(sourceArr[i] >= sourceArr[j])
	            tempArr[k++] = sourceArr[j++];
	        else
	            tempArr[k++] = sourceArr[i++];
	    }
	    while(i != midIndex+1)
	        tempArr[k++] = sourceArr[i++];
	    while(j != endIndex+1)
	        tempArr[k++] = sourceArr[j++];
	    for(i=startIndex; i<=endIndex; i++)
	        sourceArr[i] = tempArr[i];
	}
	 
	//内部使用递归
	void MergeSort(int sourceArr[], int tempArr[], int startIndex, int endIndex)
	{
	    int midIndex;
	    if(startIndex < endIndex)
	    {
	        midIndex = (startIndex + endIndex) / 2;
	        MergeSort(sourceArr, tempArr, startIndex, midIndex);
	        MergeSort(sourceArr, tempArr, midIndex+1, endIndex);
	        Merge(sourceArr, tempArr, startIndex, midIndex, endIndex);
	    }
	}
