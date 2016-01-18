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
>This qsort algorithm's core idea is quite simple, before you rearrange the array, you take one element of it, then you just seperate the input array into two parts, one of which is less or equal than the standard value, while another part is greater than the standard value, you process these seperated parts with the same manner aforementioned recursively until there are no seperated part consist of more than one element and you'll get a sorted array.
