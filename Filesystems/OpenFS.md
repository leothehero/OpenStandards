# File storage

Boot sectors, RAID arrays, and partition tables were all discussed in OpenUPT.md.

## Storing files

The method in which OpenFS stores files borrows from the UNIX implementation; namely, the concept of index nodes or inodes.

The root inode sits on partition sector 2.

Each inode contains the corresponding file's starting sector, ending sector, and type (`0` or `1` for file or directory, respectively). Each of the inode sectors can allocate 32 files, equating to 2560 total allocable files on a 4-megabyte partition.

Inodes are formatted as such:

Bytes 1-32: File name (32 bytes)

Bytes 33-34: File type (1 byte)

Bytes 35-38: Last modified (4 bytes)

Bytes 39-41: Permissions (3 bytes, owner-group-other, each byte 0-7)

Bytes 41-63: Owner (15 bytes, string)

Bytes 64-508 (4 byte chunks): pointers to file data sectors

Bytes 509-512: Pointer to a sector containing more data sector pointers