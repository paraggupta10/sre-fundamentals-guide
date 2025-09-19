# Inode Concepts and File Operations

Understanding inodes is fundamental for Linux system administration and troubleshooting. This knowledge is essential for SRE professionals dealing with filesystem issues, storage management, and system optimization.

## Overview

An **inode** (index node) is a fundamental data structure in Unix/Linux filesystems. It stores metadata about a file or directory, but **not the actual data** or the file name. Understanding inodes is crucial for filesystem management, troubleshooting, and optimization.

## What is an Inode?

### Definition
An inode is a data structure that contains metadata about a file or directory. Each inode has a unique number (inode number) within a filesystem and serves as the filesystem's way of tracking file information.

### What an Inode Contains

Each inode typically stores:

- **File type** (regular file, directory, symlink, socket, etc.)
- **Permissions & mode** (read/write/execute bits, sticky bit, etc.)
- **Owner information** (UID, GID)
- **Timestamps**:
  - `atime`: Last access time
  - `mtime`: Last modification time (content)
  - `ctime`: Last change time (metadata)
  - `crtime`: Creation time (on some filesystems)
- **File size**
- **Link count** (how many directory entries point to this inode)
- **Pointers to data blocks** (addresses where actual file content is stored on disk)

### What an Inode Does NOT Contain

- **File name** (this is stored in the directory entry, which maps a name → inode number)
- **Actual file data** (stored in separate data blocks)

### Example Structure

```
Directory Entry:        Inode 12345:              Data Blocks:
┌─────────────────┐    ┌─────────────────────┐   ┌─────────────┐
│ notes.txt → 12345│    │ File type: regular   │   │ "Hello      │
└─────────────────┘    │ Permissions: 644     │   │ World!      │
                       │ Owner: user:group    │   │ This is     │
                       │ Size: 1024 bytes     │   │ file data"  │
                       │ Links: 1             │   └─────────────┘
                       │ Data pointers: →     │
                       └─────────────────────┘
```

## Inode Operations and File Management

### File Operations and Inode Behavior

#### 1. Copy a File to Different Location

When you copy a file (`cp file1 /new/path/file1`):
- A **new inode is created** for the copied file
- The copied file has **its own inode number** and its own data blocks
- Even if the content is identical, it's a separate file from the original

**Result**: Inode changes when you copy.

```bash
# Example
$ ls -i original.txt
123456 original.txt

$ cp original.txt copied.txt
$ ls -i copied.txt  
789012 copied.txt    # Different inode number
```

#### 2. Move (Cut/Paste) Within Same Filesystem

When you move a file (`mv file1 /new/path/file1`) **within the same filesystem**:
- The file's **inode does not change**
- Only the **directory entry mapping** (filename → inode number) is updated
- This is why moving files within the same disk/partition is very fast (just metadata update)

**Result**: Inode stays the same.

```bash
# Example
$ ls -i /home/user/file.txt
123456 /home/user/file.txt

$ mv /home/user/file.txt /tmp/file.txt
$ ls -i /tmp/file.txt
123456 /tmp/file.txt    # Same inode number
```

#### 3. Move Across Different Filesystems

When you move a file across filesystems (e.g., from `/home` to `/mnt/usb`):
- It behaves like a **copy + delete** operation
- A **new inode is created** in the target filesystem
- The old inode in the source filesystem is released (link count decremented)

**Result**: Inode changes because the file now lives in a different filesystem.

### Summary of File Operations

| Operation | Same Filesystem | Cross Filesystem | Inode Behavior |
|-----------|----------------|------------------|----------------|
| **Copy** | New inode | New inode | Always changes |
| **Move** | Same inode | New inode | Changes only cross-filesystem |
| **Rename** | Same inode | New inode | Changes only cross-filesystem |

## Hard Links vs Soft Links

### Hard Links

A **hard link** is:
- A direct pointer to the same inode as the original file
- Both the original file and the hard link **share the same inode number**
- Since they point to the same inode, they share the same data blocks
- If you delete the original file, the data is still accessible via the hard link (since inode exists until link count = 0)

#### Hard Link Characteristics
- **Same inode number** as original file
- **Cannot cross filesystems** (inodes are filesystem-specific)
- **Cannot link to directories** (prevents circular references)
- **Link count increases** in inode metadata

### Soft Links (Symbolic Links)

A **soft link** (symlink) is:
- A special file that contains the path to another file
- Has its **own inode number** (different from the target file)
- If the original file is deleted, the symlink becomes "broken"
- Can cross filesystems and link to directories

#### Soft Link Characteristics
- **Different inode number** from target
- **Can cross filesystems**
- **Can link to directories**
- **Contains path information** in its data blocks

### Example Comparison

```bash
# Create original file
$ echo "Hello World" > file1
$ ls -i file1
123456 file1

# Create hard link
$ ln file1 file_hard
$ ls -i file*
123456 file1        # Same inode
123456 file_hard    # Same inode

# Create soft link
$ ln -s file1 file_soft
$ ls -i file*
123456 file1        # Original inode
123456 file_hard    # Same as original
789012 file_soft    # Different inode
```

### Quick Comparison Table

| Aspect | Hard Link | Soft Link |
|--------|-----------|-----------|
| **Inode** | Same as original | Different inode |
| **File system** | Same filesystem only | Can cross filesystems |
| **Target deletion** | Data remains accessible | Link becomes broken |
| **Directory linking** | Not allowed | Allowed |
| **Storage overhead** | None (just directory entry) | Small (stores path) |
| **Performance** | Direct access | Extra lookup required |

## Hard Link Use Cases

### 1. Space-Efficient Backups
```bash
# Create incremental backups using hard links
cp -al /data/backup-2023-01-01/ /data/backup-2023-01-02/
# Only changed files consume additional space
```

### 2. Software Version Management
```bash
# Multiple versions sharing common files
/opt/software/v1.0/bin/program -> inode 12345
/opt/software/v1.1/bin/program -> inode 12345  # Same binary
```

### 3. Atomic File Updates
```bash
# Safe file replacement
cp new_config temp_config
ln temp_config production_config  # Atomic replacement
rm temp_config
```

## Soft Link Use Cases

### 1. Cross-Filesystem References
```bash
# Link to files on different partitions
ln -s /mnt/external/data /home/user/data
```

### 2. Flexible Path Management
```bash
# Current version pointer
ln -s /opt/app/v2.1 /opt/app/current
# Easy to update: rm /opt/app/current && ln -s /opt/app/v2.2 /opt/app/current
```

### 3. Directory Organization
```bash
# Organize files without moving them
ln -s /var/log/application /home/user/logs
```

## Inode Limitations and Troubleshooting

### Inode Exhaustion

Even with free disk space, you can run out of inodes:

```bash
# Check inode usage
$ df -i
Filesystem      Inodes   IUsed   IFree IUse% Mounted on
/dev/sda1      1000000  999999      1  100% /

# This filesystem is full of inodes but may have free space
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       10G   5.0G  4.5G  53% /
```

### Common Issues

#### 1. "No space left on device" with Available Disk Space
- **Cause**: Inode exhaustion
- **Solution**: Remove unnecessary files or increase inode count

#### 2. Hard Link Limitations
- **Cannot cross filesystems**: Hard links are filesystem-specific
- **Cannot link directories**: Prevents circular references

#### 3. Broken Symbolic Links
- **Cause**: Target file moved or deleted
- **Detection**: `ls -l` shows broken links
- **Solution**: Update or remove broken links

### Troubleshooting Commands

```bash
# Check file inode
ls -i filename
stat filename

# Find files by inode
find /path -inum 123456

# Check filesystem inode usage
df -i

# Find hard links to a file
find /path -samefile filename
find /path -inum $(stat -c %i filename)

# Find broken symbolic links
find /path -type l -exec test ! -e {} \; -print
```

## Inode Pointer Structure

### Direct and Indirect Pointers

Modern filesystems like ext4 use a sophisticated pointer structure:

```
Inode Structure:
┌─────────────────────┐
│ Metadata            │
├─────────────────────┤
│ Direct Pointers     │ → Data Block 1
│ (12 pointers)       │ → Data Block 2
│                     │ → ...
│                     │ → Data Block 12
├─────────────────────┤
│ Single Indirect     │ → Indirect Block → Data Blocks
├─────────────────────┤
│ Double Indirect     │ → Indirect Block → Indirect Blocks → Data Blocks
├─────────────────────┤
│ Triple Indirect     │ → Indirect Block → Indirect Blocks → Indirect Blocks → Data Blocks
└─────────────────────┘
```

### File Size Scaling

| Pointer Type | Block Size | Max File Size |
|--------------|------------|---------------|
| Direct (12) | 4KB | 48 KB |
| Single Indirect | 4KB | ~4 MB |
| Double Indirect | 4KB | ~4 GB |
| Triple Indirect | 4KB | ~4 TB |

## SRE Relevance

### Monitoring and Alerting
```bash
# Monitor inode usage
df -i | awk 'NR>1 {if($5+0 > 80) print $6 " is " $5 " full (inodes)"}'

# Alert on inode exhaustion
if [ $(df -i / | awk 'NR==2 {print $5}' | sed 's/%//') -gt 90 ]; then
    echo "WARNING: Root filesystem inode usage > 90%"
fi
```

### Capacity Planning
- Monitor inode usage trends
- Plan for filesystems with many small files
- Consider filesystem types based on inode needs

### Backup and Recovery
- Understand hard link behavior in backup tools
- Plan for inode-efficient backup strategies
- Consider filesystem-specific backup requirements

### Performance Optimization
- Understand inode cache and buffer behavior
- Optimize for workloads with many small files
- Consider filesystem tuning parameters

## Interview Questions & Answers

### Q1: What is an inode and what information does it store?
**Answer**: An inode (index node) is a data structure that stores metadata about a file or directory, including file type, permissions, ownership, timestamps, size, link count, and pointers to data blocks. It does NOT store the filename (stored in directory entries) or the actual file data (stored in separate data blocks).

### Q2: What happens to the inode when you copy vs move a file?
**Answer**: When copying a file, a new inode is always created with its own inode number and data blocks. When moving a file within the same filesystem, the inode number stays the same and only the directory entry is updated. When moving across filesystems, it behaves like a copy+delete operation, creating a new inode.

### Q3: What's the difference between hard links and soft links?
**Answer**: Hard links point directly to the same inode as the original file, sharing the same inode number and data blocks. They cannot cross filesystems or link to directories. Soft links (symlinks) are special files with their own inode that contains the path to the target file. They can cross filesystems and link to directories but become broken if the target is deleted.

### Q4: How can a filesystem be full even with available disk space?
**Answer**: This happens when all inodes are used up. Each file requires an inode regardless of its size, so a filesystem with many small files can exhaust inodes while still having free disk space. Check with `df -i` and clean up unnecessary files or recreate the filesystem with more inodes.

## Hands-On Exercises

### Exercise 1: Inode Exploration
```bash
# Create files and examine inodes
echo "test" > file1
ls -i file1
stat file1

# Copy and move operations
cp file1 file2
mv file1 file1_moved
ls -i file1 file2 file1_moved
```

### Exercise 2: Hard vs Soft Links
```bash
# Create original file
echo "Original content" > original.txt
ls -i original.txt

# Create hard link
ln original.txt hard_link.txt
ls -i original.txt hard_link.txt

# Create soft link
ln -s original.txt soft_link.txt
ls -il original.txt hard_link.txt soft_link.txt

# Test deletion behavior
rm original.txt
cat hard_link.txt    # Should work
cat soft_link.txt    # Should fail
```

### Exercise 3: Inode Monitoring
```bash
# Check inode usage
df -i

# Find files with most hard links
find /usr -type f -links +1 | head -10

# Find broken symbolic links
find /home -type l -exec test ! -e {} \; -print 2>/dev/null
```

### Exercise 4: Cross-Filesystem Operations
```bash
# Create test files on different filesystems
echo "test" > /tmp/test_file
echo "test" > /home/test_file

# Compare inodes
ls -i /tmp/test_file /home/test_file

# Try hard link across filesystems (should fail)
ln /tmp/test_file /home/hard_link_test 2>&1

# Try soft link across filesystems (should work)
ln -s /tmp/test_file /home/soft_link_test
ls -il /home/soft_link_test
```

## References

- [The Linux Programming Interface](https://man7.org/tlpi/)
- [Understanding Unix/Linux Programming](https://www.amazon.com/Understanding-Unix-Linux-Programming-Practice/dp/0130083968)
- [ext4 Filesystem Documentation](https://www.kernel.org/doc/html/latest/filesystems/ext4/index.html)
- [Linux Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)

---

*This guide provides comprehensive coverage of inode concepts essential for Linux system administration and SRE work.*
