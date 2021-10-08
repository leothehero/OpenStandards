# File storage

Boot sectors, RAID arrays, and partition tables were all discussed in OpenUPT.md.

## Storing files

The method in which OpenFS stores files borrows from many UNIX filesystems: namely, the concept of index nodes, or inodes.

Sector 1 contains some information about the filesystem.

```c
struct {
  // Bytes 1-4: Filesystem version
  unsigned int fs_version;

  // Bytes 5-8: Reserved
  unsigned int reserved;

  // Bytes 9-512: (126 4-byte chunks) A list of free inodes, automatically repopulated when empty.
  unsigned int free_inodes[126];
}


The root inode sits on partition sector 2.

Each inode contains the corresponding file's starting sector, type (`0` or `1` for file or directory, respectively), and various metadata. Each inode takes a single 512-byte sector.

Inodes are formatted as such:

```c
struct inode {
  // Bytes 1-32: File name (32 bytes)
  char filename[32];

  // Bytes 33-34: File type (1 byte)
  char filetype;

  // Bytes 35-38: Last modified (8 bytes)
  unsigned long last_modified;

  // Bytes 39-41: Permissions (3 bytes, owner-group-other, each byte 0-7)
  char permissions[3];

  // Bytes 41-63: Owner (15 bytes, string)
  char owner[15];

  // Bytes 65-508 (4 byte chunks): pointers to file data sectors or, if inode describes a directory, to its children
  unsigned int data_sectors[111];

  // Bytes 509-512: Pointer to a sector containing more data sector pointers, or 0 if there are none
  unsigned int next_dsblock;
}
```
