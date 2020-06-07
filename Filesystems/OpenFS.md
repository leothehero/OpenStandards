# File storage

Boot sectors, RAID arrays, and partition tables were all discussed in OpenUPT.md.

## Storing files

The method in which OpenFS stores files borrows from the UNIX implementation: namely, the concept of index nodes, or inodes.

Sector 1 contains some information about the filesystem.

Bytes 1-4: Filesystem version

Bytes 5-8: Reserved

Bytes 9-512: (126 4-byte chunks) A list of free inodes, automatically repopulated when empty.


The root inode sits on partition sector 2.

Each inode contains the corresponding file's starting sector, type (`0` or `1` for file or directory, respectively), and various metadata. Each inode takes a single 512-byte sector.

Inodes are formatted as such:

Bytes 1-32: File name (32 bytes)

Bytes 33-34: File type (1 byte)

Bytes 35-38: Last modified (8 bytes)

Bytes 39-41: Permissions (3 bytes, owner-group-other, each byte 0-7)

Bytes 41-63: Owner (15 bytes, string)

Bytes 65-508 (4 byte chunks): pointers to file data sectors or, if inode describes a directory, to its children

Bytes 509-512: Pointer to a sector containing more data sector pointers, or 0 if there are none
