***

### Prerequisites
Install image (ISO)
Boot into the install image
Hit Shift+F10 to open the Administrator Command Prompt

### Partitioning the drive
***
We can use diskpart to partition the drive. Enter the command `diskpart` and hit enter.
For UEFI, create an EFI partition using the following command sequence:

```cmd
select disk 0
create partition efi size=512
```

