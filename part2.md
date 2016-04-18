# a-tutorial-for-algorithm(part2)
###13.A simple allocator
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
