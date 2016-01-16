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
		
		int key = arr[start];
		int i = start-1, j = end+1, mid;
		while(j > i){
			if(arr[i+1] <= key){
				i += 1;
				if(arr[j-1] >= key){
					j -= 1;
				}
			}else{//arr[i+1] > key
				if(arr[j+1] <= key){
					swap(arr[i+1],arr[j+1]);
					i += 1;
					j -= 1;
				}else{
					j -= 1;
				}
			}
		}
		
		mid = i;
		quick_sort(arr, start, mid);
		quick_sort(arr, mid, end);
	}
######
	//double loop
	void quick_sort()
	{
	
	}
