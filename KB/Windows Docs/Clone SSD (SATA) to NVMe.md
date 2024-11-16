

### Objective
***
To clone an existing OS installation from a SATA SSD to an NVMe. 
***

### Prerequisites
***
- An SSD with an existing OS installation
- An NVMe
- A USB drive with [Clonezilla](https://clonezilla.org/) bootable image
- The size of the destination drive (NVMe) should be equal to or greater than the size of the source drive (SSD)
***

### Let's begin
***

For an instance, we are going to clone an existing Windows OS Installation on an SSD (SATA) into an NVMe drive

##### Step 1: Determine the type of the disk

To determine whether the existing OS is on GPT or MBR, boot into the OS.

**For Windows**:
1. Open Command Prompt as an Administrator and type the following commands
2. `diskpart`
3. `DISKPART> list disk`
4. Usually Disk 0 will be the primary. Find you primary disk and if there is an asterisk (\*) under Gpt, then it's a GPT partition.
5. If not, then it's an MBR partition

**For Linux**:
1. Open Terminal and enter the following command
2. `sudo fdisk -l`
3. On **Disklabel type**, it will say gpt or mbr

***
	***Note: If your disk type is GPT, you may proceed with the clone. If the disk type is MBR, it should be converted to GPT (without data loss - Refer Step 4) which supports the latest hardware.***
***

###### Step 1.1: Prepare the disk for the conversion

The legacy MBR partition scheme can have 4 primary partitions or 3 primary partitions and one extended partition containing logical partitions. For the conversion from MBR to GPT to happen it is better to have only 3 primary partitions and no extended/logical partition should be present.

1. **For Windows**: Open Disk Manager (Run -> diskmgmt.msc)
2. **For Linux**: Use GUI tools like **Gparted** or command line tools like **fdisk**
3. Move the data from other partitions to the primary drive (mostly C: drive) and delete all the logical partitions
4. The remaining unallocated space can be extended or can just be left as unallocated
5. Make sure that there are only 3 partitions present.

	*Refer to this [video on YouTube](https://www.youtube.com/watch?v=ytRJhwL6vAg) for more details

##### Step 2: Clone the disk

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

If the clone fails with the error, "This disk contains mismatched GPT and MBR partition: /dev/sdX", or it is possible that the disk type is MBR and there is a GPT entry present in it. Go to **Step 3**

##### Step 3: Troubleshooting the Clonezilla errors

**Scenario 1: Clone failed - This disk contains mismatched GPT and MBR partition: /dev/sdX**

If the error "This disk contains mismatched GPT and MBR partition: /dev/sdX" occurs, there might a GPT entry present in the disk. As we are only using MBR in this case, it is safe to remove the GPT entry

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

**Scenario 2: Destination disk is too small

This happens when the size of the destination drive (NVMe) is less than the size of the source drive (SSD). In this case, the NVMe disk with a larger size should be chosen or the partitions can be cloned manually.

There is another way where Clonezilla automatically shrinks the larger drive to the destination drive by eliminating the unused space. Go to **Step 3** and proceed the cloning process as usual. At step number 8, select the option  

`-icds - Skip checking destination disk size before creating partition table`

and proceed with the rest of the steps.

##### Step 4: Convert MBR to GPT (without data loss)

It is recommended to use GPT partition when using latest hardware like NVMe as the MBR partition types have some limitation and are recommended for legacy/old hardware. The conversion from MBR to GPT is possible without any data loss. However, it is always recommended to take a backup of your data before proceeding.

**For Windows**:

The conversion of an existing Windows installation from MBR (Legacy) to GPT (UEFI) can be done in two ways.

**By booting into the OS:**
1. Boot into the cloned OS by selecting the NVMe drive (make sure Legacy boot is enabled)
2. Run the command: `mbr2gpt /disk:0 /validate /allowFullOS`
3. If the validation fails with the error: "Disk layout validation failed", Refer **Step 1.1**
4. If the validation fails with the error: "Cannot find OS partition(s) for disk 0", Refer **Step 4.1**
5. After the validation succeeds, enter the command: `mbr2gpt /disk:0 /convert /allowFullOS`. This will convert the disk to GPT
6. Then restart the PC, disable Legacy on the boot menu and boot into the PC.

**Without booting into the OS**:
1. A Windows 10/11 ISO bootable image is needed. A tool like [Rufus](https://rufus.ie/en/)/[Ventoy](https://www.ventoy.net/en/index.html) can be used to copy the ISO to a USB drive and make it bootable.
2. Boot into the USB
3. On the Setup page, Click on **Repair your computer** option
4. Then select Troubleshoot -> Advanced Options -> **Command Prompt**
5. Then type the command: `mbr2gpt /disk:0 /validate`
6. If the validation fails with the error: "Disk layout validation failed", Refer **Step 1.1**
7. If the validation fails with the error: "Cannot find OS partition(s) for disk 0", Refer **Step 4.1**
8. After the validation succeeds, enter the command: `mbr2gpt /disk:0 /convert`. This will convert the disk to GPT
9. Then close the command prompt an go to Troubleshoot -> Advanced Options again and select **Startup Repair**. This will clear the errors on EFI, if there are any.

**For Linux**:

Please refer to the first answer by sancho.s in the following post on Stack Exchange.
[Convert MBR to GPT](https://askubuntu.com/questions/1314111/convert-mbr-partition-to-gpt-without-data-loss)

###### Step 4.1 Troubleshooting the MBR2GPT error

If the error: "Cannot find OS partition(s) for disk 0", do the following
1. Boot into the OS
2. Open Disk Manager (Run -> diskmgmt.msc)
3. Right on the OS Partition (mostly C: Drive) and select **Mark Partition as Active**
4. Open Command Prompt as an administrator and type the following command
5. `bcdboot C:\Windows /s C:`
6. Replace the OS drive letter if its different from C:


***