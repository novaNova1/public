# git 操作大文件时，报内存溢出，原因是没有分配交换分区


# UNIX / Linux: 2 Ways to Add Swap Space Using dd, mkswap and swapon

by RAMESH NATARAJAN on AUGUST 18, 2010

[Tweet](https://twitter.com/share)

**Question:** I would like to add more swap space to my Linux system. Can you explain with clear examples on how to increase the swap space?

**Answer:** You can either use a dedicated hard drive partition to add new swap space, or create a swap file on an existing filesystem and use it as swap space.

### How much swap space is currently used by the system?

Free command displays the swap space. free -k shows the output in KB.

```
# free -k
             total       used       free     shared    buffers     cached
Mem:       3082356    2043700    1038656          0      50976    1646268
-/+ buffers/cache:     346456    2735900
Swap:      4192956          0    4192956
```

Swapon command with option -s, displays the current swap space in KB.

```
# swapon -s
Filename                        Type            Size    Used    Priority
/dev/sda2                       partition       4192956 0       -1
```

Swapon -s, is same as the following.

```
# cat /proc/swaps
Filename                        Type            Size    Used    Priority
/dev/sda2                       partition       4192956 0       -1
```

### Method 1: Use a Hard Drive Partition for Additional Swap Space

If you have an additional hard disk, (or space available in an existing disk), create a partition using fdisk command. Let us assume that this partition is called /dev/sdc1

Now setup this newly created partition as swap area using the mkswap command as shown below.

```
# mkswap /dev/sdc1
```

Enable the swap partition for usage using swapon command as shown below.

```
# swapon /dev/sdc1
```

To make this swap space partition available even after the reboot, add the following line to the /etc/fstab file.

```
# cat /etc/fstab
/dev/sdc1               swap                    swap    defaults        0 0
```

Verify whether the newly created swap area is available for your use.

```
# swapon -s
Filename                        Type            Size    Used    Priority
/dev/sda2                       partition       4192956 0       -1
/dev/sdc1                       partition       1048568 0       -2

# free -k
             total       used       free     shared    buffers     cached
Mem:       3082356    3022364      59992          0      52056    2646472
-/+ buffers/cache:     323836    2758520
Swap:      5241524          0    5241524
```

**Note:** In the output of swapon -s command, the Type column will say “partition” if the swap space is created from a disk partition.

### Method 2: Use a File for Additional Swap Space

If you don’t have any additional disks, you can create a file somewhere on your filesystem, and use that file for swap space.

The following dd command example creates a swap file with the name “myswapfile” under /root directory with a size of 1024MB (1GB).

```
# dd if=/dev/zero of=/root/myswapfile bs=1M count=1024
1024+0 records in
1024+0 records out

# ls -l /root/myswapfile
-rw-r--r--    1 root     root     1073741824 Aug 14 23:47 /root/myswapfile
```

Change the permission of the swap file so that only root can access it.

```
# chmod 600 /root/myswapfile
```

Make this file as a swap file using mkswap command.

```
# mkswap /root/myswapfile
Setting up swapspace version 1, size = 1073737 kB
```

Enable the newly created swapfile.

```
# swapon /root/myswapfile
```

To make this swap file available as a swap area even after the reboot, add the following line to the /etc/fstab file.

```
# cat /etc/fstab
/root/myswapfile               swap                    swap    defaults        0 0
```

Verify whether the newly created swap area is available for your use.

```
# swapon -s
Filename                        Type            Size    Used    Priority
/dev/sda2                       partition       4192956 0       -1
/root/myswapfile                file            1048568 0       -2

# free -k
             total       used       free     shared    buffers     cached
Mem:       3082356    3022364      59992          0      52056    2646472
-/+ buffers/cache:     323836    2758520
Swap:      5241524          0    5241524
```

**Note:** In the output of swapon -s command, the Type column will say “file” if the swap space is created from a swap file.

If you don’t want to reboot to verify whether the system takes all the swap space mentioned in the /etc/fstab, you can do the following, which will disable and enable all the swap partition mentioned in the /etc/fstab

```
# swapoff -a

# swapon -a
```

[Tweet](https://twitter.com/share)

 
