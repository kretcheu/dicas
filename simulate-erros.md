# Simulando erro no ext4

```
debugfs -w /tmp/foo.img
debugfs 1.43.1 (08-Jun-2016)
debugfs:  write /dev/null limpar-inode
Allocated inode: 12
debugfs:  clri limpar-inode
debugfs:  q

sudo mount /tmp/foo.img /mnt
ls -sF /mnt
/bin/ls: cannot access '/mnt/file-to-clri': Structure needs cleaning
total 12
 ? file-to-clri  12 lost+found/

dmesg | tail -3

[35156.062886] EXT4-fs (loop0): mounting ext2 file system using the ext4 subsystem
[35156.065760] EXT4-fs (loop0): mounted filesystem without journal. Opts: (null)
[35161.963603] EXT4-fs error (device loop0): ext4_lookup:1608: inode #2: comm ls: deleted inode referenced: 12

sudo umount /mnt

e2fsck -y /tmp/foo.img
e2fsck 1.43.1 (08-Jun-2016)
/tmp/foo.img contains a file system with errors, check forced.
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Entry 'file-to-clri' in / (2) has deleted/unused inode 12.  Clear? yes

Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
Inode bitmap differences:  -12
Fix? yes

Free inodes count wrong for group #0 (4, counted=5).
Fix? yes

```


