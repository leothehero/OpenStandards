# BROFS - Bootloader Read-Only FileSystem

BROFS is a simple filesystem usable by bootloaders to load other filesystem drivers. It does not support directories, is limited to 64 files, and filenames are limited to 24 characters, making it in effect an organized pile of data with filenames on top.

BROFS's fixed-size file table occupies the first 1024 bytes (2 sectors) of a partition. On boot, this file table is loaded into memory and parsed.

Each inode is 32 bytes:

```c
struct Inode {
  // Bytes 1-2: Starting sector
  unsigned short start_sector;
  // Bytes 3-4: Size in bytes, rounded up to sectors.
  unsigned short size;
  // Bytes 5-6: Preallocated space-- how big can it get?
  unsigned short max_size;
  // Byte 7: flags, entry is only valid if >=1
  unsigned char flags;
  // Byte 8: Unused
  unsigned char unused;
  // Bytes 9-32: Filename (24 bytes)
  char filename[24];
}
```
