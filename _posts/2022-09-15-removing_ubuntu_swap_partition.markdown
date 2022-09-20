---
layout: custom
title: Airflow tutorial
date: 2022-09-13 14:27:00 +0900
last_modified_at: 2022-09-13 14:27:00 +0900
category: mlops
tags: ["ubuntu"]
published: false

---
> airflow tutorial

## Swap partition 삭제

- 참고: [https://mirrors.tripadvisor.com/centos-vault/3.9/docs/html/rhel-sag-en-3/s1-swap-removing.html](https://mirrors.tripadvisor.com/centos-vault/3.9/docs/html/rhel-sag-en-3/s1-swap-removing.html)
- 참고: [https://mirrors.tripadvisor.com/centos-vault/3.9/docs/html/rhel-sag-en-3/s1-swap-removing.html](https://mirrors.tripadvisor.com/centos-vault/3.9/docs/html/rhel-sag-en-3/s1-swap-removing.html)

- 사용중인 swap space size 확인
    ```bash
    $ free -h
    ```

- swap partition 검색 및 식별
    ```bash
    $ lsblk
    ```

- disable all swaps
    ```bash
    $ swapoff -a
    ```

- Remove its entry from /etc/fstab
    ```bash
    $ sudo vi /etc/fstab
    ```

- Remove the partition using parted:

At a shell prompt as root, type the command parted /dev/hdb, where /dev/hdb is the device name for the hard drive with the swap space to be removed.

At the (parted) prompt, type print to view the existing partitions and determine the minor number of the swap partition you wish to delete.

At the (parted) prompt, type rm MINOR, where MINOR is the minor number of the partition you want to remove.

Warning	Warning
 	
Changes take effect immediately; you must type the correct minor number.

Type quit to exit parted.


    - fdisk 명령으로 swap 파티션의 type을 변경