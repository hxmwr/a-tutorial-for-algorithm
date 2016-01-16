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

### 2. Quick Sort(single loop) ###
	void quick_sort(int *arr, int start, int end)
	{
		if(start == end){
			return;
		}
		
		int key = arr[start];
		int i = start, j = end, mid;
		while(i != j){
			if(arr[i] <= key){
				i += 1;
				if(arr[j] > key){
					j -= 1;
				}
			}else{
				if(arr[j] <= key){
					swap(arr[i],arr[j]);
					i += 1;
					j -= 1;
				}else{
					j -= 1;
				}
			}
		}
		
		quick_sort(arr, start, mid-1);
		quick_sort(arr, mid, end);
	}
