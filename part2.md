# a-tutorial-for-algorithm(part2)
### 13.A simple allocator
```C++
static char *heap_listp;
static char *mem_start_brk;
static char *mem_brk;
static char *mem_max_addr;

#define WSIZE 4
#define DSIZE 8
#define CHUNKSIZE (1<<12)
#define OVERHEAD 8

#define MAX(x,y) ((x)>(y)?(x):(y))

#define PACK(size,alloc)  ((size) | (alloc))//pack a size and allocated bit into a word

#define GET(p) (*(size_t *)(p)) //get a word at address p
#define PUT(p,val) (*(size_t *)(p) = (val))//set a word at address p

#define GET_SIZE(p) (GET(p) & ~0x7)//get block size
#define GET_ALLOC(p) (GET(p) & 0x1)//get alloc bit(1-valid, 0-invalid)

#define HDRP(bp) ((char *)(bp) - WSIZE) //block header
#define FTRP(bp) ((char *)(bp) + GET_SIZE(HDRP(bp)) - DSIZE) //block footer

#define NEXT_BLKP(bp) ((char *)(bp) + GET_SIZE(((char *)(bp) - WSIZE))) //next block pointer
#define PREV_BLKP(bp) ((char *)(bp) - GET_SIZE(((char *)(bp) - DSIZE))) //prev block pointer


//Apply a chunk of memory for our memory model, all of other operations depend on this chunk of memory
void mem_init(int size)
{
	mem_start_brk = (char*)malloc(size);
	mem_brk = mem_start_brk;
	mem_max_addr = mem_start_brk + size;
}

//simple model of the system sbrk function. In this model, the heap cannot be shrunk.
void *mem_sbrk(int incr)
{
	char *old_brk = mem_brk;
	if((incr < 0) || ((mem_brk+incr)>mem_max_addr)){
		errno = ENOMEM;
		return (void *)-1;
	}
	mem_brk += incr;
	return old_brk;
}

void *coalesce(void *bp)
{
	size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
	size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));
	size_t size GET_SIZE(HDRP(bp));

	if(prev_alloc && next_alloc) {
		return bp;
	}

	else if (prev_alloc && !next_alloc) {
		size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
		PUT(HDRP(bp), PACK(size, 0));
		PUT(FTRP(bp), PACK(size, 0));
		return(bp);
	}

	else if (!prev_alloc && next_alloc) {
		size += GET_SIZE(HDRP(PREV_BLKP(bp)));
		PUT(FTRP(bp), PACK(size,0));
		PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
		return (PREV_BLKP(bp));
	}

	else {
		size += GET_SIZE(HDRP(PREV_BLKP(bp))) + 
			GET_SIZE(FTRP(NEXT_BLKP(bp)));
		PUT(HDRP(PREV_BLKP(bp)), PACK(size,0));
		PUT(FTRP(NEXT_BLKP(bp)), PACK(size,0));
		return(PREV_BLKP(bp));
	}
}


void mm_free(void *bp)
{
	size_t size = GET_SIZE(HDRP(bp));

	PUT(HDRP(bp), PACK(size, 0));
	PUT(FTRP(bp), PACK(size, 0));
	coalesce(bp);
}

static void *extend_heap(size_t words)
{
	char *bp;
	size_t size;//bytes

	size = (words % 2) ? (words+1) * WSIZE : words * WSIZE;// 8 bytes alignment
	if((int)(bp = (char *)mem_sbrk(size)) < 0)
		return NULL;

	PUT(HDRP(bp),PACK(size, 0));
	PUT(FTRP(bp),PACK(size, 0));
	PUT(HDRP(NEXT_BLKP(bp)), PACK(0, 1));

	return coalesce(bp);
}

//allocator initialization
int mm_init(void)
{
	if((heap_listp = (char *)mem_sbrk(4*WSIZE)) == NULL)
		return -1;
	PUT(heap_listp, 0);
	PUT(heap_listp+WSIZE, PACK(OVERHEAD,1));
	PUT(heap_listp+DSIZE, PACK(OVERHEAD,1));
	PUT(heap_listp+WSIZE+DSIZE, PACK(0,1));
	heap_listp += DSIZE;

	if(extend_heap(CHUNKSIZE/WSIZE) == NULL)
		return -1;
	return 0;
}

//Find a free block and return the block pointer
void *find_fit_and_place(size_t size, char *start = heap_listp)
{
	char *blkp;
	int alloc;
	int blk_size;
	int blk_alloc_size;
	int blk_remain_size;
	char *blk_remainp;
	blkp = heap_listp;

	while(1) {
		blkp = NEXT_BLKP(blkp);
		blk_size = GET_SIZE(HDRP(blkp));
		alloc = GET_ALLOC(HDRP(blkp));

		if(blk_size == 0)
			return NULL;
		
		if(alloc || blk_size < size)
			continue;

		if((blk_size - size) >= DSIZE+OVERHEAD){
			blk_remain_size = blk_size-size;
			blk_remainp = blkp + size;

			PUT(HDRP(blkp), PACK(size, 1));
			PUT(FTRP(blkp), PACK(size, 1));
			PUT(HDRP(NEXT_BLKP(blkp)), PACK(0, 1));

			PUT(HDRP(blk_remainp), PACK(blk_remain_size, 0));
			PUT(FTRP(blk_remainp), PACK(blk_remain_size, 0));

			coalesce(blk_remainp);

			return blkp;
		}else {
			PUT(HDRP(blkp), PACK(size, 1));
			PUT(FTRP(blkp), PACK(size, 1));

			return blkp;
		}
	}
}

void *mm_alloc(size_t size)
{
	size_t asize;
	size_t extendsize;
	char *bp;

	if(size<=0)
		return NULL;

	if(size <= DSIZE)
		asize = DSIZE + OVERHEAD;
	else 
		asize = DSIZE * ((size + (OVERHEAD) + (DSIZE-1)) / DSIZE);

	if((bp = (char *)find_fit_and_place(asize)) != NULL) {
		return bp;
	}

	extendsize = MAX(asize, CHUNKSIZE);
	if((bp = (char *)extend_heap(extendsize/WSIZE)) == NULL)
		return NULL;
	find_fit_and_place(asize, bp);
	return bp;
}
```
### 14. bufbomb
```C++
#include "stdafx.h"
#include "stdlib.h"
#include "ctype.h"
#include "malloc.h"
#include <iostream>
#include <fstream>
using namespace std;

/* Like gets, except that characters are typed as pairs of hex digits. 
Nondigit characters are ignored. Stops when encounters newline */ 
char*getxs(char*dest) 
{ 
	int c; 
	int even =1; /* Have read even number of digits */ 
	int otherd =0; /* Other hex digit of pair */ 
	char*sp = dest; 

	std::fstream fs("C:\\Users\\mdx\\Documents\\Visual Studio 2012\\Projects\\assemble\\Debug\\test.txt",std::fstream::in);
	cout<<fs.fail()<<endl;
	char str[100];
	char buf[2048];
	int i = 0;
	int j = 0;
	memset(str,0,100);
	memset(buf,0,2048);
	while(!fs.eof()){
		fs.getline(str,100);
		memcpy(buf+i,str+12,11);
		buf[i+11] = 0x20;
		i += 12;
	}


	while ((c = buf[j++]) != EOF && c !='\n' && c != 0) { 
		if (isxdigit(c)) { 
			int val; 
			if ('0'<= c && c <='9') 
				val = c -'0'; 
			else if ('A'<= c && c <='F') 
				val = c -'A'+10; 
			else 
				val = c -'a'+10; 
			if (even) { 
				otherd = val; 
				even =0; 
			}
			else { 
				*sp++= otherd *16+ val; 
				even =1; 
			} 
		} 
	} 
	*sp++='\0'; 
	return dest; 
} 

/* $begin getbuf-c */ 
int getbuf() 
{
	char *buf = (char*) alloca(16); 
	getxs(buf); 
	return 1; 
} 

void test() 
{ 
	int val;
	printf("Please enter hex string:\n");
	val = getbuf();
	printf("Result is %d", val);
} 
/* $end getbuf-c */ 

int _tmain(int argc, _TCHAR* argv[])
{
	test();
	system("pause");
	return 0;
}


```

### epoll example
```C/C++
 EPOLLIN | EPOLLET;
	event.data.fd = sock;
	epoll_ctl(epd, EPOLL_CTL_ADD, sock, &event);

	for (;;) {
		int n = epoll_wait(epd, events, 1024, -1);
		int i;
		for (i=0; i< n;i++) {
			printf("event fd %d\n", events[i].data.fd);
			printf("event bit mask %x\n", events[i].events);
			if (events[i].events & EPOLLERR || events[i].events & EPOLLHUP || !events[i].events & EPOLLIN) {
				fprintf(stderr, "epoll error\n");
				close(events[i].data.fd);
				continue;
			} else if (events[i].data.fd == sock) {
				sock2 = accept(sock, (struct sockaddr *)&sa, &socklen);
				if (sock2 == -1) {
					continue;
					}
					int flags = fcntl(sock2, F_GETFL, 0);
				flags |= O_NONBLOCK;
				fcntl(sock2, F_SETFL, flags);
				event.data.fd = sock2;
				epoll_ctl(epd, EPOLL_CTL_ADD, sock2, &event);
			} else if (events[i].events & EPOLLIN) {
				int done;
				char buff[512];
				for (;;) {
					int count = read(events[i].data.fd, buff, 512);
					if (count == -1) {
						break;
					} else if (count == 0) {
						break;
					}
					write(1, buff, count);
				}
			}
		}

	}
	return 0;
}
```
