---
layout: default 
title:  Malloc Implementation
date:   2025-08-15 14:00:00 +0000
categories: jekyll update
permalink: /malloc
---
# Malloc Implementation

## Introduction

Note: This exercise was quite hard, and the code/ideas implemented here are primarily based on the following [tutorial](https://wiki-prog.infoprepa.epita.fr/images/0/04/Malloc_tutorial.pdf), with extra help gotten from [here](https://danluu.com/malloc-tutorial/).

`malloc` from the package `stdlib` is a function with the following signature

`void *malloc(size_t size);`

The function when called, allocates a block of size `size` and reserves the memory for writing/reading. Specifically, the program virtual memory is laid out as follows:

![Virtual memory](/assets/img/malloc/virtual_memory.png)

The text portion contains the program code, while the data portion is split into two parts, initialized memory and uninitialilzed memory. Intialized memory contains variables that are declared as auto, while the uninitialized memory houses everything that has been declared but yet to be intialized.

The stack at the top holds various pieces of data primary dealing with functions. In particular, every function has a dedicated stack frame, which stores both the function call as well as any local variables declared inside the function.

The heap, on the other hand represents memory that is reserved to be used for allocating memory dynamically at run time. Typically this is where `malloc` will allocate blocks and store data into. Typically the end of the heap is known as the program break, and is the value returned by calls to `sbrk`.

The blue portion represents memory that has yet to be mapped/allocated.

## sbrk

`sbrk(2)` is a system call which moves the program break by a certain amount (in bytes). The signature is: 

`void *sbrk(intptr_t incr);`

On most implementations, `sbrk` returns a pointer to the program break before the call. This might seem counter-intuitive, but makes sense when considering the use cases of the function. The return value of `sbrk` will point to the start of the allocated block of size `incr`. This would be especially helpful for the `malloc` implementation.

Using this format tends to ruin portability however, as some implementations will return a pointer to the program break after the increment.

Luckily the call `sbrk(0)` will return the current break point which we can then save before calling `sbrk` again to resize.

## Simple malloc

The simplest version of the `my_malloc` function can be written as follows:

```
void *my_malloc(size_t size){
    void *p;
    p = sbrk(0);
    if(sbrk(size) == (void *)-1)
        return NULL;
    return p;
}
```

When used in a simple program which keeps track and prints all of the pointers used, we can test that it works (note that 0x10 in hex is equal to 16):

```
Program ends at: 0x5640a9d3d000
After my_malloc(16), program ends at: 0x5640a9d3d010
The free block starts0x5640a9d3d000
After another my_malloc(16), program ends at: 0x5640a9d3d020
The free block starts0x5640a9d3d010
```

There are a lot of problems with this implementation. Primarly we first don't have a free function. This means that for every allocation, `my_malloc` will keep stacking memory onto the heap until it eventually runs out of room. This is incredibly inefficient. If we are not using memory, we should be able to free it, and use it in the next `my_malloc` call.To fix this problem we move to keeping track of the heap directly through a linked list.

## Linked list and the first fit malloc

Our current `my_malloc` returns a pointer to the free block, without any other data. We could try to keep track of the size of the block during the program, however the `free` function signature is as follows:

`void free(void *ptr);`

which does not accept a size argument. The question becomes, how do we know the size of the block to be freed? 

The answer lies in the form of metadata stored at the beginning of each block. We will need to modify our `my_malloc` function to allocate a slightly larger size than requested, and then store metadata in the extra bit. 

The metadata required is straightforward. We need the size, whether or not the block is free memory, and a pointer to the next block. 

While it might seem like the pointer to the next block is unecessary, we have to keep in mind that this is a virtual memory organization that might not correspond to physical addresses, i.e. if the current block is at `cur`, the next block might not necessarily be stored at `cur + header_size + data_size`.

To better keep track of the metadata we will store it in a simple link-list style structure:

```
struct block_meta {
  size_t size;
  struct block_meta *next;
  int free;
};
```

`free` will be zero if the block is used, and 1 if free.

We will store the head of the linked list in a global pointer:

`block_meta *head = NULL;`

Next, we want to write a function which will find the next free block which is greater than or equal to the `size` parameter supplied to `malloc`. By calling this function, we can re-use space that has been freed, instead of constantly allocating more space on top of the heap.

```
struct block_meta *find_free_block(struct block_meta **last,
        size_t size){
    struct block_meta cur = head;
    while(cur && !(cur->free) && cur->size >= size){
        *last = cur;
        cur = cur->next;
    }
    return cur;
}
```

Note the extra `**last` pointer that we pass as an argument. This is so that we can "return" two values. One being the pointer of the block that is free (`cur`), the other being the storage of a pointer to the previous block in `last`. This is useful to not have to go through the linked list to connect it again.

Next we handle the case of adding a block at the end of the list, provided we did not find a free block. This is very similar to our original `malloc`, except that we now have to add in the metadata and connect it to our list.

```
struct block_meta *add_block_top(struct block_meta* last,
        size_t size){
    struct block_meta *block;
    block = sbrk(0);

    if(sbrk(size + sizeof(struct block_meta)) == (void *) -1){
        printf("sbrk failed in add_block_top");
        return NULL;
    }
    //if this is not the first block
    if(last){
        //then we add it to the list
        last->next = block;
    }
    block->size = size;
    block->next = NULL; //we are at tail
    block->free = 0;
    return block;
}
```

With that we are finally ready to rewrite our `my_malloc` function. 

```
void *my_malloc(size_t size){
    struct block_meta *block;

    if(size <= 0){
        printf("Error size cannot be negative");
        return NULL;
    }

    if(head == NULL){ //first call
        block = add_block_top(NULL, size);
        if(block == NULL){
            printf("Error first block failed to allocate");
            return NULL;
        }
        head = block
    } else{
        struct block_meta *last = head;
        //find_free_block will set last equal to previous 
        //block before the free one or the last block
        //if there is no such block free
        block  = find_free_block(&last, size);
        if(block == NULL){ //no free block found
            add_block_top(last, size);
            if(block == NULL){
                printf("Error could not allocate 
                        block on top");
                return NULL;
            }
        }else{ //Free block found
            block->free = 0;
        }
    }
    return (block + 1);
}
```

We return `block + 1` to skip the metadata at the start of the block. Since `block` is a pointer of type `struct block_meta`, the 1 becomes 1 times `sizeof(struct block_meta)`. 

This function is still missing a lot. Namely we still waste a lot of space by allocating too much memory in a block. If a block is of larger size than we need, we simply grab the entire block without freeing the tail end which we will not need. 

## my_free()

None of the above optimizations actually end up mattering, unless we have a working `free` function. After all, the `find_free_block` will never find a free block unless it has been freed somehow. In fact, we can't even test out the code above.

```
void my_free(void *ptr){
    //get the address of the struct to access the metadata.
    //the metadata is stored right behind the start of the
    //data for the block
    struct block_meta *block = (struct block_meta *)ptr - 1;
    if(block->free != 0){
        printf("Error cannot free already free memory");
        return;
    }
    block->free = 1;
}
```

## split()

The last optimization to do, at least for this implementation, is to split blocks if they are too big, leaving parts of them free to re-use. 

```
void split_block(block_meta *block, size_t size){
    struct block_meta *new;
    // Cast to char to do byte precision
    new = (char *)block + sizeof(struct block_meta) 
        + sizeof(size);
    new->size = b->size - s - sizeof(struct block_meta);
    new->next = b->next;
    new->free = free;
    b->size = s;
    b->next = new;
}
```

Note the `char` cast. If we were to do the pointer arithmetic without it, then everything would be multplied by a factor of `sizeof(struct block_meta)` since `block` is of type `struct block_meta *`.

## Conclusion

Putting it all together, the final `my_malloc` function is as follows, all other functions stay the same:

```
void *my_malloc(size_t size){
    struct block_meta *block;

    if(size <= 0){
        printf("Error size cannot be negative");
        return NULL;
    }

    if(head == NULL){ //first call
        block = add_block_top(NULL, size);
        if(block == NULL){
            printf("Error first block failed to allocate");
            return NULL;
        }
        head = block
    } else{
        struct block_meta *last = head;
        //find_free_block will set last equal to previous 
        //block before the free one or the last block
        //if there is no such block free
        block  = find_free_block(&last, size);
        if(block == NULL){ //no free block found
            add_block_top(last, size);
            if(block == NULL){
                printf("Error could not allocate 
                        block on top");
                return NULL;
            }
        }else{ //Free block found
            if(b->size > sizeof(struct block_meta) + size){
                split(block, size);
            }
            b->free = 0;
        }
    }
    return (block + 1);
}
```

Note that we still have not implemented alignment.

This was an excellent exercise in both pointers and memory management, and I suggest anyone learning C to go through this at some point since it brings so many concepts in together.

Both of the resources I used (see Introduction) made frequent mentios of debugging tools, however I was able to both test and debug this program with the standard `printf`'s by just using the conversion specification `%p` to constantly print the pointers at every step during the execution. Additionally I used the following function to print the entire linked list that was being kept track of.

```
void print_list(){
    for(int i = 0, struct block_meta *cur = head; 
            cur != NULL; cur = cur->next, i++){
        printf("Block %d of size %d, is located at
                %p and %s free", i, cur->size, cur, 
                cur->free ? "is " : "is not ");
    }
}
```
