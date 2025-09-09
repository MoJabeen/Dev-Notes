
# Flash Drive

```bash

sudo diskutil list

sudo diskutil eraseDisk FAT32 USB /dir 
sudo diskutil unmountDisk /dir 

sudo dd if=xxx.iso of=/dir bs=1m

```