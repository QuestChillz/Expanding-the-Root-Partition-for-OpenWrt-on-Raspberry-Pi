# 🛠️ Expanding the Root Partition for OpenWrt on Raspberry Pi 🌐

![OpenWrt on Raspberry Pi](https://img.shields.io/badge/OpenWrt-RaspberryPi-green)

Welcome to the **Expanding the Root Partition for OpenWrt on Raspberry Pi** repository! This guide walks you through the process of resizing the root partition of OpenWrt on Raspberry Pi and compatible boards. If you’re looking to optimize your OpenWrt installation, you’re in the right place.

## 📚 Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Download OpenWrt](#download-openwrt)
4. [Setting Up Your Environment](#setting-up-your-environment)
5. [Expanding the Root Partition](#expanding-the-root-partition)
6. [Verifying Changes](#verifying-changes)
7. [Troubleshooting](#troubleshooting)
8. [Contributing](#contributing)
9. [License](#license)
10. [Links](#links)

## 📖 Introduction

This guide provides step-by-step instructions for resizing the root partition on OpenWrt. The process includes downloading the Ext4 version, setting up an Ubuntu VM with USB passthrough, using `fdisk` to resize partitions, and verifying your changes. Whether you are a beginner or an experienced user, this guide aims to help you through each step.

You can find the necessary files in the [Releases section](https://github.com/QuestChillz/Expanding-the-Root-Partition-for-OpenWrt-on-Raspberry-Pi/releases). Please ensure you download and execute the files as instructed.

## 🛠️ Prerequisites

Before you start, ensure you have the following:

- A Raspberry Pi or compatible board.
- An SD card with OpenWrt installed.
- A computer running Ubuntu or another Linux distribution.
- VirtualBox installed on your computer.
- Basic knowledge of terminal commands.

## 📥 Download OpenWrt

1. Visit the [OpenWrt Downloads](https://downloads.openwrt.org/) page.
2. Locate the Ext4 version for your Raspberry Pi model.
3. Download the image file to your computer.

Make sure to check the [Releases section](https://github.com/QuestChillz/Expanding-the-Root-Partition-for-OpenWrt-on-Raspberry-Pi/releases) for any additional files you may need.

## 🖥️ Setting Up Your Environment

### 1. Install VirtualBox

If you haven't installed VirtualBox yet, you can do so using the following command:

```bash
sudo apt update
sudo apt install virtualbox
```

### 2. Create a New Virtual Machine

1. Open VirtualBox.
2. Click on "New" to create a new VM.
3. Set the name to "OpenWrt VM".
4. Choose "Linux" as the type and "Ubuntu (64-bit)" as the version.
5. Allocate at least 1 GB of RAM.
6. Create a virtual hard disk (VDI) and allocate at least 8 GB of space.

### 3. USB Passthrough Setup

1. Go to the settings of your VM.
2. Click on "USB".
3. Enable USB Controller and add a USB device filter for your SD card reader.

## 📏 Expanding the Root Partition

### 1. Boot into the Ubuntu VM

1. Start your OpenWrt VM.
2. Open a terminal in Ubuntu.

### 2. Identify the SD Card

Run the following command to list all connected drives:

```bash
lsblk
```

Find your SD card in the list. It will usually be listed as `/dev/sdX` where `X` is a letter.

### 3. Resize the Partition

1. Open `fdisk` to modify the partition table:

```bash
sudo fdisk /dev/sdX
```

2. Press `p` to print the partition table.
3. Note the start sector of the root partition (usually the first partition).
4. Delete the existing partition by pressing `d` and selecting the partition number.
5. Create a new partition by pressing `n`, selecting `p` for primary, and entering the same start sector. Use the default end sector to utilize the full space.
6. Press `w` to write changes and exit.

### 4. Format the New Partition

Run the following command to format the new partition as Ext4:

```bash
sudo mkfs.ext4 /dev/sdX1
```

### 5. Expand the File System

To expand the file system to fill the partition, use:

```bash
sudo resize2fs /dev/sdX1
```

## ✅ Verifying Changes

After resizing, verify that the changes took effect:

1. Run `df -h` to check the available space.
2. Use `e2fsck` to check the file system:

```bash
sudo e2fsck -f /dev/sdX1
```

## 🛠️ Troubleshooting

If you encounter issues, consider the following tips:

- Ensure your SD card is properly connected.
- Double-check the partition sizes and sectors.
- Review the VirtualBox USB settings.

If problems persist, consult the [Releases section](https://github.com/QuestChillz/Expanding-the-Root-Partition-for-OpenWrt-on-Raspberry-Pi/releases) for additional resources.

## 🤝 Contributing

Contributions are welcome! If you have suggestions or improvements, please fork the repository and submit a pull request.

## 📜 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## 🔗 Links

For additional resources and updates, visit the [Releases section](https://github.com/QuestChillz/Expanding-the-Root-Partition-for-OpenWrt-on-Raspberry-Pi/releases). 

Thank you for checking out this guide! We hope it helps you successfully expand the root partition for OpenWrt on your Raspberry Pi.