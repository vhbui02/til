# Linux Installation documentation

## Table of Contents

1. [Introduction](#introduction)
2. [To-do List](#to-do-list)
3. [Step-by-steps](#step-by-steps)
4. [Miscellaneous](#miscellaneous)
5. [Consideration](#consideration)

## Introduction

This is my attempt to create documentation for all of my tweaks during the Linux installation. Based on the following references:

- [Bill Dietrich's Computer > Linux > Installing Linux](https://www.billdietrich.me/InstallingLinux.html)

## To-do List

## Step-by-steps

- Create partition layout

* **Method 1: LVM2**: For a no-LUKS approach, follow [HowTo: Set up Ubuntu Desktop with LVM Partitions](https://help.ubuntu.com/community/UbuntuDesktopLVM). NOTE:

> [!NOTE]
> Skip [Set up hard drive partitions](https://help.ubuntu.com/community/UbuntuDesktopLVM#Set_up_hard_drive_partitions) step and use **GParted** to format the appropriate parition with `LVM2 pv` [#ubuntu2204]()

- **Method 2: btrfs**: Set `@` as the label for the partition to make root a subvolume automatically (untested). [#fedorakde40]()

* Mount the `/boot/efi` directory onto the same partition that holds Windows's boot file. **DO NOT FORMAT EFI PARTITION IF YOU"RE DUAL BOOTING!**.

* For a 1TB drive, here is my reference when resizing partitions:

| Partition                |   Size   |
| :----------------------- | :------: |
| EFI                      |  100MB   |
| Windows recovery         |   16MB   |
| Windows system           |  145GB   |
| Windows user data        |  100GB   |
| Boot partition           | 1GB (\*) |
| Linux system + user data |  ~700GB  |

- 1GB is enough to hold 3 kernel versions
