# Mount: NTFS signature is missing when mounting Windows 11 partition
````
mount -t ntfs /dev/device_name ./windows
...
NTFS signature is missing
````
# 1. Turn off Device encryption on Windows 11
- Type and search Device encryption settings in the Windows search bar, then click Open
- On the Device encryption field, set the option to Off
- Confirm whether you need to turn off device encryption, select Turn off to disable the device encryption function
# 2. Turn off Standard BitLocker encryption on Windows 11
- Type and search Manage BitLocker in the Windows search bar①, then click Open
- Click Turn off BitLocker on the drive that you want to decrypt.
- If the drive is under locked status, you need to click Unlock drive and type the password to turn off BitLocker.
- Confirm whether you want to decrypt your drive, then select Turn off BitLocker to start turning off BitLocker, and your drive will not be protected anymore. 
