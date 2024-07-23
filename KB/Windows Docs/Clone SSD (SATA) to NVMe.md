

### Objective
***
To clone an existing OS installation from a SATA SSD to an NVMe. 
***

### Prerequisites
***
- An SSD with an existing OS installation
- An NVMe
- A USB drive with [Clonezilla](https://clonezilla.org/) bootable image
***

### Let's begin
***

For an instance, we are going to clone an existing Windows OS Installation on an SSD (SATA) into an NVMe drive

##### Step 1: Determine the type of the disk

To determine whether the existing OS is on GPT or MBR, boot into the OS.

**For Windows**:
1. Open cmd (Command Prompt) and type the following commands
2. `diskpart`
3. `DISKPART> list disk`
4. Usually Disk 0 will be the primary. Find you primary disk and if there is an asterisk (\*) under Gpt, then it's a GPT partition.
5. If not, then it's an MBR partition

**For Linux**:
1. Open Terminal and enter the following command
2. `sudo fdisk -l`
3. On **Disklabel type**, it will say gpt or mbr

***
***Note: If your disk type is GPT, skip to Step 3. If your disk type is MBR, please continue to Step 2 (Optional Step)
***

##### Step 2: Convert MBR to GPT without data loss (Optional Step)

It is recommended to use GPT partition when using latest hardware like NVMe as the MBR partition types have some limitation and are recommended for legacy/old hardware. The conversion from MBR to GPT is possible without any data loss. However, it is always recommended to take a backup of your data.

**For Windows**:

To convert an existing Windows installation from MBR (Legacy) to GPT (UEFI), do the following.

1. A Windows 10/11 ISO bootable image is needed. A tool like [Rufus](https://rufus.ie/en/)/[Ventoy](https://www.ventoy.net/en/index.html) can be used to copy the ISO to a USB drive and make it bootable.
2. Boot into the USB
3. On the Setup page, Click on **Repair your computer** option
4. Then select Troubleshoot -> Advanced Options -> **Command Prompt**
5. Then type the command: `mbr2gpt /validate`
6. If the validation fails with the error: "Disk layout validation failed", make sure there are only 4 partitions and no logical partition is present. Refer to this [YouTube video](https://www.youtube.com/watch?v=ytRJhwL6vAg) for more details
7. After the validation succeeds, enter the command: `mbr2gpt /convert`. This will convert the disk
8. Then close the command prompt an go to Troubleshoot -> Advanced Options again and select **Startup Repair**. This will clear the errors on EFI, if there are any.

**For Linux**:

Please refer to the first answer by sancho.s in the following post on Stack Exchange.
[Convert MBR to GPT](https://askubuntu.com/questions/1314111/convert-mbr-partition-to-gpt-without-data-loss)

##### Step 3: Clone the disk

Make sure both the SSD and NVMe are connected together on the PC. Then boot into the USB where Clonezilla is. Refer to this [Clonezilla USB Setup Guide](https://clonezilla.org/liveusb.php) for more details.

1. Select **Clonezilla Live** from the GRUB Menu
2. Choose your preferred Language and Keyboard Layout
3. Click on **Start_Clonezilla**. **Proceed with Caution** from this point.
4. Select **device-device**
5. Select **Expert_mode**
6. Select **disk_to_local_disk**
7. Choose the **Source** disk on the first page. Then on the next page, choose the **Target**/**Destination** disk. ***Caution**: Choosing the source and target disks wrong will lead to data loss*.
8. On **Clonezilla on-the-fly advanced extra parameters** page, unselect the first option **Reinstall GRUB on target hard disk boot sector** (For a Linux installation, you may leave this on default and continue)
9. Select **Skip checking/repairing source filesystem**
10. Select **Use partition table from the source disk**
11. Select **Choose reboot/shutdown/etc when everything is finished** and then Press **Enter** to continue.
12. Verify the target drive is ready and its going to be formatted message and Press **Y** and hit enter.
13. Repeat **step number 12**
14. The cloning process will start. It will take a while depending upon the data and the number of partitions on the disk
15. When the cloning completes, choose **Shutdown** and turn off the PC
16. Remote the SATA SSD and boot from NVMe. If you encounter "No boot device found" message, make sure to enable UEFI and boot from UEFI.

If the clone fails with the error, "This disk contains mismatched GPT and MBR partition: /dev/sdX", it is possible that the disk type is MBR and there is a GPT entry present in it. Go to **Step 4**

##### Step 4: Clone failed (This disk contains mismatched GPT and MBR partition)

If you didn't convert the disk from MBR to GPT and the error: "This disk contains mismatched GPT and MBR partition: /dev/sdX" occurs, there might a GPT entry present in the disk. As we are only using MBR in this case, it is safe to remove the GPT entry

1. Boot into the USB again and select **Clonezilla Live** from the GRUB Menu
2. Choose your preferred Language and Keyboard Layout
3. Choose **Enter_command_line**
4. Type the command `sudo gdisk /dev/sda` (sdX - where X is the letter of your disk)
5. The following output shows
```
GPT fdisk (gdisk) version 0.7.2

Partition table scan:  
MBR: MBR only
BSD: not present  
APM: not present  
GPT: present  
  
Found valid MBR and GPT. Which do you want to use?  
1 - MBR  
2 - GPT  
3 - Create blank GPT  
  
Your answer: 
```
6. For **Your answer** select 1
7. For **Command**, enter **x**
8. For **Expert Command**, enter **z**
9. For the prompt `About to wipe out GPT on /dev/sda. Proceed? (Y/N):` enter **y**
10. For the prompt `Blank out MBR? (Y/N):` enter **n**. **DO NOT** choose **Y** here as it will **wipe** all your data
11. After this, Go back to **Step 3** and start over.