
# CS60038: Assignment 2

Building a simple file system with an emulated disk

Assignment Date: 7th August 2019\
Deadline: 28th August 2019, EOD

---

The objective of this assignment is to get hands-on experience in building a simple file system.


This assignment will involve building three components:

* **A disk emulator**
* **A simple file system: SFS**
* **FUSE interface**


## Disk Emulator

Since it is difficult to test our file system on a physical raw disk, we will need to emulate it somehow. For that we are going to create a simple file based disk emulator using standard open, read, write, and lseek calls in Linux. This will provide APIs for block level access to the emulated disk.

Our disk emulator interface will have the following:

```c
int create_disk(char *filename, int nbytes);
int open_disk(char *filename);
disk_stat* get_disk_stat(int disk);
int read_block(int disk, int blocknr, void *block_data);
int write_block(int disk, int blocknr, void *block_data);
int close_disk(int disk);
```


The disk will have fixed block size of `4KB` (4019 bytes). In addition to providing the API for reading and writing blocks, it will also maintain some stats about the disk.


The structure for the stats is as follows:

```c
typedef struct disk {
    uint32_t size; // size of the entire disk file
    uint32_t blocks; // number of blocks (except stat block)
    uint32_t reads; // number of block reads performed
    uint32_t writes; // number of block writes performed
} disk_stat;
```

These stats will take up `16 bytes` of storage at the beginning of the disk.

Therefore, if we consider a disk of size `409600 bytes = (4KB x 100)`, the number of usable blocks will be `99`.

Implement the disk emulator functions as follows:


#### `int create_disk(char *filename, int nbytes)`

Creates a disk file of size `nbytes`. Initializes `disk_stat` struct with calculated number of blocks, and reads, writes as 0. Writes it to the befinning of the disk. Returns 0 if successful, -1 for error.

**CHECK**: Remember to close the file and free the memory for disk_stat 


#### `int open_disk(char *filename)`

Opens the disk file of given filename. Check if valid stat is present. Return the **file descriptor** if successful. Return error otherwise.

**CHECK**: File descriptor not pointer. Free the stat.


#### `disk_stat* get_disk_stat(int disk)`

Reads disk_stat from the disk. And returns its pointer.


#### `int read_block(int disk, int blocknr, void *block_data)`

Assumes that `block_data` is a 4KB buffer. Checks if `blocknr` is a valid block pointer. Reads the block contents into the buffer. Returns 0 if success, -1 otherwise. Also updates the `disk_stat` to increment the `reads` count.

**CHECK:** valid block pointer check is done or not. Check if memory is freed for disk_stat.

#### `int write_block(int disk, int blocknr, void *block_data)`

Assumes that the `block_data` is a `4KB` buffer. Check if `blocknr` is a valid block pointer. Write the contents of the `block_data` buffer into the correct block. Return 0 if success, -1 otherwise. Also, update the `disk_stat` to increment the `writes` counter.\
**NOTE**: Updating disk_stat does not affect the writes counter.


#### `int close_disk(int disk)`

Close the emulated disk file.



## Simple File System SFS

Using the emulated disk we will be implementing a simple file system: SFS. Let us first take a look a the file system layout:

SFS will have the following components:
* Super Block
* Inodes and Inode Blocks
* Data Blocks
* Indirect Blocks
* Inode Bitmap
* Data block bitmap

#### Super Block
The first block of SFS will be the super block.
The super block will have the following info:

```c
typedef struct super_block {
	uint32_t magic_number;	// File system magic number
	uint32_t blocks;	// Number of blocks in file system (except super block)

	uint32_t inode_blocks;	// Number of blocks reserved for inodes == 10% of Blocks
	uint32_t inodes;	// Number of inodes in file system == length of inode bit map
	uint32_t inode_bitmap_block_idx;  // Block Number of the first inode bit map block
	uint32_t inode_block_idx;	// Block Number of the first inode block

	uint32_t data_block_bitmap_idx;	// Block number of the first data bitmap block
	uint32_t data_block_idx;	// Block number of the first data block
	uint32_t data_blocks;  // Number of blocks reserved as data blocks
} super_block;
```

#### Inodes and Inode Blocks

```c
typedef struct inode {
	uint32_t valid; // 0 if invalid
	uint32_t size; // logical size of the file
	uint32_t direct[5]; // direct data block pointer
	uint32_t indirect; // indirect pointer
} inode;
```

Therefore the size of each inode is x bytes. Each inode block will contain y inodes.

The number of blocks assigned for inodes will be 10% of the total number of available blocks in the disk (except super block).

According to the total number of inodes, the length of the inode bit map is set.

Rest of the blocks are used for data blocks and bit map for data block.


#### Data Blocks

Data blocks are the blocks used for storing actual file contents. We can also use these blocks as indirect blocks for storing indirect block pointers. Later we will be using these data blocks for storing directory structure also.

#### Indirect Blocks

These are data blocks storing pointers to other data blocks. In case direct pointers are not enought to accomodate al the block pointers, the indirect blocks will be used to store more pointers.

#### Inode Bitmap

#### Data block bitmap



The following interfaces needs to be implemented for SFS:

```c
int format(int disk);
int mount(int disk);
int create();
int remove(int inumber);
int stat(int inumber);
int read(int inumber, char *data, int length, int offset);
int write(int inumber, char *data, int length, int offset);
```

