Windows 10 Dual Boot
====================

Shrinking the Encrypted Root Filesystem
---------------------------------------

### Open Encrypted Filesystem

    root@archiso ~ # cryptsetup open /dev/nvme0n1p2 cryptroot
    Enter passphrase for /dev/nvme0n1p2: YOUR-FILESYSTEM-ENCRYPTION-PASSWORD
    cryptsetup open /dev/nvme0n1p2 cryptroot  6.25s user 0.26s system 101% cpu 6.396 total

### Verify ext4 Partition Integrity

    root@archiso ~ # e2fsck -f /dev/mapper/cryptroot
    e2fsck 1.45.6 (20-Mar-2020)
    Pass 1: Checking inodes, blocks, and sizes
    Pass 2: Checking directory structure
    Pass 3: Checking directory connectivity
    Pass 4: Checking reference counts
    Pass 5: Checking group summary information
    /dev/mapper/cryptroot: 311359/62480384 files (0.4% non-contiguous), 12010238/249915729 blocks

### Resize ext4 Filesystem

To resize the ext4 filesystem to e.g. 700 GB:

    root@archiso ~ # resize2fs -p /dev/mapper/cryptroot 700G
    resize2fs 1.45.6 (20-Mar-2020)
    Resizing the filesystem on /dev/mapper/cryptroot to 183500800 (4k) blocks
    Begin pass 3 (max = 6000)
    Scanning inode table          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    The filesystem on /dev/mapper/cryptroot is now 183500800 (4k) blocks long.

    root@archiso ~ #

### Calculate Filesystem's Current Size

    root@archiso ~ # dumpe2fs -h /dev/mapper/cryptroot | grep '^Block'
    dumpe2fs 1.45.6 (20-Mar-2020)
    Block count:              183500800
    Block size:               4096
    Blocks per group:         32768

In the above case, calculate `183500800 * 4096` to get the total number of bytes, then divide by `1024 * 1024 * 1024` to get the number of gigabytes. Confirm that it matches the amount specified above.

### Verify ext4 Partition Integrity Again

    root@archiso ~ # e2fsck -f /dev/mapper/cryptroot
    e2fsck 1.45.6 (20-Mar-2020)
    Pass 1: Checking inodes, blocks, and sizes
    Pass 2: Checking directory structure
    Pass 3: Checking directory connectivity
    Pass 4: Checking reference counts
    Pass 5: Checking group summary information
    /dev/mapper/cryptroot: 311359/45875200 files (0.4% non-contiguous), 10967247/183500800 blocks

### Resize the Encrypted Filesystem

First, determine the sector size of the encrypted filesystem.

    root@archiso ~ # cryptsetup status /dev/mapper/cryptroot
    /dev/mapper/cryptroot is active
      type:    LUKS2
      cipher:  aes-xts-plain64
      keysize: 512 bits
      key location: keyring
      device:  /dev/nvme0n1p2
      sector size: 512
      offset:  32768 sectors
      size:    1999325839 sectors
      mode:    read/write

In the above example, the sector size is 512 bytes. Thus, the partition size is 1,023,654,829,568 bytes, or 953 GB (1,999,325,839 sectors, 512 bytes each). In addition, the partition is offset by 16 MB, or 32,768 sectors. Calculate the amount of size needed to accommodate the ext4 partition. In the above example of a 700GB partition, 1,468,006,400 sectors would be needed.

    root@archiso ~ # cryptsetup -b 1468006400 resize /dev/mapper/cryptroot
    Enter passphrase for /dev/nvme0n1p2:
    cryptsetup -b 1468006400 resize /dev/mapper/cryptroot  5.56s user 0.25s system 88% cpu 6.562 total

Verify the changes again with `cryptsetup status`:

    root@archiso ~ # cryptsetup status /dev/mapper/cryptroot
    /dev/mapper/cryptroot is active
      type:    LUKS2
      cipher:  aes-xts-plain64
      keysize: 512 bits
      key location: keyring
      device:  /dev/nvme0n1p2
      sector size: 512
      offset:  32768 sectors
      size:    1468006400 sectors
      mode:    read/write

Calculate the number of sectors needed to house the smaller encrypted filesystem by adding `offset` and `size` from above. In this example, the total number of sectors needed is 1,468,039,168 (`32768` + `1468006400` = `1468039168`).

### Verify The ext4 Filesystem Again

Verify the ext4 filesystem still works:

    root@archiso ~ # e2fsck -f /dev/mapper/cryptroot
    e2fsck 1.45.6 (20-Mar-2020)
    Pass 1: Checking inodes, blocks, and sizes
    Pass 2: Checking directory structure
    Pass 3: Checking directory connectivity
    Pass 4: Checking reference counts
    Pass 5: Checking group summary information
    /dev/mapper/cryptroot: 311359/45875200 files (0.4% non-contiguous), 10967247/183500800 blocks

### Close Encrypted Filesistem

    root@archiso ~ # cryptsetup close /dev/mapper/cryptroot

### Resize the Partitions

NOTE: This step is irreversible.

Start a `parted` session on the disk, `/dev/nvme0n1`:

    root@archiso ~ # parted /dev/nvme0n1
    GNU Parted 3.3
    Using /dev/nvme0n1
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted)

Next, set Parted's unit of measurement to "sector" (512 bytes) by typing `unit`, pressing ENTER, then typing `s`, then pressing ENTER again:

    (parted) unit
    Unit?  [compact]? s
    (parted)

Print the partitions, whose size measurements should now be in sectors:

    (parted) p
    Model: INTEL SSDPEKNW010T8 (nvme)
    Disk /dev/nvme0n1: 2000409264s
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags:

    Number  Start     End          Size         File system    Name                  Flags
     1      2048s     1050623s     1048576s     fat32          EFI system partition  boot, esp
     2      1050624s  2000409230s  1999358607s                 Linux filesystem

Note the start of the root filesystem partition (`1050624s`, or 1,050,624 sectors). Determine the new end sector of the partition by adding the starting sector to the total number of sectors needed by the encrypted partition that was calculated previously (`1468039168` in this example), and then subtracting by one (since Parted's end sector is inclusive of the final sector in the partition).

>  1050624 + 1468039168 - 1 = 1469089791

Resize the partition by giving its number (`2`) and the desired end sector (`1469089791`) to the `resizepart` command, agreeing to the associated risk of data loss:

    (parted) resizepart 2 1469089791
    Warning: Shrinking a partition can cause data loss, are you sure you want to continue?
    Yes/No? Yes
    (parted)

Verify the partition was correctly resized by changing the unit to gigabytes (`GiB`) and then invoking `p` again:

    (parted) unit
    Unit?  [compact]? GiB
    (parted) p
    Model: INTEL SSDPEKNW010T8 (nvme)
    Disk /dev/nvme0n1: 954GiB
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags:

    Number  Start     End          Size         File system    Name                  Flags
     1      0.00GiB   0.50GiB      0.50GiB      fat32          EFI system partition  boot, esp
     2      0.50GiB   701GiB       700GiB                      Linux filesystem

Finally, exit `parted` with `q`:

    (parted) q
    Information: You may need to update /etc/fstab.
    root@archiso ~ #

### Verify the Partition

Re-open the encrypted partition with `cryptsetup` to ensure it can still be decrypted:

    root@archiso ~ # cryptsetup open /dev/nvme0n1p2 cryptroot
    Enter passphrase for /dev/nvme0n1p2: YOUR-FILESYSTEM-ENCRYPTION-PASSWORD
    cryptsetup open /dev/nvme0n1p2 cryptroot  6.25s user 0.26s system 101% cpu 6.396 total

Verify the integrity of the ext4 root filesystem:

    root@archiso ~ # e2fsck -f /dev/mapper/cryptroot
    e2fsck 1.45.6 (20-Mar-2020)
    Pass 1: Checking inodes, blocks, and sizes
    Pass 2: Checking directory structure
    Pass 3: Checking directory connectivity
    Pass 4: Checking reference counts
    Pass 5: Checking group summary information
    /dev/mapper/cryptroot: 311359/62480384 files (0.4% non-contiguous), 12010238/249915729 blocks

Mount the ext4 root filesystem and ensure it has contents:

    root@archiso ~ # mount /dev/mapper/cryptroot /mnt
    root@archiso ~ # ls /mnt
    bin  boot  dev  etc  home  lib  lib64  lost+found  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
    root@archiso ~ #

Reboot and verify that Arch still functions properly.

    root@archiso ~ # umount /mnt
    root@archiso ~ # reboot

Install Windows
---------------
Reboot into a Windows installation media. Be sure to select 'custom install' and not 'express install,' so that Windows will get installed in the free space that was allocated previously.

TODO: tell Windows to use UTC, so that it doesn't ruin the clock for Linux

TODO: driver installation
