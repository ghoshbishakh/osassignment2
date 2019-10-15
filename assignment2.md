CS60038: Assignment 2
Building a simple file system with an emulated disk
Assignment Date: 7th August 2019
Deadline: 28th August 2019, EOD

The objective of this assignment is to get hands-on experience in building a simple file system.


This assignment will involve building three components:

A disk emulator
A simple file system
Robustness and recovery / NFS interface / FUSE


Disk Emulator

Since it is difficult to test our file system on a physical raw disk, we will need to emulate it somehow. For that we are going to create a simple file based disk emulator using standard open, read, write, and seek files in Linux. This will provide APIs for block level access to the emulated disk.

The disk interface will have the following:


int create_disk(char *filename, int nbytes);
int open_disk(char *filename);
disk_stat* get_disk_stat(int disk);
int read_block(int disk, int blocknr, void *block_data);
int write_block(int disk, int blocknr, void *block_data);
int close_disk(int disk);






The disk will have fixed block size of 4KB. In addition to providing the API for reading and writing blocks, it will also maintain some stats about the disk.


The structure for the stats is as follows:

typedef struct disk {
    uint32_t size; // size of the entire disk file
    uint32_t blocks; // number of blocks (except stat block)
    uint32_t reads; // number of block reads performed
    uint32_t writes; // number of block writes performed
} disk_stat;


These stats will take up 16bytes of storage at the beginning of the disk.

Therefore, if we consider a disk of size 409600 bytes = (4KB x 100), the number of usable blocks will be 99.


int create_disk(char *filename, int nbytes);

Creates a disk file of size nbytes. Initializes disk_stat struct with calculated number of blocks reads and writes as 0. Writes it to the first block. Returns 0 if successful, -1 for error.

CHECK: Remember to close the file and free the memory for disk_stat 


int open_disk(char *filename);

Opens the disk file of given filename. Check if valid stat is present. Return the file descriptor if successful. Return error otherwise.

CHECK: File descriptor not pointer. Free the stat.


disk_stat* get_disk_stat(int disk);

Reads disk_stat from the disk. And returns its pointer.








int read_block(int disk, int blocknr, void *block_data);

Assumes that block_data is a 4KB buffer. Checks if blocknr is a valid block pointer. Reads the block contents into the buffer. Returns 0 if success, -1 otherwise. Also updates the disk_stat to increment the reads count.

CHECK: valid block pointer check is done or not. Check if memory is freed for disk_stat.

int write_block(int disk, int blocknr, void *block_data);

Assumes that the block_data is a 4KB buffer. Check if blocknr is a valid block pointer. Write the contents of the block_data buffer into the correct block. Return 0 if success, -1 otherwise. Also, update the disk_stat to increment the writes counter.
NOTE: Updating disk_stat does not affect the writes counter.


int close_disk(int disk);

Close the emulated disk file.



Simple File System SFS

Using the emulated disk we will be implementing a simple file system. Let us first take a look a the file system layout:

SFS will have the following components:
Super Block
Inodes and Inode Blocks
Data Blocks
Indirect Blocks
Inode Bitmap
Data block bitmap

Super Block
The first block of SFS will be the super block.
The super block will have the following info:

MAGIC
Blocks
Inode Blocks
Inodes
Data Block Bitmap Length
Inode Bitmap Length

typedef struct super_block {
uint32_t MagicNumber;   // File system magic number
	uint32_t Blocks;    	// Number of blocks in file system
	uint32_t InodeBlocks;   // Number of blocks reserved for inodes
	uint32_t BitMapBlocks;  // Number of blocks reserved for bitmap
	uint32_t Inodes;    	// Number of inodes in file system
} super_block;


Inodes and Inode Blocks

typedef struct inode {
	uint32_t valid; // 0 if invalid
	Uint32_t size;
	uint32_t direct[5]; // direct data block pointer
	uint32_t indirect; // indirect pointer
} inode;

Therefore the size of each inode is x bytes. Each inode block will contain y inodes.

The number of blocks assigned for inodes will be 10% of the total number of available blocks in the disk (except super block).

According to the total number of inodes, the length of the inode bit map is set.

Rest of the blocks are used for data blocks and bit map for data block.















The following interfaces needs to be implemented for SFS

