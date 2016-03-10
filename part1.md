## A Tutorial To Algorithms (part 1) ##
### Introduction ###
These stuffs are not organized purpersely, nor it's a note of the book ***Introduction Algorithms***. Mostly it is like a sequence of my personal footprints when I learned Algorithm, which record my ideas, inspirations and deep thoughts. I hope to review them often and make these knowlegement more firm in my brain. That's a good learning method I think, also a good excersise for brain which would make the brain more sharp and effcient on normal problems of the computer world.
### 1. Insert Sort ###
``` c
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
```
### 2. Quick Sort ###
``` c
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
```
######
``` c
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
```
>**NOTE**<br>
>This qsort algorithm's core idea is quite simple, before you rearrange the array, you take one element of it as standard value, then you just seperate the input array into two parts, one of which is less or equal than the standard value, while another part is greater than the standard value, you process these seperated parts with the same manner aforementioned recursively until there are no seperated part consist of more than one element and you'll get a sorted array. But the actual codes of this algorithm are something confusing, you have to keep two pointers, one of which moves from the most left to right, another moves from the most right to left, in every loop you dereference these two pointers and base on these two values you decide whether or not swap the target elements and  move these pointers. At the same time you should deal with some side effects carefully which are not so straightforward.

### 3.Merge Sort ###
``` c
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
```
>**NOTE**<br>
The idea behind this algorithm is clear which is seperating the original array into two parts and sort them seperately by ***merge sort*** itself also known as recursive invoke, then merge them in another block of empty memory space which is the same size with the original array. The key point for this is ***merge***, in each loop you sequencely pick a smaller element out from these two seperated arraies and put it into the newly applied memory one by one until there is no elements left in these two arraies. The below gives another ***merge*** method which is slightly different from the above one.

``` c
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
```
### 4.Heap Sort
``` c
//自下而上建堆
void build_heap(int a[],int len)
{
	int i=len;
	bool f=false;
	while((i*2)>len)
	{
		int j=i;
		while(j!=1)
		{
			int& c=(int&)(max(a[j/2-1],max(a[j-1],j/2?a[j-2]:a[j])));
			if(c!=(a[j/2-1])){swap(c,a[j/2-1]);f=true;}
			j/=2;
		}
		i-=2;
	}
	if(f)build_heap(a,len);
	else return;
}
//取最大值放在数组末尾，其余元素再进行大顶堆创建
void heap_sort(int a[],int len)
{
	build_heap(a,len);
	while(len>1)
	{
		swap(a[0],a[len-1]);
		swap(a[len-1],a[len]);
		build_heap(a,len--);
	}
	swap(a[0],a[1]);
}
```
``` c
//自上而下建堆
//注意左右孩子节点公式 left_child = 2*cur_idx+1, right_child = left_child + 1
void heap_build(int *arr, int cur_idx, int arr_len)
{
	int left_child;
	int right_child;
	left_child = 2*cur_idx+1;
	right_child = left_child + 1;

	if(left_child < arr_len && right_child < arr_len){
		heap_build(arr,left_child,arr_len);
		heap_build(arr,right_child,arr_len);
		if(arr[cur_idx] < arr[left_child] && arr[left_child] < arr[right_child]){
			swap(arr[cur_idx],arr[right_child]);
		}else if(arr[cur_idx] < arr[left_child]){
			swap(arr[cur_idx], arr[left_child]);
		}
	}else if(left_child < arr_len && arr[cur_idx] < arr[left_child]){
		swap(arr[cur_idx], arr[left_child]);
	}

	return;
}

void heap_sort(int *arr, int arr_len)
{
	while(arr_len > 1){
		heap_build(arr, 0, arr_len);
		swap(arr[0], arr[arr_len-1]);
		arr_len--;
	}
}
```
### 5.Select Sort
```C++
void select_sort(int *arr, int arr_len)
{
	for(int i=0;i<arr_len-1;i++){
		for(int j=i+1;j<arr_len;j++){
			if(arr[i] >= arr[j]){
				swap(arr[i], arr[j]);
			}
		}
	}
}

```
### 6.Find minimum number of coins that make a given value
```C++
int minCoins(int coins[], int m, int V)
{
	if(V == 0) return 0;
	
	int res = INT_MAX;
	
	for(int i=0; i<m; i++)
	{
		if(coins[i] <= V)
		{
			int sub_res = minCoins(coins, m, v-coins[i];
			if(sub_res != INT_MAX && sub_res + 1 < res)
				res = sub_res + 1;
		}
	}
	
	return res;
}
```
###7.Longest Increasing Subsequence
```C++
// O(nlgn)
int _tmain(int argc, _TCHAR* argv[])
{
	int arr[] = {1,5,2,9,3,5,7,9,6,9,9,10,10,8};
	int n = sizeof(arr)/sizeof(arr[0]);
	int count = 0;
	int max_len = 0;
	for(int i=0;i<n;i++){
		count++;
		if((i+1)>=n || arr[i]>arr[i+1]){
			if(count>max_len){
				max_len = count;
			}
			count = 0;
		}
	}

	cout<<max_len<<endl;
	system("pause");
	return 0;
}
```
```C++
// O(n^2)
int lis(int A[], int n){
    int *d = new int[n];
    int len = 1;
    for(int i=0; i<n; ++i){
        d[i] = 1;
        for(int j=0; j<i; ++j)
            if(A[j]<=A[i] && d[j]+1>d[i])
                d[i] = d[j] + 1;
        if(d[i]>len) len = d[i];
    }
    delete[] d;
    return len;
}
```
>***NOTE***<br>
To solve this problem you must know what is the `Longest Increasing Subsequence`, The `Subsequence` is a increasing sequence and it may not be a continuous sequence, so there maybe a lot of possible subsequences, which make the problem a little more complexity. 

### 8.Shortest Path In Undirected Graph



![](http://my.csdn.net/uploads/201205/09/1336492910_7645.jpg)

```C++
#include "stdafx.h"
#include <iostream>
#include <vector>
using namespace std;

#define MAX_NODES 100
#define PATH_LEN 100

typedef struct ANode
{
	int adjvex;
	ANode * next_arc;
	int weight;
} ArcNode;

typedef struct VNode
{
	ArcNode * first_arc;
} VexNode;

typedef struct G
{
	VexNode nodes[MAX_NODES];
	int n;
} Graph;

void AddNode(Graph &g, int arcs[][2], int n)
{
	VexNode v;
	v.first_arc = NULL;
	g.nodes[n] = v;
	
	if(g.n = 0) return;

	for(int i=0;i<n;i++){
		ArcNode &anode = *(new ArcNode);
		anode.adjvex = n;
		anode.weight = arcs[i][1];
		anode.next_arc = g.nodes[arcs[i][0]].first_arc;
		g.nodes[arcs[i][0]].first_arc = &anode;

		ArcNode &anode2 = *(new ArcNode);
		anode2.adjvex = arcs[i][0];
		anode2.weight = arcs[i][1];
		anode2.next_arc = g.nodes[n].first_arc;
		g.nodes[n].first_arc = &anode2;
	}
	g.n += 1;
}

void MinPath(Graph &g, int start, int end, int path[], int n, int path_weight, int &min_weight, int min_path[])
{
	if(start == end){
		if(path_weight < min_weight){
			min_weight = path_weight;
			memset(min_path, 0, PATH_LEN*4);
			memcpy(min_path, path, n*4);
		}

		return;
	}
	ArcNode *p = g.nodes[start].first_arc;

	while(p != NULL){
		int flag = 0;
		for(int i=0;i<n;i++){
			if(path[i] == p->adjvex) {
				flag = 1;
				break;
			}
		}

		if(flag){
			p = p->next_arc;
			continue;
		}else{
			path[n] = p->adjvex;
			MinPath(g, p->adjvex, end, path, n+1, path_weight+p->weight, min_weight, min_path);
			p = p->next_arc;
		}
	}
}

void AddArcs(Graph &g, int arcs[][3], int n)
{
	for(int i=0;i<n;i++){
		ArcNode &anode = *(new ArcNode);
		anode.adjvex = arcs[i][1];
		anode.weight = arcs[i][2];
		anode.next_arc = g.nodes[arcs[i][0]].first_arc;
		g.nodes[arcs[i][0]].first_arc = &anode;

		ArcNode &anode2 = *(new ArcNode);
		anode2.adjvex = arcs[i][0];
		anode2.weight = arcs[i][2];
		anode2.next_arc = g.nodes[arcs[i][1]].first_arc;
		g.nodes[arcs[i][1]].first_arc = &anode2;
	}
}

int _tmain(int argc, _TCHAR* argv[])
{
	Graph g;
	g.n = 12;

	for(int i=0;i<g.n;i++){
		g.nodes[i].first_arc = NULL;
	}

	int arr[][3] = {
		{0,3,2},
		{0,2,5},
		{0,1,3},
		{1,2,4},
		{3,7,3},
		{3,2,2},
		{2,6,6},
		{7,6,8},
		{1,4,10},
		{2,5,1},
		{4,5,4},
		{5,11,2},
		{6,11,2},
		{4,8,2},
		{5,10,8},
		{11,10,5},
		{8,10,6},
		{8,9,1},
		{9,10,9}
	};

	AddArcs(g, arr, sizeof(arr)/sizeof(arr[0]));
   
	int path[PATH_LEN];
	int min_path[PATH_LEN];
	int min_weight = INT_MAX;

	path[0] = 0;
	MinPath(g, 0, 4, path, 1, 0, min_weight, min_path);

	//show adjlist
	for(int j=0;j<g.n;j++){
		ArcNode *p = g.nodes[j].first_arc;
		cout<<j<<"|";
		while (p!=NULL){
			cout<<"->"<<p->adjvex<<"["<<p->weight<<"]";
			p = p->next_arc;
		}
		cout<<endl;
	}

    system("pause");
    return 0;
}
```
```C++
//Dijkstra Algorithm
void ShortestPath_DIJ(Graph &g, int v)
{
	int S[MAX_NODES][2];
	S[0][0] = v;
	S[0][1] = 0;
	g.nodes[v].in_S = 1;
	int n = 1;
	while(n <= g.n)
	{
		int u;
		int min = INT_MAX;
		for(int i=0;i<n;i++){
			ArcNode *p = g.nodes[S[i][0]].first_arc;
			while(p != NULL){
				if(g.nodes[p->adjvex].in_S == 0){
					if((p->weight + S[i][1]) < min){
						min = p->weight + S[i][1];
						u = p->adjvex;
					}			
				}
				p = p->next_arc;
			}
		}
		S[n][0] = u;
		S[n][1] = min;
		g.nodes[u].in_S = 1;
		n++;
	}
}
```
```C++
//The Floyd Algorithm is really clear, it gives the shortest path length of each pair of vertex, but it can not give the complete path info.
void ShortestPath_FLOYD(int M[][4])
{
	for(int k=0;k<4;k++){
		for(int i=0;i<4;i++){
			for(int j=0;j<4;j++){
				if(M[i][j]>(M[i][k]+M[k][j])){
					M[i][j] = M[i][k]+M[k][j];
				}
			}
		}
	}
}
```
###9. Max Value In Bag
```C++
#include "stdafx.h"
#include <iostream>
#include <vector>
using namespace std;

void bag01(int things[][3], int n, int M, int idx, int m, int value, int &maxV, int in_bag[], int res[])
{
	things[idx][2] = 1;
	if((things[idx][0] + m) <= M){
		m += things[idx][0];
		value += things[idx][1];
		in_bag[idx] = 1;
	}

	int i;
	int flag = 0;
	for(i=0;i<n;i++){
		if(things[i][2] == 0){
			flag = 1;
			bag01(things, n, M, i, m, value, maxV, in_bag,res);
		}
	}

	if(!flag){
		if(value > maxV){
			maxV = value;
			memcpy(res,in_bag,20);
		}	
	}

	in_bag[idx] = 0;
	things[idx][2] = 0;
}

int _tmain(int argc, _TCHAR* argv[])
{
	
	int arr[][3] = {
		{2,6,0},
		{2,3,0},
		{6,5,0},
		{5,4,0},
		{4,6,0}
	};

	int in_bag[5];
	int res[5];
	memset(in_bag, 0, 20);
	memset(res, 0, 20);
	int maxV = 0;
	for(int i=0;i<5;i++){
		bag01(arr,5,10,i,0,0,maxV,in_bag,res);
	}

	cout<<maxV<<endl;
    system("pause");
    return 0;
}
```
```C++
//This algorithm works as the above one, but consume less time. 
//And I conclude that if you want to build a recursive algorithm, 
//you have to find a sub-problem which has the same structure as its parent problem, 
//further more, if you want to make the algorithm more efficient, you must find a good sub-problem, 
//just like these two algorithms, one of which has a good sub-problem, another of which has a bad sub-problem. 
int package01(int M, int A[][3], int V, int n)
{
	int v1, v2;
	for(int i=0;i<n;i++){
		if(A[i][2] == 0 && A[i][0] <= M){
			A[i][2] = 1;
			v1 = package01(M-A[i][0], A, V + A[i][1], n);
			v2 = package01(M, A, V, n);
			return max(v1,v2);
		}
	}

	return V;
}

```
###10.Find Maxium Sub-Array
```C++
struct SubArray
{
	int low;
	int high;
	int sum;
	SubArray(int a, int b, int c):low(a),high(b),sum(c){}
};

SubArray FindMaxCrossingSubArray(int arr[], int low, int mid, int high)
{
	int left_sum = INT_MIN;
	int sum = 0;
	int max_left;
	for(int i=mid;i>=low;i--)
	{
		sum = sum + arr[i];
		if(sum > left_sum){
			left_sum = sum;
			max_left = i;
		}
	}

	int right_sum = INT_MIN;
	sum = 0;
	int max_right;
	for(int j=mid+1;j<=high;j++)
	{
		sum = sum + arr[j];
		if(sum > right_sum){
			right_sum = sum;
			max_right = j;
		}
	}

	return SubArray(max_left, max_right, left_sum+right_sum);
}

SubArray FindMaxiumSubarray(int arr[], int low, int high)
{
	if(high == low) return SubArray(low,high, arr[low]);

	int mid = (high+low)/2;

	SubArray low_sub_arr = FindMaxiumSubarray(arr, low, mid);
	SubArray high_sub_arr = FindMaxiumSubarray(arr, mid+1, high);
	SubArray crossing_sub_arr = FindMaxCrossingSubArray(arr, low, mid, high);

	if(low_sub_arr.sum >= high_sub_arr.sum && low_sub_arr.sum >= crossing_sub_arr.sum){
		return low_sub_arr;
	}else if(high_sub_arr.sum >= low_sub_arr.sum && high_sub_arr.sum >= crossing_sub_arr.sum){
		return high_sub_arr;
	}else{
		return crossing_sub_arr;
	}
}
```
###11.Haffman Tree
```C++
typedef struct TNode
{
	TNode * lchild;
	TNode * rchild;
	int weight;

	TNode(int w)
	{
		lchild = NULL;
		rchild = NULL;
		weight = w;
	}

	TNode(TNode *left, TNode *right)
	{
		lchild = left;
		rchild = right;
		weight = left->weight + right->weight;
	}

	static bool compare(TNode *a, TNode *b)
	{
		return a->weight < b->weight;
	}
} TreeNode;

typedef struct Tree
{
	bool removed;
	TNode *root;
};

TNode* haffman(list<TNode*> tlist)
{
	while(tlist.size() != 1){
		tlist.sort(TNode::compare);

		TNode *left = tlist.front();
        tlist.pop_front();

        TNode *right = tlist.front();
        tlist.pop_front();

        tlist.push_back(new TNode(left,right));
	}

	return tlist.front();
}

```
