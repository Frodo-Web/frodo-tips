# Clean FreeBSD cache, old data to free disk space
Here is some directories you may want to clean:
````
/var/cache/pkg/*  // 88 MB
/var/crash // 88KB
/var/log // 80 KB
````
You can also use pkg to clean package cache:
````
sudo pkg clean // 30MB
````
Old freebsd-update data:
````
sudo du -d 1 -h /var/db/freebsd-update/
>> 734 MB
````
Old kernel data:
````
sudo du -d 1 -h /boot/kernel.old
>> 140 MB
````
Some data created at ports/base system building:
````
/usr/ports/obj 
/usr/src/obj 
````
A list of files from largest to smallest:
````
du -k | sort -rn | more
````
