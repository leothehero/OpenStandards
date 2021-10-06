# The Cynosure File System

This document describes the Cynosure File System.

The Cynosure File System is a filesystem similar to the Second Extended Filesystem (ext2) but optimized for small drives.  It will scale to large drives, though it is better suited for flash storage than for magnetic storage as it makes no effort to prevent fragmentation.

Both this document and the filesystem itself assume a sector size of 512 bytes.  All timestamps are stored as a number of seconds since 12AM on January 1st, 1970.

The first sector of any CFS drive shall contain the boot sector, containing code to load a boot-loader from somewhere on the disk.  After this comes the superblock.

#### 1.1.  Superblock structure
```c
struct Superblock {
  unsigned char revision;
  // see OS/OSID.md in this repository
  unsigned char osid;
  // unused in revision 0
  unsigned char flags;
  // statii:
  // 0: not valid
  // 1: OK
  // 2: not cleanly unmounted, run fsck before mounting again.
  unsigned char status;
  // time of filesystem creation
  unsigned long ctime;
  char uuid[16];
  unsigned long inodes;
  unsigned long blocks;
  unsigned long used_inodes;
  unsigned long used_blocks;
  char label[64];
  // the rest of the 512 bytes is unused
}
```

Immediately following the superblock are the inode and block bitmaps.  These shall be sector-aligned - i.e. they shall always be padded to the nearest sector.  Their actual size in bytes may be determined by examining the `Superblock.inodes` and `Superblock.blocks` fields.

Following these is the inode table itself.  All inodes in the filesystem are stored here.

Each inode is 64 bytes in size.  There are 8 inodes per sector.  Inodes do not contain a filename - these are stored in directory entries.

#### 1.2. Inode structure
```c
struct Inode {
  // posix file mode
  unsigned short mode;
  unsigned short uid;
  unsigned short gid;
  // creation time
  unsigned long ctime;
  // access time
  unsigned long atime;
  // modification time
  unsigned long mtime;
  // size in bytes
  unsigned long size;
  // number of times this inode has been linked
  unsigned short nlink;
  // the first in the sequence of data blocks belonging to this inode
  unsigned long dblock;
  // OS-dependent structure
  char osd[16];
}
```

#### 1.3. POSIX file modes
| Value  | Meaning                      |
| :----  | :--------------------------- |
| | -- high 4 bits determine type --    |
| 0xC000 | Socket                       |
| 0xA000 | Symbolic Link                |
| 0x8000 | Regular file                  |
| 0x6000 | Block device                 |
| 0x4000 | Directory                    |
| 0x2000 | Character device             |
| 0x1000 | FIFO                         |
| | -- lower 12 bits set permissions -- |
| 0x0800 | setuid bit                   |
| 0x0400 | setgid bit                   |
| 0x0200 | sticky bit                   |
| 0x0100 | owner can read               |
| 0x0080 | owner can write              |
| 0x0040 | owner can execute            |
| 0x0020 | group can read               |
| 0x0010 | group can write              |
| 0x0008 | group can execute            |
| 0x0004 | others can read              |
| 0x0002 | others can write             |
| 0x0001 | others can execute           |

The first inode (inode 0) is always the root directory.  Its `dblock` is always `0`.  Wherever a `0` is encountered during traversal of block groups, it shall mark the termination of that block group.

Block groups specify a contiguous chunk of data that may be split across any arbitrary number of blocks.  All block groups are stored with the structure shown in `1.4`.

#### 1.4. Datablock structure
```c
struct Datablock {
  char data[1016];
  unsigned long next;
}
```

If the value of `next` is `0`, it marks the end of that block group.

Directories are stored as an array of `Dirent`s (see `1.5`) inside a block group.

#### 1.5.  Dirent structure
```c
struct Dirent {
  // Length in bytes of the entry.  May be used to jump over certain areas - for example, a directory index.
  // Must not be smaller than name_len + 8.
  unsigned short ent_len;
  unsigned long inode;
  // The POSIX file type of this directory entry.
  unsigned char file_type;
  // The name is effectively a length-prefixed string and may be parsed as such
  unsigned char name_len;
  char name[name_len];
}
```

## Recommended Filesystem Sizes
Here are the recommended inode and block counts for various filesystem sizes.  All sizes are power-of-2.

### OpenComputers drives
| Size | Inodes | Blocks | Comment |
| ---- | ------ | ------ | ------- |
| 512K | 384    | 486    | Default floppy disk |
| 1MB  | 640    | 982    | Default T1 hard disk |
| 2MB  | 1024   | 1982   | Default T2 hard disk |
| 4MB  | 2048   | 3966   | Default T3 hard disk |
| 12MB | 4096   | 12029  | Tier 3 RAID |

### Real-world drives
| Size   | Inodes | Blocks  | Comment |
| ------ | ------ | ------- | ------- |
| 1.44MB | 768    | 1390    | Standard 3.5" floppy |
| 8MB    | 5120   | 7869    |  |
| 100MB  | 65535  | 98283   |  |
| 512MB  | 131072 | 516016  |  |
| 1GB    | 204800 | 1035623 |  |
